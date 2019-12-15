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





