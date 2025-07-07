### What are your risk areas? Identify and describe them.

* Because I did not create any joins to answer the assigned questions, I felt my risks were minimal. I created a simple query to validate the total revenues by visitid (see query 1 below).

* I was also cautious about my rankings for question #4, which is why I developed the ranks as a subquery so that I could manually validate it. This is described under question 4; see query 2 below for a sample of the subquery referenced.

* For the questions I came up with myself, I knew that I was introducing a lot of risk by executing sequential queries and joining on the results. I also knew that the `analytics` table contains duplicates that I couldn't simply remove by excluding nulls as before, since I was interested in looking at both visits that did result in orders and visits that didn't. To confirm that I didn't duplicate revenues, I ran query 3 to count cases where revenue by fullvisitorid in the view I had created did not equal revenue by fullvisitorid in `all_sessions`. 

### Describe your QA process and include the SQL queries used to execute it.

* Query 1 checks total revenue by country from the view I created against total revenue by country from the original `all_sessions` table:

```
SELECT 
	distinct A.country AS matched_country,
	A.total_revenue AS orig_total_revenue,
	B.total_revenue AS view_total_revenue,
	100*(B.total_revenue - A.total_revenue)/A.total_revenue AS percent_error

FROM

(SELECT
	visitid,
	country,
	city,
	productsku,
	date,
	round(totaltransactionrevenue / 1000000::numeric,2) as total_revenue
	FROM all_sessions
	WHERE totaltransactionrevenue IS NOT NULL
	) AS A
JOIN 
	(SELECT visitid,
	total_revenue
	FROM session_totals
	WHERE total_revenue IS NOT NULL) AS B
ON B.visitid = A.visitid
ORDER BY percent_error DESC
```

* Query 2 generates a ranking list that allowed me to validate that my final query produced the correct top product result.

```
-- subquery generates the product rankings by country
WITH rankings AS (
	SELECT
    productname,
    country,
	sum(total_revenue),
    DENSE_RANK() OVER (PARTITION BY country ORDER BY sum(total_revenue) DESC) AS rank_by_country
  FROM session_totals
  GROUP BY country, productname
)
```

* Query 3 produces a table that shows the revenues by fullvisitorid from the `visitors_with_orders` view that I created and compares them to the revenues by fullvisitorid from `all_sessions`. It then checks whether the sum of all revenue is correct and produces either a 1 or a 0, then sums those results to count the number of instances of incorrect values.

```
WITH revcheck AS (
SELECT distinct vo.fullvisitorid, 
	totaltransactionrevenue as vo_revenue,
	A.totaltransactionrevenue AS orig_revenue
FROM visitors_with_orders vo
JOIN all_sessions A
ON vo.fullvisitorid = A.fullvisitorid
JOIN analytics B
ON vo.fullvisitorid = B.fullvisitorid
)

SELECT sum(CASE 
	WHEN vo_revenue <> orig_revenue THEN 1
	ELSE 0 END) as false_revenue
FROM revcheck
```