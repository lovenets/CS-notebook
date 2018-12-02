Spark's Architecture

![Spark architecture](img\Spark architecture.jpg)

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

# Spark SQL

## Dataset & DataFrame

A Dataset is a distributed collection of data. Dataset is a new interface added in Spark 1.6 that provides the benefits of RDDs (strong typing, ability to use powerful lambda functions) with the benefits of Spark SQL’s optimized execution engine. A Dataset can be [constructed](https://spark.apache.org/docs/latest/sql-getting-started.html#creating-datasets) from JVM objects and then manipulated using functional transformations (`map`, `flatMap`, `filter`, etc.).

A DataFrame is a *Dataset* organized into named columns. It is conceptually equivalent to a table in a relational database or a data frame in R/Python, but with richer optimizations under the hood.  In [the Scala API](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset), `DataFrame` is simply a type alias of `Dataset[Row]`.

## Basic

```scala
    val spark = SparkSession.builder.appName("SQL Example").getOrCreate()
    // for implicit conversions like converting RDDS to DataFrames
    import spark.implicits._

    // create a DataFrame from a JSON file
    val df = spark.read.json("people.json")
    df.show()

    // print the schema in a tree format
    df.printSchema()
    // select a specific column
    df.select("name").show()
    // select multiple columns and "update" values using S-Expressions
    // NOTE: select method will not actually modify values
    df.select($"name", $"age" + 1).show()
    // using filtering conditions
    df.filter($"age" > 21).show()
    // do some statistics
    df.groupBy("age").count().show()

    // register the DataFrame as a SQL temporary view
    df.createOrReplaceTempView("people")
    val sqlDF = spark.sql("SELECT name,age FROM people WHERE age > 18")
    sqlDF.show()

    // encoders are created for case classes
    val caseClassDS = Seq(People("Andy", 32)).toDS()
    caseClassDS.show()
    // encoders for most common types are imported implicitly
    val primitiveDS = Seq(1, 2, 3).toDS()
    primitiveDS.map(_ + 1).collect().foreach(print)
    // DataFrames can be converted to a Dataset by providing a class.
    // Mapping will be done by name
    val peopleDS = spark.read.json("people.json").as[People]
    peopleDS.show()

    // create an RDD of People objects from a text file
    // using case class People as schema
    val peopleDF = spark.sparkContext
      .textFile("people.txt")
      .map(_.split(","))
      .map(attributes => People(attributes(0), attributes(1).trim.toInt))
      .toDF()
    peopleDF.createOrReplaceTempView("people")
    val teenagersDF = spark.sql("SELECT name,age FROM people WHERE age BETWEEN 13 AND 19")
    // we can access columns by field index or by field name
    teenagersDF.map(t => "Name: " + t(0)).show()
    teenagersDF.map(t => "Name: " + t.getAs[String]("name")).show()
    // we must define encoders for Dataset[Map[K,V]] explicitly
    implicit val mapEncoder = org.apache.spark.sql.Encoders.kryo[Map[String, Any]]
    teenagersDF.map(t => t.getValuesMap[Any](List("name", "age"))).collect().foreach(println)

    // Programmatically Specifying the Schema
    val peopleRDD = spark.sparkContext.textFile("people.txt")
    // the schema is encoded in a string then generate the schema
    // based on the strings of schema
    val schemaString = "name age"
    val fields = schemaString.split(" ")
      .map(fieldName => StructField(fieldName, StringType, nullable = true))
    val schema = StructType(fields)
    // convert records of RDDs to Rows
    val rowRDD = peopleRDD
      .map(_.split(","))
      .map(attr => Row(attr(0), attr(1).trim))
    // apply the schema to RDD
    val peopleDF1 = spark.createDataFrame(rowRDD,schema)
    peopleDF1.show()
    
    spark.stop()
```

1.`SparkSession`

The entry point into all functionality in Spark is the [`SparkSession`](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SparkSession) class. If you have some configurations :

```scala
val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .getOrCreate()
```

`SparkSession` in Spark 2.0 provides builtin support for Hive features including the ability to write queries using HiveQL, access to Hive UDFs, and the ability to read data from Hive tables. To use these features, you do not need to have an existing Hive setup.

2.creating DataFrames

With a `SparkSession`, applications can create DataFrames from an [existing `RDD`](https://spark.apache.org/docs/latest/sql-getting-started.html#interoperating-with-rdds), from a Hive table, or from [Spark data sources](https://spark.apache.org/docs/latest/sql-data-sources.html).

When we print a DataFrame, we will find that it looks like a table in RDB. That's why we call DataFrames `Datasets of rows`.

3.DataFrame operations

DataFrames provide a domain-specific language for structured data manipulation.

For a complete list of the types of operations that can be performed on a Dataset refer to the [API Documentation](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Dataset).

In addition to simple column references and expressions, Datasets also have a rich library of functions including string manipulation, date arithmetic, common math operations and more. The complete list is available in the [DataFrame Function Reference](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.functions$).

4.running SQL programmatically

 The `sql` function on a `SparkSession` enables applications to run SQL queries programmatically and returns the result as a `DataFrame`.

5.create Datasets

WHY?

Datasets are similar to RDDs, however, instead of using Java serialization or Kryo they use a specialized [Encoder](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.Encoder) to serialize the objects for processing or transmitting over the network. While both encoders and standard serialization are responsible for turning an object into bytes, encoders are code generated dynamically and use a format that allows Spark to **perform many operations like filtering, sorting and hashing without deserializing the bytes back into an object.**

6.converting RDDs into Datasets

(1) Inferring the Schema Using Reflection

The case class defines the schema of the table. The names of the arguments to the case class are read using reflection and become the names of the columns. Case classes can also be nested or contain complex types such as `Seq`s or `Array`s. This RDD can be implicitly converted to a DataFrame.

(2) Programmatically Specifying the Schema

When case classes cannot be defined ahead of time (for example, the structure of records is encoded in a string, or a text dataset will be parsed and fields will be projected differently for different users), a `DataFrame` can be created programmatically with three steps.

- Create an RDD of `Row`s from the original RDD;

- Create the schema represented by a `StructType` matching the structure of `Row`s in the RDD created in last step.

- Apply the schema to the RDD of `Row`s via `createDataFrame` method provided by `SparkSession`.

7.aggregatioin

(1) Untyped User-Defined Aggregate Functions

Users have to extend the [UserDefinedAggregateFunction](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.UserDefinedAggregateFunction) abstract class to implement a custom untyped aggregate function.

```scala
// $example on:untyped_custom_aggregation$
import org.apache.spark.sql.{Row, SparkSession}
import org.apache.spark.sql.expressions.MutableAggregationBuffer
import org.apache.spark.sql.expressions.UserDefinedAggregateFunction
import org.apache.spark.sql.types._
// $example off:untyped_custom_aggregation$

object UserDefinedUntypedAggregation {

  // $example on:untyped_custom_aggregation$
  object MyAverage extends UserDefinedAggregateFunction {
    // Data types of input arguments of this aggregate function
    def inputSchema: StructType = StructType(StructField("inputColumn", LongType) :: Nil)
    // Data types of values in the aggregation buffer
    def bufferSchema: StructType = {
      StructType(StructField("sum", LongType) :: StructField("count", LongType) :: Nil)
    }
    // The data type of the returned value
    def dataType: DataType = DoubleType
    // Whether this function always returns the same output on the identical input
    def deterministic: Boolean = true
    // Initializes the given aggregation buffer. The buffer itself is a `Row` that in addition to
    // standard methods like retrieving a value at an index (e.g., get(), getBoolean()), provides
    // the opportunity to update its values. Note that arrays and maps inside the buffer are still
    // immutable.
    def initialize(buffer: MutableAggregationBuffer): Unit = {
      buffer(0) = 0L
      buffer(1) = 0L
    }
    // Updates the given aggregation buffer `buffer` with new input data from `input`
    def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
      if (!input.isNullAt(0)) {
        buffer(0) = buffer.getLong(0) + input.getLong(0)
        buffer(1) = buffer.getLong(1) + 1
      }
    }
    // Merges two aggregation buffers and stores the updated buffer values back to `buffer1`
    def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
      buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
      buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
    }
    // Calculates the final result
    def evaluate(buffer: Row): Double = buffer.getLong(0).toDouble / buffer.getLong(1)
  }
  // $example off:untyped_custom_aggregation$

  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder()
      .appName("Spark SQL user-defined DataFrames aggregation example")
      .getOrCreate()

    // $example on:untyped_custom_aggregation$
    // Register the function to access it
    spark.udf.register("myAverage", MyAverage)

    val df = spark.read.json("examples/src/main/resources/employees.json")
    df.createOrReplaceTempView("employees")
    df.show()
    // +-------+------+
    // |   name|salary|
    // +-------+------+
    // |Michael|  3000|
    // |   Andy|  4500|
    // | Justin|  3500|
    // |  Berta|  4000|
    // +-------+------+

    val result = spark.sql("SELECT myAverage(salary) as average_salary FROM employees")
    result.show()
    // +--------------+
    // |average_salary|
    // +--------------+
    // |        3750.0|
    // +--------------+
    // $example off:untyped_custom_aggregation$

    spark.stop()
  }

}
```

(2) Type-Safe User-Defined Aggregate Functions

User-defined aggregations for strongly typed Datasets revolve around the [Aggregator](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.expressions.Aggregator) abstract class. 

```scala
// $example on:typed_custom_aggregation$
import org.apache.spark.sql.{Encoder, Encoders, SparkSession}
import org.apache.spark.sql.expressions.Aggregator
// $example off:typed_custom_aggregation$

object UserDefinedTypedAggregation {

  // $example on:typed_custom_aggregation$
  case class Employee(name: String, salary: Long)
  case class Average(var sum: Long, var count: Long)

  object MyAverage extends Aggregator[Employee, Average, Double] {
    // A zero value for this aggregation. Should satisfy the property that any b + zero = b
    def zero: Average = Average(0L, 0L)
    // Combine two values to produce a new value. For performance, the function may modify `buffer`
    // and return it instead of constructing a new object
    def reduce(buffer: Average, employee: Employee): Average = {
      buffer.sum += employee.salary
      buffer.count += 1
      buffer
    }
    // Merge two intermediate values
    def merge(b1: Average, b2: Average): Average = {
      b1.sum += b2.sum
      b1.count += b2.count
      b1
    }
    // Transform the output of the reduction
    def finish(reduction: Average): Double = reduction.sum.toDouble / reduction.count
    // Specifies the Encoder for the intermediate value type
    def bufferEncoder: Encoder[Average] = Encoders.product
    // Specifies the Encoder for the final output value type
    def outputEncoder: Encoder[Double] = Encoders.scalaDouble
  }
  // $example off:typed_custom_aggregation$

  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder()
      .appName("Spark SQL user-defined Datasets aggregation example")
      .getOrCreate()

    import spark.implicits._

    // $example on:typed_custom_aggregation$
    val ds = spark.read.json("examples/src/main/resources/employees.json").as[Employee]
    ds.show()
    // +-------+------+
    // |   name|salary|
    // +-------+------+
    // |Michael|  3000|
    // |   Andy|  4500|
    // | Justin|  3500|
    // |  Berta|  4000|
    // +-------+------+

    // Convert the function to a `TypedColumn` and give it a name
    val averageSalary = MyAverage.toColumn.name("average_salary")
    val result = ds.select(averageSalary)
    result.show()
    // +--------------+
    // |average_salary|
    // +--------------+
    // |        3750.0|
    // +--------------+
    // $example off:typed_custom_aggregation$

    spark.stop()
  }

}
```

## Data Sources

1.Generic Load/Save Functions

In the simplest form, the default data source is Apache Parquet. 

```scala
val usersDF = spark.read.load("users.parquet")
```

If you want to load other data source, you need to add some extra options.

```scala
val peopleDF = spark.read.format("json").load("people.json")
val peopleDFCsv = spark.read.format("csv")
  .option("sep", ";")
  .option("inferSchema", "true")
  .option("header", "true")
  .load("people.csv")
```

Data sources are specified by their fully qualified name (i.e., `org.apache.spark.sql.parquet`), but for built-in sources you can also use their short names (`json`, `parquet`, `jdbc`, `orc`, `libsvm`, `csv`, `text`). DataFrames loaded from any data source type can be converted into other types using this syntax.

2.Run SQL on files directly

Instead of using read API to load a file into DataFrame and query it, you can also query that file directly with SQL.

```scala
val sqlDF = spark.sql("SELECT * FROM parquet.`users.parquet`")
```

NOTE: Specify data source using FROM clause like [type].`path`.

3.Bucketing, Sorting and Partitioning

For file-based data source, it is also possible to bucket and sort or partition the output. Bucketing and sorting are applicable only to persistent tables:

```scala
peopleDF.write.bucketBy(42, "name").sortBy("age").saveAsTable("people_bucketed")
```

while partitioning can be used with both `save` and `saveAsTable` when using the Dataset APIs.

```scala
usersDF.write.partitionBy("favorite_color").format("parquet").save("namesPartByColor.parquet")
```

It is possible to use both partitioning and bucketing for a single table:

```scala
usersDF
  .write
  .partitionBy("favorite_color")
  .bucketBy(42, "name")
  .saveAsTable("users_partitioned_bucketed")
```

`partitionBy` creates a directory structure as described in the [Partition Discovery](https://spark.apache.org/docs/latest/sql-data-sources-parquet.html#partition-discovery) section. Thus, it has limited applicability to columns with high cardinality. In contrast `bucketBy` distributes data across a fixed number of buckets and can be used when a number of unique values is unbounded.

4.save modes

![save modes](img\save modes.jpg)

```scala
peopleDFCsv.write.mode("").save("")
```

5.JSON files

Note that the file that is offered as *a json file* is not a typical JSON file. Each line must contain a separate, self-contained valid JSON object. 

Spark SQL can automatically infer the schema of a JSON dataset and load it as a `Dataset[Row]`. 

```scala
    // For a regular multi-line JSON file, set the multiLine option to true.
    spark.read.option("multiLine ","true").json(path)
```

6.JDBC

To get started you will need to include the JDBC driver for your particular database on the spark classpath.

In SBT, you can add the dependency like this:

```scala
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-sql" % "2.3.2",
  "mysql" % "mysql-connector-java" % "8.0.13"
)
```

```scala
    // load data from DB using load method
    val jdbcDF = spark.read
      .format("jdbc")
      .option("url", "jdbc:mysql://localhost:3306/db")
      .option("dbtable", "table")
      .option("user", "username")
      .option("password", "password")
      .option("serverTimezone", "UTC")
      .load()
    jdbcDF.show()
    // load data from DB using jdbc method
    val connectProp = new Properties()
    connectProp.put("user", "username")
    connectProp.put("password", "password")
    connectProp.put("serverTimezone", "UTC")
    val jdbcDF2 = spark.read
      .jdbc("jdbc:mysql://localhost:3306/db", "table", connectProp)
    jdbcDF2.show()

    // save data to a JDBC source using save method
    jdbcDF.write
      .format("jdbc")
      .option("url", "jdbc:postgresql:dbserver")
      .option("dbtable", "schema.tablename")
      .option("user", "username")
      .option("password", "password")
      .save()
    // save data using jdbc method
    jdbcDF2.write
      .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectProp)
```



