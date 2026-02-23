# Reflected XSS in Loan Management System (Index Page)

## Overview

* **Vulnerability Type**: Reflected Cross-Site Scripting (XSS)
* **Product**: Loan Management System (SourceCodester)
* **Version**: 1.0
* **Component**: Index Page (`index.php`)
* **Vulnerable Parameter**: `page` (GET)

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in the **SourceCodester Loan Management System 1.0**. The vulnerability is located in the `index.php` file.

The application accepts the `page` parameter via a GET request (used for dynamic page routing) and reflects it into the HTML document without proper sanitization. By crafting a URL with a malicious payload designed to break out of the existing HTML/Script context (e.g., `</script><script>...`), a remote attacker can execute arbitrary JavaScript code in the browser of the victim who visits the link.

## Steps to Reproduce

1. **Setup**: Deploy the Loan Management System locally.
2. **Exploit**: Access the following crafted URL in a web browser.

**PoC Payload:**

```
/index.php?page=%3C/script%3E%3Cscript%3Ealert(207252)%3C/script%3E

```

## Proof of Concept

When the victim accesses the URL, the payload `</script><script>alert(207252)</script>` is successfully rendered and executed, triggering an alert box with the number `207252`.

<img width="1185" height="154" alt="6f9dedec01c6b60a1073d2215dc20215" src="https://github.com/user-attachments/assets/ad928ac4-89cc-443c-9f2d-cace7961c804" />


## Impact

* **Session Hijacking**: Attackers can steal session cookies of authenticated users or administrators.
* **Phishing**: Attackers can modify the page DOM to trick users into entering credentials.
* **Malicious Actions**: Execute arbitrary actions on behalf of the victim within the application.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
