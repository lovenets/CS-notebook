## DAG

When you create an RDD, a direct acyclic graph is created. Transformations will update the graph, but nothing will happen until some actions are called.

### Display a DAG

We can view a DAG by calling `toDebugString` method.

```scala
val data = 1 to 100000
val distData = sc.parallelize(data)
val DAG = distData.filter(_ % 2 == 0).map(_ + "").toDebugString
println(DAG)
```

Output:

```
(1) MapPartitionsRDD[2] at map at test.scala:21 []
 |  MapPartitionsRDD[1] at filter at test.scala:21 []
 |  ParallelCollectionRDD[0] at parallelize at test.scala:20 []
```

We read it from the bottom up. It starts with a data source and then a series of transformations.

## Distributed operations

![what happened when an action is executed (1)](img\what happened when an action is executed (1).jpg)

![what happened when an action is executed (2)](img\what happened when an action is executed (2).jpg)

Then executors read HDFS blocks to prepare the data for the operations in parallel.

![what happened when an action is executed (3)](img\what happened when an action is executed (3).jpg)

![what happened when an action is executed (4)](img\what happened when an action is executed (4).jpg)

![what happened when an action is executed (5)](img\what happened when an action is executed (5).jpg)

Exercise:

1.Find the most frequent word in a text and  its occurring times.

```scala
    val mostFrequentWord = text.flatMap(line => line.split(" "))
      .map((_, 1))
      .reduceByKey(_ + _)
      .map(t => (t._2, t._1))
      .max()
```

2.Find the most frequent character in a text and its occurring times.

```scala
    val mostFrequentChar = text.flatMap(line => line.split(" ").flatMap(_.toCharArray))
      .map(c => (c, 1))
      .reduceByKey(_ + _)
      .reduce((t1, t2) => if (t1._2 > t2._2) t1 else t2)
```

Tip: use `flatMap` to collect lots of collections into a big collection.

## Partitioning mechanism

![generating an RDD](img\generating an RDD.jpg)

![RDD example](img\RDD example.jpg)

Each RDD in a graph has a reference to its dependencies, up to the root RDD which dependents on nothing. The root RDD describes the original partitioning and these partitions are inherited by child RDDs. 

![change partitioning](img\change partitioning.jpg)

### What affects partitioning

![partitioning considerations](img\partitioning considerations.jpg)

### Optimize Partitioning

#### keys co-located

When operating key-value pairs, we can benefit from keeping records with the same keys co-located. This lets transformations run quickly within the same JVM when operating over the same keys. 

![partitionBy](img\partitionBy.jpg)

Calling `partitionBy` will incur a shuffle but downstream operations will benefit from the co-located records. In the case of `HashPartitioner` we specify the number of partitions and it will ensure that all keys with the same hash will be in the same partition.

![optimizing joins by co-partitioning](img\optimizing joins by co-partitioning.jpg)

## Resilience

When  a failure occurs, RDD will re-compute results from root/persisted RDD so RDD is resilient.

![resilience1](img\resilience1.jpg)

![resilience](img\resilience.jpg)

## Serializing

### "Task Not Serializable"

Following codes will not work well:

```scala
class Helper {
    def foo(input: String): String = input
}

val helper = new Helper()

val output = input.map(helper.foo(_))
```

Remember that codes will be serialized and sent to nodes. The problem is that “Helper” doesn’t implement Serializable. So if we extend our class with `Serializable` and try to run it again it will work properly.

What if “Helper” in turn is also referencing another 3rd party library as a member variable which is not serializable? We may not want (or be able to) change the code to make it serializable. The solution is marking the member variable transient and lazy. This will make it instantiated locally within each task.

```scala
class Helper extends Serializable {
    @transient lazy val inner = new External()
    
    def foo(input: String): String = inner.bar(input)
}

val helper = new Helper()

val output = input.map(helper.foo(_))
```

## Avoid shuffling

**Shuffling** is a process of [redistributing data across partitions](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-rdd-partitions.html) (aka *repartitioning*) that may or may not cause moving data across JVM processes or even over the wire (between executors on separate machines).

By default, shuffling doesn’t change the number of partitions, but their content.

**NOTE**: Avoid shuffling at all cost. Think about ways to leverage existing partitions. Leverage partial aggregation to reduce data transfer.

- Avoid `groupByKey` and use `reduceByKey` or `combineByKey` instead.
  - `groupByKey` shuffles all the data, which is slow.
  - `reduceByKey` shuffles only the results of sub-aggregations in each partition of the data.

## Persistence

![persistence](img\persistence.jpg)

It's a good idea to persist before map and reduce actions.

![persistence2](img\persistence2.jpg)

