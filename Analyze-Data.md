Rakam API provides both high and low level APIs for analyzing your event data. 
High level APIs are similar to APIs provided by analytics services. 
Currently we provide Funnel, Retention, Event Explorer modules natively.

### Event Explorer API

Event Explorer

### Funnel API

Funnel API allows you to create funnels by analyzing your event data. 
In order to be able to it, the collected events must have attached _user and _time properties.
It basically analyze the events of individual users based on your query and return the funnel results. 
You can run funnel queries directly with [Rakam API](https://api.rakam.io/#funnel) or funnel page in Rakam BI.

### Retention API

Retention API is used for creating cohort analysis reports of your users. Similar to Funnel API

### SQL Query API (low-level API)


#### Schemas
##### Collections (raw data) (collection.*)
##### Materialized Tables (materialized.*)
##### Continuous Queries (continuous.*)
##### Remote Tables (external.*)
###### Databases
###### Remote files
