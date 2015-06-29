Rakam is an analytics platform that provides a set of analytics tools for you to create analytics services easily. It's also completely modular. You can either plug your own database or use existing storage implementation, create reports on web interface that are customized just for you and extend Rakam by writing modules for your specific needs.

At its core, Rakam is an event analytics software. It collects events via Collection APIs and makes them available for querying via Analysis APIs. On top of this event analytics software, I have added other analytical capabilities like customer, real-time analytics. You can plug your existing databases that support SQL query language or use one of the deployment types that Rakam implements.

Currently, there are two different storage solutions for events:

Postgresql
Amazon Kinesis & Redshift
Kafka & PrestoDB & (Hadoop, S3 or a distributed file system)
Depending on the volume of data and the types of queries you will execute on event dataset, you can choose one of these implementations or plug your existing databases by extending Rakam.