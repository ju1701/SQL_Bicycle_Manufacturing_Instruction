# SQL_BicycleManufacturing_Instruction
*Explore Manufacturing in ***Sales, Production, and Purchasing Departments*** utilizing Google BigQuery dataset through ***demand analysis,%YoY growth rate, cost analysis,and cohort analysis,***...*

***
### Table of Content 
[I. Introduction]()
 - Prerequisites
 - How to assess the Database
 - Data dictionary, Data field, and data scheme

[II. Project objectives]()

[III. Dataset Exploration]()
 - Disclaim
 - Queries
   
***

**I. Introduction** 

The project dataset comes from a free and public dataset from BigQuery, The AdventureWorks database supports standard online transaction processing scenarios for a fictitious bicycle
manufacturer (Adventure Works Cycles). 

Scenarios include Manufacturing, Sales, Purchasing, Product Management, Contact Management, and Human Resources.

→ Human Resources: contains table of Department, Employee, Employee Department History, …

→ Person: contains tables of address, address type,..

→ Production: contains tables of bills of materials, culture, illustration,..

→ Purchasing: contains tables of ProductVendor, Purchase OrderDetail,…

→ Sales: contains table of CountryRegionCurrency, Currency, CurrencyRate,…

**- Prerequisites**

- [Google Cloud Platform account](https://cloud.google.com/)
- Project on Google Cloud Platform
- [Google BigQuery API](https://cloud.google.com/bigquery/docs/enable-transfer-service#:~:text=Enable%20the%20BigQuery%20Data%20Transfer%20Service,-Before%20you%20can&text=Open%20the%20BigQuery%20Data%20Transfer,Click%20the%20ENABLE%20button.) (Application Programming Interface) enabled
- [SQL query editor](https://cloud.google.com/monitoring/mql/query-editor) or IDE (Integrated Development Environment)
  
**- How to asses the database**

- Log in to Google Cloud Platform account and create a new project.
- Navigate to the BigQuery console and select created project.
- In the navigation panel, click “Add data”
- On the left, click “Start a project”, then type “adventureworks2019’’, it will appear on the star so that we can easily asses it later on our project.

**- Data dictionary, Data field and data scheme**

Google prepare Data Dictionary for further understanding dataset schema here: [LINK](https://www.notion.so/Project-1-1c99e008eb5e80d6a241fda407f66f7c?pvs=21)

***

**II. Project objectives**

Through querying metrics over Quantity of items, Sales value, Order quantity, % YoY growth rate ,TerritoryID with Order quantity, Total Discount Cost, Cohort analysis, ... the project aim to support for:

- Sales efficiency evaluation and tracking
- Stock management 
- Product growth and trend analysis
- Profitable sales region and optimize resource allocation
- Impact of discounts on profit margins, monitor overall discounting practices
- Customer loyalty evaluation and effectiveness of customer retention strategies
-  inventory efficiency

***

**III. Dataset Exploration by SQL**

**Disclaim:**
- The resulting query after each query just shows a few rows, for further, please access the project link [HERE], for each query, click the query button to see the full version.
- The possible insights extracted after each query come from the initial observation of the query result without context and the business understanding. To make it more clearer, the actionable insight I wrote would need further check I suggest after each query.
- For further business intelligence and decision-making support, the result might come to the dashboard visualization stage using BI tools (Power BI, Tableau,..) and the context understanding of business.
**- Query 1:** Calc Quantity of items, Sales value & Order quantity by each Subcategory in L12M (find out the latest date, and then get data in latest 12 hour)  - Sales Department

| **Metric** | **Definition** | **Example** | **What It Tells?** |
| --- | --- | --- | --- |
| **Quantity of Items** | Total number of individual units sold. | A store sells 50 laptops, 100 mice, and 200 keyboards. **Total = 350 items**. | Tracks product demand and helps in inventory management. |
| **Sales Value** | Total revenue from sales. Formula: **Quantity Sold × Price per Unit**. | (50 × $1,000) + (100 × $20) + (200 × $50) = **$62,000**. | Measures financial performance and revenue generation. |
| **Order Quantity** | Total number of customer orders placed. | 3 customers place separate orders for different products. **Total = 3 orders**. | Helps track customer purchasing behavior and order fulfillment. |
| **Subcategory** | A smaller classification within a product category. | Category: **Electronics** → Subcategories: **Laptops, Monitors, Keyboards, Mice**. | Supports product trend analysis and category-level performance insights. |

***Query:***
```sql
SELECT
      FORMAT_DATETIME('%b %Y',ModifiedDate) AS period,
      Subcategory as name,
      SUM(OrderQty) as sum_item,
      SUM(LineTotal) as sum_value,
      COUNT(SalesOrderID) as order_cnt
FROM `adventureworks2019.Sales.SalesOrderDetail` as sales
INNER JOIN `adventureworks2019.Sales.Product` as product
ON sales.ProductID=product.ProductID
WHERE sales.ModifiedDate >= TIMESTAMP(
  DATE_ADD(
    (SELECT DATE(MAX(ModifiedDate)) FROM `adventureworks2019.Sales.SalesOrderDetail`),
    INTERVAL -12 MONTH))
GROUP BY period, name
ORDER BY period desc, name asc;
```

***Query result (few rows):***
![Screenshot 2025-04-04 at 09 26 58](https://github.com/user-attachments/assets/41a05b51-4fcb-4021-aea6-4c8e782860d4)

***Example of possible evaluation after visualizing:*** 
This query group product and sales performance to have a raw data table of sales performance of latest 12 hour, havnt resolved for ordering and visualising. To make evaluation, we further visualise it into BI tools and figure out insightful patterns. This can be used for sales efficiency evaluation and stock management
-> Actionable way forward: After figuring out trend, the business can based on current patterns of problem to suggest aligned solutions. 

**- Query 2:** Calc % YoY growth rate by SubCategory & release top 3 cat with highest growth rate. (By quantity_item)

| **Aspect** | **Definition** | **Example** | **What It Tells** |
| --- | --- | --- | --- |
| **Details** | Year-over-Year (YoY) Growth Rate by Quantity measures the percentage change in the number of items sold from one year to the next. It helps track demand trends and assess business performance. | **Scenario:** A company sold **10,000 units in 2023** and **12,500 units in 2024**.   **Interpretation:** Sales volume increased by **25% from 2023 to 2024**. | **Positive Growth might reflect** Increasing demand, effective sales strategies, potential for expansion.                        **Negative Growth might reflect d**eclining demand, competition, pricing issues, or market downturn.                                **Flat or Near-Zero Growth (0-2%)** might reflect that Sales are stagnant, indicating the need for improvement in strategy or market reach.   |

***Query:***
```sql
With raw_data as    
    ( SELECT
      FORMAT_DATETIME('%Y',ModifiedDate) AS period,
      Subcategory as name,
      SUM(OrderQty) as sum_item,
    FROM `adventureworks2019.Sales.SalesOrderDetail` as sales
    INNER JOIN `adventureworks2019.Sales.Product` as product
    ON sales.ProductID=product.ProductID
    GROUP BY period, name
    ORDER BY period desc, name asc),
raw_2 as
   (SELECT
        period,
        name,
        sum_item,
        LAG(sum_item) OVER (partition by name ORDER BY period) as previous_sum 
    FROM raw_data)
SELECT 
    name,
    sum_item,
    previous_sum,
    ROUND(abs((previous_sum-sum_item)/previous_sum),2) as sum_diff
FROM raw_2
ORDER BY sum_diff DESC
LIMIT 3;
```

***Query Result:***
![Screenshot 2025-04-04 at 09 32 16 (1)](https://github.com/user-attachments/assets/1aa5135d-8502-43c4-8d12-d0a61673f736)

***Example of possible evaluation after visualizing: ***

From above results, the top 3 all reach explosive growth from 300-500%. For further action with business context, we can go check the factors for business growth (seasonality, promotions, or market trends); also prepare the supply chain to handle demand and analyze customer segments to make deeper valuation with actionable way forwards 

**- Query 3:** Ranking Top 3 TerritoryID with the  biggest Order quantity of every year. If there's TerritoryID with the same quantity in a year, do not skip the rank number

| **Metric** | **Definition** | **Example** | **What It Tells?** |
| --- | --- | --- | --- |
| **Order Quantity** | Total number of customer orders placed. | 3 customers place separate orders for different products. **Total = 3 orders**. | Helps track customer purchasing behavior and order fulfillment. |
| TerritoryID | ID of the territory in which the state or province is located. |  |  |
|  |  |  |  |

***Query:***

```sql
With raw_data as ---join 3 bảng, lấy số liệu 
    (SELECT 
    FORMAT_DATETIME('%Y',sod.ModifiedDate) AS year,
    soh.TerritoryID,
    SUM(sod.OrderQty) as sum_qty
    FROM `adventureworks2019.Sales.SalesOrderDetail` as sod
    INNER JOIN `adventureworks2019.Sales.SalesOrderHeader` soh ON soh.SalesOrderID=sod.SalesOrderID
    GROUP BY year,soh.TerritoryID),
raw_2 as --- rank
    (SELECT
    year, 
    sum_qty,
    territoryID,
    DENSE_RANK() OVER (partition by year order by sum_qty desc) as rank_qty
    FROM raw_data)
SELECT 
      year,    ---lấy 3
      territoryID,
      sum_qty,
      rank_qty
FROM raw_2
WHERE rank_qty IN (1,2,3)
ORDER BY year desc,rank_qty asc
```

***Query Result:***
![Screenshot 2025-04-04 at 09 35 53](https://github.com/user-attachments/assets/13e2968b-a47e-4fa6-82ea-6f78e658aa24)

***Example of possible evaluation after visualizing***

Overall, we can see territory 4 consistently rank first in 4 consecutive years, while territories 6 and 1  remain its positions as top 2, top 3 

-> Actionable way forward: 

Based on business context and analysis, we can analyze why Territory 4 dominates sales, prepares its supply chain, and develops or maintains products to keep its growth. 

Besides, we can investigate and boost growth for Territory 6 and Territory 3 too. 

**- Query 4:** Calc Total Discount Cost belongs to Seasonal Discount for each SubCategory

| **Aspect** | **Definition** | **Example** | **What It Tells** |
| --- | --- | --- | --- |
| **Total Discount Cost** | The total monetary value of discounts applied to sales over a period, track how much revenue was reduced due to discounts. | A company sells **1,000 items at $50 each**, but offers a **10% discount**. The **Total Discount Cost = 1,000 × ($50 × 10%) = $5,000**. | **High discount costs** may indicate aggressive promotions, while **low discount costs** suggest minimal reliance on discounts for sales. |
| **Seasonal Discount** | Discounts offered during specific seasons or events to drive sales. | A retailer offers **50% off winter coats in January** to clear inventory for spring collections. | Helps **increase sales in low-demand periods**, clear inventory, and attract customers. Can also impact profit margins if discounts are too deep. |
| **Subcategory** | A smaller classification within a product category. | Category: **Electronics** → Subcategories: **Laptops, Monitors, Keyboards, Mice**. | Supports product trend analysis and category-level performance insights. |

***Query:*** 
```sql
SELECT 
    EXTRACT(Year FROM soh.OrderDate) AS OrderYear,
    ps.Name AS SubcategoryName,
    SUM(sod.UnitPrice * sod.OrderQty * so.DiscountPct) AS TotalDiscountCost
FROM 
    `adventureworks2019.Sales.SalesOrderDetail` sod
JOIN 
    `adventureworks2019.Sales.SalesOrderHeader` soh ON sod.SalesOrderID = soh.SalesOrderID
JOIN 
    `adventureworks2019.Production.Product` p ON sod.ProductID = p.ProductID
JOIN 
    `adventureworks2019.Production.ProductSubcategory` ps ON CAST(p.ProductSubcategoryID as INT) = ps.ProductSubcategoryID
JOIN 
    `adventureworks2019.Sales.SpecialOffer` so ON sod.SpecialOfferID = so.SpecialOfferID
WHERE 
    LOWER(so.Type) LIKE '%seasonal discount%'
GROUP BY 
    EXTRACT(Year FROM soh.OrderDate)
    , ps.Name
ORDER BY 
    OrderYear asc, TotalDiscountCost DESC;
```

***Query Result:***
![Screenshot 2025-04-04 at 09 38 20](https://github.com/user-attachments/assets/eb9209a9-180f-4a31-b5f8-1e93244ff0de)

***Example of possible evaluation after visualizing***

Initially, we can see that only the subcategory “Helmets” has seasonal discounts. By which the Total Discount cost go double from 2012 to 2013. 

-> Actionable ways forward, we can go check the  business context if this product is a seasonal product, check sales volumes to see whether the discount cost drives sales,and  whether the product depends on promotion,..

**- Query 5:** Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)

**→ Cohort Analysis:** Cohort analysis is a type of behavioral analysis that segments data within a dataset into related groups before performing analysis. 

These groups, or cohorts, typically share common characteristics or experiences within a defined period of time.

Each group of users is a cohort—participants in an experiment across their lifecycles. You can compare cohorts against one another to see if, on the whole, key metrics are getting better over time.

**→ Types of cohorts:**

**Event-Based Cohorts**

This subset of behavioral cohorts groups customers based on a specific event or action—for example, all users who purchased an item during a Black Friday sale.

**Time-Based Cohorts**

This groups customers based on a specific timeframe—for example, all users who downloaded a fitness tracking app in January.

**Size-Based Cohorts.**

This groups customers by size, such as net worth or number of employees—for example, all customers who are small businesses.

**Funnel-Based Cohorts**

This groups customers according to their stage in a funnel—for example, all the people who have put an item in their online shopping cart but have not started the checkout process.
![Screenshot 2025-04-04 at 09 39 39](https://github.com/user-attachments/assets/ee3a3be7-37db-4571-b373-297bd99c235c)

A cohort analysis presents a much clearer perspective.

Cohort analysis can be done for revenue, churn, viral word of mouth, support costs, or any other metric business care about.

**→ Type of cohort for this query:** 

This query use Time-Based Cohort

| **Aspect** | **Definition** | **Example** | **What It Tells** |
| --- | --- | --- | --- |
| **Retention Rate of Customer** | The percentage of customers who continue to buy from a company over a period, or from funnel stages to other | If a company had **1,000 customers last year**, and **700 of them made a purchase again this year**, the **Retention Rate = (700 / 1,000) × 100 = 70%**. | A **higher retention rate** means strong CVR from funnel, while a **low rate** suggests customer churn. Helps businesses focus on improving customer satisfaction and engagement. |
| **Successfully Shipped Orders** | Orders that were processed, packed, and delivered to customers/ | Out of **10,000 total orders**, if **9,800 were successfully shipped**, the success rate = **(9,800 / 10,000) × 100 = 98%**. | A **high success rate** indicates an efficient logistics system, while a **low rate** may signal problems like supply chain issues, failed deliveries, or operational inefficiencies. |

***Query:***
```sql
ITH successfully_order AS (
    SELECT DISTINCT 
        CustomerID,
        FORMAT_DATE('%m', ShipDate) AS month_ship
    FROM `adventureworks2019.Sales.SalesOrderHeader`
    WHERE EXTRACT(YEAR FROM ShipDate) = 2014 
        AND Status = 5
),
customer_cohort AS (
    SELECT 
        CustomerID, 
        month_ship,
        MIN(month_ship) OVER (PARTITION BY CustomerID) AS cohort_month
    FROM successfully_order
)
    SELECT 
        cohort_month,
        CONCAT('M - ',CAST(month_ship as INT) - CAST(cohort_month as INT)) as month_diff,
        COUNT(DISTINCT CustomerID) AS customer_count
    FROM customer_cohort
    GROUP BY cohort_month, month_diff
    ORDER BY cohort_month, month_diff;
```

***Query Result(few rows):***
![Screenshot 2025-04-04 at 09 45 13](https://github.com/user-attachments/assets/c9bd81c6-4307-46b4-ad0e-8a43f7c198ef)

***Example of possible evaluation after visualizing:*** 

By further visualize this data in the dashboard, we can analyze behavioral patterns over months with a time-based cohort. 

-> Actionable way forward: For deeper insight, we can combine industry trends and business patterns to define the why among each pattern and go for business optimization decisions.
Time-based cohort helps track customer retention rates over time, measure the impact of business changes and identify seasonal trends and patterns.

**- Query 6:** :Trend of Stock level & MoM diff % by all product in 2011. If the %gr rate is null then 0. Round to 1 decimal

| **Metric** | **Definition** | **Formula** / Example | **Insights** |
| --- | --- | --- | --- |
| **Stock Level** | The quantity of inventory available at a given time, prepared to sell | Example: January stock = **500 units**, February stock = **400 units** | - Helps track **inventory health**, prevent **stockouts** or **overstocking**. Business often applies in demand forecasting, inventory optimization adn supply chain efficiency |
| **MoM Diff %** | Month-over-month percentage change in stock level. | Stock decreased by 20% | **Negative indicates** High demand, risk of **stockout**.
**Positive may implies** Overstocking, low demand, or **restocking delays**. |- These 2 metrics are important for businesses to apply in demand forecasting, inventory optimization, supply chain efficiency, seasonal planning, cost reduction and sales strategy adjustments.

***Query:***
```sql
With raw_data as
   (SELECT
     FORMAT_DATE('%m', pw.ModifiedDate) AS month,
     pp.Name as Name,
     SUM(pw.StockedQty) as sum_stocked
    FROM `adventureworks2019.Production.Product` pp 
    INNER JOIN `adventureworks2019.Production.WorkOrder` pw
    USING (ProductID)
    WHERE EXTRACT (year from pw.ModifiedDate)=2011
    GROUP BY month,Name),
raw_2 as
  (SELECT 
    month, Name,
    sum_stocked,
    LAG(sum_stocked) OVER (partition by Name order by month) as previous_month
   FROM raw_data)

SELECT 
    month,
    Name,
    sum_stocked as current_stock,
    previous_month as previous_stock,
    ROUND(NULLIF((sum_stocked-previous_month)*100/previous_month,0),1)
FROM raw_2
ORDER BY month desc, Name asc
```
***Query result (few rows):***
![Screenshot 2025-04-04 at 09 46 16](https://github.com/user-attachments/assets/48fe5535-b90e-4653-8b90-62ebd90ca2f6)

***-Example of Possible insights after visualizing:***

With an initial look, we can see a  significant decrease in stock Levels. The reasons behind this can indicate the increasing demand, which lead to deleting stock quickly; or supply chain issues, by which there are delays in restocking could have caused a significant drop or it can be the cause of seasonal sales,…

-> Actionable ways forward: for each type of cause, we need to navigate to a specific business process to see whether the assumption is accurate and suggest actionable ways forward for stock replenishment, reordering speed optimization, and forecasting improvement,..  

**- Query 7:** Calc the Ratio of Stock / Sales in 2011 by product name, by month, Order results by month desc, ratio desc. Round Ratio to 1 decimal

| **Aspect** | **Definition** | **Example** | **What it Tells** |
| --- | --- | --- | --- |
| **Definition** | The ratio of stock (inventory) to sales, indicates how many days of sales are covered by the on-hand inventory  | If a business has $100,000 in stock and $50,000 in monthly sales:  **Stock-to-Sales Ratio = 100,000 / 50,000 = 2**. This means it has enough stock to cover 2 months of sales. | - A higher ratio can indicate overstocking, meaning the company might not be turning over inventory quickly enough. Besides, a lower ratio might indicate potential stockouts or inefficient inventory management.These are useful for evaluating how well a company is managing inventory relative to demand. |

***Query:***
```sql
With sales_info as
    (SELECT
        FORMAT_DATE('%m', sod.ModifiedDate) AS month,
        pp.Name as name,
        SUM(sod.OrderQty) as sales_qty
     FROM `adventureworks2019.Sales.SalesOrderDetail` sod
     INNER JOIN `adventureworks2019.Production.Product` pp 
     USING (ProductID)
     WHERE EXTRACT (year from sod.ModifiedDate)=2011
     GROUP BY month,Name),
stocked_info as
    (SELECT
        FORMAT_DATE('%m', wo.ModifiedDate) AS month,
        pp.Name as name,
        SUM(wo.StockedQty) as stocked_qty
     FROM `adventureworks2019.Production.WorkOrder` wo
     INNER JOIN `adventureworks2019.Production.Product` pp 
     USING (ProductID)
     WHERE EXTRACT (year from wo.ModifiedDate)=2011
     GROUP BY month,Name)
SELECT
    sales_info.month,
    sales_info.name,
    sales_info.sales_qty,
    stocked_qty,
    ROUND(stocked_qty/sales_qty,1) as ratio
FROM sales_info
INNER JOIN stocked_info
ON sales_info.name=stocked_info.name and sales_info.month=stocked_info.month
ORDER BY month desc, ratio desc;
```

***Query Result:***
![Screenshot 2025-04-04 at 09 47 31](https://github.com/user-attachments/assets/fff4e6d9-ecf2-44f3-9edc-f348d26f9ebd)

***Example of Possible insight after visualizing:***

Overall we can see e common high ratio with the product listed. 

-> Actionable way forward: for this, we should analyze the benchmark of stock level in this car manufacturing industry to see whether its reasonable. Its also be influenced by inventory and sales strategy where the stock are prepared for upcoming peak sales,.. 

Combined with business context and bechmark, we could suggest actionable way forward  to optimize stock based on demand trend and cross-functional strategy. 

**- Query 8:**  No of order and value at Pending status in 2014

| **Term** | **Definition** | **Example** | **What It Tells** |
| --- | --- | --- | --- |
| **Order** | A request placed by a customer for a product or service. | A customer orders 10 bicycles from a supplier. | Shows demand and helps in production/inventory planning. |
| **Value** | The total worth of an order, calculated as **Quantity × Unit Price**. | 10 bicycles × $500 each = $5,000 | Indicates revenue potential and financial impact. |
| **Pending Status** | The state of an order that has been placed but not yet fulfilled (shipped/delivered). | A supplier confirms an order but has not yet shipped the bicycles. | Highlights possible delays, supply chain efficiency, or backlog issues. |

***Query:***

```sql
SELECT 
    COUNT(Distinct poh.PurchaseOrderID) as order_quantity,
    SUM(TotalDue) as sum_value
FROM `adventureworks2019.Purchasing.PurchaseOrderHeader` poh 
WHERE status=1;
```
***Query result (few rows)***
![Screenshot 2025-04-04 at 09 49 40](https://github.com/user-attachments/assets/b622fff8-99a4-4e01-a0cf-b26493ee6a6b)

***Example of Possible insight after visualizing***

First, compare this with the benchmark to see how significant it is to business health. 

Overall, we can see hthe igh number of order-quantity and sum_value at pending status, which might indicate possible backlog, fulfillment delays, or inefficiencies in processing orders.

-> Actionable way forwards: By analyzing this, the analyst can suggest an actionable way forward to identify bottlenecks in the supply chain, warehouse, or procurement process; prioritize boost sales for high-value products to improve business revenue and follow up with stakeholders to address issues.

***

**III. Conclusion**
- By exploring the Bicycle Manufacturing works on Google BigQuery through SQL, those reveal several interesting **insights for each business question** and prompt **actionable ways forward** for business optimization. The project proves the **power of using SQL** on Google Bigquery to gain insights through large datasets.
- From the above exploration, we explore varied metrics used in **Sales, Supply Chain, Purchasing, and  Production and approach cross-functional issues**, evaluate metrics by cohort, stock/sales,…each of the metrics indicates several insights, and understanding those and their pattern is crucial for tracking and making accurate evaluating.
- To gain comprehensive insight and evaluate key trends for business, the **next steps** would be about using Business Intelligence tools (Power BI, Tableau,..) to build up the dashboard and report to the business people in charge and suggest actionable ways forward.
