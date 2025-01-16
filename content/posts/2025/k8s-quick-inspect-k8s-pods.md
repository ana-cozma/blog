---
title: "Quickly Inspect Kubernetes Pods' Environment Variables and Secrets"
description: "Quickly inspect your Kubernetes Pod configuration environment variables and secrets with this script."
date: 2025-01-16T12:12:48+01:00
draft: false
tags: ["kubernetes", "devops"]
---
Something that I love is to be able to troubleshoot fast (or at least have the right tooling for it). Working in a Kubernetes environment, I often find myself needing to get pods environment variables and secrets either for debugging purposes or auditing. I wanted something that would give me everything I need fast and at a glance in my terminal.

In this blog post I'll share the little script I created to achieve this and how to add it to your shell as a function.

## Why Extract Environment Variables and Secrets?

These are some of the cases I found myself needing to have access to the values for:

- **Debugging configuration issues:** Confirming that environment variables are correctly injected.
- **Auditing security:** Ensuring secrets are appropriately configured and securely resolved.
- **Monitoring:** Verifying the dynamic configuration at runtime.

## Retrieving Environment Variables and Secrets in One Command

I used a combination of `kubectl` and `jq` to extract all environment variables, including those that reference Kubernetes secrets.

Here's the full script:

```bash
kubectl get pod <pod-name> -n <namespace> -o json | jq -r '
  .spec.containers[].env[] |
  if .valueFrom.secretKeyRef.name then
    "Secret: \(.name) -> \(.valueFrom.secretKeyRef.name)/\(.valueFrom.secretKeyRef.key)"
  elif .value then
    "Variable: \(.name) -> \(.value)"
  else
    "Other: \(.name) -> \(.)"
  end
' | while read line; do
  if [[ $line == Secret:* ]]; then
    secret_name=$(echo $line | awk -F "-> " '{print $2}' | awk -F "/" '{print $1}')
    secret_key=$(echo $line | awk -F "-> " '{print $2}' | awk -F "/" '{print $2}')
    resolved_value=$(kubectl get secret "$secret_name" -n <namespace> -o json | jq -r ".data.\"$secret_key\" | @base64d")
    echo "$line (Resolved Value: $resolved_value)"
  elif [[ $line == Other:* ]]; then
    echo "$line"
  else
    echo "$line"
  fi
done

```

What the script does:

1. **Environment Variables Directly Assigned:**

    For variables with a direct value (`value`), it prints:\
    `Variable: <name> -> <value>`.

1. **Environment Variables Referencing Secrets:**

    For variables referencing secrets (`valueFrom.secretKeyRef`), it prints:\
    `Secret: <name> -> <secret-name>/<key>`.

1. **Secrets Decoding:**

    The script fetches the secret using `kubectl get secret`, decodes it with jq's `@base64d` function, and appends the resolved value.

1. **Handling "Other" Configurations:**

    It outputs additional details for variables configured with references like `fieldRef`, making it easier to debug and prints it:\
    `Other: <name>`.

    The "Other" section captures configurations like:

    - `fieldRef:` References fields of the pod, such as status.podIP.
    - `resourceFieldRef`: References specific resources, such as limits or requests. These are less common but can be vital for debugging dynamic pod configurations.

1. **Dynamic Input:**

    You can pass the pod name and namespace dynamically to analyze any pod's configuration.

This script will give you an *output* similar to this one:

```bash
Other: MY_POD_IP -> {"name":"MY_POD_IP","valueFrom":{"fieldRef":{"fieldPath":"status.podIP"}}}
# Environment variable dynamically set to the pod's IP address.

Variable: AGENT_VERSION -> 0.0.1
# Static environment variable with a direct value.

Secret: LISTENER_URL -> my-pod-secret/my-pod-metrics-listener (Resolved Value: https://listener.io)
# Secret referencing a Kubernetes secret, resolved to its decoded value.

Secret: SPM_TOKEN -> my-pod-secret/my-pod-spm-shipping-token (Resolved Value: abc1234tokenvalue)
# Another secret resolved to its decoded token value.
```

But I don't want to remember this script every time I need to run it. Which takes us to the next step.

## Automating with a Shell Function

### Option 1: Create a GitHub-Style Alias in Your Shell

To make this more convenient, you can create a shell function and add it to your `.bashrc` or `.zshrc`:

```bash
k8s-pod-env-secrets() {
    POD_NAME=$1
    NAMESPACE=$2

    if [[ -z $POD_NAME || -z $NAMESPACE ]]; then
        echo "Usage: k8s-pod-env-secrets <pod-name> <namespace>"
        return 1
    fi

    kubectl get pod "$POD_NAME" -n "$NAMESPACE" -o json | jq -r '
      .spec.containers[].env[] |
      if .valueFrom.secretKeyRef.name then
        "Secret: \(.name) -> \(.valueFrom.secretKeyRef.name)/\(.valueFrom.secretKeyRef.key)"
      elif .value then
        "Variable: \(.name) -> \(.value)"
      else
        "Other: \(.name) -> \(.)"
      end
    ' | while read line; do
      if [[ $line == Secret:* ]]; then
        secret_name=$(echo $line | awk -F "-> " '{print $2}' | awk -F "/" '{print $1}')
        secret_key=$(echo $line | awk -F "-> " '{print $2}' | awk -F "/" '{print $2}')
        resolved_value=$(kubectl get secret "$secret_name" -n "$NAMESPACE" -o json | jq -r ".data.\"$secret_key\" | @base64d")
        echo "$line (Resolved Value: $resolved_value)"
      elif [[ $line == Other:* ]]; then
        echo "$line"
      else
        echo "$line"
      fi
    done
}
```

After adding this function, simply reload your shell:

```bash
source ~/.bashrc
```

{{<tip>}}
If you use multiple environments, consider adding a check to confirm the current context with: `kubectl config current-context`
{{</tip>}}

And in order to use the function created you can simply run:

```bash
k8s-pod-env-secrets <pod-name> <namespace>
```

### Option 2: Create a Shell Script

This is very similar to the first approach. Save the following script to a file, for example, `get-pod-env-secrets.sh`:

```sh
#!/bin/bash

POD_NAME=$1
NAMESPACE=$2

if [[ -z $POD_NAME || -z $NAMESPACE ]]; then
    echo "Usage: $0 <pod-name> <namespace>"
    exit 1
fi

kubectl get pod "$POD_NAME" -n "$NAMESPACE" -o json | jq -r '
  .spec.containers[].env[] |
  if .valueFrom.secretKeyRef.name then
    "Secret: \(.name) -> \(.valueFrom.secretKeyRef.name)/\(.valueFrom.secretKeyRef.key)"
  elif .value then
    "Variable: \(.name) -> \(.value)"
  else
    "Other: \(.name)"
  end
' | while read line; do
  if [[ $line == Secret:* ]]; then
    secret_name=$(echo $line | awk -F "-> " '{print $2}' | awk -F "/" '{print $1}')
    secret_key=$(echo $line | awk -F "-> " '{print $2}' | awk -F "/" '{print $2}')
    resolved_value=$(kubectl get secret "$secret_name" -n "$NAMESPACE" -o json | jq -r ".data.\"$secret_key\" | @base64d")
    echo "$line (Resolved Value: $resolved_value)"
  else
    echo "$line"
  fi
done
```

Make the script executable:

```bash
chmod +x get-pod-env-secrets.sh
```

Run the script with passing the pod name and namespace:

```bash
./get-pod-env-secrets.sh <pod-name> <namespace>
```

And that is it! Whether you're troubleshooting, auditing, or monitoring, this script provides all the information you need in one glance.

*Hope this helps someone else as well as it did me when I was troubleshooting and debugging!*