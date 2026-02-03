# Stored XSS in Online Food Ordering System

## Overview

* **Vulnerability Type**: Stored Cross-Site Scripting (XSS)
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Category Maintenance (`/admin/?page=maintenance`)
* **Vulnerable Parameter**: Category Name (POST)

## Description

A Stored Cross-Site Scripting (XSS) vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the Category management module within the admin panel.

The application fails to properly sanitize user input supplied to the "Category Name" field when creating or updating a category. The malicious payload is stored permanently in the SQLite database. Whenever an administrator or user visits the Category list page (or any page where this category is rendered), the injected JavaScript executes immediately in their browser.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System.
2. **Login**: Log in to the application as an administrator (Default: `admin` / `admin`).
3. **Navigate**: Go to the **Maintenance** -> **Category List** page (`/admin/?page=maintenance`).
4. **Inject**: Click "Create New", and in the "Name" field, enter the XSS payload.
5. **Trigger**: Click "Save". The browser will immediately pop up the alert box.
6. **Persistence**: Refresh the page or visit the Category list again; the alert will trigger again, confirming the payload is stored.

## PoC Payload

**Payload:**

```html
<img src="x" onerror="alert(778899)">

```

## Proof of Concept

The following screenshot shows the payload being entered into the Category Name field and the subsequent alert execution upon saving.

<img width="864" height="686" alt="bfbf88ae547f36b6fc85d9ab3466eac7" src="https://github.com/user-attachments/assets/01d31892-e06b-46b6-816c-5ccafc79a6a0" />
<img width="870" height="807" alt="8c91787964ffbd997d1ba6e21856294b" src="https://github.com/user-attachments/assets/be0e303b-742d-4d49-af5b-f0b8254cf246" />


## Impact

This vulnerability allows an attacker to:

1. **Persistent Attack**: Execute malicious scripts on every visit to the affected page without user interaction (beyond visiting the page).
2. **Session Hijacking**: Steal administrator cookies to take over the application.
3. **Defacement**: Modify the visual appearance of the application permanently.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)

---
