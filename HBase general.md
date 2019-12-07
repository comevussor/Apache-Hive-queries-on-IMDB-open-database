# HBase

## Design
A table is defined with a set of column families + a column for row key.
Columns are added at filling time on the fly.
No need to fill empty fields with Null values.

## HBase on HDFS
Rows in a table are ordered by row key, distributed in subsets called regions, stored in the worker nodes of the cluster.
Everything is stored in binary format.

## HBase components
HBase master : responsible for creating a table and communicating with slaves. Usually with a stand by master attached for high availability.
Region servers : slaves, answering the client, notifying the master, compacting
Zookeeper for high availability.

If both masters think they are masters, we enter a split brain scenario, it is always very dangerous => needs shutdown.
To avoid that, we require a zookeeper quorum to choose the master server => choose an even number of zookeepers (usually 3 or 5) to avoid equality. If one zookeeper is down, require absolute majority (2/2 or 3/4).

When a client starts writing on a region server :
- it starts writing events (row added for instance) on a WAL file : events are added one after the other, it is not a file modification.
- data is stored in RAM in a Memstore
- when Memstore is full, it is flushed in an HFile.

Region servers are also in charge of compaction : 
- small compaction : bring together small HFiles 
- major compaction : (once a week for instance) take all HFiles of a region and create one HFile per column family

When a client reads data :
- region server fetches the data from HFile and joins it with Memstore (most recent data, used as a buffer) if any.
Remember we are in HDFS, everything is stored in blocks. So an HFile may be made of 100 blocks.
When data is read from a block, the whole block is loaded in RAM to anticipate future needs.
Data is stored according to row key.

=> It is important do design row key adequately to improve access speed and save RAM. 

=> HBase is very useful when mostly working with rows. While Hive is useful to work with columns, agregations.

## Create and fill a table

I'm using an edge node of an Hadoop cluster.

Open HBase shell :

(linux) `hbase shell`

We want to create a table named judgement with 2 column families : opinion and meta :

(HBase shell) `create 'judgement', 'opinion', 'meta'`

Fill the table :

```
put 'judgement', 'tt001_001', 'opinion:vote', '7'
put 'judgement', 'tt001_001', 'opinion:comments', 'Good enough'
put 'judgement', 'tt001_001', 'meta:title', 'tt001'
put 'judgement', 'tt001_001', 'meta:date', '20181215'
```

For the row key, we could do something like : title+vote+username (or username hash for anonymity)

View data : `scan 'judgement'`

yields 
```
ROW                       COLUMN+CELL
 tt001_001                column=meta:date, timestamp=1575293588720, value=20181215
 tt001_001                column=meta:title, timestamp=1575293575790, value=tt001
 tt001_001                column=opinion:comments, timestamp=1575293553830, value=Good enough
 tt001_001                column=opinion:vote, timestamp=1575293406626, value=7
1 row(s)
```

Get one row only :

`get 'judgement', 'tt001_001'`

If the vote is to be changed to 8 for instance :

`put 'judgement', 'tt001_001', 'opinion:vote', '8'`

This will append the new data without removing the previous one. Timestamping will ensure that at reading time, the updated data is loaded.

## Query HBase

* HBase shell
* SQL
  * Apache Phoenix will translate SQL into low level HBase api for OLTP commands BUT not OLAP otherwise aggregation will be done locally, not distributed => big problems out of memory. It also allows ACID transactions 
  * Hive for OLAP : Hiver does not handle the storage, we just create an external table with adequate columns (=> restrict freedom of creating columns)

If a region server crash, we loose Memcache and blockcache. WAL and HFile is on HDFS, still available.
The region will be reassigned to another server which will download WAL and HFile.
The WAL is replayed by the new region server to get the missing RAM data.
During this process, the data is not available...
