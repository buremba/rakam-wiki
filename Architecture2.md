## Modularity
We extensively use Guice for dependency injection and Java ServiceLoader for registering modules. Most of the core features are also implemented as modules but Rakam acts single box that you install the modules you need using the configuration file and deploy to multiple nodes optionally for high-availability and distributing the load across the computing resources. You can plug your own databases, You can also easily extend Rakam by developing modules, all you need to do is to extend the interfaces in Java depending on what you need to do, add your module class to ServiceLoader in Java, package your module and add *jar* file into the classpath.

## Networking
We use an in-house RESTFul server based on Netty called [netty-rest](https://github.com/buremba/netty-rest). It allows us to deploy API services easily and automatically generates the API documentation. Netty-rest also supports various protocols like Websockets and Server-Sent events and they're also used by Rakam heavily. For example, when we need to deploy an RESTFul service, we implement HttpService interface and add the class to Guice MultiBinder for HttpService interface. Rakam automatically registers the service and generate API documentation for the service that we developed. [Developing Modules]()

## Serialization
We use Apache Avro in order to serialize events in a compact format, which is a portable serialization library available in many programming languages. The serialized data doesn't include schema definition and the schema must be stored explicitly. Therefore we developed a schema registry service embedded in Rakam. Currently, the supported types are:

> STRING, ARRAY, LONG, DOUBLE, BOOLEAN, DATE,  TIME, TIMESTAMP

Rakam generates Avro schemas and backend database schemas using these intermediate data types.

## Query Parsing
In order to be able to control the queries that are performed via Analysis API, we needed a query parser. Unfortunately, almost all databases don't use the exact same SQL syntax so all deployment types have different implementation of query parsing. Fortunately, PrestoDB exposes the query parser module, since we already use PrestoDB for query processing and it has quite similar syntax to Postgresql we use it for now.
Rakam parses the query and re-writes it for backend storage. For example, Rakam has *materialized*, *continuous* and *collection* schemas and the default schema is collection but the backend databases usually uses schemas for project abstraction: each project has its own schema and both materialized and continuous query tables and event collection tables is stored in same schema of specific project. Therefore, the table references will be re-written and the output query will be sent to the backend database.

## Event Mappers
Event Mapping a feature that modifies event fields before storing them in databases. The cleaning and enrichment is done in this step. Rakam provides a set of enrichment mappers such us ip-to-geolocation and referrer extraction that attaches extra values to events based on previous values. For example, ip-to-geolocation event mapper looks up *ip* field and if it exists, it search the value of *ip* field in a ip-to-geolocation database called [MaxMind GeoIP2](https://www.maxmind.com/en/geoip2-databases) and attaches geo-location data to the event. Event Mappers must declare the fields they can attach as module specification, Rakam automatically register those fields at runtime and prevent any data type conflicts among the event mappers and collection fields. A tutorial can be found in [Developing module for Rakam]() section.

## Configuration
Rakam uses [Airlift configuration library](https://github.com/airlift/airlift) in order to parse the configurations using the *config.properties* file and VM options. It allows us to load configurations that are required by modules in a nice way.

## Web application
We developed a web application that uses Rakam Analysis APIs in order to create custom analytics services. Rakam does not shipped with the web application but it can download the static files of the web application and turn the API server into a web application. The web application includes the web interfaces of the Rakam modules and a report analyzer that basically allows you to execute SQL queries and draw charts from the query result and save reports interactively. The report analyzer module has similar features with Chart.io, Periscope.io and ModeAnalytics but hosted version of them. You can customize the web application, enable the modules you need, create interactive reports and finally you will end up with an analytics service that is customized just for you.