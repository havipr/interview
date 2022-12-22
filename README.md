# interview

Data science interview questions - with answers

SQL

Suppose we have the following schema with two tables: Ads and Events

Ads(ad_id, campaign_id, status)
status could be active or inactive
Events(event_id, ad_id, source, event_type, date, hour)
event_type could be impression, click, conversion


Write SQL queries to extract the following information:

1) The number of active ads.

SELECT count(*) FROM Ads WHERE status = 'active';


3) The number of active campaigns.

SELECT COUNT(DISTINCT a.campaign_id)
FROM Ads AS a
WHERE a.status = 'active';

4) The number of events per each ad — broken down by event type.



SELECT a.ad_id, e.event_type, count(*) as "count"
FROM Ads AS a
  JOIN Events AS e
      ON a.ad_id = e.ad_id
GROUP BY a.ad_id, e.event_type
ORDER BY a.ad_id, "count" DESC;

5) The number of events over the last week per each active ad — broken down by event type and date (most recent first).



SELECT a.ad_id, e.event_type, e.date, count(*) as "count"
FROM Ads AS a
  JOIN Events AS e
      ON a.ad_id = e.ad_id
WHERE a.status = 'active'
   AND e.date >= DATEADD(week, -1, GETDATE())
GROUP BY a.ad_id, e.event_type, e.date
ORDER BY e.date ASC, "count" DESC;
