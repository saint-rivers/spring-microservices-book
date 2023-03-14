# Kafka

Apache Kafka is a publish-subscribe based durable message broker used to exchange data between processes or servers. It is an open-source used for high-performance data streaming pipelines, streaming analytics and data integration applications. Its high throughput, distributed and scalable nature makes it incredibly useful for data analytics and building event-driven mircroservices. 

## Background

Apache Kafka is written in Scala and Java and is the creation of former LinkedIn data engineers. As early as 2011, the technology was handed over to the open-source community. Today, Apache Kafka is part of the Confluent Stream Platform.

## Apache Kafka vs. Confluent Kafka

Apache Kafka is free and open source. It is available under Apache 2.0.

The Confluent Kafka Platform is free to download and use under the Confluent Community License.  Unlike Apache Kafka which is available under the Apache 2.0 license, the Confluent Community License is not open source and has restrictions.  

However, Confluent does provide a community version of Confluent Kafka which includes the functionalities of Apache Kafka's data streaming (producers, consumers, topics, partitions, etc) and also some additional features of the the Confluent Platform under the Confluent Community License. These features include:
- Confluent Schema Registry: a centralized service that manages schemas for Kafka data
- Confluent Connectors: pre-built connectors that integrate Kafka with various sources and sinks, such as databases, cloud services, etc
- Confluent ksqlDB: a distributed SQL engine that enables stream processing on Kafka data

Commercial features of the Confluent Platform includes:
- Confluent Cloud
- Confluent Control Center
- Confluent Replicator

## Building an application with Kafka

Apache Kafka is a software where topics can be defined (think of a topic as a category), applications can add, process and reprocess records.

<img src="https://www.cloudkarafka.com/img/blog/durable-message-system.png"/>

Applications connect to this system and transfer a record onto the topic. A record can include any kind of information; for example, information about an event that has happened on a website, or an event that is supposed to trigger an event. Another application may connect to the system and process or re-process records from a topic. The data sent is stored until a specified retention period has passed by.

Records are byte arrays that can store any object in any format. A record has four attributes, key and value are mandatory, and the other attributes, timestamp, and headers are optional. The value can be whatever needs to be sent, for example, JSON or plain text.

These are four main parts in a Kafka system:

- Broker: The server that handles all requests from clients (produce, consume, and metadata) and keeps data replicated within the cluster. There can be one or more brokers in a cluster. 
- Zookeeper: The server that keeps the state of the cluster (brokers, topics, users).
- Producer: Sends records to a broker. 
- Consumer: Consumes batches of records from the broker.

In the remaining parts of the books, Confluent Kafka will be used as the brokers and the producer and consumer will be written using Spring Boot.

## Zookeeper

Zookeeper is a centralized service that maintains naming and configuration data and provides synchronization within distributed systems. It also provides services such as leader election and node coordination in a distributed system. It is used by projects such as Apache Kafka, Hadoop, HBase and others to provide high availability and scalability in a system.

Kafka uses Zookeeper to manage **service discovery** for the brokers that form the cluster. Zookeepers also keeps track of the topics and partitions. Zookeeper itself is allowing multiple clients to perform simultaneous reads and writes and acts as a shared configuration service within the system. The Zookeeper atomic broadcast (ZAB) protocol is the brains of the whole system, making it possible for Zookeeper to act as an atomic broadcast system and issue orderly updates.

> Note: Kafka is only a message broker and cannot manage its own partitions and topics. It is essentially a slave to Zookeeper and requires a single instance of Zookeeper to operate, even in a cluster where only a single instance of Kafka is running. 

> However, there is a plan to remove Zookeeper as a dependency in the future, and use a Raft-based protocol to store and manage metadata within a single Kafka instance. This simplifies the architecture greatly for applications running a only one Kafka instance and will also be useful for Kubernetes deployment. This is called the "KRaft" mode and is available in the Apache Kafka 3.1 release, but it is still an experimental feature as of the writing of this book.

## Kafka Broker

A Kafka cluster consists of one or more servers (Kafka brokers) running Kafka. Producers are processes that push records into Kafka topics within the broker. A consumer pulls records off a Kafka topic.

Running a single Kafka broker is possible but it doesn’t give all the benefits that Kafka in a cluster can give, for example, data replication.

<img src="https://www.cloudkarafka.com/img/blog/kafka-broker-beginner.png"/>

Management of the brokers in the cluster is performed by Zookeeper. There may be multiple Zookeepers in a cluster, in fact the recommendation is three to five, keeping an odd number so that there is always a majority and the number as low as possible to conserve overhead resources.

## References:

https://www.cloudkarafka.com/blog/part1-kafka-for-beginners-what-is-apache-kafka.html?gclid=Cj0KCQjwk7ugBhDIARIsAGuvgPYgu0jFLqB1Dc3g2MO6I0ogl121TW4Q2r6flPNEgMBxc8UJ0mQaolwaAjxOEALw_wcB
