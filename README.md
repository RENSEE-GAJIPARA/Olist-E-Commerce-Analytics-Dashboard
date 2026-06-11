<div align="center">

# 🛒 Olist E-Commerce Analytics Dashboard

### Power BI Practical Report 3 — Data Modelling & Interactive Dashboards

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Dataset](https://img.shields.io/badge/Dataset-Kaggle%20Olist-20BEFF?style=for-the-badge&logo=kaggle&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)
![Institute](https://img.shields.io/badge/Institute-Red%20%26%20White%20Skill%20Education-CC0000?style=for-the-badge)


---

</div>

## 📌 Project Overview

This project is a comprehensive Power BI data modelling and analytics solution built on the **Brazilian E-Commerce Public Dataset by Olist** (Kaggle). It covers end-to-end Power BI development — from raw CSV ingestion through star schema modelling to a fully interactive 3-page dashboard.

| Field | Detail |
|---|---|
| **Institute** | Red & White Skill Education |
| **Subject** | Power BI |
| **Project** | Practical Report 3 (PR 3) |
| **Total Marks** | 10 |
| **Dataset** | Brazilian E-Commerce Public Dataset by Olist (Kaggle) |
| **Dataset URL** | [https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) |

---

## 📊 Dashboard Preview

### Page 1 — Sales Overview

<img width="1825" height="1046" alt="Dashboard" src="https://github.com/user-attachments/assets/c35cc24d-605f-4877-87c9-471a1d55b8f0" />

> **Key Metrics:** 99,441 Total Orders · R$1,35,91,643.70 Total Revenue · 4.09 Avg Customer Rating  
> **Top Category:** Health & Beauty at R$1.26M revenue

---

## 🗂️ Dataset

The dataset contains **9 CSV files** connected via primary and foreign keys, forming a normalised relational schema ideal for star-schema modelling in Power BI.

| Table | Type | Rows | Key Columns |
|---|---|---|---|
| `olist_orders_dataset.csv` | Core Orders | 99,441 | order_id (PK), customer_id (FK), order_status, timestamps |
| `olist_order_items_dataset.csv` | Fact Table | 112,650 | order_id (FK), product_id (FK), seller_id (FK), price, freight_value |
| `olist_customers_dataset.csv` | Dim Customers | 99,441 | customer_id (PK), customer_city, customer_state |
| `olist_products_dataset.csv` | Dim Products | 32,951 | product_id (PK), product_category_name |
| `olist_sellers_dataset.csv` | Dim Sellers | 3,095 | seller_id (PK), seller_city, seller_state |
| `olist_order_payments_dataset.csv` | Fact Payments | 103,886 | order_id (FK), payment_type, payment_value |
| `olist_order_reviews_dataset.csv` | Fact Reviews | 99,224 | review_id (PK), order_id (FK), review_score (1–5) |
| `olist_geolocation_dataset.csv` | Geo Lookup | 1,000,163 | geolocation_zip_code_prefix, lat, lng |
| `product_category_name_translation.csv` | Category Lookup | 71 | product_category_name (FK), product_category_name_english |

---

## 🔄 Data Transformations Performed

### Task 1 — Data Loading & Power Query Transformations

- Loaded all 9 CSV files into Power BI Desktop via **Home → Get Data → Text/CSV**
- In **Power Query Editor**, merged `olist_products_dataset` with `product_category_name_translation` on `product_category_name` using a **Left Outer Join**, expanding only the `product_category_name_english` column and renaming it `Product_Category_EN`
- Renamed all 9 tables to clean, structured names: `FactOrderItems`, `DimOrders`, `DimCustomers`, `DimProducts`, `DimSellers`, `FactPayments`, `FactReviews`, `DimGeolocation`, `CategoryTranslation` (disabled from loading post-merge)

### Task 2 — Star Schema Planning

- Identified **FactOrderItems** as the central Fact Table (contains numeric measures: `price`, `freight_value`, and foreign keys)
- Identified 4 Dimension Tables: `DimOrders`, `DimCustomers`, `DimProducts`, `DimSellers`
- Mapped all Primary Key → Foreign Key relationships across tables before building them in Power BI

### Task 3 — DimDate Calendar Table (Power Query M Code)

Built a custom calendar table using **Blank Query → Advanced Editor** with the following M code, generating a full date dimension from 2016-01-01 to 2018-12-31:

```m
let
    StartDate = #date(2016, 1, 1),
    EndDate = #date(2018, 12, 31),
    Duration = Duration.Days(EndDate - StartDate),
    Dates = List.Dates(StartDate, Duration + 1, #duration(1, 0, 0, 0)),
    Table = Table.FromList(Dates, Splitter.SplitByNothing(), {"Date"}),
    ChangedType = Table.TransformColumnTypes(Table, {{"Date", type date}}),
    AddYear = Table.AddColumn(ChangedType, "Year", each Date.Year([Date]), Int64.Type),
    AddQuarter = Table.AddColumn(AddYear, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date])), type text),
    AddMonth = Table.AddColumn(AddQuarter, "Month_Num", each Date.Month([Date]), Int64.Type),
    AddMonthName = Table.AddColumn(AddMonth, "Month_Name", each Date.ToText([Date], "MMMM"), type text),
    AddWeekday = Table.AddColumn(AddMonthName, "Weekday", each Date.ToText([Date], "dddd"), type text),
    AddYearQuarter = Table.AddColumn(AddWeekday, "Year_Quarter", each Text.From([Year]) & " " & [Quarter], type text)
in
    AddYearQuarter
```

- Named the query `DimDate` with 7 columns: `Date`, `Year`, `Quarter`, `Month_Num`, `Month_Name`, `Weekday`, `Year_Quarter`
- Marked `DimDate` as the official **Date Table** via Table Tools → Mark as Date Table

### Task 4 — Relationships in Model View

Created all table relationships in **Model View** with the following configuration:

| # | Relationship | Columns | Cardinality | Cross-Filter | Status |
|---|---|---|---|---|---|
| 1 | FactOrderItems → DimOrders | order_id | Many to One (*:1) | Single | ✅ Active |
| 2 | FactOrderItems → DimProducts | product_id | Many to One (*:1) | Single | ✅ Active |
| 3 | FactOrderItems → DimSellers | seller_id | Many to One (*:1) | Single | ✅ Active |
| 4 | DimOrders → DimCustomers | customer_id | Many to One (*:1) | Single | ✅ Active |
| 5 | DimDate → DimOrders | Date → order_purchase_timestamp | One to Many (1:*) | Single | ✅ Active |
| 6 | FactPayments → DimOrders | order_id | Many to One (*:1) | Single | ✅ Active |
| 7 | FactReviews → DimOrders | order_id | Many to One (*:1) | Single | ✅ Active |
| 8 | DimDate → DimOrders | Date → order_delivered_customer_date | One to Many (1:*) | Single | ✅ Active |

### Task 5 — Star Schema vs Snowflake Schema

- **Star Schema core:** `FactOrderItems` connects directly to `DimOrders`, `DimProducts`, `DimSellers`, `DimDate` — all dimension tables are one hop from the Fact Table
- **Snowflake extension identified:** `DimOrders → DimCustomers` creates a 2-level chain (FactOrderItems → DimOrders → DimCustomers), which is a Snowflake pattern
- **Inactive relationship** created for `order_delivered_customer_date` with a dashed line in Model View; to be activated with `USERELATIONSHIP()` in DAX when needed
- **Bi-directional filter risk** demonstrated by temporarily changing `DimOrders → DimCustomers` to Both — showed ambiguous filter paths — then reverted to Single

### Task 6 — Multiple Fact Tables (Galaxy / Fact Constellation Schema)

- Connected `FactPayments[order_id] → DimOrders[order_id]` to form a **Multi-Fact Model**
- Connected `FactReviews[order_id] → DimOrders[order_id]` — resulting in a **Fact Constellation (Galaxy Schema)** with 3 Fact Tables sharing `DimOrders`
- **Filter flow confirmed:** filters propagate from Dimension → Fact (e.g., DimDate → DimOrders → FactOrderItems), never in reverse
- **Technical fields hidden** from Report View: `customer_zip_code_prefix`, `seller_zip_code_prefix`, `product_name_lenght`, `product_description_lenght`, `order_item_id`

### Task 7 — Data Formats & Data Categories

- **Currency format** applied to `FactOrderItems[price]`, `FactOrderItems[freight_value]`, and `FactPayments[payment_value]` — Symbol: R$ (Brazilian Real), 2 decimal places
- **Whole Number format** applied to `FactReviews[review_score]` and `FactPayments[payment_installments]`
- **Geography Data Categories** set: `customer_city` → City, `customer_state` → State or Province, `seller_city` → City, `seller_state` → State or Province (enables correct map plotting)
- **Sort by Column** applied to `DimDate[Month_Name]` → sorted by `Month_Num` (prevents alphabetical month sorting)
- **Model layout** organised: FactOrderItems at centre, Dimension Tables surrounding it, FactPayments and FactReviews at the bottom

### Task 8 — Hierarchies for Drill-Down

Built 4 hierarchies enabling drill-down navigation in all visuals:

| Hierarchy | Table | Levels (Top → Bottom) |
|---|---|---|
| Date Hierarchy | DimDate | Year → Quarter → Month_Name |
| Product Hierarchy | DimProducts | Product_Category_EN → product_id |
| Seller Location | DimSellers | seller_state → seller_city |
| Customer Location | DimCustomers | customer_state → customer_city |

### Task 9 — KPI Cards & Core Visuals

- **Total Orders** card: `COUNT(DISTINCT DimOrders[order_id])` → **99,441**
- **Total Revenue** card: `SUM(FactOrderItems[price])` → **R$1,35,91,643.70**
- **Avg Order Value** explicit DAX measure: `AVERAGEX(DimOrders, CALCULATE(SUM(FactOrderItems[price])))`
- **Avg Customer Rating** card: `AVERAGE(FactReviews[review_score])` → **4.09**
- **Bar Chart:** Top 10 Product Categories by Revenue (Top N filter on `Product_Category_EN`)
- **Line Chart:** Monthly Order Volume 2016–2018 using `Date Hierarchy` on X-axis
- **Map Visual:** Order Distribution by Brazilian State using `customer_state` Data Category
- **Matrix:** Payment Value by Type × Year with conditional data bar formatting

### Task 10 — Slicers, Filters & Drill-Down

- **Year Slicer** (Tile style): 2016 / 2017 / 2018 — filters all visuals simultaneously
- **Order Status Slicer** (Dropdown): delivered, shipped, canceled, invoiced, etc.
- **Product Category Slicer** (Dropdown): filters by `Product_Category_EN`
- **Report-level filter** applied: `order_status = delivered` across all pages by default
- **Drill-down demonstrated** on Line Chart: Year → Quarter → Month using Date Hierarchy
- **Filter flow validated:** selecting `health_beauty` in Category slicer updates all visuals via DimProducts → FactOrderItems relationship

### Task 11 — 3 Report Pages Built

| Page | Title | Visuals |
|---|---|---|
| 1 | Sales Overview | 4 KPI Cards + Top 10 Categories Bar Chart + Monthly Orders Line Chart + 3 Slicers |
| 2 | Geographic Analysis | Map (orders by state) + Top 10 Seller Cities Bar Chart + Revenue by Seller State Column Chart + Year Slicer |
| 3 | Payments & Reviews | Payment Type Matrix + Payment Mix Donut Chart (78.34% credit card) + Avg Review Score by Category Bar Chart + Year Slicer |

### Task 12 — Model Validation & Final Polish

- Validated all 7 active + 1 inactive relationships via **Edit Relationship** dialog — verified correct table, column, cardinality, and cross-filter direction
- Applied built-in Power BI theme for consistent colours and fonts across all 3 pages
- KPI card formatting: 28 pt Bold values, White background, light grey 1 pt border
- Added model description text box on Page 1: *"Data Model: Star Schema · 3 Fact Tables · 4 Dimension Tables · 1 Calendar Table · 7 Active Relationships · Source: Kaggle Olist Dataset"*

---

## 🗃️ Repository Structure

```
pr3/
├── Assets/
│   ├── Page1_Icon.png
│   ├── Page2_Icon.png
│   └── Page3_Icon.png
├── CSV Dataset/<img width="1825" height="1046" alt="Dashboard" src="https://github.com/user-attachments/assets/87d85752-afb2-4f92-9bc6-b58ddda58489" />

│   ├── olist_customers_dataset.csv
│   ├── olist_geolocation_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_orders_dataset.csv
│   ├── olist_products_dataset.csv
│   ├── olist_sellers_dataset.csv
│   └── product_category_name_translation.csv
├── Screenshots/
│   ├── Dashboard.png
│   ├── DrilldownFeature.png
│   ├── filter applied.png
│   ├── filter applied2.png
│   ├── filter applied3.png
│   ├── Galaxy Schema (Fact Constellation).png
│   ├── Snowflak Schema.png
│   └── Star Schema.png
├── LICENCE
├── Olist E-Commerce Analytics Dashboard.pbix
├── Olist E-Commerce Analytics Dashboard.pdf
├── PR 3.docx
└── README.md
```

## 🛠️ Tools Used

![Power BI Desktop](https://img.shields.io/badge/Power%20BI%20Desktop-F2C811?style=flat-square&logo=powerbi&logoColor=black)
![Power Query](https://img.shields.io/badge/Power%20Query%20(M)-217346?style=flat-square&logo=microsoft&logoColor=white)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=flat-square&logo=microsoft&logoColor=white)
![CSV](https://img.shields.io/badge/CSV%20Dataset-Kaggle-20BEFF?style=flat-square&logo=kaggle&logoColor=white)

---

## 👤 Author

<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=700&size=28&pause=1000&color=A7551B&center=true&vCenter=true&width=500&lines=Power+BI+Developer;Data+Analyst" alt="Typing SVG" />

<br/>

![wave](https://capsule-render.vercel.app/api?type=waving&color=A7551B&height=100&section=footer&text=RENSEE%20GAJIPARA&fontSize=24&fontColor=000000&fontAlignY=65&animation=fadeIn)

</div>
