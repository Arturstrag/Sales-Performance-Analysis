# Sales-Performance-Analysis

**Tools**: Microsoft Power BI, Power Query, DAX

## Project description

This project presents an analysis of simulated sales data for a fictional company operating in the Polish market. The dataset contains more than **16,000** records in **6** related tables covering sales, customers, products, sales channels, geography, the calendar, and monthly sales targets.

The main transaction table contains **15,000** sales records. The remaining tables serve as dimension and supporting tables. The data covers the period from January **2025** to December **2026**.

## Table of contents
  - [Business objective](#business-objective)
  - [Data structure](#data-structure)
  - [Data preparation and cleaning](#data-preparation-and-cleaning)
  - [Data model](#data-model)
  - [Data analysis using DAX](#data-analysis-using-dax)
  - [Results](#results)
  - [Findings](#findings)
  - [Business recommendations](#business-recommendations)
  - [Summary](#summary)

## Business objective

The main objective was to create an interactive report enabling the analysis of:

- sales performance in **2025–2026**,
- profitability and margin,
- sales channel and customer segment shares,
- best- and worst-performing products,
- regional performance,
- sales trends over time and target achievement.

## Data structure

The model consists of 7 tables:

| Table | Number of records | Contents |
| --- | ---: | --- |
| **sales** | 15,000 | transaction data: sale and shipment dates, product, customer, region, channel, quantity, price, discount, cost, and order status |
| **customers** | 500 | customer data: name, segment, region, account manager, and geographic assignment |
| **products** | 30 | product catalog: name, category, subcategory, brand, and base price |
| **channels** | 4 | sales channels and their types |
| **geography** | 4 | regions, headquarters, and regional managers |
| **calendar** | 737 | date table covering 2025–2026 |

## Data preparation and cleaning

The data was intentionally prepared to resemble information originating from different business systems. Therefore, it required transformation and organization before analysis.

The ETL process was performed in Power Query, which was used to import, transform, and prepare the data for further modeling and analysis.

The data-cleaning process began with the sales table.

![Sales table](screens/Tabela_sprzedaz.png)
![Sales table 1](screens/Tabela_sprzedaz_1.png)
![Sales table 2](screens/Tabela_sprzedaz_2.png)

As part of the data preparation:

- correct headers and data types were set, including appropriate regional settings for dates,
- invalid values in date columns were removed,
- correct identifiers were extracted from disorganized customer and product fields,
- customer identifiers were standardized by adding the appropriate prefix,
- unnecessary spaces and redundant characters were removed,
- region names were cleaned and standardized,
- order status values were standardized as `Done`, `In progress`, and `Cancelled`,
- missing discount values were replaced with zeros,
- unnecessary technical columns were removed,
- duplicate transactions were identified and removed.

Separate customer, product, geography, and sales channel tables were also prepared and organized. A channel dictionary was created based on the sales data, and the product table was enriched with price information and the average product price calculated through grouping.

**Customers table**

![Customers table](screens/Tabela_klienci.png)

**Products table**

![Products table](screens/Tabela_produkty.png)

**Geography table**

![Geography table](screens/Tabela_geografia.png)

**Channels table**

![Channels table](screens/Tabela_kanały.png)

One element of the transformation was the creation of a dedicated calendar table. Its range was determined automatically based on the minimum and maximum sales dates, allowing the calendar to adapt to the source data range.

The table was enriched with the year, month number and name, weekday number, and weekday name. Month and weekday names were formatted appropriately, and text columns were assigned the correct sort order.

![Calendar table](screens/Tabela_kalendarz.png)

The M transformation code is available in the **ETL** folder.

## Data model

After the transformation was completed, a relational data model was created based on separating the fact table from the dimension tables. The sales table serves as the central fact table, while customer, product, channel, geography, and time data serve as dimensions that allow results to be filtered and grouped.

![Data model](screens/Model.png)

Relationships were created manually, taking cardinality and filter direction into account. Two relationships were prepared between the sales and calendar tables: an active relationship for the sale date and an inactive relationship for the shipment date. Technical key columns were hidden, and the model structure was organized to improve readability.

At a later stage, a table of monthly sales targets was also added to the model by linking it to the calendar and geography tables.

![Star schema model](screens/Model_gwiazda_platek.png)

## Data analysis using DAX

A set of DAX measures was created to calculate, among other things:

- total quantity of products sold,
- gross and net sales,
- average price and average transaction value,
- cost of sales,
- gross margin and margin percentage,
- number of customers and distinct products,
- sales channel and customer segment shares,
- average time from sale to shipment.

The complete DAX measure dictionary is available in **measures_DAX.txt**.

## Results

The report presents the company's sales results from January 2025 to December 2026. Key indicators include gross sales, number of transactions, margin percentage, and sales growth compared with the previous year. The dashboard presents the following business metrics:

- **Gross sales** — total sales value during the analyzed period,
- **Number of transactions** — total volume of sales operations,
- **Gross margin and margin percentage** — sales profitability,
- **Average transaction value** — average order value,
- **Cost of sales** — expenses related to fulfilling sales,
- **Number of customers and distinct products** — reach and product assortment structure,
- **Sales channel share** — contribution of individual channels to revenue,
- **Customer segment share** — contribution of each segment to total sales,
- **Year over Year** (YoY) % growth — change in sales compared with the previous year.

![Report page 1](images/Strona_1.png)
![Report page 2](images/Strona_2.png)
![Report page 3](images/Strona_3.png)

## Findings

The key findings from the analysis are as follows:

- **Dynamic sales growth**: sales increased by **156%** compared with the previous year. Sales amounted to **PLN 2.87 million** in 2025 and **PLN 7.35 million** in 2026. This means that sales reached **256%** of the previous year's level.
- **Average transaction value** remained within the range of approximately **PLN 650–710**, while monthly sales varied significantly. Growth was therefore driven mainly by a higher number of transactions. September had the highest average transaction value at **PLN 711.39**, while March had the lowest at **PLN 638.09**.
- **Margin percentage** remained at approximately **50%**.
- **Strong seasonality** was visible. Across 2025–2026, November was the best month, generating **PLN 2.16 million** from **3,168** transactions. October generated **PLN 1.69 million**, and December **PLN 1.28 million**. Peak demand therefore occurred in Q4.
- Sales were strongly concentrated in the **Partner** and **Online** channels, which together accounted for **76.38%** of gross sales. Store sales accounted for **17.06%**, while Telephone sales accounted for only **6.56%**.
- The **B2B** and **VIP** customer segments played the most important role, together accounting for **73.17%** of revenue.
- Sales varied significantly by region: South generated **49.95%** of gross sales, West **31.95%**, East **11.12%**, and North **6.98%**. The two largest regions together accounted for **81.90%** of sales.
- The best-selling product was **P013**, generating **PLN 563.99 thousand**. The five best-selling products together accounted for **24.40%** of sales. The weakest product, **P027**, generated **PLN 61.14 thousand**.
- Order fulfillment was strong: **84.93%** of orders were completed, **9.47%** were in progress, and only **5.60%** were cancelled.

## Business recommendations

- Strengthen sales in the East and North regions by analyzing product availability, sales representative activity, and the effectiveness of local campaigns.
- Protect the South and West regions. Together, they account for **81.90%** of sales, so loyalty programs should be developed and appropriate inventory levels maintained.
- Focus investment on the **Partner** and **Online** channels, which together generate **76.38%** of sales. Increase the online marketing budget and reward the most effective partners.
- Improve the efficiency of the Store and Telephone channels, whose combined share is only **23.62%**. Assess their profitability and implement cross-selling initiatives, particularly during the Q4 seasonal peak.
- Increase inventory, staffing, and campaign activity before October. November generates the highest combined sales across **2025–2026**.
- Develop sales of high-margin products.
- Accessories achieve the highest margin at **54%**. A good approach would be to offer them in bundles with electronics and computers.
- Reduce the share of cancelled orders, currently **5.60%**, by analyzing cancellations by product, channel, and region.

## Summary

The 2025–2026 sales analysis made it possible to assess revenue dynamics, category profitability, the importance of channels and customer segments, and order fulfillment effectiveness. Before the analysis, the data was cleaned and standardized in Power Query. The analysis covered 14,811 transactions; however, products assigned to the “Others” group require further organization.

The most important conclusions are:

- Sales are in a phase of dynamic growth. Gross sales increased from **PLN 2.87 million** in 2025 to **PLN 7.35 million** in 2026, representing a 156.09% increase.
- Sales are strongly concentrated in the Partner and Online channels. Partner accounts for **39.51%** and Online for **36.88%** of gross sales. Together, they generate **76.38%** of the result, while Store accounts for **17.06%** and Telephone for **6.56%**. Further investments should focus on the most effective channels while evaluating the profitability of the remaining ones.
- The **VIP** and **B2B** segments form the foundation of company revenue. Together, they generate **73.17%** of sales, supporting the development of personalized service, loyalty offers, and retention activities. However, this high concentration also creates dependence on a limited customer base.
- Sales performance varies significantly by region. South accounts for **49.95%** and West for **31.95%** of gross sales. Together, these regions generate **81.90%** of the result, while East and North account for only **18.10%**. This requires both protecting the position of the key regions and investigating growth potential in weaker markets.
- High electronics sales do not translate into the highest profitability. Electronics generate the largest sales value at **PLN 3.62 million**, but their margin is **48%**. Accessories achieve the highest margin at **54%**, which supports developing bundle sales and cross-selling accessories with electronics and computers.
- Sales show clear seasonality. Across 2025–2026, November was the best month, generating **PLN 2.16 million** from **3,168** transactions. The increase in demand during Q4 indicates the need to prepare inventory, staffing, and marketing campaigns in advance.
- The order fulfillment process performs well. **84.93%** of orders were completed, **9.47%** remain in progress, and **5.60%** were cancelled. Cancellations should be analyzed by channel, product, and region, with the aim of reducing their share below **4%**.
- Product data quality requires further improvement. Products P031–P050 were aggregated into “Others”, causing this group to represent a significant share of sales and making it difficult to identify the actual assortment leaders. Their names, brands, and categories should be completed before detailed product decisions are made.

In summary, the company is achieving dynamic growth with a high margin of approximately **50%**. The greatest potential for further development lies in the **Partner** and **Online** channels, **VIP** and **B2B** segments, profitable accessories, and effective use of the autumn season. Key challenges include sales concentration, development of weaker regions, reducing cancellations, and improving product data quality.
