Spark's Architecture

![Spark architecture](G:\CS notebook\Spark\img\Spark architecture.jpg)

# Dataset

After Spark 2.0, RDDs are replaced by Dataset, which is strongly-typed like an RDD, but with richer optimizations under the hood. So Dataset is the main programming interface of Spark 2.

## Basics

Spark’s primary abstraction is a distributed collection of items called a Dataset. Datasets can be created from Hadoop InputFormats (such as HDFS files) or by transforming other Datasets.

Following demo will go through some basic operations of Dataset.

```scala
object Demo{
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder.appName("demo").getOrCreate()
    // Importing the SparkSession gives access to all the SQL functions and implicit conversions.
    import spark.implicits._

    // make a new Dataset from a text file
    val text = spark.read.textFile("D:/test.txt")

    // count the number of items in this Dataset
    println(s"the number of items : ${text.count()}")

    //transfer this Dataset to a new one
    // let's say we want to find the line with the most words

    val lineWithMostWords = text.map(line => line.split(" ").length).reduce((a,b) => Math.max(a,b))
    println(s"There are $lineWithMostWords words at most in a line.")

    // if data is accessed frequently, make it to be cached
    // we can do this even data is striped across thousands of nodes
    val cache = text.cache()
    println(s"the number of items : ${cache.count()}")

    spark.stop()
  }
}
```

**Note:** When you are not programming in shell, `import spark.implicits._` is important.

# RDD

 is a collection of elements partitioned across the nodes of the cluster that can be operated on in parallel. 

1. RDDs are created by starting with a file in the Hadoop file system (or any other Hadoop-supported file system), or an existing Scala collection in the driver program, and transforming it.
2. Spark can *persist* an RDD in memory, allowing it to be reused efficiently across parallel operations.
3. RDDs automatically recover from node failures.

```scala
    // create a SparkContext object, which tells Spark how to access a cluster
    // a SparkConf object that contains information about your application.
    val conf = new SparkConf().setAppName("demo").setMaster("local")
    val sc = new SparkContext(conf)

    // parallelize a collection
    val data = Array(1, 2, 3, 4, 5)
    val distData = sc.parallelize(data)
    println(s"The sum of distData is ${distData.sum()}")

    // create a RDD from external datasets
    // Spark will read a text file as a collection of lines
    // NOTE: distFile is just a pointer to the file
    val distFile = sc.textFile("D:/test.txt")

    val lineLengths = distFile.map(_.length)
    // save lineLengths in memory after the first time it's computed
    lineLengths.persist()
    val totalLength = lineLengths.reduce(_ + _)
    println(s"The sum of length of lines is $totalLength.")

    // count how many times each line of text occurs in a file
    val pairs = distFile.map(S => (S, 1))
    val counts = pairs.reduceByKey(_ + _)
    counts.foreach(println)

    // create a broadcast variable
    val broadcastVar = sc.broadcast(Array(1, 2, 3))
    // its value can be accessed by calling the value method
    broadcastVar.value.foreach(println)

    // create a numeric accumulator
    val accum = sc.longAccumulator("My Accumulator")
    sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum.add(x))
    println(s"The value of accumulator is ${accum.value}.")

    // before creating a new SparkContext object, you must stop the old one
    sc.stop()
```

**Note:**

## 1.`SparkConf`

The `appName` parameter is a name for your application to show on the cluster UI. `master` is a [Spark, Mesos or YARN cluster URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls), or a special “local” string to run in local mode. In practice, when running on a cluster, you will not want to hardcode `master` in the program, but rather [launch the application with `spark-submit`](https://spark.apache.org/docs/latest/submitting-applications.html) and receive it there. However, for local testing and unit tests, you can pass “local” to run Spark in-process.

## 2.parallel

One important parameter for parallel collections is the number of *partitions* to cut the dataset into. Spark will run one task for each partition of the cluster. Normally, Spark tries to set the number of partitions automatically based on your cluster. However, you can also set it manually by passing it as a second parameter to `parallelize` (e.g. `sc.parallelize(data, 10)`). 

## 3.read files

(1) If using a path on the local filesystem, the file must also be accessible at the same path on worker nodes. Either copy the file to all workers or use a network-mounted shared file system.

(2) you can use directories, compressed files or wildcards such as `textFile("/my/directory")`, `textFile("/my/directory/*.txt")`, and `textFile("/my/directory/*.gz")`.

(3) By default, Spark creates one partition for each block of the file (blocks being 128MB by default in HDFS), but you can also ask for a higher number of partitions by passing a larger value. Note that you cannot have fewer partitions than blocks.

(4) Spark’s Scala API also supports several other data formats: `SparkContext.wholeTextFiles`, `sequenceFile[K, V]` and son on.

## 4.RDD operations

RDDs support two types of operations: **transformations**, which create a new dataset from an existing one, and **actions**, which return a value to the driver program after running a computation on the dataset.

(1) All transformations in Spark are *lazy*, in that they do not compute their results right away. Instead, they just remember the transformations applied to some base dataset (e.g. a file).

(2) By default, each transformed RDD may be recomputed each time you run an action on it. However, you may also *persist* an RDD in memory using the `persist` (or `cache`) method, in which case Spark will keep the elements around on the cluster for much faster access the next time you query it.

## 5.closure

```scala
var counter = 0
var rdd = sc.parallelize(data)

// Wrong: Don't do this!!
rdd.foreach(x => counter += x)

println("Counter value: " + counter)
```

The behavior of the above code is undefined, and may not work as intended. To execute jobs, Spark breaks up the processing of RDD operations into tasks, each of which is executed by an executor. Prior to execution, Spark computes the task’s **closure**. The closure is those variables and methods which must be visible for the executor to perform its computations on the RDD (in this case `foreach()`). This closure is serialized and sent to each executor.

The variables within the closure sent to each executor are now copies and thus, when **counter** is referenced within the `foreach` function, it’s no longer the **counter** on the driver node. There is still a **counter** in the memory of the driver node but this is no longer visible to the executors! Thus, the final value of **counter** will still be zero.

```scala
rdd.foreach(println)
```

The above behavior can work on a single machine. However, in `cluster` mode, the output to `stdout` being called by the executors is now writing to the executor’s `stdout` instead, not the one on the driver, so `stdout` on the driver won’t show these! One can use the `collect()` method to first bring the RDD to the driver node thus: `rdd.collect().foreach(println)`, which can cause the driver to run out of memory, though, because `collect()` fetches the entire RDD to a single machine; if you only need to print a few elements of the RDD, a safer approach is to use the `take()`: `rdd.take(100).foreach(println)`.

## 6.K-V pairs

 A few special operations are only available on RDDs of key-value pairs. The most common ones are distributed “shuffle” operations, such as grouping or aggregating the elements by a key. In Scala, these operations are automatically available on RDDs containing [Tuple2](http://www.scala-lang.org/api/2.11.12/index.html#scala.Tuple2) objects (the built-in tuples in the language, created by simply writing `(a, b)`).

## 7.shuffle

A shuffle operation typically involves copying data across executors and machines, making the it a complex and costly operation.

For example, `reduceByKey` reduces task to execute, Spark needs to perform an all-to-all operation. It must read from all partitions to find all the values for all keys, and then bring together values across partitions to compute the final result for each key - this is called the **shuffle**.

(1) shuffle operations

Operations which can cause a shuffle include **repartition** operations like [`repartition`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#RepartitionLink) and [`coalesce`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#CoalesceLink), **‘ByKey** operations (except for counting) like [`groupByKey`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#GroupByLink) and [`reduceByKey`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#ReduceByLink), and **join** operations like [`cogroup`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#CogroupLink) and [`join`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#JoinLink).

(2) performance impact

The **Shuffle** is an expensive operation since it involves disk I/O, data serialization, and network I/O.

- Certain shuffle operations can consume significant amounts of heap memory since they employ in-memory data structures to organize records before or after transferring them. 
- Shuffle also generates a large number of intermediate files on disk. 

## 8.persistence

When you persist an RDD, **each node stores any partitions of it** that it computes in memory and reuses them in other actions on that dataset (or datasets derived from it). This allows future actions to be much faster (often by more than **10x**). Spark’s cache is fault-tolerant – if any partition of an RDD is lost, it will automatically be recomputed using the transformations that originally created it.

In addition, each persisted RDD can be stored using a different *storage level*, allowing you to store data in different places. These levels are set by passing a `StorageLevel` object ([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.storage.StorageLevel), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/storage/StorageLevel.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.StorageLevel)) to `persist()`. The `cache()` method is a shorthand for using the default storage level, which is `StorageLevel.MEMORY_ONLY` (store deserialized objects in memory).

NOTE: If you want to reuse it, all `persist` on the resulting RDD.

[Which Storage Level to Choose?](https://spark.apache.org/docs/latest/rdd-programming-guide.html#which-storage-level-to-choose)

## 9.shared variables

Normally, when a function passed to a Spark operation (such as `map` or `reduce`) is executed on a remote cluster node, it works on separate copies of all the variables used in the function. These variables are copied to each machine, and no updates to the variables on the remote machine are propagated back to the driver program. Supporting general, read-write shared variables across tasks would be inefficient. However, Spark does provide two limited types of *shared variables* for two common usage patterns: broadcast variables and accumulators.

### Broadcast Variables

Broadcast variables allow the programmer to keep a **read-only** variable cached on each machine rather than shipping a copy of it with tasks.

The data broadcasted this way is cached in serialized form and deserialized before running each task. This means that explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in deserialized form is important.

After the broadcast variable is created, it should be used instead of the value `v` in any functions run on the cluster so that `v` is not shipped to the nodes more than once. In addition, the object `v` should not be modified after it is broadcast in order to ensure that all nodes get the same value of the broadcast variable (e.g. if the variable is shipped to a new node later).

### Accumulators

 Accumulators are variables that are only “added” to through an associative and commutative operation and can therefore be efficiently supported in parallel. They can be used to implement **counters** (as in MapReduce) or **sums**. Spark natively supports accumulators of numeric types, and programmers can add support for new types.

Tasks running on a cluster can then add to it using the `add` method. However, they cannot read its value. Only the driver program can read the accumulator’s value, using its `value` method.

We can track accumulators in the web UI.

NOTE: 

1.For accumulator updates performed inside **actions only**, Spark guarantees that each task’s update to the accumulator will only be applied once, i.e. restarted tasks will not update the value. In transformations, users should be aware of that each task’s update may be applied more than once if tasks or job stages are re-executed.	

2.Accumulator updates are not guaranteed to be executed when made within a lazy transformation like `map()`.

```scala
val accum = sc.longAccumulator
data.map { x => accum.add(x); x }
// Here, accum is still 0 because no actions have caused the map operation to be computed.
```



