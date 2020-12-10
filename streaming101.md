# Streaming

## Streaming Data

* generated continuosly
* send simultaneously from different sources and in small sizes

logs, social netw, telemetry,...

* process sequentially line by line
* or in small batches

then - aggregation, analytics,...

* respond for emergent situation
* alarms from apps based on data
* ML algorythms
* Hadoop - for batch jobs

non-batch jobs - interactive jobs, real time queries, big data streams

* MR - no recursive or iterative jobs inherently
* all input must be ready before job. So - no online processing and streams
* copy code, schedule - another minus

## Kafka streams

distributed streaming platform

* publish ans subsribe to messages like message queue
* store streams of records in fault-tolernt durable way
* process streams as they occur

Used for:

* building real time big data pipelines
* build apps that react to the streams of data transforming them
* kafka is run as cluster on one or more servers
* streams are stored in topics
* each record = key, value, timestamp
* 4 core APIs: 
* producer - publish stream to topic, 
* consumer - subscribe to topic and get data
* stream api - stream processor - get data from topic and stream it to other topics, transform input stream to output stream;
* connector API - build reusable producres and consumers that connect topics to apps or data systems. Connector to DB - reacts to every change in a table

### Kafka streams architecture

* Kafka producer and consumer libraries 
* Data parallelism
* Distributed coordination
* Fault tolerance
* Operational simplicity

#### Partitions

Messaging layer: data is partitioned for storing and transporting

Kafka streams: data is partitioned for processing

Used for

* data locality
* Know where your data is stored
* Understand the underlying physical infrastructure \(and its bottlenecks\)
* Take advantage of the distance to the data

  Data Locality is usually measured as a distance between the data producer or consumer in term of host, rack

* elasticity

  Эластичные вычисления — это возможность быстро наращивать и освобождать ресурсы процессоров, памяти и хранения для удовлетворения меняющихся требований без необходимости планировать загрузку и предпринимать меры для обработки пиковых нагрузок.

* scalability
* high performance
* fault tolerance
* Kafka defines topics, which are composed of 1 or more partitions
* Stream partition is mapped to a Kafka topic partition
* Data record maps to a message in kafka topic
* Key in data record defines what partition does data go to
* Kafka assign data partitions to tasks and this assignment never changes
* Task is a fixed unit of parallelism in app
* Task can have multiple partitions
* For each partition task has a buffer. Messages are taken one at a time from buffer
* Tasks are processed independently and in parallel
* Partitions have 1 or more replicas
* Partitions’ replica are assigned to a broker and physically stored on disks.
* Every partition elect a leader amongst the replica, from which reads and writes happen.
* Kafka streams lib runs anywhere where stream processing app runs.
* Several app instances can be on same machine or several machines. 
* Tasks are distributed to app instances automatically by kafka
* If one app instance goes down - it's tasks will be executed on other app instance

## Spark streaming

* extension of spark api
* Provides a way to consume continuous streams of data
* Scalable, high-throughput, fault-tolerant

Система называется масштабируемой, если она способна увеличивать производительность пропорционально дополнительным ресурсам.

High-throughput computing \(HTC\) is a computer science term to describe the use of many computing resources over long periods of time to accomplish a computational task.

There are many differences between high-throughput computing, high-performance computing \(HPC\), and many-task computing \(MTC\).

### HPC tasks

* needing large amounts of computing power for short periods of time, 
* HPC environments are often measured in terms of FLOPS - floating point operations per second.

### HTC tasks

* also require large amounts of computing, but for much longer times \(months and years, rather than hours and days\)
* not concerned about operations per second, but rather operations per month or per year; is more interested in how many jobs can be completed over a long period of time instead of how fast.

HTC is “a computing paradigm that focuses on the efficient execution of a large number of **loosely-coupled tasks**”, while HPC systems tend to focus on **tightly coupled parallel jobs**, and as such they must execute within a particular site with low-latency interconnects. Conversely, HTC systems are independent, sequential jobs that can be individually scheduled on many different computing resources across multiple administrative boundaries. HTC systems achieve this using various grid computing technologies and techniques.

MTC aims to bridge the gap between HTC and HPC. MTC is reminiscent of HTC, but it differs in the emphasis of using many computing resources over short periods of time to accomplish many computational tasks \(i.e. including both dependent and independent tasks\), where the primary metrics are measured in seconds \(e.g. FLOPS, tasks/s, MB/s I/O rates\), as opposed to operations \(e.g. jobs\) per month. MTC denotes high-performance computations comprising multiple distinct activities, coupled via file system operations.

* supports sources: kafka, flume, tcp sockets, kinesis, HDFS
* Built on top of Spark Core
* Based on RDDs
* API is similar to Core API
* Supports many inputs
* Integrates with other Spark modules
* Micro batch architecture

Work sequence:

* receive live input stream
* divide it into batches
* process it by spark engine
* generate result in batches

### D Streams

* Provides high level abstraction - discretised stream or DStream.
* Dstreams are made of other Dstreams or from input streams \(see sources above\)
* it's a sequence of rdds

### Difference with batching

* the piece of data is processed at a later time when collected into a group - batch: it can be scheduled time interval or trigger \(5 elements ariived =&gt; process\)
* the piece of data is processed when it ariives - micro batching, Dstream: tiny micro batches
* data is received in parallel
* buffered into memory of workers
* run small tasks to process batches
* tasks are assigned dynamically based on data locality
* better load balancing
* faster fault recovery
* batch is rdd =&gt; all rdd transformations available
* Dstream is a sequence of rdds

### Main metrics

Batches includes blocks

#### Block interval

* Blocks of incoming data
* Defaults to 200ms
* Configurable

#### Batch interval

* Regular time intervals for new batches 
* A new batch at the beginning of an interval 
* Arriving data is added to the batch
* A new RDD at the end of each interval 
* Configured by application developer
* Between 500ms and several sec

One streaming batch corresponds to one RDD. That RDD will have n partitions, where n = batch interval / block interval. Let's say you have the standard 200ms block interval and a batch interval of 2 seconds, then you will have 10 partitions. Blocks are created by a receiver, and each receiver is allocated in a host. So, those 10 partitions are in a single node and are replicated to a second node.

When the RDD is submitted for processing, the hosts running the task will read the data from that host. Tasks executing on the same node will have "NODE\_LOCAL" locality, while tasks executing on other nodes will have "ANY" locality and will take longer.

Therefore, to improve parallel processing, it's recommended to allocate several receivers and use union to create a single DStream for further processing. That way data will be consumed and processed by several nodes in parallel.

### Receiver

Each Dstream is associated with receiver

* Runs as separate thread in an Executor
* Receives data from source
* Divides data into blocks
* Stores data in memory for processing
* Replicates blocks for fault tolerance

All receivers are inside executor. One receiver occupies one core/thread. So need more for processing. Cores &gt; number or receivers!!!

To process the data, most traditional stream processing systems are designed with a continuous operator model, which works as follows:

1. There is a set of worker nodes, each of which run one or more continuous operators.
2. Each continuous operator processes the streaming data one record at a time and forwards the records to other operators in the pipeline.
3. There are “source” operators for receiving data from ingestion systems, and “sink” operators that output to downstream systems.

**Fast failure and straggler recovery** – With greater scale, there is a higher likelihood of a cluster node failing or unpredictably slowing down \(i.e. stragglers\). The system must be able to automatically recover from failures and stragglers to provide results in real time. Unfortunately, the static allocation of continuous operators to worker nodes makes it challenging for traditional systems to recover quickly from faults and stragglers.

**Load balancing** – Uneven allocation of the processing load between the workers can cause bottlenecks in a continuous operator system. This is more likely to occur in large clusters and dynamically varying workloads. The system needs to be able to dynamically adapt the resource allocation based on the workload.

### Dynamic load balancing

Dividing the data into small micro-batches allows for fine-grained allocation of computations to resources. For example, consider a simple workload where the input data stream needs to partitioned by a key and processed. In the traditional record-at-a-time approach taken by most other systems, if one of the partitions is more computationally intensive than the others, the node statically assigned to process that partition will become a bottleneck and slow down the pipeline. In Spark Streaming, the job’s tasks will be naturally load balanced across the workers — some workers will process a few longer tasks, others will process more of the shorter tasks.

### Fast failure and straggler recovery

In case of node failures, traditional systems have to restart the failed continuous operator on another node and replay some part of the data stream to recompute the lost information. Note that only one node is handling the recomputation, and the pipeline cannot proceed until the new node has caught up after the replay. In Spark, the computation is already discretized into small, deterministic tasks that can run anywhere without affecting correctness. So failed tasks can be relaunched in parallel on all the other nodes in the cluster, thus evenly distributing all the recomputations across many nodes, and recovering from the failure faster than the traditional approach.

### Unification of batch, streaming and interactive analytics

The key programming abstraction in Spark Streaming is a DStream, or distributed stream. Each batch of streaming data is represented by an RDD, which is Spark’s concept for a distributed dataset. Therefore a DStream is just a series of RDDs. This common representation allows batch and streaming workloads to interoperate seamlessly. Users can apply arbitrary Spark functions on each batch of streaming data: for example, it’s easy to join a DStream with a precomputed static dataset \(as an RDD\).

```text
// Create data set from Hadoop file
val dataset = sparkContext.hadoopFile("file")
// Join each batch in stream with the dataset
kafkaDStream.transform { batchRDD =>
  batchRDD.join(dataset).filter(...)
}
```

Since the batches of streaming data are stored in the Spark’s worker memory, it can be interactively queried on demand. For example, you can expose all the streaming state through the Spark SQL JDBC server, as we will show in the next section. This kind of unification of batch, streaming and interactive workloads is very simple in Spark, but hard to achieve in systems without a common abstraction for these workloads.

### Minuses of Spark Streaming

* Different API for working with static data
* Dsterams API are similar to RDDs, but not the same
* No direct support for event times
* Dstreams are based on batch times
* Very difficult to process event times
* Hard to deal with late data
* No built-in end to end guarantees
* Must be handled in code
* Exactly once processing is complex

## Structured Streaming

* Datastream is a table continuosly appended.
* Every record = appended row.
* Triggered interval = rows are appended
* write to external sink
* reads latest available data, process it
* discards source data
* stores only data to update result

### Output sinks

#### Output modes

**Append**

* Only the new rows appended in the Result Table since the last trigger will be written to the external storage. 
* Default mode

**Complete**

* The entire updated Result Table will be written to the external storage. 
* It is up to the storage connector to decide how to handle writing of the entire table.

**Update**

* Only the rows that were updated in the Result Table since the last trigger will be written to the external storage \(available since Spark 2.1.1\). 

### Input Sources

* File Source: text, csv, json, orc, parquet
* Rate source: generates consecutive numbers with timestamp that can be useful for testing: value and timestamp
* Kafka source
* Socket source: for testing no fault tolerance; from socket connection

### Outout sinks

FT = fault tolerance

* file: append
* kafka: append+complete+update
* foreach: append+complete+update; can be FT or not
* console: for debug; append+complete+update; no FT
* memory: for debug; append+complete; in driver memory table; no FT

### Watermarking

* old aggregations are kept in memory during set time - watermark
* they are updated if data arrives late but within watermark
* they are dropped and close for update after watermark
* update+append

### Joins

* Inner - Supported, optionally specify watermark on both sides + time constraints for state cleanup
* Left Outer - Conditionally supported, must specify watermark on right + time constraints for correct results, optionally specify watermark on left for all state cleanup
* Right Outer - Conditionally supported, must specify watermark on left + time constraints for correct results, optionally specify watermark on right for all state cleanup
* Full Outer - Not supported

### Streaming Deduplication

You can deduplicate records in data streams using a unique identifier in the events. This is exactly same as deduplication on static using a unique identifier column. The query will store the necessary amount of data from previous records such that it can filter duplicate records. Similar to aggregations, you can use deduplication with or without watermarking.

* With watermark - If there is an upper bound on how late a duplicate record may arrive, then you can define a watermark on an event time column and deduplicate using both the guid and the event time columns. The query will use the watermark to remove old state data from past records that are not expected to get any duplicates any more. This bounds the amount of the state the query has to maintain.
* Without watermark - Since there are no bounds on when a duplicate record may arrive, the query stores the data from all the past records as state.

### foreach writer

When the streaming query is started, Spark calls the function or the object’s methods in the following way:

A single copy of this object is responsible for all the data generated by a single task in a query. In other words, one instance is responsible for processing one partition of the data generated in a distributed manner.

This object must be serializable, because each task will get a fresh serialized-deserialized copy of the provided object. Hence, it is strongly recommended that any initialization for writing data \(for example. opening a connection or starting a transaction\) is done after the open\(\) method has been called, which signifies that the task is ready to generate data.

The lifecycle of the methods are as follows:

For each partition with partition\_id:

For each batch/epoch of streaming data with epoch\_id:

Method open\(partitionId, epochId\) is called.

If open\(…\) returns true, for each row in the partition and batch/epoch, method process\(row\) is called.

Method close\(error\) is called with error \(if any\) seen while processing rows.

The close\(\) method \(if it exists\) is called if an open\(\) method exists and returns successfully \(irrespective of the return value\), except if the JVM or Python process crashes in the middle.

### Managing Streaming Queries

The StreamingQuery object created when a query is started can be used to monitor and manage the query.

```text
val query = df.writeStream.format("console").start()   // get the query object

query.id          // get the unique identifier of the running query that persists across restarts from checkpoint data

query.runId       // get the unique id of this run of the query, which will be generated at every start/restart

query.name        // get the name of the auto-generated or user-specified name

query.explain()   // print detailed explanations of the query

query.stop()      // stop the query

query.awaitTermination()   // block until query is terminated, with stop() or with error

query.exception       // the exception if the query has been terminated with error

query.recentProgress  // an array of the most recent progress updates for this query

query.lastProgress    // the most recent progress update of this streaming query
```

You can start any number of queries in a single SparkSession. They will all be running concurrently sharing the cluster resources. You can use sparkSession.streams\(\) to get the StreamingQueryManager \(Scala/Java/Python docs\) that can be used to manage the currently active queries.

```text
val spark: SparkSession = ...

spark.streams.active    // get the list of currently active streaming queries

spark.streams.get(id)   // get a query object by its unique id

spark.streams.awaitAnyTermination()   // block until any one of them terminates
```

### Unsupported Operations

There are a few DataFrame/Dataset operations that are not supported with streaming DataFrames/Datasets. Some of them are as follows.

* Multiple streaming aggregations \(i.e. a chain of aggregations on a streaming DF\) are not yet supported on streaming Datasets.
* Limit and take the first N rows are not supported on streaming Datasets.
* Distinct operations on streaming Datasets are not supported.
* Sorting operations are supported on streaming Datasets only after an aggregation and in Complete Output Mode.
* Few types of outer joins on streaming Datasets are not supported. See the support matrix in the Join Operations section for more details.

In addition, there are some Dataset methods that will not work on streaming Datasets. They are actions that will immediately run queries and return results, which does not make sense on a streaming Dataset. Rather, those functionalities can be done by explicitly starting a streaming query \(see the next section regarding that\).

* count\(\) - Cannot return a single count from a streaming Dataset. Instead, use ds.groupBy\(\).count\(\) which returns a streaming Dataset containing a running count.
* foreach\(\) - Instead use ds.writeStream.foreach\(...\) \(see next section\).
* show\(\) - Instead use the console sink \(see next section\).

If you try any of these operations, you will see an AnalysisException like “operation XYZ is not supported with streaming DataFrames/Datasets”. While some of them may be supported in future releases of Spark, there are others which are fundamentally hard to implement on streaming data efficiently. For example, sorting on the input stream is not supported, as it requires keeping track of all the data received in the stream. This is therefore fundamentally hard to execute efficiently.

## Stateful operations

Stateful transformation a particular property of spark streaming, it enables us to maintain state between micro batches. In other words, it maintains the state across a period of time, can be as long as an entire session of streaming jobs. Also, allow us to form sessionization of our data. This is achieved by creating checkpoints on streaming applications, though windowing may not require checkpoints, other operations on windowing may.

In other words, these are operations on DStreams that track data across time. It defines, uses some data from the previous batch to generate the results for a new batch.

State-ful DStreams are of two types – window based tracking and full session tracking.

For stateful tracking all incoming data should be transformed to key-value pairs such that the key states can be tracked across batches. This is a precondition.

Further we should also enable checkpointing.

