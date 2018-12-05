Spark libraries are the extension of core API and are built on the top of it.

# Spark SQL

```scala
  case class Weather(date: String,temp: Int,precipitation: Double)

  private def runSQL = {
    val spark = SparkSession.builder.appName("SQL Example").getOrCreate()
    // for implicit conversions like converting RDDS to DataFrames
    import spark.implicits._

    val weather = spark.read.textFile("LabData/nycweather.csv")
      .map(_.split(","))
      .map(w => Weather(w(0),w(1).trim.toInt,w(2).trim.toDouble))
      .toDF()

    weather.createOrReplaceTempView("weather")

    val hottest_with_percip = spark.sql("SELECT * FROM weather WHERE precipitation > 0.0 ORDER BY temp DESC")
    hottest_with_percip.collect()
    hottest_with_percip.map(x => ("Date: " + x(0), "Temp : " + x(1), "Precip: " + x(2))).take(10).foreach(println)

    spark.stop()
  }
```

**NOTE: ** The field names should be the same as column names.

# Spark Streaming

 ![Spark Streaming](img\Spark Streaming.jpg)

![Spark Streaming Internals](img\Spark Streaming Internals.jpg)

In a windowed computation, every time the window slides over a source of DStream, the source RDDs that falls within the window are combined and operated upon to produce the resulting RDD. There are two parameters for a sliding window. The **window length** is the duration of the window and the **sliding interval** is the interval in which the window operation is performed. Both of these must be in multiples of the batch interval of the source DStream.

**Example**

NOTE: add the dependency in SBT:

```scala
libraryDependencies += "org.apache.spark" % "spark-streaming_2.11" % "2.4.0"
```

And then you can initialize Spark Streaming:

```scala
    // To initialize a Spark Streaming program, a StreamingContext object has to be created
    // which is the main entry point of all Spark Streaming functionality.
    val conf = new SparkConf().setAppName("Spark Pi").setMaster("local")
    val ssc = new StreamingContext(conf, Seconds(1))
```

After a context is defined, you have to do the following.

- Define the input sources by creating input DStreams.
- Define the streaming computations by applying transformation and output operations to DStreams.
- Start receiving data and processing it using `streamingContext.start()`.
- Wait for the processing to be stopped (manually or due to any error) using `streamingContext.awaitTermination()`.
- The processing can be manually stopped using `streamingContext.stop()`.

##### Points to remember:

- Once a context has been started, no new streaming computations can be set up or added to it.
- Once a context has been stopped, it cannot be restarted.
- Only one StreamingContext can be active in a JVM at the same time.
- stop() on StreamingContext also stops the SparkContext. To stop only the StreamingContext, set the optional parameter of `stop()` called `stopSparkContext` to false.
- A SparkContext can be re-used to create multiple StreamingContexts, as long as the previous StreamingContext is stopped (without stopping the SparkContext) before the next StreamingContext is created.

Following is a complete example.

```scala
    // To initialize a Spark Streaming program, a StreamingContext object has to be created
    // which is the main entry point of all Spark Streaming functionality.
    val conf = new SparkConf().setAppName("Spark Pi").setMaster("local")
    val ssc = new StreamingContext(conf, Seconds(1))

    // create a new DStream
    // and then do something
    val lines= ssc.socketTextStream("localhost",9999)
    val words = lines.flatMap(_.split("" ))
    val pairs = words.map(w => (w,1))
    val wordCounts = pairs.reduceByKey(_ + _)
    println(wordCounts)

    // no real things happen until you tell it
    ssc.start()

    // wait for the computation to terminate
    ssc.awaitTermination()
```

# Spark MLlib

```scala
  private def runML = {
    val conf = new SparkConf().setAppName("ml").setMaster("local")
    val sc = new SparkContext(conf)
      
    // load and clean data
    val taxiData = sc.textFile("LabData/nyctaxisub.csv")
      .filter(_.contains("2013"))
      .filter(_.split(",")(3) != "")
      .filter(_.split(",")(4) != "")
    val taxiFence = taxiData.filter(_.split(",")(3).toDouble > 40.70)
      .filter(_.split(",")(3).toDouble < 40.86)
      .filter(_.split(",")(4).toDouble > (-74.02))
      .filter(_.split(",")(4).toDouble < (-73.93))

    // K-Means clustering
    val taxi = taxiFence.map(line => Vectors.dense(line.split(',').slice(3, 5).map(_.toDouble)))
    val iterationCount = 20
    val clusterCount = 3
    val model = KMeans.train(taxi, clusterCount, iterationCount)
    val clusterCenters = model.clusterCenters.map(_.toArray)
    clusterCenters.foreach(lines => println(lines(0), lines(1)))
  }
```

# Spark GraphX

GraphX is a new component in Spark for graphs and graph-parallel computation. At a high level, GraphX extends the Spark [RDD](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD) by introducing a new [Graph](https://spark.apache.org/docs/latest/graphx-programming-guide.html#property_graph) abstraction: a directed multigraph with properties attached to each vertex and edge.

```scala
    val conf = new SparkConf().setMaster("local").setAppName("graphx")
    val sc = new SparkContext(conf)

    val users = sc.textFile("LabData/users.txt")
      .map(line => line.split(","))
      .map(parts => (parts.head.toLong, parts.tail))
    // parse the edge data
    val followerGraph = GraphLoader.edgeListFile(sc, "LabData/followers.txt")
    // attach attributes
    val graph = followerGraph.outerJoinVertices(users) {
      case (uid, deg, Some(attrList)) => attrList
      case (uid, deg, None) => Array.empty[String]
    }
    val subgraph = graph.subgraph(vpred = (vid, attr) => attr.size == 2)
    val pagerankGraph = subgraph.pageRank(0.001)
    val userInfoWithPageRank = subgraph.outerJoinVertices(pagerankGraph.vertices) {
      case (uid, attrList, Some(pr)) => (pr, attrList.toList)
      case (uid, attrList, None) => (0.0, attrList.toList)
    }
    println(userInfoWithPageRank.vertices.top(5)(Ordering.by(_._2._1)).mkString("\n"))
```

