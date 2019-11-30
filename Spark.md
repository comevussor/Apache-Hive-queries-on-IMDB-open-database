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
- Notebook (Zeppelin)

## Spark core
- storage is based on Resilient distributed dataset (RDD) : collection of objects of different types
  - immutable (consistent with HDFS)
  - partitionned
  - persisted in memory (except if you don't use it for a while, it it flushed to HDFS)

Possible operations on RDD :
  - transformations : filtering, mapping, arrays to columns (flatmap) (all that with lambda function)
  
  Some transformations are only for key-value DB : reduceByKey
  
  - actions : collecting data (`collect()`), `count()`, collectin part of the data `take(n)`
  
Spark uses lazy evaluation : transformations will not be implemented until action is called.
