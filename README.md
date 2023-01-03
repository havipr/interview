
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

<br/>


## Coding (Python)

**1) FizzBuzz.** Print numbers from 1 to 100

* If it’s a multiplier of 3, print “Fizz”
* If it’s a multiplier of 5, print “Buzz”
* If both 3 and 5 — “Fizz Buzz"
* Otherwise, print the number itself

Example of output: 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, Fizz Buzz, 16, 17, Fizz, 19, Buzz, Fizz, 22, 23, Fizz, Buzz, 26, Fizz, 28, 29, Fizz Buzz, 31, 32, Fizz, 34, Buzz, Fizz, ...

```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print('Fizz Buzz')
    elif i % 3 == 0:
        print('Fizz')
    elif i % 5 == 0:
        print('Buzz')
    else:
        print(i)
```

<br/>

**2) Factorial**. Calculate a factorial of a number

* `factorial(5)` = 5! = 1 * 2 * 3 * 4 * 5 = 120
* `factorial(10)` = 10! = 1 * 2 * 3 * 4 * 5 * 6 * 7 * 8 * 9 * 10 = 3628800

```python
def factorial(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result
```

We can also write this function using recursion:

```python
def factorial(n: int):
    if n == 0 or n == 1:
        return 1
    else:
        return n * factorial(n - 1) 



**4) STD**. Calculate the standard deviation of elements in a list.

* `std([1, 2, 3, 4]) = 1.29`
* `std([1]) = NaN` (use `float('NaN')`)
* `std([]) = NaN`

<img src="img/formula_std.png" />

```python
from math import sqrt
from statistics import mean

def std_dev(numbers):
    if len(numbers) > 1:
        avg = mean(numbers)
        var = sum((i - avg) ** 2 for i in numbers) / (len(numbers) - 1)
        std = sqrt(var)
        return std
    return float('NaN')
```

<br/>

<br/>

**3) Mean**. Compute the mean of number in a list

* `mean([4, 36, 45, 50, 75]) = 42`
* `mean([]) = NaN` (use `float('NaN')`)

<img src="img/formula_mean.png" />

```python
def mean(numbers):
    if len(numbers) > 0:
        return sum(numbers) / len(numbers)
    return float('NaN')
```

<br/>

**4) STD**. Calculate the standard deviation of elements in a list.

* `std([1, 2, 3, 4]) = 1.29`
* `std([1]) = NaN` (use `float('NaN')`)
* `std([]) = NaN`

<img src="img/formula_std.png" />

```python
from math import sqrt
from statistics import mean

def std_dev(numbers):
    if len(numbers) > 1:
        avg = mean(numbers)
        var = sum((i - avg) ** 2 for i in numbers) / (len(numbers) - 1)
        std = sqrt(var)
        return std
    return float('NaN')
```

<br/>
        
        
