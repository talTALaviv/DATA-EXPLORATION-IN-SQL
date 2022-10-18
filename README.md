# DATA-EXPLORATION-IN-SQL

Analyze eCommerce database

# 4 - Analyzing Traffic Sources

# Bid Optimization & Trend Analysis #

SELECT 
	YEAR(created_at) AS YEAR,
	WEEK(created_at) AS WEEK,
	MIN(DATE(created_at)) AS Week_start,
    COUNT(DISTINCT website_session_id) AS Sessions
FROM website_sessions
GROUP BY 1,2
--------------------------------------------------------------------------------------------------

ֳֳ## Pivoting Data with count & case

SELECT 
    primary_product_id,
    COUNT(CASE WHEN items_purchased = 1 THEN order_id ELSE NULL END ) AS count_single_item_orders, 
    COUNT(CASE WHEN items_purchased = 2 THEN order_id ELSE NULL END ) AS tow_single_item_orders
FROM orders
GROUP BY 1

--------------------------------------------------------------------------------------------------
# Traffic Source Trending, monitor volume levels after change bid.

SELECT MIN(date(created_at)) AS Week_started_at,
	count(distinct website_session_id) as session
FROM website_sessions
WHERE utm_source = "gsearch" 
	AND utm_campaign ="nonbrand" 
	AND created_at < "2012-05-12"
GROUP BY 
	YEAR(created_at),
	week(created_at) 
--------------------------------------------------------------------------------------------------

## Bid optimization for paid traffic.
# Examination of conversion rates from sessions to order by device type.

SELECT 
	website_sessions.device_type,
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
	COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id)/ COUNT(DISTINCT website_sessions.website_session_id) AS CONV_rt
FROM website_sessions 
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < "2012-05-11"
	AND utm_source = "gsearch"
    AND utm_campaign = "nonbrand"
GROUP BY 1
--------------------------------------------------------------------------------------------------

# Weekly trend for both desktop and moblie after campaingns up on

SELECT 
	MIN(date(created_at)) AS Week_start_date,
	COUNT(CASE WHEN device_type = "mobile" THEN 1 ELSE NULL END ) AS mob_sessions,
    COUNT(CASE WHEN device_type = "desktop" THEN 1 ELSE NULL END ) AS dtop_sessions
FROM website_sessions
WHERE created_at <"2012-06-09"
	AND created_at > "2012-04-15"
    AND utm_source = "gsearch"
    AND utm_campaign = "nonbrand"
GROUP BY 
	YEAR(created_at),
	WEEK(created_at)
--------------------------------------------------------------------------------------------------

## 5 - Analyzing website performance

# Analyzing top website pages & entry pages.

SELECT 
	pageview_url,
    COUNT(DISTINCT website_pageview_id) AS pvs 
FROM website_pageviews
WHERE website_pageview_id < 1000
GROUP BY pageview_url
ORDER BY pvs DESC
--------------------------------------------------------------------------------------------------

CREATE temporary TABLE first_pageview
SELECT 
	website_session_id,
    MIN(website_pageview_id) AS min_pv_id
FROM website_pageviews
WHERE website_pageview_id < 1000
GROUP BY website_session_id
--->
SELECT 
	website_pageview_id.pageview_url AS Landing_page,
    COUNT(DISTINCT first_pageview,website_session_id) AS sessions_hitting_this_lander
FROM first_pageview 
	LEFT JOIN website_pageviews
		ON first_pageview.min_pv_id = website_pageviews.website_pageview_id
GROUP BY 
	website_pageviews.pageview_url
--------------------------------------------------------------------------------------------------

# Finding top website pages.
SELECT 
	pageview_url,
    COUNT(DISTINCT website_pageview_id) AS num_of_views
FROM website_pageviews
WHERE created_at < "2012-06-09"
GROUP BY 1
ORDER BY 2 DESC
--------------------------------------------------------------------------------------------------

## Finding top entry pages
-- STEP 1: find the first pageview for each session
-- STEP 2: find the url the customer saw on that first pageview

CREATE temporary table first_page_v_per_session
SELECT 
	website_session_id,
    min(website_pageview_id) AS first_pv
FROM website_pageviews
WHERE created_at < "2012-06-12"
GROUP BY 1 

SELECT 
	W.pageview_url AS landing_page_url,
    COUNT(DISTINCT F.website_session_id) AS sessions_hitting_page
FROM first_page_v_per_session F
	LEFT JOIN website_pageviews W
		ON F.first_pv = W.website_pageview_id
GROUP BY W.pageview_url
--------------------------------------------------------------------------------------------------


## Calculating bounce rates 

-- STEP 1: finding the first website_pageview_id for relevant sessions
-- STEP 2: identifying the landing page of each session
-- STEP 3: counting pageviews for each sessions, to identify bounces 
-- STEP 4: summarizing  by counting total sessions and bounced sessions 

create temporary table first_pageviews
SELECT
	website_session_id,
    min(website_pageview_id) AS min_pageview_id
FROM website_pageviews
WHERE created_at < "2012-06-01"
GROUP BY 1

create temporary table sessions_w_home_landong_page
SELECT 
	f.website_session_id,
    w.pageview_url AS landing_page
FROM first_pageviews f
	LEFT JOIN website_pageviews w
		ON w.website_pageview_id = f.min_pageview_id
WHERE w.pageview_url = "/home"

--------------------------------------------------------------------------------------------------



