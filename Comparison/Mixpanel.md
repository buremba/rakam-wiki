## Rakam vs Mixpanel

Mixpanel is one of the first and probably the leading event analytics solution in the SaaS market. 
They collect data with their SDKs from your clients and allow you analyze your data with a user interface. 
We also have the similar flow for data ingestion, you can use our SDKs or API directly in order the send the event data 
and similar to Mixpanel, we provide segmentation (event explorer), funnel, retention, explore (user attribution) and insight (user drilldown).
The main difference between Rakam and Mixpanel is we provide PaaS instead or SaaS but since we do have similar features,  

### Differences

Mixpanel provide you a set of features to analyze your data, we handle the ETL and data warehouse for you and allow you to run any kind of complex analytic queries on your event dataset. 
On top of that, we have behavioral analytics APIs such as funnel and retention and do not enforce any limit for you to run your custom queries.
Rakam is also designed to handle any kind of event data and provide many more features such as webhooks and scheduled tasks 
so it's more like a data hub for your company. 
For example, we have customers that uses both Mixpanel and Rakam, 
they export metrics to Rakam periodically with scheduled tasks and integrate 
Facebook Ads and other third party services such as Intercom.io with webhooks and be able to see the big picture of their customer data. 

### SQL vs JQL

Recently, Mixpanel released a feature that allows you to run your custom queries with Javascript and called the feature JQL. They have some arguments about using SQL and prefer Javascript as query language. Since most of their customers have small or medium data volume size and their data is unstructured, it may work just fine for them but for larger data-sets, we argue that SQL is still the best solution. We even have a blog post about [JQL and SQL in the wild](https://blog.rakam.io/why-sql-superior-for-analytic-queries-comparison-with-mixpanels-jql-ec9935f292bd).


### Price and data ownership
Mixpanel uses their in-house database to store the event data. AFAIK, it's JSON based and in-memory. Since JSON occupies much more space than columnar storage types and memory is expensive, their pricing is expensive even for other SaaS services. We try to optimize the way we store the data, infer the schema from the data you sent, uses columnar storage formats for better compression and efficiency. The data is on disk and will be loaded in memory only when you need it, you can scale the Rakam cluster when you need to run complex queries on billions of events and scale down when you're done. You can also use materialized views and continuous queries for incremental calculation so that you don't need processing billions of data every time you execute a query. Our pricing model is different than Mixpanel since we're a PaaS company but it's way cheaper than it if you have >500k events monthly.

You can install Rakam on your AWS account and own the data you have, even install it on your on-premise servers in your local datacenter. Mixpanel is a third-party service, you give the data you have and let it to process for you. However, luckily Mixpanel allows you to export your raw events to other services such as Rakam.

### ETL
AFAIK, Mixpanel doesn't do ETL, it stores the events you sent. Instead, we have [event mapper](/doc/buremba/rakam-wiki/master/Event-Mappers) mechanism which allow you to enrich and sanitize the events you want to collect. We have native event mappers and also give you the ability to implement new event mappers either natively in java or dynamically in javascript with custom event mappers.

### BI (Business intelligence)


### When to use Mixpanel or Rakam
We do not have the ability to send push notifications and predication feature of Mixpanel yet 
but these features can be implemented as user action and service in Rakam. On the other hand, Rakam provides

### Other analytics services
Most of the SaaS analytics services are similar to Mixpanel, they do differnciate 
