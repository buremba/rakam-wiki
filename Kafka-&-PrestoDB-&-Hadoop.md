If you need a scalable solution that you can deploy to your own cluster, you may consider to use this deployment type. This deployment type uses multiple open-source software in order to process the event dataset and save it wherever you want.

## Storing Events
At first, the events are converted to a compact data structure using Apache Avro by Rakam and then they are sent to the to EventStore implementation that basically serialize that data structure and send the byte array to Kafka based on their event collections. Kafka acts as distributed commit-log, we basically push all data to Kafka and periodically process and store the data in a columnar format in a distributed file-system such as Hadoop using PrestoDB.Since inserting rows one-by-one to columnar databases is not usually efficient, we use micro-batching technique using Kafka.Since it's not actually a queue, we can also replay micro-batches if the nodes fail during data processing phase.The process is basically as follows:

- Serialize event to byte sequence using Apache Avro
- Send byte sequence to Kafka based on the event collection name. (We use collection names as Kafka topics)
- Execute a query periodically that fetches data from Kafka with last offset value, de-serializes it and appends to columnar storage file.
- Save last offset of processed rows in Kafka to metadata server.

There are multiple components in this system: metadata server (for event collection schemas and Kafka offsets), Kafka, Zookeeper, PrestoDB and Hadoop.
