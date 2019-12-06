# Stats using dataframes in pyspark

I used Zeppelin notebook

For formatting data see https://github.com/comevussor/Hadoop-and-Spark-ecosystems/blob/master/spark%20city%20revenue%20RDD.md

```
rdd_raw = sc.wholeTextFiles('city_revenue')
rdd_store = rdd_raw.map( lambda line : (
                                          str(line[0].split('/')[-1].split('.')[0]) ,
                                          str(line[1]).split('\r\n')
                                        )
                        ) \
                   .flatMap(lambda (store, months) : 
                          map (
                                  lambda x : ( store.split('_'), x.split(' ') ), 
                                  months ) 
                              ) \
                   .map( lambda row :
                                (row[0][0],
                                1,
                                row[1][0],
                                int(row[1][1]))
                                if len(row[0])==1 else
                                (row[0][0],
                                int(row[0][1]),
                                row[1][0],
                                int(row[1][1]))
                            )
rdd_store.take(100)
```

## Load data in dataframe

```
df = spark.createDataFrame(rdd_store, ('City', 'Store', 'Month', 'Income'))
df.show(20)

+----------+-----+-----+------+
|      City|Store|Month|Income|
+----------+-----+-----+------+
|     anger|    1|  JAN|    13|
|     anger|    1|  FEB|    12|
|     anger|    1|  MAR|    14|
|     anger|    1|  APR|    15|
|     anger|    1|  MAY|    12|
|     anger|    1|  JUN|    15|
|     anger|    1|  JUL|    19|
|     anger|    1|  AUG|    15|
|     anger|    1|  SEP|    13|
|     anger|    1|  OCT|     8|
|     anger|    1|  NOV|    14|
|     anger|    1|  DEC|    16|
|marseilles|    1|  JAN|    21|
|marseilles|    1|  FEB|    21|
|marseilles|    1|  MAR|    21|
|marseilles|    1|  APR|    27|
|marseilles|    1|  MAY|    25|
|marseilles|    1|  JUN|    25|
|marseilles|    1|  JUL|    21|
|marseilles|    1|  AUG|    22|
+----------+-----+-----+------+
only showing top 20 rows
```

## Select a column

- `df.Income` : shorter and unambiguous
- `F.col("Income")` : useful when chaining (don't know the name of the intermediate RDD) but risk of ambiguity

```
from pyspark.sql import functions as F
df_income = df.select(F.col("Income").alias("x_Income"))
df_income.show()

+--------+
|x_Income|
+--------+
|      13|
|      12|
|      14|
|      15|
|      12|
|      15|
|      19|
|      15|
|      13|
|       8|
|      14|
|      16|
|      21|
|      21|
|      21|
|      27|
|      25|
|      25|
|      21|
|      22|
+--------+
only showing top 20 rows
```

## Monthly average

```
df_income_avg = df.agg(F.sum(F.col("Income")/12).alias ("avg_income"))
df_income_avg.show()

+-----------------+
|       avg_income|
+-----------------+
|301.5833333333334|
+-----------------+
```

## Total per city

Aggregation will select only `groupBy()` and `agg()` column(s) (pre-1999 SQL type). To get more columns, a joint is necessary (see below).

```
df_city = df.groupBy(F.col("City")) \
            .agg(F.sum(F.col("Income")).alias("sum_income")) \
            .orderBy(F.col("City"))
            
df_city.show()

+----------+----------+
|      City|sum_income|
+----------+----------+
|     anger|       166|
|      lyon|       193|
|marseilles|       515|
|    nantes|       207|
|      nice|       203|
|    orlean|       196|
|     paris|      1568|
|    rennes|       180|
|  toulouse|       177|
|    troyes|       214|
+----------+----------+
```

## Average monthly income of the shop in each city

Numerical transformation of a column.

```
df_avg_city = df_city.select(F.col("City"), (F.col("sum_income")/12).alias("avg_city"))
df_avg_city.show()

+----------+------------------+
|      City|          avg_city|
+----------+------------------+
|     anger|13.833333333333334|
|      lyon|16.083333333333332|
|marseilles|42.916666666666664|
|    nantes|             17.25|
|      nice|16.916666666666668|
|    orlean|16.333333333333332|
|     paris|130.66666666666666|
|    rennes|              15.0|
|  toulouse|             14.75|
|    troyes|17.833333333333332|
+----------+------------------+
```

## Total revenue per store per year

With `concat()`

```
from pyspark.sql.functions import concat

df_total_store = df.select(concat(df.City, df.Store).alias("city_store"), df.Income) \
            .groupBy(F.col("city_store")) \
            .agg(F.sum(F.col("Income")).alias("sum_income")) \
            .orderBy(F.col("sum_income"))
            
df_total_store.show()

+-----------+----------+
| city_store|sum_income|
+-----------+----------+
|     anger1|       166|
|  toulouse1|       177|
|    rennes1|       180|
|      lyon1|       193|
|    orlean1|       196|
|      nice1|       203|
|    nantes1|       207|
|    troyes1|       214|
|marseilles2|       231|
|marseilles1|       284|
|     paris3|       330|
|     paris1|       596|
|     paris2|       642|
+-----------+----------+
```

OR without `concat()` but multiple `groupBy()` arguments

```
df_total_store_2 = df.select(df.City, df.Store, df.Income) \
            .groupBy(F.col("City"), F.col("Store")) \
            .agg(F.sum(F.col("Income")).alias("sum_income")) \
            .orderBy(F.col("sum_income"))

df_total_store_2.show()

+----------+-----+----------+
|      City|Store|sum_income|
+----------+-----+----------+
|     anger|    1|       166|
|  toulouse|    1|       177|
|    rennes|    1|       180|
|      lyon|    1|       193|
|    orlean|    1|       196|
|      nice|    1|       203|
|    nantes|    1|       207|
|    troyes|    1|       214|
|marseilles|    2|       231|
|marseilles|    1|       284|
|     paris|    3|       330|
|     paris|    1|       596|
|     paris|    2|       642|
+----------+-----+----------+
```

## Best store of the month

Create an intermediate table.

```
df_best_store = df.select(df.Month.alias("Month"), df.City.alias("City"), df.Store, df.Income.alias("Income")) \
                    .groupBy(F.col("Month")) \
                    .agg(F.max(F.col("Income")).alias("Income"))
df_best_store.show()                

+-----+------+
|Month|Income|
+-----+------+
|  JAN|    51|
|  FEB|    42|
|  MAR|    44|
|  APR|    57|
|  MAY|    72|
|  JUN|    85|
|  JUL|    61|
|  AUG|    45|
|  SEP|    63|
|  OCT|    68|
|  NOV|    64|
|  DEC|    71|
+-----+------+
```

Join original table and intermediate one. A `select()` is required to avoid duplicated columns.

Default join is inner.

See also the month formatting.

```
df_best_store_2 = df.join(df_best_store, ["Income", "Month"]) \
                            .select(F.col("Month"), F.col("City"), F.col("Store"), F.col("Income")) \
                            .orderBy(F.unix_timestamp(F.col("Month"), "MMM"))
                    
df_best_store_2.show()

+-----+-----+-----+------+
|Month| City|Store|Income|
+-----+-----+-----+------+
|  JAN|paris|    1|    51|
|  FEB|paris|    2|    42|
|  MAR|paris|    2|    44|
|  APR|paris|    1|    57|
|  MAY|paris|    2|    72|
|  JUN|paris|    2|    85|
|  JUL|paris|    1|    61|
|  AUG|paris|    2|    45|
|  SEP|paris|    2|    63|
|  OCT|paris|    1|    68|
|  NOV|paris|    2|    64|
|  DEC|paris|    1|    71|
+-----+-----+-----+------+
```

OR right join

```
df_best_store_2 = df.join(df_best_store, ["Income", "Month"], "right") \
                            .orderBy(F.unix_timestamp(F.col("Month"), "MMM"))
df_best_store_2.show()

+------+-----+-----+-----+
|Income|Month| City|Store|
+------+-----+-----+-----+
|    51|  JAN|paris|    1|
|    42|  FEB|paris|    2|
|    44|  MAR|paris|    2|
|    57|  APR|paris|    1|
|    72|  MAY|paris|    2|
|    85|  JUN|paris|    2|
|    61|  JUL|paris|    1|
|    45|  AUG|paris|    2|
|    63|  SEP|paris|    2|
|    68|  OCT|paris|    1|
|    64|  NOV|paris|    2|
|    71|  DEC|paris|    1|
+------+-----+-----+-----+
```
