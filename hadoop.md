# Hadoop

## Hadoop

* **open-source** software for reliable, scalable, distributed computing.
* framework that allows the distributed processing of large data sets across clusters of computers using simple programming models
* It is designed to scale up from single servers to thousands of machines, each offering local computation and storage. 
* it is designed to detect and handle failures at the application layer, so delivering a highly-available service on top of a cluster of computers, each of which may be prone to failures.
* stores data in distributed manner
* files as blocks
* blocks are replicated
* horizontal scaling: add extra nodes
* variety of data: all kinds
* write once, read many model
* data processing speed problem: move computation to data

Pros:

* scalability - add nodes, grow system to handle more data or add hardware to the node
* flexibility - don't preprocess data, just store it, you can decide later, how to use it
* fault-tolerance - if node goes down, jobs are redirected
* computing power - distributed model allows fast processing, more nodes - faster
* HA - if node goes down, we can reach data from its replicas; feature enables you to run redundant NameNodes in the same cluster in an Active/Passive configuration with a hot standby. This eliminates the NameNode as a potential single point of failure \(SPOF\) in an HDFS cluster.
* uses commodity hardware - no great machines needed

Cons:

1. security: storage and network levels of hadoop miss encryption
2. supports Kerberos authentication, which is hard to manage.
3. HDFS supports access control lists \(ACLs\) and a traditional file permissions model. However, third-party vendors have enabled an organization to leverage Active Directory Kerberos and LDAP for authentication
4. written in Java, a language most widely used, hence java been most heavily exploited by cyber criminals and as a result, implicated in numerous security breaches.
5. overkill for small files
6. small file is significantly smaller than the HDFS block size \(default 128MB\).
7. HDFS is for working properly with a small number of large files for storing large data sets rather than a large number of small files. 
8. If there are too many small files, then the NameNode will get overload since it stores the namespace of HDFS.
9. merge the small files to create bigger files and then copy bigger files to HDFS.
10. use HAR files \(Hadoop Archives\) for reducing the problem of lots of files
11. HAR files works by building a layered filesystem on the top of HDFS
12. HAR files are created running a MapReduce job to pack the files being archived into a small number of HDFS files. 
13. reading through HAR is not more efficient than reading through files in HDFS. 
14. each HAR file access requires two index files read as well the data file to read, this makes it slower.
15. Sequence files in which we use the filename as the key and the file contents as the value. we can put them into a single Sequence file and then we can process them in a streaming fashion operating on the Sequence file. MapReduce can **break the Sequence file into chunks** and operate on each chunk **independently** because the **Sequence file is splittable**.
16. Storing files in HBase is a very common design pattern to overcome small file problem with HDFS. We are not actually storing millions of small files into HBase, rather adding the binary content of the file to a cell.
17. stability issues
18. open-source =&gt; many developers =&gt; mistakes - run on latest stable version
19. high costs for infrastructure
20. Slow Processing Speed, Latency
21. MapReduce requires a lot of time to perform these tasks thereby increasing latency.
22. Data is distributed and processed over the cluster in MapReduce which increases the time and reduces processing speed.
23. Batch Processing only
24. it does not process streamed data
25. No Real-time Data Processing
26. use spark & flink
27. No Delta Iteration
28. not efficient for iterative processing because does not support cyclic data flow \(i.e. a chain of stages in which each output of the previous stage is the input to the next stage\).
29. Not Easy to Use
30. MapReduce developers need to hand code for each and every operation which makes it very difficult to work.
31. MapReduce has no interactive mode, but adding one such as hive and pig makes working with MapReduce a little easier for adopters.
32. No Caching
33. MapReduce cannot cache the intermediate data in memory for a further requirement

## Hadoop components

* Hadoop Distributed File System \(HDFS\) – It is the storage layer of Hadoop.
* Map-Reduce – It is the data processing layer of Hadoop.
* structured & unstructured data
* processingin parallel
* divide job into independent tasks
* break process into map & reduce phases
* YARN – It is the resource management layer of Hadoop.
* management & monitoring workloads
* implement security controls
* deliver data governance tools across cluster
* RM on master: scheduler & app manager
* node manager on slave: container & app master
* Zookeeper - cluster node configuration - coordinator
* Ambari - management & monitoring service
* Hbase - no sql db
* Hcatalog
* Thrift
* Drill
* Mahout
* Sqoop
* Flume
* Oozie

### HDFS

* distributed storage for Hadoop
* master-slave topology

### HDFS Master \(Namenode\)

* regulates file access to the clients. 
* maintains and manages the slave nodes and **assigns tasks** to them. 
* executes file system namespace operations like opening, closing, and renaming files and directories.
* runs on the high configuration hardware.

### HDFS Slave \(Datanode\)

* n number of slaves \(n up to 1000\) or DataNodes 
* manages **storage of data**. 
* the actual worker nodes that do the tasks and serve **read and write requests** from the file system’s clients.
* perform block creation, deletion, and replication upon instruction from the NameNode. 
* Once a block is written on a DataNode, it replicates it to other DataNode, and the process continues until creating the required number of replicas.
* run on **commodity hardware** having an average configuration.

HDFS has two daemons running for it.

#### NameNode Daemon

* runs on the master machine
* responsible for maintaining, monitoring and managing DataNodes.
* records the metadata of the files like the location of blocks, file size, permission, hierarchy etc.
* metadata is available **in memory** in the master for faster retrieval
* in the local disk, a copy of the metadata is available for persistence
* NameNode memory should be high as per the requirement
* captures all the changes to the metadata like deletion, creation and renaming of the file in edit logs.
* regularly receives heartbeat and block reports from the DataNodes.

#### DataNode daemon

* runs on the slave machine.
* stores the actual business data.
* serves the read-write request from the user.
* does the ground work of creating, replicating and deleting the blocks on the command of NameNode.
* after every 3 seconds, by default, it sends heartbeat to NameNode reporting the health of HDFS.

### Data storage in HDFS

* divide files into blocks \(default 128 mb\)
* create replicas for the block \(default replication factor is 3\)
* blocks are stored across different DataNodes in the cluster

### Rack Awareness in Hadoop HDFS

#### Rack

* collection of 40-50 DataNodes connected using the same network switch. 
* If the network goes down, the whole rack will be unavailable. 

#### Hadoop principles considering racks

* cluster is deployed in multiple racks.
* Communication between the DataNodes on the **same rack is more efficient** than between DataNodes on different racks.
* To reduce the network traffic during file read/write, NameNode chooses **the closest DataNode** for client request. 
* NameNode maintains rack ids of each DataNode to achieve rack information. 
* concept of choosing the closest DataNode based on the rack information is known as **Rack Awareness**.

#### Rack awareness policies

* Not more than one replica be placed on one node.
* Not more than two replicas are placed on the same rack.
* Also, the number of racks used for block replication should always be smaller than the number of replicas.

### Advantages of Rack Awareness

1. Preventing data loss against rack failure
2. Minimize the cost of write and maximize the read speed
3. Maximize network bandwidth and low latency \(пропускная способность сети и задержки\)

### Main features of HDFS

* Fault Tolerance, 
* Replication, 
* Reliability, 
* High Availability, 
* Distributed Storage, 
* Scalability

### Hadoop 3 добавил много важных фич

• Agility - теперь строится на docker контейнерах • Reduced TCO - сокращение кол-ва необходимых блоков памяти \(6 блока \* 3 репликейшена = 18 блоков памяти\), а стало 9 блоков • Scalability - появляются несколько namenode для нэймспейсов, hadoop2 имеет 1 StanbyNode, hadoop3 - несколько • Erasure coding - вместо репликации используется кодирование рида-соломона, ксор и т.д.

* improved timeline service in Hadoop3
* enables scheduling of additional resources - disks + GPUs - for deeper integration with containers for deep learning and ML
* added intra-node disk balancing when adding new disks to server \(balance data through HDD on a machine\)

### Hadoop distributions

* open-source: need to use stable version, a lot of skills, no out-of-box components
* pre-built: include different frameworks, easier to install & use, enterprise support
* cloud: everything configured for you

### HA

* the availability of system or data in the wake of component failure in the system.
* Availability if DataNode fails
* if NameNode does not receive a heartbeat from DataNode within a specified time \(10 minutes default\), the NameNode considers the DataNode to be dead.
* NameNode checks for the data in DataNode and initiates data replication. 
* NameNode instructs DataNodes containing a copy of data to replicate that data on other DataNodes.
* when a user requests to access his data, NameNode provides the IP of the closest DataNode containing user data. 
* if DataNode fails, the NameNode redirects the user to the other DataNode containing a copy of the same data.
* Availability if NameNode fails
* feature in Hadoop 2 provides a **fast failover** to the Hadoop cluster. 
* Hadoop HA cluster consists of two NameNodes \(more after Hadoop 3\) running in a cluster in an active/passive configuration with a hot standby. 
* if an active node fails, then a passive node becomes the active NameNode, takes the responsibility, and serves the client request.

#### Prior to Hadoop 2.0

NameNode is the **single point of failure** in a Hadoop cluster.

#### After Hadoop 2.0

* active node and passive node should always be in sync and must have same metadata. 
* allows to restore cluster to the same namespace where it crashed.
* Only one NameNode in the same cluster must be active at a time. 
* If two NameNodes are active at a time, then cluster gets divided into smaller clusters, each one believing it is the only active cluster. 
* This is known as the **"Split-brain scenario"** which leads to data loss or other incorrect results. 
* **Fencing** is a process that ensures that only one NameNode is active at a time.

#### Implementation of NameNode HA architecture

1. Using Quorum Journal Nodes
2. active node and passive nodes communicate with a group of separate daemons called "JournalNodes"
3. active node writes the edit log modification to the majority of JournalNodes.
4. generally three JournalNode daemons allow the system to tolerate the failure of a single machine.
5. system can tolerate at most \(N-1\)/2 failures when running with N JournalNodes.
6. should run an odd number of JNs **нечетное!** to increase the number of failures the system tolerates.
7. active NameNode updates the edit log in the JNs.
8. standby nodes, continuously watch the JNs for edit log change. 
9. Standby nodes read the changes in edit logs and apply them to their namespace.
10. If the active NameNode fails, the standby will ensure that it has read all the edits from JournalNodes before promoting itself to the Active state. 
11. standby node ensures that the NameNode state is properly synchronized before a failure occurs.
12. For fast failover, Standby node must have up-to-date information regarding the location of blocks in the cluster. To achieve this, DataNodes have the IPs of all NameNodes and send block information and heartbeats to all.

**Fencing Of NameNode**

to prevent "split-brain scenario" only one NameNode should be active at a time - this process is fencing.

* JournalNodes will allow only a single NameNode to be a writer at a time.
* During failover, the standby will take over the responsibility of writing to the JournallNodes, preventing other NameNode from continuing in the active state.
* new active node can safely proceed with failover.
* Using Shared Storage
* active node and standby nodes have access to a directory on a shared storage device in order to keep their states synchronized with each other
* During any namespace modification by active NameNode, active node log the modification record to an edit log file that is stored in the shared storage device.
* standby nodes constantly watch this directory for edits. When edits occur, the Standby node applies them to its own namespace.
* In the case of failure, the standby node must ensure that it has read all edits from the shared directory before promoting itself to the active state. So the namespace state is fully synchronized before a failover occurs.
* DataNodes send heartbeat messages and block locations to all the NameNodes. It makes standby nodes to have up-to-date information about the location of blocks in the cluster.

**Fencing of NameNode**

* administrator must configure at least one fencing method. 
* fencing method can include the killing of the NameNode process and preventing its access to the shared storage directory.
* We can fence the previously active name node with one of the fencing techniques called STONITH or "shoot the other node in the head". It uses a specialized power distribution unit to forcibly power down the NameNode machine.

### Ambari

* open source administration tool which is responsible for keeping track of running applications and their status
* open source web-based management tool which manages, monitors as well as provisions the health of Hadoop clusters.
* visualize the progress as well as the status of every application over cluster
* permits a range of tools to be installed on the cluster and administers their performances
* Removing or adding hosts to the cluster.
* Start, stop, add, remove or restart the services.
* After configuration changes, restarting the clusters or services.
* Also, it allows rollback, edits the service/components configurations.
* It helps to move the nodes to a different host.

#### Ambari Server

An authoritative process which communicates with the agents which are installed on each node on the cluster. there is an instance of Postgres database which handles all the metadata here.

#### Ambari Agent

the active member which sends the health status of every node along with diverse operational metrics

### Zookeeper

distributed coordination service which also helps to manage the large set of hosts

* Naming service: identifies the nodes by name.
* Configuration management: for a joining node, latest and up-to-date configuration information of the system.
* Cluster management: In real time, Joining / leaving of a node in a cluster and node status.
* Leader election: For coordination purpose, electing a node as the leader.
* Locking and synchronization service: while modifying it, locks the data. while connecting other distributed applications like Apache HBase, this mechanism helps  in automatic fail recovery.
* The highly reliable data registry

> Even when one or a few nodes are down guarantees the availability of data. provides a complete mechanism to overcome challenges with distributed applications. Moreover, using fail-safe synchronization approach, we can handle race condition and deadlock. resolves the inconsistency of data with atomicity.

### Yarn

