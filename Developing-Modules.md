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
Cont

## Materialized Query Tables
## User modules

#### UserStorage
#### AbstractUserService
#### UserMailboxStorage

## ServiceLoader mechanism
## Packaging & installing modules
