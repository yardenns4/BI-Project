# Data Warehouse – Star Schema

## Overview
This section describes the Data Warehouse designed for the BI solution.

The warehouse was designed to support operational, tactical, and strategic decision-making through multiple fact tables at different levels of granularity. The final and relevant version is the updated model described in Part B, where ratings were merged into FACT_ORDERITEMS, shipments were modeled as a fact table, and the schema was refined to improve analytical consistency. :contentReference[oaicite:5]{index=5}

---

## Design Logic
The warehouse uses:
- Multiple fact tables with different grains
- Shared dimensions for consistent slicing and filtering
- Slowly Changing Dimensions (Type 2) where historical tracking is important
- Date and time dimensions for time-based analysis

The chosen structure enables flexible analysis from shipment level, to order level, to product-in-order level. 

---

## Fact Tables

### FACT_ORDERITEMS
**Grain:** one product line within one order

This is the most detailed fact table and supports product, profitability, discount, and rating analysis.

| Field | Type | Description |
|---|---|---|
| DW_OrderItemKey | Int | Surrogate key for order item |
| OrderID | Int | Original order identifier |
| DW_OrderDate | Date | Linked date dimension |
| DW_CustomerKey | Int | Linked customer dimension |
| DW_ProductKey | Varchar(50) | Linked product dimension |
| ShippingID | Varchar | Shipment identifier |
| DW_ShipmentCompanyID | Int | Linked shipment company dimension |
| Quantity | Int | Quantity of the product in the order |
| UnitPriceAtSale | Money | Unit sale price |
| TotalPrice | Money | Total price for this order line |
| TotalOriginalPrice | Money | Original total price before discount |
| DiscountRate | Decimal | Discount rate for the line |
| UnitDiscount | Money | Discount amount per unit |
| ProductValueContributionToOrder | Ratio | Share of the order value contributed by this line |
| ProductValueContributionToShipment | Ratio | Share of the shipment value contributed by this line |
| ShippingCostShareLine | Money | Allocated shipping cost for the line |
| ProfitFromProduct | Money | Operational profit from the order line |
| ProfitFromProductUnit | Money | Profit per unit |
| Rating | Int | Product rating associated with the line |

---

### FACT_ORDERS
**Grain:** one row per order

Supports order-level analysis, customer purchase behavior, discounts, order profitability, and SLA tracking.

| Field | Type | Description |
|---|---|---|
| DW_OrderKey | Int | Surrogate key for order |
| OrderID | Int | Original order identifier |
| DW_OrderDate | Date | Linked date dimension |
| DW_OrderTime | Time | Linked time dimension |
| DW_CustomerKey | Int | Linked customer dimension |
| ShippingID | Varchar | Shipment identifier |
| DW_ShippingCompanyID | Int | Linked shipment company dimension |
| EstimatedArrivalDate | Date | Expected arrival date |
| PaymentMethod | Varchar | Payment method |
| SLA_STATUS | Binary / Text | OnTime or Late |
| SLA_days | Int | Delay in days |
| OrderTotalPrice | Money | Final order value |
| OrderTotalOriginalPrice | Money | Original order value before discounts |
| OrderTotalDiscount | Money | Total order discount |
| OrderTotalDiscountRate | Ratio | Total discount rate |
| ValueContrubutionToShipment | Ratio | Relative contribution of the order to shipment value |
| Total Quantity | Int | Total units in the order |
| Delivery Days | Int | Days between order and delivery |
| ShipmentFeePerOrder | Money | Allocated shipping fee |
| ShipmentFeePerUnit | Int | Shipping fee per unit |
| AvgOrderRate | Int | Average rating in the order |

---

### FACT_SHIPMENTS
**Grain:** one row per shipment

Supports logistics and fulfillment analysis, especially SLA, delays, and carrier comparison.

| Field | Type | Description |
|---|---|---|
| DW_ShippingID | Varchar | Surrogate shipment identifier |
| ShippingID | Varchar | Original shipment identifier |
| ShippingDate | Date | Shipment date |
| DW_ShipmentCompanyID | Int | Linked shipment company dimension |
| ShipmentCompanyName | Varchar | Shipping company name |
| ShippingTo | Varchar | Destination city |
| Fee | Money | Shipment fee |
| ActualDeliveryDate | Date | Actual delivery date |
| OrdersInShipment | Int | Number of orders included in shipment |
| LateOrdersNum | Int | Number of late orders in shipment |
| SLA_Status | Binary / Text | OnTime or Late |
| LateOrderRate | Ratio | Late orders divided by total orders in shipment |
| TotalUnitsInShip | Int | Total units shipped |
| TotalPriceOnShip | Int | Total price of all orders in shipment |

---

## Dimension Tables

### DIM_CUSTOMERS
**Type:** Slowly Changing Dimension Type 2

Used to preserve customer history, especially when attributes like address change over time. :contentReference[oaicite:7]{index=7}

| Field | Type | Description |
|---|---|---|
| DW_Customer | Int | Surrogate customer key |
| CustomerID | Int | Original customer identifier |
| RegistrationDT | DateTime | Registration date and time |
| Birthdate | Date | Customer birth date |
| Gender | Varchar(10) | Gender |
| Region | Varchar(10) | Customer region |
| City | Varchar(30) | Customer city |
| LengthOfRelationship | Int | Customer tenure |
| Age | Int | Customer age |
| Age Group | Varchar(10) | Age segment |
| Valid From | Date | Version start date |
| Valid Until | Date | Version end date |

---

### DIM_PRODUCTS
**Type:** Slowly Changing Dimension Type 2

Tracks changes in product-related attributes over time, including price and brand-related data.

| Field | Type | Description |
|---|---|---|
| DW_ProductKey | Varchar(50) | Surrogate product key |
| ProductID | Int | Original product identifier |
| ProductName | Varchar(30) | Product name |
| Category | Varchar(30) | Product category |
| UnitCost | Money | Product unit cost |
| UnitPrice | Money | Product selling price |
| BrandID | Int | Brand identifier |
| BrandName | Varchar(20) | Brand name |
| IsGlobal | Boolean | Whether the brand is global |
| WarrantyMonths | Int | Warranty period |
| BrandAvgPrice | Int | Average brand price |
| BrandPriceClass | Varchar | Price class by percentile grouping |
| Valid From | Date | Version start date |
| Valid Until | Date | Version end date |

---

### DIM_SHIPMENT_COMPANIES
**Type:** Slowly Changing Dimension Type 2

Tracks shipping company attributes over time, especially fee changes.

| Field | Type | Description |
|---|---|---|
| DW_ShippingCompanyID | Int | Surrogate shipping company key |
| ShippingCompanyID | Int | Original shipping company identifier |
| CompanyName | Varchar | Shipping company name |
| Fee | Int | Shipment fee |
| Valid From | Date | Version start date |
| Valid Until | Date | Version end date |

---

### DIM_DATES
**Type:** Static / Type 0 time dimension

Used for date-based slicing and trend analysis.

| Field | Type | Description |
|---|---|---|
| Date | Date | Date key |
| Year | Int | Year |
| Quarter | Int | Quarter |
| Month | Int | Month |
| Day | Int | Day of month |
| Weekday | Varchar(10) | Weekday name |
| isHoliday | Boolean | Holiday flag |
| isWeekend | Boolean | Weekend flag |
| MM-YYYY | Varchar(7) | Month-year label |

---

### DIM_TIMES
**Type:** Static / Type 0 time dimension

Used for hour-of-day analysis.

| Field | Type | Description |
|---|---|---|
| Time | Time | Time key |
| Hour | Int | Hour |
| Minute | Int | Minute |
| Second | Int | Second |
| Part of the Day | Varchar(10) | Morning / Afternoon / Evening / Night |

---

## Granularity Summary

| Table | Grain |
|---|---|
| FACT_SHIPMENTS | One row per shipment |
| FACT_ORDERS | One row per order |
| FACT_ORDERITEMS | One row per product within an order |

This layered granularity allows analysis from a high-level operational shipment view down to detailed product-level profitability and rating analysis. 

---

## Why This Schema Was Chosen
The updated schema reflects several design improvements:
- Ratings were merged into FACT_ORDERITEMS because both share the same highest grain
- SHIPPINGS was removed as a dimension and redefined as FACT_SHIPMENTS because shipments are transactional by nature
- DIM_SHIPMENT_COMPANIES was added to support carrier performance analysis
- unnecessary date/time foreign keys were removed where they had no analytical contribution
- SCD Type 2 validity is managed by start and end dates rather than an `IsCurrent` flag :contentReference[oaicite:9]{index=9}

---

## Analytical Use Cases Supported
This schema supports:
- product profitability analysis
- category and brand comparison
- customer segmentation
- repeat purchase analysis
- positive review analysis
- shipping company performance analysis
- SLA and delay investigation
- drill-down and OLAP exploration

