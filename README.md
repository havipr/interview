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

