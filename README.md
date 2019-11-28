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
```
Create external table pointing to the file.
```
CREATE EXTERNAL TABLE IF NOT EXISTS ext_imdb_tablename(
field1 type1,
field2, type2,
...)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
location '/user/user_name/imdb_tablename'
tblproperties ("skip.header.line.count"="1");
```
Create new hive table structure, ingest and check
```
CREATE TABLE IF NOT EXISTS imdb_tablename(
field1 type1,
field2, type2,
...)
STORED AS ORC;

INSERT INTO imdb_tablename
        SELECT * FROM ext_imdb_tablename;

SELECT * FROM imdb_tablename LIMIT 50;
```

## IMDB Queries (note that results are purely indicative as dataset is constantly updated)

### 1) Number of titles with duration superior than 2 hours.
```
SELECT COUNT(*) 
FROM imdb_title_basics 
WHERE runtimeminutes > 120;
```
Result : 60446

### 2) Average runtime (minutes) of titles containing the string "world"
```
SELECT AVG(runtimeminutes) 
FROM imdb_title_basics
WHERE primarytitle LIKE "%world%";
```
Result : 43.58

### 3) Average rating of titles being comedies
`JOIN` is better than a cartesian product `FROM tableA, tableB` when dealing with big data to avoid possible explosion in size. Result is an array. `array_contains()` is a Hive buit-in user-defined function.
```
SELECT AVG(averagerating) 
FROM imdb_title_ratings
JOIN imdb_title_basics
ON imdb_title_ratings.tconst = imdb_title_basics.tconst
WHERE array_contains(imdb_title_basics.genres, "Comedy");
```
Result : 6.97

### 4) Average rating of titles not being comedies
```
SELECT AVG(averagerating) 
FROM imdb_title_ratings
JOIN imdb_title_basics 
ON imdb_title_ratings.tconst = imdb_title_basics.tconst
WHERE NOT array_contains(imdb_title_basics.genres, "Comedy");
```
Result : 6.88

### 5) Top 5 movies directed by Tarantino
To save space, certain fields in IMDB dataset are given as arrays :

> title.crew.tsv.gz – Contains the director and writer information for all the titles in IMDb. Fields include:

> tconst (string) - alphanumeric unique identifier of the title

> directors (array of nconsts) - director(s) of the given title

> writers (array of nconsts) – writer(s) of the given title

In that case, to be able to filter on directors, it is necessary first to explode the column in an additional column, we name it `L1.one_director`. It is possible to keep on using to table names (here `T3` and `L1`) or rename the whole thing (here `T1`). `explode()` is a buit-in user-defined function, result is a true table, not an array. 

```
SELECT T6.primarytitle, T5.averagerating
  FROM
    (SELECT T1.tconst
      FROM
        (select T3.tconst, L1.one_director
          FROM imdb_title_crew
        AS T3
          LATERAL VIEW explode(director) L1 AS one_director)
        AS T1
  JOIN imdb_name_basics AS T2
  ON T1.one_director = T2.nconst
  WHERE T2.primaryname = "Quentin Tarantino")
  AS T4
JOIN imdb_title_ratings as T5
ON T4.tconst = T5.tconst
JOIN imdb_title_basics as T6
ON T4.tconst = T6.tconst
WHERE T6.titletype = "movie"
ORDER BY T5.averagerating DESC
LIMIT 5
;
```
Result
```
+-------------------------------------+-------------------+
|           t6.primarytitle           | t5.averagerating  |
+-------------------------------------+-------------------+
| Pulp Fiction                        | 8.9               |
| Kill Bill: The Whole Bloody Affair  | 8.8               |
| Django Unchained                    | 8.4               |
| Reservoir Dogs                      | 8.3               |
| Inglourious Basterds                | 8.3               |
+-------------------------------------+-------------------+
```
