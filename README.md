# Sales-Performance-Analysis

**Tools:** Microsoft Power BI, Power Query, DAX

## Project description

This project tells the story of a fictional company operating on the Polish market and answers a simple business question: **what is driving sales growth, where is the business strongest, and where should management look next?**

The analysis is based on simulated data covering the period from **January 2025 to December 2026**. The central transaction table contains **15,000 sales records**, complemented by customer, product, channel, geography, calendar, and monthly sales target data.

The project follows the full analytical workflow: raw data preparation in Power Query, dimensional modeling, DAX measure development, and finally an interactive Power BI report designed to move from a high-level business overview to more detailed analysis of channels, customer segments, products, profitability, and transaction-level context.

## Table of contents

- [Business objective](#business-objective)
- [Data structure](#data-structure)
- [Data preparation and cleaning](#data-preparation-and-cleaning)
  - [Customers](#customers)
  - [Products](#products)
  - [Geography](#geography)
  - [Sales channels](#sales-channels)
  - [Calendar](#calendar)
- [Data model](#data-model)
- [Data analysis using DAX](#data-analysis-using-dax)
- [Report structure and analytical flow](#report-structure-and-analytical-flow)
  - [Main page — business overview](#1-main-page--business-overview)
  - [Channels and customer segments](#2-channels-and-customer-segments)
  - [Products and sales efficiency](#3-products-and-sales-efficiency)
  - [Tooltip page](#4-tooltip-page)
  - [Drillthrough 1 — product detail page](#5-drillthrough-1--product-detail-page)
  - [Drillthrough 2 — monthly transaction detail](#6-drillthrough-2--monthly-transaction-detail)
- [Results](#results)
- [Findings](#findings)
- [Business recommendations](#business-recommendations)
- [Summary](#summary)

## Business objective

The purpose of the report was not only to present KPIs, but to create a guided analytical path. A user should be able to start with the overall business situation, identify the areas responsible for growth, and then move deeper into the data when a pattern requires explanation.

The report supports analysis of:

- sales performance in **2025–2026**,
- profitability and gross margin,
- sales channel and customer segment shares,
- best- and worst-performing products,
- regional performance,
- sales trends over time,
- monthly target achievement,
- order status and cancellation share,
- detailed context through tooltips and drillthrough pages.

## Data structure

The analytical model is built around the **`sprzedaz`** table, which acts as the main fact table. The remaining tables provide descriptive business context and allow the report to be filtered by time, customer, product, channel, and geography.

| Table | Number of records | Contents |
| --- | ---: | --- |
| **sprzedaz** | 15,000 | transaction data: sale and shipment dates, product, customer, channel, quantity, price, discount, cost, and order status |
| **klienci** | 500 | customer data: name, segment, region, account manager, and geographic assignment |
| **produkty** | 30 | product catalog: name, category, subcategory, brand, and base price |
| **kanaly** | 4 | sales channels and their types |
| **geografia** | 4 | regions, headquarters, and regional managers |
| **kalendarz** | 737 | date table covering 2025–2026 |
| **cele** | — | monthly sales targets by month and region |

A dedicated **`_Measures`** table is used to organize DAX measures and keep analytical logic separate from descriptive data.

## Data preparation and cleaning

The source data was intentionally prepared to resemble information coming from different business systems. Before any analysis could begin, the data had to be standardized and reshaped into a structure suitable for reporting.

The ETL process was performed in Power Query. The first stage focused on the sales table, because every later calculation depends on the quality of the transaction data.

![Sales table](screens/Tabela_sprzedaz.png)

![Sales table 1](screens/Tabela_sprzedaz_1.png)

![Sales table 2](screens/Tabela_sprzedaz_2.png)

As part of the preparation process:

- correct headers and data types were set, including appropriate regional settings for dates,
- invalid values in date columns were removed,
- customer and product identifiers were extracted from disorganized source fields,
- customer identifiers were standardized by adding the appropriate prefix,
- unnecessary spaces and redundant characters were removed,
- region names were cleaned and standardized,
- order status values were standardized as `Done`, `In progress`, and `Cancelled`,
- missing discount values were replaced with zeros,
- unnecessary technical columns were removed,
- duplicate transactions were identified and removed.

Once the fact table was cleaned, the descriptive dimensions were prepared. Customer, product, geography, and channel data were separated and standardized so that each business entity could be analyzed independently.

### Customers

![Customers table](screens/Tabela_klienci.png)

The customer table provides the context needed to analyze sales by customer, segment, region, and account manager.

### Products

![Products table](screens/Tabela_produkty.png)

The product table contains the assortment hierarchy used in the report: product, brand, subcategory, category, and base price.

### Geography

![Geography table](screens/Tabela_geografia.png)

The geography table provides a single regional dimension that can filter both customer-based analysis and sales target analysis.

### Sales channels

![Channels table](screens/Tabela_kanały.png)

A separate channel dictionary was created from the sales data so that channel analysis is based on a consistent set of values.

### Calendar

A dedicated calendar table was created dynamically. Its date range is based on the minimum and maximum dates present in the transaction data, allowing the model to adapt to the source range instead of relying on manually entered boundaries.

The calendar was enriched with year, month number, month name, weekday number, and weekday name. Text labels were assigned the correct sort order so that visuals follow chronological rather than alphabetical order.

![Calendar table](screens/Tabela_kalendarz.png)

The M transformation code is available in the **ETL** folder.

## Data model

After the data was cleaned, it was organized into a relational model centered on the **`sprzedaz`** fact table. Most relationships follow a classic one-to-many structure: dimension tables contain unique business keys, while the sales table contains repeated foreign keys for individual transactions.

![Data model](screens/Model.png)

The key relationships are:

| From | Cardinality | To | Filter direction | Status |
| --- | --- | --- | --- | --- |
| `sprzedaz[DataSprzedazy]` | `*:1` | `kalendarz[Data]` | `kalendarz → sprzedaz` | Active |
| `sprzedaz[DataWysylki]` | `*:1` | `kalendarz[Data]` | `kalendarz → sprzedaz` | Inactive |
| `sprzedaz[IDKanalu]` | `*:1` | `kanaly[IDKanalu]` | `kanaly → sprzedaz` | Active |
| `sprzedaz[IDKlienta]` | `*:1` | `klienci[IDKlienta]` | `klienci → sprzedaz` | Active |
| `sprzedaz[IDProduktu]` | `*:1` | `produkty[IDProduktu]` | `produkty → sprzedaz` | Active |
| `klienci[Region]` | `*:1` | `geografia[Region]` | `geografia → klienci` | Active |
| `cele[Region]` | `*:1` | `geografia[Region]` | `geografia → cele` | Active |
| `cele[Miesiac]` | `*:*` | `kalendarz[PierwszyDzienMiesiaca]` | bidirectional | Active |

The two date relationships serve different analytical purposes. The active relationship between **sale date** and the calendar drives the standard time analysis throughout the report. The second relationship, based on **shipment date**, remains inactive and can be activated in selected measures when shipment timing rather than sale timing is required.

The geography dimension sits one level above customers and targets. Filtering a region can therefore affect both customer-related sales analysis and the monthly target table through a consistent regional definition.

The **`cele`** table is slightly different from the other dimensions because targets are stored at a monthly level. It is linked to geography through `Region` and to the calendar through the first day of the month. This allows actual sales and planned targets to be compared in the same report context.

![Star schema model](screens/Model_gwiazda_platek.png)

Technical key columns were hidden where they were not needed by report users, keeping the field list focused on business-facing attributes and measures.

## Data analysis using DAX

Once the model was in place, DAX measures were created to turn raw transactions into business indicators. The measure layer covers both basic aggregation and analytical context, including:

- total quantity of products sold,
- gross and net sales,
- average price and average transaction value,
- cost of sales,
- gross margin and margin percentage,
- number of customers and distinct products,
- sales channel and customer segment shares,
- Year-over-Year sales dynamics,
- share of cancelled orders,
- average time from sale to shipment,
- target-related analysis.

The complete DAX measure dictionary is available in **`measures_DAX.txt`**.

## Report structure and analytical flow

The Power BI report contains **6 pages** in total. Three of them form the visible analytical story, while the remaining pages support contextual exploration through tooltip and drillthrough functionality.

### 1. Main page — business overview

The first page answers the question: **how is the company performing overall?** It brings together the main KPIs and time-based performance indicators so that the user can immediately assess sales scale, profitability, transaction volume, and growth.

The page can be filtered using four main slicers:

- `kalendarz[Rok]`,
- `geografia[Region]`,
- `kanaly[Kanał]`,
- `klienci[Segment]`.

These slicers create a common analytical context that is reused across the main report pages.

![Report page 1](images/Strona_1.png)

### 2. Channels and customer segments

The second page shifts the story from overall performance to **where revenue comes from**. It compares the contribution of individual sales channels and customer segments and makes it possible to identify concentration in the commercial structure.

The same four slicers — year, region, channel, and segment — are available here, so the user can narrow the analysis without losing the context established on the first page.

![Report page 2](images/Strona_2.png)

### 3. Products and sales efficiency

The third visible page focuses on **what is being sold and how efficiently the assortment generates value**. Product categories, individual products, sales value, and profitability can be analyzed together, which helps distinguish high-volume products from high-margin products.

Again, the page uses the same four slicers for year, region, channel, and customer segment, keeping navigation consistent across the report.

![Report page 3](images/Strona_3.png)

### 4. Tooltip page

The report also contains a dedicated hidden **Tooltip** page. Instead of forcing the user to leave the current visual, the tooltip adds an extra layer of context directly on hover.

Depending on the current filter context, the tooltip can display measures such as:

- gross sales,
- gross margin,
- margin percentage,
- margin class,
- number of transactions,
- number of customers,
- segment share,
- cancelled order share,
- Year-over-Year sales dynamics.

This design supports the storytelling flow because a user can inspect an interesting point without interrupting the main analysis.

### 5. Drillthrough 1 — product detail page

The first hidden drillthrough-style page is designed as a detailed product view. It contains product-level fields such as:

- category,
- subcategory,
- brand,
- product name,
- base price,
- gross sales,
- margin percentage,
- margin class.

### 6. Drillthrough 2 — monthly transaction detail

The second hidden drillthrough page is actively configured around:

`kalendarz[Miesiac]`

This means that a user can move from a monthly context in the report to a more detailed table showing what contributed to the selected month.

The detail view includes fields and measures such as:

- month,
- weekday,
- account manager,
- customer label,
- product category,
- total quantity,
- gross sales,
- number of transactions,
- average transaction value.

## Results

The report presents the company's sales results from January 2025 to December 2026. The story that emerges is one of strong growth, but also of concentration: a relatively small number of channels, customer segments, regions, and seasonal periods account for a large part of the result.

The main business metrics include:

- **Gross sales** — total sales value during the analyzed period,
- **Number of transactions** — total volume of sales operations,
- **Gross margin and margin percentage** — sales profitability,
- **Average transaction value** — average order value,
- **Cost of sales** — expenses related to fulfilling sales,
- **Number of customers and distinct products** — reach and assortment structure,
- **Sales channel share** — contribution of individual channels to revenue,
- **Customer segment share** — contribution of each segment to total sales,
- **Year over Year (YoY) growth** — change in sales compared with the previous year.

## Findings

The analysis starts with growth. Sales increased by **156%** compared with the previous year, rising from **PLN 2.87 million in 2025** to **PLN 7.35 million in 2026**. In other words, 2026 sales reached approximately **256% of the 2025 level**.

The average transaction value remained relatively stable, at approximately **PLN 650–710**, while monthly revenue changed much more strongly. This suggests that growth was driven primarily by a higher number of transactions rather than by a major increase in order value. September had the highest average transaction value at **PLN 711.39**, while March had the lowest at **PLN 638.09**.

Profitability remained stable as well. The **margin percentage stayed close to 50%**, so the rapid increase in sales volume did not come at the cost of a substantial decline in margin.

The time perspective reveals clear seasonality. Across 2025–2026, **November** was the strongest month, generating **PLN 2.16 million** from **3,168 transactions**. October generated **PLN 1.69 million**, and December **PLN 1.28 million**, making Q4 the most important period of the year.

When the analysis moves from time to commercial structure, concentration becomes visible. The **Partner** and **Online** channels together generated **76.38%** of gross sales. Store sales accounted for **17.06%**, while Telephone sales represented only **6.56%**.

The same pattern appears in the customer base. The **B2B** and **VIP** segments together generated **73.17%** of revenue, meaning that most sales depend on two customer groups.

Regional performance is also uneven. **South** generated **49.95%** of gross sales and **West** another **31.95%**. Together, the two largest regions accounted for **81.90%** of sales, while East and North played a much smaller role.

At product level, **P013** was the best-selling product, generating **PLN 563.99 thousand**. The five leading products together accounted for **24.40%** of sales. At the opposite end, **P027** generated **PLN 61.14 thousand**.

Finally, the order process itself performs well. **84.93%** of orders were completed, **9.47%** remained in progress, and **5.60%** were cancelled.

## Business recommendations

The report does not end with a list of KPIs. The purpose of the analytical story is to translate patterns into business actions.

- Sales in **East and North** should be investigated further by looking at product availability, account manager activity, and the effectiveness of local campaigns.
- **South and West** should be protected because together they account for **81.90%** of sales. Retention, service quality, and inventory availability are especially important in these regions.
- Investment should remain focused on the **Partner** and **Online** channels, which together generate **76.38%** of sales, while the profitability and role of Store and Telephone should be reassessed.
- Operational preparation should begin before the Q4 peak. Inventory, staffing, and campaign activity should be increased before October because November is the strongest month in the analyzed period.
- Product decisions should consider profitability, not only sales volume. **Accessories reach the highest margin at 54%**, which makes them a natural candidate for bundles and cross-selling with electronics and computers.
- The **5.60% cancellation rate** should be broken down by product, channel, region, and customer context to identify avoidable sources of lost revenue.

## Summary

This project demonstrates how a Power BI report can be designed as a connected analytical journey rather than as a collection of independent charts.

The process begins with data preparation in Power Query, continues through a relational model in which `sprzedaz` is the central fact table, and ends with a six-page report that combines overview pages with contextual navigation. Shared slicers keep the analytical context consistent, the tooltip provides additional information without breaking the flow, and drillthrough allows selected patterns — especially monthly performance — to be investigated in greater detail.

From a business perspective, the company is growing quickly while maintaining a margin of approximately **50%**. Growth is strongly concentrated in **Partner and Online channels**, **VIP and B2B segments**, **South and West regions**, and the autumn sales peak. This creates clear opportunities for further development, but it also highlights concentration risk and the need to investigate weaker regions, reduce cancellations, and improve product data quality.

The analysis covered **14,811 valid transactions** after data cleaning. Products P031–P050 were grouped into **“Others”**, which limits detailed assortment analysis for that part of the catalog and remains an area for future data-quality improvement.


