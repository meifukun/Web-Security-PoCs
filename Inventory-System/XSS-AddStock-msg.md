# Reflected XSS in Inventory System (Add Stock)

## Overview

* **Vulnerability Type**: Reflected Cross-Site Scripting (XSS)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Add Stock (`add_stock.php`)
* **Vulnerable Parameter**: `msg` (GET)

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in **SourceCodester Inventory System 1.0**. The vulnerability is located in the `add_stock.php` file.

The application accepts the `msg` parameter via a GET request and reflects it back to the user without proper sanitization. This allows remote attackers to inject malicious JavaScript code. Since this page requires authentication, an attacker can target logged-in administrators to steal session cookies or perform actions on their behalf.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application as an administrator.
3. **Exploit**: Access the following URL in the web browser.

**PoC Payload:**

```
http://127.0.0.1:8089/add_stock.php?msg=%3Cimg%20src=%22x%22%20onerror=%22alert(808774)%22%3E

```

## Proof of Concept

When the logged-in user visits the malicious link, the payload `<img src="x" onerror="alert(808774)">` is rendered, causing an alert box with the number `808774` to appear.

<img width="870" height="682" alt="图片" src="https://github.com/user-attachments/assets/40e5f305-89c8-4074-a990-3c6f700725dd" />


## Impact

* **Session Hijacking**: Attackers can steal the administrator's session cookies.
* **Privilege Escalation**: If the victim is an admin, the attacker can manipulate the system.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
