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










## Compute some averages

```
tot_france = rdd_store.map( lambda row : (row[3], 1) ).reduce(lambda a,b : (a[0]+b[0], a[1]+b[1]) )
print tot_france

avg_france = tot_france[0] / tot_france[1]
print "Overall monthly average by shop is ", avg_france

avg_month = tot_france[0] / 12
print "Overall monthly average for the brand is ", avg_month
```

Yields :

```
(3619, 156)
Overall monthly average by shop is  23
Overall monthly average for the brand is  301
```

## Total revenue per city

```
tot_city = rdd_store.map( lambda row : (row[0], row[3]) )
tot_city.collect()

tot_city2 = tot_city.reduceByKey(lambda a,b : a+b)
tot_city2.collect()
```

yields :

```
[('troyes', 214), ('paris', 1568), ('lyon', 193), ('toulouse', 177), ('anger', 166), ('orlean', 196), ('rennes', 180), ('nice', 203), ('marseilles', 515), ('nantes', 207)]
```

## Best shop each month

```
max_shop = rdd_store.map( lambda row : (row[2], (row[0] + str(row[1]), row[3]) ) )
max_shop2 = max_shop.reduceByKey(lambda a,b : a if (a[1] > b[1]) else b)
max_shop2.collect()
```

yields :

```
[('FEB', ('paris2', 42)), ('AUG', ('paris2', 45)), ('APR', ('paris1', 57)), ('JUN', ('paris2', 85)), ('JUL', ('paris1', 61)), ('JAN', ('paris1', 51)), ('MAY', ('paris2', 72)), ('NOV', ('paris2', 64)), ('MAR', ('paris2', 44)), ('DEC', ('paris1', 71)), ('OCT', ('paris1', 68)), ('SEP', ('paris2', 63))]
```
