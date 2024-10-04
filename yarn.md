# YARN and MapReduce

## Yarn

* manages resources - disk, memory, cpu - on cluster
* includes job scheduler and data operating system
* use resources in shared, secure and multi-tenant manner
* manages distribution of resources depending on configuration, several scheduling modes
* to efficiently run tasks - uses cluster topology: data locality - run tasks where data is located

### Features

*   multi-tenancy

    Динамическое управление ресурсами: ресурсы кластера используются несколькими механизмами доступа. набор ограничений для предотвращения монополизации ресурсов очереди или кластера одним приложением, пользователем или очередью, чтобы гарантировать, что кластер не перегружен.
* scalability - Yarn ResourceManager поддерживает расширение кластера до тысяч нод
* compatibility - Приложения MapReduce, разработанные для Hadoop 1, работают на YARN без каких-либо сбоев в существующих процессах. Yarn обеспечивает совместимость API с предыдущей стабильной версии платформы Hadoop.

Yarn lauches tasks in containers and makes applications portable

## Main idea

**separate 2 roles with deffirent responsibilities**:

* resource management
* application management

## Components

* client - job submission
* resource manager
* app master
* node manager
* container - runs applications

### RM

knows what resources cluster has and what free storage there is => yarn knows on which node run tasks.

includes:

* **scheduler** - has several configs, allocates resources, tracks their status and monitor progress

**RM** - is the ultimate authority that arbitrates resources among all the applications in the system.

#### RM components

* scheduler
* app manager
* resource tracker

**App manager**

* validates, admits, rejects apps
* allows to run app
* forwards app to scheduler

**Scheduler**

* allocates resources for apps
* partitions cluster based on policy

**Resource tracker**

* checks status of node and resources inside node
* registers new nodes
* obtains heartbeats from nodes
* forward node status to scheduler

### App master

* located on every node
* entrypoint
* manages app lifecycle and the container which is run

Yarn - framework that combine engine and API written in Java. It's an abstraction - unified data processing system used for MR, Spark, Tez... Can cooperate with HDFS to get data locality info to optimize task processing.

From logical POV:

**main idea** - separate 2 roles with deffirent responsibilities: global resource manager and per-application app master

**AM** - a framework specific entity and is tasked with negotiating resources with RM and working with Node Managers to execute and monitor the components tasks. AM negotiates appropriate resources containers from **scheduler** - part of RM which allocates resources.

From system POV:

**AM itself runs a normal container**.

From physical POV:

RM and Node Manager form new generic system for managing apps in distributed manner

### Node manager

* located on every node
* manages resources on current node
* slave
* launch app containers
* monitor app resources usage
* report to RM

### Container

we pack app in it, launch it and provide resources on some host to launch it. it makes app portable

* has launch API
* API is platform agnostic (not just Java)
* CLI to lauch process within container
* security tokens
* local resources: jars, shared objects, data files

## App lifecycle

1. App approaches RM (client submits job to RM), request of certain amount of disk and memory
2. RM checks with Node Managers how many resources they have, receives info from them
3. RM sends execution to the node. RM launches AM in a container. App master registers request from client and starts process on node. He need to start container to run app (containers execute tasks).
4. Node manager launches containers. Status of container execution is returned to AM.
5. AM sends to RM that job is done and resources can be freed up. AM deregisters

## Yarn logs and statistics

* hadoop.log.dir property or [http://namenode:port/logs](http://namenode/:port/logs)
* `hadoop daemonlog` - set log level for daemon or [http://namenode:port/logLevel](http://namenode/:port/logLevel)
* stacks: [http://namenode:port/stacks](http://namenode/:port/stacks)
* ambari, cloudera manager
* `yarn logs` - by app id, container id
* `yarn daemonlog` - set log level

## Yarn debugging

* **only in local mode**

```
dfs.replication="1"
fs.defaultFS="file:///"
fs.default.name="file:///"
mapreduce.framework.name="local"
```

* then run debugging through IDE
* test with unit tests as much as possible
* MapReduce, Cascading/Scalding, Tez and others support local mode and local debugging, all of them provides possibility to run unit tests

## Yarn access

* CLI
* GUI
* programming interface

## Yarn HA

active-standby architecture one RM is active, others - standby, wait for active RM failover

* automatically - with configured zookeeper
* manually

Active RM writes it's state to zookeper

### Many RMs

the configuration (yarn-site.xml) used by clients and nodes is expected to list all the RMs.

```
<property>
  <name>yarn.resourcemanager.ha.enabled</name>
  <value>true</value>
</property>
<property>
  <name>yarn.resourcemanager.cluster-id</name>
  <value>cluster1</value>
</property>
<property>
  <name>yarn.resourcemanager.ha.rm-ids</name>
  <value>rm1,rm2</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname.rm1</name>
  <value>master1</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname.rm2</name>
  <value>master2</value>
</property>
```

The default implementation is org.apache.hadoop.yarn.client.ConfiguredRMFailoverProxyProvider

1. Clients, ApplicationMasters (AMs) and NodeManagers (NMs) try connecting to the RMs in a round-robin fashion until they hit the Active RM.&#x20;
2. If the Active goes down, they resume the round-robin polling until they hit the "new" Active.&#x20;
3. After RM is found, we ask: where can we run, and RM returns all the failed jobs

## Schedulers

* several types

### FIFO scheduler

* sends container to a queue and run in order of sending
* not required on prod: can't start job until previous is done if there are no resources, so we can't stop running job even if critical one is in the queue - **no priority**

### Capacity scheduler

HDP default scheduler

* allows sharing of a cluster along organizational lines, whereby each organization is allocated a certain capacity of the overall cluster.
* when there are free resources and job arrives, it can be processed in a separate queue
* dedicated queue: each organization is set up with a dedicated queue
* queue hierarchy: queues may be divided in hierarchical fashion for groups
* queue elasticity: allocates spare resources to jobs in another queue

### Fair scheduler

* dynamically balances resources between all running jobs.
* first job allocates all resources
* second job starts while first is still running, resources that are freed from 1st job are assigned to the second job
* after a while each job is using half of the resources
* first gets back all resources when second job is finishing&#x20;
* so small jobs can have some resources, finish quickly and return resources back

### Testing in production

* fair or capacity scheduler is applied
* It makes sense to have separate queues/pools for dev team and production, and run dev tasks only in this dev scope
* HDP is configured to use Capacity Scheduler by default
* Cloudera CDH is configured to use FIFO Scheduler by default, but recommends Fair

Fair scheduling - это метод распределения ресурсов между заданиями таким образом, чтобы все задания получали в среднем равную долю ресурсов с течением времени. Когда выполняется одно задание, оно использует весь кластер. При отправке других заданий освободившиеся ресурсы назначаются новым заданиям, так что каждое задание получает примерно одинаковое количество процессорного времени. В отличие от стандартного планировщика Hadoop, который формирует очередь заданий, это позволяет коротким заданиям завершаться в разумные сроки, не объедая при этом длинные задания. Это также разумный способ разделить кластер между несколькими пользователями. Наконец, справедливое распределение может также работать с приоритетами заданий - приоритеты используются в качестве весов для определения доли общего вычислительного времени, которое должна получить каждая работа.

CapacityScheduler предназначен для совместного использования большого кластера, предоставляя каждой организации минимальную гарантию кол-ва ресурсов. Основная идея заключается в том, что доступные ресурсы в кластере Hadoop Map-Reduce распределяются между несколькими организациями, которые совместно финансируют кластер на основе вычислительных потребностей. Существует дополнительное преимущество, заключающееся в том, что организация может получить доступ к любому избыточному потенциалу, не используемому другими. Это обеспечивает эластичность для организаций экономически эффективным способом. ([https://docs.arenadata.io/adh/administration/yarn/CapacityScheduler.html](https://docs.arenadata.io/adh/administration/yarn/CapacityScheduler.html))

## Security

* Authentication is the process of ascertaining that somebody really is who he claims to be
* Authorization refers to rules that determine who is allowed to do what. E.g. Alex may be authorized to create and delete directories, while Nick is only authorized to read
* In Hadoop security is not trivial field: it requires a lot of efforts and we should be aware of it and plan these activities in advance
* The Kerberos principals for the ResourceManager and NodeManager must be configured in the yarn-site.xml file.&#x20;
* Make sure that each user who will be running YARN jobs exists on all cluster nodes (that is, on every node that hosts any YARN daemon) or each cluster node connected to the same LDAP directory.

On every node add these lines to yarn-site.xml

```
<property>
  <name>yarn.nodemanager.keytab</name>
  <value>/etc/hadoop/conf/yarn.keytab</value>   <!-- path to the YARN keytab -->
</property>
<property>
  <name>yarn.nodemanager.principal</name>   
  <value>yarn/_HOST@YOUR-REALM.COM</value>
</property> 
<property>
  <name>yarn.nodemanager.container-executor.class</name>    
  <value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>
</property> 
<property>
  <name>yarn.nodemanager.linux-container-executor.group</name>
  <value>yarn</value>
</property>
<property>
  <name>yarn.resourcemanager.keytab</name>
  <value>/etc/hadoop/conf/yarn.keytab</value>   <!-- path to the YARN keytab -->
</property>
<property>
  <name>yarn.resourcemanager.principal</name>   
  <value>yarn/_HOST@YOUR-REALM.COM</value>
</property>
<property> <!-- To enable SSL -->
  <name>yarn.http.policy</name>
  <value>HTTPS_ONLY</value>
</property>
```

To mapred-site.xml on every node:

```
<property>
  <name>mapreduce.jobhistory.address</name>
  <value>host:port</value> <!-- Host and port of the MapReduce Job History Server -->
</property>
<property>
  <name>mapreduce.jobhistory.keytab</name>
  <value>/etc/hadoop/conf/mapred.keytab</value> <!-- path to the MAPRED keytab for the Job History Server -->
</property> 
<property>
  <name>mapreduce.jobhistory.principal</name>   
  <value>mapred/_HOST@YOUR-REALM.COM</value>
</property>
<property>
  <name>mapreduce.jobhistory.http.policy</name> <!-- To enable SSL -->
  <value>HTTPS_ONLY</value>
</property>
```

Create a file called container-executor.cfg for the Linux Container Executor program that contains the following information:

```
yarn.nodemanager.local-dirs=<comma-separated list of paths to local NodeManager directories. Should be same values specified in yarn-site.xml. Required to validate paths passed to container-executor in order.>
yarn.nodemanager.log-dirs=<comma-separated list of paths to local NodeManager log directories. Should be same values specified in yarn-site.xml. Required to set proper permissions on the log files so that they can be written to by the user's containers and read by the NodeManager for log aggregation
yarn.nodemanager.linux-container-executor.group=yarn<configured value of yarn.nodemanager.linux-container-executor.group>
banned.users=hdfs,yarn,mapred,bin<comma separated list of users who can not run applications>
min.user.id=1000<prevent other super-users>
```

### Extra security services

* Ranger
* Knox

#### Ranger

* delivers a comprehensive approach to security for a Hadoop cluster
* Provides central security policy administration across the core enterprise security requirements of authorization, audit and data protection
* Works with Kerberos, LDAP and AD; all users, groups, policies and rules are stored in Ranger Server
* Works for HDFS, Yarn, Hive, HBase, Storm, Knox, Solr, Kafka and others which support Ranger plugin
* As any security field in Hadoop, this is very not trivial too

#### Knox

* a single point of secure access for Hadoop clusters, forwards requests, providing such features as
* Simplified access – extend Hadoop’s REST services by encapsulating Kerberos within the cluster
* Enhanced security - expose Hadoop’s REST services without revealing network details, with SSL provided out of box
* Centralized control - centrally enforce REST API security and route requests to multiple Hadoop clusters Enterprise integration – support LDAP, SSO, SAML and other authentication systems

## MapReduce

* replaced in prod with Spark and Flink
* Distributed data processing model and execution environment that runs on large clusters of commodity machines
* Parallel processing of blocks of files stored in HDFS on multiple nodes (computers in cluster)
* MapReduce is one of the several Yarn applications that runs batchjobs

### MR concepts

* Automatic parallelization and distribution
* Fault-tolerance
* Status and monitoring tools
* Clean abstraction for programmers
* MapReduce programs are usually written in Java, but can also be written in any scripting language using Hadoop Streaming
* All of Hadoop is written in Java
* MapReduce abstracts all the 'housekeeping' away
* Developer can concentrate simply on writing the Map and Reduce functions
* Scalable file processing

**Map task**

* Process input files blocks
* Produce (key-value) pairs
* Executed in parallel

**Reduce task**

* Take (key-value) pairs sorted by keys
* Combine/aggregate them to produce final result
* Can be zero, one or more (executed in parallel)

### Map reduce architecture

* Input data is stored in HDFS, spread across nodes and replicated.
* MR framework consists of single master JobTracker and one slave TaskTracker per node
* Master schedules job components tasks on slaves, monitor them and reexecuted failed ones.
* Slaves execute tasks directed by Master
* Application submits job (mapper, reducer, input) to job tracker
* Job tracker splits input data into independent chunks
* Job tracker schedules and monitors various map and reduce tasks and reexecute failed ones
* Task Tracker execute map and reduce tasks

### Mapper

* Mapper maps input key/value pairs to a set of intermediate key/value pairs.
* organize data to further processing of reduce phase
* map(inKey, inValue) -> list(intermediateKey, intermediateValue)
* inKey = data record
* inValue = offset of the data record from the begining of the data file
* output - collection of k/v pairs

`class MyMapper extends MapReduceBase implements Mapper<Object, Text, Text, IntWritable>`

**Types**

* Identity mapper. Implements mapper and maps inputs directly to outputs
* Inverse mapper. implements mapper and reverses the key value pair
* Regex mapper.  implements mapper and generates a (match, 1) pair for every regular expression match.
* Token count mapper. It implements mapper and generates(Token,1) pair when the input is tokenized.

### Reducer

* reduce(intermediateKey, list(intermediateValue)) -> list(outKey, outValue)
* each reduce function processes the intermediate values for a particular key generated by the map function and generates the output.

`class MyReducer extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable>`

* sorts data on k-v pair, groups together all values of the same key iterating values with given key
* outputCollector retreives output of reducer process and writes into output file
* reporter provides extra info about reducer and task processes

**Types**

* Identify Reducer. It implements a reducer\[key,value] and map inputs directly to the outputs.
* Long sum reducer. It implements a reducer\[key, long writable,] to get the given key

MR

* handcode each operation
* no interactive mode
* no iterative processing
