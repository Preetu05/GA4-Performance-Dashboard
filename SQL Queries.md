**Total Event:**

SELECT

event\_name,

COUNT(\*) AS total\_events

FROM `datacareerapp.analytics\_475926274.events\_\*`

WHERE \_TABLE\_SUFFIX >= '20260215'

GROUP BY event\_name

ORDER BY total\_events DESC



**Total User:**

SELECT

COUNT(DISTINCT user\_pseudo\_id) AS total\_users

FROM `datacareerapp.analytics\_475926274.events\_\*`

WHERE \_TABLE\_SUFFIX >= '20260215'



**New User:**

SELECT

COUNT(DISTINCT user\_pseudo\_id) AS new\_users

FROM `datacareerapp.analytics\_475926274.events\_\*`

WHERE event\_name = 'first\_visit'

AND \_TABLE\_SUFFIX >= '20260215'



**Total Session:**

SELECT

&#x20; COUNT(\*) AS total\_sessions

FROM `datacareerapp.analytics\_475926274.events\_\*`

WHERE event\_name = 'session\_start'

&#x20; AND \_TABLE\_SUFFIX >= '20260215'



**Engagement Rate:**

SELECT

CASE WHEN COUNTIF(event\_name = 'page\_view') = 0 THEN 0

ELSE COUNTIF(event\_name IN ('scroll','click','user\_engagement')) / COUNTIF(event\_name = 'page\_view')

END AS engagement\_rate

FROM `datacareerapp.analytics\_475926274.events\_\*`

WHERE \_TABLE\_SUFFIX >= '20260215'



First Seen:

SELECT user\_pseudo\_id, MIN(event\_date) AS first\_seen\_date

FROM `datacareerapp.analytics\_475926274.events\_\*`

WHERE \_TABLE\_SUFFIX >= '20260215'

GROUP BY user\_pseudo\_id



**Cohort Analysis:**

WITH

&#x20; first\_seen AS (

&#x20;   SELECT

&#x20;     user\_pseudo\_id,

&#x20;     MIN(PARSE\_DATE('%Y%m%d', \_TABLE\_SUFFIX)) AS first\_seen\_date

&#x20;   FROM `datacareerapp.analytics\_475926274.events\_\*`

&#x20;   WHERE \_TABLE\_SUFFIX >= '20260215'

&#x20;   GROUP BY user\_pseudo\_id

&#x20; )

SELECT

&#x20; user\_pseudo\_id,

&#x20; first\_seen\_date,

&#x20; PARSE\_DATE('%Y%m%d', \_TABLE\_SUFFIX) AS event\_date,

&#x20; DATE\_DIFF(PARSE\_DATE('%Y%m%d', \_TABLE\_SUFFIX), first\_seen\_date, WEEK)

&#x20;   AS week\_number

FROM `datacareerapp.analytics\_475926274.events\_\*`

JOIN first\_seen

&#x20; USING (user\_pseudo\_id)

WHERE \_TABLE\_SUFFIX >= '20260215'

ORDER BY first\_seen\_date, week\_number



**Events:**

SELECT

&#x20; -- Core

&#x20; event\_date,

&#x20; event\_timestamp,

&#x20; event\_name,

&#x20; user\_pseudo\_id,



&#x20; -- GA4 Session ID

&#x20; (SELECT value.int\_value

&#x20;  FROM UNNEST(event\_params)

&#x20;  WHERE key = "ga\_session\_id") AS ga\_session\_id,



&#x20; -- Engagement flags (NULL → 0)

&#x20; COALESCE(

&#x20;   (SELECT value.int\_value

&#x20;    FROM UNNEST(event\_params)

&#x20;    WHERE key = "session\_engaged"), 0

&#x20; ) AS session\_engaged,



&#x20; COALESCE(

&#x20;   (SELECT value.int\_value

&#x20;    FROM UNNEST(event\_params)

&#x20;    WHERE key = "engaged\_session\_event"), 0

&#x20; ) AS engaged\_session\_event,



&#x20; -- Engagement metrics (NULL → 0)

&#x20; COALESCE(

&#x20;   (SELECT value.int\_value

&#x20;    FROM UNNEST(event\_params)

&#x20;    WHERE key = "engagement\_time\_msec"), 0

&#x20; ) AS engagement\_time\_msec,



&#x20; COALESCE(

&#x20;   (SELECT value.int\_value

&#x20;    FROM UNNEST(event\_params)

&#x20;    WHERE key = "percent\_scrolled"), 0

&#x20; ) AS percent\_scrolled,



&#x20; -- Page fields

&#x20; (SELECT value.string\_value

&#x20;  FROM UNNEST(event\_params)

&#x20;  WHERE key = "page\_title") AS page\_title,



&#x20; (SELECT value.string\_value

&#x20;  FROM UNNEST(event\_params)

&#x20;  WHERE key = "page\_location") AS page\_location,



&#x20; (SELECT value.string\_value

&#x20;  FROM UNNEST(event\_params)

&#x20;  WHERE key = "page\_referrer") AS page\_referrer,



&#x20; -- Traffic source (session-level)

&#x20; traffic\_source.source AS session\_source,

&#x20; traffic\_source.medium AS session\_medium,

&#x20; traffic\_source.name AS channel\_group,



&#x20; -- Device

&#x20; device.category AS device\_category,

&#x20; device.operating\_system AS os,

&#x20; device.web\_info.browser AS browser,



&#x20; -- Geo

&#x20; geo.country AS country,

&#x20; geo.city AS city,

&#x20; geo.region AS region



FROM

&#x20; `datacareerapp.analytics\_475926274.events\_\*`



WHERE

&#x20; \_TABLE\_SUFFIX >= '20260215'



**Events frequency / user cohort:**

WITH first\_seen AS (

&#x20; SELECT

&#x20;   user\_pseudo\_id,

&#x20;   DATE\_TRUNC(MIN(PARSE\_DATE('%Y%m%d', \_TABLE\_SUFFIX)), WEEK(MONDAY)) AS first\_seen\_date

&#x20; FROM `datacareerapp.analytics\_475926274.events\_\*`

&#x20; WHERE \_TABLE\_SUFFIX >= '20260215'

&#x20; GROUP BY user\_pseudo\_id

)



SELECT

&#x20; first\_seen\_date,

&#x20; DATE\_DIFF(PARSE\_DATE('%Y%m%d', \_TABLE\_SUFFIX), first\_seen\_date, WEEK) AS weeks\_since\_cohort,

&#x20; COUNT(\*) / COUNT(DISTINCT user\_pseudo\_id) AS avg\_events\_per\_user

FROM `datacareerapp.analytics\_475926274.events\_\*`

JOIN first\_seen USING (user\_pseudo\_id)

WHERE \_TABLE\_SUFFIX >= '20260215'

GROUP BY first\_seen\_date, weeks\_since\_cohort

ORDER BY first\_seen\_date, weeks\_since\_cohort





