## Rakam vs Snowplow

Snowplow is an open-source analytics platform that can be installed on AWS. 
They handle the event data pipeline, collect data from multiple sources, enrich & sanitize and send the events to your data-warehouse.
In fact, we learned a lot from Snowplow, their workflow is quite nice and flexible but the main problem is it's still [not easy to setup](https://github.com/snowplow/snowplow/wiki/Setting-up-SnowPlow). 
They provide you the tools separately, you need to glue them together and build your own solution on top of Snowplow.

Snowplow can be an alternative to the event pipeline solution that we provided as part of the Rakam API.
We also allow you collect data from multiple sources, enrich and sanitize the events with event mappers 
and store the data in our data-warehouse. Snowplow doesn't have its own data warehouse solution,
instead you choose your database and make them to send the data to it. Currently, they support Redshift, Postgresql and Elasticsearch and we provide two solutions; 
Postgresql and PrestoDB. 

Postgresql is flexible, real-time and fast for small and medium size data volumes but unfortunately not horizontally scalable and the default row-oriented storage format has too much overhead in big-data. 
For larger data volumes, we have PrestoDB deployment type. 

Unfortunately most of the data-warehouse solutions such as PrestoDB and Redshift are not real-time. 
Therefore, we forked PrestoDB and implemented a near real-time data warehouse on top of it. 
Since ad-hoc queries are not enough for dashboards, we also implemented stream processing inside the database 
with continuous queries, just write the SQL query and create the continuous query background job. Rakam will process the data at ingestion time so you don't need to process billions of data when visiting your dashboards.
Since we have full control on our data-warehouse, we also allow you to mount your remote database schemas as virtual tables,
mount your remote files (CSV, AVRO etc.) 
as virtual tables so that you get enough flexibility to create your custom analytics service easily.

The main problem with Redshift is it's fully managed by AWS. You need to load the data to it when you want to analyze and 
even if you usually run queries on small part of the data-set (last month, last day etc.),
Redshift still charges you for all the data you have. With PrestoDB, our customers usually start more nodes when they need to run complex queries
on larger data-sets and terminate them when they don't need. It affects your bills at the end of the month because you will be allocating the real resource 
you consume on the cloud.

Our ETL pipeline is indeed more advanced because we allow you to implement native event mappers add them as modules to Rakam 
and also allow you to implement custom event mappers with Javascript at runtime. For example, you can add Clearbit integration
with just one click (Since we provide boilerplate for it) or implement support for similar third party service with ~15 lines of JS code. 
The Clearbit integration just gets an attribute from the event (user email), perform a HTTP request and attach some values to the event.

Snowplow doesn't provide an user interface for analyzing the data, instead companies plug a third party BI solution 
to their data-warehouse and analyze it there. Instead we have Rakam UI that allows you to create user interface for your analytics service. 
It's not exactly the same as other BI solutions such as Tableau, instead focuses on creating an UI for analytics service similar 
to Google Analytics or other specialized analytics solutions. The end-users should be able to use the interface without any complexity.
If you already have a BI solution that you're familiar with, you can also plug it to Rakam API.

We believe that you should be able to install your full-stack analytics service with one click, learn it fast and iterate quickly. 
Big-data applications doesn't need to be setup, maintain and scale.
