# Business Logic Flaw (Negative Interest Rate) in Loan Management System

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Vendor**: SourceCodester
* **Product**: [Loan Management System](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
* **Version**: 1.0
* **Component**: Plan Management (`ajax.php?action=save_plan`)
* **Vulnerable Parameter**: `interest_percentage`

## Description

A Business Logic vulnerability exists in **Loan Management System v1.0** due to improper server-side validation.

The application allows administrators to create "Loan Plans" with specific interest rates. While the frontend interface prevents users from entering negative numbers, this constraint is not enforced on the backend. An authenticated attacker can bypass the client-side restriction by manipulating the HTTP POST request to submit a negative value for the `interest_percentage`.

This results in the creation of loan plans with negative interest rates, which fundamentally breaks the application's financial logic.

## Steps to Reproduce

1. **Setup**: Deploy the Loan Management System locally.
2. **Login**: Log in to the application as an administrator.
3. **Navigate**: Go to the **Plan** page (`index.php?page=plan`).
4. **Observe Frontend Restriction**: Attempt to type a negative number (e.g., `-2`) in the "Interest" field. The browser or UI will block the action, displaying a warning like "Please select a value not less than 0".
5. **Bypass & Exploit**: Send a direct HTTP POST request (using `curl` or Burp Suite) to the `ajax.php?action=save_plan` endpoint with a negative value for `interest_percentage`.
6. **Verify**: Refresh the Plans page to see the successfully created plan with a negative interest rate.

## PoC Command

The following `curl` command demonstrates how to bypass the frontend validation and inject a negative interest rate (`-5.5%`):

```bash
curl --compressed -s -i -X POST 'http://127.0.0.1:8082/ajax.php?action=save_plan' \
-F "id=" \
-F "months=12" \
-F "interest_percentage=-5.5" \
-F "penalty_rate=2" \
-w "\nHTTP_STATUS:%{http_code}" \
-H "Cookie: PHPSESSID=YOUR_COOKIE_HERE"

```

## Proof of Concept

### 1. Frontend Validation (Bypassable)

The following screenshot demonstrates that the web interface correctly blocks negative input.

<img width="1082" height="417" alt="图片" src="https://github.com/user-attachments/assets/9f575780-8d52-42cf-9acb-5cecb5ce356f" />


### 2. Successful Exploitation

After sending the malicious request, the backend accepts the data. The screenshot below shows the new plan listed in the system with a **-5.5%** interest rate, confirming the vulnerability.
<img width="591" height="260" alt="图片" src="https://github.com/user-attachments/assets/a3ba5057-2738-4940-965f-cf02f155a031" />
<img width="1102" height="600" alt="图片" src="https://github.com/user-attachments/assets/14049cdc-e77a-462f-8deb-d92cabefdc08" />

## Impact

This vulnerability allows an attacker to corrupt the integrity of the financial data within the system.

1. **Business Logic Disruption**: Loan calculations based on these plans will yield incorrect results (e.g., negative interest amounts).
2. **Financial Loss**: If such a plan is assigned to a real loan, the calculation formulas may deduct money from the total payable amount instead of adding interest, leading to potential financial discrepancies.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14471/loan-management-system-using-phpmysql-source-code.html)
