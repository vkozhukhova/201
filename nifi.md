# NiFi

## Main goal

* connect producer and consumer
* problem 1: no single standard for moving data
* problem 2: too many input sinks and too many output sinks

Moving data is hard because of the number of reasons:

* Formats
* Protocols
* Compliances
* Schemas
* Volume/Variety/Velocity of data
* Ensuring security
* Ensuring reliability
* Ensuring availability
* Additional effort to onboard new producer/consumer

-problem 3: Telegram problem \(read about it\)

## Nifi

framework to build scalable direct graphs of data routing, transformation ans system mediation logic

### NIFI features:

* Web-based user interface
  * Seamless experience between design, control, feedback, and monitoring
* Highly configurable
  * Loss tolerant vs guaranteed delivery
  * Low latency vs high throughput
  * Dynamic prioritization
  * Flow can be modified at runtime
  * Back pressure: Backpressure is when the progress of turning that input to output is resisted in some way. In most cases that resistance is computational speed — trouble computing the output as fast as the input comes in — so that’s by far the easiest way to look at it. But other forms of backpressure can happen too: for example, if your software has to wait for the user to take some action. NiFi provides two configuration elements for Back Pressure. These thresholds indicate how much data should be allowed to exist in the queue before the component that is the source of the Connection is no longer scheduled to run. This allows the system to avoid being overrun with data. The first option provided is the "Back pressure object threshold." This is the number of FlowFiles that can be in the queue before back pressure is applied. The second configuration option is the "Back pressure data size threshold." This specifies the maximum amount of data \(in size\) that should be queued up before applying back pressure. 
* Data Provenance \(происхождение\)
  * Track dataflow from beginning to end
* Designed for extension
  * Build your own processors and more
  * Enables rapid development and effective testing
* Secure
  * SSL, SSH, HTTPS, encrypted content, etc...
  * Multi-tenant authorization and internal authorization/policy management

## flow-based programming \(FBP\)

is a programming paradigm that defines applications as networks of "black box" processes, which exchange data across predefined connections by message passing, where the connections are specified externally to the processes. These black box processes can be reconnected endlessly to form different applications without having to be changed internally. FBP is thus naturally component-oriented.

FBP is a particular form of dataflow programming based on bounded buffers, information packets with defined lifetimes, named ports, and separate definition of connections.

* we minimize amount of time to implement code and apply drag-n-drop techniques in building flow

## NiFi terms

### FlowFile

is an object transported through the system. NiFi is data agnostic which means FlowFile can contain any type of information in any format. Similar to HTTP data, Flow file contains two parts:

* Header, with metadata
* Body, with message content

## NiFi architecture

* executes in JVM on host OS

### components of nifi jvm

* web server: host nifi http based commands and control API
* flow controller: brain of operation. Provides threads for extensions to run on and manages the schedule when extension receives resources to execute
* extensions: they operate and execute within jvm
* Flowfile repository: it's where nifi actually keeps track of the state of what it knows of a given flowfile that is persistently active in the flow. Implementation of repository is **pluggable**. Default approach - a persistent write-ahead log located on a spesified disk partition.
* Content repository: is where actual content bytes of a given flowfile leave. Implementation of repository is **pluggable**. Default approach - simple mechanism - stores blocks of data in his filesystem. More than one filesystem storage location can be specified so there are different physical partitions engaged to reduce contention on any single volume
* Provenance repository - where all provenance data is stored. Implementation of repository is **pluggable**. Default approach - to use one or more physical disk volumes. Within each location event data is indexed and searchable.

NiFi is also opertated within cluster - increase availability, reliability and throughput of solutions. It can operate on a single machine.

## Nifi 1.0

zero master clustering paradigm.

* Each node of the cluster performs same tasks on data but each one operate on different set of data.
* zookeeper elects a single node as a cluster coordinator and failover is handled automatically by zookeeper. all cluster nodes report heartbeat to cluster coordinator. 
* cluster coordinator is responsible for connecting and disconnecting nodes.
* every cluster has primary node elected by zookeeper
* can interact with nifi cluster through user interface of any node. Any change is replicated to all nodes of cluster allowing multiple entrypoints.

Q1: What is Apache NiFi?

Ans: NiFi is helpful in creating DataFlow. It means you can transfer data from one system to another system as well as process the data in between.

Q2: What is NiFi FlowFile?

Answer: A FlowFile is a message or event data or user data, which is pushed or created in the NiFi. A FlowFile has mainly two things attached with it. Its content \(Actual payload: Stream of bytes\) and attributes. Attributes are key value pairs attached to the content \(You can say metadata for the content\).

Q3. Can NiFi Flow file have unstructured data as well?

Ans: Yes, FlowFile in NiFi can have both the Structured \(e.g. XML, JSON files\) as well as Unstructured \(Image files\) data.

Q4. Where does content of FlowFile stored?

Ans: FlowFile does not store content itself. It stores the reference to the contents, which are stored in the content repository.

Q5. What is a NiFi Processor?

Ans: Processor is a main component in the NiFi, which will really work on the FlowFile content and helps in creating, sending, receiving, transforming routing, splitting, merging, and processing FlowFile.

Q6. What is the Processor Node?

Ans: Processor Node is a wrapper around the Processor and maintain the state about the processor. Processor Node maintains the

* Processors positioning in the graph.
* Configured properties of the processor
* Settings of the Processor
* Schedule states of the processor etc.

Q7. Can NiFi installed as a service?

Ans: Yes, it’s currently supported in Linux and MacOS only.

Q8. What is relationship in NiFi dataflow?

Ans: When a processor finishes with processing of FlowFile. It can result in Failure or Success or any other relationship. And based on this relationship you can send data to the Downstream or next processor or mediated accordingly.

Q9. What is Reporting Task?

Ans: A Reporting Task is a NiFi extension point that is capable of reporting and analyzing NiFi's internal metrics in order to provide the information to external resources or report status information as bulletins that appear directly in the NiFi User Interface.

Q10. Can processor commit or rollback the session?

Ans: Yes, processor is the component through session it can commit and rollback. If a Processor rolls back the session, the FlowFile that were accessed during that session will all be reverted to their previous states. If a Processor instead chooses to commit the session, the session is responsible for updating the FlowFile Repository and Provenance Repository with the relevant information.

