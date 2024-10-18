# Spark. Environment

Here are the differences between `spark.sql.shuffle.partitions` and `spark.default.parallelism`:

* **spark.sql.shuffle.partitions**:
  * Default: 200.
  * Controls the number of partitions to use when shuffling data for SQL operations.
  * Primarily used in Spark SQL and DataFrame operations.
* **spark.default.parallelism**:
  * Default: Number of cores on all executor nodes.
  * Controls the default number of partitions in RDDs when not set explicitly by the user.
  * Used for operations like `reduceByKey` and `groupByKey` in RDDs.

In summary, `spark.sql.shuffle.partitions` is specific to SQL and DataFrame shuffling, while `spark.default.parallelism` is a more general setting for RDD operations.
