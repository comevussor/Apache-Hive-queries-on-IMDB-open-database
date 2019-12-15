Content :

[Hadoop ecosystem](#hadoop-ecosystem) : [Hadoop solution](#hadoop-solution), [For business](#for-business)







# Hadoop ecosystem

## What is Big Data ?
Big Data is characterized by :
- Variety : structured, semi-structured, unstructured data => need to link, mach, cleanse, transform
- Volume : tera 10^12, peta 10^15, exa 10^18 => storage cost, question of data relevance
- Velocity : need for fast ingestion, fast analysis, address periodic peaks

Processing big data requires to distribute data accross multiple machines in **Data Centers**.

A **cluster** is a set of interconnected servers in a data center.

## Hadoop solution
Distribution of data and processing implies :
- a specific file system : Hadoop Distributed File System **HDFS** (java based)
- specific databases : NoSQL, HBase, ElasticSearch
- map and reduce parallelized algorithms running on Yet Another Resource Negociator **YARN**

We use Hortonworks Hadoop Distribution : **Hortonworks Data Platform**
- Operations
  - Apache Ambari : web-based framework to provision, manage and monitor Hadoop cluster
  - Apache Zookeeper : coordination service for distributed applications
  - Hortonworks Cloudbreak : provision and manage Hadoop cluster in the cloud
  - Apache Oozie : workflow scheduler for Hadoop jobs
- Integration
  - Apache Hadoop WebHDFS : REST API to manage HDFS through HTTP
  - Apache Flume : collect, aggregate, move streaming data (removed)
  - Apache Sqoop : import and export data between Hadoop and RDBMS
  - Apache Atlas : governance services to meet compliance and data integration requirements
- Streaming
  - Apache **NiFi** : directed graphs of data routing, transformation and system mediation logic
  - Apache **Kafka** : publish/subscribe messaging system
  - Apache Flink : stateful computation over unbounded and bounded data streams
  - Apache Storm : realtime processing of unbounded data
- Security
  - Apache Knox : gateway providing perimeter security to a Hadoop cluster
  - Apache Ranger : fine-grained policy controls for HDFS, Hive, HBase, Knox, Storm, Kafka, Solr
  - (Kerberos protocol : authentication through tickets developped by MIT)
- Data access
  - Apache Pig : extract, transform and analyze datasets (forget it)
  - Apache **Hive** : data warehouse (repository) allowing SQL queries
  - Apache **HBase** : NoSQL database that supports structured data storage for large tables
  - Apache Phoenix : SQL layer to porovie low-latency access to HBase data
  - Apache Solr : distributed search platform to index PB of data (built on Apache Lucene)
  - Apache **Spark** : general purpose processing engine to build and run SQL, streaming, machine learning, or graphic applications
  - Apache **Zeppelin** : web-based notebook for data analytics (many possible languages)

Example of Basic architecture :
- All nodes should have same OS and config
- Master nodes : each master has a stand-by twin, Zookeeper is installed on an even number of master nodes
  - Name Node : managing HDFS to locate files
  - Resource Manager : handling YARN
  - Hiver Server
  - HBase master
- Utility nodes
  - Knox with client gateway, connected to web services through HTTP
  - Ambari Server with client gateway
- Many worker nodes, each one with a Node Manager (for YARN) and a Data Node (for HDFS). If HBase is present, they also have a Region Server.
- Edge nodes : used to connect with masters through clients like Beeline for HBase.

## For Business
Business strategies :
- data discovery : diversify internal and external sources
- single view : build dashboards to watch priorities
- decide based on predictive analytics : reduce uncertainty, prevent failures, predict capacity requirements, recommend customers, no surprise

Cost saving approach :
- active archive : lower storage cost, store any kind of data, long term storage, ingest more, easy to access old data
- extract transform load : better capacity allocation, don't ignore difficult to transform data, get access to raw data, value all data
- data enrichment : combine data sources

# CAP theorem

For any big data base we want :
- Consistency : only one possible result for one query at a certain time
- Availability : data is directly available without waiting for some synchronization time
- Partition tolerance

BUT unfortunately, only 2 conditions out of 3 can be achieved simulaneously !

When dealing with big data, we need partition tolerance. Therefore we have to choose between availability and consistency.
For selling online, we would rather choose availability but for banking, we need consistency.

HBase is a consistent (CP) type of DB. Cassandra is available (AP).

# ACID transactions 
- Atomicity : transactions are processed at once, no intermediate operation
- Consistency : succeed or come back to previous state
- Isolation : different users don't see each other's work
- Durability : once transaction is done, database is updated even if there is a crash

# HBase

## Design
A table is defined with a set of column families + a column for row key.
Columns are added at filling time on the fly.
No need to fill empty fields with Null values.

## HBase on HDFS
Rows in a table are ordered by row key, distributed in subsets called regions, stored in the worker nodes of the cluster.
Everything is stored in binary format.

## HBase components
- HBase master : responsible for creating a table and communicating with slaves. Usually with a stand by master attached for high availability.
- Region servers : slaves, answering the client, notifying the master, compacting
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

# Security with Kerberos

Authentication mechanism by tickets allowing users to access services and services to communicate between each-other. Kerberos realms allow different clusters to be distributed on the same machines. A realm can "trust" another realm, means it delegates authentication.

Kerberos main component : Key Distribution Center KDC

KDC has 2 components :
- authentication service AS
- Ticket Granting Service GTS

Each entity (user) has a principle : name@realm .

- see my ticket : `klist`
- request a ticket : `kinit`, sends a Ticket Granting Ticket (TGT) request to AS, receives an initial ticket.
- delete a ticket : `kdestroy`

If I try `hdfs dfs -ls`, it will sent a TGT to TGS and get a specific ticket (Ticket Service TS) from it. I sent my request with TS to Name Node (NN) to get my request processed.

example : use Kerberos to authenticate on a webserver. After authentication, SPNEGO protocol allows putting TGT in http protocol through base64 encapsulating (put anyting into string) of TS into a header authentication : Negociate + token with token = base64(TS).

**Apache Knox** service encapsulates SPNGO into LDAP authentication.

Apache Ranger deals with authorization :
- HDFS : who can access (r, w, x)
- YARN : who, in which queue
- Hive tables
- HBase tables
- Knox topologies
- Kafka topics

Any service in the cluster as a Ranger plugin

In the end, security is ensured like that :
- identification is ensured by LDAP
- authentication is ensured by Kerberos
- authorisation is ensured by Ranger

# Streaming on Hadoop

We are dealing with :
- unbounded data sets (produced continuously)
- unbounded data processing (in time)
- low-latency, approximate and speculative results

Used tools :
- distributed messaging systems
  - publish/consume messages to/from queues
  - fault-tolerant
- distributed streaming processing engines
  - exactly-once fault-tolerant processing
  - aggregations, event-time windows

Architectures :
- Lambda architecture : separate batch processing (Hive) from streaming
- Kappa architecture : based on Kafka messaging system, build a sequence of queues with some processing in between each one.

Main opensource solutions : Apache Kafka and RabbitMQ. Both rely on publish-subscribe model. Kafka does not re-route messages (ssmart consumer )and is good for hard storage while RabbitMQ can re-route (dumb consumer) messages between queues but bad at hard storage.

## Apache Kafka

- Components are called brokers (servers).
- Queues are called topics that can be partitionned on different brokers.
- Producers and consumers can send to or receive from different brokers.
- Mid/long term storage
- Good integration with Hadoop ecosystem (Spark, Hive) : can query Kafka topics with Hive
    
Stream processing problematics
- event time vs processing time : difference is called skew
- we define time "windows" :
  - fixed : compute every 10 minutes
  - sliding windows : one 20' window every 10'
  - sessions : based on activity
- time marking : 
  - watermark : gives a deadline (processing time P) from which it is assumed that all data generated prior to an event time E have been observed, not always true, may discard some messages
  - trigger : time to materialize results
  
Dataflow model (Google model) :
- what results ar calulated ? aggregations
- where in event time are result calculated ? windows
- when in processing time are results materialized ? trigger (eg. every 10' if there is a new event)
- how do refinements of results relate ? 
  - discarding : send each result separately
  - accumulating : send the aggregated result
  - both
  
Streaming (Apache Flink each event is computed separately, Google dataflow) vs micro-batching (Spark streaming) 

## Data flows

- Manage flow of information between systems :
  - routing/filtering
  - parsing
  - transforming
- not targeting latency but throughput + delivery guaranties

This comes before the datalake.

## Apache NiFi

- Flow file : Any incoming data is considered as a file with content, attributes, provenance.
- Processors
- Back pressure mechanism : if a queue is full, it will redirect the flow to another queue so that nothing is lost
