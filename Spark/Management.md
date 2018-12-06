# Configuration

In the default configuration directory(SPARK_HOME/conf), we can override some configuration files.

For example, change the INFO to ERROR in the log4j.properties to reduce verbose in the shell.

```properties
log4j.rootCategory=ERROR, console
```

## Spark properties

![Spark properties](img\Spark properties.jpg)

**Precedence**

Properties set directly on the `SparkConf` take highest precedence, then flags passed to `spark-submit` or `spark-shell`, then options in the `spark-defaults.conf` file

# Monitoring

![Spark monitoring](img\Spark monitoring.jpg)