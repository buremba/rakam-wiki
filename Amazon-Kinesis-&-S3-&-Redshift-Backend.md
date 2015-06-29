This deployment type uses the big data solutions of Amazon that is easy to scale and manage. We deploy our services that are using Beanstalk, S3, Kinesis, Data pipeline, Redshift Amazon products and coordinate between these products using Rakam. As a result, we get an fault-tolerant, scalable and cost-effective analytics services.

## Storing Events 

At first, we deploy Rakam Collection API services to Amazon EC2 instances using Beanstalk that serialize data and send it to Amazon Kinesis. Similar to the Apache Kafka in Apache Kafka & PrestoDB deployment type, Kinesis is a real-time stream processing engine thats acts like commit-log in our architecture. Collection APIs the events are converted to a compact data structure using Apache Avro by Rakam and then they are sent to the to EventStore implementation that basically serialize that data structure and send the byte array to Kinesis partitions based on their event collections.

The other part of the application that is using Beanstalk consumes the data in Kinesis parallely and store the data in S3 in batches using buffering technique. The data stored in S3 is in uncompressed CSV format so not efficient for querying. We also use an Amazon Data Pipeline job that copies data from S3 to Amazon Redshift, which is a distributed columnar database that store data in an efficient format for complex aggregation queries. It periodically (usually a few hours) feeds Redshift with new event dataset and allow us to execute SQL queries on that dataset efficiently.

## Analyzing Events

