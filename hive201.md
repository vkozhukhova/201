# Hive part 2

## UDFs

UDF language = jvm based or python

3 types

* standard
* aggregation \(UDAF\)
* table generating \(UDTF\)

### Standard

* takes a row arguments: one/more columns values, constants
* returns single value

### UDAF

* 1 or more arguments from one or many rows
* returns single value

### UDTF

* takes a row arguments: one/more columns values, constants
* returns 0 or many new rows

## Create UDF

* class extends UDF or GenericUDTF
* implement evaluate method for UDF and initialize, process, close for UDTF
* ADD JAR jarfile.jar;
* CREATE TEMPORARY FUNCTION  AS 'name of class in jar';
* initialize: argument types to expect, return object inspector corresponding to row objects that udtf will generate
* process:    produce rows to other operators calling forward
* close

## Incremental updates

how update hive table?

* record id: timestamp, delete date
* condition for merging new and existing data
* way of determining which data is new and which should be deleted from table
* multiple records with same if =&gt; take only latest to avoid duplicates

### Components:

* Master table – an internal table, the main table containing the records. This table users query for data
* Delta table – an external table, contains the updated or added records only
* Reconciliation View – Reflects the final record set, a Hive view which joins the master and delta tables via UNION ALL operation. May include INNER JOIN to himself to include only latest records with same id, may support deleting by support omitting records in view. It's a logical object, no storage. Need to create temp table from it. 
* Reporting Table – a snapshot of the Reconciliation View.  The view changes as the delta table changes, so we need to write a copy of the data into a new table created from the view \(CREATE TABLE reporting\_table AS SELECT \* FROM reconciliation\_view\).

Then recreate master table from reporting table. Remove delta data from HDFS.

#### The master table does not exist

* Import data to HDFS
* Create the internal Master table by inserting the data from HDFS.
* Done

#### The master table already exists

* Import the updated data to HDFS
* Create an external Delta table pointing to the location of the new data
* Create a Reconciliation view as a UNION select from the Master and Delta tables
* Create an internal Reporting table as SELECT \* FROM reconciliation\_view.
* Remove data from the Delta table by removing the files from HDFS.
* Recreate the Master table and initialize it with the data from the Reporting table
* Done

```sql
CREATE VIEW reconciliation_view AS
    SELECT delta1.* FROM
        (SELECT * FROM master_table
         UNION
         SELECT * FROM delta_table) delta1
     JOIN
         (SELECT id, max(timestamp) max_timestamp FROM
             (SELECT * FROM master_table
              UNION
              SELECT * FROM delta_table) 
      GROUP BY id) delta2
   ON delta1.id = delta2.id AND delta1.timestamp = delta2.max_timestamp
   AND delta1.delete_date IS NULL;
```

### New MERGE command Hive 2.2

* master table has to be transactional
* any number of `when mathched` and `when not matched` statements followed by insert, update, delete operations.

```sql
merge into master
 using delta on master.id = delta.id
when matched then update set
   country=delta.country,
   state=delta.state
when matched and delete_date is not NULL then delete
when not matched then insert
   values(delta.id, delta.country, delta.state);
```

## ACID

* transactions must be enabled globally
* HDP: switch in Ambari on config page
* manually??

### Transactional table

* table need to be internal
* bucketed
* stored in orc
* `tblproperties('transactional'='true')`
* partitioning no required but encouraged! - speeds up operations
* table is transactional forever

```sql
CREATE TABLE transactional_table (id int, username string) PARTITIONED BY (timestamp_date date)
CLUSTERED BY(username) INTO 3 BUCKETS
STORED AS ORC TBLPROPERTIES ('transactional'='true');
```

### Properties

* ACID disabled by default
* full ACID semantica on row level: one app read data, other updates rows
* auto-commit
* no support for BEGIN, COMMIT, ROLLBACK
* ABORT is there
* client send heartbeat to hive =&gt; abort transaction automatically if not present
* Snapshot level isolation. Each query is provided with snapshot of data
* LOAD DATA is not supported when transactions are enabled
* Required for UPDATE, DELETE, MERGE
* Optimistic concurrency. ACID updates and deletes are resolved by having first commit to win. Happens on partition level or table level for unpartitioned tables
* Designed for updates in bulk. Each transaction creates new files in HDFS which are periodically compacted. Each select reads data from these files. Updates should be performed only in large batches
* Partitioning is recommended for large datasets
* Are periodically compacted. Each transaction creates a delta file to save updates. These files are merges for compaction from time to time.

### level components of ACID transactions in Hive

* base files - main files containing the table data
* delta files - created for each UPDATE and DELETE command
* compactor
* transaction/lock manager

#### compactor

* keeps number of delta files low, 
* merges Delta and Base files for each bucket. 
* Run background MR jobs to compact delta and base files. 
* 2 types: Major and Minor compaction. 
* Minor compaction - merges many small delta files into 1 big delta file. 
* Major compaction - takes delta files and merges with base files, creates new file, removes old.
* Compaction is done for each bucket separately. 
* Delta and base files are created per bucket. 
* we Can set how often cpmpaction runs, or max number of background jobs to run to compact tables. 
* Queries are long with many uncompacted files. 
* Update and delete create new Delta files with changes, they don't update existiong delta files.
* Queries base files and delta files and combinig them before returning results

#### transaction/lock manager

* manages transaction locks in Hive Metastore. Transactions & locks are durable and are restored in case of server failure. transaction/lock manager is stored in metastore, transaction manager keeps state \(open, commit, abort\), lock manager maintains locks for transactional tables. Transaction initiators and lock holders send heartbeat to metastore - to remove any abandoned transactions and locks.

### Commands for transactions

* SHOW TRANSACTIONS - returns a list of all currently open and aborted transactions. Id, state, who started transaction, machine, start timestamp, last heartbeat time.
* ABORT TRANSACTIONS - cleans up the specified transaction IDs from the Metastore. ACID queries periodically heartbeat and if detect that underneath transaction was aborted, they will exit.
* SHOW LOCKS - displays the locks on a table or partition. Was there even before transactions had been introduced but now shows additional information for transactional tables. Lock can be shared, read/write or exclusive
* SHOW COMPACTIONS – returns a list of all tables and partitions currently being compacted or scheduled for compaction. Shows compaction type, database, table, state, starttime, duration

## Hive statictics

### EXPLAIN command

* `EXPLAIN` shows execution plan for the query
* `extended` clause produce extra info about operators in the plan: physical info like filenames
* query is converted into sequence \(DAG\) of stages
* stages are MR stages or stages doing metastore of filesystem operations \(move, rename\)
* output of command has 3 parts: 
  * abstract syntax tree for the query, 
  * stage dependencies for stages of the plan
  * description of each stage: sequence of operators with metadata asscociated with operators. Metadata may include things like filter expressions for filter operator, select expressions for select operator, filenames for sink operator.
* `dependency` clause: produces extra info about inputs in plan. Shows variuos attributes for inputs. Inputs contain tables and partitions. Table is present even of no partitions is accessed in query. Shows parents if table accessed via view
* `authorisation` clause: shows all entties needed to be aythorised to execute query and authorisation failures oi any exist.

```text
EXPLAIN [EXTENDED|DEPENDENCY|AUTHORIZATION] query
```

```text
EXPLAIN
FROM src INSERT OVERWRITE TABLE dest_g1 SELECT src.key, sum(substr(src.value,4)) GROUP BY src.key;
```

### Statistics

* number or rows in table/partition, column histo
* use for query optimization
* it's input for cost functions of optimizer =&gt; can compare different plans and choose among them
* users can query statistica instead of run long queries on tables.
* quantile of users' age distribution, top 10 apps used by people & number of distinct sessions.
* stats is stored in metastore
* for partitions: number of rows, files, size in bytes
* for tables: same + number of partitions;
* column level stats: top K levels. disabled by default.

For newly created tables and/or partitions \(that are populated through the INSERT OVERWRITE command\), statistics are automatically computed by default. The user has to explicitly set the boolean variable `hive.stats.autogather` to false so that statistics are not automatically computed and stored into Hive MetaStore.

For existing tables and/or partitions, the user can issue the ANALYZE command to gather statistics and write them into Hive MetaStore.

```text
ANALYZE TABLE Table1 PARTITION(ds='2008-04-09', hr) COMPUTE STATISTICS FOR COLUMNS;

ANALYZE TABLE Table1 COMPUTE STATISTICS;
```

The name and top K values of the most skewed column is stored in the partition or non-partitioned table’s skewed information, if user did not specify skew. This works for both newly created and existing tables.

Top K statistics are disabled by default. The user can set the boolean variable hive.stats.topk.collect to be true to enable computing top K and putting top K into skewed information.

`set hive.stats.topk.collect=true;` The user can also specify the number of K by setting the integer variable `hive.stats.topk.num`, and the minimal row percentage that a value needs to hold to be in the top K result, by setting the float variable `hive.stats.topk.minpercent`.

`set hive.stats.topk.num=8;` `et hive.stats.topk.minpercent=5.0;` Another integer variable, hive.stats.topk.poolsize, specifies the number of values to be monitored while computing top K. The accuracy of top K estimate increases as this number gets larger.

`set hive.stats.topk.poolsize=200;` Computing top K for a large number of partitions simultaneously can be stressful to memory. The user can specify the integer variable hive.stats.topk.maxpartnum for the maximal number of partitions to collect Top K. When this number is exceeded, top K will be disabled for all the remaining partitions.

`set hive.stats.topk.maxpartnum=10;`

## Schema optimization

* normalization: several tables combine at runtime to get results
* joins are expensive and difficult
* less emphasis on normalisation compared to RDBMS
* The use-cases for Hive are not transactional but analytic.
* The many-way joins required for normalized data tend to be more expensive for Hive.
* Hive’s massive parallelism eliminates many of the disk-I/O limitations of an RDBMS.
* Hive is often used with data volumes for which it would impractical to use normalized data. 
* The savings in data volume are often more than offset by the high compression rates in Hive. 
* Abandoned from CPU and absence of central processing bottleneck makes compression and decompression relatively cheap
* hive can work with denormalised data like lists or maps within column

## Accumulo integration

* sorted, distributed key-value store, based on google big table paper. API is based on key-value operations, high level flexibility to read/write. high level query abstractions - exercise for user. high throughput access, low latency

## HBase integration

* hadoop DB, distributed, scalable bigdata store. Hive can access hbase tables and join with hive tables

## Druid

* open source analytics store for BI OLAP queries. Low latency, real time data ingestion, flexible data exploration, fast data aggregation. thrillions of events, pethabytes of data. Index data from hive into druid =&gt; efficient execution olap queries in hive, sql ontop of druid, index complex query results

## Best practices of hive config

* execution engine -  globally introduces, but can be changed per query. Recommended - tez: reuse of containers, pre-warm of containers
* ORC format: impression for individual data column types, skipping processing chunks of files, parallelism. Maintains index with stats and aggregation for every column: min, max, sum. ACID supported only in ORC. 
  * Predicate pushdown - where conditions are passed to reader to filter records while read before buffering. If value from where is not in min-max range, entire chunk is skipped.
  * Vectorised processing - run operations faster. Low level optimisations - code for commonly used functions + compiler and hardware features of modern CPUs =&gt; streanline processing of data in chunks of 1024 row at a time. Within large block of data of same type compiler can generate code to execute identical functions. In a tight loop we're going through the long code path that would be required for independent fuction calls. Can be enabled globally or per query.
  * cost based optimizer - tune up query plan according to metadata. Gathered in advance. Stats include min, max, number of distinct values, partitions, table ddl. CBO on/off - in configs
* partitioning and bucketing: bucket pruning used be tez engine - read only relevant bucket

