## - What are the top 5 brands by receipts scanned among users 21 and over?

```sql
SELECT TOP 5 brand, COUNT(receipt_id) AS receipt_num
FROM PRODUCTS_TAKEHOME p
LEFT JOIN TRANSACTION_TAKEHOME t
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
SOUR PATCH KIDS	    4
GREAT VALUE	    4
HERSHEY'S	    4
```

## - What are the top 5 brands by sales among users that have had their account for at least six months?

```sql
SELECT TOP 5 p.brand, SUM(t.FINAL_SALE) AS total_sales
FROM PRODUCTS_TAKEHOME p
LEFT JOIN TRANSACTION_TAKEHOME t
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
            ELSE 'Baby Boomers'
        END AS generation
    FROM USER_TAKEHOME u
)
SELECT 
    g.generation,
    ROUND(SUM(t.FINAL_SALE) * 100.0 / (SELECT SUM(final_sale) FROM TRANSACTION_TAKEHOME), 2) AS sales_percentage
FROM TRANSACTION_TAKEHOME t
JOIN Generations g ON t.user_id = g.id
JOIN PRODUCTS_TAKEHOME p ON t.barcode = p.barcode
WHERE p.category_1 = 'Health & Wellness'
GROUP BY g.generation;
```
Result:
```
generation	sales_percentage
Gen X	        0.03
Millennials	0.02
Baby Boomers	0.06
```
