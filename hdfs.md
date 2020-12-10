# HDFS

## HDFS

* distributed
* scalable
* fault-tolerant
* HA
* dynamically add/remove nodes
* high throughput access
* written in JAVA
* store file in blocks same size except last

Pros:

* less files + huge size
* write once, read many
* for commodity and heterogeniuos hardware

Cons:

* no low latency data access
* no small files with big amount
* no multiple writers
* no arbitrary file modification

## Components

* namenode: master, stores metadata: number of blocks, replicas, locations, file ownership and permissions; maintains and manages the slave nodes and **assigns tasks** to them.
* data node: slave; store data, read/write operations for client; more data nodes = better performance - "scale out"

**One namenode with HA option**

**Secondary namenode - no HA**

**Standby namenode** - has HA

## Architecture

* client
* master namenode
* data node

### Namenode

metadata is stored on disk and is read into memory when namenode daemon starts; = fs image + edit log

fsimage - efficient to read, inefficient to update; paths + block ids + user+group permisiions

changes to metadata made in RAm and writtent to disk in edit log.

Metadata in RAM =&gt; more files =&gt; out of RAM

### Secondary namenode

for Memory intensive administrative functions

* combine current fsimage with edit log
* give it back to namenode
* run on separate machine
* RAM as on namenode

### HA

2 namenodes - active and passive

* automatic failover
* manually initiated failover \(maitenance\)

issues:

* active & passive nodes should be in sync: same metadata
* one active at a time \(corruption of data!\): no split brain scenario \(cluster divided to clusters\)
* fencing: process of ensuring that only one namenode is active at a time
* journal nodes \(daemons\) or shared storage to connect active & passive nodes. They organise a ring topology
* data nodes send heartbeats to both namenodes

### Automatic failover

process when control redirects to seconady twin system in case of failure

Zookeper coordinates failover

* has a session with namenode; if it fails, session expires and failover process starts
* passive namenode aquire a lock from zookeeeper to become new actibe namenode
* standby NN reads from journal nodes
* zookeeper failover controllers - a processes, running on each namenode
* ZKFC use zookeeper service to determine what namenode is active
* quorum journal manager on active namenode writes FS log to journal nodes
* log is assummed written when it's on the majority of journal nodes

### Data nodes

* datanode daemon - on everu data node
* no namenode = can't access data
* rebalancing data across namenodes
* data locality info is shared with yarn to improve performance
* 3 replicas
* datanode daemon controls access to the blocks and communicates with namenode
* verification of its blocks at configurable intervals

### READ anatomy

* client call open on DFS object
* DFS with RPC calls namenode to have location of file blocks
* namenode return addresses of data nodes with the copy of the block
* data nodes are sorted by proximity to the client
* DFS returns FSDataInputStream to client
* client reads stream
* stream connects to data nodes
* data streams to client
* by end of block connection to datanode closed and opened for next block with another datanode \(by data locality\)
* client call close on FS object
* FSDataInputStream retries read on failure, verifies check sums
* corrupted blocks are reported to datanode

### WRITE anatomy

* client call open on DFS object
* DFS with RPC calls namenode to have location of file blocks
* DFS creates new file in a namespace with no blocks associated
* DFS returns FSDataOutputStream to client
* client write data
* FSDataOutputStream split data into packets and write them to internal data queue
* data queue is consumed by data streamer which asks namenode to allocate space on nearest data nodes 
* list of datanodes form a pipeline \(3 nodes for 3 replicas\)
* data streamer streams packet to nearest data node, which than streams it to second and so on
* FSDataOutputStream monitors ack queue: packet is written if all datanodes in pipeline have it and than it is removed from ack queue
* client call close
* packets flushed to queue and stream waits for acknowledgements
* namenode already knows where data is as data streamer asked him in process to allocate space for blocks

## Concepts

### Checkpointing

process of combining fs image with edit log, writing new fsimage and truncating old edit log

* CPU and IO intensive operation
* other namenode id doing it \(if it's HA cluster =&gt; secondary one\)

### HDFS metadata

* base filesystem table: "fsimage" file
* edit log: list of changes if file "edits"
* checkpoint period - maximum delay between 2 checkpoints, 1h by default
* checkpoint txns - number of uncheckpointed transactions on namenode after which chekcpointing wil be triggered even if period hasn't ended; 1 million by default
* checkpoint force
* import checkpoint - can import backed up checkpoint fs image at start of namenode

### Data locality

* moving computation to data
* yarn uses data locality
* minimize network IO

### Replication

* blocks size and repl factor configurable **per file!**
* app can specify repl factor
* repl factor can be changed later
* one write at a time in HDFS

### Rack awareness

mode in which replication happens on nodes from a different rack than the primary data node

* improve data availability, reliability
* use network bandwith properly

### Small file problem

&lt; block size

* 150 bytes for block metadata in memory

## HDFS Access

* cli
* gui: ambari, hue
* API: java, web rest, C

### Web ui

namenode-name:50070 web page

/logs /fsck /stacks /metrics

### CLI

1. hdfs dfs
2. put
3. ls
4. cat
5. mkdir
6. rm -R
7. hdfs dfsadmin
8. report - stats about cluster
9. safemode - enter this mode
10. finalizeUpgrade - remove prev backup
11. refreshNodes
12. printTopology - displays tree of racks and datanodes

### Balancer

* add new data nodes
* need to balance data across cluster
* if **utilization of datanode** = used space on node / total capacity of node **not equal** to **utilization of cluster** by more than given percentage =&gt; need to balance
* blocks are not moved across volumes of node, but across cluster
* 1 replica - on the node that is writing this block
* 2 replica - on the same rack but different node \(minimize network usage\)
* 3 replica - another rack \(fault tolerance\)

### FSCK

* problems with files
* missing blocks
* underreplicated files
* doesn't correct errors it detects!
* namenode corrects  automatically
* ignores opened files by default
* can run on whole FS or4 a subset of files

### Formatting

* hdfs dfs -rm -r "/\*"
* hadoop fs -expunge
* stop cluster
* delete data for every node \(see conf/hdfs-site.xml\)
* hdfs namenode -format
* start cluster

## HDFS Debugging

* allocation of resources
* amount ram, cpu, etc. used
* poot program logic
* network is busy
* poor configuration, monitoring and management
* ignoring logs
* no bulk changes to cluster
* check defaults
* play with java parameters
* look after GC
* test changes

## Tracing with Apache Htrace

???

## Log GC

all hadoop daemons = java apps =&gt; can pay with jvm options considering GC

* Xloggc
* verbose:gc
* etc.

Can be set in hadoop-env.sh file.

## Hadoop local analysys

* `hdfs jmxget` - dump jmx info from a service
* `jstack -F [datanode_pid]` - get stack for java threads
* `jmap -histo [pid]` - histogram of objects in jvm heap
* `jmap -histo:live [pid]` - histogram of objects in jvm heap after GC
* use htop, visualVM, etc.

## File types

* text
* parquet
* avro
* sequence
* orc

### Text files

* json
* csv
* xml
* txt
* human readable & parsable
* data stored in bulky way
* not efficient for querying and analytics
* limited compression capabilities
* easy to write - fast, inefficient for storage
* easy to read & parse, but slow

### Sequence file

* Provides persistent data structure for binary key-value pairs
* Row-based
* Commonly used to transfer Hadoop MR-jobs
* Can be used as archive to pack small files
* Not efficient for querying and analytics
* Limited compression capabilities
* Support splitting even when data is compressed
* for write: useful when data needs to be shared between MR job
* Easy to read and parse

### Avro

* Widely used as a serialization platform
* Row-based
* Offers compact and fast compression format
* Schema is stored in the file, but is segregated from data
* Splittable
* Support schema evolution
* for write: Works as serialization framework, handle schema evolution
* Easy to read and parse

### Parquet

* Column-oriented binary file format
* Uses record shredding and assembly algorithm 
* Each data file contains the values for a set of rows
* Efficient for I/O in case specific columns needs to be queried
* Schema is moved to the footer
* Integrated compression and indexes
* for write: Additional parsing needs to be done
* Easy to read and parse
* Efficient when columns needs to be queried
* Useful in scenarios when schema evolving by adding columns

### ORC

* Considered evolution of RCFile
* Stores collection of rows within the collection
* The row data is stored in columnar format
* Introduces a lightweight indexing that enables skipping of irrelevant blocks of rows
* Splittable: allow parallel processing of row collections
* It comes with basic statistics for columns
* Schema is segregated into footer
* for write: Additional parsing needs to be done
* Easy to read and parse
* Efficient when columns needs to be queried

