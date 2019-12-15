# Hadoop ecosystem

Big Data is characterized by :
- Variety : structured, semi-structured, unstructured data => need to link, mach, cleanse, transform
- Volume : tera 10^12, peta 10^15, exa 10^18 => storage cost, question of data relevance
- Velocity : need for fast ingestion, fast analysis, address periodic peaks

Processing big data requires to distribute data accross multiple machines in **Data Centers**.

A **cluster** is a set of interconnected servers in a data center.

Distribution of data and processing implies :
- a specific file system : HDFS
- specific databases : NoSQL, HBase, ElasticSearch
- map and reduce parallelized algorithms
