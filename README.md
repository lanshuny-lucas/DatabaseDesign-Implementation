# Beverage Retail Database — Design, Implementation & Analytics

> **One‑sentence description:** End‑to‑end relational database for a beverage chain covering orders, inventory, logistics, promotions, and feedback, with SQL reports and BI (regression + ARIMA) for operational decisions. 

---

## 1) Project Overview
This repository documents a complete database solution for **Notting’s Taste**, a fast‑growing beverage brand. It covers the full lifecycle: **conceptual design → logical schema → SQL implementation → data visualization → business intelligence (regression & ARIMA forecasting)** to support strategic decisions on **store expansion, channel mix, and product portfolio**. 【60†UG‑DDI Database Design Report】

**Why this matters.** The company’s expansion created unstable profits and data silos. The new database integrates **orders, stock, deliveries, suppliers, discounts/ads, invoices, and customer feedback** so HQ can analyze profitability, detect bottlenecks, and optimize operations. 【60†UG‑DDI Database Design Report】

---

## 2) Business Context & Requirements
- Pre‑made tea line launched in 2022; stores grew from 15 (Ningbo) to 175 nationwide, but **store income fluctuates** and **profit margins lag**. The legacy database can’t explain the causes. 【60†UG‑DDI Database Design Report】  
- The new system must support three goals:  
  1) **Third‑party analysis** (supplier/logistics performance),  
  2) **Store operations** (orders, inventory, staffing, invoicing),  
  3) **Product performance** (cost, price, discounts, acceptance). 【60†UG‑DDI Database Design Report】

**Key business rules (selected):**  
- Materials auto‑deduct when products are made; warehouse auto‑updates upon supplier receipt.  
- Delivery orders forward customer info to logistics; pickup orders issue a pickup code automatically.  
- Discounts require daily manual confirmation; invoices auto‑generated for both materials and sales.  
- Each shop has **one manager**; other staff can check stock and process orders.  
- Real‑time updates and **historical data retention** (sample demo shows one day). 【60†UG‑DDI Database Design Report】

---

## 3) System Architecture (Conceptual Design)
**Core modules:** orders, customer services, logistics, stock, advertising, staff (see ER model). 【60†UG‑DDI Database Design Report】

**Main entities (excerpt):**  
- **Customer, Staff, Shop, Supplier, LogisticsProvider** — actors and locations  
- **Product, Material, Warehouse** — catalog and inventory  
- **Order, OrderItem, MaterialOrder, MaterialOrderItem** — sales & procurement flows  
- **Advertisement, AdvertisementItem, DiscountEvent, DiscountedItem** — promotions/discounts  
- **CustomerFeedback** — NPS/ratings on product, shop, logistics  
- **Invoices (SaleInvoice, PurchaseInvoice)** — accounting integration  
- **DeliveryRecord, PickUpRecord, InventoryInRecord, InventoryOutRecord** — execution logs  
All entities include properly defined **PK/FK constraints, data types, and nullability** (see Data Dictionary). 【60†UG‑DDI Database Design Report】

> 💡 *ERD tip:* Keep relationship attributes in relationship tables; do not replicate FKs in entities twice; mark PK/FK clearly. 【60†UG‑DDI Database Design Report】

---

## 4) Data Dictionary (Snapshot)
The report provides a comprehensive dictionary with attribute semantics, types, and keys. Highlights:  
- **Customer**(`Customer_ID` PK, `cFName`, `cLName`, `cContactNumber` AK, `cGender`, `cAge`)  
- **Staff**(`Staff_ID` PK, `Shop_ID` FK→Shop, `stFName`, `stLName`, `stPosition`, `stHireDate`, `stEndDate`)  
- **Product**(`Product_ID` PK, `pName` AK, `pPrice`, `pCost`, `pType` {P/F}, `pSize`)  
- **Order**(`Order_ID` PK, `Customer_ID` FK, `Staff_ID` FK, `oDate`, `oTime`, `oTotalPrice`, `oType` {On/Off}, `oChannelType` {P/D})  
- **Warehouse**(composite PK: `WH_ID`,`Material_ID`, `whCurrentQuantity`)  
- **DeliveryRecord**(PK: `LP_ID`,`Order_ID`,`Customer_ID`, with expected/actual time, timeout flag & status)  
- **Promotion & Discount** via `AdvertisementItem` and `DiscountedItem` junction tables. 【60†UG‑DDI Database Design Report】

---

## 5) Logical Schema (Relational Design)
The logical model normalizes core entities and junctions with **composite PKs** and **derived fields** (e.g., durations). Example constraints:  
- `OrderItem` PK(`Order_ID`,`oiNumber`); FKs to Order/Product/Staff.  
- `ProductComponent` PK(`Product_ID`,`Material_ID`) links BOM to materials.  
- `SaleInvoice` PK(`Shop_ID`,`Order_ID`); `PurchaseInvoice` PK(`Shop_ID`,`MO_ID`).  
- `AdvertisementItem` PK(`Product_ID`,`AD_ID`); store `adiStartDate/adiEndDate/adiCost`.  
- `DiscountedItem` PK(`Product_ID`,`DE_ID`); store `dpDiscountedPrice`. 【60†UG‑DDI Database Design Report】

> 🧭 **Workflows covered:** Order‑to‑Cash, Procure‑to‑Stock, Promotions, and Feedback‑to‑Action. 【60†UG‑DDI Database Design Report】

---

## 6) Feature Use‑Cases & Example SQL
Below are representative analytics/ops queries implemented and showcased in the report (with sample outputs).

1) **Customer order history** (items & totals by `Customer_ID`).  
2) **Current discounted price** for a given `Product_ID`.  
3) **Top‑5 popular products** overall (recommender‑style).  
4) **Top‑3 frequently co‑purchased items** with a given product.  
5) **Employee order stats** (by `Staff_ID`).  
6) **Employee feedback pulled from orders** handled.  
7) **Logistics timeout rate** per provider.  
8) **Profit margin ranking** across products.  
9) **Total sales of a shop** (by `Shop_ID`).  
10) **Channel mix counts** (order channel × delivery channel).  
11) **Current warehouse stock** by `Shop_ID` + `Material_ID`. 【60†UG‑DDI Database Design Report】

> The report includes screenshots of results for examples like **C001**, **P001**, **ST001**, **LP001**, and **S001** to verify correctness. 【60†UG‑DDI Database Design Report】

---

## 7) Data Visualization (Ops Dashboards)
- **Product strategy:** stacked bars for pre‑made vs fresh tea by province (sales, cost, profit); radar for margins & preference; growth trends over months; regional preference maps; word clouds for customer tastes.  
- **Store monitoring:** store sales vs inventory; ratings map; daily order volume map.  
- **Channel monitoring:** heatmap by hour×channel; Sankey for monthly channel flows; line chart for channel sales by hour. 【60†UG‑DDI Database Design Report】

---

## 8) Business Intelligence (Insights)
### 8.1 Regression Analysis
Five models evaluate the effect of core drivers on **Ln(Sales)**: Temperature, Advertisement, Discount, Rating, **Location**, **Channel**, **Type**, and **three interaction terms**. Key findings:  
- Baseline shows all core drivers significant; adj. R² ≈ 0.35 → add missing factors.  
- **Location (1st/2nd‑tier)** ≈ **+7%** vs others; suggests reevaluating expansion mix.  
- **Channel (online)** ≈ **+26%** vs offline; expand online presence.  
- **Type (pre‑made)** ≈ **+20%** vs non‑pre‑made; continue the product line.  
- **Interactions** among Location×Channel×Type are all positive (>0.3) and significant, indicating **reinforcing effects** when combined. 【60†UG‑DDI Database Design Report】

### 8.2 Forecasting (ARIMA)
Using one‑year historical data, ARIMA yields good fit (significant Wald test), with projected **rising sales trend** and plausible short‑term dips (e.g., launch frictions, outages). Forecast corroborates regression insights: **downsize low‑yield stores, grow online, double‑down on pre‑made**. 【60†UG‑DDI Database Design Report】

---

## 9) How to Reproduce (Suggested Repo Layout)
```
beverage-db/
├─ schema/               # SQL DDL (tables, constraints, indexes)
│  ├─ create_tables.sql
│  └─ sample_seed.sql
├─ queries/              # All analytics/ops SQL from the report
│  ├─ customer_history.sql
│  ├─ discounted_price.sql
│  ├─ top_popular.sql
│  ├─ co_purchase.sql
│  ├─ staff_orders.sql
│  ├─ staff_feedback.sql
│  ├─ logistics_timeout.sql
│  ├─ product_margins.sql
│  ├─ shop_sales.sql
│  ├─ channel_mix.sql
│  └─ stock_by_shop_material.sql
├─ viz/                  # Charts exported as PNG (from notebook)
│  ├─ product_strategy/
│  ├─ store_monitoring/
│  └─ channel_monitoring/
├─ notebooks/
│  ├─ regression.ipynb   # OLS models (1)–(5)
│  └─ forecasting.ipynb  # ARIMA (ACF/PACF, fit, forecast)
└─ README.md
```

### Setup (example with PostgreSQL)
```bash
psql -U <user> -d <db> -f schema/create_tables.sql
psql -U <user> -d <db> -f schema/sample_seed.sql
```

### Run sample analytics
```bash
psql -U <user> -d <db> -f queries/top_popular.sql
```

> If you plan to share the repo publicly, include **synthetic seed data** (no sensitive data).

---

## 10) Design Notes & Best Practices
- **Keys & Normalization:** composite PKs for junctions; AKs on unique business fields (e.g., contact numbers, product names + size).  
- **Indices:** add indexes on FK columns and frequent filters (`Order.Customer_ID`, `DeliveryRecord.drStatus`, `Warehouse.Material_ID`).  
- **Data quality:** enforce domains (`pType` in {P,F}; `oChannelType` in {P,D}); use check constraints for ratings/timeouts.  
- **Auditing:** keep `*_Time` fields and use derived durations for SLAs (delivery expected vs actual).  
- **Security:** grant least privilege by role (HQ analyst, store staff, logistics).  
- **Performance:** batch updates for inventory; avoid double‑counting in promotions/discounts logic. 【60†UG‑DDI Database Design Report】

---

## 11) Limitations & Future Work
- Current sample shows **one‑day data**; integrate longer horizons for robust trends.  
- Missing **non‑product financials** (rent, taxes, decoration) → add to improve true profitability modeling.  
- Inventory thresholds are subjective; consider **EOQ / safety stock** logic.  
- Add CDC/ETL to support near‑real‑time dashboards. 【60†UG‑DDI Database Design Report】

---

## 12) Contributors
Renyu Jiang · Yuxiao Deng · Ziyu Liu · Zhengyi Lin · **Lanshun Yuan** (BI analysis lead) 【60†UG‑DDI Database Design Report】

---

## 13) License
MIT (for schema and example SQL). Replace if your course requires a different license.

