# Spark

## RDD

At a high level, every Spark application consists of a *driver program* that runs the user’s `main` function and executes various *parallel operations* on a cluster.

The main abstraction Spark provides is a *resilient distributed dataset* (RDD), which is a collection of elements partitioned across the nodes of the cluster that can be operated on in parallel.

RDDs are created by starting with a file in the Hadoop file system (or any other Hadoop-supported file system), or an existing Scala collection in the driver program, and transforming it.

 Users may also ask Spark to *persist* an RDD in memory, allowing it to be reused efficiently across parallel operations. Finally, RDDs automatically recover from node failures.



A second abstraction in Spark is *shared variables* that can be used in parallel operations. 

By default, when Spark runs a function in parallel as a set of tasks on different nodes, it ships a copy of each variable used in the function to each task. Sometimes, a variable needs to be shared across tasks, or between tasks and the driver program. 

Spark supports two types of shared variables: *broadcast variables*, which can be used to cache a value in memory on all nodes, and *accumulators*, which are variables that are only “added” to, such as counters and sums.



Spark revolves around the concept of a *resilient distributed dataset* (RDD), which is a fault-tolerant collection of elements that can be operated on in parallel. 

There are two ways to create RDDs: *parallelizing* an existing collection in your driver program, or referencing a dataset in an external storage system, such as a shared filesystem, HDFS, HBase, or any data source offering a Hadoop InputFormat.



### Parallelized Collections

Parallelized collections are created by calling `SparkContext`’s `parallelize` method on an existing collection in your driver program

One important parameter for parallel collections is the number of *partitions* to cut the dataset into. Spark will run one task for each partition of the cluster. Typically you want 2-4 partitions for each CPU in your cluster. 

### External Datasets

Some notes on reading files with Spark:

- If using a path on the local filesystem, the file must also be accessible at the same path on worker nodes. Either copy the file to all workers or use a network-mounted shared file system.
- All of Spark’s file-based input methods, including `textFile`, support running on directories, compressed files, and wildcards as well.
- The `textFile` method also takes an optional second argument for controlling the number of partitions of the file. By default, Spark creates one partition for each block of the file (blocks being 128MB by default in HDFS), but you can also ask for a higher number of partitions by passing a larger value. Note that you cannot have fewer partitions than blocks.

Apart from text files, Spark’s Scala API also supports several other data formats:

- `SparkContext.wholeTextFiles` lets you read a directory containing multiple small text files, and returns each of them as (filename, content) pairs. This is in contrast with `textFile`, which would return one record per line in each file. Partitioning is determined by data locality which, in some cases, may result in too few partitions. For those cases, `wholeTextFiles` provides an optional second argument for controlling the minimal number of partitions.
- For [SequenceFiles](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html), use SparkContext’s `sequenceFile[K, V]` method where `K` and `V` are the types of key and values in the file. These should be subclasses of Hadoop’s [Writable](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/io/Writable.html) interface, like [IntWritable](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/io/IntWritable.html) and [Text](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/io/Text.html). In addition, Spark allows you to specify native types for a few common Writables; for example, `sequenceFile[Int, String]` will automatically read IntWritables and Texts.
- For other Hadoop InputFormats, you can use the `SparkContext.hadoopRDD` method, which takes an arbitrary `JobConf` and input format class, key class and value class. Set these the same way you would for a Hadoop job with your input source. You can also use `SparkContext.newAPIHadoopRDD` for InputFormats based on the “new” MapReduce API (`org.apache.hadoop.mapreduce`).
- `RDD.saveAsObjectFile` and `SparkContext.objectFile` support saving an RDD in a simple format consisting of serialized Java objects. While this is not as efficient as specialized formats like Avro, it offers an easy way to save any RDD.

### RDD Operations

RDDs support two types of operations: *transformations*, which create a new dataset from an existing one, and *actions*, which return a value to the driver program after running a computation on the dataset.

All transformations in Spark are *lazy*, in that they do not compute their results right away.

Instead, they just remember the transformations applied to some base dataset (e.g. a file). 

The transformations are only computed when an action requires a result to be returned to the driver program. This design enables Spark to run more efficiently.

By default, each transformed RDD may be recomputed each time you run an action on it. 

However, you may also *persist* an RDD in memory using the `persist` (or `cache`) method, in which case Spark will keep the elements around on the cluster for much faster access the next time you query it. 

There is also support for persisting RDDs on disk, or replicated across multiple nodes.

#### Basics 



```scala
val lines = sc.textFile("data.txt")
val lineLengths = lines.map(s => s.length)
//lineLengths.persist()
val totalLength = lineLengths.reduce((a, b) => a + b)
```

#### Passing Functions to Spark

Spark’s API relies heavily on passing functions in the driver program to run on the cluster.

There are two recommended ways to do this:

- 匿名函数

- 单例对象静态方法

  ```scala
  object MyFunctions {
    def func1(s: String): String = { ... }
  }
  
  myRdd.map(MyFunctions.func1)
  ```

- 可以传递传递实例对象的方法，但不推荐，会导致整个类对象发送至集群

  ```scala
  class MyClass {
    def func1(s: String): String = { ... }
    def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(func1) }
  }
  ```

#### Understanding closures

```scala
var counter = 0
var rdd = sc.parallelize(data)

// Wrong: Don't do this!!
rdd.foreach(x => counter += x)

println("Counter value: " + counter)
```



##### Local vs. cluster modes

The behavior of the above code is undefined, and may not work as intended. 

To execute jobs, Spark breaks up the processing of RDD operations into tasks, each of which is executed by an executor.

 Prior to execution, Spark computes the task’s **closure**. 

The closure is those variables and methods which must be visible for the executor to perform its computations on the RDD (in this case `foreach()`). 

This closure is serialized and sent to each executor.

The variables within the closure sent to each executor are now copies and thus, when **counter** is referenced within the `foreach` function, it’s no longer the **counter** on the driver node.

 There is still a **counter** in the memory of the driver node but this is no longer visible to the executors! The executors only see the copy from the serialized closure. 

Thus, the final value of **counter** will still be zero since all operations on **counter** were referencing the value within the serialized closure.

##### Printing elements of an RDD

Another common idiom is attempting to print out the elements of an RDD using `rdd.foreach(println)` or `rdd.map(println)`. 

On a single machine, this will generate the expected output and print all the RDD’s elements. However, in `cluster` mode, the output to `stdout` being called by the executors is now writing to the executor’s `stdout` instead, not the one on the driver, so `stdout` on the driver won’t show these! 

To print all elements on the driver, one can use the `collect()` method to first bring the RDD to the driver node thus: `rdd.collect().foreach(println)`.

 This can cause the driver to run out of memory, though, because `collect()` fetches the entire RDD to a single machine; if you only need to print a few elements of the RDD, a safer approach is to use the `take()`: `rdd.take(100).foreach(println)`.

#### Working with Key-Value Pairs

#### Transformations 

#### Actions

#### Shuffle operations

In Spark, data is generally not distributed across partitions to be in the necessary place for a specific operation. 

During computations, a single task will operate on a single partition - thus, to organize all the data for a single `reduceByKey` reduce task to execute, Spark needs to perform an all-to-all operation.

 It must read from all partitions to find all the values for all keys, and then bring together values across partitions to compute the final result for each key - this is called the **shuffle**.



Although the set of elements in each partition of newly shuffled data will be deterministic, and so is the ordering of partitions themselves, the ordering of these elements is not. If one desires predictably ordered data following shuffle then it’s possible to use:

- ```scala
  mapPartitions
  ```

- ```
  repartitionAndSortWithinPartitions
  ```

- ```
  sortBy
  ```

Operations which can cause a shuffle include **repartition** operations like [`repartition`](http://spark.apache.org/docs/latest/rdd-programming-guide.html#RepartitionLink) and [`coalesce`](http://spark.apache.org/docs/latest/rdd-programming-guide.html#CoalesceLink), **‘ByKey** operations (except for counting) like [`groupByKey`](http://spark.apache.org/docs/latest/rdd-programming-guide.html#GroupByLink) and [`reduceByKey`](http://spark.apache.org/docs/latest/rdd-programming-guide.html#ReduceByLink), and **join** operations like [`cogroup`](http://spark.apache.org/docs/latest/rdd-programming-guide.html#CogroupLink) and [`join`](http://spark.apache.org/docs/latest/rdd-programming-guide.html#JoinLink).

###### Performance Impact

The **Shuffle** is an expensive operation since it involves disk I/O, data serialization, and network I/O.

 To organize data for the shuffle, Spark generates sets of tasks - *map* tasks to organize the data, and a set of *reduce* tasks to aggregate it.

 This nomenclature comes from MapReduce and does not directly relate to Spark’s `map` and `reduce` operations.

Internally, results from individual map tasks are kept in memory until they can’t fit. 

Then, these are sorted based on the target partition and written to a single file. On the reduce side, tasks read the relevant sorted blocks.

Certain shuffle operations can consume significant amounts of heap memory since they employ in-memory data structures to organize records before or after transferring them. 

Specifically, `reduceByKey` and `aggregateByKey` create these structures on the map side, and `'ByKey` operations generate these on the reduce side.

 When data does not fit in memory Spark will spill these tables to disk, incurring the additional overhead of disk I/O and increased garbage collection.

Shuffle also generates a large number of intermediate files on disk. 

As of Spark 1.3, these files are preserved until the corresponding RDDs are no longer used and are garbage collected. This is done so the shuffle files don’t need to be re-created if the lineage is re-computed. 

Garbage collection may happen only after a long period of time, if the application retains references to these RDDs or if GC does not kick in frequently.

 This means that long-running Spark jobs may consume a large amount of disk space. 

The temporary storage directory is specified by the `spark.local.dir` configuration parameter when configuring the Spark context.

### RDD Persistence

One of the most important capabilities in Spark is *persisting* (or *caching*) a dataset in memory across operations.

 When you persist an RDD, each node stores any partitions of it that it computes in memory and reuses them in other actions on that dataset (or datasets derived from it). 

This allows future actions to be much faster (often by more than 10x). 

Caching is a key tool for iterative algorithms and fast interactive use.

You can mark an RDD to be persisted using the `persist()` or `cache()` methods on it. 

The first time it is computed in an action, it will be kept in memory on the nodes. 

Spark’s cache is fault-tolerant – if any partition of an RDD is lost, it will automatically be recomputed using the transformations that originally created it.

In addition, each persisted RDD can be stored using a different *storage level*, allowing you, for example, to persist the dataset on disk, persist it in memory but as serialized Java objects (to save space), replicate it across nodes. 

These levels are set by passing a `StorageLevel` object ([Scala](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.storage.StorageLevel), [Java](http://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/storage/StorageLevel.html), [Python](http://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.StorageLevel)) to `persist()`.

 The `cache()` method is a shorthand for using the default storage level, which is `StorageLevel.MEMORY_ONLY` (store deserialized objects in memory). The full set of storage levels is:

| **Storage Level**                         | **Meaning**                                                  |
| ----------------------------------------- | ------------------------------------------------------------ |
| MEMORY_ONLY                               | Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, some partitions will not be cached and will be recomputed on the fly each time they're needed. This is the default level. |
| MEMORY_AND_DISK                           | Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, store the partitions that don't fit on disk, and read them from there when they're needed. |
| MEMORY_ONLY_SER<br />(Java and Scala)     | Store RDD as *serialized* Java objects (one byte array per partition). This is generally more space-efficient than deserialized objects, especially when using a [fast serializer](http://spark.apache.org/docs/latest/tuning.html), but more CPU-intensive to read. |
| MEMORY_AND_DISK_SER<br />(Java and Scala) | Similar to MEMORY_ONLY_SER, but spill partitions that don't fit in memory to disk instead of recomputing them on the fly each time they're needed. |
| DISK_ONLY                                 | Store the RDD partitions only on disk.                       |
| MEMORY_ONLY_2, MEMORY_AND_DISK_2          | Same as the levels above, but replicate each partition on two cluster nodes. |
| OFF_HEAP (experimental)                   | Similar to MEMORY_ONLY_SER, but store the data in [off-heap memory](http://spark.apache.org/docs/latest/configuration.html#memory-management). This requires off-heap memory to be enabled. |

Spark also automatically persists some intermediate data in shuffle operations (e.g. `reduceByKey`), even without users calling `persist`. 

This is done to avoid recomputing the entire input if a node fails during the shuffle.

 We still recommend users call `persist` on the resulting RDD if they plan to reuse it.

#### Which Storage Level to Choose?

Spark’s storage levels are meant to provide different trade-offs between memory usage and CPU efficiency. We recommend going through the following process to select one:

- If your RDDs fit comfortably with the default storage level (`MEMORY_ONLY`), leave them that way. This is the most CPU-efficient option, allowing operations on the RDDs to run as fast as possible.
- If not, try using `MEMORY_ONLY_SER` and [selecting a fast serialization library](http://spark.apache.org/docs/latest/tuning.html) to make the objects much more space-efficient, but still reasonably fast to access. (Java and Scala)
- Don’t spill to disk unless the functions that computed your datasets are expensive, or they filter a large amount of the data. Otherwise, recomputing a partition may be as fast as reading it from disk.
- Use the replicated storage levels if you want fast fault recovery (e.g. if using Spark to serve requests from a web application). *All* the storage levels provide full fault tolerance by recomputing lost data, but the replicated ones let you continue running tasks on the RDD without waiting to recompute a lost partition.

#### Removing Data

Spark automatically monitors cache usage on each node and drops out old data partitions in a least-recently-used (LRU) fashion.

 If you would like to manually remove an RDD instead of waiting for it to fall out of the cache, use the `RDD.unpersist()` method.

### Shared Variables

Normally, when a function passed to a Spark operation (such as `map` or `reduce`) is executed on a remote cluster node, it works on separate copies of all the variables used in the function. 

These variables are copied to each machine, and no updates to the variables on the remote machine are propagated back to the driver program. 

Supporting general, read-write shared variables across tasks would be inefficient. 

However, Spark does provide two limited types of *shared variables* for two common usage patterns: broadcast variables and accumulators

#### Broadcast Variables

Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks. 

They can be used, for example, to give every node a copy of a large input dataset in an efficient manner. Spark also attempts to distribute broadcast variables using efficient broadcast algorithms to reduce communication cost.

Spark actions are executed through a set of stages, separated by distributed “shuffle” operations. Spark automatically broadcasts the common data needed by tasks within each stage.

 The data broadcasted this way is cached in serialized form and deserialized before running each task.

 This means that explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in deserialized form is important.

```scala
scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
broadcastVar: org.apache.spark.broadcast.Broadcast[Array[Int]] = Broadcast(0)

scala> broadcastVar.value
res0: Array[Int] = Array(1, 2, 3)
```

After the broadcast variable is created, it should be used instead of the value `v` in any functions run on the cluster so that `v` is not shipped to the nodes more than once. 

In addition, the object `v` should not be modified after it is broadcast in order to ensure that all nodes get the same value of the broadcast variable (e.g. if the variable is shipped to a new node later).

#### Accumulators 

Accumulators are variables that are only “added” to through an associative and commutative operation and can therefore be efficiently supported in parallel.

 They can be used to implement counters (as in MapReduce) or sums. Spark natively supports accumulators of numeric types, and programmers can add support for new types.

A numeric accumulator can be created by calling `SparkContext.longAccumulator()` or `SparkContext.doubleAccumulator()` to accumulate values of type Long or Double, respectively. 

Tasks running on a cluster can then add to it using the `add` method. However, they cannot read its value. Only the driver program can read the accumulator’s value, using its `value` method.



For accumulator updates performed inside **actions only**, Spark guarantees that each task’s update to the accumulator will only be applied once, i.e. restarted tasks will not update the value.

 In transformations, users should be aware of that each task’s update may be applied more than once if tasks or job stages are re-executed.

Accumulators do not change the lazy evaluation model of Spark. 

If they are being updated within an operation on an RDD, their value is only updated once that RDD is computed as part of an action.

 Consequently, accumulator updates are not guaranteed to be executed when made within a lazy transformation like `map()`. The below code fragment demonstrates this property: