# Spark.Stages

#### Additional Metrics in Stages Tab

1. **Scheduler Delay**
   * Time taken by the scheduler to submit the task and for the task to be launched on the executor.
2. **Task Deserialization Time**
   * Time taken to deserialize the task on the executor.
3. **GC Time**
   * Time spent in garbage collection during the task execution.
4. **Result Serialization Time**
   * Time taken to serialize the task result before sending it back to the driver.
5. **Getting Result Time**
   * Time taken to fetch the task result by the driver.
6. **Peak Execution Memory**
   * Peak memory usage by the task.
7. **Shuffle Read Metrics**
   * **Remote Blocks Fetched**: Number of remote blocks fetched in shuffle.
   * **Local Blocks Fetched**: Number of local blocks fetched in shuffle.
   * **Fetch Wait Time**: Time spent waiting for fetch.
   * **Remote Bytes Read**: Bytes read remotely.
   * **Local Bytes Read**: Bytes read locally.
8. **Shuffle Write Metrics**
   * **Shuffle Bytes Written**: Total bytes written during shuffle.
   * **Shuffle Write Time**: Total time taken to write shuffle data.

Deserialization of tasks in Spark is needed because tasks, which are created and serialized on the driver, need to be transferred over the network to the executors. Once received, executors deserialize these tasks to execute them. This process ensures that the tasks can be effectively transmitted across distributed systems and run in parallel on different executors.

#### Difference Between Fetched and Read in Shuffle Read Metrics

1. **Shuffle Read Fetched**:
   * Refers to the number of blocks of data fetched from remote nodes during the shuffle phase. It measures the amount of data transferred over the network.
2. **Shuffle Read Read**:
   * Refers to the total amount of data read locally and remotely during the shuffle phase. This includes data read from local disk as well as data fetched from other nodes.

In the context of Spark, "remote" during shuffle refers to data that is fetched from nodes other than the local node where the task is running.

#### Correspondence with Locality Levels:

1. **Process Local**: Data is on the same executor process.
2. **Node Local**: Data is on the same node but different executor process.
3. **Rack Local**: Data is on a different node but within the same rack.
4. **Any (Remote)**: Data is on a different rack or data center.

"Remote" corresponds to the "Any" locality level, where data is fetched across network boundaries.

* During the shuffle phase, the executor writes data that needs to be shuffled to its local disk.
* Other executors then read this data from the local disk of the executor that wrote it.

Here are some common reasons why Spark might skip stages during execution:

* **RDD Caching**: If the RDDs in the stage have already been computed and cached, Spark can skip the recomputation.
* **Failure and Retry**: Spark can skip stages if it detects that recomputation is unnecessary due to job failures and retries.
* **Data Locality**: Spark might skip stages if the data is already available in the desired location, avoiding unnecessary computation.
