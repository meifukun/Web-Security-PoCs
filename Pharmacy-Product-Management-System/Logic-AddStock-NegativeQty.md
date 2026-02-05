# Business Logic Error in Pharmacy Product Management System (Add Stock)

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Product**: [Pharmacy Product Management System](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
* **Version**: 1.0
* **Component**: Add Stock (`add-stock.php`)
* **Vulnerable Parameter**: `txtqty` (POST)

## Description

A Business Logic vulnerability exists in the **SourceCodester Pharmacy Product Management System 1.0**. The vulnerability is located in the `add-stock.php` file (Stock Entry).

The application fails to properly validate the quantity of items being added to the inventory (`txtqty`). It expects a positive integer to increase the stock level (`CurrentStock + InputQty`). However, an attacker can submit a negative value (e.g., `-100`).

Due to simple arithmetic logic in the backend, adding a negative number results in subtraction (`CurrentStock + (-100)`). This allows an attacker to silently reduce or destroy inventory records without authorized access to a "delete" function, leading to inventory corruption or Denial of Service (DoS) by zeroing out stock.

## Steps to Reproduce

1. **Setup**: Deploy the Pharmacy Product Management System locally.
2. **Login**: Log in to the application.
3. **Navigate**: Go to the "Add Stock" or "Stock In" page.
4. **Exploit**: Intercept the "Save" request using a proxy tool.
5. **Modify**: Change the `txtqty` parameter to a negative number (e.g., `-100`).

**PoC Request:**

```http
POST /add-stock.php HTTP/1.1
Host: 127.0.0.1:8090
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=your_session_id_here

cmddrug=Bosca&txtstockinDate=2026-02-05+10:00:00&txtstock=100&txtcategory=Tablet&txtexpirydate=2028-01-01&txtprice=50&txtproductID=23764&txttotalcost=500&btnsave=&txtqty=-100

```
<img width="1102" height="335" alt="图片" src="https://github.com/user-attachments/assets/0644bedd-e59a-4f5c-9dd7-a6c42a2bdac8" />



## Impact

1. **Inventory Destruction**: Attackers can maliciously reduce stock levels.
2. **Denial of Service**: By reducing stock to 0 (or negative), legitimate sales may be blocked if the system checks for availability.
3. **Data Integrity**: Corrupts accurate stock-in records.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/17883/web-based-product-alert-system.html)
