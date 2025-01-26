# MONDAY COFFEE EXPANSION ANALYSIS WITH SQL
![](https://github.com/BERLINSAMUELRAJ/MONDAY-COFFEE-EXPANSION-ANALYSIS-WITH-SQL/blob/main/1.png)
## Objective
The goal of this project is to analyze the sales data of Monday Coffee, a company that has been selling its products online since January 2023. Based on consumer demand and sales performance, we recommend the top three major cities in India for opening new coffee shop locations.
## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Monday Coffee Dataset](https://www.kaggle.com/datasets/najir0123/monday-coffee-sql-data-analysis-project/)

## Key Questions

### 1.Coffee Consumers Count
- How many people in each city are estimated to consume coffee, given that 25% of the population does?
```sql
WITH EST AS(
	SELECT CITY_NAME, 
	CONCAT(ROUND((CAST(POPULATION AS float) * 0.25)/1000000,2), ' MILLION')  AS "TOTAL COFFEE CONSUMERS",
	DENSE_RANK() OVER(ORDER BY ROUND((CAST(POPULATION AS float) * 0.25)/1000000,2) DESC) AS "RANK"
	FROM city
)
SELECT * FROM EST
GO
```

### 2.Total Revenue from Coffee Sales (OVERALL SALES)
- What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?
```sql
WITH TOTAL_SALES_CTE AS(	
	SELECT *,
	DATEPART(QUARTER, sale_date) AS "QUARTER",
	DATEPART(YEAR, sale_date) AS "YEAR"
	FROM sales
)

SELECT CONCAT(ROUND(SUM(TOTAL)/1000000,3), ' MILLION') AS "TOTAL SALES" FROM TOTAL_SALES_CTE
WHERE YEAR= 2023 AND QUARTER=4; 
GO
```

### 3.Total Revenue from Coffee Sales (CITYWISE SALES)
- What is the total revenue generated from coffee sales for each city in the last quarter of 2023?
```sql
WITH TOTAL_SALES_CITY AS(	
	SELECT CITY_NAME,TOTAL,
	DATEPART(QUARTER, sale_date) AS "QUARTER",
	DATEPART(YEAR, sale_date) AS "YEAR"
	FROM sales S JOIN customers CU 
	ON S.customer_id= CU.customer_id 
	JOIN city C ON CU.city_id = C.city_id
)

SELECT CITY_NAME, CONCAT(ROUND(SUM(TOTAL)/1000,3), ' K') AS "TOTAL SALES",
DENSE_RANK() OVER(ORDER BY ROUND(SUM(TOTAL)/1000,3) DESC) AS "RANK"
FROM TOTAL_SALES_CITY
WHERE YEAR= 2023 AND QUARTER=4
GROUP BY city_name;
GO
```

### 4.Sales Count for Each Product
- How many units of each coffee product have been sold?
```sql
WITH  TOP_COFFEE_PRODUCTS AS(
	SELECT S.product_id, product_name, COUNT(*) AS "TOTAL ORDERS",
	DENSE_RANK() OVER(ORDER BY COUNT(*) DESC) AS "RANK"
	FROM sales S JOIN products P
	ON S.product_id = P.product_id
	GROUP BY S.product_id, product_name, total
)

SELECT * FROM TOP_COFFEE_PRODUCTS;
GO
```

### 5.Average Sales Amount per City
- What is the average sales amount per customer in each city?
```sql
WITH AVG_SALES_PER_CUSTOMER AS (
	SELECT city_name, COUNT(DISTINCT(CU.CUSTOMER_ID)) AS "TOTAL CUSTOMERS",
	SUM(S.total) AS "TOTAL SALES",
	ROUND(SUM(S.total)/COUNT(DISTINCT(CU.CUSTOMER_ID)),3) AS "AVG SALES PER CUSTOMER FOR EACH CITY",
	DENSE_RANK() OVER(ORDER BY ROUND(SUM(S.total)/COUNT(DISTINCT(CU.CUSTOMER_ID)),3) DESC) AS "RANK"
	FROM city C JOIN customers CU
	ON C.city_id = CU.city_id JOIN sales S
	ON CU.customer_id = S.customer_id JOIN products P
	ON S.product_id = P.product_id
	GROUP BY city_name
)
SELECT * FROM AVG_SALES_PER_CUSTOMER;
GO
```

### 6.City Population and Coffee Consumers 
- Provide a list of cities along with their populations and estimated coffee consumers.(Note- 25% Of people consume coffee in each city on an average)
```sql
WITH EST_CONSUMERS AS(
	SELECT city_name AS "CITY NAME", 
	COUNT(DISTINCT(S.customer_id)) AS "TOTAL CUSTOMERS",
	population AS "POPULATION",
	CONCAT((ROUND(CAST(population AS float)*0.25,3)/1000000), ' MILLION') 
	AS "ESTIMATED COFFEE CONSUMERS" 
	FROM city C JOIN customers CU
	ON C.city_id = CU.city_id JOIN sales S
	ON CU.customer_id = S.customer_id 
	GROUP BY city_name, population
	)
SELECT * FROM EST_CONSUMERS
ORDER BY population DESC
```

### 7.City Population and Coffee Consumers by Using cte's and joining cte's
- Provide a list of cities along with their populations and estimated coffee consumers.(Note- 25% Of people consume coffee in each city on an average)
```sql
WITH CITY_CTE AS (
	SELECT city_name, 
	CONCAT((ROUND(CAST(population AS float)*0.25,3)/1000000), ' MILLION') 
	AS "ESTIMATED COFFEE CONSUMERS" 
	FROM city 
),

CUSTOMERS_CTE AS(
	SELECT city_name AS "CITY NAME",
	COUNT(DISTINCT(S.customer_id)) AS "TOTAL CUSTOMERS"
	FROM city C JOIN customers CU
	ON C.city_id = CU.city_id JOIN sales S
	ON CU.customer_id = S.customer_id 
	GROUP BY city_name
)
SELECT CE.CITY_NAME, 
CE.[ESTIMATED COFFEE CONSUMERS],
CUE."TOTAL CUSTOMERS"   
FROM CITY_CTE CE JOIN CUSTOMERS_CTE CUE
ON CE.city_name = CUE.[CITY NAME]
```

### 8.Top Selling Products by City
- What are the top 3 selling products in each city based on sales volume?
```sql
WITH TOP_3_SALES_VOLUME AS(
	SELECT city_name,P.product_name, COUNT(*) AS "SALES VOLUME",  
	DENSE_RANK() OVER(PARTITION BY C.CITY_NAME ORDER BY COUNT(*) DESC) AS "RANK"
	FROM city C JOIN customers CU
	ON C.city_id = CU.city_id JOIN sales S 
	ON CU.customer_id = S.customer_id JOIN products P
	ON S.product_id = P.product_id
	GROUP BY city_name, product_name
)
SELECT * FROM TOP_3_SALES_VOLUME
WHERE RANK<=3;
GO
```

### 9.Customer Segmentation by City
- How many unique customers are there in each city who have purchased coffee products?
```sql
WITH CUSTOMER_PER_CITY AS (
	SELECT city_name,
	COUNT(DISTINCT(S.customer_id)) AS "UNIQUE CUSTOMERS"
	FROM city C JOIN customers CU
	ON C.city_id = CU. city_id JOIN sales S
	ON CU. customer_id = S.customer_id
	GROUP BY city_name
	
)
SELECT * FROM CUSTOMER_PER_CITY
ORDER BY [UNIQUE CUSTOMERS] DESC;
GO
```

### 10.Average Sale vs Rent
- Find each city and their average sale per customer and average rent per customer.
```sql
WITH CITY_SALE_RENT_CTE AS (
	SELECT city_name AS "CITY NAME",
	(SUM(S.TOTAL)) AS "TOTAL REVENUE",
	(COUNT(DISTINCT S.customer_id )) AS "TOTAL CUSTOMERS",
	ROUND((SUM(S.TOTAL))/(COUNT(DISTINCT(S.customer_id))),3) AS "AVG SALE PER CUSTOMER",
	SUM(ESTIMATED_RENT) AS "ESTIMATED RENT",
	ROUND((SUM(estimated_rent))/(COUNT(DISTINCT(S.customer_id))),3) AS "AVG RENT PER CUSTOMER"
	FROM city C JOIN customers CU
	ON C.city_id= CU.city_id JOIN sales S
	ON CU.customer_id = S.customer_id
	GROUP BY city_name
),

CITY_RENT_CTE AS (
	SELECT city_name AS "CITY NAME",
	estimated_rent AS "ESTIMATED RENT"
	FROM city
)

SELECT CSR.[CITY NAME], CSR.[ESTIMATED RENT], CSR.[TOTAL REVENUE],
CSR.[AVG SALE PER CUSTOMER],
CSR.[AVG RENT PER CUSTOMER]
FROM CITY_SALE_RENT_CTE CSR JOIN CITY_RENT_CTE CR
ON CSR.[CITY NAME] = CR.[CITY NAME];
GO
```

### 11.Monthly Sales Growth
- Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly).
```sql
WITH MonthlySales AS (
    SELECT 
        DATEPART(YEAR, SALE_DATE) AS "YEAR",
        DATEPART(MONTH, SALE_DATE) AS "PRESENT MONTH",
        SUM(total) AS "PRESENT MONTH SALES"
    FROM sales
    GROUP BY DATEPART(YEAR, SALE_DATE), DATEPART(MONTH, SALE_DATE)
)
SELECT 
    YEAR,
    "PRESENT MONTH",
    "PRESENT MONTH SALES",
    LAG("PRESENT MONTH SALES") OVER (ORDER BY YEAR, "PRESENT MONTH") AS "PREVIOUS MONTH SALES",
    (("PRESENT MONTH SALES" - LAG("PRESENT MONTH SALES") OVER (ORDER BY YEAR, "PRESENT MONTH")) 
      / NULLIF(LAG("PRESENT MONTH SALES") OVER (ORDER BY YEAR, "PRESENT MONTH"), 0)) * 100 
      AS "SALES PERCENTAGE GROWTH"
FROM MonthlySales
ORDER BY YEAR, "PRESENT MONTH";
```

### 12.Market Potential Analysis
- Identify top 3 cities based on highest sales. Return:
  - City name
  - Total sales
  - Total rent
  - Total customers
  - Estimated coffee consumers
```sql
WITH PA AS(
	SELECT city_name, 
	SUM(TOTAL) AS "TOTAL REVENUE",
	COUNT(DISTINCT CU.CUSTOMER_ID) AS "TOTAL CUSTOMERS",
	ROUND((SUM(S.TOTAL))/(COUNT(DISTINCT(S.customer_id))),3) AS "AVG SALE PER CUSTOMER"
	FROM city C JOIN customers CU
	ON C.city_id = CU.city_id JOIN sales S
	ON CU.customer_id = S.customer_id
	GROUP BY C.city_name, C.population, C.estimated_rent
),
CITY_RENT AS(
	SELECT CITY_NAME, 
	ESTIMATED_RENT AS "ESTIMATED RENT",
	CONCAT(ROUND(CAST(POPULATION AS float)*0.25/1000000,3), ' MILLION') AS "ESTIMATED COFFEE CONSUMER"
	FROM city
)
SELECT PAS.city_name,
[TOTAL REVENUE],
[TOTAL CUSTOMERS],
[ESTIMATED COFFEE CONSUMER],
[AVG SALE PER CUSTOMER],
ROUND([ESTIMATED RENT]/[TOTAL CUSTOMERS],3) AS "AVG RENT PER CUSTOMER"
FROM PA PAS JOIN CITY_RENT CRE
ON PAS.city_name = CRE.city_name
ORDER BY 2 DESC;
GO
```
## Recommendations
After analyzing the data, the recommended top three cities for new store openings are:

### **City 1: Pune**
- Average rent per customer is very low.
- Highest total revenue.
- Average sales per customer is also high.

### **City 2: Delhi**
- Highest estimated coffee consumers at 7.7 million.
- Highest total number of customers, which is 68.
- Average rent per customer is 330 (still under 500).

### **City 3: Jaipur**
- Highest number of customers, which is 69.
- Average rent per customer is very low at 156.
- Average sales per customer is better at 11.6k.

## How to Use This Data
This analysis provides insights into customer demand, sales trends, and market potential in different cities. Businesses looking to expand their coffee shop locations can use this data to make informed decisions.

---

### Contributors
- Monday Coffee Data Team

### License
This project is open-source and available under the [MIT License](LICENSE).
