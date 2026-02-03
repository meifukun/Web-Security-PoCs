# Business Logic Flaw (Negative Loan Duration) in Loan Management System

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Vendor**: SourceCodester
* **Product**: [Loan Management System](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
* **Version**: 1.0
* **Component**: Plan Management (`ajax.php?action=save_plan`)
* **Vulnerable Parameter**: `months`

## Description

A Business Logic vulnerability exists in **Loan Management System v1.0** due to the lack of proper input validation.

The application allows administrators to define "Loan Plans" which determine the duration of a loan (in months). However, the backend fails to validate that the duration must be a positive integer. An attacker can submit a negative value for the `months` parameter. The system accepts this invalid data and creates a loan plan with a negative duration (e.g., "-2 months").

This flaw compromises the business logic, as a loan cannot have a negative lifespan. This can lead to errors in loan schedule generation, due date calculations, and potential database inconsistencies.

## PoC Command

```bash
curl --compressed -s -i -X POST 'http://127.0.0.1:8082/ajax.php?action=save_plan' \
-F "id=" \
-F "months=-2" \
-F "interest_percentage=2" \
-F "penalty_rate=2" \
-w "\nHTTP_STATUS:%{http_code}" \
-H "Cookie: PHPSESSID=YOUR_COOKIE_HERE"

```

## Proof of Concept

The screenshot below shows that a Loan Plan was successfully created with a duration of **-2 months**, confirming that the server accepts negative time values.

<img width="730" height="779" alt="图片" src="https://github.com/user-attachments/assets/ba0de4ff-3866-4c14-9f5a-03688aed5da7" />


## Impact

This vulnerability allows the creation of logically invalid loan plans.

1. **Logic Corruption**: Calculations dependent on time duration (start date vs. end date) will be fundamentally broken.
2. **System Instability**: If a loan is created using this plan, the schedule generation algorithm may fail, throw exceptions, or enter infinite loops when trying to calculate payment dates based on a negative timeframe.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
