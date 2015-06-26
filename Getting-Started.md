# Rakam
Rakam is an analytics platform that provides a set of analytics tools for you to create analytics services easily. It's also completely modular. You can either plug your own database or use existing storage implementation, create reports on web interface that are customized just for you and extend Rakam by writing modules for your specific needs.

At its core, Rakam is an event analytics software. It collects events via Collection APIs and makes them available for querying via Analysis APIs. On top of this event analytics software, I have added other analytical capabilities like customer, real-time analytics. You can plug your existing databases that support SQL query language or use one of the deployment types that Rakam implements.

Currently, there are two different storage solutions for events:
 - Postgresql 
 - Amazon Kinesis & Redshift
 - Kafka & PrestoDB & (Hadoop, S3 or a distributed file system)

Depending on the volume of data and the types of queries you will execute on event dataset, you can choose one of these implementations or plug your existing databases by extending Rakam. [technical details]()

## Setup Rakam
We use a configuration file *config.properties* file that specifies the backend storage and the modules that will be used. You can either use a custom configuration file or use pre-defined configurations for most common use cases. We provide *DockerFiles* and *Heroku Procfile* if you want to use Rakam in a container.

#### Using Docker
Rakam has a few *Dockerfile* for the most common deployment types. If you want to use Postgresql you can create two containers for Rakam and Postgresql:
```sh
docker
```
There is also a *DockerFile* that installs Apache Kafka, PrestoDB and Rakam:
```sh
docker
```

If you want to start Rakam using a custom configuration file, you can use the following command:
```sh
docker
```
#### Deploying Heroku
You can easily test Rakam using "Deploy to Heroku" button. It uses Heroku Postgres as backend:

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

#### Building Rakam
You can try the master branch by pulling the source code from Github and building Rakam using Maven:
###### Requirements
- Java 8
- Maven 3.2.3+ (for building)

```sh
git clone https://github.com/buremba/rakam.git
cd rakam/rakam
mvn clean install package -DskipTests
java -cp rakam/target/classes:rakam/target/dependency/* org.rakam.ServiceStarter ./config.properties
```

#### Running Rakam in your IDE
Since we already use Maven, you can import Rakam to your IDE using the root pom.xml file. We recommend using Intellij IDEA since the core team uses it when developing Rakam. Here is a sample configuration for executing Rakam in your IDE:

Main Class: org.rakam.ServiceStarter
VM Options: -ea -Xmx2G -Dconfig=etc/config.properties -Dlog.levels-file=etc/log.properties
Working directory: $MODULE_DIR$
Use classpath of module: rakam

## Collecting Events <sub>*[api-doc]()*</sub>
Rakam provides client libraries for a few programming languages for collecting events easily.
Also you can send events Rakam in JSON format to an endpoint using a RESTFul Web Service that Rakam provides. 

Client APIs:
- [Javascript Client Library]()
- [Java Client Library]()
- [Python Client Library]()
- [Sending event data in JSON data]()

> You can also send event data in JSON to the RESTFul web service of Rakam but we suggest you > to use Rakam client libraries if it's available. You can also find the specification of the > client APIs if you want to develop a client library for Rakam. [Specification]()

You can think of event as the row of a table in RDBMSs. Similar to the tables in RDBMSs, we have event collections that store the events. Event collections also have schemas but we don't enforce you do define the schema before collecting events. We can generate the schema in runtime automatically so that you don't have to do that manually.

We have aimed to make the schema evolution easy since the analytical systems almost always need it and it can be hard to do manually in runtime. As you start to send your events to Rakam, it will create the schemas based on the value types of the fields that you send as event properties. Let's say you send the following event to the Rakam:

```json
{
    "collection": "pageView",
    "properties": {
        "user_agent": "Firefox",
        "locale": "TR-tr",
        "url": "http://mysite.com/blog-post",
        "referrer": "http://google.com/?q=term",
        "ip": "186.45.356.33",
        "platform": "web",
        "page_duration": 5,
        "session_id": "c88d7bad-af01-433f-bef1-dc107cee4334"
        "user": "fdsy8fy7d"
    }
}
```
Rakam will generate the schema for this event collection based on the values that you sent. Then, creates the collection in backend storage. For example, Postgresql backend will execute a CREATE TABLE query similar to this one:

```sql
CREATE TABLE pageView (
    user_agent     TEXT,
    locale         TEXT,
    url            TEXT,
    referrer       TEXT,
    ip             TEXT,
    platform       TEXT,
    page_duration  BIGINT,
    session_id     TEXT,
    user           TEXT
);
```

Similarly, when Rakam encounters a new field, it will update the table with the new field. Therefore, even though Rakam has the ability to update the schema, it's always a good practice to have a fixed set of fields because the events are usually not stored as JSON data in underlying database, they usually have a strict schema.

> Rakam will check the fields and if they exists and values match the existing schema, the event will be sent to the storage backend as you sent. However; let's say the field *ip* is created previously and the type is *integer*. Since *string* can't cast to *integer*, the field *ip* will be ignored.

> The schema evolution feature may cause a security problem if you don't have the control on > the client side that sends the events to Rakam. For example, if you install the Javascript > tracker on website that sends the events directly from the users' browsers to Rakam, an attacker may send randomly generated fields to Rakam so you may end up event collections that have 100s of fields. Therefore we suggest disabling dynamic schemas using *disable_dynamic_schema=true* once you created the schema of your event collections or disable this feature completely on production. See config: [disable_dynamic_schema]().

#### Event Mappers
When events data is parsed, the second step is mapping events with Event Mappers. Since the data will be stored in denormalized format, you may pre-process the events before storing in backend storages. This pre-processing step may be attaching values based on existing ones or deleting the existing field values.
One of the common use cases of event mappers is the geolocation data. For example, it's possible to get IPs of users in browsers but not always possible to get the geolocation data (country, city, latitude, altitude etc.) via Javascript running on browsers. You need to attach geolocation data using a database that resolves IPs and returns geolocation data.
Rakam provides a set of event mappers like ip-to-geolocation, current time and referrer url resolution. You can enable these mappers using the configuration file. See: ([configuration file]()) It's also quite easy to add new event mappers depending on your needs. See: ([Event Mappers]())

#### Event Storage
Now the event is ready for storing. The last step for the events that you sent to Rakam is sending them to the storage implementation that you configured via configuration file. (config.properties). Depending on the implementation, you will be able to querying the event via SQL query language data-set in a short period. (See: (postgresql event storage) (presto event storage)).

## Analyzing Events <sub>*[api-doc]()*</sub>
You're ready to analyze your events. Now, we'll discuss query tables in Rakam that allows you to create reports for your analytics service. You may either use Analysis API (analysis api documentation) or web interface of Rakam (web interface documentation) for creating query tables from SQL queries. Currently, we provide 2 different query tables:

###### Materialized Query Table <sub>*[api-doc]()*</sub>
Materialized reports are similar to MATERIALIZED VIEWs in RDBMSs. Some of the queries you execute may take some time to execute so you want to the cache query results and update periodically. (hourly, daily etc.) Materialized reports may run your queries periodically and serve the cached query response when you request it. Even though the data that may not be up-to-date, materialized reports are usually efficient because the queries will not be executed too often. You can access materialized query tables by using materialized prefix and *table_name* of the query table when writing SQL queries. Let's say you generated a query table called *daily_pageviews* in your project, it's possible to query this table with *materialized.daily_pageviews* reference.

###### Continuous Query Table <sub>*[api-doc]()*</sub>
If you need real-time reports that returns the most current data, continuous query tables is your friend. Internally, continuous query tables works as state machines. Similar to the stream processing engines the event dataset will be processed in micro-batches periodically. By default, the event dataset every 5 seconds, the new dataset will be aggregated and the table will be updated so that you will be able to get the result in real-time. Similar to materialized query tables, you can reference continuous query tables with continuous prefix. In order to do be able to do incremental computation, the query must be an aggregation query.
> Please note that the implementation of continuous query tables may vary depending on which backend storage you choose. Also some of the aggregation functions may not be available depending on the underlying storage engine. You can find the detailed documentation about existing continuous query table implementations here: [Continuous query table]()

###### Reporting (Only available in web interface)
Dynamic reports treats SQL queries as if they are templates. You can use variables in SQL queries, it may be inside a WHERE condition or a function parameter. When you write a query that references continuous query tables, materialized query tables and event collection tables and put some variables inside the query, the web interface automatically asks you the types of those variables and generates the UI elements that allows you to set the value of those variables. (date-pickers, dropdowns etc.) It allows you make your reports interactive, when users set / change the values, the SQL query will be generated and executed using Rakam Analysis API. Let's do an example:
```sql
SELECT platform, count(*) FROM materialized.view_product_registered_user WHERE to_timestamp(time) > {time} AND to_timestamp(time) < {time} GROUP BY 1
```
The web interface detects the range variable and asks the variable type in a tab. When you select the data-range type, it automatically renders a date range picker that allows you to set the value of *time* variable.
Currently the supported types are string, date and date-range; we're also working on dynamic value population based on custom queries. (distinct values of a field etc.) This reporting type is only available in web interface.

> The nice feature about the continuous and materialized reports is that they can be used to generate flows of your event collections. Let's say we created a materialized report called *hourly_pageviews* from *pageview* event collection table. In order to generate *daily_pageviews*, you don't need to execute a query *pageview* event collection table. Instead, you can roll up *daily_pageviews* from *hourly_pageviews* easily.

## Modules

##### Customer Analytics Module <sub>*[api-doc]()*</sub>
Customer Analytics Modules allows you to search for users by their attributes, their actions (events) and view event of individual users. It's quite similar to [Mixpanel](), [Gosquared](), [Kissmetrics](), [Woopra]() or any other analytics service that has customer analytics feature but hosted version of them. Currently, we only support Postgresql, you can also plug your own user database if you wish. This module attaches *user* field to your events, and integrate customer analytics to your analytics service. You can easily perform a query on event dataset and join the result to your user database using the web interface.

##### CRM module (Customer Mailbox) <sub>*[api-doc]()*</sub>
Customer Mailbox module is developed by customer analytics module and allows making contact with individual users. This module enables a mailbox for each user in the system and allows you to send messages to individual users. The configuration file defines the behavior of mailbox mechanism. If you need a live support chat application like [Zopim]() or [Olark](), you may enable persistent mailbox and use the Websocket connections on client side in order to create a live chat service. It's also possible to trigger custom actions like sending e-mails to users when you send a message to a user's mailbox, the mailbox mechanism is customizable and extendible.

##### Real-time Analytics Module <sub>*[api-doc]()*</sub>
Real-time Analytics Module uses continuous query tables and provides you a high level interface to create real-time reports. It has a pre-defined set of aggregation functions that are implemented by backend continuous query table implementation. Instead of writing SQL queries, you can easily generate report and query that report for a specific time frame. The web interface also can create real-time charts for created reports similar to the real-time analytics services like [Chartbeat]().

##### Event Stream module <sub>*[api-doc]()*</sub>
Event stream module basically allows you to subscribe the event stream in real-time so that you can listen events and display them in real-time. Let's say we're running an e-commerce website and want to build a dashboard that displays the customer purchases of more than $1000 in real-time. We can subscribe *purchase* event collection stream with the filter *total_amount > 1000*.

##### Developing module for Rakam 
Developing modules for Rakam is fairly easy since most of the core parts of Rakam are developed in modular fashion. You can [follow the tutorials](), [read technical details]() or [ask for a specific module to the community]()!