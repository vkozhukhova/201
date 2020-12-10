# Spark part 2

## Shared Variables

* broadcast
* accumulators

### broadcast

* efficiently transfer data \(e.g. dicts\) to executors
* joins also can be used
* handled by torrent-like protocol
* nodes all try to distribute variable as quickly and efficiently as possible by uploading and dowloading what they can.
* faster than one node pushing data to all nodes
* `b = context.broadcast(variable)` created on driver and send to executors; `b.value` - to show
* on executors it'a avaliable in read-only mode

### accumulators

* write-only
* to count statistics
* only added though assosiative and commutative operations =&gt; can be supported in parallel
* used for counters and sums
* extend AccumulatorParam for custom type accumulator; numeric types - by default
* result can be seen in web ui if accumulator **is named**
* `a = context.accumulator(0, name)`; `context.parallelize(variable).foreach(x=>a+=x)`; `a.value`
* For accumulator updates performed inside actions only, Spark guarantees that each task’s update to the accumulator will only be applied once, i.e. restarted tasks will not update the value. In transformations, users should be aware of that each task’s update may be applied more than once if tasks or job stages are re-executed.

## Caching

* for optimization
* dag can be viewed in ui
* without caching on failure we'll nedd to repeat all calculations from beginning
* can cache some point and use these intermediate results on failure
* data stored in memory
* use case: a basic dataframe in which we do operations/aggregations
* `contect.cacheTable("")` or `df.cache()`
* different levels of storage - memory or disk - in different formats
* auto select best compression
* minimize memory usage and GC pressure

## Smart sources

### data sources with capabilities of:

* server side filtering \(predicate pushdown\): records with certain condition. optimizer passes info to data source engine, and read are only necessary. Cassandra, RDBMS, Hbase
* column pruning: in columnar format optimizer knows which columns are needed. optimizer passes info to data source engine, and read are only necessary.Parquet, ORC
* partition pruning: optimizer knows that records from certain partitions are needed

### jdbc

for reading from other DBs.

* Successor of jdbcRDD.    
* returns result as DataFrame
* enable smart source capabilities

```scala
val jdbcDF = sqlContext.read
.format("jdbc")
.options(
Map("url" -> "jdbc:postgresql:dbserver",
"dbtable" -> "schema.tablename"))
.load()
```

### parquet

* spark can read only certain columns \(orc the same\)
* stores data in groups by columns.
* each column having certain index. We can choose what offsets read from disk
* Compact binary encoding
* Compression
* Columns stored separately with index
  * Allows skipping of unread columns
* Partitioning support
* Data skipping using statistics
* Schema merging
* Smart source:
  * Column/Partition pruning
  * Predicate pushdown

## Distributed SQL engine

* can send queries to HiveServer2 via jdbc/odbc and run on spark
* idea - have a server as one entrypoint to cluster who can serve different jobs and tasks
* Spark SQL can act as distributed query engine

## SQL joins

### Sort-merge join

* fast default algorithm
* keys must be sortable
* set `spark.sql.join.preferSortMergeJoin = false` to stop using it by default

### Shuffle hash join

* when key isn't sortable
* join key is disabled
* one side of plan should be much smaller than other
* spark must be able to build a hashmap
* to trick catalyst - change max size for a broadcast table or change number of partitions for join

### Broadcast join \(aka map-side join\)

* no shuffle
* need to set `spark.sql.join.autoBroadcastJoinTreshold = 10485760`
* if catalyst tells you not to use broadcast join, but you're sure, you can change the previus value or hint catalyst with function `broadcast`: `org.apache.sql.functions: def broadcast[T](df: Dataset[T])`
* can build local map for table

  \`\`\`

  /\*\*

* Matches a plan whose single partition should be small enough to build a hash table.

  \*

* Note: this assume that the number of partition is fixed, requires additional work if it's
* dynamic.

  \*/

  private def canBuildLocalHashMap\(plan: LogicalPlan\): Boolean = {

  plan.stats.sizeInBytes &lt; conf.autoBroadcastJoinThreshold \* conf.numShufflePartitions

  }

  \`\`\`

## Tuning techniques

* driver - main component of architecture: performs tasks 7 coordinates executors
* driver requests resources from cluster manager \(mesos, yarn, spark\)
* each executor has cache and tasks
* shuffle service???

### Driver optimizations

increasing memory:

* `spark-submit --driver-memory 8g`
* `spark-submit --executor-memory 8g`

### Executor optimizations

memory usage consumed by:

* execution
* storage
* **Execution memory** refers to used computation in shuffles, joins, sorts and aggregations.
* **Storage memory** used for caching, broadcast variable and

  propagation internal data across the cluster.

* **User memory** can store your own data structures there that

  would be used in RDD transformations.

* **Reserved memory** for internal needs.

Execution & storage share unified region of memory. If no execution =&gt; storage can have all memory and vice versa.

1. off-heap
2. object is serialized to byte array
3. managed by the operating system but stored outside the process heap in native memory
4. not processed by the garbage collector.
5. slower than accessing the on-heap storage but still faster than

   reading/writing from a disk.

6. To turn on:

   ```text
   spark.memory.offHeap.enabled = true
   spark.memory.offHeap.size = 3g
   ```

7. leveraging speculation
8. Some tasks are completed and one task is super slow. =&gt; can enable speculative tasks execution.
9. Spark runs additional task if suspects a task is running on straggler node.
10. Good in case 95-99% of task is already finished.
11. By default this setting is off
12. to turn on:

    ```text
    spark.speculation = true
    spark.speculation.interval = 100
    // denotes rate to see if speculation is needed
    spark.speculation.multiplier = 1.5
    // define how many time task hast to be slower than medial to trigger speculation
    spark.speculation.quantile = 0.95
    // defined the percentage of completed task before triggering speculation
    ```

13. level of parallelism
14. to fully utilize cluster
15. spark auto sets number of map tasks to run on each file according to it's size. you can tune it with `context.textFile()`
16. for distributed operations like groupBy Key, reduceByKey - uses largest parent RDD's  number of partitions
17. `sqlContext.sql("set spark.sql.shuffle.partitions=10");`
18. you can pass level of parallelism as 2 argument \(number of partitions\) `val rdd = sc.textFile("path" , 6)`
19. set 2-3 partitions per core
20. optimal value depend on cluster, app and value itself

```text
spark.sql.shuffle.partition = 200
// Configures the number of partitions to use when shuffling data for joins or
// aggregations.
spark.default.parralelizm = 10
// default number of partitions in RDDs after transformation like join
// reduceByKey, parallelize
```

### Shuffle service optimizations

dynamic allocation of the resources

```text
spark.dynamicAllocation.enable = true
spark.dynamicAllocation.executorIdleTimeout = 2m
spark.dynamicAllocation.minExecutors = 0
spark.dynamicAllocation.maxExecutors = 2000
spark.shuffle.service.enabled = true
```

* `spark.shuffle.service.enabled = true` reduces the load for executors as shuffling will be dome by shuffle service
* shuffle service can serve multiple executors
* tasks appear in the queue in scheduler, and executors start launching. More tasks -&gt; more executors until full load of cluster. As tasks are completed, sperk start to reduce number of executors up to `spark.dynamicAllocation.minExecutors`. 
  * better resource utilization
  * good for multi-tenant env

## Debug and test

### debug

* run in local mode
* logging
* use sample dataset

### unit testing

* spark-testing-base framework can be useful for comparing RDDs: handles ordered / unordered comparisons, but won't show differences
* Spark is friendly to unit testing with any popular unit test framework
  * Create SparkContext in your test with the master = local
  * Run your operations
  * Call SparkContext.stop\(\) to tear it down
  * Make sure you stop context within finally block or test framework’s tearDown method

### Debugging in yarn client mode

* YARN supports client mode job submission
* Spark Application can be launched by IDE
* Driver will run on the developer’s machine
* Executors will be scheduled and launched on cluster
* Important: developer machine and application master’s node in the cluster have to be able to reach each other

### Monitor spark jobs

* cluster manager UI 
  * spark master: workers, running apps, completed apps
  * yarn RM: list of apps and their details
  * mesos web ui: active tasks, completed tasks
* spark ui 
  * same for all cluster managers
  * exposed by driver
  * info for spark app
  * tabs: jobs, stages, storage, env, executors, SQL

## Catalyst optimizer

* dataframe includes catalyst & tungsten    
* engine which builds a tree of operations based on received queries. It simplifies process of usage datasets, dataframes, spark sql.  
* extensible optimizer. Based on functional programming constants in scala. 
* contains a general library for representing trees & applying rules to manipulate them. 
* trees can be manipulated using rules which are functions from tree to another tree
* common aproach: use set of pattern-matching fuctions that find and replace subtrees with specific structure
* we write query, catalyst checks for correction, build a syntax tree and based on rules it already has he'll optimize plan.
* rule and cost based optimization

### Execution plan

* catalyst operates with plans. transformation = tree manipulation
* dataframe.explain\(\)
* optimization 

execution plan consists of:

* unresolved logical plan: tree of operators build, check for correct operators usage, check that columns are there
* logical plan \(LP\): 
  * sequence of opeartions is known, but realization of joins is undecided. 
  * tree representing data & schema
  * dataframe is container for logical plan.

3 types of logical plans:

* parsed LP
* analysed LP
* optimized LP

One more plan - physical \(spark\) plan - generated after parsing DSL, generated by specific parsers like HiveQL parser, dataframe DSL parser etc.

* transformations are represented as 3 nodes
* straight-forward translation with no tweaks

Analysed plan:

* different entities are resolved: relations, references, data type casting

optimized LP:

* eliminates subqueries for further optimization
* constant folding
* simplifies expressions
* simplifies filters \(remove always true filters\)
* push filter near data source
* remove unnecessary projections

Chain of execution plan:

sql query or dataframe -&gt; unresolved logical plan -&gt; LP -&gt; Optimized LP -&gt; physical plans -&gt; cost model -&gt; selected physical plan -&gt; RDDs

#### Analysis \(Rule executor\) stage

* from unresolved plan to LP
* uses Catalog to find where datasets and columns are coming from and types of columns
* rules applied to syntax tree
* predicate or projection pushdown - reduces volume of data for processing
* **Predicate** refers to the where/filter clause which effects the amount of rows returned. **Projection** refers to the selected columns.

#### Logical optimization \(Rule executor\) stage

* transforms LP to Optimized LP

#### Physical planning \(Strategies + Rule executor\) stage

* transforms Optimized LP to Physical plan
* Rule executor is used to adjust the physical plan to make it ready for execution
* Strategies imply that there are rules that e.g. depending on data size either sortmerge or broadcast join will be performed.

For query you need:

* attribute - column
* expression - new value computed on input

## CBO

* additional tool which allows to increase query performance
* since 2.3 - uses gathered statistics on source instead of applying pre-written rules; intermediate results also contain stats, which constantly is updated
* allows to understand the cost of all operations for number of lines, size of output
* picks the most optimal execution plan

### table stats

`ANALYZE TABLE table COMPUTE STATISTICS`

* number of rows 
* table size in bytes

### column stats

`ANALYZE TABLE table COMPUTE STATISTICS FOR COLUMNS col1, col2`

* distinct count
* null count
* average length
* max length
* min, max - not for strings/binary

CBO use stats for:

* estimate cardinality of operations - filters, joins
* select opt plan
* select hash join implementation \(broadcast/shuffle\)
* reorder joins

## Tungsten

* responsible for code generation

3 main initiatives:

* memory management & binary processing: leverage app semantica to manage memory explicitly & eliminate overhead of JVM object model & garbage collection. It has it's own memory model & works with binary data
* cache-aware computation: algorithms & data structures to exploit memory hierarchy
  * effective usage of L1, L2, L3 CPU caches
  * less time to fetch data from memory
* code generation: exploit modern compilers & CPUs - allow efficient operation directly on binary data

