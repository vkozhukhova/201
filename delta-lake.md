# Delta Lake

I see in spark ui the job "Delta: filtering files for query". What does the job do?

The physical plan of related to this job sql query is as following:

\*(1) SerializeFromObject \[staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).path, true, false) AS path#7308, externalmaptocatalyst(lambdavariable(ExternalMapToCatalyst\_key, ObjectType(class java.lang.String), true, -1), staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, lambdavariable(ExternalMapToCatalyst\_key, ObjectType(class java.lang.String), true, -1), true, false), lambdavariable(ExternalMapToCatalyst\_value, ObjectType(class java.lang.String), true, -2), staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, lambdavariable(ExternalMapToCatalyst\_value, ObjectType(class java.lang.String), true, -2), true, false), knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).partitionValues) AS partitionValues#7309, knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).size AS size#7310L, knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).modificationTime AS modificationTime#7311L, knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).dataChange AS dataChange#7312, staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).stats, true, false) AS stats#7313, externalmaptocatalyst(lambdavariable(ExternalMapToCatalyst\_key, ObjectType(class java.lang.String), true, -3), staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, lambdavariable(ExternalMapToCatalyst\_key, ObjectType(class java.lang.String), true, -3), true, false), lambdavariable(ExternalMapToCatalyst\_value, ObjectType(class java.lang.String), true, -4), staticinvoke(class org.apache.spark.unsafe.types.UTF8String, StringType, fromString, lambdavariable(ExternalMapToCatalyst\_value, ObjectType(class java.lang.String), true, -4), true, false), knownnotnull(assertnotnull(input\[0, org.apache.spark.sql.delta.actions.AddFile, true])).tags) AS tags#7314]\
\+- \*(1) MapElements org.apache.spark.sql.Dataset\$$Lambda$3938/1904523958@66778949, obj#7307: org.apache.spark.sql.delta.actions.AddFile\
\+- \*(1) DeserializeToObject newInstance(class scala.Tuple1), obj#7306: scala.Tuple1\
\+- \*(1) Project \[add#567]\
\+- \*(1) Filter isnotnull(add#567)\
\+- \*(1) Scan ExistingRDD Delta Table State #101 - s3a://yac-sb-gen-rupn-data-00005-20230908143357373700000001/distilled/atlas/z2lis\_11\_vaitm/snapshot/\_delta\_log\[txn#566,add#567,remove#568,metaData#569,protocol#570,cdc#571,commitInfo#572]

Here's a detailed explanation of the physical plan when read from bottom to top:

1. **Scan ExistingRDD Delta Table State**:
   * This node scans the existing RDD (Resilient Distributed Dataset) representing the Delta Table state.
   * It reads from the `_delta_log` directory in the specified S3 path.
   * The attributes scanned include `txn`, `add`, `remove`, `metaData`, `protocol`, `cdc`, and `commitInfo`.
2. **Filter**:
   * This node filters out any null values in the `add` attribute.
3. **Project**:
   * This node selects the `add` attribute from the deserialized objects.
4. **DeserializeToObject**:
   * This node deserializes the data back into objects. It creates instances of `scala.Tuple1`.
5. **MapElements**:
   * This node applies a lambda function to map elements from the input data. In this case, it maps the `AddFile` object to the serialized format.
6. **SerializeFromObject**:
   * This node serializes the input objects into Spark SQL rows. The attributes being serialized include:
     * `path`: The file path.
     * `partitionValues`: The partition values.
     * `size`: The file size.
     * `modificationTime`: The modification time.
     * `dataChange`: A flag indicating if the file represents a data change.
     * `stats`: The statistics of the file.
     * `tags`: Tags associated with the file.

In summary, this physical plan starts by scanning the Delta Table state files, filters out null values, selects and deserializes the `add` attribute, maps it using a lambda function, and finally serializes the `AddFile` objects into Spark SQL rows. This confirms that the job is related to filtering delta log JSON files.

In what format does spark keeps rdd in memory?

Spark keeps RDD (Resilient Distributed Dataset) in memory in serialized format to optimize for space and performance. Specifically, it uses Java serialization or Kryo serialization for storing the data, which helps in reducing the space used by the data in memory.

I see in spark ui job "Delta: Compute snapshot for version 78". Physical plan of related sql query is as following:

\== Physical Plan ==\
CollectLimit 1\
\+- ObjectHashAggregate(keys=\[], functions=\[last(protocol#122, true), last(metaData#121, true), collect\_set(txn#118, 0, 0), sum(add#119.size), count(add#119), count(metaData#121), count(protocol#122), count(remove#120), count(txn#118)], output=\[protocol#148, metadata#150, setTransactions#152, sizeInBytes#154L, numOfFiles#156L, numOfMetadata#158L, numOfProtocol#160L, numOfRemoves#162L, numOfSetTransactions#164L])\
\+- Exchange SinglePartition, true, \[id=#87]\
\+- ObjectHashAggregate(keys=\[], functions=\[partial\_last(protocol#122, true), partial\_last(metaData#121, true), partial\_collect\_set(txn#118, 0, 0), partial\_sum(add#119.size), partial\_count(add#119), partial\_count(metaData#121), partial\_count(protocol#122), partial\_count(remove#120), partial\_count(txn#118)], output=\[last#224, valueSet#225, last#226, valueSet#227, buf#228, sum#229L, count#230L, count#231L, count#232L, count#233L, count#234L])\
\+- \*(1) Project \[txn#118, add#119, remove#120, metaData#121, protocol#122]\
\+- \*(1) Scan ExistingRDD Delta Table State #78 - s3a://yac-sb-gen-rupn-data-00003-20230411131136265600000002/distilled/mip/stockbalancereconcilation/snapshot/\_delta\_log\[txn#118,add#119,remove#120,metaData#121,protocol#122,cdc#123,commitInfo#124]

Here's a detailed explanation of the physical plan, read from bottom to top:

1. **Scan ExistingRDD Delta Table State**:
   * This node scans the existing RDD representing the Delta Table state.
   * It reads from the `_delta_log` directory in the specified S3 path.
   * The attributes scanned include `txn`, `add`, `remove`, `metaData`, `protocol`, `cdc`, and `commitInfo`.
2. **Project**:
   * This node selects the relevant attributes from the scanned data: `txn`, `add`, `remove`, `metaData`, and `protocol`.
3. **ObjectHashAggregate (partial)**:
   * This node performs a partial aggregation on the selected attributes.
   * It computes partial results for the following functions:
     * `partial_last(protocol#122, true)`: Partially computes the last protocol.
     * `partial_last(metaData#121, true)`: Partially computes the last metadata.
     * `partial_collect_set(txn#118, 0, 0)`: Partially collects a set of transactions.
     * `partial_sum(add#119.size)`: Partially sums the sizes of added files.
     * `partial_count(add#119)`: Partially counts the number of added files.
     * `partial_count(metaData#121)`: Partially counts the number of metadata entries.
     * `partial_count(protocol#122)`: Partially counts the number of protocol entries.
     * `partial_count(remove#120)`: Partially counts the number of removed files.
     * `partial_count(txn#118)`: Partially counts the number of transactions.
4. **Exchange**:
   * This node redistributes the data to a single partition. This is necessary for the final aggregation.
   * The `SinglePartition` exchange ensures that all data is collected to a single node for the final aggregation.
5. **ObjectHashAggregate (final)**:
   * This node performs the final aggregation on the partially aggregated results.
   * It computes the final results for the following functions:
     * `last(protocol#122, true)`: Computes the last protocol.
     * `last(metaData#121, true)`: Computes the last metadata.
     * `collect_set(txn#118, 0, 0)`: Collects a set of transactions.
     * `sum(add#119.size)`: Sums the sizes of added files.
     * `count(add#119)`: Counts the number of added files.
     * `count(metaData#121)`: Counts the number of metadata entries.
     * `count(protocol#122)`: Counts the number of protocol entries.
     * `count(remove#120)`: Counts the number of removed files.
     * `count(txn#118)`: Counts the number of transactions.
6. **CollectLimit**:
   * This node limits the result to a single row, which is the final output of the query.

In summary, this physical plan scans the Delta Table state from an S3 path, selects the relevant attributes, performs partial and final aggregations to compute various metrics (such as the last protocol/metadata, set of transactions, sum of file sizes, and counts of various entries), redistributes the data to a single partition for the final aggregation, and limits the result to one row. This process is used to compute the snapshot for version 78 of the Delta Table.
