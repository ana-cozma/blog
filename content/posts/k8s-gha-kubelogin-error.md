---
title: "K8s: Solve AKS authentication errors in GitHub Actions by leveraging kube_admin_config_raw"
description: "Learn how to resolve AKS authentication errors in GitHub Actions by leveraging kube_admin_config_raw for seamless CI/CD pipeline integration."
date: 2024-11-26T14:30:10+01:00
draft: true
tags: ["kubernetes", "devops", "aks", "iac", "terraform"]
---
Recently I was using Azure Kubernetes Service (AKS) clusters integrated with Microsoft Entra ID (formerly Azure AD) and I encountered the following error during the running of the CI/CD pipelines in GitHub Actions:

```bash
kubelogin is not installed which is required to connect to AAD enabled cluster.
```

In this article I'd like to explain why this happens and what you can do about it.

## What Changed in AKS Authentication

Starting with Kubernetes version 1.24, AKS introduced a token-based authentication mechanism with kubelogin to improve integration with Microsoft Entra ID. While this is beneficial for interactive logins and RBAC enforcement, it can introduce challenges for automation scenarios, where token expiration and additional dependencies complicate workflows.

Read more on [managed identities on AKS](https://learn.microsoft.com/en-us/azure/aks/enable-authentication-microsoft-entra-id).

{{<note>}}
*Microsoft Entra integrated clusters using a Kubernetes version newer than version 1.24 automatically use the kubelogin format. Starting with Kubernetes version 1.24, the default format of the clusterUser credential for Microsoft Entra ID clusters is exec, which requires kubelogin binary in the execution PATH. There is no behavior change for non-Microsoft Entra clusters, or Microsoft Entra ID clusters running a version older than 1.24. Existing downloaded kubeconfig continues to work. An optional query parameter format is included when getting clusterUser credential to overwrite the default behavior change. You can explicitly specify format to azure if you need to maintain the old kubeconfig format.*
{{</note>}}

Installing and configuring kubelogin in a CI/CD environment is an option, but it can introduce complexity, including dependency management and token expiration issues.

## Differences between kube_admin_config_raw and kube_config_raw

If you check the `azurerm_kubernetes_cluster` resource you'll notice that there are actually two attributes related to the kube config and I though it would be nice to clarify them a bit since I also wondered which one I should or could use and what the impact would be.

`kube_admin_config_raw`

This is the raw kubeconfig file for the Kubernetes cluster admin account.
It uses a *certificate-based authentication* mechanism tied to the admin credentials.
And it provides *cluster-wide admin privileges*, meaning it can manage every resource and namespace in the cluster.

What you'd use it for:

- Initial cluster setup.
- Administrative tasks requiring elevated permissions.
- Automation tasks (e.g., GitHub Actions pipelines) that need unrestricted access to the cluster.

`kube_config_raw`

The second one is the raw kubeconfig file for Microsoft Entra ID (formerly Azure AD)-enabled user authentication.
This one relies on *external token-based authentication* (e.g., kubelogin) to connect. This integrates with Microsoft Entra ID for managing user access.
The *level of access depends on the user's role assignments and permissions within the cluster.*

What you'd use it for:

- Individual developers or users who log in interactively via Microsoft Entra ID.
- Scenarios requiring Role-Based Access Control (RBAC) enforcement to limit permissions.

**Some differences to keep in mind:**

| **Feature**               | **kube_admin_config_raw**                             | **kube_config_raw**                                       |
|---------------------------|-------------------------------------------------------|----------------------------------------------------------|
| **Authentication**         | Certificate-based (admin credentials).                | Token-based (Microsoft Entra ID + kubelogin).            |
| **Access Level**           | Full admin access (no RBAC restrictions).             | Role-based access (RBAC enforced).                       |
| **Dependencies**           | No external tools required.                           | Requires kubelogin and Microsoft Entra ID setup.         |
| **Ideal Use Case**         | Automation, CI/CD pipelines, cluster admin tasks.     | Interactive user logins, RBAC-restricted tasks.          |
| **Ease of Use**            | Simple and self-contained.                            | Complex setup; requires kubelogin.                       |

## Advantages of kube_admin_config_raw for Automation

If you're using Terraform: a simpler and more robust solution is to use the value of the `kube_admin_config_raw` output from your AKS cluster deployment as a Kubernetes secret in GitHub Actions. Whether you use Terraform as your IaC or get the value manually, which I will show in a bit, using `kube_admin_config_raw` from the AKS module provides a kubeconfig file, that uses the Kubernetes admin account to authenticate, bypasses the need for kubelogin, as the admin account uses a certificate-based authentication method instead of tokens.

This makes it ideal for CI/CD workflows, as it:

- Avoids the need for kubelogin.
- Works out-of-the-box without additional dependencies.
- Provides consistent, reliable authentication for automated workflows.

If you’re not using Terraform, you can retrieve the kube_admin_config_raw manually with:

```bash
az aks get-credentials --resource-group <your-resource-group> --name <your-cluster-name> --admin
```

This command will provide a kubeconfig file with admin credentials that can be stored securely as a GitHub Secret for CI/CD pipelines.

{{<warning>}}
While kube_admin_config_raw simplifies CI/CD workflows, it provides full admin access to the cluster. Ensure this credential is stored securely and consider rotating it periodically to mitigate risks.
{{</warning>}}

## How this looks in GitHub actions using Terraform

1. Extract the `kube_admin_config_raw` value

    Ensure your Terraform module or script outputs the kube_admin_config_raw configuration. For example:

    ```hcl
        output "kube_admin_config_raw" {
        value = azurerm_kubernetes_cluster.example.kube_admin_config_raw
        sensitive = true
        }
    ```

    This provides the raw, admin-level Kubernetes configuration.

1. Store kube_admin_config_raw as a GitHub Actions Secret

    - Copy the output: Run your Terraform script locally and copy the value of the kube_admin_config_raw output.

    - Add a GitHub Secret:
        - Navigate to your GitHub repository.
        - Go to Settings > Secrets and Variables > Actions.
    Add a new secret called KUBE_CONFIG and paste the value of kube_admin_config_raw.

1. A very simple pipeline as an example to illustrate how you'd use the   secret in the pipeline:
    ```yaml
    name: Deploy to AKS

    on:
    push:
        branches:
        - main

    jobs:
    deploy:
        runs-on: ubuntu-latest

        steps:
        - name: Check out code
        uses: actions/checkout@v3

        - name: Set up Kubernetes context
        env:
            KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
        run: |
            echo "${KUBE_CONFIG}" > ~/.kube/config
            kubectl config view

        - name: Deploy to AKS
        run: |
            kubectl apply -f k8s/deployment.yaml
    ```

## Conclusion

In automation scenarios like GitHub Actions, using `kube_admin_config_raw` is advantageous because:

- It eliminates the need for external tools like kubelogin.
- There’s no token expiration issue, as certificate-based authentication remains valid as long as the certificate is.
- It simplifies the pipeline setup, allowing direct access to the cluster without requiring interactive or additional authentication steps.

While you should use kube_config_raw when:

- You need to enforce RBAC restrictions in your cluster.
- Different users or applications require specific permissions based on their roles.
- You want to manage Kubernetes access using Microsoft Entra ID for tighter integration with your organization’s identity management policies.

*Hope this helps someone else as well as it did me when I was troubleshooting and debugging the error!*