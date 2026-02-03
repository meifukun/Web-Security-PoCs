# Business Logic Flaw (Negative Penalty Rate) in Loan Management System

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Vendor**: SourceCodester
* **Product**: [Loan Management System](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
* **Version**: 1.0
* **Component**: Plan Management (`ajax.php?action=save_plan`)
* **Vulnerable Parameter**: `penalty_rate`

## Description

A Business Logic vulnerability exists in **Loan Management System v1.0** due to improper server-side validation.

The application allows administrators to create "Loan Plans" with specific penalty rates for overdue payments. While the frontend interface prevents users from entering negative numbers in the "Monthly Overdue Penalty" field, this constraint is not enforced on the backend. An authenticated attacker can bypass the client-side restriction by manipulating the HTTP POST request to submit a negative value for the `penalty_rate`.

This results in the creation of loan plans with negative penalty rates, which fundamentally breaks the application's financial logic regarding overdue loan handling.

## Steps to Reproduce

1. **Setup**: Deploy the Loan Management System locally.
2. **Login**: Log in to the application as an administrator.
3. **Navigate**: Go to the **Plan** page (`index.php?page=plan`).
4. **Observe Frontend Restriction**: Attempt to type a negative number (e.g., `-5`) in the "Monthly Overdue Penalty" field. The browser or UI will block the action, displaying a warning like "Please select a value not less than 0".
5. **Bypass & Exploit**: Send a direct HTTP POST request (using `curl` or Burp Suite) to the `ajax.php?action=save_plan` endpoint with a negative value for `penalty_rate`.
6. **Verify**: Refresh the Plans page to see the successfully created plan with a negative penalty rate.

## PoC Command

The following `curl` command demonstrates how to bypass the frontend validation and inject a negative penalty rate (`-10%`):

```bash
curl --compressed -s -i -X POST 'http://127.0.0.1:8082/ajax.php?action=save_plan' \
-F "id=" \
-F "months=12" \
-F "interest_percentage=10" \
-F "penalty_rate=-10" \
-w "\nHTTP_STATUS:%{http_code}" \
-H "Cookie: PHPSESSID=YOUR_COOKIE_HERE"

```

## Proof of Concept

### 1. Frontend Validation (Bypassable)

The following screenshot demonstrates that the web interface correctly blocks negative input for the penalty field.

<img width="1080" height="467" alt="图片" src="https://github.com/user-attachments/assets/7334c81e-38ed-4ccb-b492-ebb4f3ecac20" />


### 2. Successful Exploitation

After sending the malicious request, the backend accepts the data. The screenshot below shows the new plan listed in the system with a **-10%** penalty rate, confirming the vulnerability.

<img width="594" height="261" alt="图片" src="https://github.com/user-attachments/assets/fb049287-1486-4f11-87c2-535c970374b5" />

<img width="1082" height="681" alt="图片" src="https://github.com/user-attachments/assets/cf018490-3866-47d4-a745-b2ab69a28f9f" />


## Impact

This vulnerability allows an attacker to corrupt the integrity of the financial data within the system.

1. **Business Logic Disruption**: Penalty calculations for overdue loans will yield incorrect results (e.g., negative penalty amounts).
2. **Financial Loss**: If a borrower is late on payments, the system calculation may deduct money from their total payable amount (reducing their debt) instead of charging a penalty fee, leading to financial discrepancies.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
