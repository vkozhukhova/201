# HBase

* nosql datastore built on top of hdfs
* all data stored in HDFS
* column family oriented DB
* based on google bigtable paper
* data can be retrieved quickly or batch processed with MR/ Spark

## Use cases for Hbase

* useful for terabyte/petabyte big data volumes
* high-throughput
* variable columns: some will appear, some will not
* need random reads and writes

## Architecture

### Daemons

* HBase Master
* RegionServer
* ZooKeeper
* HDFS
  * NameNode/Standby NameNode
  * DataNode
* on master nodes: HBase Master - slaves coordinator - several Masters, HA with Zookeeper.
  * Coordinating the region servers&#x20;
  * Assigning regions on startup , re-assigning regions for recovery or load balancing&#x20;
  * Monitoring all RegionServer instances in the cluster (listens for notifications from zookeeper)
  * Admin functions - Interface for creating, deleting, updating tables
* on data nodes: RegionServer - worker - storing, reading, scanning data&#x20;

HBase is composed of three types of servers in a master slave type of architecture.

* Region servers serve data for reads and writes. When accessing data, clients communicate with HBase RegionServers directly.&#x20;
* HBase Master process. Region assignment, DDL (create, delete tables) operations
* Zookeeper maintains a live cluster state.
  * maintains which servers are alive and available,&#x20;
  * provides server failure notification.&#x20;
  * uses consensus to guarantee common shared state. There should be three or five machines for consensus.
* The Hadoop DataNode stores the data that the Region Server is managing.
* All HBase data is stored in HDFS files.&#x20;
* Region Servers are collocated with the HDFS DataNodes, which enable data locality (putting the data close to where it is needed) for the data served by the RegionServers.&#x20;
* HBase data is local when it is written, but when a region is moved, it is not local until compaction.

## Hmaster & Zookeeper

* Zookeeper is used to coordinate shared state information for members of distributed systems.&#x20;
* Region servers and the active HMaster connect with a session to ZooKeeper.&#x20;
*   ZooKeeper maintains ephemeral nodes for active sessions via

    heartbeats. Ephemeral node in zookeeper are are temporary kind of znodes. These znodes exists for a specific session only. They gets created for a session and as soon as session ends they also get deleted.
* Each Region Server creates an ephemeral node.&#x20;
*   HMaster monitors these nodes to discover available

    region servers, and it also monitors these nodes for server failures.&#x20;
* HMasters vie to create an ephemeral node.
* Zookeeper determines the first one and uses it to make sure that only one master is active.&#x20;
* The active HMaster sends heartbeats to Zookeeper, and the inactive HMaster listens for notifications of the active HMaster failure.
* If a region server or the active HMaster fails to send a heartbeat, the session is expired and the corresponding ephemeral node is deleted. Listeners for updates will be notified of the deleted nodes.&#x20;
*   The active HMaster listens for region servers,

    and will recover region servers on failure.&#x20;
* The Inactive HMaster listens for active HMaster failure, and if an active HMaster fails, the inactive HMaster becomes active.

## NoSql table architecture

* namespace (table)
* column descriptors (column names)
* row key (row identifier)
* column family - several columns stored together - physical and logical level. We can group frequently accessed data together. Settings can be customized per column or per family. Read faster. Can compress column families
* if some column value doesn't exist, we won't store null, we just don't store anything - more efficient
* can be multiple versions of data - has timestamps
* types can differ between rows. e.g. can store strings for some column values and integers for others. So it's just about Bytes, not data types.

## Scalability

* regions - unit of scalability
* region - subsets of rows
* region server serves some regions
* when write to hbase with yoyr clienbt, you need to understand which region server is serving the row key you are writing
* client asks hbase master, which region server should I write to
* client contacts region server directly
* same for read

## Region

* Physically, a table is split into multiple blocks, each of which is an HRegion.&#x20;
* We use the table name + start/end primary key to distinguish each HRegion.&#x20;
* One HRegion will save a piece of continuous data in a table.&#x20;
* A complete table data is stored in multiple HRegions.

## HStore

* HRegion is composed of multiple HStores.&#x20;
* Each HStore corresponds to the storage of one column family in the logical table.&#x20;
* It can be seen that each column family is a centralized storage unit.&#x20;
* to improve operational efficiency, it is preferable to place columns with common I/O characteristics in one column family.
* hstore consists of **MemStore** and **StoreFiles**.&#x20;
* MemStore is a memory buffer.&#x20;
* The data written by the user will first be put into MemStore.&#x20;
* When MemStore is full, Flush will be a StoreFile (the underlying implementation is HFile).&#x20;
* When the number of StoreFile files increases to a certain threshold, the Compact merge operation will be triggered, merge multiple StoreFiles into one StoreFile, and perform version merge and data delete operations during the merge process.&#x20;
* HBase only adds data, and all update and delete operations are performed in the subsequent Compact process, so that the user’s write operation can be returned as soon as it enters the memory, ensuring the high performance of HBaseI/O.&#x20;
* When StoreFiles Compact, it will gradually form a larger and larger StoreFile.&#x20;
* When the size of a single StoreFile exceeds a certain threshold, the Split operation will be triggered.&#x20;
* At the same time, the current HRegion will be split into 2 HRegions, and the parent HRegion will go offline.&#x20;
* The two sub-HRegions are assigned to the corresponding HRegionServer by HMaster so that the load pressure of the original HRegion is shunted to the two HRegions.

## HLog

* Each HRegionServer has an HLog object, which is a pre-written log class that implements the Write Ahead Log.&#x20;
* Each time a user writes data to MemStore, it also writes a copy of the data to the HLog file.&#x20;
* The HLog file is periodically scrolled and deleted, and the old file is deleted (data that has been persisted to the StoreFile).&#x20;
* When HMaster detects that a HRegionServer is terminated unexpectedly by the Zookeeper, HMaster first processes the legacy HLog file, splits the HLog data of different HRegions, puts them into the corresponding HRegion directory, and then redistributes the invalid HRegions.&#x20;
* In the process of loading HRegion, HRegionServer of these HRegions will find that there is a history HLog needs to be processed so the data in Replay HLog will be transferred to MemStore, then Flush to StoreFiles to complete data recovery.

## API

* java api - the only first class citizen??
* rest interface allows http access
* thrift gateway allow non-java programmatic access
* native sql interfaces: apache phoenix, spark, impala, presto, hive - from 3.0.0+ version

### Types of Access

* Gets: Gets a row’s data based on the row key, implemented through Scan
* Puts: Upserts a row with data based on the row key, no range
* Scans: Finds all matching rows based on the row key. Scan logic can be increased by using filters
* Deletes: by row key, no range. It's a martker added, actula delete - during compaction

```java
Get g = new Get(ROW_KEY_BYTES);
Result r= table.get(g);
//get bytes from cell
byte[] byteArray = r.getValue(COLFAM_BYTS,COLDESC_BYTS);
// you need to know what type of data is in cell
String columnValue = Bytes.toString(byteArray);

Put p = new Put(Bytes.toBytes(ROW_KEY_BYTES));
// add columns to put
p.addColumn(COLFAMILY_BYTES, COLDESCRIPTOR_BYTES, Bytes.toBytes("value"));
table.put(p);
```

## Architecting hbase solutions

* RDBMS is about relationships and normalization
* HBASE is about how to access data and denormalize it
* you should think of access patterns properly
* fastest ways to read/write data
* how organize data

### Row key design

* row key design is engineering problem
* you only have one index or primary key
* getting it right takes time and effort, it's not just auto-inc number
* row key often contains multiple pieces of data (composite)
* row keys bytes are stored sorted
* keep RK as short as reasonable
* **Hotspotting** occurs when a large amount of client traffic is directed at one node, or only a few nodes, of a cluster. To avoid:
  * **salting** - adding a randomly-assigned prefix to the row key to cause it to sort differently than it otherwise would. The number of possible prefixes correspond to the number of regions you want to spread the data across.&#x20;
  * **Hashing** - Instead of a random assignment, you could use a one-way hash that would cause a given row to always be "salted" with the same prefix, in a way that would spread the load across the RegionServers, but allow for predictability during reads. Using a deterministic hash allows the client to reconstruct the complete rowkey and use a Get operation to retrieve that row as normal.
  * **Reversing the Key** - reverse a fixed-width or numeric row key so that the part that changes the most often (the least significant digit) is first. This effectively randomizes row keys, but sacrifices row ordering properties.

### Schema design

* Access pattern must be known and ascertained
* Denormalize to improve performance - Fewer, bigger tables
* use bulk loading for incremental or time series data

## Root and Meta

* All HRegion metadata of HBase is stored in the .META. table.&#x20;
* As HRegion increases, the data in the .META table also increases and splits into multiple new HRegions.
* To locate the location of each HRegion in the .META table, the metadata of all HRegions in the .META the table is stored in the -ROOT-table,&#x20;
* the location information of the ROOT-table is recorded by Zookeeper.
* clients need to first access Zookeeper to obtain the location of -ROOT-, then access the -ROOT-table to get the location of the .META table, and finally determine the location of the user data according to the information in the META table, as follows:&#x20;
* The -ROOT-table is never split.&#x20;
* It has only one HRegion, which guarantees that any HRegion can be located with only three jumps.&#x20;
* To speed up access, all regions of the .META table are kept in memory.
* The client caches the queried location information, and the cache does not actively fail.&#x20;
* If the client still does not have access to the data based on the cached information, then ask the Region server of the relevant .META table to try to obtain the location of the data.&#x20;
* If it still fails, ask where the .META table associated with the -ROOT-table is.
* Finally, if the previous information is all invalid, the data of HRegion is relocated by ZooKeeper.&#x20;
* So if the cache on the client is entirely invalid, you need to go back and forth six times to get the correct HRegion.

### META table

* META table is an HBase table that keeps a list of all regions in the system.
* The .META. table is like a B-tree.
* The .META. table structure is as follows:&#x20;
  * Key: (table, region start key, region id)
  * Values: RegionServer

## RegionServer

* Region Server runs on an HDFS data node components:
* WAL: Write Ahead Log is a file on the **distributed file system**.
  *   is used to store new data that hasn't yet been

      persisted to permanent storage;&#x20;
  * is used for recovery in the case of failure.
* BlockCache: is the read cache.&#x20;
  * It stores frequently read data in memory.&#x20;
  * Least Recently Used data is evicted when full.
* MemStore: is the write cache. **Sorted map of KeyValues in memory**
  * It stores new data which has not yet been written to disk.&#x20;
  * It is sorted before writing to disk.&#x20;
  * There is one MemStore per column family per region.
* Hfile: is immutable and store the rows as **sorted KeyValues** (Sorted on rowkey + qualifier + timestamp) **on disk**. Each column family is stored in a separate file (called HFile)
* Key & Version numbers are replicated with each column family
* Empty cells are not stored   &#x20;

## Write procedure

* Instruction is directed to Write Ahead Log and first writes important logs to it.&#x20;
*   Although it is not the area where the data

    is stored, it is done for the fault tolerant purpose.&#x20;
* later if any error occurs while writing data, HBase always has WAL to look into.
* Edits are appended to the end of the WAL file that is stored on disk.
* Once the log entry is done, the data to be written is forwarded to MemStore which is actually the RAM of the data node.
* All the data is written in MemStore which is faster than RDBMS (Relational databases).
* Later, the data is dumped in HFile, where the actual data is stored in HDFS.&#x20;
*   If the MemCache is full, the data is stored in

    HFile directly.
* Once writing data is completed, ACK (Acknowledgement) is sent to client as a confirmation of task completed.

Issue: during update/insert WAL writes on HDFS requires network traffic -> slow

Client can control WAL strategies per mutation

* USE\_DEFAULT: do whatever the default is for the table. This is the client default.
* SKIP\_WAL: do not write to the WAL
* ASYNC\_WAL: write to the WAL asynchronously as soon as possible
* SYNC\_WAL: write WAL synchronously&#x20;
* FSYNC\_WAL: write to the WAL synchronously and guarantee that the edit is on disk (not currently supported)
* The MemStore stores updates in memory as sorted KeyValues, the same as it would be stored in an HFile.
* There is one MemStore per column family.&#x20;
* The updates are sorted per column family.
* When the MemStore accumulates enough data, the entire sorted set is written to a new HFile in HDFS.&#x20;
* HBase uses multiple HFiles per column family, which contain the actual cells, or KeyValue instances.&#x20;
* These files are created over time as KeyValue edits sorted in the MemStores are flushed as files to disk.

Note that this is one reason why there is a limit to the number of column families in HBase.

* There is one MemStore per CF;&#x20;
*   when one is full, they all flush. It also saves the last written sequence number so the system knows what was

    persisted so far.
* The highest sequence number is stored as a meta field in each HFile, to reflect where persisting has ended and where to continue.&#x20;
* On region startup, the sequence number is read, and the highest is used as the sequence number for new edits.
* Data is stored in an HFile which contains sorted key/values.
*   When the MemStore accumulates enough data, the entire

    sorted KeyValue set is written to a new HFile in HDFS.&#x20;
* This is a sequential write. It is very fast, as it avoids moving the disk drive head.

## HFile

* contains a multi-layered index which allows HBase to seek to the data without having to read the whole file.&#x20;
* The multi-level index is like a b-tree: Key value pairs are stored in increasing order
* Indexes point by row key to the key value data in 64KB “blocks”
* Each block has its own leaf-index
* The last key of each block is put in the intermediate index
* The root index points to the intermediate index
* The trailer points to the meta blocks, and is written at the end of persisting the data to the file. The trailer also has information like bloom filters and time range info.&#x20;
* Bloom filters help to skip files that do not contain a certain row key.
* The time range info is useful for skipping the file if it is not in the time range the read is looking for.

## Read

* client sends request to Hbase.&#x20;
* A request is sent to zookeeper which keeps all the status for the distributed system, where HBase is also present.&#x20;
* Zookeeper has location for META table which is present in HRegion Server.&#x20;
* When a client requests zookeeper, it gives the address for the table (1).
* The process continues to HRegionServer and gets to META table, where it gets the region address of table where the data is present to be read (2).
* Moving forward to a specific HRegion, the process enters the BlockCache where data is present from previous read.&#x20;
* If a user queries the same records, the client will get the same data in no time.&#x20;
* If the table is found, the process returns to client with the data as result (3).
*   If the table is not found, the process starts to search MemStore since data would have been written to HFile sometime

    back.&#x20;
* If it is found, the process returns to client with the data as result (4).
* If the table is not found, the process moves forward in search of data within the HFile.&#x20;
* The data will be located here and once the search is completed, the process takes required data and moves forward (5).
* The data taken from HFile is the latest read data and can be read by user again.&#x20;
* Hence the data is written in BlockCache, so that the next time, it can be instantly accessed by the client (6).&#x20;
* the read process can be completed just after step (3) the next time for the same data because of this read procedure of Hbase.
*   When the data is written in BlockCache and all the search is completed, the read process with required data will be

    returned to client along with ACK(Acknowledgment) (7).

Block cache

* keeps data blocks resident in memory after they’re read
* One blockcachce per regionserver
* Different implementations
  * LRUBlockCache: on the heap, least recently used
  * Slabcache: off heap, no GC overhead, LRU
  * BucketCache: heap OR offheap OR file(ssd). 14 buckets allocated, with different size each containing multiple buckets

### HBase Read Merge

* KeyValue cells corresponding to one row can be in multiple places
* row cells already persisted are in Hfiles,&#x20;
* recently updated cells are in the MemStore, &#x20;
* recently read cells are in the Block cache.&#x20;

When you read a row, how does the system get the corresponding cells to return?

* A Read merges Key Values from the block cache, MemStore, and HFiles in the following steps:
* the scanner looks for the Row cells in the Block cache - the read cache. Recently Read Key Values are cached here, and Least Recently Used are evicted when memory is needed.
* the scanner looks in the MemStore, the write cache in memory containing the most recent writes.
* If the scanner does not find all of the row cells in the MemStore and Block Cache, then HBase will use the Block Cache indexes and bloom filters to load HFiles into memory, which may contain the target row cells.
* there may be many HFiles per MemStore, which means for a read, multiple files may have to be examined, which can affect the performance.
* This is called read amplification.
* HFile Index - The index is loaded when the HFile is opened and kept in memory. This allows lookups to be performed with a single disk seek.

## Compaction

### HBase Minor Compaction

* HBase will automatically pick some smaller HFiles and rewrite them into fewer bigger Hfiles.&#x20;
*   it reduces the number of storage files by rewriting smaller files into fewer but larger ones,

    performing a merge sort.

### HBase Major Compaction

*   it merges and rewrites all the HFiles in a region to one HFile per column family, and in the process, drops

    deleted or expired cells.&#x20;
*   it improves read performance; however, since major compaction rewrites all of the files, lots

    of disk I/O and network traffic might occur during the process -it's write amplification.
* it can be scheduled to run automatically.&#x20;
* Due to write amplification, major compactions are usually scheduled for weekends or evenings.&#x20;
* it also makes any data files that were remote, due to server failure or load balancing, local to the region server.

## Read load balancing

*   Splitting happens initially on the same region server, but for load balancing reasons, the HMaster may schedule for new

    regions to be moved off to other servers.&#x20;
* This results in the new Region server serving data from a remote HDFS node until a major compaction moves the data files to the Regions server’s local node.&#x20;
* HBase data is local when it is written, but when a region is moved (for load balancing or recovery), it is not local until major compaction

## Backup RegionServers

* data is partitioned and each key belongs to one region server
* each key has a primary region server and back up region server
* When client reads, it can read older data from a backup RS
* Timeline consistency : while replicas may not be consistent with each other, updates are guaranteed to be applied in the same order at all replicas
* You can choose runtime: strong OR timeline consistency

## Data replication

* All writes and Reads are to/from the primary node.&#x20;
* The WAL file and the Hfiles are persisted on disk and replicated.&#x20;
* HDFS replicates the WAL and HFile blocks.&#x20;
* HFile block replication happens automatically. HBase relies on HDFS to provide the data safety as it stores its files.&#x20;
* When data is written in HDFS, one copy is written locally, and then it is replicated to a secondary node, and a third copy is written to a tertiary node.

So how does HBase recover the MemStore updates not persisted to HFiles?

## Crash recovery

* Zookeeper determines Node failure when it loses region server heart beats.&#x20;
* HMaster will be notified that Region Server has failed.
* HMaster reassigns the regions from the crashed server to active Region servers.&#x20;
* In order to recover the crashed region server’s memstore edits that were not flushed to disk, The HMaster splits the WAL belonging to the crashed region server into separate files and stores these file in the new region servers’ data nodes.&#x20;
* Each Region Server then replays the WAL from the respective split WAL, to rebuild the memstore for that region.

### Data Recovery

* WAL files contain a list of edits, with one edit representing a single put or delete.&#x20;
* Edits are written chronologically, so, for persistence, additions are appended to the end of the WAL file that is stored on disk.
* What happens if there is a failure when the data is still in memory and not persisted to an HFile? The WAL is replayed.
* Replaying a WAL is done by reading the WAL, adding and sorting the contained edits to the current MemStore. At the end, the MemStore is flush to write changes to an HFile.

## Coprocessors

The coprocessor framework provides mechanisms for running your custom code directly on the RegionServers managing your data.

Coprocessor Analogies

* Triggers and Stored Procedure. An Observer coprocessor is similar to a trigger in a RDBMS in that it executes your code either before or after a specific event (such as a Get or Put) occurs. An endpoint coprocessor is similar to a stored procedure in a RDBMS because it allows you to perform custom computations on the data on the RegionServer itself, rather than on the client.
* MapReduce. MapReduce operates on the principle of moving the computation to the location of the data. Coprocessors operate on the same principal.
* AOP. you can think of a coprocessor as applying advice by intercepting a request and then running some custom code, before passing the request on to its final destination (or even changing the destination).
* Your class should implement one of the Coprocessor interfaces - Coprocessor, RegionObserver, CoprocessorService - to name a few.
* Load the coprocessor, either statically (from the configuration) or dynamically, using HBase Shell.&#x20;
* Call the coprocessor from your client-side code. HBase handles the coprocessor transparently.

Types of Coprocessors 1. Observer Coprocessors - are triggered either before or after a specific event occurs. Observers that happen before an event use methods that start with a pre prefix, such as prePut. Observers that happen just after an event override methods that start with a post prefix, such as postPut. Exapmles: Before performing a Get or Put operation, you can check for permission using preGet or prePut methods. OR: if you have a business rule that every insert to the `users` table must be followed by a corresponding entry in the `user_daily_attendance` table, you could implement a coprocessor to use the prePut method on user to insert a record into `user_daily_attendance`. 2. Endpoint Coprocessor - allow you to perform computation at the location of the data. An example is the need to calculate a running average or summation for an entire table which spans hundreds of regions.

## Bloom filters

* named for its creator, Burton Howard Bloom, it's a data structure which is designed to predict whether a given element is a member of a set of data.&#x20;
* A positive result from a Bloom filter is not always accurate, but a negative result is guaranteed to be accurate.&#x20;
* Bloom filters are designed to be "accurate enough" for sets of data which are so large that conventional hashing mechanisms would be impractical.&#x20;
* provide a lightweight in-memory structure to reduce the number of disk reads for a given Get operation (Bloom filters do not work with Scans) to only the StoreFiles likely to contain the desired Row. The potential performance gain increases with the number of parallel reads.
* Bloom filters themselves are stored in the metadata of each HFile and never need to be updated. When an HFile is opened because a region is deployed to a RegionServer, the Bloom filter is loaded into memory.
* row-based Bloom filters are enabled by default
* If Bloom filters are enabled, the value of `blockCacheHitRatio` in the RegionServer metrics should increase, because the Bloom filter is filtering out blocks that are definitely not needed.
* Bloom filters are enabled on a Column Family.
*   Valid values are NONE, ROW (default), or ROWCOL

    ```
    hbase> create 'mytable',{NAME => 'colfam1', BLOOMFILTER => 'ROWCOL'}
    ```

## Where to put a Value?

Key OR Column-name OR Value

* Hbase folds the row into a single column table on disc.
* You can shift value to column key or even to rowkey
* It will have the same storage requirement !
* If you have info which required to be access fast, put it in the key. If you need the data rarely, put in the value!

example: RK : value OR RK : CF : value or RK : CF : qualifier (column descriptor/name) : timestamp : value

## Read performance

* Timestamp is stored with each file. If you know you need the data only from 24h back, then Hbase can skip entire Hfiles
* If you know which column family to use, specify it&#x20;
* Pure value based filtering is a full table scan
* Best is to use the row key: best performance
* Bloom Filters can skip store files
* Filters often are a full table scan too, but reduce network traffic

## Use cases for HBase

*   Internet of Things (IOT) applications – Hbase is perfect for consuming and analyzing lots of fast-incoming data from

    devices, sensors and similar mechanisms that exist in many different locations.
*   Product catalogs and retail apps - For retailers that need durable shopping cart protection, fast product catalog input

    and lookups, and similar retail application support, HBase is the database of choice.
*   User activity tracking and monitoring - Media, gaming and entertainment companies use HBase to track and monitor

    the activity of users' interactions with their movies, music, games, website and online applications.
*   Messaging apps - HBase serves as the database backbone for numerous mobile phone, telecommunication,

    cable/wireless, and messaging providers' applications.
*   Social media analytics and recommendation engines - Online companies, websites, and social media providers use

    HBase to ingest, analyze, and provide analysis and recommendations to their customers.

## Tools

* hbck

> fsck for HBase
>
> ```
> hbase hbck -j /path/to/HBCK2.jar
> ```
>
> * Hfile Tool
>
> To view a textualized version of HFile content
>
> ```
> hbase hfile -v -f hdfs://10.81.47.41:8020/hbase/default/TEST/1418428042/DSMP/475950861828684547
> ```
>
> * WAL Tools :  WAL player and WALPrettyPrinter
>
> WALPrettyPrinter - is a tool with configurable options to print the contents of a WAL or a recovered.edits file. You can invoke it via the HBase cli with the 'wal' command.

* CompressionTest Tool

> You can use the CompressionTest tool to verify that your compressor is available to HBase:
>
> ```
> $ hbase org.apache.hadoop.hbase.util.CompressionTest hdfs://host/path/to/hbase snappy
> ```

* CopyTable

> a utility that can copy part or of all of a table, either to the same cluster or another cluster. The usage is as follows:
>
> \`\`\` $ hbase org.apache.hadoop.hbase.mapreduce.CopyTable \[--starttime=X] \[--endtime=Y] \[--new.name=NEW] \[--peer.adr=ADR] tablename

hbase org.apache.hadoop.hbase.mapreduce.CopyTable --starttime=1265875194289 --endtime=1265878794289 --peer.adr=server1,server2,server3:2181:/hbase TestTable

```
- Export

> a utility that will dump the contents of table to HDFS in a sequence file.
```

$ hbase org.apache.hadoop.hbase.mapreduce.Export   \[ \[ \[]]]

```
- ImportTsv

> a utility that will load data in TSV format into HBase. It has two distinct usages: 
- loading data from TSV format in HDFS into HBase via Puts,
- preparing StoreFiles to be loaded via the completebulkload.
To load data via Puts (non-bulk loading):
```

$ hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns=a,b,c &#x20;

```
To generate StoreFiles for bulk-loading:
```

$ hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns=a,b,c -Dimporttsv.bulk.output=hdfs://storefile-outputdir &#x20;

```
- CompleteBulkLoad

> utility will move generated StoreFiles into an HBase table. This utility is often used in
conjunction with output from "ImportTsv".
two ways to invoke, with explicit classname and via the driver:
```

$ hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles [hdfs://storefileoutput](hdfs://storefileoutput)&#x20;

```
```

HADOOP\_CLASSPATH=`${HBASE_HOME}/bin/hbase classpath` ${HADOOP\_HOME}/bin/hadoop jar ${HBASE\_HOME}/hbase-VERSION.jar completebulkload [hdfs://storefileoutput](hdfs://storefileoutput)&#x20;

```
- WALPlayer

> a utility to replay WAL files into HBase.The WAL can be replayed for a set of tables or all tables, and a timerange can be provided (in milliseconds). The WAL is filtered to this set of tables. The output can optionally be mapped to another set of tables. WALPlayer can also generate HFiles for later bulk importing, in that case only a single table and no mapping can be specified.
```

$ hbase org.apache.hadoop.hbase.mapreduce.WALPlayer \[options]   \[]>

```
```

$ hbase org.apache.hadoop.hbase.mapreduce.WALPlayer /backuplogdir oldTable1,oldTable2 newTable1,newTable2

` `` WALPlayer, by default, runs as a mapreduce job. To NOT run WALPlayer as a mapreduce job on your cluster, force it to run all in the local process by adding the flags `-Dmapred.job.tracker=local\` on the command line.
