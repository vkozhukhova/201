# Spark part 1

## Spark

* open-source cluster computing framework
* speed 
* ease of use
* in-memory performance
* general execution engine
* improved performance than MR both in memory and on disk
* highly accecable: api for java, python, R, scala, SQL
* a lot of integrated libs: ML, SQL, streaming
* competed with MR not with hadoop itself
* doesn't have its own DFS
* can use HDFS

### compare to MR

#### MR - batch processing, it's a sequence of actions:

> read from cluster,
>
> update data,
>
> write back to cluster
>
> repeat.
>
> Компьютеризированная пакетная обработка - это выполнение "заданий, которые могут выполняться без взаимодействия с конечным пользователем или могут быть запланированы для выполнения по мере освобождения ресурсов."

* more lines of code
* same task will take less space
* complex
* difficult to write and debug
* no interactive mode
* developed in java; supp C, Ruby, python
* disk-based
* slower
* commodity hardware
* data processing
* batch processing
* no built in sceduler
* replication as FT
* resilient
* secured: Kerberos, ACL
* scalable highly
* uses mahaout for ML

#### Spark - in memory:

> read from cluster
>
> process everything in memory
>
> write back to cluster.

* Disk for data if it not fits into memory
* has interactive mode
* developed in scala, supp all above
* uses memory
* faster 100 times in memory
* mid to high level HW
* data analytics
* batch processing and real time processing \(streaming\)
* RDD as fault tolerance
* DAG as recovery \(rerun from checkpoint\)
* shared secred password auth
* scalable highly
* built in ML tools

## Components

Spark core

* basic functionality: rdd, api
* distributed task dispatching: scheduling, basic I/O, rdd abstraction, API
* interacts with scheduler
* interacts with with cluster manager: send for execution

cluster managers supported:

* yarn
* mesos
* kubernetes

#### Spark SQL

* quering data with SQL, HQL
* hive, parquet, json sources
* mix sql with rdd manipulations: python, java, scala
* qlick view and tableau BI tools supported

#### Spark streaming

* based on macro batching
* real time streaming data
* Dstreams = series of rdds
* similar api to rdd api: easy for developers

#### MLlib

ML support

#### Graph frames

* graph processing
* DF based graph qeries, serialization

#### Spark app

= driver process + set of executor processes

**Driver**

* own driver node
* maintain info about app
* respond to user program
* analyze work and distribute it across executors

**Executor**

* execute code
* send info and results back to driver

**Cluster manager**

* allocate resources to spark app
* standalone manager, yarn, mesos, kubernetes

#### Driver process

* JVM process
* hosts context for spark app
* master node in spark app
* uses schedulers
* hosts web ui for end
* splits app into tasks to run on executors
* spread tasts across executors
* coordinates workers
* use add services: shuffle manager, memory manager, broadcast manager

#### Spark context

* heart of spark app
* set up internal services
* establish connection to execution env
* create rdd, accumulators, broadcast variables, run spark jobs
* client of execution env
* only one per JVM

#### Spark session

* includes context, streaming context, sql context, hive c-xt,
* one entrypoint for execution env for all contexts
* unifies process of reading data in different formats

#### executor

* exec code
* report state to driver
* managed by executor backends - part of cluster manager
* = host + id
* provide in-memory storage for rdd that are cached in spark app
* executors amount = data nodes amount

## Spark architecture

* master-worker arch
* driver -&gt; single coordinator \(master\) - manages workers
* driver and executors has own java processes
* can run this processes os same \(horizontal cluster\) or separate machines \(vertical cluster\) or mixed

### RDD

* resilient distributed dataset - устойчивый распределенный набор данных
* fundamantal data anstraction
* represent distributed data in spark
* READ-ONLY PARTITIONED collection of records!!!!

### Spark job

* top-level execution for spark app that corresponds to an action
* Action is called by driver
* Job = set of tasks, arranged in stages
* Compute a job = compute partitiones of rdd
* Call an action and there goes job submission
* Stage = phisical unit of execution, a set of parallel tasks, one task per partition
* task = smallest unit of execution to compute partition

### DAG

* finite directed graph with no directed cycles: нет циклов, но есть параллельные пути
* graph with topological ordering: последовательность вершин, таких, что каждое ребро напрвлено из более ранней вершины, к более поздней в последовательности
* transitive closure: добавление параллеьных путей, чтобы можно было точно также пдостигать каждой вершины но другими путями

### MR

* all MR operations are independent
* hadoop not know what operation is next
* till the completion of a job all jobs are blocked from beginning
* long time and small data volume

### Spark computing

* execution plan optimized in DAG
* create context
* create rdd and transform
* create dag
* dag scheduler: splits dag into stages and submits stage when as ready to task sceduler
* task sceduler: launch with cluster manager, retry failed tasks
* backend received worker list from cluster manager
* launch task at executor
* block manager of executor deals with shuffled data and cached rdds
* new task runner starts at executor and process a set of tasks

## RDD and transformations

### RDD

resilient distributed dataset computed in many jvms

* resilient = fault tolerant with lineage graph and can recompute fault partitions
* distributed = use multiple nodes
* partition = chunk of dataset \(logical\)
* one concurrent task for each partition
* partitions amount = cores amount or 2-3 times more
* partition &lt; 128 mb
* shuffle block &lt; 2 gb

#### RDD features

* in-memory computation: data stored there
* lazy evaluation: no computation until action is executed
* immutable or read-only: only create new rdd
* fault-tolerant: track data lineage info from dag and rebuilt on failure. rdd remembers how it was created
* persistence: hold data in persistence storage like ram or disk
* partitioned
* location stickness: define placement for partitions as close to record as possible; task is as close to data as possible
* coarse-grained operation: The coarse-grained operation means to apply operations on all the objects at once. Fine-grained operations mean to apply operations on a smaller set.  \(крупнозернистая операция, мелкозернистая операция\)

### Transformations

* produces new rdd from existing rdd
* narrow: input and output stays in same partition, no data movement is needes: map, filter, union, zip \(created pair rdd\)
* wide: input from other partition is required, data shuffling is needed before processing: intersection, distinct. groupby, reduce, join, coalesce \(decrease amount of partitions\), repartition \(decrease and increase\)

### Actions

* no rdd values on return: count, collect, reduce, saveas, take, aggregate, foreach

### PairRDD

* key-value pairs
* mapValues, keys, values, groupbykey, reducebykey, aggregatebykey, sortbykey
* two pairrdd transformations: cogroup, substractbykey, join: inner, legtouter, rightouter
* actions: countbykey

### PairRdd Shared Variables

* variable are copied to each machine and not get back
* exclusions:

  > broadcast: efficient sharing through speacial protocol \(e.g. large dataset\)
  >
  > accumulators: only added though operation: counters, sums

## Spark ETL

* extract from source 
* transform \(apply function\) 
* load to destination \(+ cleaning as 2 step, ECL\)

Extract:

* define number of generations to be kept,
* the format of data to extract 
* the size to extract

Cleansing

* removed no sense data
* validation \(correct data\)
* filtering \(noise data remove\)
* correct corruoted
* remove duplication

Transform

* data type conversion
* data joining
* aggregation
* encryption

### File formats

* text
* csv
* xml
* json
* seq
* object
* parquet
* interacts with hive
* interacts with jdbc
* work with hdfs and Amazon s3 \(simple storage servcice\): scalability, HA, low latency, low procing. high speed and low performance over public internet
* cassandra, mongo, hbase, solr: scalability, HA, accesed through sql and api, organise data as key-value.

### DataFrames

* rdd - set of java and scala objects representing data
* api - map, filter, reduce =&gt; new rdd
* OOP principles, as work with objects
* java serializer \(or kryo\) - sending data between nodes, serializing includes class info and data, GC costs, constructuion of objects

#### DF API

* schema to describe data, only pass data between nodes
* performance increase: serialize data into off-heap storage in a binary format and transform, no GC costs, no construction of objects
* relational query plan, spark catalyst optimizer executres it
* distribited collection of data. organised into named columns: table, df

Minuses:

* col names in queries are not verified, can cause runtime exceptions
* very scala centric, support for java limited: when converting from rdd it assumes that it implemetns scala interfaces

#### DATAset API

* object oriented as rdd
* compile time type safety as rdd
* performance benefits of query optimizer: effecient off-heap storage
* encoders between jvm representation and spark default binary format
* generate bytecode to interact with off-heap data, no nedd to deserialize entire object
* no custom encoders 
* work with java and scala object

> DATASET - строго типизированная коллекция характериных для предметной области \(бизнес-логики\) объектов которая поддается параллельным преобразованиям с помощью функциональных или реляционных операций
>
> DATAFRAME - нетипизированный вид датасета, это датасет из строк
>
> переход от rdd к df - это переход от неструктурированных вычислений к структурированным

rdd

* jvm objects
* java serialization
* heap storage
* can be converted to df and DS

DF and DS

* row objects
* of heap storage
* DS can be converted to DF and rdd, while DF - not

> R and Python has no compile time type safety =&gt; only untyped api = DF

DF

* only type through column
* same amount of rows for each column
* schema can be defined manually or construct by spark \(infer\)
* csv don't have datatypes, json has
* no need to have utility class to read from file and have an object, we can have schema

## Spark deployment

schedulers supported:

* local
* standalone
* yarn
* mesos
* kubernetes

## spark-submit

### locally:

* one thread, several threads or give all possible cores
* non distributed mode
* all components in same jvm: driver and workers, can be in different cores
* driver is used for execution
* testing, debugging, reaserch
* tasks not rexecuted on failure

### Cluster manager achitectures

Figure 1 visualises the different approaches: a gray square corresponds to a machine, a coloured circle to a task, and a rounded rectangle with an "S" inside corresponds to a scheduler.0 Arrows indicate placement decisions made by schedulers, and the three colours correspond to different workloads \(e.g., web serving, batch analytics, and machine learning\). ![monolithic](http://firmament.io/img/blog/scheduler-arch-monolithic.png) ![two-level](http://firmament.io/img/blog/scheduler-arch-twolevel.png) ![shared-state](http://firmament.io/img/blog/scheduler-arch-sharedstate.png)

\(a\) Monolithic scheduler. \(b\) Two-level scheduling. \(c\) Shared-state scheduling. Figure 1: Different cluster scheduler architectures. Gray boxes represent cluster machines, circles correspond to tasks and Si denotes scheduler i.

Many cluster schedulers – such as most high-performance computing \(HPC\) schedulers, the Borg scheduler, various early Hadoop schedulers and the Kubernetes scheduler – are **monolithic**. A single scheduler process runs on one machine \(e.g., the JobTracker in Hadoop v1, and kube-scheduler in Kubernetes\) and assigns tasks to machines. All workloads are handled by the same scheduler, and all tasks run through the same scheduling logic \(Figure 1a\). This is simple and uniform, and has led to increasingly sophisticated schedulers being developed. As an example, see the Paragon and Quasar schedulers, which use a machine learning approach to avoid negative interference between workloads competing for resources.

Most clusters run different types of applications today \(as opposed to, say, just Hadoop MapReduce jobs in the early days\). However, maintaining a single scheduler implementation that handles mixed \(heterogeneous\) workloads can be tricky, for several reasons:

1. It is quite reasonable to expect a scheduler to treat long-running service jobs and batch analytics jobs differently.
2. Since different applications have different needs, supporting them all keeps adding features to the scheduler, increasing the complexity of its logic and implementation.
3. The order in which the scheduler processes tasks becomes an issue: queueing effects \(e.g., head-of-line blocking\) and backlog can become an issue unless the scheduler is carefully designed.

Overall, this sounds like the makings of an engineering nightmare – and the never-ending lists of feature requests that scheduler maintainers receive attests to this.

**Two-level scheduling architectures** address this problem by separating the concerns of resource allocation and task placement. This allows the task placement logic to be tailored towards specific applications, but also maintains the ability to share the cluster between them. The Mesos cluster manager pioneered this approach, and YARN supports a limited version of it. In Mesos, resources are offered to application-level schedulers \(which may pick and choose from them\), while YARN allows the application-level schedulers to request resources \(and receive allocations in return\).2 Figure 1b shows the general idea: workload-specific schedulers \(S0–S2\) interact with a resource manager that carves out dynamic partitions of the cluster resources for each workload. This is a very flexible approach that allows for custom, workload-specific scheduling policies.

Yet, the separation of concerns in two-level architectures comes with a drawback: the application-level schedulers lose omniscience, i.e., they cannot see all the possible placement options any more. Instead, they merely see those options that correspond to resources offered \(Mesos\) or allocated \(YARN\) by the resource manager component. This has several disadvantages:

1. Priority preemption \(higher priority tasks kick out lower priority ones\) becomes difficult to implement: in an offer-based model, the resources occupied by running tasks aren't visible to the upper-level schedulers; in a request-based model, the lower-level resource manager must understand the preemption policy \(which may be application-dependent\).
2. Schedulers are unable to consider interference from running workloads that may degrade resource quality \(e.g., "noisy neighbours" that saturate I/O bandwidth\), since they cannot see them.
3. Application-specific schedulers care about many different aspects of the underlying resources, but their only means of choosing resources is the offer/request interface with the resource manager. This interface can easily become quite complex.

**Shared-state architectures** address this by moving to a semi-distributed model, in which multiple replicas of cluster state are independently updated by application-level schedulers, as shown in Figure 1c. After the change is applied locally, the scheduler issues an optimistically concurrent transaction to update the shared cluster state. This transaction may fail, of course: another scheduler may have made a conflicting change in the meantime.

The most prominent examples of shared-state designs are Omega at Google, and Apollo at Microsoft, as well as the Nomad container scheduler by Hashicorp. All of these materialise the shared cluster state in a single location: the "cell state" in Omega, the "resource monitor" in Apollo, and the "plan queue" in Nomad.5 Apollo differs from the other two as its shared-state is read-only, and the scheduling transactions are submitted directly to the cluster machines. The machines themselves check for conflicts and accept or reject the changes. This allows Apollo to make progress even if the shared-state is temporarily unavailable.6

A "logical" shared-state design can also be achieved without materialising the full cluster state anywhere. In this approach \(somewhat similar to what Apollo does\), each machine maintains its own state and sends updates to different interested agents such as schedulers, machine health monitors, and resource monitoring systems. Each machine's local view of its state now forms a "shard" of the global shared-state.

However, shared-state architectures have some drawbacks, too: 1. they must work with stale information \(unlike a centralized scheduler\), and 2. may experience degraded scheduler performance under high contention \(although this can apply to other architectures as well\).

Spark - no support for shared-state schedulers

### Cluster managers

#### Standalone

* part of spark distribution
* HA for master
* resilient for worker failures
* can manage resources per application
* run along existing hadoop deployment and hdfs
* include deploy scripts
* **monolithic** architecture
* simple FIFO scheduler
* by default each app uses all available nodes of cluster
* number of nodes can be limited by app, by user or globally
* memory, cpu can be controlled by app's spark conf object

**2 deploy modes for standalone:**

* client: driver in same process as client that submits app
* cluster: driver is launched from one of the worker processes and client process exits as soon as app is submitted and don't wait for app finish.

#### yarn

* HA for master and slaves
* support docker containers in non-secure mode
* **two-level architechture** with limited impelementation
* shares features of monolithic and two-level architecture
* app level logic can't choose resources unless it requests much more than it need from resource manager
* can only place app level tasks for pre existing containers that represent cluster level task
* good for MR system: Ms and Rs must be assigned to a dynamic collection of workers
* optimized for scheduling batch jobs with long runtimes
* not for fast jobs or real time
* good for stateless batch jobs that can be restarted on failure

**2 modes on yarn:**

* cluster: spark driver runs inside app master process that is managed by yarn on cluster. client can go away after initiating app
* client: driver runs in client process and app master is only used to request resources from yarn

> if we are in cluster network - mode is client
>
> if we are out of cluster network - mode is cluster to minimize latency btween drivers and executors and it's less vulnerable to network crashes

**Spark app deploy with cluster mode on yarn**

* yarn RM launches app master in yarn container
* master requests resources, launches containers with them

  and spark executors inside

* executors are registered with the driver and are ready to exec

**Spark app deploy with client mode on yarn**

* driver runs on a client node and it's only difference

### Mesos

* global RM for entire data center
* knows what resources are available and makes offers to app scheduler
* app scheduler = framework
* framework accepts or declines offers
* two-level scheduler
* scheduling algorithms are pluggable
* can develop own scheduling algorithms with accepting/declining strategies
* each framework can decide what algorithm to use
* is an arbiter resolving conflicts between multiple schedulers
* framework accepts offered resources and exec task or declines offer and waits for another offer
* based on years of OS research and is scalable

#### Mesos modes:

* client
* cluster
* coarse-grained \(default\): each spark executor runs a single mesos task: low latency \(задержка\). regards large subcomponents
* fine-grained \(deprecated\): each spark task inside executor runs a separate mesos task: higher utilization. regards smaller subcomponents

### Kubernetes

открытое программное обеспечение для автоматизации развёртывания, масштабирования контейнеризированных приложений и управления ими monolithic scheduler for pods - базовая единица для управления и запуска приложений, один или несколько контейнеров, которым гарантирован запуск на одном узле, обеспечивается разделение ресурсов, межпроцессное взаимодействие и предоставляется уникальный в пределах кластера IP-адрес. Последнее позволяет приложениям, развёрнутым на поде, использовать фиксированные и предопределённые номера портов без риска конфликта. Поды могут напрямую управляться с использованием API Kubernetes или управление ими может быть передано контроллеру: collection of collocated containers that share same namespaces

* lightweight
* portable
* massively scalable
* decoupled design: can be split into many components: control plane and worker node services
* control plane: assign containers to nodes and manages cluster configuration
* worker node services: run on individual machines and manage local containers
* pod: share same namespace - used for service discovery and segregation

### Build management systems

* sbt
* gradle
* maven

#### Spark web UI

* available while driver is running
* stage event timeline
* dag visualization

