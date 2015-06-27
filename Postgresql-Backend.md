# Postgresql Backend
> Rakam supports Postgresql >= 9.4.

It's possible to just use Postgresql as backend storage for Rakam. If your data can fit in one node, it's the suggested deployment type since Postgresql is quite mature and feature-complete way to use Rakam. The other deployment types usually target *big data* so they have more complex architectures than this one. However; Postgresql is actually a good choice for analytical data storage since it supports [Window function]() and [With statements](). It also has advanced features such as triggers, listen/notify that are required for Rakam. You can also also optimize Postgresql for your use cases.

## Storing Events
Rakam uses Postgresql SCHEMAs to abstract projects between each other. Each project has its own SCHEMA and **events of event collections** are stored as **rows of tables**.
PostgresqlMetastore handles for schema related operations. When the event data has a new field or a new event collection is found, Rakam calls *PostgresqlMetastore.createOrGetCollectionField* method. It converts the fields that are defined as *SchemaField* to columns in Postgresql, event collections also converted to tables in public schema in the database. We have evaluated *json*, *jsonb* and *hstore* types but found that storing data in tabular way is still the best option considering the data is not too sparse. Therefore it's always good practice to not add too many columns using Postgresql as backend.

**json** data type actually stores data as *text* in Postgresql, it just validates the json data. Since it doesn't do any optimizations, the data usually takes too much space and querying over the json columns is not efficient.

**hstore** is more efficient compared to *json* type but both the keys and values are *text* values. It makes using any other data types other than *string* for values impossible. (casting is also quite efficient so it's not an option for us.)

**jsonb** has significant improvements over *json* data type. It stores data in binary format and the query performance is much more better than *json* data type. However; we actually assume that the data in event collections is not too sparse and storing column names in each row is not a good way to store the data. Storing data in tabular format is still more efficient and convenient than using *jsonb* format.

Internally, we use a pool of JDBC connection for inserting event data to Postgresql. *PostgresqlEventStore* generates the INSERT query and executes it using the JDBC pool.

> Currently, unlike the other storage implementations, we store data in Postgresql in row-oriented format since it's the native way. Also we do not create INDEXes for columns so you should create them manually if you want to perform aggregation queries. You can also use custom foreign data wrappers such as [cstore_fdw](https://github.com/citusdata/cstore_fdw) that store data more efficiently.

## Analyzing Events
For security reasons, we do not allow executing raw Postgresql queries via Analysis API. Instead, each query is parsed and re-written before executing on Postgresql. It allows us to abstract projects and disable writable statements. When a user send a query like this one with the project tracker code *shop*:
```sql
SELECT count(*) FROM view_product
```
Rakam checks whether the user has the permission to execute query on that project and if she has, PostgresqlQueryExecutor.buildQuery rewrites the query as: (Each project has its own SCHEMA as discussed in *Storing Events* section)
```sql
SELECT count(*) FROM shop.view_product
```
Also all statements other than *SELECT* is forbidden for security reasons. Similarly continuous and materialized query tables are actually Postgresql tables with special prefixes. The prefixes are defined in *PostgresqlQueryExecutor.CONTINUOUS_QUERY_PREFIX* and *PostgresqlQueryExecutor.MATERIALIZED_VIEW_PREFIX*.

#### Materialized Query Tables
Fortunately, Postgresql supports [MATERIALIZED VIEWs](http://www.postgresql.org/docs/9.4/static/sql-creatematerializedview.html) that materialize any given query to a table. We use that feature in order to create tables and  perform [REFRESH MATERIALIZED VIEW](http://www.postgresql.org/docs/9.4/static/sql-refreshmaterializedview.html) whenever it's triggered manually or periodically if automatic refresh option is set in report definition. 
#### Continuous Query Tables
Since Postgresql does not have any streaming feature, we implemented this function in application code. Currently, we parse the SQL query and check if it's an aggregation query since it's required for incremental computation. Then depending on the GROUP BY clause, we find out the keys in aggregation query and based on those keys update the the continuous query table periodically. Since Postgresql does not expose commit log mechanism by default, we use an extra field called *time* in events that indicates the time of the creation of the events. The process is as follows:
- Execute a query that fetches new events based on *time* field and perform continuous query on the dataset. (Let's say the continious query is SELECT COUNT(*) FROM stream and we want to process *pageView* event collection. Then, the query that will be executed peridocally is similar to this one: *SELECT COUNT(*) FROM pageView WHERE time > [last commit (epoch second]*)
- Lock continious query table in order to avoid any race condition.
- Insert missing keys to the continious query table.
- Update existing keys by merging the values based on the aggregation function. (if aggregation function is *min*, then *min(existingValue, microBatchValue)* will be performed but if it's *count* it will be *existingValue+microBatchValue*)
- Release lock for continious query table.

We extensively use [CTEs](http://www.postgresql.org/docs/9.4/static/queries-with.html) in order to these operations in one transaction and [explicit locking](http://www.postgresql.org/docs/9.4/static/explicit-locking.html) in order to lock whole tables.

## Modules

##### Customer Analytics Module <sub>*[api-doc]()*</sub>
Customer analytics module in Postgresql is implemented as a seperate module and support all features in Customer Analytics Module API. By default, it stores user data in a table called *_users* in the project Postgresql schema. However; if you already use Postgresql to store user data, you may also plug your own database by setup up JDBC connection of your database in configuration.

##### CRM module (Customer Mailbox) <sub>*[api-doc]()*</sub>
We use [LISTEN](http://www.postgresql.org/docs/9.4/static/sql-listen.html)/[NOTIFY](http://www.postgresql.org/docs/9.4/static/sql-notify.html) feature of Postgresql in order to send messages to connected users in real-time. If persistency is enabled, then Postgresql creates a table called *_user_mailbox* that stores message data.

##### Real-time Analytics Module <sub>*[api-doc]()*</sub>
There's nothing fancy here, it uses continious query tables under the hood.

##### Event Stream Module <sub>*[api-doc]()*</sub>
We have a hacky solution for Event Stream Module with [LISTEN](http://www.postgresql.org/docs/9.4/static/sql-listen.html)/[NOTIFY](http://www.postgresql.org/docs/9.4/static/sql-notify.html), [TRIGGERS](http://www.postgresql.org/docs/9.4/static/sql-createtrigger.html) and PL/pgSQL.  When a user subscribe an event stream, we automatically generate a PL/pgSQL function that NOTIFYs a particular channel if the event satisfies the filter expression and add this procedure as TRIGGER for subscribed event streams. We also LISTEN that channel and send data directly to the user in JSON format. When the user closes the connection, we automatically delete the trigger, PL/pgSQL function and [UNLISTEN](http://www.postgresql.org/docs/9.4/static/sql-unlisten.html) the channel. Fortunatelly, TRIGGER feature of Postgresql is pretty rock solid and LISTEN/NOTIFY can be used for multiplexing data in runtime. It might not be the best solution but given that the data is not that big, we found that this is the cheapest solution since we're able to solve this problem using just Postgresql.

## Master Election using Postgresql
Since we allow users to deploy multiple Rakam nodes, in order to perform periodical tasks such as updating materialized query tables and continious query tables, we need a master Rakam node. Otherwise, all Rakam nodes might perform same tasks over and over and it might cause duplication in continious query tables or overhead in the system. In order to elect a master node, we use [explicit locking](http://www.postgresql.org/docs/9.4/static/explicit-locking.html) feature. Each node tries to acquire a specific lock when it's started, only one of them acquires the lock and becomes the master node. If it fails, the Postgresql connection will be closed and Postgresql will release the lock automatically so that one the other nodes will be able to acquire it automatically.