# Mongo

## Mongo

* document DB
* object-oriented
* json
* high-performance
* highly scalable
* no strong consistency guarantees like hbase, just eventual
  * hbase - shared-disk architecture
  * mongo - shared-nothing architecture, depends on replication between nodes to sync data in cluster

## Document

json:

* string ""
* number
* boolean - true/false
* object {}
* array \[]
* null -

## CRUD

* no data definition language, no schema
* will accept any typos and misspellings
* database creates if not exists, same for collection
* collection - table, group of related documents
* document - row
* show collections
* all commands are javascript functions
* connect: `mongo <DBname>`

### Insert

* `db.<collectionName>.insert ({<object>}})`
* `db.<collectionName>.find()` - show documents
* `db.<collectionName>.count()` - num of docs
* `_id` field - 12 byte hexadecimal
  * 4 byte timestamp (precise to 1 sec)
  * 3 byte machine ID
  * 2 byte process ID
  * 3 byte increment counter

### Selection

* `db.<collectionName>.find({_id: ObjectID("...")})` - show by condition, boolean test of equality. Here can be any field
*   $lt - < less then, $gt - > greater then, $ne - not equal, $in

    ```
    db.<collectionName>.find({hourse: {$lt : 40}})
    db.<collectionName>.find({"address.state": {$ne : "TX"}})
    db.<collectionName>.find({"address.state": {$in : ["TX", "TN"]}})
    db.<collectionName>.find({$or:[{"address.state": "TX"}, {"address.state": "TN"}]})
    ```
*   // for regex

    ```
    db.<collectionName>.find({"address.street": /Ave/})
    ```

#### Projection

* `db.<collectionName>.find({<query>}, {<projection>})`
* exclusive - project everything except exclude {attribute: 0}
* inclusive - project only what we mention {attribute: 1}
*   `_id` is returned always except ypu exclude it

    ```
    db.<collectionName>.find({"address.street": /Ave/}, {name:true, address:1, _id:0})
    ```
* {} - blank object in query placeholder - means no filtration, all documents

### update

```
db.<collectionName>.update({<query>}, {<update>} [, upsert, multi])
```

* upsert - insert if not exists
* multi - allow multiple docs to be updated at once
*   updating is for whole doc, not the field!

    \`\`\`

    db..update({name: "Peter"}, {$set:{"address.state": "AZ"\}} )

db..update({"address.state": "AZ"}, {$set:{"address.state": "Arizona"\}}, false, true ) - for multi update

```
- $set - a set field to value
- $unset - remove field
- $inc - increment
- $pop - remove last element from array
- $push - adds element to array
- $pull, $pullAll - remove mathcing elements from array

## remove

- `db.<collectionName>.remove({})` - no safety
- `db.<collectionName>.drop()`
- `db.dropDatabase()`    

# import csv, json
run from cli:
```

mongoimport --db= --collection= --file=

```
> `--legacy` flag for older json version
> `--type=csv --headerline` - for csv with header

# advanced queries
- $and or just colnames  within comma
- $nin
- $group, $match, $sort
- $sum, $avg, $min, $max, __NO__ $count -  use "$sum":1
```

db.orders.aggregate(\[ { $match: { status: "A" } }, { $group: { \_id: "$cust\_id", total: { "$sum": "$amount" } } } ])

\`\`\`

First Stage: The $match stage filters the documents by the status field and passes to the next stage those documents that have status equal to "A".

Second Stage: The $group stage groups the documents by the cust\_id field to calculate the sum of the amount for each unique cust\_id.

## performance & architecture

* one primary server & any numer of secondary
* only primary is writable
* odd number of servers - better
* partition tolerance with odd number of servers - clearly identifies majority

four essential capabilities in meeting modern application needs:

* Availability
* Workload isolation
* Scalability
* Data locality

### Availability

* maintains multiple copies of data using replica sets.
* replica sets are self-healing as failover and recovery is fully automated

### Workload Isolation

* replica sets also provide a foundation for combining different classes of workload on the same MongoDB cluster, each operating against its own copy of the data.
* can run exploratory queries and generate reports, and data scientists can build machine learning models without impacting operational applications.

### Scalability

* supports horizontal scaling through sharding.&#x20;
* Sharding automatically partitions and distributes data across multiple physical instances called shards.&#x20;
* Each shard is backed by a replica set to provide always-on availability and workload isolation.&#x20;
* Sharding allows to seamlessly scale the database as their apps grow beyond the hardware limits of a single server, and it does this without adding complexity to the application.&#x20;
* nodes can be added or removed from the cluster in real time, and MongoDB will automatically rebalance the data accordingly, without manual intervention.&#x20;
* Sharding is transparent to applications; whether there is one or a thousand shards, the application code for querying MongoDB remains the same.&#x20;
* Applications issue queries to a query router that dispatches the query to the appropriate shards.

### Data Locality

* zoned sharding allows precise control over where data is physically stored in a cluster.&#x20;
* allows to accommodate a range of application needs – for example controlling data placement by geographic region for latency and governance requirements, or by hardware configuration and application feature to meet a specific class of service.

## Indexes

* Create Indexes to Support Your Queries

An index supports a query when the index contains all the fields scanned by the query. Creating indexes that support queries results in greatly increased query performance.

* Use Indexes to Sort Query Results

To support efficient queries, use the strategies here when you specify the sequential order and sort order of index fields.

* Ensure Indexes Fit in RAM

When your index fits in RAM, the system can avoid reading the index from disk and you get the fastest processing.

* Create Queries that Ensure Selectivity

Selectivity is the ability of a query to narrow results using the index. Selectivity allows MongoDB to use the index for a larger portion of the work associated with fulfilling the query.

* MongoDB uses the shard key to distribute the collection’s documents across shards.
* MongoDB distributes the read and write workload across the shards in the sharded cluster, allowing each shard to process a subset of cluster operations.&#x20;
* Both read and write workloads can be scaled horizontally across the cluster by adding more shards.
* For queries that include the shard key or the prefix of a compound shard key, mongos can target the query at a specific shard or set of shards.&#x20;
* These targeted operations are generally more efficient than broadcasting to every shard in the cluster.
* Sharding distributes data across the shards in the cluster, allowing each shard to contain a subset of the total cluster data.&#x20;
* As the data set grows, additional shards increase the storage capacity of the cluster.
* The deployment of config servers and shards as replica sets provide increased availability.

## Sharing strategies

* Hashed Sharding involves computing a hash of the shard key field’s value. Each chunk is then assigned a range based on the hashed shard key values.
* Ranged sharding involves dividing data into ranges based on the shard key values. Each chunk is then assigned a range based on the shard key values.
* Zones can help improve the locality of data for sharded clusters that span multiple data centers.
* In sharded clusters, you can create zones of sharded data based on the shard key. You can associate each zone with one or more shards in the cluster. A shard can associate with any number of zones. In a balanced cluster, MongoDB migrates chunks covered by a zone only to those shards associated with the zone.

## Sharded Cluster

consists of the following components:

* shard: Each shard contains a subset of the sharded data. Each shard can be deployed as a replica set. replica set is a cluster of MongoDB servers that implements replication and automated failover.&#x20;
* mongos: The mongos acts as a query router, providing an interface between client applications and the sharded cluster. Starting in MongoDB 4.4, mongos can support hedged reads to minimize latencies.
* config servers: Config servers store metadata and configuration settings for the cluster.

## Storage Engines

The storage engine is the component of the database that is responsible for managing how data is stored, both in memory and on disk.

Starting in version 4.2, MongoDB removes the deprecated MMAPv1 storage engine.

### WiredTiger Storage Engine (Default)

WiredTiger is the default storage engine starting in MongoDB 3.2. It is well-suited for most workloads and is recommended for new deployments. WiredTiger provides a document-level concurrency model, checkpointing, and compression, among other features.

In MongoDB Enterprise, WiredTiger also supports Encryption at Rest.

### In-Memory Storage Engine

In-Memory Storage Engine is available in MongoDB Enterprise. Rather than storing documents on-disk, it retains them in-memory for more predictable data latencies.

## GridFS

GridFS is a specification for storing and retrieving files that exceed the BSON-document size limit of 16 MB.

* for storing files larger than 16 MB.
* for storing any files for which you want access without having to load the entire file into memory.
* If your filesystem limits the number of files in a directory
* When you want to keep your files and metadata automatically synced and deployed across a number of systems and facilities
* GridFS divides the file into parts, or chunks \[1], and stores each chunk as a separate document.
* By default, GridFS uses a default chunk size of 255 kB;&#x20;
* GridFS divides a file into chunks of 255 kB with the exception of the last chunk. The last chunk is only as large as necessary.&#x20;
* files that are no larger than the chunk size only have a final chunk, using only as much space as needed plus some additional metadata.
* When you query GridFS for a file, the driver will reassemble the chunks as needed.&#x20;
* You can perform range queries on files stored through GridFS.&#x20;
* You can also access information from arbitrary sections of files, such as to “skip” to the middle of a video or audio file.

GridFS stores files in two collections:

* chunks stores the binary chunks.
* files stores the file’s metadata.&#x20;

GridFS places the collections in a common bucket by prefixing each with the bucket name.

By default, GridFS uses two collections with a bucket named fs:

* fs.files
* fs.chunks

## Replication

* single master architecture
* primary DB
* secondary  - only copies of primary DB: consistency over availability
* a replica set = 1 primary server + several secondary backup nodes
* if primary goes down, one of secondary takes place.
* auto replication from primary to secondary
* majority of servers must agree on the primary election, even number of server don't work => at least 3 servers
* can set an arbiter node (only one) to elect primary if you don't want 3 servers
* app should know about enugh servers in replica set to be able to reach one to learn who's primary. If you change cluster configureation (e.g. add secondaries) - the app should be provided with these info
* read from secondaries not recommended
* DB will go to read-only mode while new primary is elected
* delayed secondary - can set amount of time between promary and secondary (e.g. 1 hour) to restore 1 hour ago DB copy in case of failure

Sharding:

![](.gitbook/assets/mongo\_sharding.png)

* in big cluster there are multiple replica sets
* each replica set is responsible for some range of data based on some index. That index is necessary to balance data across replica sets
* you can have multiple app server processes. On these servers, there are mongos processes, that talk to a collection of configuration servers - they know how data is partitioned and which replica set you need.
* mongos runs a load balancer - can rebalance data based on chosen index (e.g. user id like in picture)
* auto-sharding sometimes fails - can't rebalance if config servers are down or it even doesn't start if you restart mongos process too often
* at least 3 config servers prior to 3.2 version, now - config server are part of replica ser, it just has to have a primary config server
* you nedd to use a shard key with high cardinality
* The primary node receives all write operations.
* A replica set can have only one primary capable of confirming writes with { w: "majority" } write concern;&#x20;
* another mongod instance may transiently believe itself to also be primary.&#x20;
* The primary records all changes to its data sets in its operation log, i.e. oplog.&#x20;

**A priority 0 member** is a member that cannot become primary and cannot trigger elections.

* Other than the aforementioned restrictions, secondaries that have priority 0 function as normal secondaries: they maintains a copy of the data set, accept read operations, and vote in elections.
* Configure a secondary to have priority 0 to prevent it from becoming primary, which is particularly useful in multi-data center deployments.
* The secondaries replicate the primary’s oplog and apply the operations to their data sets such that the secondaries’ data sets reflect the primary’s data set. If the primary is unavailable, an eligible secondary will hold an election to elect itself the new primary.
* You may add an extra mongod instance to a replica set as an **arbiter**. Arbiters do not maintain a data set. The purpose of an arbiter is to maintain a quorum in a replica set by responding to heartbeat and election requests by other replica set members.&#x20;
* Because they do not store a data set, arbiters can be a good way to provide replica set quorum functionality with a cheaper resource cost than a fully functional replica set member with a data set.
* If your replica set has an even number of members, add an arbiter to obtain a majority of votes in an election for primary. Arbiters do not require dedicated hardware.
* A **hidden member** maintains a copy of the primary’s data set but is invisible to client applications.
* Hidden members are good for workloads with different usage patterns from the other members in the replica set.&#x20;
* Hidden members must always be priority 0 members and so cannot become primary.&#x20;
* Hidden members may vote in elections.
* **Delayed members** contain copies of a replica set’s data set.
* delayed member’s data set reflects an earlier, or delayed, state of the set. For example, if the current time is 09:52 and a member has a delay of an hour, the delayed member has no operation more recent than 08:52.
* Because delayed members are a “rolling backup” or a running “historical” snapshot of the data set, they may help you recover from various kinds of human error.&#x20;
* For example, a delayed member can make it possible to recover from unsuccessful application upgrades and operator errors including dropped databases and collections.
* Must be priority 0 members. Set the priority to 0 to prevent a delayed member from becoming primary.
* Should be hidden members. Always prevent applications from seeing and querying delayed members.
* vote in elections for primary, if members\[n].votes is set to 1.

## Use cases

* single view
* IOT
* Real-time analytics
* Mobile
* Catalog
* Personalization
* Content management

## Differences between mongo, cassandra and hbase

1. Hbase - master-slave architecture, cassandra - all nodes are equal, coordinator node only for query, mongo - single master architecture, primary DB
2. mongo - document db, hbase and cassandra - column store
3. cassandra - cql like sql, mongo - javascript shell support & read-only SQL queries via the MongoDB Connector for BI, hbase - no sql
4. cassandra - AP db, hbase & mongo - CP db
5. seconadary indexing - yes for mongo, cassandra - only for equality queries, hbase - no
6. acid: cassandra - atomicity & isolation for single operators, hbase - acid for single row, mongo - multi-document acid with snapshot isolation
7. replication: cassandra & hbase - selective RF, mongo - master-slave replication
8. data model: hbase - Tables, Rows, Column Families, Columns, Cells and Versions; cassandra - keyspaces, tables (column families), columns; mongo - collection, document, field&#x20;
