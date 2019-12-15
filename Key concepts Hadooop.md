# Hadoop ecosystem

Big Data is characterized by :
- Variety : structured, semi-structured, unstructured data => need to link, mach, cleanse, transform
- Volume : tera 10^12, peta 10^15, exa 10^18 => storage cost, question of data relevance
- Velocity : need for fast ingestion, fast analysis, address periodic peaks

Processing big data requires to distribute data accross multiple machines in **Data Centers**.

A **cluster** is a set of interconnected servers in a data center.

Distribution of data and processing implies :
- a specific file system : Hadoop Distributed File System **HDFS** (java based)
- specific databases : NoSQL, HBase, ElasticSearch
- map and reduce parallelized algorithms running on Yet Another Resource Negociator **YARN**

We use Hortonworks Hadoop Distribution.

Main components :
- Operations
  - Ambari : web-based framework to provision, manage and monitor Hadoop cluster
  - Zookeeper : coordination service for distributed applications
  - Cloudbreak : provision and manage Hadoop cluster in the cloud
  - Oozie : workflow scheduler for Hadoop jobs
- Data access
  - Pig : extract, transform and analyze datasets (forget it)
  - **Hive** : data warehouse (repository) allowing SQL queries
  - **HBase** : NoSQL database that supports structured data storage for large tables
  - Phoenix : SQL layer to porovie low-latency access to HBase data
  - Solr : distributed search platform to index PB of data
  - **Spark** : general purpose processing engine to build and run SQL, streaming, machine learning, or graphic applications
- Integration
  - WebHDFS : REST API to manage HDFS through HTTP
  - Flume : collect, aggregate, move streaming data
  - Sqoop : import and export data between Hadoop and RDBMS
  - Atlas : governance services to meet compliance and data integration requirements
- Streaming
  - **NiFi** : directed graphs of data routing, transformation and system mediation logic
  - **Kafka** : publish/subscribe messaging system
  - Flink : stateful computation over unbounded and bounded data streams

