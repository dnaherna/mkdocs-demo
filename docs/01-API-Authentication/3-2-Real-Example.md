---
title: Real Example 2
---

## AWS IAM

API authentication is a process that checks whether users are allowed to access data or features of an application. It helps protect sensitive information and functions by making sure only authorized users can use the API. If the authentication process is weak or done improperly, malicious users can exploit this vulnerability to gain unauthorized access.

To fully understand the result of weak or improper authentication, we will cover a real example found in AWS Identity and Access Management (IAM). This is a service designed to securely manage access to AWS resources. In this real example, we will cover a username enumeration vulnerability to learn how improper security mechanisms can create opportunities for attackers to exploit.

## Login Flow

The AWS IAM login flow is designed to authenticate users attempting to access AWS resources. The process starts when a user submits their username and password through the AWS IAM login portal. The system then validates the credentials against the stored records so they can login.

![image](https://rhinosecuritylabs.com/wp-content/uploads/2025/02/6-1-750x350.png)

The login flow is designed to verify both the username and password before granting access. However, the system's behavior during this process, such as response times or error messages, can help an attacker infer whether a username is valid or invalid because of subtle timing differences.

## Username Enumeration

In this real example, AWS gives users the option to include multi-factor authentication (MFA). They understood that by including multiple login methods, users can make it harder for attackers to steal their account.

![image](https://rhinosecuritylabs.com/wp-content/uploads/2025/02/7.png)

However, because this is an optional feature, not all users will use MFA. For those that don't, the login flow will only check the username and password to verify a user. Generally, the username is checked before continuing with the password. This means if the username exists, there will be an additionally check on the password. This is a subtle but measurable timing difference.

## Proof of Concept

To check if this was true, researchers at Rhino Security Labs created a test account with the username `bfme-console`. They made sure MFA was disabled and recorded the response times compared to other users.

Through this experiment, they recorded about a 100ms increase in the response time for users without MFA. This timing difference is too small for people to notice, but a script can measure these differences accurately.

This means attackers can create a list of possible usernames and determine whether it is valid based on these timing differences. In addition, the attackers will also know that these accounts will not have MFA enabled.

## Mitigations

To prevent attacks that exploit username enumeration vulnerabilities, we should implement additional measures that address this attack vector. These measures help maintain the confidentiality of user accounts.

**Uniform Response Times:** Ensure that the server responds with the same delay for valid and invalid usernames. By making response times consistent regardless of whether a username exists or not, attackers cannot infer whether a username is valid based on timing differences.

**Generic Error Messages:** Replace specific error messages such as "Invalid username" with more generic responses like "Authentication failed." This prevents attackers from identifying valid usernames from invalid ones.

The real example found in AWS demonstrates how timing attacks can exploit subtle flaws in the authentication process. By including other security measures, we can design a strong and secure authentication process to protect the information and functions of our API.
