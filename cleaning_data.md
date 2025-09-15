What issues will you address by cleaning the data?



1. Transaction totals are overstated by 1000000

```sql
SELECT
	round(totaltransactionrevenue/1000000::numeric,2) as total _revenue
FROM all _sessions
```

2. City information was not always available, so I excluded missing rows when asked to comment on cities.

```sql
SELECT
	CASE 
	WHEN city = 'not available in demo dataset' THEN NULL
 	ELSE city END as city
FROM all _sessions
```

3. Product categories were not always available, so I excluded missing rows when asked to comment on product categories.

```sql
SELECT productcategory FROM all _sessions
WHERE productcategory NOT IN
('${escCatTitle}','(not set)')
```

4. New York was listed as being in Canada. Since most other visitors who were located in New York were in the United States (and I'm not aware of a New York in Canada), I amended the country label.

```sql
SELECT
	CASE
 	WHEN city = 'New York' THEN 'United States'
 	ELSE country END as country
FROM all _sessions
```

5. Product quantity was not always available for order information. Where a transaction occurred but no quantity information was available, and because where quantity was available it was always equal to the count of skus present in the order, I assumed a quantity of 1 item per sku.

```sql
SELECT 
count(distinct productsku) as num _products _ordered
FROM all _sessions
GROUP BY visitid
```

6. The majority of visits had no associated revenues, so I removed all visit data where there was no revenue for the purposes of this exercise.

```sql
SELECT  *
FROM all _sessions
WHERE totaltransactionrevenue <> 0 AND
totaltransactionrevenue IS NOT NULL
```

7. I wanted to see how many products had been ordered vs. the amount in stock, including items that are out of stock, so I created a CTE called `inventory_status` to assign an inventory flag to each product.

```sql
WITH inventory_status AS (
	SELECT st.sku,
		CASE
			WHEN st.sku NOT IN (SELECT sku FROM products) then 'not stocked'
			WHEN st.sku IN (SELECT p.sku
				FROM products p
				RIGHT JOIN session_totals st
				ON p.sku = st.sku
				where stocklevel = 0) then 'no stock'
			ELSE 'in stock'
				END
			as stocking_status
	FROM session_totals st
	)

SELECT * FROM inventory_status
```

8. I wanted to review the seasonality of revenues over different periods, so I extracted the date of each order using different cuts:

```sql
SELECT country,
	productcategory, 
	productname, 
	sku,
	EXTRACT(YEAR from date_of_visit) as visityear, 
	EXTRACT(QUARTER from date_of_visit) as visitquarter, 
	EXTRACT(MONTH from date_of_visit) as visitmonth, 
	EXTRACT(DAY from date_of_visit) as visitday, 
	EXTRACT(MONTH from date_of_visit) as visitmonth, 
	DATE_PART('isodow',date_of_visit) as visitday_ofweek, -- 1 = Monday
	SUM(total_revenue) AS total_revenue
FROM session_totals
```
