# Kafka

## Kafka

distributed streaming platform with 3 key capabilities:

* publish & subscribe to streams of records like message queue or enterprise messaging system
* store streams of records in FT, durable way
* process streams of records as they occur.

Used for 2 classes of apps:

* building real-time data streaming pipelines - move data between systems or apps
* building real-time data streaming apps that transform or react to the data streams

### Publish-subscribe pattern

* sender \(publisher\) - classifies the message somehow
* piece of data \(message\)
* receiver \(subscriber\) - subscribes to certain classes of messages
* broker - central point where messages are published

## Features

* fast, scalable, durable, and fault-tolerant publish-subscribe messaging system
* often used in place of traditional message brokers like JMS and AMQP because of its higher throughput, reliability and replication
* may work in combination with Storm, Spark, Samza, Flink, etc. for real-time analysis and rendering of streaming data
* brokers \(it's a verb\) massive message streams for low-latency analysis in Enterprise Apache Hadoop

## Supports

* high throughput
* reliable delivery
* horizontal scalability

## Use cases

* messaging: better throughput, built-in partitioning, replication, FT comparing to traditional QMS; good for large scale processing apps
* streaming processing: raw data to pipeline is consumed from kafka topics =&gt; aggregated, enriched, transformed into new topics for other processing or consumption; from 0.10 Kafka Streams library -  alternatives: Apache Storm, Apache Samza
* metrics collection & monitoring: used for operational monitoring data: aggregating statistics from distributed apps to produce centralized feeds of operational data  
* website activity tracking: original use case - rebuild user activity tracking pipeline as a set of real-time publish-subscribe feeds. page views, searches are published to central topics with 1 topic per activity type.
* event sourcing - style of app design where state changes are logged as a time ordered sequence of records. Kafka support for large stored log data makes it a backend for app build with this style.
* log aggregation -  collect physical log files of servers and puts into safe place \(file server, HDFS\) for processing. Kafka performs as an abstraction to gathers logs and show them as a stream of messages. Allow low latency processing and easier support for multiple data sources and distributed data consumption. Compare to log gathering systems like scribe or Flume: offers equally good performance, sctronger durability, guaranteed replication and lower end-to-end latency
* commit log \(log replication\) - log helps to replicate data between nodes and acts as a resyncing mechanism for failed nodes to restore their data. Lof compaction feature helps to support this usage. Similat to Apache Bookkeeper project

## Qualities

* Scalability - Distributed messaging system scales easily with no downtime
* Durability - Persists messages on disk, and provides intra-cluster replication
* Reliability - Replicates data, supports multiple consumers, and automatically balances consumers in case of failure
* Performance - High throughput for both publishing and subscribing, with disk structures that provide constant performance even with many terabytes of stored messages

## High level architecture

* Kafka cluster consists of one or more servers called brokers
* Producers send messages to Kafka cluster which serves them to consumers
* Broker manages the persistance and replication of message data; brokers scale and perform well in part because they are not responsible on keeping track of which messages have been consumed. Message consumer is responsible for this. This eliminates the potential for back-pressure when consumers process messages at different rates
* Zookeeper to manage the cluster. Coordinated broker-cluster topology. Elects leader among brokers and topic partition pairs. Also manages service discovery for kafka brokers. Topology changes are send by zookeper to kafka. Each node of the cluster knows when a new broker joined to the cluster, broker died, topic was removed/added. Provides an in-sinc view of kafka cluster configuration.

## APIs

* connect: build and run reusable producers & consumers that connect kafka topics to exesting apps or data systems
* consumer: subscribe to 1 or more topics and process the stream of records
* producer: publish a stream of records to 1 or more kafka topics
* stream: allows app to act as a stream processor, consuming input stream from 1 or more topics & producing output stream to 1 or more topics

## Topics

* is a feed name to which messages are published
* for each topic kafka maintains a structured commit log with 1 or more partitions
* messages are appended to a partition in an ordered immutable sequence continually appended to a commit log
* each message is assigned with sequential number that identifies message within partition. It's called **offset**

## Partitions

* provide parallelism
* writing to partition is sequential so the number of hard disk seeks is minimized, this reduces latency and increases performance
* topic may have many partitions, can handle an arbitrary amount of data \(произвольный\)

## Replicas

* is backup of a partition
* never used to read/write data; used to prevent data loss
* one broker functions as a cluster controller - responsible for administrating operations: assigning partitions to brokers, monitoring for broker failures
* partition is owned by a single broker, it's called a **leader of partition**. other replicas placed on available brokers in cluster. The replica ID is equal to broker ID who hosts replica.

## Kafka Replication

2 types of replicas:

* leader replica: each partition has a single replica that is a leader. All producer/consumer requests go through the leader to guarantee consistency
* follower replica: other replicas. Update themselves with leader to most recent state

On failure on of followers becomes a leader

## In-sync replicas

replica is in-sync if:

* it's a leader of partition
* it a follower with the following conditions:
  * it has an active session with Zookeeper- it sent a heartbeat to Zookeeper in the last 6 seconds \(configurable\).
  * it fetched messages from the leader in the last 10 seconds \(configurable\).
  * fetched the most recent messages from the leader in the last 10 seconds

otherwise replica is out-of-sync

Reasons:

* slow replica: can't catch with leader for a certain period of time: input/output bottleneck on a replica causing append copied messages from leader slower that they are consumed
* stuck replica: replica stopped fetching from leader for a certain period of time: due to GC pause or it failed/died
* bootstraping replica: user increases replication number =&gt; new replica uot-of-sync =&gt; need to follow leaders log

Kafka 0.8.2: lag \(diffrenece between follower & leader\) is measured as number of messages replica is behind leader or time for which replica has not attempted to fetch the leader

now: time that replica has been out-of-sinc with leader

## Producer load balance

* publish data to topics of their choice: which message to assign with which partition within topic: round robin fashion or semantic partitions

## Consumer load balance

2 models of messaging:

* queueing: pool of consumers read from server and each message goes to one of them
* publish-subscribe: the message is broadcast to all consumers

Kafka uses single-consumer abstraction which generalises both of them:

**consumer group**:

* consumers label themselves with group name
* each message is delivered to one consumer instance within each subscribe group
* if all consumers are in one group,  it works like a traditional queue, balancing load among consumers
* if all consumers have different groups, it works like publish-subscribe - all messages broadcast to all consumers.

Each group is composed of many consumer instances:

* scalability
* FT
* ordering guarantees \(partitioning\) and load balancing - assigning partitions of topic to consumers of consumer group: each partition is consumed exactly by one consumer in the group. One consumer is the only reader of this partition and data is ordered.
* no more consumer instances than partitions

## Consumer FT

* consumer on group can rebalance themselves \(delete one and data will be received by other consumer in the group\)

## Log compaction

* Kafka can delete data based on time or size of the log
* supports compaction for record key compaction
* log compaction retains last known value, it's full snapshot of latest records; useful for storing state after crash/ system failure
* new records are appended to head of log
* only tail get compacted
* kafka log cleaner does log compaction
* log cleaner has a pool of threads
* these threads recopy log segments files removing all records whose key appears recently in the log
* each thread chooses topic log that has the highest ratio of head/tail.
* thread recopies log from start to end removing keys that ocuur later in the log
* To turn on compaction for a topic use topic config `log.cleanup.policy=compact`

## Delivery semantics

* at least once: producer receives ack=all from broker means message written once to topic. If ack times out or receives error =&gt; retry sending message assuming message wasn't written to topic. If broker failed right before sending ack but after message was written =&gt; leads to message written twice and consumer receives duplicates
  * messages pulled one or more times, processed each time
  * receipt guaranteed
  * likely duplicates
  * no missing data
* at most once: if producer does not retry when ack times out or error returned than message might not been written to topic
  * messages pulled once
  * may not be received
  * no duplicates
  * possible missing data
* exactly once: requires interaction betwwen messaging systems and app. retried on failure message wouldn't be written twice.
  * messages pulled one or more times, processed once
  * receipt guaranteed
  * no duplicates
  * no missing data

## Topic creation

* CLI: `kafka-topics.sh --zookeeper zookeeper1:2181 --create --topic zerg.hydra --partitions 3 --replication-factor 2 --config x=y`
* API
* auto creation: auto.create.topics.enable = true    

## Topic details

`kafka-topics.sh --zookeeper zookeeper1:2181 --describe --topic zerg.hydra`

Topic:zerg2.hydra PartitionCount:3 ReplicationFactor:2 Configs: Topic: zerg2.hydra Partition: 0 Leader: 1 Replicas: 1,0 Isr: 1,0 Topic: zerg2.hydra Partition: 1 Leader: 0 Replicas: 0,1 Isr: 0,1 Topic: zerg2.hydra Partition: 2 Leader: 1 Replicas: 1,0 Isr: 1,0

* Leader: brokerID of the currently elected leader broker. Replica ID == broker ID
* ISR = “in-sync replica”, replicas that are in sync with the leader. In the above example:

  Broker 0 is leader for partition 1.

  Broker 1 is leader for partitions 0 and 2.

* All replicas are in-sync with their respective leader partitions.

## Producers & consumers

* consumers track their reading via tuples: \(offset, partition, topic\)
* offset - sequantial number of message within partition
* producers append to tail

## Producers

* Java/Scala, C/C++, Python, Ruby, etc.
* The same API provides async and sync writes
* Address of one or more brokers
* Choose a topic where to produce
* Highly configurable and tunable:
  * Partitioner
  * Number of acks \(async=0, leader=1, replicas=all\)
  * Batching improves throughput, tradeoff - data loss if client dies before pending messages have been sent; buffer size, timeouts, retries, ...
* Message is considered commited when any required ISR have applied it to their data log. Number of acks \(`acks` property\):
  * 0 - producer never waits for the ack from the broker: lowest latency, weakest durability
  * 1 - producer gets an ack after the leader replica received data: better durability. All messages written to now dead leader but not replicated will be lost
  * \(-1\) or "all" - producer get an ack after all ISR received data: best durability, no data lost
* producer is thread safe; sharing same producer across threads will be faster than multiple instances
* producer consists of pool of buffer space that holds records that have not yet been transmitted to the server; also input/output thread that turn messages into input/output requests to transmit to cluster
* if not close producer, than resources leak
* send method is synchronious; batch together records for efficiency \(`batch.size`\). One buffer for each active partition. If bigger - more memory used
* if all bootsrap brokers go down, producer stops working \(`bootstrap.servers`\).
* `retries` - number of retries on failure
* `linger.ms` - buffer defaulted to send immediately even there is additional space in buffer. If you want to send less regularly, than configure this property not equal to zero. Producer will wait this amount of milliseconds before sending request hoping that more records fill in the same batch. this increases latency but fewer requests. Records arriving close to each other go to same batch even if `linger.ms` equals 0.
* `buffer.memory` - total amount of ram available for producer to buffering
* 0.11 Kafka - transactions added to batch atomically. Records will be visible to consumers only when you commit transaction if consumers are running in read commited mode.
* The original list of messages is partitioned \(randomly if the default partitioner is used\) based on their destination partitions/topics, i.e. split into smaller batches.
* Each post-split batch is sent to the respective leader broker/ISR \(the individual `send()` happens sequentially\), and each is acknowledged by its respective leader broker according to acks.

## Consumers

* resposible to track their read positions i.e. offsets.
* Java/Scala, C/C++, Python, Ruby, etc.
* Although “High-level” and “Simple” consumer API compatibility options are still available in the latest Kafka versions, the new Consumer API is highly recommended for use

New Consumer API provides:

* Cross-Version Compatibility
* Offsets and Consumer Position
* Consumer Groups and Topic Subscriptions
* Detecting Consumer Failures

### Automatic offset commiting

* `enable.auto.commit` = true: offsets are commited automatically with interval that is set by `auto.commit.interval.ms`
* `bootstrap.servers` - list brokers
* broker automatically detect failed processes by heartbeat
* consumer periodically pings cluster to show it's alive
* if it stops heartbeating for period longer than `session.timeout.ms`than it's considered dead and his partitions are asiigned to other consumer

### Manual offset commiting

`consumer.commitSync()` if process fail before sending commit confirmation than his substitute will receive data since last commit therefore there will be duplicates - at least once.

**Consumer not thread safe**

## Kafka connect

framework witihn Kafka that integrates kafka with other systems

* make easy to add new systems to your stream data pipelines
* user initiates kafka connector for the system from which data comes \(source\) or goes to \(sink\)
* source connector import data from other systems \(RDB\)
* sink connector export data

### Concept

* Connectors – the high level abstraction that coordinates data streaming by managing tasks; logical job of copying data between system and kafka
* Tasks – the implementation of how data is copied to or from Kafka; main actors; each connector has a set of tasks
* connectors and tasks are logical units and must be scheduled to execute in a worker process
* Workers – the running processes that execute connectors and tasks
  * standalone
  * distributed
* Converters – the code used to translate data between Connect and the system sending or receiving data
* Transforms – simple logic to alter each message produced by or sent to a connector

Standard confluent connectors

* Sources:
  * ActiveMQ
  * IBM MQ
  * JDBC
  * JMS
  * Replicator
* Sinks:
  * Amazon S3
  * Elastic
  * HDFS
  * JDBC

## Kafka System tools

* Kafka Manager
* Consumer Offset Checker
* Dump Log Segment
* Export Zookeeper Offsets
* Get Offset Shell
* Import Zookeeper Offsets
* JMX Tool
* Kafka Migration Tool
* Mirror Maker
* Replay Log Producer
* Simple Consumer Shell
* State Change Log Merger
* Update Offsets In Zookeeper
* Verify Consumer Rebalance

## Monitoring

Use of standard monitoring tools is recommended

* Graphite
* Puppet module: [https://github.com/miguno/puppet-graphite](https://github.com/miguno/puppet-graphite)
* Java API, also used by Kafka: [http://metrics.codahale.com/](http://metrics.codahale.com/) 
* JMX [https://kafka.apache.org/documentation.html\#monitoring](https://kafka.apache.org/documentation.html#monitoring)
* Collect logging files into a central place
* Logstash/Kibana and friends
* Helps with troubleshooting, debugging, etc. – notably if you can correlate logging data with numeric metrics

## Performance tuning

* Increase maximum size of a message the broker will accept `message.max.bytes` \(default:1000000\) to accommodate your largest message. 
* Brokers will need to allocate a buffer of size `replica.fetch.max.bytes` for each partition they replicate \(number of partitions \* the size of the largest message must not exceed available memory\). This has to be larger than `message.max.bytes`, or a broker will accept messages and fail to replicate them.
* Same for consumers and `fetch.message.max.bytes` \(must be enough memory for the largest message for each partition the consumer reads\). This should be greater than or equal to `message.max.bytes` configuration on the broker
* Large messages may cause longer garbage collection pauses. Kafka can lose the ZooKeeper session, you may need to configure longer timeout values for `zookeeper.session.timeout.ms`.
* Default size of a Kafka data file `log.segment.bytes` \(default: 1GB\) should be fine since large messages should not exceed 1GB in size. 

