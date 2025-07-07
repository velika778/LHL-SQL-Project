What issues will you address by cleaning the data?



1. Transaction totals are overstated by 1000000

```
SELECT
	round(totaltransactionrevenue/1000000::numeric,2) as total _revenue
FROM all _sessions
```

2. City information was not always available, so I excluded missing rows when asked to comment on cities.

```
SELECT
	CASE 
	WHEN city = 'not available in demo dataset' THEN NULL
 	ELSE city END as city
FROM all _sessions
```

3. Product categories were not always available, so I excluded missing rows when asked to comment on product categories.

```
SELECT productcategory FROM all _sessions
WHERE productcategory NOT IN
('${escCatTitle}','(not set)')
```

4. New York was listed as being in Canada. Since most other visitors who were located in New York were in the United States (and I'm not aware of a New York in Canada), I amended the country label.

```
SELECT
	CASE
 	WHEN city = 'New York' THEN 'United States'
 	ELSE country END as country
FROM all _sessions
```

5. Product quantity was not always available for order information. Where a transaction occurred but no quantity information was available, and because where quantity was available it was always equal to the count of skus present in the order, I assumed a quantity of 1 item per sku.

```
SELECT 
count(distinct productsku) as num _products _ordered
FROM all _sessions
GROUP BY visitid
```

6. The majority of visits had no associated revenues, so I removed all visit data where there was no revenue for the purposes of this exercise.

```
SELECT  *
FROM all _sessions
WHERE totaltransactionrevenue <> 0 AND
totaltransactionrevenue IS NOT NULL
```
