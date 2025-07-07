I was interested in finding out if we can draw any conclusions about the amount of time a visitor spends on the site and how much money they spend. I created the following view to help me query the data:



```

CREATE OR REPLACE VIEW visitors\_with\_orders



AS 

(

WITH session\_analytics AS (

&nbsp;	SELECT

&nbsp;		a.fullvisitorid,

&nbsp;		a.visitid,

&nbsp;		s.channelgrouping,

&nbsp;		time,

&nbsp;		country,

&nbsp;		city,

&nbsp;		round(totaltransactionrevenue/1000000::numeric,2) as totaltransactionrevenue,

&nbsp;		revenue,

&nbsp;		a.pageviews,

&nbsp;		a.timeonsite,

&nbsp;		s.date,

&nbsp;		productcategory,

&nbsp;		productname,

&nbsp;		productsku,

&nbsp;		CASE

&nbsp;			WHEN a.fullvisitorid IN (SELECT distinct(fullvisitorid) FROM all\_sessions a1

&nbsp;				WHERE totaltransactionrevenue IS NOT NULL)

&nbsp;				THEN 1

&nbsp;				ELSE 0 END

&nbsp;			AS has\_made\_purchase -- identifies visitors who have made a purchase even if they have also visited the site and not made a purchase

&nbsp;	FROM analytics a

&nbsp;		LEFT JOIN all\_sessions s

&nbsp;		ON (s.fullvisitorid = a.fullvisitorid

&nbsp;		AND a.visitid IS NOT NULL -- limits the join to include only repeat visitors with unique visitids

&nbsp;		AND s.timeonsite > 0 -- limits the join to include only visitors who spent time on the site

&nbsp;		AND s.timeonsite = a.timeonsite) -- limits the join to include only visitors where timeonsite is the same across both tables

),

-- identifies the date of a visitor's first visit as well as the date of their first purchase

first\_dates AS (

&nbsp;	SELECT fullvisitorid, 

&nbsp;	MIN(date::date) OVER (PARTITION BY fullvisitorid ORDER BY date ASC) AS first\_visit,

&nbsp;	MIN(CASE when totaltransactionrevenue IS NOT NULL then date::date END) OVER (PARTITION BY fullvisitorid ORDER BY coalesce(totaltransactionrevenue,0) ASC) AS first\_purchase,

&nbsp;	has\_made\_purchase

&nbsp;	FROM session\_analytics

&nbsp;	WHERE date IS NOT NULL

&nbsp;	GROUP BY has\_made\_purchase, fullvisitorid, totaltransactionrevenue, date

&nbsp;	ORDER BY has\_made\_purchase DESC

)



SELECT 

&nbsp;	distinct(sa.fullvisitorid),

&nbsp;	first\_visit,

&nbsp;	first\_purchase,

&nbsp;	productcategory,

&nbsp;		productname,

&nbsp;		productsku,

&nbsp;		country,

&nbsp;		city,

&nbsp;	sum(totaltransactionrevenue) OVER (PARTITION BY sa.fullvisitorid) as total\_revenue, -- dedupes revenues

&nbsp;	first\_purchase - first\_visit as days\_to\_first\_purchase,

&nbsp;	count(totaltransactionrevenue::varchar) OVER (PARTITION BY sa.fullvisitorid) as num\_orders, -- dedupes the number of orders

&nbsp;	sum(timeonsite) OVER (PARTITION BY sa.fullvisitorid) as total\_timeonsite, -- dedupes time in seconds

&nbsp;	round(sum(timeonsite) OVER (PARTITION BY sa.fullvisitorid)/60,2) as total\_timeonsite\_inmins, -- dedupes time in minutes

&nbsp;	sum(pageviews) OVER (PARTITION BY sa.fullvisitorid) as total\_pageviews -- dedupes the number of page views per visitor

FROM session\_analytics sa

&nbsp;	JOIN first\_dates fd

&nbsp;	ON sa.fullvisitorid = fd.fullvisitorid

&nbsp;		AND first\_purchase IS NOT NULL -- removes duplicate lines where a visitor visited the site but did not make a purchase on that date but later did make a purchase

GROUP BY sa.fullvisitorid, 

&nbsp;	first\_visit,

&nbsp;	first\_purchase,

&nbsp;	productcategory,

&nbsp;		productname,

&nbsp;		productsku,

&nbsp;		country,

&nbsp;		city,

&nbsp;	totaltransactionrevenue,

&nbsp;	timeonsite,

&nbsp;	pageviews

ORDER BY days\_to\_first\_purchase DESC

)

```



Question 1: Do people spend more money the more time they spend on the site?



SQL Queries:



```

-- query calculates the dollar value of 1 minute on the site based on the revenue generated per unique visitor over time

SELECT

&nbsp;	DISTINCT fullvisitorid, 

&nbsp;	days\_to\_first\_purchase, 

&nbsp;	total\_revenue,

&nbsp;	total\_timeonsite\_inmins,

&nbsp;	round(total\_revenue / total\_timeonsite\_inmins,2) as time\_vs\_revenue

FROM visitors\_with\_orders

```



Answer: I played with ordering by `total\_revenue`, `total\_timeonsite\_inmins` and `time\_vs\_revenue` but did not find any meaningful correlation between time spent on the site and revenue.



!\[Revenue per minute](images/avgdays.png)



Question 2: How many days is there between the average visitor's first visit and their first purchase?

SQL Queries:



```

SELECT

&nbsp;	AVG(days\_to\_first\_purchase) as avg\_days,

&nbsp;	PERCENTILE\_CONT(0.5) WITHIN GROUP(ORDER BY days\_to\_first\_purchase) as median\_days,

&nbsp;	MODE() WITHIN GROUP(ORDER BY days\_to\_first\_purchase) AS mode\_days

FROM visitors\_with\_orders

```



Answer:

!\[Mean, median and mode number of days between first visit and first purchase](images/avgdays.png "Mean, median and mode number of days between first visit and first purchase")



Question 3: Are visitors from certain countries likely to spend more money than others?



SQL Queries:



```

SELECT

&nbsp;	distinct country,

&nbsp;	round(AVG(total\_revenue) OVER (PARTITION BY country),2)

FROM visitors\_with\_orders

WHERE country IS NOT NULL

GROUP BY country, total\_revenue

```



Answer:

!\[Avg revenue by country](images/avgrevbycountry.png "Avg revenue by country")



Question 4: Are visitors from certain countries likely to spend more time on the site than others?

SQL Queries:



```

SELECT

&nbsp;	distinct country,

&nbsp;	round(AVG(total\_timeonsite\_inmins) OVER (PARTITION BY country),0)

FROM visitors\_with\_orders

WHERE country IS NOT NULL

GROUP BY country, total\_timeonsite\_inmins

```



Answer:

!\[Avg time spent by country](images/avgtimebycountry.png "Avg time spent by country")





