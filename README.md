# Final Project Transforming and Analyzing Data with SQL

### Project/Goals

I wanted to challenge myself to write code that was simple and easy to follow with minimal commentary. I also wanted to arrive at the most complete dataset possible without introducing any new errors.

### Process

1. I retrieved the relevant data files from the [LHL Shared Drive](https://drive.google.com/drive/folders/1efDA4oc9w bTbAvrESdOJpg9u gEUBhJ) and created 5 tables:
    * all_sessions
    * analytics
    * products
    * sales_by_sku
    * sales_report

2. I reviewed the tables to identify potential linkages and keys
    * `productsku` appears to be the only complete key available, appearing in `products`, `sales_by_sku`, `sales_report` and `all_sessions`. I created this as a primary key in `products` and as a foreign key in all other tables where it's present.
    * `fullvisitorid` and `visitid` are potential keys shared between `all_sessions` and `analytics`, but contain too many nulls. I did not create them as keys but did use them for other analysis.

3. With reference to the assigned questions, `all_sessions` appears to be the only table that contains order data by city and country. I considered linking `all_sessions` with `analytics` using `visitid` as a key, which would have allowed me to fill in some missing product quantity and revenue values. However, there were too few matches between the 2 tables to produce meaningful results (see below under Challenges), so I chose to focus on `all_sessions` alone.

### Results

* I found that limited insights were available regarding completed orders, as most interactions did not lead to a purchase.
* `products` and `sales_report` contain information about inventory levels, but no references to date which made it impossible to tell what point in time they reflect.
* There did not seem to be any connection between `products`, `sales_report` and/or `all_sessions` or `analytics` which meant that I also couldn't tell what impact orders had on inventory levels.
* Since most fields in `all_sessions` were irrelevant to answering the assigned questions, I created a view called `session_totals` that I used to query a subset of the data more efficiently:

```
CREATE OR REPLACE VIEW session_totals
AS 
    (SELECT 
	visitid,
        CASE
			WHEN city = 'New York' THEN 'United States'
			ELSE country END as country, -- corrects item where New York appears under Canada
        city,
	round(totaltransactionrevenue/1000000::numeric,2) as total_revenue,
	round(productprice/1000000::numeric,2),
        date::DATE as date_of_visit,
        productsku AS sku,
        productname,
        productcategory
    FROM all_sessions
    WHERE totaltransactionrevenue <> 0 AND
	totaltransactionrevenue IS NOT NULL)
```


### Challenges

* I wanted to have as complete data as possible regarding revenue and quantity, so initially I linked `all_sessions` with `analytics` using `visitid` as a key. At first I thought I could simply use `coalesce` on `totaltransactionrevenue` and `revenue`, but because `analytics` contains duplicate `visitid`s while they're unique in `all_sessions`, I ended up having to dedupe them first using `sum(revenue)/count(visitid)`. Once I had done this, I validated the results against `totaltransactionrevenue` but found that where `totaltransactionrevenue` already existed in `all_sessions`, it was sometimes a different value than the one I got from deduping. But because the values sometimes did match, I couldn't conclude that my logic was wrong but rather that `totaltransactionrevenue` and `revenue` only sometimes contain the same values as a deliberate choice. This was also true when I tried to backfill product quantity using similar logic. Without a data dictionary to confirm how these fields are related (if at all), I decided I couldn't join the 2 tables for this purpose and was therefore left with less precise information.

* I wanted to look at how much time each visitor spent on the site, but found that many of the relevant columns were missing values with no discernible pattern. This meant I only had a very small sample size to answer any questions about correlation between time spent on the site, number of visits, revenue earned, etc.

### Future Goals

* I would like to spent more time looking at what products users viewed but ultimately did not purchase. What products are users spending the most time viewing? Can we determine why they didn't end up purchasing the item(s), e.g. is a stock issue? Checkout issue?
* I would also have liked to understand some of the columns better to provide deeper insights. For example, what does sentiment refer to in the `sales_report` table?