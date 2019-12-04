# How to load text files, re-arrange data, produce stats with pyspark and RDD datasets

I used Zeppelin notebook

Files are in an hdfs folder named city_revenue (see download in this repository)

Our files are named city.txt or city_storeNumber.txt .

Content is a list of pairs month, amount. See it in Linux : 

```
hdfs dfs -cat city_revenue/anger.txt
JAN 13
FEB 12
MAR 14
APR 15
MAY 12
JUN 15
JUL 19
AUG 15
SEP 13
OCT 8
NOV 14
DEC 16
```

Now, in Zeppelin : 

Load files in a new RDD structure

`rdd_raw = sc.wholeTextFiles('city_revenue')`

rdd_raw is a list of key/value pairs :

`[(<first_file_path>, <first_line>/r/n<secondline>....),
(<second_file_path>, <first_line>/r/n<secondline>....), 
(...,...)]`

We clean the file path to keep only the file name and split the lines into a list. (convert into string to clean the data)

`rdd_noUrl = rdd_raw.map( lambda line: (  str(line[0].split('/')[-1].split('.')[0]) , str(line[1]).split('\r\n') ) )`

Results is still a list of key/value pairs wher key is a string and value a list :

```
[('anger', ['JAN 13', 'FEB 12', 'MAR 14', ...]),
('marseilles_1', ['JAN 21', 'FEB 21', 'MAR 21', ...],
(...,..)]
```

We now convert each key/value pair into a list of key/value pairs with the same key. At the same time, we split the store number when there is one and separate month from value

`rdd_expand = rdd_noUrl.flatMap(lambda (store, months) : map( lambda x : (store.split('_'), x.split(' '))  , months )   )`

Result is still a list of key/value pairs where both key and value are lists :
```
[(['anger'], ['JAN', '13']), (['anger'], ['FEB', '12']), (['anger'], ['MAR', '14']), (['anger'], ...),
(['marseilles', '1'], ['JAN', '21']), (['marseilles', '1'], ['FEB', '21']), (['marseilles', '1'], ...),
(['marseilles', '2'], ['JAN', '11']), (['marseilles', '2'], ['FEB', '11']), (['marseilles', '2'], ...),
(...,...)]
```

We now get our final RDD dataset, rearranging "columns" :

```
rdd_store = rdd_expand.map( lambda row :
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
```                            

Result is :

```[('anger', 1, 'JAN', 13), ('anger', 1, 'FEB', 12), ('anger', 1, 'MAR', 14), ('anger', 1, 'APR', 15), (...,...,...,...),
('marseilles', 1, 'JAN', 21), ('marseilles', 1, 'FEB', 21), ('marseilles', 1, 'MAR', 21), (...,...,...,...),
('marseilles', 2, 'JAN', 11), ('marseilles', 2, 'FEB', 11), ('marseilles', 2, 'MAR', 11), (...,...,...,...),
(...,...,...,...)]
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
