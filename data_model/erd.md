
# ERD – Operational Database Schema

## Overview
This section describes the operational database structure used as the source system for the BI project.

The database represents the transactional activity of an e-commerce company selling electronics products. It includes customers, products, orders, shipments, shipping companies, ratings, brands, and cities. The source data supports the company’s core process: order placement, shipment preparation, delivery, and customer feedback. 

---

## Main Entities

### CUSTOMERS
Stores customer demographic and registration information.

| Field | Type | Description |
|---|---|---|
| CustomerID | Int | Unique customer identifier |
| FullName | Varchar | Customer full name |
| Email | Varchar | Customer email address |
| BirthDate | Date | Customer birth date |
| Gender | Varchar | Customer gender |
| Address_City | Varchar | Customer city, linked to CITIES |
| Address_street | Varchar | Customer street |
| Address_houseNum | Int | Customer house number |
| RegistrationDT | DateTime | Customer registration date and time |

---

### CITIES
Stores the city-to-region mapping.

| Field | Type | Description |
|---|---|---|
| City | Varchar | City name / identifier |
| Region | Varchar | Region of the city |

---

### BRANDS
Stores product brand information.

| Field | Type | Description |
|---|---|---|
| BrandID | Int | Unique brand identifier |
| BrandName | Varchar | Brand name |
| IsGlobal | Boolean | Whether the brand is global |
| WarrantyMonth | Int | Warranty length in months |

---

### PRODUCTS
Stores product-level information.

| Field | Type | Description |
|---|---|---|
| ProductID | Int | Unique product identifier |
| ProductName | Varchar | Product name |
| Category | Varchar | Product category |
| UnitCost | Money | Cost per unit for the company |
| UnitPrice | Money | Selling price per unit |
| BrandID | Int | Linked brand identifier |

---

### ORDERS
Stores order-level transactional information.

| Field | Type | Description |
|---|---|---|
| OrderID | Int | Unique order identifier |
| CustomerID | Int | Linked customer identifier |
| OrderDT | DateTime | Order date and time |
| EstimatedDeliveryDate | Date | Promised delivery date |
| PaymentMethod | Varchar | Selected payment method |
| ShippingID | Int | Linked shipment identifier |

---

### ORDER_ITEMS
Stores the products included in each order.

| Field | Type | Description |
|---|---|---|
| OrderID | Int | Linked order identifier |
| ProductID | Int | Linked product identifier |
| Quantity | Int | Quantity ordered |
| UnitPriceAtSale | Money | Unit price at time of sale |
| TotalPrice | Money | Total line price |

---

### SHIPPINGS
Stores shipment-level fulfillment data.

| Field | Type | Description |
|---|---|---|
| ShippingID | Varchar | Unique shipment identifier |
| ShippingDT | DateTime | Shipment dispatch date and time |
| ActualDeliveryDate | Date | Actual delivery date |
| ShippingTo | Varchar | Destination city |
| ShipmentCompanyID | Int | Linked shipping company identifier |

---

### SHIPMENT_COMPANIES
Stores information about external shipping providers.

| Field | Type | Description |
|---|---|---|
| Company_ID | Int | Unique shipping company identifier |
| CompanyName | Varchar | Shipping company name |
| Fee | Int | Fixed shipping fee charged to the company per shipment |

---

### RATINGS
Stores product ratings submitted by customers.

| Field | Type | Description |
|---|---|---|
| Rating_id | Int | Unique rating identifier |
| OrderID | Int | Linked order identifier |
| ProductID | Int | Linked product identifier |
| RatingDT | DateTime | Rating date and time |
| Rating | Int | Customer rating value |

---

## Relationships

### Customer Relationships
- One customer can place many orders.
- Each order belongs to one customer.
- Customers are linked to cities through `Address_City`. :contentReference[oaicite:2]{index=2}

### Product Relationships
- One brand can be linked to many products.
- Each product belongs to one brand.

### Order Relationships
- One order can contain many order items.
- Each order item belongs to one order.
- Each order is linked to one shipment.

### Shipment Relationships
- One shipping company can handle many shipments.
- Each shipment is linked to one shipping company.

### Rating Relationships
- Ratings are linked to both orders and products.
- This allows product feedback to be analyzed in the context of actual purchases.

---

## Business Purpose of the Source Model
The operational database supports the main business process of the company:
1. Customer browses products
2. Customer places an order
3. Order is assigned to shipment
4. Shipment is delivered
5. Customer may rate the purchased product

This transactional structure is the source for the BI layer and the Data Warehouse built later in the project. 
