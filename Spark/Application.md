# Spark Context

`SparkContext`is the main entry point of Spark functionality, which represents the connection to a Spark cluster. In a Spark program, import some classes and implicit conversions:

```scala
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.SparkContext._
```

# Initializing Spark

```scala
    // create a SparkContext object, which tells Spark how to access a cluster
    // a SparkConf object that contains information about your application.
    val conf = new SparkConf().setAppName("demo").setMaster("local")
    val sc = new SparkContext(conf)
```

`setMaster` will provide a Spark, Mesos, or YARN URL (or a special "local" string to run in local mode). In production mode, do not hardcode master in the program. Launch with `spark-submit`and provide it there.

# Passing functions 

There are three ways to pass functions in the driver program.

(1) anonymous function syntax

(2) static methods in a global singleton object

```scala
object MyFunctions {
    def foo (s: String): String = {
        //...
    }
}
```

(3) passing by reference

To avoid send the entire object, consider copying the function to a local variable.

```scala
val field = "aaa"
def doStuff(rdd: RDD[String]): RDD[String] = {
    val field_ = this.field
    rdd.map(x => field_ + x)
}
```

# Submit applications to a cluster

You can write any Spark applications using SBT to manage dependencies.

After you complete coding a Spark application, you can package it into a JAR. Then you can use `spark-submit`command, which is located under the $SPARK_HOME/bin directory to submit your app. Type `spark-submit`in the terminal and then you will see some help.

```shell
spark-submit [options] <app jar | python file | R file> [app arguments]
```

There are some important options:

- `--class`: The main entry point to your class. If it is under a package name,
  you must provide the fully qualified name.

- `--master`: The master URL is where your cluster is located.
  Remember that it is recommended approach to provide the master URL here, instead of hardcoding it in your application code.

- `--deploy-mode`: Whether you want to deploy your driver on the worker nodes (cluster) or locally as an external client (client). The default mode is client.

- `--conf`: Any configuration property you wish to set in key=value format.

  Finally, if the application has any arguments, you would supply it after the jar file.

**Example**

1.Create a SBT project

A typical SBT project's directory will look like:

```
.build.sbt
./src
./src/main
./src/main/scala
```

2.Package the project into a .jar

Run SBT command `sbt package`in the root of the directory.

3.Use `spark-submit`

```shell
spark-submit --class "SparkPi" --master local[4] sbtspark_2.11-0.1.jar
```

`local[4]`means using 4 cores of the local machine.

# Test

![test](img\test.jpg)

**Example: Testing in Intellij IDEA**

1. On the project pane on the left, expand `src` => `main`.

2. Right-click on `scala` and select **New** => **Scala class**.

3. Call it `CubeCalculator`, change the **Kind** to `object`, and click **OK**.

4. Replace the code with the following:

   ```scala
   object Count {
     def countWord(input: String) = {
       val conf = new SparkConf().setAppName("test").setMaster("local")
         .set("spark.driver.allowMultipleContexts","true")
       val sc = new SparkContext(conf)
   
       sc.makeRDD(Seq(input))
         .flatMap(_.split(" "))
         .map((_,1))
         .reduceByKey(_ + _)
         .collectAsMap()
     }
   }
   ```

5. On the project pane on the left, expand `src` => `test`.

6. Right-click on `scala` and select **New** => **Scala class**.

7. Replace the code with the following:

```scala
class SampleTestClass extends FunSuite {
  test("word count") {
    val input = "hello world world"
    val expected = Map("hello"->1,"world"->2)

    assert(Count.countWord(input).equals(expected))

    assertResult(expected) {
      Count.countWord(input)
    }
  }
}
```

8. Run the test class.