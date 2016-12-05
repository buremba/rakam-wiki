An event is an immutable action defined in your specific use-case. They have collection name and properties. If you have a website, all page views are events. You may use `pageview` as collection name and url, client location data etc. are properties. Or if you are an IoT company, all the sensor data coming from your devices is an event. Events should have an actor and timestamp of the occurence. Rakam provides multiple methods for collecting events. It has high-level methods such as trackers, client libraries, importers and also <sub>[RESTFul API](//api.getrakam.com/#event)*</sub>, webhook support, schedulers for collecting events from multiple sources. You can embed trackers in your client applications, send events from your favourite programming languages via client libraries or directly using RESTFul API, integrate your third party services via webhook or schedulers. We aim to make Rakam as your data hub so you should be able to collect data from everywhere to Rakam.

You don't need to define the collection schema, Rakam automatically handles schema evolution of the events for you. One ceveat is that if you have a property called *ip* and the type is *integer*, it tries to cast *string* to *integer* and if it fails the field *ip* will be ignored.

> Rakam will check the fields and if they exists and values match the existing schema, the event will be sent to the storage backend as you sent. 

# Trackers

Currently we have [Javascript tracker](https://rakam.io/doc/buremba/rakam-javascript/master/README) for websites and [Android SDK](https://rakam.io/doc/buremba/rakam-android/master/README) for Android applications. We're also actively working on [IOS tracker](https://rakam.io/doc/buremba/rakam-ios/master/README). Trackers are the preferred way to collect data because they're easy to integrate and automatically collect most of the data you need. For example, Javascript tracker automatically collect data about client machine, keep track of user ids between sessions and provide ways to collect platform related data such as [time on page](https://rakam.io/doc/buremba/rakam-javascript/master/README#timer) the user spend in website.

# Client Libraries

If you want to send events directly from your applications, you can use client libraries. They're basically a wrapper that uses <sub>[RESTFul API](//api.getrakam.com/#event)*</sub>.

Client APIs:
- [Javascript Client Library](https://rakam.io/doc/rakam-io/rakam-java-client/master/README)
- [Java Client Library](https://rakam.io/doc/rakam-io/rakam-php-client/master/README)
- [Python Client Library](https://rakam.io/doc/rakam-io/rakam-python-client/master/README)
- [C# Client Library](https://rakam.io/doc/rakam-io/rakam-csharp-client/master/README)
- [Ruby Client Library](https://rakam.io/doc/rakam-io/rakam-ruby-client/master/README)

Client libraries are current is in beta, they're automatically generated with Swagger. They provide classes and methods for all API endpoints in your Rakam API, you can also directly use [RESTFul API](https://api.rakam.io) to send data to Rakam.

# RESTFul Libraries

<sub>[RESTFul API](//api.getrakam.com/#event)*</sub> powers trackers and client libraries. You can always use the directly. Here is an example event:

```javascript
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
        "_user": "someuser@rakam.io"
        "_time": 1480457838042
    }
}
```

If `_time` attribute is not set, Rakam automatically attaches the current timestamp to the event. If you know the user who did the event, you should also set `_user` attribute to user id. These attributes will be used by funnel, retention and event explorer modules.

> If you want to disable schema evolution for security reasons, you can use *disable_dynamic_schema=true* config once you created the schema of your event collections. It's recommended if you are running Rakam in production and open to the clients.

# Webhook

If you want to integrate third party services with Rakam and the third party service supports Webhook, you may use our webhook support. For example, Stripe sends payment and order data, Mailgun sends mail data (unsubscribe, click, open etc.) via webhook. Webhook support is implemented as follow: You define an identifier such as `mailgun_mails` and write JS code that transforms the request body and headers and build the event using JS code. When we receive a request from `[RAKAM_API]/webhook/collect/mailgun_mails`, we invoke the JS code if it returns JSON data, we add it as a new event.
Webhook support is available in Rakam BI and we provide templates for common services such as Mailgun and Stripe.

# Scheduled Tasks

Rakam tasks are basically scheduled or continuous jobs that fetches data from other services, transform them and send event data to Rakam. We have example [Twitter task](https://rakam.io/doc/buremba/rakam-twitter/master/README) that collects tweet data from Twitter in real-time and send it to Rakam. We also have scheduled jobs module in Rakam that uses services like AWS Lambda to run code with scheduled intervals and collect data to Rakam. You can right code with your favorite programming language, set the schedule interval and collect data from third party services. 

# Importers
You can import CSV, JSON or AVRO files directly to your Rakam project. If you're already using analytics services such as Mixpanel and want to import to Rakam, we have importers that fetches raw event data from them and send it to Rakam. Currently, we have Mixpanel integration, you can find the documentation [here](https://rakam.io/doc/buremba/rakam-data-importer/master/README).

