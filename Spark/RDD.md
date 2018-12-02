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



