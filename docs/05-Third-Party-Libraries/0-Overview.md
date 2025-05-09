---
title: Overview
---

## Topics

This module explores the use of third-party libraries in software development and examines their benefits as well as the risks they can introduce. Key concepts include:

- **What Are Third-Party Libraries:** Explains how developers use pre-built components to streamline development but highlights vulnerabilities from outdated or abandoned dependencies.

- **Risks and Vulnerabilities:** Reviews critical security risks, including compromised libraries, unpatched vulnerabilities, and transitive dependency issues.

- **Strategies for Mitigation:** Emphasizes best practices like regular updates, vetting sources, and using secure dependency management techniques.

## Real Examples

Two case studies illustrate the consequences of relying on third-party libraries:

- **Pdoc and Polyfill.io:** Demonstrates how the compromise of `Polyfill.io` led to widespread malware injection across applications, including Pdoc’s documentation output.

- **Coinbase and TJ-Actions:** Explores how attackers attempted to compromise Coinbase by exploiting a vulnerable GitHub Action (`tj-actions/changed-files`) to expose CI/CD credentials in public logs.

These examples highlight the importance of monitoring third-party dependencies and maintaining secure development practices.

## Practice

The practice section offers interactive exercises designed to simulate real-world scenarios:

- **Redirect Clients:** Exploit third-party scripts to redirect users to unintended websites.

- **Steal Cookies:** Learn how attackers target user cookies using malicious scripts and simulate a server-side attack.

- **Keylogging:** Examine how third-party scripts can capture sensitive data, like user key presses, and send it to malicious servers.

By completing these exercises, participants gain a deeper understanding of the risks associated with third-party libraries and learn techniques to safeguard applications effectively.
