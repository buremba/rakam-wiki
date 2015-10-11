## Setup Rakam
We use a configuration file *config.properties* file that specifies the backend storage and the modules that will be used. You can either use a custom configuration file or use pre-defined configurations for most common use cases. We provide `DockerFile`s and `Heroku Procfile` if you want to use Rakam in a container.

#### Building Rakam
You can try the master branch by pulling the source code from Github and building Rakam using Maven:

###### Requirements
- Java 8
- Maven 3.2.5+ (for building)

```sh
git clone https://github.com/buremba/rakam.git
cd rakam/rakam
mvn clean install package -DskipTests -Pbundled-with-ui
```
> `-Pbundled-with-ui` argument is an optional argument that enables a profile in Maven that downloads and builds [Rakam Web application](//github.com/buremba/rakam-ui) to your local environmental. If you already downloaded it, you may use [`ui-directory setting`](//getrakam.com/config#org.rakam.ui.RakamUIModule) in your config file.

#### Running Rakam in your IDE
Since we already use Maven, you can import Rakam to your IDE using the root pom.xml file. We recommend using Intellij IDEA since Rakam is also developed in IDEA. You should import Rakam as Maven project to your favourite IDE, the configuration for executing Rakam is as follows:

Main Class: `org.rakam.ServiceStarter`

VM Options: `-ea -Xmx2G -Dconfig=config.properties -Dlog.levels-file=etc/log.properties`

Working directory: `$MODULE_DIR$`

Use classpath of module: `rakam`

#### Event Mappers
When events data is parsed, the second step is mapping events with Event Mappers. Since the data will be stored in denormalized format, you may pre-process the events before storing in backend storages. This pre-processing step may be attaching values based on existing ones or deleting the existing field values.
One of the common use cases of event mappers is the geolocation data. For example, it's possible to get IPs of users in browsers but not always possible to get the geolocation data (country, city, latitude, altitude etc.) via Javascript running on browsers. You need to attach geolocation data using a database that resolves IPs and returns geolocation data.
Rakam provides a set of event mappers like ip-to-geolocation, current time and referrer url resolution. You can enable these mappers using the configuration file. See: ([configuration file]()) It's also quite easy to add new event mappers depending on your needs. See: ([Event Mappers]())

#### Event Storage
Now the event is ready for storing. The last step for the events that you sent to Rakam is sending them to the storage implementation that you configured via configuration file. (config.properties). Depending on the implementation, you will be able to querying the event via SQL query language data-set in a short period. (See: (postgresql event storage) (presto event storage)).

## Analyzing Events <sub>*[api-doc](//getrakam.com/api?tags=event,query)*</sub>
You're ready to analyze your events. Now, we'll discuss query tables in Rakam that allows you to create reports for your analytics service. You may either use Analysis API (analysis api documentation) or web interface of Rakam (web interface documentation) for creating query tables from SQL queries. Currently, we provide 2 different query tables:

###### Materialized Query Table <sub>*[api-doc](//getrakam.com/api?tags=materialized-table)*</sub>
Materialized reports are similar to MATERIALIZED VIEWs in RDBMSs. Some of the queries you execute may take some time to execute so you want to the cache query results and update periodically. (hourly, daily etc.) Materialized reports may run your queries periodically and serve the cached query response when you request it. Even though the data that may not be up-to-date, materialized reports are usually efficient because the queries will not be executed too often. You can access materialized query tables by using materialized prefix and `table_name` of the query table when writing SQL queries. Let's say you generated a query table called `daily_pageviews` in your project, it's possible to query this table with `materialized.daily_pageviews` reference.

###### Continuous Query Table <sub>*[api-doc](//getrakam.com/api?tags=continuous-query)*</sub>
If you need real-time reports that returns the most current data, continuous query tables is your friend. Internally, continuous query tables works as state machines. Similar to the stream processing engines the event dataset will be processed in micro-batches periodically. By default, the event dataset every 5 seconds, the new dataset will be aggregated and the table will be updated so that you will be able to get the result in real-time. Similar to materialized query tables, you can reference continuous query tables with continuous prefix. In order to do be able to do incremental computation, the query must be an aggregation query.
> Please note that the implementation of continuous query tables may vary depending on which backend storage you choose. Also some of the aggregation functions may not be available depending on the underlying storage engine. Continuous query table implementations developed by Rakam: [`Postgresql`](//getrakam.com/doc/PrestoDB-Backend#continuousquerytables), [`PrestoDB`](//getrakam.com/doc/Postgresql-Backend#continuousquerytables)

###### Reporting (Only available in web interface)
Dynamic reports treats SQL queries as if they are templates. You can use variables in SQL queries, it may be inside a WHERE condition or a function parameter. When you write a query that references continuous query tables, materialized query tables and event collection tables and put some variables inside the query, the web interface automatically asks you the types of those variables and generates the UI elements that allows you to set the value of those variables. (date-pickers, dropdowns etc.) It allows you make your reports interactive, when users set / change the values, the SQL query will be generated and executed using Rakam Analysis API. Let's do an example:
```sql
SELECT platform, count(*) FROM materialized.view_product_registered_user WHERE to_timestamp(time) > {time} AND to_timestamp(time) < {time} GROUP BY 1
```
The web interface detects the range variable and asks the variable type in a tab. When you select the data-range type, it automatically renders a date range picker that allows you to set the value of `time` variable.
Currently the supported types are string, date and date-range; we're also working on dynamic value population based on custom queries. (distinct values of a field etc.) This reporting type is only available in web interface.

> The nice feature about the continuous and materialized reports is that they can be used to generate flows of your event collections. Let's say we created a materialized report called `hourly_pageviews` from `pageview` event collection table. In order to generate `daily_pageviews`, you don't need to execute a query `pageview` event collection table. Instead, you can roll up `daily_pageviews` from `hourly_pageviews` easily.
