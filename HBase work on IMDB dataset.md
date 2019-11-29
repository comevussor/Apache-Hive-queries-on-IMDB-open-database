# HBase work based on IMDB dataset

I'm using an edge node of an Hadoop cluster.

Open HBase shell :

(linux) `hbase shell`

We want to create a table named judgement with 2 column families : opinion and meta :

(HBase shell) `create 'judgement', 'opinion', 'meta'`

Fill the table :

`put 'judgement', 'tt001_001', 'opinion:vote', '7', 'opinion:comments', 'Good enough', 'meta:title', 'tt001', 'meta:date', '20181215'`

View data : `scan 'judgement'`

Get one row only :

`get 'judgement', 'tt001_001'`

If the vote is to be changed to 8 for instance :

`put 'judgement', 'tt001_001', 'opinion:vote', '8', 'opinion:comments', 'Good enough', 'meta:title', 'tt001', 'meta:date', '20181215'`

This will append the new data without removing the previous one. Date stamping will ensure that at reading time, the updated data is loaded.
