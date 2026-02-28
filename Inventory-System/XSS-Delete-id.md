# Reflected XSS in Sales and Inventory System (Delete Action)

## Overview

* **Vulnerability Type**: Reflected Cross-Site Scripting (XSS)
* **Product**: Sales and Inventory System (SourceCodester)
* **Version**: 1.0
* **Component**: Delete Action (`delete.php`)
* **Vulnerable Parameter**: `id` (GET)

## Description

A Reflected Cross-Site Scripting (XSS) vulnerability exists in the **SourceCodester Sales and Inventory System 1.0**. The vulnerability is located in the `delete.php` file.

The application accepts the `id` parameter via a GET request (used to specify which record to delete) and reflects it into the response without proper sanitization or encoding. By crafting a URL with a malicious payload, a remote attacker can execute arbitrary JavaScript code in the browser of the victim. Because this action typically requires administrative privileges to perform a deletion, an attacker could target a logged-in administrator to hijack their session.

## Steps to Reproduce

1. **Setup**: Deploy the Sales and Inventory System locally.
2. **Login**: Log in to the application as an administrator.
3. **Exploit**: Access the following crafted URL in a web browser.

**PoC Payload:**

```text
http://127.0.0.1:8089/delete.php?id=%3CscrIpt%3Ealert%281%29%3B%3C%2FscRipt%3E&table=customer_details&return=view_customers.php

```

## Proof of Concept

When the authenticated victim accesses the URL, the payload `<scrIpt>alert(1);</scRipt>` is successfully rendered and executed by the browser, triggering an alert box with the number `1`.
<img width="1363" height="814" alt="图片" src="https://github.com/user-attachments/assets/9fba5fe0-8e88-41c5-866d-cde2e59dacd4" />


## Impact

* **Session Hijacking**: Attackers can steal the session cookies of administrators.
* **Privilege Escalation**: Execute actions on behalf of the administrator, such as adding rogue accounts or modifying system settings.
* **Malicious Actions**: Force the administrator's browser to perform unintended deletions or state-changing requests.
