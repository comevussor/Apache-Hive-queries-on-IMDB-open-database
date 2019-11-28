# Apache-Hive-queries-on-IMDB-open-database
To see some specificities of Hive queries 

IMDB data is downloadable here : https://www.imdb.com/interfaces/

For these queries, I'm using Beeline client from an edge Linux node on an Hadoop cluster to connect with Hive.

## Import tables in Hive

Store data on HDFS (Linux)

```
hdfs dfs -mkdir imdb_tablename
wget my_URL/tablename.tsv.gz
gunzip tablename.tsv.gz
```
Rename file if necessary (to remove . for instance).
```
hdfs dfs -put <tablename.tsv> imdb_tablename
```

Make sure you are using the proper database and have proper privileges on it (Hive)

```
use <my_db>;
CREATE EXTERNAL TABLE IF NOT EXISTS ext_imdb_tablename(
field1 type,
field2, type,
...)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
location '/user/user_name/imdb_tablename'
tblproperties ("skip.header.line.count"="1");
```
