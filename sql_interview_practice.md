# Technical interview questions

<table>
   <tr>
      <td>⚠️</td>
      <td>
         The answers here are given by the community. Be careful and double check the answers before using them. <br>
         If you see an error, please create a PR with a fix
      </td>
   </tr>
</table>

The list is based on [this post](https://medium.com/data-science-insider/technical-data-science-interview-questions-f61cd9cf218?source=friends_link&sk=01f4de0de746d28fe714d92a1e91e190)


## Table of contents

* [SQL](#sql)


<br/>

## SQL

Suppose we have the following schema with two tables: Ads and Events

* Ads(ad_id, campaign_id, status)
* status could be active or inactive
* Events(event_id, ad_id, source, event_type, date, hour)
* event_type could be impression, click, conversion

<img src="img/schema.png" />


## Use the following ddl to generate tables for MySQL 5.6 in sqlfiddle


CREATE TABLE IF NOT EXISTS `Ads` (
  `ad_id` int(6) unsigned NOT NULL,
  `campaign_id` int(3) unsigned NOT NULL,
  `status` varchar(200) NOT NULL,
  PRIMARY KEY (`ad_id`)
) DEFAULT CHARSET=utf8;


INSERT INTO `Ads` (`ad_id`, `campaign_id`, `status`) VALUES
  ('1', '10', 'active'),
  ('2', '10', 'active'),
  ('3', '20', 'inactive'),
  ('4', '30', 'active');


CREATE TABLE IF NOT EXISTS `Event` (
  `event_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `date` date DEFAULT NULL,
  `ad_id` int(3) unsigned NOT NULL,
  `source` varchar(200) NOT NULL,
  `event_type` varchar(200) NOT NULL,
    `hour` int(2),
  PRIMARY KEY (`event_id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;


INSERT INTO `Event` (`event_id`, `date`, `ad_id`, `source`, `event_type`,`hour`) VALUES
	(100, '2012-02-27', '1', 'source1', 'impression',   '00'),
	(200, '2012-02-27', '2','source2', 'click','00'),
	(300, '2012-02-27', '3', 'source3','click','00'),
	(400, '2012-02-27', '4', 'source4','click','00'),
	(500, '2012-11-22', '1', 'source1','impression',  '00'),
	(600, '2012-11-22', '2', 'source2','conversion','00'),
	(700, '2012-11-22', '3', 'source3','click','00'),
	(800, '2012-11-22', '4', 'source4', 'impression', '00'),
	(900, '2015-03-10', '1','source1','click', '00'),
	(1000, '2015-03-10', '2','source2', 'conversion','00'),
	(1100, '2015-03-10', '1','source3', 'click', '00'),
	(1200, '2015-03-10', '1','source4','click', '00'),
    (1300, '2015-03-10', '1','source1', 'conversion', '00'),
	(1400, '2015-03-11', '1','source2','conversion', '00'),
    (1500, '2015-03-11', '2','source3', 'conversion', '00'),
	(1600, '2015-03-11', '1','source4','conversion', '00'),
    (1700, '2015-03-11', '3','source1', 'conversion', '00'),
	(1800, '2015-03-11', '4','source2','conversion', '00')
    
    
    ;





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
Select a.ad_id, a.event_type, a.date,count(a.event_id) as count
From event a
Inner join ads b
On a.ad_id = b.ad_id
Where b.status = 'active'
Group by a.ad_id, a.event_type, a.date
Order by a.date asc, count(a.event_id) desc
;
 
```

<br/>

**6)** The number of events per campaign — by event type.

<img src="img/sql_6_example.png" />


```sql
Select a.campaign_id,b.event_type, count(*) as count
From ads a
Inner join event b
On a.ad_id = b.ad_id
Group by a.campaign_id,b.event_type
Order by a.campaign_id
;

```

<br/>

**7)** The number of events over the last week per each campaign — broken down by date (most recent first).

<img src="img/sql_7_example.png" />

```sql
-- for Postgres

Select a.campaign_id, b.event_type, b.date,count(*) as count
From ads a
Inner join event b
On a.ad_id = b.ad_id
Group by a.campaign_id, b.event_type,date
Order by date asc, count(*)  desc ;

```

<br/>

**8)** CTR (click-through rate) for each ad. CTR = number of impressions / number of clicks.

<img src="img/sql_8_example.png" />

```sql
Select ad_id,   concat(format(CTR*100,2)," %") as ctr
from
 (
              Select a.ad_id,
              (
              sum( case when b.event_type = 'impression' then 1 else 0 end)/
              sum(case when b.event_type = 'click' then 1 else 0 end) 
              ) as CTR
              From ads a
              Inner join event b
              On a.ad_id = b.ad_id
              Group by a.ad_id
) as T;

```

<br/>

**9)** CVR (conversion rate) for each ad. CVR = number of clicks / number of installs.

<img src="img/sql_9_example.png" />

```sql
Select ad_id,   concat(format(CVR*100,2)," %") as cvr
from
 (
              Select a.ad_id,
              (
              sum( case when b.event_type = 'click' then 1 else 0 end)/
              sum(case when b.event_type = 'conversion' then 1 else 0 end) 
              ) as CVR
              From ads a
              Inner join event b
              On a.ad_id = b.ad_id
              Group by a.ad_id
) as T;

```

<br/>

**10)** CTR and CVR for each ad broken down by day and hour (most recent first).

<img src="img/sql_10_example.png" />

```
select ad_id,date,hour,concat(format(CTR*100,2)," %") as CTR, concat(format(CVR*100,2)," %") as CVR

FROM 
(

select a.ad_id,b.date,b.hour,
(
              sum( case when b.event_type = 'impression' then 1 else 0 end)/
              sum(case when b.event_type = 'click' then 1 else 0 end) 
              ) as CTR
,
  (
              sum( case when b.event_type = 'click' then 1 else 0 end)/
              sum(case when b.event_type = 'conversion' then 1 else 0 end) 
              ) as CVR
from ads a
inner join event b
on a.ad_id = b.ad_id
group by a.ad_id,b.date,b.hour
order by b.date,b.hour
)
AS T;
```

<br/>

**11)** CTR for each ad broken down by source and day

<img src="img/sql_11_example.png" />

```
select ad_id,source,date,concat(format(CTR*100,2)," %") as CTR
FROM 
(

select a.ad_id,b.source,b.date,
(
              sum( case when b.event_type = 'impression' then 1 else 0 end)/
              sum(case when b.event_type = 'click' then 1 else 0 end) 
              ) as CTR

from ads a
inner join event b
on a.ad_id = b.ad_id
group by a.ad_id,b.date,b.hour
order by b.source,b.date
)
AS T;

```

<br/>

<br/>
