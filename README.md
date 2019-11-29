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
A table is defined with a set of column families + a column for row key.
Columns are added at filling time on the fly.
No need to fill empty fields with Null values.

### HBase on HDFS
Rows in a table are distributed in subsets called regions, stored in the worker nodes of the cluster.
Everything is stored in binary format.

### HBase components
HBase master : responsible for creating a table and communicating with slaves. Usually with a stand by master attached for high availability.
Region servers : slaves.
Zookeeper for high availability.

If both masters think they are masters, we enter a split brain scenario, it is always very dangerous => needs shutdown.
To avoid that, we require a zookeeper quorum to choose the master server => choose an even number of zookeepers (usually 3 or 5) to avoid equality. If one zookeeper is down, require absolute majority (2/2 or 3/4).

When a client starts writing on a region server :
- it starts writing events (row added for instance) on a WAL file : events are added one after the other, it is not a file modification.
- data is stored in RAM in a Memcache
- when Memcache is full, it is flushed in an HFile.

When a client reads data :
- region server fetches the data from HFile and joins it with Memcache if any.
Remember we are in HDFS, everything is stored in blocks. So an HFile may be made of 100 blocks.
When data is read from a block, the whole block is loaded in RAM to anticipate future needs.
Data is stored according to row key.

=> It is important do design row key adequately to improve access speed and save RAM. 
