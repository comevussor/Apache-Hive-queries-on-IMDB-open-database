# Hadoop-and-Spark-ecosystems

The content of this repository is my personal work based on the course given by Adaltas in DSTI in october 2019.

## CAP theorem
For any big data base we want :
- Consistency : only one possible result for one query at a certain time
- Availability : data is directly available without waiting for some synchronization time
- Partition tolerance

BUT unfortunately, only 2 conditions out of 3 can be achieved simulaneously !

When dealing with big data, we need partition tolerance. Therefore we have to choose between availability and consistency.
For selling online, we would rather choose availability but for banking, we need consistency.

HBase is a consistent (CP) type of DB. Cassandra is available (AP).

## HBase
### Design
A table is defined with a set of column families.
Columns are added at filling time on the fly.
No need to fill empty fields with Null values.

### HBase on HDFS
Rows in a table are distributed in subsets called regions, stored in the worker nodes of the cluster.

### HBase components
HBase master : responsible for creating a table and communicating with slaves.
Region servers : slaves.
