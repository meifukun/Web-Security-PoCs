# Stored XSS in Inventory System (Update Store Details)

## Overview

* **Vulnerability Type**: Stored Cross-Site Scripting (XSS)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Store Details (`update_details.php`)
* **Vulnerable Parameter**: `website` (POST)

## Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in **SourceCodester Inventory System 1.0**. The vulnerability is located in the `update_details.php` file (Store Settings).

The application fails to properly sanitize the `website` parameter when updating store details via a POST request. This allows an authenticated attacker to inject arbitrary JavaScript code. The malicious script is stored in the database and is executed whenever the "Store Details" page is viewed by an administrator or user, leading to persistent XSS attacks.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application as an administrator.
3. **Navigate**: Go to the "Store Setting" or "Update Details" page (`update_details.php`).
4. **Exploit**: In the "WEBSITE" input field, enter the following XSS payload and click **SUBMIT**.

**PoC Payload:**

```html
"><script>alert(123654);</script>

```

## Proof of Concept

As shown in the attached screenshot, upon submitting the form, the application processes the input and renders the page. The injected JavaScript executes immediately, displaying an alert box with the number `123654`.

<img width="1235" height="662" alt="图片" src="https://github.com/user-attachments/assets/02f7317d-cb63-4a73-8f18-e2bb8e65841f" />


## Impact

* **Persistent Attack**: The malicious script is stored. Anyone viewing the Store Details page will be attacked.
* **Session Hijacking**: Attackers can steal session cookies of administrators who view the settings.
* **Defacement**: The attacker can modify the appearance of the settings page.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
