# Spark.Jobs

In Apache Spark, some jobs have associated SQL queries while others do not due to the following reasons:

1. **Job Type**: Jobs triggered by SQL queries through Spark SQL will have an associated SQL query. Other jobs, like those triggered by direct RDD operations, will not have an associated SQL query.
2. **Execution Tracking**: SQL jobs are tracked and linked with SQL execution IDs, which are then displayed in the Spark UI. Non-SQL jobs do not have this linkage.
3. **Properties and Listeners**: SQL jobs set specific properties and listeners to track and display associated SQL queries, whereas non-SQL jobs do not utilize these mechanisms.

Examples of non-SQL jobs in Apache Spark include:

* Jobs created using RDD transformations like `map`, `filter`, and `reduce`.
* Jobs triggered by actions such as `collect`, `count`, and `saveAsTextFile`.
* Jobs involving machine learning algorithms from the Spark MLlib library.
* Graph processing jobs using GraphX.
* Stream processing jobs using DStream API in Spark Streaming.
