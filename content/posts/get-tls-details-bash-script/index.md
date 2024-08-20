---
title: "How to Check TLS Configuration of URLs with Curl and Bash Script"
description: "How to Check TLS Configuration of URLs with Curl and Bash Script"
date: 2024-08-19T15:24:34+02:00
draft: true
tags: ["azure", "security"]
---
If you are working in an Azure environment and you are using Azure Availability Tests you might run into the following Health Advisory event:

> On 31 October 2024, in alignment with the Azure wide legacy TLS deprecation, TLS 1.0/1.1 protocol versions and the below listed TLS 1.2/1.3 legacy Cipher suites and > Elliptical curves will be retired for Application Insights availability tests.

For a list of deprecated versions and remaining supported versions have a look over the official documentation [here](https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability?tabs=standard#deprecating-tls-configuration).

But how do you quickly check which endpoint in your availability tests is impacted?

This was the initial scenario that led me to create and ultimately write this blog post, but these checks can be applied to any case where you need to retrieve and verify the TLS configuration of URLs. Whether you're ensuring compliance with security standards, troubleshooting connection issues, or simply gathering information for audits, the script can help you get the information you need.

## Using Curl for one URL

You can use the curl command with the -v (verbose) option to see detailed information about the TLS handshake, including the TLS version by running the command below:

```bash
curl -s -v https://example.com 2>&1 | grep "SSL connection using"
```

Explanation of the command and its parameters:

- Use `curl` to make a request to the specified URL.
- The `-s` option makes curl silent, except for errors.
- The `-v` option outputs verbose information.
- The `2>&1` redirects the standard error (where verbose output is written) to standard output, allowing grep to filter it.
- The `grep "SSL connection using"` command filters out the line containing the TLS version.

By running the command the output will be something like this:

```bash
* SSL connection using TLSv1.2 / ECDHE_RSA_AES_256_GCM_SHA384
```

But what if you have more than one URL you need to check? Running this command manually can be tiring and it will involve a lot of copy-pasting which can be time-consuming. So let's look into saving up some time.

## Using Curl and a Bash script to loop through a list of URLs

We can take this a step further and create a script that will accept a list of URLs, loop through them and output the information we need. And you can achieve this by following the steps below:

**1. Create the file:**

First, let's create a file `check_tls_version.sh` or you can name it however you want:

```bash
touch check_tls_version.sh
```

**2. Make the Script Executable:**

```bash
chmod +x check_tls_details.sh
```

**3. Create the Script:**

```bash
vi check_tls_version.sh
```

You can use vi, nano or just open in any Editor of your choice and paste inside the newly created file the code below:

```bash
#!/bin/bash

# Check if a file was provided as an argument
if [ -z "$1" ]; then
  echo "Usage: $0 <file_with_urls>"
  exit 1
fi

# Read the file line by line
while IFS= read -r url; do
  # Make sure the line is not empty
  if [ -n "$url" ]; then
    echo "Checking $url..."

    # Use curl to fetch the TLS details
    output=$(curl -s -v "$url" 2>&1)

    # Extract the TLS version
    tls_version=$(echo "$output" | grep "SSL connection using" | awk '{print $5}')

    # Extract the cipher suite
    cipher_suite=$(echo "$output" | grep "SSL connection using" | awk -F'/' '{print $2}' | xargs)

    # Extract the elliptic curve (if available)
    elliptic_curve=$(echo "$output" | grep "SSL certificate verify" | grep -o '(?<=using ).*(?= curve)')

    # Output the results
    echo "URL: $url"
    echo "TLS Version: ${tls_version:-Could not retrieve TLS version}"
    echo "Cipher Suite: ${cipher_suite:-Could not retrieve cipher suite}"
    if [ -n "$elliptic_curve" ]; then
      echo "Elliptic Curve: $elliptic_curve"
    else
      echo "Elliptic Curve: Could not retrieve elliptic curve or not applicable"
    fi
    echo ""
  fi
done < "$1"
```

In the sample above, I added comments on what each line does, but feel free to modify it to extract more or less of the information you are interested in.

**4. Prepare a File with URLs:**

Create a text file with one URL per line, e.g., `urls.txt`:

```bash
https://example.com
https://anotherexample.com
```

**5. Run the Script:**

Having both the script (`heck_tls_details.sh`) and our list of URLs we need to check(`urls.txt`), we can now run the script we created:

```bash
./check_tls_details.sh urls.txt
```

**6. Sample Output:**

The script will have an output similar to this one for each URL you provided:

```yaml
Checking https://example.com...
URL: https://example.com
TLS Version: TLSv1.3
Cipher Suite: AEAD-AES128-GCM-SHA256
Elliptic Curve: X25519

Checking https://anotherexample.com...
URL: https://anotherexample.com
TLS Version: TLSv1.2
Cipher Suite: ECDHE-RSA-AES256-GCM-SHA384
Elliptic Curve: prime256v1
```

Explanation of the output parameters:

*URL:* The URLs being checked - the ones you provide in the urls.txt file.\
*TLS Version:* The TLS version used by the URL.\
*Cipher Suite:* The cipher suite used for the connection.\
*Elliptic Curve:* The elliptic curve used, if applicable.

Now that you have your data you can simply compare the TLS version, Cipher Suite or Elliptic Curve against the deprecated or supported versions and take appropriate actions to update them.

*Thank you for reading and I hope this helps someone out there with your use case!*
