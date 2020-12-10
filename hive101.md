# Hive part 1

Apache Hive - система управления базами данных на основе платформы Hadoop. Позволяет выполнять запросы, агрегировать и анализировать данные, хранящиеся в Hadoop.

Hadoop -

* get data to  a fault-tolerant storage, 
* write code to process, 
* save results back. 
* Need to write low-level jobs.

Hive - know SQL, can analyze big data

* open-source
* abstract hadoop complexity
* provides hql
* do ad-hoc querying, summarization and stata analysis

> Ad hoc queries — узкоспециализированные запросы, то есть запросы, созданные для решения единоразовой проблемы. В общем смысле термин ad hoc используется для обозначения решений под конкретные проблемы и задачи, без обобщенности.

* convert hql to DAG of MR, Tez or Spark jobs that are submitted to hadoop for execution

Support features:

* indices
* buckets
* partitions
* ACID transactions
* customer UDFs
* joins
* sampling

ACID - система требований к транзакциям.

1. Atomicity — Атомарность

Атомарность гарантирует, что никакая транзакция не будет зафиксирована в системе частично. Будут либо выполнены все её подоперации, либо не выполнено ни одной.

1. Consistency — Согласованность

Транзакция, достигающая своего нормального завершения \(EOT — end of transaction, завершение транзакции\) и, тем самым, фиксирующая свои результаты, сохраняет согласованность базы данных. Другими словами, каждая успешная транзакция по определению фиксирует только допустимые результаты. Это условие является необходимым для поддержки четвёртого свойства.

Например, в банковской системе может существовать требование равенства суммы, списываемой с одного счёта, сумме, зачисляемой на другой. Если какая-либо транзакция произведёт списание, но не произведёт зачисления, то система останется в некорректном состоянии и свойство согласованности будет нарушено.

1. Isolation — Изолированность

Во время выполнения транзакции параллельные транзакции не должны оказывать влияния на её результат.

1. Durability — Стойкость

Если пользователь получил подтверждение от системы, что транзакция выполнена, он может быть уверен, что сделанные им изменения не будут отменены из-за какого-либо сбоя.

They take time to implement manually

Hive built on top of hadoop =&gt; limitations of hadoop are inherited

can't be used for online transcation processing

suitable for traditional data warehousing tasks

## Architecture of hive \(high-level\)

Request from client -&gt; hive services -&gt; execution layer -&gt; processed by configured engine -&gt; write to hdfs

## Client layer

> HiveServer2 based on Thrift. Thrift — язык описания интерфейсов, который используется для определения и создания служб под разные языки программирования. Является фреймворком к удалённому вызову процедур \(RPC\). Thrift является двоичным протоколом связи.

* any language supported Thrift: Ruby, Python
* JDBC and ODBC interfaces

> JDBC — платформенно-независимый стандарт взаимодействия Java-приложений с различными СУБД. JDBC реализован в виде пакета java. sql, входящего в состав Java SE \(Standart Edition\). JDBC позволяет устанавливать соединение с базой данных согласно специально описанному URL.
>
> Open Database Connectivity \(ODBC\) is a standard application programming interface \(API\) for accessing database management systems \(DBMS\) . The designers of ODBC aimed to make it independent of database systems and operating systems.\[citation needed\] An application written using ODBC can be ported to other platforms, both on the client and server side, with few changes to the data access code.

## Resource management

* pluggable execution engine - can run tasks via MR, tez, Spark
* abstracts user from this implementation
* yarn - RM
* pluggable distributed storage: hdfs, azure, google...

## UI component

* submit queries and commands to system
* CLI: 

> interactive: submit required queries manually and get the result
>
> non-interactive: with -f option we can specify the location of a file which contains HQL queries. For example- hive -f my-script.q

* rest api

## HiveServer2

* build on thrift
* allows different clients to make requests, and get responses back
* they are supported over tcp and http
* support multi-client concurrency and authentication
* support for open-api clients like jdbc and odbc
* jetty web-server for configuration, logging, metrics and active session info

## Hive Driver

* managing lifecycle of hql statements
* maintains session handle and statistics
* communicates with compiler, optimizer and executor
* acts like a controller which receives the HiveQL statements. The driver starts the execution of the statement by creating sessions. It monitors the life cycle and progress of the execution. Driver stores the necessary metadata generated during the execution of a HiveQL statement. It also acts as a collection point of data or query result obtained after the Reduce operation.

## Compiler

* query parsing
* type checking
* semantic analysis
* converts the query to an execution plan. The plan contains the tasks. It also contains steps needed to be performed by the MapReduce to get the output as translated by the query. The compiler in Hive converts the query to an Abstract Syntax Tree \(AST\). First, check for compatibility and compile-time errors, then converts the AST to a Directed Acyclic Graph \(DAG\).

## Optimizer

* get logical plan in form of DAG of jobs
* performs various transformations on the execution plan to provide optimized DAG. It aggregates the transformations together, such as converting a pipeline of joins to a single join, for better performance. The optimizer can also split the tasks, such as applying a transformation on data before a reduce operation, to provide better performance.

## Executor

* excution of jobs against hadoop
* takes care of pipelining the tasks.

## Hive metastore

Hive metastore consists of two fundamental units:

1. A service that provides metastore access to other Apache Hive services.
2. Disk storage for the Hive metadata which is separate from HDFS storage.

### Supports RDBMS:

* Derby
* MySQL
* MS SQL Server
* Oracle
* Postgres

### Features:

* central repo of hive metadata
* provides data abstraction and data discovery
* keeps both data and metadata in sync
* stores system catalog about tables, columns, partitions, schema, location in RDBMS
* by default runs in same JVM as Hive service and uses embedded DB - Derby - stored in local FS
* embedded mode = only one hive session at a time, only one embedded Derby database can access the database files on disk at any one time
* local mode = many Hive sessions, Hive metastore in same process as Hive service but metastore process is separate and on separate host
* remote mode = Hive metastore is separate JVM process, does not require administrator to share jdbc login info, clients no longer need share database credentials with each Hive user to access the metastore database

## Hive has the following limitations:

* Apache does not offer real-time queries and row level updates.
* Hive also provides acceptable latency for interactive data browsing.
* It is not good for online transaction processing.
* Latency for Apache Hive queries is generally very high.

## Data units

* DB
* tables
* partitions
* buckets

### DB

* provide manespace for tables
* prevent naming conflicts

### Tables

* internal - delete data and metadata
* external - provide location, data outside hive directories, only metadata deleted

### Partitions

* separate data by columns values
* directories, not written to table.
* usually year, id...
* reduces scan, performance improvement

#### Partitioning strategies

* need enable dynamic partitions
* non-strict mode - all partitions can be dynamic \(by default one should be static\)
* INSERT OVERWRITE TABLE .. PARTITION statement for each partition

**Static Partition**

Static Partition saves your time in loading data compared to dynamic partition. You “statically” add a partition in table and move the file into the partition of the table. We can alter the partition in static partition.

You can get the partition column value from the filename, day of date etc without reading the whole big file. Static partition is in Strict Mode. You should use where clause to use limit in static partition.

**Dynamic Partition**

Usually dynamic partition load the data from non partitioned table. Dynamic Partition takes more time in loading data compared to static partition.

Incase of dynamic partition whole big file i.e. every row of the data is read and data is partitioned through a MR job into the destination tables depending on certain field in file.

There is no required where clause to use limit. We can’t perform alter on Dynamic partition

If you want to use Dynamic partition in hive then mode is nonstrict. Can create large number of partitions.

**Mixed**

### Buckets

* based on hash function of columns
* bucket created as a file
* for e.g. fast sampling can be no partitions, just buckets \(getsmall subset of data for local processing\)
* perform efficient map-side joins
* partitions maybe too big - need to divide more
* CLUSTERED BY statement INTO .. buckets
* buckets can't be the same as partitions
* hash\(column\) % number of buckets

> Map side join is a process where joins between two tables are performed in the Map phase without the involvement of Reduce phase. Map-side Joins allows a table to get loaded into memory ensuring a very fast join operation, performed entirely within a mapper and that too without having to use both map and reduce phases. In case your queries frequently run with small table joins , you might see a very substantial decrease in the time taken to compute the queries after usage of map-side joins. If tables being joined are bucketed on the join columns, and the number of buckets in one table is a multiple of the number of buckets in the other table, the buckets can be joined with each other. To perform the SortMergeBucket Map join, we need to have two tables with the same number of buckets on the join column and the records are to be sorted on the join column.

## File Formats

Input and output Formats = tell how to read and write files

Supports

* txt
* seq
* rc
* orc
* avro
* parquet

### Text

* default
* data = lines: 1 record per line
* sep by \n char

### seq

* binary
* stores as key-value pairs
* manage many small files together
* have single metadata record in hdfs

### avro

schema evolution

### rc

* binary
* row and columnar approaches
* divide data horizontally into partitions, 4 mb row groups
* parttition = columnar format
* compressed at columnar level
* lazy decompression: if condidition not in column, whole row group is skipped and no other columns are decompressed

### orc

* support by hdp
* row group = stripe
* size of row group = 64+mb
* indices and satistics about columns
* does more encoding =&gt; less size, more compressed
* only format supporting ACID

### parquet

* cloudera
* columnar

### implement both formats yourself

## Hive SerDe

interface that handles both serialization and deserialization in Hive. Also, interprets the results of serialization as individual fields for processing. In addition, to read in data from a table a SerDe allows Hive. Further writes it back out to HDFS in any custom format. However, it is possible that anyone can write their own SerDe for their own data formats.

HDFS files –&gt; InputFileFormat –&gt;  –&gt; Deserializer –&gt; Row object

Row object –&gt; Serializer –&gt;  –&gt; OutputFileFormat –&gt; HDFS files

It is very important to note that the “key” part is ignored when reading, and is always a constant when writing. However, row object is stored into the “value”.

## Data types

* primitive: numeric, string, boolean
* column: date, timestamp
* literals: decimal
* intervals: time units: hours, seconds
* complex: array, map, struct, union

## joins

* inner
* left
* right
* full outer
* left semi
* cross join

Features:

* convert to MR jobs
* not commutative
* left assosiative
* largest table to the right for best performance

## inserts

* insert overwrite table OR **\[local\] directory**  =&gt; can save to hdfs or locally
* insert into = append

## tablesample

## Functions

* built-in
* aggregate
* analytic
* table-generated: lateral view, explode
* can be udfs

