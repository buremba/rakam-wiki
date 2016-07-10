## Network

#### Registering new endpoints to RESTFul API
Rakam extensively uses an in-house RESTFul server library [netty-rest](https://github.com/buremba/netty-rest). It makes it easy to create jax-ws style API endpoints with annotations. You can example HTTP services in main Rakam repository, if you want to bind new endpoints via modules. You need to extend `org.rakam.server.http.HttpService` abstract class, create methods and map them to HTTP endpoints by adding `org.rakam.server.http.annotations.ApiOperation` annotation and bind the class you created with Guice in `RakamModule`. Here is a simple example:

    import com.google.auto.service.AutoService;
    import com.google.inject.Binder;
    import org.rakam.plugin.RakamModule;
    import org.rakam.server.http.HttpService;
    import org.rakam.server.http.annotations.Api;
    import org.rakam.server.http.annotations.ApiOperation;
    import org.rakam.server.http.annotations.ApiParam;
    import org.rakam.server.http.annotations.JsonRequest;

    import javax.ws.rs.Path;

    @AutoService(RakamModule.class)
    class MyRakamModule extends RakamModule
    {
        @Override
        protected void setup(Binder binder)
        {
            binder.bind(HttpService.class).to(MyHttpService.class);
        }

        @Path("/mymodule")
        @Api(value = "/mymodule")
        public class MyHttpService extends HttpService
        {
            @JsonRequest
            @ApiOperation(value = "Greet users")
            @Path("/greet")
            public String greet(@ApiParam("name") String name)
            {
                return String.format("Hello %s!", name);
            }
        }

        @Override
        public String name()
        {
            return "example";
        }

        @Override
        public String description()
        {
            return "My new module";
        }
    }

#### Websockets & server-sent events
Rakam also supports Websockets and server-sent events. The Query API `QueryHttpService` uses server-sent events for real-time query stats and User Mailbox API `UserMailboxHttpService` uses Websockets for real-time chats. You can find implementations in these classes.

## Event storage
The collection API uses `org.rakam.plugin.EventStore` implementation in order to store events. If you want to implement new storage backend, you need to implement `org.rakam.plugin.EventStore`. Otherwise, if you use one the available `store.adapter` implementations, you don't need to implement this interface.

## Query executor
The implementation of `org.rakam.report.QueryExecutor` is responsible for executing SQL queries on event data-set. If you use custom EventStore, you also need to implement this interface.

## Event mappers
Event Mappers used for event richment and sanitisation. Raw events are sent to event mappers before they're stored. They can remove or modify existing attributes or attach new ones if required. For example UserAgentEventMapper parses the value of `_user_agent` attribute, remove the value of `_user_agent` attribute and attach 6 new attributes _user_agent_family, _user_agent_version, _os, _os_version and _device_family derived from raw user agent string.
Event mappers implement `org.rakam.plugin.EventMapper` interface, if they need to attach new attributes, they specify them by overriding addFieldDependency method of EventStore interface.
Event mappers can be activated via Guice Multibinder binding. After you created your Event Mapper implementation `MyEventMapper`, use the following snippet in module class to activate it:

    Multibinder<EventMapper> timeMapper = Multibinder.newSetBinder(binder, EventMapper.class);
    timeMapper.addBinding().to(MyEventMapper.class).in(Scopes.SINGLETON);
        
## User Property Mappers
User proeperty mappers are similar to event mappers but for user attributes. Similar to event mappers, they can modify or remove the existing user attributes or attach new ones. User property mappers implement `org.rakam.plugin.UserPropertyMapper` interface.
Event mappers can be activated via Guice Multibinder binding. After you created Event Mapper implementation, use the following snippet in module class to activate it:

    Multibinder<UserPropertyMapper> timeMapper = Multibinder.newSetBinder(binder, UserPropertyMapper.class);
    timeMapper.addBinding().to(MyUserPropertyMapper.class).in(Scopes.SINGLETON);
    
## Continuous Query Service
Contionuous query service is responsible from creating and deleting continuous queries. When you create a continuous query, the database continuously update the query state and allow you to reference continuous queries with `continuous` schema in SQL queries.
Let's say that you created a continuous query `SELECT device_type, count(*) as total FROM pageview GROUP BY device_type` and saved it with table name `devices`. When you execute the following query `SELECT device_type, total FROM continuous.devices ORDER BY total DESC LIMIT` the system will fetch the state from continuous query, sorts the results and return it without actually processing the data in `pageview` table.

If you want to implement a custom continuous query service, you need to create a class and implement `ContinuousQueryService` interface, then you need bind it in your module class.

Currently, there are three Continuous Query Service implementations. The first one is for Presto deployment and uses Presto streaming module created for Rakam. The other one is for Clickhouse deployment and uses `AggregatingMergeTree` table engine in Clickhouse database. The last one is for Postgresql but since Postgresql doesn't provide a way to perform stream processing on dataset, we use VIEWs in Postgresql.

## Materialized Query Tables
Materialized query tables are similar to continuous queries but instead of continuosly processing data, it just materializes the query result and allow you to query the materialized data. The functionality is exactly same as Materialized views in RDBMSs but it also provides lazy materialization at query time, data expiration and partitioning for efficiently analzing event dataset.

If you want to implement custom Materialized Query Service, you need to create a class and implement `MaterializedQueryService` interface, then you need bind it in your module class.

## User modules
User module is simply allows you to track your users, their attributes and communicate with them.
#### UserStorage
`UserStorage` stores user attributes and implements the methods for incrementing a user attribute, appending / prepending values to array user attributes, and delete / set / set_once actions for user attributes. Currently, we provide Postgresql and Dynamodb as user storage implementations but you can also implement your own database or just plug an existing database to Rakam.

#### AbstractUserService
AbstractUserService perform user related actions in API. For user storage actions, it works similar to proxy. For example, set_properties endpoint invokes set_property method of AbstractUserService, then it executes set_property method of UserStorage implementation.

It's also responsible for querying user events based on user attributes and user groups. Since we abstract user attributes and event storage in Rakam, performing queries that needs to process both user attributes and events is not that easy. For example, if you use Presto deployment and want to get users who has attribute `paid` set to true and did at least 10 pageview events, PrestoUserService fetches user ids who has `paid` set to true from UserStorage and constructs a Presto SQL query that limits users for that set. However if you store both events and user attributes in Postgresql, PostgresqlUserService simply JOINs the datasets in your Postgresql database.

#### UserActionService
User action mechanism is simply created for you to interact with your users. Currently, the only user action is email, it allows you to send email to your users. You can spesify filters for users based on their events (i.e. did at least 10 checkpoint since this monday) and their attributes (i.e. the user language is turkish) and perform an action for the user set.

If you want to implement a custom User Action, you need to create a class and implement `org.rakam.plugin.userUserActionService`. You can find an example user action implementation in `org.rakam.plugin.user.UserEmailActionService` in rakam-main module. User actions also implements `HttpService` because we also create new endpoints for the user actions. 

## Packaging & installing modules
Creating new modules for Rakam is quite simple. You just need to add `rakam-spi` maven dependency to your project, create a class for your module and extend the class from `org.rakam.plugin.RakamModule`, add `@AutoService(RakamModule.class)` in order to make Rakam discover your new module and bind your services, interface implementations bind Guice bindings. in `public void setup(Binder)` method. You can find examples in rakam-main projects.

## ServiceLoader mechanism
Almost all features of Rakam are added as modules. We simply extends `org.rakam.plugin.RakamModule`, bind new services / functionalies in our module class and add `@AutoService(RakamModule.class)` annotation to the class. `AutoService` automatically creates ServiceLoader bindings for our modules. When Rakam is initilialized, it looks for modules that is installed via ServiceLoader in classpath and installs them.
