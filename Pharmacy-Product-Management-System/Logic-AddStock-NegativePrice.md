# Business Logic Error in Pharmacy Product Management System (Stock Financial Manipulation)

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Product**: [Pharmacy Product Management System](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
* **Version**: 1.0
* **Component**: Add Stock (`add-stock.php`)
* **Vulnerable Parameter**: `txtprice`, `txttotalcost` (POST)

## Description

A critical Business Logic vulnerability exists in the **SourceCodester Pharmacy Product Management System 1.0**. The vulnerability is located in the `add-stock.php` file (Stock Entry module).

The application fails to properly validate that the procurement price (`txtprice`) and total cost (`txttotalcost`) must be positive values. An attacker can intercept the HTTP request and submit negative values (e.g., `-500`).

In a financial context, adding stock is an **expense** or an increase in **asset value**. By injecting negative costs, an attacker can manipulate the system to record these transactions as "credits" or negative expenses. This leads to severe corruption of financial statements, such as artificially lowering the Cost of Goods Sold (COGS), inflating profits, or reducing the calculated value of inventory assets.

## Steps to Reproduce

1. **Setup**: Deploy the Pharmacy Product Management System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the "Add Stock" page.
4. **Exploit**: Intercept the "Save" request using a proxy tool.
5. **Modify**: Change the `txtprice` and `txttotalcost` parameters to negative values.

**PoC Request:**

```http
POST /add-stock.php HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=your_session_id_here

cmddrug=Bosca&txtstockinDate=2026-02-05+10:00:00&txtstock=100&txtcategory=Tablet&txtqty=10&txtexpirydate=2028-01-01&txtproductID=23764&btnsave=&txtprice=-500&txttotalcost=-5000

```

<img width="1080" height="354" alt="图片" src="https://github.com/user-attachments/assets/219f2d30-bd98-430a-b82d-426373c4d8e8" />


## Impact

1. **Financial Reporting Fraud**: Manipulation of inventory value and expenses.
2. **Data Integrity**: Corrupts the database records regarding procurement costs.
3. **Business Logic Bypass**: Can be used to balance out illicit inventory removal by manipulating cost basis.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
