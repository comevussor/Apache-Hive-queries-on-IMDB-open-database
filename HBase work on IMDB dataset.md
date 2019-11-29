# HBase based on IMDB dataset

## Create and fill a table

I'm using an edge node of an Hadoop cluster.

Open HBase shell :

(linux) `hbase shell`

We want to create a table named judgement with 2 column families : opinion and meta :

(HBase shell) `create 'judgement', 'opinion', 'meta'`

Fill the table :

`put 'judgement', 'tt001_001', 'opinion:vote', '7', 'opinion:comments', 'Good enough', 'meta:title', 'tt001', 'meta:date', '20181215'`

For the row key, we could do something like : title+vote+username (or username hash for anonymity)

View data : `scan 'judgement'`

Get one row only :

`get 'judgement', 'tt001_001'`

If the vote is to be changed to 8 for instance :

`put 'judgement', 'tt001_001', 'opinion:vote', '8', 'opinion:comments', 'Good enough', 'meta:title', 'tt001', 'meta:date', '20181215'`

This will append the new data without removing the previous one. Date stamping will ensure that at reading time, the updated data is loaded.

## Query HBase

* HBase shell
* SQL
  * Apache Phoenix will translate SQL into low level HBase api for OLTP commands BUT not OLAP otherwise aggregation will be done locally, not distributed => big problems out of memory.
  * Hive for OLAP : Hiver does not handle the storage, we just create an external table with adequate columns (=> restrict freedom of creating columns)

If a region server crash, we loose Memcache and blockcache. WAL and HFile is on HDFS, still available.
The region will be reassigned to another server which will download WAL and HFile.
The WAL is replayed by the new region server to get the missing RAM data.
During this process, the data is not available...
