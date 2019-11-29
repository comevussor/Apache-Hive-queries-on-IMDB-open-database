# Hadoop-and-Spark-ecosystems


## CAP theorem
For any big data base we want :
- Consistent : only one possible result for one query at a certain time
- Available : data is directly available without waiting for some synchronization time
- Partition tolerant

BUT unfortunately, only 2 conditions out of 3 can be achieved simulaneously !

When dealing with big data, we need partition tolerance. Therefore we have to choose between availability and consistency.
For selling online, we would rather choose availability but for banking, we need consistency.

HBase is a consistent (CP) type of DB. Cassandra is available (AP).
