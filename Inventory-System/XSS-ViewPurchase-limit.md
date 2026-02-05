# Reflected XSS in Inventory System (View Purchase)

## Overview

* **Vulnerability Type**: Reflected Cross-Site Scripting (XSS)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: View Purchase (`view_purchase.php`)
* **Vulnerable Parameter**: `limit` (GET)

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in **SourceCodester Inventory System 1.0**. The vulnerability is located in the `view_purchase.php` file.

The application accepts the `limit` parameter via a GET request (used for pagination control) and reflects it back to the user without proper sanitization. This allows remote attackers to inject malicious JavaScript code. Since this page requires authentication, an attacker can target logged-in administrators to steal session cookies or perform actions on their behalf.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application as an administrator.
3. **Exploit**: Access the following URL in the web browser.

**PoC Payload:**

```
http://127.0.0.1:8089/view_purchase.php?limit=%22%3E%3Cscript%3Ealert(15623);%3C/script%3E

```

## Proof of Concept

When the logged-in user visits the malicious link, the payload `"><script>alert(15623);</script>` is rendered, causing an alert box with the number `15623` to appear.

<img width="1154" height="806" alt="图片" src="https://github.com/user-attachments/assets/a7f48be2-d2ed-461c-9f64-d2f5c81a3f21" />


## Impact

* **Session Hijacking**: Attackers can steal the administrator's session cookies.
* **Privilege Escalation**: If the victim is an admin, the attacker can manipulate the system.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
