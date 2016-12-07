## Rakam vs Mixpanel

Mixpanel is one of the first and probably the leading event analytics solution in the SaaS market. 
They collect data with their SDKs from your clients and allow you analyze your data with a simple user interface. 
We also have the similar flow for data ingestion, you can use our SDKs or API directly in order the send the event data 
and similar to Mixpanel, we provide segmentation (event explorer), funnel, retention, explore (user attribution) and insight (user drilldown) features.
The main difference between Rakam and Mixpanel is we provide PaaS instead or SaaS.

### Feature Differences

Mixpanel stores the data as you sent in their data warehouse and provides you a set of features to analyze your data but we enrich your data, sanitize it depending on what you need and store it in your data warehouse for you and allow you to run both high level features similar to Mixpanel and also any kind of complex analytic queries on your event dataset. 

You have the full control over how the data is stored, enriched and transformed, you can even write custom modules and add them to Rakam.

Rakam is also designed to handle any kind of event data and provide many more features such as webhooks and scheduled tasks so it's more like a data hub for your company. 
For example, we have customers that uses both Mixpanel and Rakam, 
they export metrics to Rakam periodically with scheduled tasks and integrate 
Facebook Ads and other third party services such as Intercom.io with webhooks and be able to see the big picture of their customer data. 

### SQL vs JQL

Recently, Mixpanel released a feature that allows you to run your custom queries with Javascript and called the feature JQL. They have some arguments about using SQL and prefer Javascript as query language. Since most of their customers have small or medium data volume size and their data is unstructured, it may work just fine for them but for larger data-sets, we think that SQL is still the best solution for analytics queries. We even have a blog post about [JQL and SQL in the wild](https://blog.rakam.io/why-sql-superior-for-analytic-queries-comparison-with-mixpanels-jql-ec9935f292bd).


### Price and data ownership
Mixpanel uses their in-house database to store the event data. AFAIK, it's JSON based and in-memory. Since JSON occupies much more space than columnar storage types and memory is expensive, their price is expensive even for other SaaS services. We try to optimize the way we store the data, infer the schema from the data you sent, use columnar storage formats for better compression and efficiency. The data is on disk and will be loaded in memory only when you need it, you can scale the Rakam cluster when you need to run complex queries on billions of events and scale down when you're done. You can also use materialized views and continuous queries for incremental calculation so that you don't need processing billions of data every time you execute a query. Our pricing model is different than Mixpanel since we're a PaaS company but it's way cheaper than it if you have >500k events monthly.

You can install Rakam on your AWS account and own the data you have, even install it on your on-premise servers in your local datacenter. Mixpanel is a third-party service, you give the data you have and let it to process for you. However, luckily Mixpanel allows you to export your raw events to other services such as Rakam.

### ETL
AFAIK, Mixpanel doesn't do ETL, it stores the events you sent. Instead, we have [event mapper](/doc/buremba/rakam-wiki/master/Event-Mappers) mechanism which allow you to enrich and sanitize the events you want to collect. We have native event mappers and also give you the ability to implement new event mappers either natively in java or dynamically in javascript with custom event mappers. For example, you can enrich your events by looking up user email from external services such as Clearbit or your database, you have the full control in ETL process.

### BI (Business intelligence)
Mixpanel doesn't let you to visualize the data returned from your custom query in a chart that you want, it has pre-defined reports, allow you to change the parameters and it automatically updates the charts depending on the parameters you selected. Rakam UI is designed to create custom reports and custom dashboards similar to other BI tools. You can write SQL queries, parametrize and save them as reports. The analysts don't need to see the underlying SQL query, they will use Rakam as if they're using Google Analytics. From this persfective, Mixpanel is a generic event analytics solution whereas Rakam can be specialized just for you.

### When to use Mixpanel or Rakam
We do not have the ability to send push notifications and predication feature of Mixpanel yet 
but these features can be implemented as user action and service in Rakam in the future if needed. On the other hand, you have the full control over your data with Rakam, you can run ad-hoc queries and visualize them as you wish. If your data volume is not big, the price is affordable and the features provided by Mixpanel is enough for you, Mixpanel might be good alternative. However; if you want to look out an advanced solution that is affordable for larger data volumes, flexible enough to create custom reports and dashboards, able to collect data from multiple sources and integrate it with your third party services and databases you may want to try Rakam.

### Other analytics services
Most of the SaaS analytics services are similar to Mixpanel, most of the features are the same and they all add some other features in order to differentiate from each other. However; that's how SaaS market works, you find a problem and fix it from your perspective. The thing is that when you use multiple analytics services and cannot integrate each other, you won't be able to see the big picture, that's why companies usually switch to ETL + data warehouse + BI solutions. Unfortunately, the data warehouse market is not similar to SaaS market, it needs too much developer effort, time and has risks if the team is not experienced. Rakam tries to find a solution to this problem, you will have full-stack ETL + data warehouse + BI as if you're using a SaaS service similar to Mixpanel with an affordable price.
