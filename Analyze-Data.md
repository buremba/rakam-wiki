Rakam API provides both high and low level APIs for analyzing your event data. 
High level APIs are similar to APIs provided by analytics services and usually enough for simple use-cases.
Currently we provide Funnel, Retention, Event Explorer modules as high-level APIs and you can also run complex queries on your data-set with SQL Query API.

# Event Explorer API

Event Explorer API provides simple metrics such as the event occurences between specific time periods and allow you to query the data data-warehouse by simply spesifying measure, dimension, segmentation and filters instead of writing big complex SQL queries. You can find the API documentation of Event Explorer API [here](https://api.rakam.io/#event-explorer), if you're also using Rakam BI, you can use the API with [a nice UI](https://app.rakam.io/event-explorer). 

# Event Stream API

Event Stream API allows you to subscribe the events collected in real-time. It's useful for inspecting individual events that are occurred in real-time. You can basically set a filter expression and only see the events you want to inspect. You can find the API documentation of Event Stream API [here](https://api.rakam.io/#event-stream), if you're also using Rakam BI, [here is the page for you](https://app.rakam.io/stream).

# Funnel API

Funnel API allows you to create funnels by analyzing your event data. 
In order to be able to it, the collected events must have attached _user and _time properties.
It basically analyze the events of individual users based on your query and return the funnel results. 
You can run funnel queries directly with [Rakam API](https://api.rakam.io/#funnel) or [funnel page in Rakam BI](https://app.rakam.io/funnel).

# Retention API

Retention API is used for creating cohort analysis reports of your users. Similar to Funnel API, it also uses _user and _time properties. You can run retention queries directly with [Rakam API](https://api.rakam.io/#retention) or [retention page in Rakam BI](https://app.rakam.io/retention) if you want to see cohort charts. 

# Real-time API

Real-time API basically provides you a simple API for you to create continuous queries and perform queries on them. You can create a real-time table by simply specifying the columns that you want to measure or dimension, Later on you can perform real-time queries on created continuous table such as the number of online users, their locations etc and real-time dashboards similar to [Chartbeat](https://chartbeat.com). You can find the API documention [here](https://api.rakam.io/#realtime) or the webpage for this feature in Rakam BI from [here](https://app.rakam.io/real-time).

# User API

If you activate user module, you can also analyze your users by their attributes or events with User API, get events for an individual user or perform action (send email to a set of users etc.) with User API. You can find the API documention [here](https://api.rakam.io/#user) or the webpage for this feature in Rakam BI from [here](https://app.rakam.io/people).

# SQL Query API (low-level API)

SQL Query API allows you to run complex queries on you event data-set. The syntax is based on [Presto](https://prestodb.io)'s ANSI SQL, when you send a query, we parse it and re-write it for backend database. Currently, we provide 5 schemas that can be used for different purposes, you can reference tables from different schemas in same SQL query, join them and materialize or continuously calculate the result if you need.

The SQL Query API allows you export the result as CSV or JSON and sample the referenced tables automatically if you need low latency response. You can find the API documentation [here](https://api.rakam.io/#execute-query-on-event-data-set), you also run SQL queries and create reports in [playground page](https://app.rakam.io/report-playground) of Rakam BI. Note that API uses text/event-stream content type, it will stream the query stats (percentage, processed rows and bytes etc.) in real-time.

You can find the documentation of SQL syntax in Presto's website: [https://prestodb.io/docs/current/](https://prestodb.io/docs/current/)

## Schemas

### Collections (raw data) (collection.*)
*collection* schema can be used for accessing raw event data. When you send an event via collection API, Rakam will parse it, create the schema from event properties and insert it to a physical table.

Ex. 
```sql 
SELECT _device_type, count(*) as count from collection.pageview GROUP BY _device_type
```

### Materialized Tables (materialized.*)
*materialized* schema is used for accessing created materialized tables. They're similar to materialized table concept in most of the RDBMSs but also provide incremental materialization and update interval features. When you reference a materialized table, Rakam checks the last updated time for the materialized table and update it if it's not fresh. Materialized tables are quite useful analytics services, you can execute and materialize the result of complex SQL queries periodically or lazily and serve them in dashboard with low-latency. You can create materialized views via Rakam API [(API Documentation)](https://api.rakam.io/#materialized-view) or using report playground webpage in Rakam BI.

Ex. 
```sql 
SELECT count from materialized.device_type_views WHERE _device_type = 'Android'
```

### Continuous Queries (continuous.*)
*continuous* schema is used for continuous queries. Similar to materialized tables, you can create continuous queries via Rakam API [(API Documentation)](https://api.rakam.io/#continuous-query) or using report playground webpage in Rakam BI. The main difference between continuous queries and materialized tables is that continuous queries are real-time and mutable. The continuous queries must be aggregation queries, Rakam will update the aggregation state in near real-time using our in-house stream processing engine.

Let's say that we need to create funnel and retention charts of users who did pageview event at a given day.

1. Create the continuous query using Rakam API. Once you create the continuous query, a set will be created for each 

```bash

curl [RAKAM_API]/continuous-query/create --body '{"tableName": "daily_users_who_viewed_page", "query": "SELECT cast(_time as date) day, set(_user) as user_set FROM pageviews GROUP BY day"}'

```

2. Create the retention chart from pre-computed values with the following query. It will merge the user sets from _2016-10-10_ to the next 30 days and find the intersection of sets in order to find out whether the user did pageview event after X days from _2016-10-10_:

```sql
SELECT cardinality(first.user_set), cardinality_intersection(first._user_set, second._user_set) FROM continuous.daily_users_who_viewed_page first JOIN continuous.daily_users_who_viewed_page second ON (first.day + INTERVAL '30' day < second.day) WHERE first.day = date '2016-10-10'
```

### Remote data-sources

External data-sources can be used for joining your external databases and files with your event data-set. You can plug your external databases into Rakam as a schema and also add your remote _CSV_ or _AVRO_ files as tables. When you reference the schema or table in a SQL query, the database engine will execute the query on your database or fetch the remote file and use the data in query. You can also materialize the remote tables by referencing them in your materialized tables if you want to cache them to reduce the load on your database servers.

The external databases use a schema identifier specified by you (mysqldb), and the remote files use remotefile schema and the identifier as table name that is set by you.

ex. 
```sql 
SELECT count(*) FROM mysqldb.users JOIN remotefile.my_csv_file ON (my_csv_file.user_id = users.id)
```

#### Special tables

If you don't set a schema (i.e. ```sql SELECT count(*) from pageview```), Rakam will use collection schema by default. The only exception is __all_ table and it's an alias for *UNION ALL* of all tables. When you execute a query similar to ```sql SELECT cast(_time as date), count(*) FROM _all GROUP BY 1```, Rakam will union all tables and expose __time_, __user_ and _properties_ columns.
