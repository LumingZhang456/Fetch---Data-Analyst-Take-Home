## What are the top 5 brands by receipts scanned among users 21 and over?

```sql
SELECT brand, COUNT(receipt_id) AS receipt_num
FROM PRODUCTS_TAKEHOME p
LEFT JOIN TRANSACTION_TAKEHOME t
USING(barcode)
LEFT JOIN USER_TAKEHOME u
ON t.USER_ID = u.id
WHERE (julianday('now') - julianday(birth_date)) / 365.25 >= 21 AND (brand <> '')
GROUP BY brand
ORDER BY receipt_num DESC
LIMIT 5;

