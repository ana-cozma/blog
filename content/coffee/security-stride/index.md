---
title: "Decoding Software Security: A Guide to Assessing Requirements with the STRIDE model"
date: 2024-02-16T09:48:41+01:00
draft: false
tags: ["security", "coffee-reads"]
---

For this coffee read, I wanted to write about the STRIDE model and how, when working in your day to day job, you can use it to break down a functionality or requirement you may have and identify potential security threats. 

The blog post is more of a guide in how create a security first mindset to approach your work. It's not a guide on how to implement security measures or how to fix the threats (which I will add as part of tech articles). It's more about how to identify them and categorize them.

## Overview of the STRIDE model

The STRIDE model is a framework used to analyze and categorize different types of security threats that can affect computer systems and software applications. Each letter in ["STRIDE" represents a different category of threat](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats):

> **Spoofing Identity**: _This involves attackers impersonating users, systems, or entities to gain unauthorized access or privileges within a system. Spoofing attacks often exploit vulnerabilities in authentication mechanisms._
>
> **Tampering with Data**: _This refers to unauthorized modifications or alterations to data within a system. Tampering attacks can involve changing, deleting, or inserting data to disrupt the integrity or reliability of the information._
>
>**Repudiation**: _Repudiation threats involve the ability of a user to deny having performed a particular action within a system. This can complicate efforts to trace security incidents and hold individuals accountable for their actions._
>
>**Information Disclosure**: _This category covers threats related to the unauthorized exposure or disclosure of sensitive information. Information disclosure attacks can lead to the exposure of confidential data, compromising the privacy and security of individuals or organizations._
>
>**Denial of Service (DoS)**: _Denial of Service attacks aim to disrupt the availability or functionality of a system or network, making it inaccessible to legitimate users. DoS attacks typically overwhelm resources, such as bandwidth or processing power, rendering the system unusable._
>
>**Elevation of Privilege**: _Elevation of privilege threats involve attackers gaining unauthorized access to higher levels of privilege or permissions within a system. This allows attackers to perform actions or access resources that are typically restricted to privileged users._

## Implementing the STRIDE model

Let's say, as part of your work, you are given a requirement or a functionality that is being developed or is going to be implemented and you have to analyze it from a security perspective. You understand the STRIDE model in general, but how would you actually approach breaking down the statement using the STRIDE model especially if you are new to security or if you don't have yet the actual implementation details?

When breaking down a statement using the STRIDE model I followed a few steps that I found helpful:

### 1. Identify the Threats

Start by reading the statement carefully and identify any potential security threats or vulnerabilities mentioned. If the requirement concerns a feature about to be implemented think a bit about how that feature will look like and ask yourself questions like: _How will the user authenticate? Will it be using a token? How much information will be stored in that token? If someone gets access to that token what can they find about the user? What functionality in your platform will they have access to? Can they manipulate it?_

Look for any actions or scenarios that could lead to unauthorized access, data manipulation, information disclosure, denial of service, or privilege escalation.

At this point you want to write down any security concerns that come to mind. Don't worry about categorizing them yet, just write them down. Doesn't matter if they are small or big, if it's a statement or a question. I always think it's better to have an abundance of information that you can later filter out than to miss something important.

Rule of thumb: If you have a concern or question about it, write it down and analyze it the next steps.

### 2. Categorize the Threats

Once you've identified potential threats, try your best to categorize them according to the STRIDE model. Determine which letters in STRIDE are relevant to each threat you've identified.

A single threat can be part of multiple categories. For example, a vulnerability in an authentication mechanism could be categorized as a "Spoofing Identity" threat, while also posing a risk of "Elevation of Privilege" if exploited. Depending on how you decide to work with the STRIDE model, you can add it to the main category or add it to multiple categories. Whatever makes sense to you and your team should be your guide. The important thing is that everyone has the same understanding on the type of threat and the level of priority that it needs to be handled with, which we will deal with in the next step.

### 3. Analyze the Impacts

After you have sifted through all of them, the next thing is to consider the potential impacts or consequences of each threat on the system or application described in the statement or your functionality. Think about how each threat could affect the confidentiality, integrity, availability, and accountability of the system or its users.

Based on this, you can determine which threats pose the greatest risk to the security of the system and start tackling them first. This will help you focus on the most critical threats while also not disrupting your current workload.

### 4. Propose Mitigations

Lastly, propose potential mitigations or countermeasures to address the identified threats and reduce their likelihood or impact. This could involve implementing security controls, patches, or best practices to mitigate vulnerabilities and protect against security threats.

What I'd like to highlight here is, when proposing mitigations, it's important to consider the trade-offs. While you focus on security it's a good idea to also keep in mind how the performance or usability of the system will be affected.

When in doubts, it's always a good idea to read up on the best practices of the technology you are using or the type of threat you are dealing with. It's always better to have a good understanding of the threat and the technology you are using before you start implementing any mitigations. Especially if you are just starting out with security.

## Example

Having gone over the thought process of breaking down a statement using the STRIDE model, let's go through an example to see how it can be applied in practice.

Let's say you have your basic application that allows users to sign in or sign up. With this you also have a Reset Password functionality that allows users to reset their password in case they forget it. Pretty common scenario for most of the platforms we use day to day.

And let's assume we run a test on that endpoint and we find out that it's vulnerable to Cross-Site Request Forgery (CSRF) attacks. Specifically, if the CSRF token is removed from the request, the request is still processed successfully.

### Identify the Threats

First thing, let's think of the potential threats that could arise from this vulnerability. If the attacker is successful in exploiting this vulnerability, they could potentially gain:

- Unauthorized access to user accounts
- Unauthorized modification of user passwords
- Information disclosure of user credentials
- Potential for account takeover
- Denial of service due to unauthorized password changes
etc.

### Categorize the Threats

Then, let's categorize these threats according to the STRIDE model:

_Spoofing identity:_

The attacker could create a CSRF payload, which they host on a malicious website. If users unwittingly trigger this request, it may prompt a password reset action within their authenticated sessions. Consequently, the attacker gains the ability to reset passwords without requiring the user's current password.

_Tampering with data:_

Through the exploitation of CSRF vulnerabilities or open redirect attacks, the attacker can manipulate the password reset process. What this means is they can designate a password of their preference to reset their own account's password, circumventing the necessity for the current user's password.

_Repudiation:_

The attacker might disown their actions by denying involvement in initiating the password reset requests or by asserting that they had authorization for such actions.

_Information disclosure:_

Successful password reset requests could potentially lead to the exposure of sensitive information like user credentials or personal data if the attacker get access to the user's account.

_Denial of service:_

These attacks have the potential to result in denial of service by disrupting the normal flow of password reset processes. This could prevent real users to reset their passwords or access their accounts.

_Elevation of privilege:_

The attacker can elevate their privileges by resetting passwords without the need for the current user's password. This enables them to obtain unauthorized entry into user accounts, potentially compromising sensitive data or doing malicious activities on behalf of the user.

### Conclusion

To mitigate these threats, you could implement the following apporaches to your work:

First, make sure you have some sort of automated security checks or a security scanner in place. You need to be aware of the vulnerabilities in your system and get fast feedback if new ones are introduced or if the existing ones are indeed fixed (or not).

Then, you can start by implementing a validation of the CSRF token in the password reset request. For example, validating that the CSRF token in the request body is indeed present and that it is valid. Maybe even add a check that the token is tied to the user's authenticated session information. Just to get you started. Then you can think of more ways to streghten the security like not making the endpoint available to everyone maybe.

It's always a good idea to stay up to date on the latest threats whether it is by watching security news and checking if it's applicable to you or checking [OWASP TOP 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) or in our example, have a look over [CSRF prevention cheat sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

And finally, always have a security review before you release a new feature or functionality. It's always better to catch these things before they go live.

_I hope this guide helps you on how to approach breaking down the STRIDE model to your functionality/scenario. If not, I hope it's still a good way to start thinking about security and starting a conversation with your team about it._

__Enjoy your coffee!__ ☕️
