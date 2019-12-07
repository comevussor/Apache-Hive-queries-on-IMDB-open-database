# Spark

Having data in memory allows streaming, avoids I/O operations.

Spark is an in-memory process engine, running on Hadoop : can use data from HDFS, Hive..., can distribute processing.

It is open source and started in 2014, part of Apache.

Good for data engineering processing : cleaning, moving... For example, helps to merge different sources to normalize format.
Can also be used to be GDPR compliant.

## How to use Spark
Write with :
- Scala : more efficient
- Java : more people know it
- Python : easier to write, but it will be executed in Java in the end and now, overhead is only 10%

Submit job with :
- spark-shell pyspark
- `submit to spark`
- Notebook (Zeppelin) with keyword : `%spark2.pyspark`

## Spark core RDD
- storage is based on Resilient distributed dataset (RDD) : collection of objects of different types
  - immutable (consistent with HDFS)
  - partitionned
  - persisted in memory (except if you don't use it for a while, it it flushed to HDFS)

Possible operations on RDD :
  - transformations : filtering, mapping, arrays to columns (flatmap) (all that with lambda function)
  
  Some transformations are only for key-value DB : reduceByKey
  
  - actions : collecting data (`collect()`), `count()`, collectin part of the data `take(n)`

It is a very flexible type of objects, enabling work based on raw text for instance, data without schema. But for regular tabular data, (join 2 tables for example), when we have a schema, we'd rather work with DataFrames

Spark uses lazy evaluation : transformations will not be implemented until action is called.

## Spark internals

Spark application (also called the spark drive) : it is present on a singe machine of the cluster, responsible for distributing the work on the cluster through YARN. It also gives the spark context (`sc.`)

For a `map` operation, if my RDD is split into n partitions (usually one per block or per CPU), there will be n tasks launched in each executor of each of the n workers.

For a `reduceByKey` data will be shuffled first.

=> chaining `map`is not costly compared to reducing transformations.

## How to optimize the number of partitions of an RDD

By default, we have 1 partition per CPU, but it results in idle time to wait for the slowest.

Recommended value is 3 partitions per CPU : each CPU will perform 3 tasks one after the other and the differences between each CPU will average out, resulting in less idle time.

Adding too many partitions results in too much overhead.

## Dataframes

Dataframes store data in RDDs but with metadata (schema).

They can load data from different sources : Hive tables (transform and save under a new Hive table), HDFS (csv, json, xml).

By inheritance, dataframes are also immutable.




