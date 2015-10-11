### Customer Analytics Module <sub>*[api-doc](//getrakam.com/api?tags=user)*</sub>
Customer Analytics Modules allows you to search for users by their attributes, their actions (events) and view event of individual users. It's quite similar to [Mixpanel](), [Gosquared](), [Kissmetrics](), [Woopra]() or any other analytics service that has customer analytics feature but hosted version of them. Currently, we only support Postgresql, you can also plug your own user database if you wish. This module attaches *user* field to your events, and integrate customer analytics to your analytics service. You can easily perform a query on event dataset and join the result to your user database using the web interface.

### CRM module (Customer Mailbox) <sub>*[api-doc](//getrakam.com/api?tags=user-mailbox)*</sub>
Customer Mailbox module is developed by customer analytics module and allows making contact with individual users. This module enables a mailbox for each user in the system and allows you to send messages to individual users. The configuration file defines the behavior of mailbox mechanism. If you need a live support chat application like [Zopim]() or [Olark](), you may enable persistent mailbox and use the Websocket connections on client side in order to create a live chat service. It's also possible to trigger custom actions like sending e-mails to users when you send a message to a user's mailbox, the mailbox mechanism is customizable and extendible.

### Real-time Analytics Module <sub>*[api-doc](//getrakam.com/api?tags=realtime)*</sub>
Real-time Analytics Module uses continuous query tables and provides you a high level interface to create real-time reports. It has a pre-defined set of aggregation functions that are implemented by backend continuous query table implementation. Instead of writing SQL queries, you can easily generate report and query that report for a specific time frame. The web interface also can create real-time charts for created reports similar to the real-time analytics services like [Chartbeat]().

### Event Stream module <sub>*[api-doc](//getrakam.com/api?tags=stream)*</sub>
Event stream module basically allows you to subscribe the event stream in real-time so that you can listen events and display them in real-time. Let's say we're running an e-commerce website and want to build a dashboard that displays the customer purchases of more than $1000 in real-time. We can subscribe *purchase* event collection stream with the filter *total_amount > 1000*.

### Developing module for Rakam 
Developing modules for Rakam is fairly easy since most of the core parts of Rakam are developed in modular fashion. You can [follow the tutorials](), [read technical details]() or [ask for a specific module to the community]()!
