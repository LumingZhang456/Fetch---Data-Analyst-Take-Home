## - What are the top 5 brands by receipts scanned among users 21 and over?

```sql
SELECT TOP 5 brand, COUNT(receipt_id) AS receipt_num
FROM TRANSACTION_TAKEHOME t
LEFT JOIN PRODUCTS_TAKEHOME p
ON p.barcode = t.barcode
LEFT JOIN USER_TAKEHOME u
ON t.USER_ID = u.id
WHERE DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) >= 21 AND (brand <> '')
GROUP BY brand
ORDER BY receipt_num DESC;
```
Result:
```
brand	            receipt_num
NERDS CANDY	    6
DOVE	            6
GREAT VALUE	    4
SOUR PATCH KIDS	    4
HERSHEY'S	    4
```

## - What are the top 5 brands by sales among users that have had their account for at least six months?

```sql
SELECT TOP 5 p.brand, SUM(t.FINAL_SALE) AS total_sales
FROM TRANSACTION_TAKEHOME t
LEFT JOIN PRODUCTS_TAKEHOME p
ON p.barcode = t.barcode
LEFT JOIN USER_TAKEHOME u
ON t.USER_ID = u.id
WHERE DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) >= 21 AND (brand <> '')
GROUP BY p.brand
ORDER BY total_sales DESC;
```
Result:
```
brand	        total_sales
CVS	       72
TRIDENT	       46.7200012207031
DOVE	       42.8800005912781
COORS LIGHT    34.9599990844727
QUAKER	       16.6000003814697
```

## - What is the percentage of sales in the Health & Wellness category by generation?

```sql
WITH Generations AS (
    SELECT 
        u.id,
        CASE
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 18 AND 24 THEN 'Gen Z'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 25 AND 40 THEN 'Millennials'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 41 AND 56 THEN 'Gen X'
            ELSE 'No info'
        END AS generation
    FROM USER_TAKEHOME u
)
SELECT 
    g.generation,
    ROUND(SUM(t.FINAL_SALE) * 100.0 / (SELECT SUM(final_sale) FROM TRANSACTION_TAKEHOME), 2) AS sales_percentage
FROM TRANSACTION_TAKEHOME t
LEFT JOIN Generations g 
ON t.user_id = g.id
LEFT JOIN PRODUCTS_TAKEHOME p 
ON t.barcode = p.barcode
WHERE p.category_1 = 'Health & Wellness'
GROUP BY g.generation;
```
Result:
```
generation	sales_percentage
Gen X	        0.03
Millennials	0.02
NULL	        24.04
No info	0.06
```
## - Who are Fetch’s power users?
- If it is categoried by gender:
```sql
SELECT GENDER, COUNT(receipt_id) AS receipt_num
FROM TRANSACTION_TAKEHOME t
LEFT JOIN PRODUCTS_TAKEHOME p
ON p.barcode = t.barcode
LEFT JOIN USER_TAKEHOME u
ON t.USER_ID = u.id
WHERE GENDER IS NOT NULL
GROUP BY GENDER
ORDER BY receipt_num DESC;
```
`Female` is the power group
```
GENDER	receipt_num
female	216
male	44
```
- If it is categories by ages:
```sql
WITH Generations AS (
    SELECT 
        u.id,
        CASE
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 18 AND 24 THEN 'Gen Z'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 25 AND 40 THEN 'Millennials'
            WHEN DATEDIFF(YEAR, u.BIRTH_DATE, GETDATE()) BETWEEN 41 AND 56 THEN 'Gen X'
            ELSE 'No info'
        END AS generation
    FROM USER_TAKEHOME u
)
SELECT g.generation, COUNT(receipt_id) AS receipt_num
FROM TRANSACTION_TAKEHOME t
LEFT JOIN Generations g 
ON t.user_id = g.id
LEFT JOIN PRODUCTS_TAKEHOME p
ON p.barcode = t.barcode
LEFT JOIN USER_TAKEHOME u
ON t.USER_ID = u.id
GROUP BY generation
ORDER BY receipt_num DESC;
```
`Age 18 to 24` is the power group:
```
generation	receipt_num
NULL	        49762
No info	        106
Gen X	        96
Millennials     60
```

## - Which is the leading brand in the Dips & Salsa category?
```sql
SELECT TOP 1 BRAND, COUNT(receipt_id) AS receipt_num
FROM TRANSACTION_TAKEHOME t
LEFT JOIN PRODUCTS_TAKEHOME p
ON p.barcode = t.barcode
WHERE CATEGORY_2 = 'Dips & Salsa'
GROUP BY BRAND
ORDER BY receipt_num DESC;
```
It is `TOSTITOS`
```
BRAND	  receipt_num
TOSTITOS	  72
```

## - At what percent has Fetch grown year over year?
```sql
WITH user_growth AS (
    SELECT 
        COUNT(ID) AS new_user, 
        YEAR(CREATED_DATE) AS year
    FROM 
        USER_TAKEHOME
    GROUP BY 
        YEAR(CREATED_DATE)
)
SELECT 
    curr.year,
    curr.new_user,
    prev.new_user AS prev_year_users,
    ROUND(
        ((curr.new_user - prev.new_user) * 1.0 / prev.new_user) * 100, 
        2
    ) AS growth_percentage
FROM 
    user_growth curr
LEFT JOIN 
    user_growth prev ON curr.year = prev.year + 1
ORDER BY 
    curr.year;
```
The result table is:
```
year	new_user	prev_year_users	growth_percentage
2014	30	        NULL	        NULL
2015	51	        30	        70.000000000000
2016	70	        51	        37.250000000000
2017	644	        70	        820.000000000000
2018	2168	        644	        236.650000000000
2019	7093	        2168	        227.170000000000
2020	16883	        7093	        138.020000000000
2021	19159	        16883	        13.480000000000
2022	26807	        19159	        39.920000000000
2023	15464	        26807	        -42.310000000000
2024	11631	        15464	        -24.790000000000
