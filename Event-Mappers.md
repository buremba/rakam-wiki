### Event Mappers
Rakam enriches and cleans events collected by Rakam instances via event mappers. Event mappers nodes processes events and attaches additional fields or clean them by removing existing fields.
Currently, Rakam provides a limited set of event mappers for common use cases, you can also extend Rakam by writing custom event mappers and attaching them your Rakam nodes.
Here are the list of event mappers provided by Rakam:

###### IP Geolocation Module
Attaches a set of geolocation related fields to events based on ip field of events or socket address of the event collection request.
By default, it uses community version of [`MaxMind GeoIP API`](http://dev.maxmind.com/geoip/geoip2/downloadable/),
if you already have enterprise version on `MaxMind GeoIP2`, you can also use it as ip-to-geolocation database.

For example; if you want to create a website analytics service using Rakam and give the ability to your users analyze the visitors' location information, 
you need to attach location information to visitors' events.
Unfortunately, browsers doesn't share location information by default and requires additional permission to give you the accurate geolocation information. ([HTML Geolocation](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/Using_geolocation))
Moreover, the HTML Geolocation API is not available in older browsers so you need a ip-to-geolocation database that resolves IP to location data and attach events the location information using event mappers.

Currently, IP Geolocation module supports these attributes: `country`, `country_code`, `region`, `city`, `latitude`, `longitude`, `timezone`
By default, `ip` field will be used to resolve geolocation data but you can also customize module to use the socket address of the event collection request.

[IP Geolocation Module Configuration Reference](//getrakam.com/config#org.rakam.collection.mapper.geoip.GeoIPModule)

###### Website Referrer Module
Parses `referrer` attribute of the events, analyze referer address and attaches additional attributes 
`referrer_medium`, `referrer_source` and `referrer_term` that are related with referrer address.
It uses [Snowplow Referer Parser](https://github.com/snowplow/referer-parser) library under the hood.

[Website Mapper Module Configuration Reference](//getrakam.com/config#org.rakam.module.website.WebsiteEventMapperModule)

###### Website User-Agent Module
Parses `user_agent` attribute that contains raw user-agent data of the events, 
extract it and attaches `user_agent_family`, `user_agent_version`, `os`, `os_version` and `device_family` attributes to the event.
It also removes `user_agent` attribute from the events since the added attributes already represent it.
It uses [`ua-parser library`](https://github.com/ua-parser/uap-java) in order to parse user agent attribute. 

[Website Mapper Module Configuration Reference](//getrakam.com/config#org.rakam.module.website.WebsiteEventMapperModule)

###### Embedded Event Mappers in Modules

Some of the modules uses custom event mappers to attach additional information to events and use them later when analyzing events.
For example, event explorer module and real-time modules attaches time attribute to the events that represents the collection time.
Then they use that attribute when processing and grouping events.

###### Developing Custom Event Mappers

Developing custom event mappers is fairly easy. You need to create class that implements [`org.rakam.plugin.EventMapper`](https://github.com/buremba/rakam/blob/master/rakam-spi/src/main/java/org/rakam/plugin/EventMapper.java)
and make your event mapper identified by Rakam by adding is to EventMapper MultiBinder as follows:

```java
Multibinder<EventMapper> multiBinder = Multibinder.newSetBinder(binder, EventMapper.class);
multiBinder.addBinding().to(MyEventMapper.class).in(Scopes.SINGLETON);
```

You must also add `@AutoService(RakamModule.class)` annotation to your modules in order add them to Rakam runtime.

