
## Table of contents

* [SQL](#sql)
* [Coding (Python)](#coding-python)
* [Algorithmic Questions](#algorithmic-questions)

<br/>

## SQL

Suppose we have the following schema with two tables: Ads and Events

* Ads(ad_id, campaign_id, status)
* status could be active or inactive
* Events(event_id, ad_id, source, event_type, date, hour)
* event_type could be impression, click, conversion

<img src="img/schema.png" />


Write SQL queries to extract the following information:

**1)** The number of active ads.

```sql
SELECT count(*) FROM Ads WHERE status = 'active';
```

<br/>


**2)** All active campaigns. A campaign is active if there’s at least one active ad.

```sql
SELECT DISTINCT a.campaign_id
FROM Ads AS a
WHERE a.status = 'active';
```

<br/>

**3)** The number of active campaigns.

```sql
SELECT COUNT(DISTINCT a.campaign_id)
FROM Ads AS a
WHERE a.status = 'active';
```

<br/>

**4)** The number of events per each ad — broken down by event type.

<img src="img/sql_4_example.png" />

```sql
SELECT a.ad_id, e.event_type, count(*) as "count"
FROM Ads AS a
  JOIN Events AS e
      ON a.ad_id = e.ad_id
GROUP BY a.ad_id, e.event_type
ORDER BY a.ad_id, "count" DESC;
```

<br/>

**5)** The number of events over the last week per each active ad — broken down by event type and date (most recent first).

<img src="img/sql_5_example.png" />

```sql
SELECT a.ad_id, e.event_type, e.date, count(*) as "count"
FROM Ads AS a
  JOIN Events AS e
      ON a.ad_id = e.ad_id
WHERE a.status = 'active'
   AND e.date >= DATEADD(week, -1, GETDATE())
GROUP BY a.ad_id, e.event_type, e.date
ORDER BY e.date ASC, "count" DESC;
```

<br/>

**6)** The number of events per campaign — by event type.

<img src="img/sql_6_example.png" />


```sql
SELECT a.campaign_id, e.event_type, count(*) as count
FROM Ads AS a
  INNER JOIN Events AS e
    ON a.ad_id = e.ad_id
GROUP BY a.campaign_id, e.event_type
ORDER BY a.campaign_id, "count" DESC
```

<br/>

**7)** The number of events over the last week per each campaign and event type — broken down by date (most recent first).

<img src="img/sql_7_example.png" />

```sql
-- for Postgres

SELECT a.campaign_id, e.event_type, e.date, count(*)
FROM Ads AS a
  INNER JOIN Events AS e
    ON a.ad_id = e.ad_id
WHERE e.date >= DATEADD(week, -1, GETDATE())
GROUP BY a.campaign_id, e.event_type, e.date
ORDER BY a.campaign_id, e.date DESC, "count" DESC;
```

<br/>

**8)** CTR (click-through rate) for each ad. CTR = number of clicks / number of impressions.

<img src="img/sql_8_example.png" />

```sql
-- for Postgres

SELECT impressions_clicks_table.ad_id,
       (impressions_clicks_table.clicks * 100 / impressions_clicks_table.impressions)::FLOAT || '%' AS CTR
FROM
  (
  SELECT a.ad_id,
         SUM(CASE e.event_type WHEN 'impression' THEN 1 ELSE 0 END) impressions,
         SUM(CASE e.event_type WHEN 'click' THEN 1 ELSE 0 END) clicks
  FROM Ads AS a
    INNER JOIN Events AS e
      ON a.ad_id = e.ad_id
  GROUP BY a.ad_id
  ) AS impressions_clicks_table
ORDER BY impressions_clicks_table.ad_id;
```

<br/>

**9)** CVR (conversion rate) for each ad. CVR = number of conversions / number of clicks.

<img src="img/sql_9_example.png" />

```sql
-- for Postgres

SELECT conversions_clicks_table.ad_id,
       (conversions_clicks_table.conversions * 100 / conversions_clicks_table.clicks)::FLOAT || '%' AS CVR
FROM
  (
  SELECT a.ad_id,
         SUM(CASE e.event_type WHEN 'conversion' THEN 1 ELSE 0 END) conversions,
         SUM(CASE e.event_type WHEN 'click' THEN 1 ELSE 0 END) clicks
  FROM Ads AS a
    INNER JOIN Events AS e
      ON a.ad_id = e.ad_id
  GROUP BY a.ad_id
  ) AS conversions_clicks_table
ORDER BY conversions_clicks_table.ad_id;
```

<br/>

**10)** CTR and CVR for each ad broken down by day and hour (most recent first).

<img src="img/sql_10_example.png" />


```sql
-- for Postgres

SELECT conversions_clicks_table.ad_id,
       conversions_clicks_table.date,
       conversions_clicks_table.hour,
       (impressions_clicks_table.clicks * 100 / impressions_clicks_table.impressions)::FLOAT || '%' AS CTR,
       (conversions_clicks_table.conversions * 100 / conversions_clicks_table.clicks)::FLOAT || '%' AS CVR
FROM
  (
  SELECT a.ad_id, e.date, e.hour,
         SUM(CASE e.event_type WHEN 'conversion' THEN 1 ELSE 0 END) conversions,
         SUM(CASE e.event_type WHEN 'click' THEN 1 ELSE 0 END) clicks,
         SUM(CASE e.event_type WHEN 'impression' THEN 1 ELSE 0 END) impressions
  FROM Ads AS a
    INNER JOIN Events AS e
      ON a.ad_id = e.ad_id
  GROUP BY a.ad_id, e.date, e.hour
  ) AS conversions_clicks_table
ORDER BY conversions_clicks_table.ad_id, conversions_clicks_table.date DESC, conversions_clicks_table.hour DESC, "CTR" DESC, "CVR" DESC;
```

<br/>

**11)** CTR for each ad broken down by source and day

<img src="img/sql_11_example.png" />


```sql
-- for Postgres

SELECT conversions_clicks_table.ad_id,
       conversions_clicks_table.date,
       conversions_clicks_table.source,
       (impressions_clicks_table.clicks * 100 / impressions_clicks_table.impressions)::FLOAT || '%' AS CTR
FROM
  (
  SELECT a.ad_id, e.date, e.source,
         SUM(CASE e.event_type WHEN 'click' THEN 1 ELSE 0 END) clicks,
         SUM(CASE e.event_type WHEN 'impression' THEN 1 ELSE 0 END) impressions
  FROM Ads AS a
    INNER JOIN Events AS e
      ON a.ad_id = e.ad_id
  GROUP BY a.ad_id, e.date, e.source
  ) AS conversions_clicks_table
ORDER BY conversions_clicks_table.ad_id, conversions_clicks_table.date DESC, conversions_clicks_table.source, "CTR" DESC;
```

<br/>
