# Business Logic Error in Pharmacy Product Management System (Negative Price)

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Product**: [Pharmacy Product Management System](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
* **Version**: 1.0
* **Component**: Add Sales (`add-sales.php`)
* **Vulnerable Parameter**: `txtprice` / `txttotalcost` (POST)

## Description

A critical Business Logic vulnerability exists in the **SourceCodester Pharmacy Product Management System 1.0**. The vulnerability is located in the `add-sales.php` file.

The application fails to properly validate that the unit price (`txtprice`) and total cost (`txttotalcost`) of a product must be positive values. An attacker can intercept the HTTP request and modify these parameters to negative numbers (e.g., `-500`).

When processed, the system records the transaction with a negative value. If this system were integrated with a billing or wallet balance system, this would effectively credit the attacker's account (refunding money instead of charging it). Even without a wallet system, this corrupts financial reports, reducing the total revenue calculations and causing direct financial data integrity issues.

## Steps to Reproduce

1. **Setup**: Deploy the Pharmacy Product Management System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the "Add Sales" page and select a product.
4. **Exploit**: Intercept the "Save" request using a proxy tool (like Burp Suite).
5. **Modify**: Change the `txtprice` to a negative value (e.g., `-500`) and the `txttotalcost` to a corresponding negative total (e.g., `-5000`).

**PoC Request:**

```http
POST /add-sales.php HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=your_session_id_here

cmdcustomer=Nneka+Chuks&txtsalesDate=2026-02-05&txtproductID=23423&cmddrug=Panadol&txtstock=100&txttotalcost=-5000&txtcategory=Tablet&txtqty=10&txtexpirydate=2028-08-28&btnsave=&txtprice=-500

```

<img width="1104" height="342" alt="图片" src="https://github.com/user-attachments/assets/971bf839-492d-491b-8a3a-47d19176042a" />


## Impact

1. **Financial Loss**: Allows transactions where the "cost" is negative, effectively making the store "pay" the customer.
2. **Data Integrity**: Corrupts sales reports and revenue calculations.
3. **Logic Bypass**: Can be used to manipulate daily sales totals.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
