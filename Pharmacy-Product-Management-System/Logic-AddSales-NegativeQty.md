# Business Logic Error in Pharmacy Product Management System (Add Sales)

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Product**: [Pharmacy Product Management System](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
* **Version**: 1.0
* **Component**: Add Sales (`add-sales.php`)
* **Vulnerable Parameter**: `txtqty` (POST)

## Description

A Business Logic vulnerability exists in the **SourceCodester Pharmacy Product Management System 1.0**. The vulnerability is located in the `add-sales.php` file.

The application fails to validate that the quantity of items sold (`txtqty`) must be a positive integer. An attacker can submit a negative value (e.g., `-10`) via a crafted HTTP POST request.

In the backend logic, the system calculates the remaining stock by subtracting the sold quantity from the current stock (`NewStock = CurrentStock - SoldQty`). When a negative value is subtracted, it results in an addition (`CurrentStock - (-10) = CurrentStock + 10`), causing the stock level to increase rather than decrease. This allows attackers to artificially inflate inventory levels.

## Steps to Reproduce

1. **Setup**: Deploy the Pharmacy Product Management System locally.
2. **Login**: Log in to the application (e.g., as a pharmacist or admin).
3. **Navigate**: Go to the "Add Sales" page.
4. **Exploit**: Intercept the "Save" request using a proxy tool (like Burp Suite) or use `curl`.
5. **Modify**: Change the `txtqty` parameter to a negative number (e.g., `-10`) and send the request.

**PoC Request:**

```http
POST /add-sales.php HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=your_session_id_here

cmdcustomer=Nneka+Chuks&txtsalesDate=2026-02-05&txtproductID=23423&cmddrug=Panadol&txtstock=100&txttotalcost=233&txtcategory=Tablet&txtexpirydate=2028-08-28&txtprice=233&btnsave=&txtqty=-10

```

<img width="1083" height="339" alt="图片" src="https://github.com/user-attachments/assets/f32c19ec-28fd-44cc-b21d-2565c98f42c6" />


## Impact

1. **Inventory Integrity Compromised**: Attackers can artificially inflate stock levels, corrupting inventory records.
2. **Financial Discrepancy**: This disrupts cost calculations and sales reports (e.g., negative total cost).
3. **Potential Logic Bypass**: Could be used to bypass stock availability checks or manipulate refund logic.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html

```
