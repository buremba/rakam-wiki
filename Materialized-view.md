The materialized views allows you to create summary tables from your raw data so that you can efficiently pre-compute your data and power your dashboards. 
The feature is also based on SQL and, you can basically process your raw data with SQL and materialize the result as table.

Basically we have three different strategy materialized views; `cache`, `incremental` and `real-time incremental`.

# Strategies

### Cache 
When you set `cache` strategy along with table update interval, your table will be updated whenever the cache is expired. 
Let's say that you used the following query and saved a materialized view with name `weekly_pageviews` and 24 hours table update interval:

```
SELECT date_trunc('week', _time), count(*) from pageview
```

Then you can run the query `SELECT * from materialized.weekly_pageviews` the first time, we will create a physical table and insert to query result to it with 
`CREATE TABLE materialized_table_weekly_pageviews AS SELECT SELECT date_trunc('week', _time), count(*) from pageview` and re-write your query as `SELECT * from materialized_table_weekly_pageviews`.
When you run the same query again, we will directly execute `SELECT * from materialized_table_weekly_pageviews` so you won't process the raw data again for the next 24 hours. When you run the same query next day, 
we will truncate the table and INSERT the refreshed data again.

### Incremental

Incremental materialiazed views are one of our core technolojy. We incrementally process your raw-data and materialize the result so that you can create OLAP cubes, pre-process your data and power your dashboards. 
Let's say that you saved a materialized view with name `weekly_pageviews` and 24 hours table update interval:

```
SELECT date_trunc('week', _time), count(*) from pageview
```

When you reference the `weekly_pageviews` materalized view for the first time, similarly the cache strategy we process the historical data and materialize it with the following query: 
`CREATE TABLE materialized_table_weekly_pageviews AS SELECT SELECT date_trunc('week', _time), count(*) from pageview`. The main difference between `cache` and `incremental` strategy is that 
instead of truncating the data when the cache is expired, we insert the recent data to the same table using the following query:

```
INSERT INTO weekly_pageviews SELECT * from materialized_table_weekly_pageviews SELECT date_trunc('week', _time), count(*) from pageview WHERE shard_time between *materialized_view_last_updated_time* AND *current_timestamp*`
```

`materialized_view_last_updated_time` column represents the time when the data is processed by our servers so it's always increasing. We don't use `_time` column since the clients can send their `_time` along with the event and they may re-send the events which are not sent when they're occured because of network issues or batching.

### Real-time Incremental

Real-time incremental strategy is quite similar to incremental strategy but it's real-time. Under to hood, it uses to method that is used for incremental strategy but also UNION ALLs the recent data in order to make your reports realtime. For the same query that we used before,
when you reference a materialized view it runs the following query:

`SELECT * from materialized_table_weekly_pageviews UNION ALL SELECT date_trunc('week', _time), count(*) from pageview WHERE shard_time > *materialized_view_last_updated_time*`
