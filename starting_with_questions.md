\# Question 1: Which cities and countries have the highest level of transaction revenues on the site?



\## SQL Queries:



\##### Top country query:



```

WITH session\_analytics AS (

&nbsp;	SELECT 

&nbsp;		alls.visitid AS visitor,

&nbsp;		country,

&nbsp;		city,

&nbsp;		totaltransactionrevenue,

&nbsp;		revenue,

&nbsp;		CASE

&nbsp;			WHEN totaltransactionrevenue IS NULL

&nbsp;			THEN sum(revenue) OVER(PARTITION BY a.visitid)

&nbsp;			ELSE totaltransactionrevenue

&nbsp;		END as revenue\_combined, -- can only be read as a min/max value

&nbsp;		alls.date,

&nbsp;		productquantity,

&nbsp;		productprice as price,

&nbsp;		productsku AS sku,

&nbsp;		productname,

&nbsp;		productcategory

&nbsp;	FROM all\_sessions alls

&nbsp;		JOIN analytics a

&nbsp;		ON alls.visitid = a.visitid

&nbsp;	WHERE COALESCE(totaltransactionrevenue,revenue) IS NOT NULL

)



SELECT country, MAX(revenue\_combined)

FROM session\_analytics

GROUP BY country

ORDER BY MAX(revenue\_combined) DESC

```



\##### Top city query:



```

WITH session\_analytics AS (

&nbsp;	SELECT 

&nbsp;		alls.visitid AS visitor,

&nbsp;		country,

&nbsp;		city,

&nbsp;		totaltransactionrevenue,

&nbsp;		revenue,

&nbsp;		CASE

&nbsp;			WHEN totaltransactionrevenue IS NULL

&nbsp;			THEN sum(revenue) OVER(PARTITION BY a.visitid)

&nbsp;			ELSE totaltransactionrevenue

&nbsp;		END as revenue\_combined, -- can only be read as a min/max value

&nbsp;		alls.date,

&nbsp;		productquantity,

&nbsp;		productprice as price,

&nbsp;		productsku AS sku,

&nbsp;		productname,

&nbsp;		productcategory

&nbsp;	FROM all\_sessions alls

&nbsp;		JOIN analytics a

&nbsp;		ON alls.visitid = a.visitid

&nbsp;	WHERE COALESCE(totaltransactionrevenue,revenue) IS NOT NULL

)



SELECT city, MAX(revenue\_combined)

FROM session\_analytics

WHERE city <> 'not available in demo dataset'

GROUP BY city

ORDER BY MAX(revenue\_combined) DESC

```



\## Answer:



The top countries by transaction revenue are:



| Country | Revenue|

---|:---:|

|United States|	1002780000|

|Israel|	32990000|

|Switzerland|	16990000|



The top cities by transaction revenue are:



| City | Revenue|

---|:---:|

|New York|	1002780000|

|Sunnyvale|	649240000|

|Seattle|	358000000|

|Chicago|	306000000|

|Mountain View|	244000000|

|San Jose|	153000000|

|Palo Alto|	151000000|

|San Francisco|	123000000|

|Tel Aviv-Yafo|	32990000|

|Zurich|	16990000|



\# Question 2: What is the average number of products ordered from visitors in each city and country?



\## SQL Queries:



\#####	Average number of products ordered per country:



```

WITH session\_analytics AS (

&nbsp;   SELECT 

&nbsp;       alls.visitid AS visitor,

&nbsp;       country,

&nbsp;       city,

&nbsp;       totaltransactionrevenue,

&nbsp;       revenue,

&nbsp;       CASE

&nbsp;           WHEN totaltransactionrevenue IS NULL

&nbsp;           THEN sum(revenue) OVER(PARTITION BY a.visitid)

&nbsp;           ELSE totaltransactionrevenue

&nbsp;       END as revenue\_combined, -- can only be read as a min/max value

&nbsp;       alls.date,

&nbsp;       productquantity,

&nbsp;       productprice as price,

&nbsp;       productsku AS sku,

&nbsp;       productname,

&nbsp;       productcategory

&nbsp;   FROM all\_sessions alls

&nbsp;       JOIN analytics a

&nbsp;       ON alls.visitid = a.visitid

&nbsp;   WHERE COALESCE(totaltransactionrevenue,revenue) IS NOT NULL

)



SELECT country, round(AVG(count(distinct(sku))) OVER (PARTITION BY country ORDER BY country DESC),0) as avg\_products

FROM session\_analytics

GROUP BY country

```



\#####	Average number of products ordered per city:



```

WITH session\_analytics AS (

&nbsp;   SELECT 

&nbsp;       alls.visitid AS visitor,

&nbsp;       country,

&nbsp;       city,

&nbsp;       totaltransactionrevenue,

&nbsp;       revenue,

&nbsp;       CASE

&nbsp;           WHEN totaltransactionrevenue IS NULL

&nbsp;           THEN sum(revenue) OVER(PARTITION BY a.visitid)

&nbsp;           ELSE totaltransactionrevenue

&nbsp;       END as revenue\_combined, -- can only be read as a min/max value

&nbsp;       alls.date,

&nbsp;       productquantity,

&nbsp;       productprice as price,

&nbsp;       productsku AS sku,

&nbsp;       productname,

&nbsp;       productcategory

&nbsp;   FROM all\_sessions alls

&nbsp;       JOIN analytics a

&nbsp;       ON alls.visitid = a.visitid

&nbsp;   WHERE COALESCE(totaltransactionrevenue,revenue) IS NOT NULL

)



SELECT city,

&nbsp;		round(AVG(count(distinct(sku))) OVER (PARTITION BY city ORDER BY city DESC),0) as avg\_products

FROM session\_analytics

WHERE city <> 'not available in demo dataset'

GROUP BY city

ORDER BY avg\_products DESC

```



\## Answer:



Average number of products per country:



| Country | Avg # Products|

---|:---:|

|United States|	23|

|Switzerland|	1|

|Israel|	1|



Average number of products per city:



| City | Avg # Products|

---|:---:|

|Mountain View|	7|

|San Francisco|	4|

|New York|	3|

|Sunnyvale|	2|

|Palo Alto|	1|

|Zurich|	1|

|Chicago|	1|

|Tel Aviv-Yafo|	1|

|Seattle|	1|

|San Jose|	1|



\# Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?



\## SQL Queries:



\##### Product categories ordered by visitors in country:



```

WITH session\_analytics AS (

&nbsp;   SELECT 

&nbsp;       alls.visitid AS visitor,

&nbsp;       country,

&nbsp;       city,

&nbsp;       totaltransactionrevenue,

&nbsp;       revenue,

&nbsp;       CASE

&nbsp;           WHEN totaltransactionrevenue IS NULL

&nbsp;           THEN sum(revenue) OVER(PARTITION BY a.visitid)

&nbsp;           ELSE totaltransactionrevenue

&nbsp;       END as revenue\_combined, -- can only be read as a min/max value

&nbsp;       alls.date,

&nbsp;       CASE

&nbsp;			WHEN productquantity IS NULL then 1

&nbsp;			ELSE productquantity END

&nbsp;		as quantity,

&nbsp;       productprice as price,

&nbsp;       productsku AS sku,

&nbsp;       productname,

&nbsp;       productcategory

&nbsp;   FROM all\_sessions alls

&nbsp;       JOIN analytics a

&nbsp;       ON alls.visitid = a.visitid

&nbsp;   WHERE COALESCE(totaltransactionrevenue,revenue) IS NOT NULL

)



SELECT distinct(country) as country, productcategory, sum(quantity) OVER (PARTITION BY visitor) as num\_products\_ordered FROM session\_analytics

WHERE productcategory NOT IN

&nbsp;	('${escCatTitle}','(not set)')

GROUP BY country, productcategory, visitor, quantity

ORDER BY country ASC

```



\##### Product categories ordered by visitors in each city:



```

WITH session\_analytics AS (

&nbsp;   SELECT 

&nbsp;       alls.visitid AS visitor,

&nbsp;       country,

&nbsp;       city,

&nbsp;       totaltransactionrevenue,

&nbsp;       revenue,

&nbsp;       CASE

&nbsp;           WHEN totaltransactionrevenue IS NULL

&nbsp;           THEN sum(revenue) OVER(PARTITION BY a.visitid)

&nbsp;           ELSE totaltransactionrevenue

&nbsp;       END as revenue\_combined, -- can only be read as a min/max value

&nbsp;       alls.date as date,

&nbsp;       CASE

&nbsp;			WHEN productquantity IS NULL then 1

&nbsp;			ELSE productquantity END

&nbsp;		as quantity,

&nbsp;       productprice as price,

&nbsp;       productsku AS sku,

&nbsp;       productname,

&nbsp;       productcategory

&nbsp;   FROM all\_sessions alls

&nbsp;       JOIN analytics a

&nbsp;       ON alls.visitid = a.visitid

&nbsp;   WHERE COALESCE(totaltransactionrevenue,revenue) IS NOT NULL

)



SELECT distinct(city) as city, productcategory, sum(quantity) OVER (PARTITION BY visitor) as num\_products\_ordered FROM session\_analytics

WHERE productcategory NOT IN

&nbsp;   ('${escCatTitle}','(not set)')

&nbsp;   AND city <> 'not available in demo dataset'

GROUP BY city, productcategory, visitor, quantity

ORDER BY city, num\_products\_ordered DESC

```



\## Answer:



I don't think there is enough data available to draw any conclusions about ordering patterns by either country or city.



By country:



| Country | # Products Ordered by Category|

---|:---:|

|Israel|Home/Shop by Brand/YouTube/|	1|

|Switzerland|Home/Apparel/Men's/Men's-T-Shirts/|	1|

|United States|Apparel|	1|

|United States|Home/Accessories/Fun/|	1|

|United States|Home/Apparel/Kid's/Kid's-Infant/|	1|

|United States|Home/Apparel/Men's/Men's-Outerwear/|	1|

|United States|Home/Apparel/Men's/Men's-Performance Wear/|	1|

|United States|Home/Apparel/Men's/Men's-T-Shirts/|	1|

|United States|Home/Apparel/Women's/Women's-Outerwear/|	1|

|United States|Home/Apparel/Women's/Women's-T-Shirts/|	1|

|United States|Home/Bags/Backpacks/|	1|

|United States|Home/Drinkware/|	1|

|United States|Home/Nest/Nest-USA/|	1|

|United States|Home/Shop by Brand/|	1|

|United States|Housewares|	1|

|United States|Nest-USA|	1|

|United States|Waze|	1|





Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?



SQL Queries:



Answer:





Question 5: Can we summarize the impact of revenue generated from each city/country?

SQL Queries:



Answer:

