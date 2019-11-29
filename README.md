# Hadoop-and-Spark-ecosystems


## CAP theorem
For any big data base we want :
- Consistency : only one possible result for one query at a certain time
- Availability : data is directly available without waiting for some synchronization time
- Partition tolerance

BUT unfortunately, only 2 conditions out of 3 can be achieved simulaneously !

When dealing with big data, we need partition tolerance. Therefore we have to choose between availability and consistency.
For selling online, we would rather choose availability but for banking, we need consistency.

HBase is a consistent (CP) type of DB. Cassandra is available (AP).
