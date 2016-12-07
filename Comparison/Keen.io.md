## Rakam vs Keen.io

Keen is also a custom analytics product similar to Rakam but is a SaaS service unlike Rakam. 
You send data to Keen and it provides you an API for you to analyze your data. 
It has enrichment methods similar to Rakam but they're pre-defined so you won't be able to implement your custom enrichment methods.
They store the data on their in-house data warehouse built with Storm and Cassandra and implemented their custom query language on top of it.
The problem is that they use JSON based query language and it has limitations compared to SQL.
We're fan of SQL, you can find out our thoughts about using other query languages for analytics data here:
[SQL is still superior for big-data analytics](https://blog.rakam.io/why-sql-superior-for-analytic-queries-comparison-with-mixpanels-jql-ec9935f292bd)

Their query language is similar to 

> SELECT dimension, agg_function(metric) FROM collection GROUP BY dimension ORDER BY 2 DESC,

we already have the same functionality in our Event Explorer module, you select the dimensions, metrics, ordering and get the data 
without actually writing SQL query.

They do not have an integrated BI tool unline Rakam UI, instead have an open-source project called [Keen Dashboard](http://keen.github.io/dashboards/). 
They provide tutorials about how to create dashboards with the open-source project, you should create your own solution on top of Keen.io.

Keen is a third-party service and is not integrated with your custom data sources or services,
you need to send data manually to Keen.io. You do not own the data you sent, you should export the data from Keen and use a 
data-warehouse solution in order to run complex queries.

In fact, even though both Rakam and Keen.io are custom analytics solutions, our target customer base is quite different. 
We aim to provide full-stack solution and give you the full control and 
AFAIK Keen is mostly used by companies to provide their customers simple statistics with a third party service.
