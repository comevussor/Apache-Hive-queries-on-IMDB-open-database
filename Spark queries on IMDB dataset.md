# Spark queries on IMDB dataset

Add a column to a table : `.withColumn("col_name")`

Remember that user defined functions (UDF) are not distributed

## Load files

```
name_basics = spark.read.load("imdb/name.basics.tsv", format="csv", sep="\t", inferSchema = "true", header = "true")
name_basics.show(20)

+---------+-------------------+---------+---------+--------------------+--------------------+
|   nconst|        primaryName|birthYear|deathYear|   primaryProfession|      knownForTitles|
+---------+-------------------+---------+---------+--------------------+--------------------+
|nm0000001|       Fred Astaire|     1899|     1987|soundtrack,actor,...|tt0050419,tt00531...|
|nm0000002|      Lauren Bacall|     1924|     2014|  actress,soundtrack|tt0038355,tt00718...|
|nm0000003|    Brigitte Bardot|     1934|       \N|actress,soundtrac...|tt0049189,tt00599...|
|nm0000004|       John Belushi|     1949|     1982|actor,writer,soun...|tt0077975,tt00725...|
|nm0000005|     Ingmar Bergman|     1918|     2007|writer,director,a...|tt0050986,tt00509...|
|nm0000006|     Ingrid Bergman|     1915|     1982|actress,soundtrac...|tt0036855,tt00381...|
|nm0000007|    Humphrey Bogart|     1899|     1957|actor,soundtrack,...|tt0037382,tt00432...|
|nm0000008|      Marlon Brando|     1924|     2004|actor,soundtrack,...|tt0078788,tt00472...|
|nm0000009|     Richard Burton|     1925|     1984|actor,producer,so...|tt0087803,tt00597...|
|nm0000010|       James Cagney|     1899|     1986|actor,soundtrack,...|tt0029870,tt00355...|
|nm0000011|        Gary Cooper|     1901|     1961|actor,soundtrack,...|tt0035211,tt00279...|
|nm0000012|        Bette Davis|     1908|     1989|actress,soundtrac...|tt0031210,tt00421...|
|nm0000013|          Doris Day|     1922|       \N|soundtrack,actres...|tt0060463,tt00483...|
|nm0000014|Olivia de Havilland|     1916|       \N|  actress,soundtrack|tt0031381,tt00408...|
|nm0000015|         James Dean|     1931|     1955| actor,miscellaneous|tt0049261,tt00485...|
|nm0000016|    Georges Delerue|     1925|     1992|composer,soundtra...|tt0069946,tt00963...|
|nm0000017|   Marlene Dietrich|     1901|     1992|soundtrack,actres...|tt0040367,tt00550...|
|nm0000018|       Kirk Douglas|     1916|       \N|actor,producer,so...|tt0049456,tt00523...|
|nm0000019|   Federico Fellini|     1920|     1993|writer,director,a...|tt0053779,tt00475...|
|nm0000020|        Henry Fonda|     1905|     1982|actor,producer,so...|tt0032551,tt00500...|
+---------+-------------------+---------+---------+--------------------+--------------------+
only showing top 20 rows

title_basics = spark.read.load("imdb/title.basics.tsv", format="csv", sep="\t", inferSchema = "true", header = "true")
title_basics.show(20)

+---------+---------+--------------------+--------------------+-------+---------+-------+--------------+--------------------+
|   tconst|titleType|        primaryTitle|       originalTitle|isAdult|startYear|endYear|runtimeMinutes|              genres|
+---------+---------+--------------------+--------------------+-------+---------+-------+--------------+--------------------+
|tt0000001|    short|          Carmencita|          Carmencita|      0|     1894|     \N|             1|   Documentary,Short|
|tt0000002|    short|Le clown et ses c...|Le clown et ses c...|      0|     1892|     \N|             5|     Animation,Short|
|tt0000003|    short|      Pauvre Pierrot|      Pauvre Pierrot|      0|     1892|     \N|             4|Animation,Comedy,...|
|tt0000004|    short|         Un bon bock|         Un bon bock|      0|     1892|     \N|            \N|     Animation,Short|
|tt0000005|    short|    Blacksmith Scene|    Blacksmith Scene|      0|     1893|     \N|             1|        Comedy,Short|
|tt0000006|    short|   Chinese Opium Den|   Chinese Opium Den|      0|     1894|     \N|             1|               Short|
|tt0000007|    short|Corbett and Court...|Corbett and Court...|      0|     1894|     \N|             1|         Short,Sport|
|tt0000008|    short|Edison Kinetoscop...|Edison Kinetoscop...|      0|     1894|     \N|             1|   Documentary,Short|
|tt0000009|    movie|          Miss Jerry|          Miss Jerry|      0|     1894|     \N|            45|             Romance|
|tt0000010|    short| Exiting the Factory|La sortie de l'us...|      0|     1895|     \N|             1|   Documentary,Short|
|tt0000011|    short|Akrobatisches Pot...|Akrobatisches Pot...|      0|     1895|     \N|             1|   Documentary,Short|
|tt0000012|    short|The Arrival of a ...|L'arrivée d'un tr...|      0|     1896|     \N|             1|   Documentary,Short|
|tt0000013|    short|The Photographica...|Neuville-sur-Saôn...|      0|     1895|     \N|             1|   Documentary,Short|
|tt0000014|    short|The Sprinkler Spr...|   L'arroseur arrosé|      0|     1895|     \N|             1|        Comedy,Short|
|tt0000015|    short| Autour d'une cabine| Autour d'une cabine|      0|     1894|     \N|             2|     Animation,Short|
|tt0000016|    short|Barque sortant du...|Barque sortant du...|      0|     1895|     \N|             1|   Documentary,Short|
|tt0000017|    short|Italienischer Bau...|Italienischer Bau...|      0|     1895|     \N|             1|   Documentary,Short|
|tt0000018|    short|Das boxende Känguruh|Das boxende Känguruh|      0|     1895|     \N|             1|               Short|
|tt0000019|    short|    The Clown Barber|    The Clown Barber|      0|     1898|     \N|            \N|        Comedy,Short|
|tt0000020|    short|      The Derby 1895|      The Derby 1895|      0|     1895|     \N|             1|Documentary,Short...|
+---------+---------+--------------------+--------------------+-------+---------+-------+--------------+--------------------+
only showing top 20 rows

title_ratings = spark.read.load("imdb/title.ratings.tsv", format="csv", sep="\t", inferSchema = "true", header = "true")
title_ratings.show(20)

+---------+-------------+--------+
|   tconst|averageRating|numVotes|
+---------+-------------+--------+
|tt0000001|          5.8|    1472|
|tt0000002|          6.4|     177|
|tt0000003|          6.6|    1100|
|tt0000004|          6.5|     106|
|tt0000005|          6.2|    1804|
|tt0000006|          5.6|      95|
|tt0000007|          5.5|     591|
|tt0000008|          5.6|    1580|
|tt0000009|          5.6|      76|
|tt0000010|          6.9|    5246|
|tt0000011|          5.4|     219|
|tt0000012|          7.4|    8856|
|tt0000013|          5.8|    1356|
|tt0000014|          7.2|    3865|
|tt0000015|          6.2|     686|
|tt0000016|          5.9|    1006|
|tt0000017|          4.8|     201|
|tt0000018|          5.5|     421|
|tt0000019|          6.6|      13|
|tt0000020|          5.1|     233|
+---------+-------------+--------+
only showing top 20 rows

title_crew = spark.read.load("imdb/title.crew.tsv", format="csv", sep="\t", inferSchema = "true", header = "true")
title_crew.show(20)

+---------+-------------------+---------+
|   tconst|          directors|  writers|
+---------+-------------------+---------+
|tt0000001|          nm0005690|       \N|
|tt0000002|          nm0721526|       \N|
|tt0000003|          nm0721526|       \N|
|tt0000004|          nm0721526|       \N|
|tt0000005|          nm0005690|       \N|
|tt0000006|          nm0005690|       \N|
|tt0000007|nm0005690,nm0374658|       \N|
|tt0000008|          nm0005690|       \N|
|tt0000009|          nm0085156|nm0085156|
|tt0000010|          nm0525910|       \N|
|tt0000011|          nm0804434|       \N|
|tt0000012|nm0525908,nm0525910|       \N|
|tt0000013|          nm0525910|       \N|
|tt0000014|          nm0525910|       \N|
|tt0000015|          nm0721526|       \N|
|tt0000016|          nm0525910|       \N|
|tt0000017|nm1587194,nm0804434|       \N|
|tt0000018|          nm0804434|       \N|
|tt0000019|          nm0932055|       \N|
|tt0000020|          nm0010291|       \N|
+---------+-------------------+---------+
only showing top 20 rows
```

## Number of titles with duration superior than 2 hours

```
from pyspark.sql import functions as F

long_titles = title_basics.filter(title_basics.runtimeMinutes>120).count()
long_titles

60446
```

## Average runtime (minutes) of titles containing the string “world”

```
world_titles = title_basics.filter(title_basics.primaryTitle.contains('world')) \
                            .agg(F.avg(F.col("runtimeMinutes")))
                           
world_titles.show(20)

+-------------------+
|avg(runtimeMinutes)|
+-------------------+
|  43.58105263157895|
+-------------------+
```

## Average rating of titles being comedies

```
title_basics.filter(F.col('genres').contains('Comedy')) \
    .select(F.col('tconst')) \
    .join(title_ratings, 'tconst') \
    .agg(F.avg(F.col('averageRating'))) \
.show(20)

+------------------+
|avg(averageRating)|
+------------------+
| 6.970428788330588|
+------------------+
```

## Average rating of titles not being comedies

```
title_basics.filter(~(F.col('genres').contains('Comedy'))) \
    .select(F.col('tconst')) \
    .join(title_ratings, 'tconst') \
    .agg(F.avg(F.col('averageRating'))) \
.show(20)

+------------------+
|avg(averageRating)|
+------------------+
| 6.886042545766083|
+------------------+
```

## Top 5 movies directed by Tarantino

```
top_five_bis = title_crew.withColumn('nconst', F.explode(F.split(title_crew.directors, ','))) \
                            .join(name_basics.filter(name_basics.primaryName == 'Quentin Tarantino'), 'nconst', 'left_semi')  \
                            .join(title_basics.filter(title_basics.titleType == 'movie'), 'tconst') \
                            .join(title_ratings, 'tconst') \
                            .orderBy(F.desc('averageRating')) \
                            .select(F.col('primaryTitle'), F.col('averageRating'))

top_five_bis.show(5)

+--------------------+-------------+
|        primaryTitle|averageRating|
+--------------------+-------------+
|        Pulp Fiction|          8.9|
|Kill Bill: The Wh...|          8.8|
|    Django Unchained|          8.4|
|Inglourious Basterds|          8.3|
|      Reservoir Dogs|          8.3|
+--------------------+-------------+
only showing top 5 rows
```



