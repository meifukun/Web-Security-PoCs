# Business Logic Error in Pharmacy Product Management System (Overselling)

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Product**: [Pharmacy Product Management System](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
* **Version**: 1.0
* **Component**: Add Sales (`add-sales.php`)
* **Vulnerable Parameter**: `txtqty` (POST)

## Description

A Business Logic vulnerability exists in the **SourceCodester Pharmacy Product Management System 1.0**. The vulnerability is located in the `add-sales.php` file.

The application fails to verify if the requested sales quantity (`txtqty`) exceeds the available stock level. An attacker can manipulate the request to purchase a quantity (e.g., `1000`) that is significantly higher than the actual available stock (e.g., `1`).

This lack of server-side validation results in the system processing the transaction successfully. Depending on the backend implementation, this leads to **negative inventory records** (e.g., stock becoming `-999`) or allows orders to be placed that cannot physically be fulfilled (Denial of Service for legitimate customers).

## Steps to Reproduce

1. **Setup**: Deploy the Pharmacy Product Management System locally.
2. **Identify Stock**: Find a product with low stock (e.g., "Panadol" with 1 unit remaining).
3. **Exploit**: Intercept the "Add Sales" request.
4. **Modify**: Change the `txtqty` to a value larger than the stock (e.g., `1000`).

**PoC Request:**

```http
POST /add-sales.php HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=your_session_id_here

cmdcustomer=Nneka+Chuks&txtsalesDate=2026-02-05&txtproductID=23423&cmddrug=Panadol&txttotalcost=233000&txtcategory=Tablet&txtexpirydate=2028-08-28&txtprice=233&btnsave=&txtstock=1&txtqty=1000

```
<img width="1090" height="428" alt="图片" src="https://github.com/user-attachments/assets/46410c9b-d9db-4e53-bbca-728a18d9bdc0" />



## Impact

1. **Inventory Corruption**: Results in negative stock values in the database.
2. **Unfulfillable Orders**: The system accepts orders that cannot be delivered.
3. **Denial of Service**: An attacker could theoretically "buy" the entire inventory (and more), preventing legitimate users from purchasing items.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
