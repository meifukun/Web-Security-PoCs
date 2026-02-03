# Reflected XSS in Zoo Management System

## Overview

* **Vulnerability Type**: Reflected Cross-Site Scripting (XSS)
* **Vendor**: SourceCodester
* **Product**: [Zoo Management System](https://www.sourcecodester.com/php/15347/zoo-management-system-source-code-php-mysql-database.html)
* **Version**: 1.0
* **Component**: Login Page (`/public_html/login`)
* **Vulnerable Parameter**: `msg` (GET)

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in **Zoo Management System v1.0**. The vulnerability is located in the login page, specifically within the `msg` parameter.

The application reflects the content of the `msg` parameter back to the user without proper HTML encoding or sanitization. This allows remote attackers to inject arbitrary web script or HTML via a crafted URL.

## Steps to Reproduce

1. **Setup**: Deploy the Zoo Management System.
2. **Attack**: Navigate to the login page with a malicious `msg` parameter containing JavaScript/HTML payload.
3. **Trigger**: The browser renders the payload immediately, executing the script.

## PoC Payload

**Payload:**

```html
<img src="x" onerror="alert(119843)">

```

**Full Exploit URL:**

```text
http://127.0.0.1:8084/public_html/login?msg=%3Cimg%20src=%22x%22%20onerror=%22alert(119843)%22%3E

```

## Proof of Concept

The following screenshot demonstrates the XSS vulnerability, showing the alert box `119843` triggered by the injected payload in the `msg` parameter.

<img width="1064" height="794" alt="图片" src="https://github.com/user-attachments/assets/0f4f38dc-d0fd-49fa-8634-d39f74814bbb" />


## Impact

This vulnerability allows an attacker to:

1. **Session Hijacking**: Steal user session cookies.
2. **Phishing**: Display fake login forms to steal credentials.
3. **Redirection**: Redirect users to malicious websites.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/15347/zoo-management-system-source-code-php-mysql-database.html)
