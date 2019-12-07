## Word count with Spark

I have previously downloaded Frankenstein book in a `raw`folder.

Locate the source to create an rdd :

`rdd_book = sc.textFile('raw/frankenstein.txt')`

Book is not loaded yet till action :

`rdd_book.take(5)`

Now it's loaded. Result is :

`[u'', u"Project Gutenberg's Frankenstein, by Mary Wollstonecraft (Godwin) Shelley", u'', u'This eBook is for the use of anyone anywhere at no cost and with', u'almost no restrictions whatsoever.  You may copy it, give it away or']`

Flatten the array of arrays into a single flat rdd structure : 

`rdd_words = rdd_book.flatMap(lambda line: line.split())`

`rdd_words.take(20)` 

yields :

`[u'Project', u"Gutenberg's", u'Frankenstein,', u'by', u'Mary', u'Wollstonecraft', u'(Godwin)', u'Shelley', u'This', u'eBook', u'is', u'for', u'the', u'use', u'of', u'anyone', u'anywhere', u'at', u'no', u'cost']`

Build a kew-value structure : 

`rdd_one = rdd_words.map(lambda word: (word, 1))`

`rdd_one.take(20)` 

yields :

`[(u'Project', 1), (u"Gutenberg's", 1), (u'Frankenstein,', 1), (u'by', 1), (u'Mary', 1), (u'Wollstonecraft', 1), (u'(Godwin)', 1), (u'Shelley', 1), (u'This', 1), (u'eBook', 1), (u'is', 1), (u'for', 1), (u'the', 1), (u'use', 1), (u'of', 1), (u'anyone', 1), (u'anywhere', 1), (u'at', 1), (u'no', 1), (u'cost', 1)]`

Pile up same words and count them : 

`rdd_count = rdd_one.reduceByKey(lambda v1,v2: v1+v2)`

`rdd_count.take(20)`

yields :

`[(u'Reuss.', 1), (u'yellow', 4), (u'four', 4), (u'ocean,', 2), (u'ocean.', 2), (u'kennel,', 1), (u'looking', 9), (u'electricity', 1), (u'recollection,', 1), (u'recollection.', 4), (u'sinking', 2), (u'wood,', 8), (u'wood.', 7), (u'pretended', 1), (u'hardships.', 2), (u'fingers,', 1), (u'bringing', 1), (u'disturb', 6), (u'uttering', 1), (u'unfulfilled.', 2)]`

Finally sort results. We must switch key and value to be able to `sortByKey`.

We can pipe functions but rdd is immutable therefore it will still create intermediate rdd :

```
rdd_sorted = rdd_count.map(lambda v: (v[1], v[0])) \
                      .sortByKey(False) \
        
rdd_sorted.take(150)
```

yields : 

`[(4056, u'the'), (2971, u'and'), (2741, u'of'), (2719, u'I'), (2142, u'to'), (1631, u'my'), (1394, u'a'), (1125, u'in'), (993, u'was'), (987, u'that'), (700, u'with'), (679, u'had'), (547, u'which'), (542, u'but'), (529, u'me'), (500, u'his'), (498, u'not'), (486, u'as'), (484, u'for'), (466, u'by'), (450, u'you'), (448, u'he'), (435, u'on'), (387, u'from'), (372, u'it'), (360, u'have'), (357, u'be'), (335, u'this'), (322, u'is'), (313, u'her'), (304, u'at'), (298, u'were'), (267, u'The'), (262, u'when'), (246, u'your'), (239, u'or'), (212, u'an'), (197, u'so'), (191, u'will'), (190, u'all'), (188, u'could'), (183, u'are'), (182, u'been'), (177, u'would'), (175, u'one'), (174, u'their'), (172, u'she'), (168, u'they'), (157, u'if'), (154, u'should'), (154, u'no'), (153, u'who'), (150, u'more'), (148, u'me,'), (147, u'him'), (136, u'some'), (133, u'these'), (130, u'now'), (128, u'But'), (126, u'He'), (125, u'upon'), (124, u'into'), (124, u'its'), (123, u'before'), (122, u'we'), (122, u'our'), (121, u'only'), (120, u'My'), (115, u'any'), (114, u'am'), (112, u'did'), (111, u'than'), (109, u'yet'), (107, u'may'), (107, u'might'), (107, u'me.'), (105, u'myself'), (103, u'every'), (102, u'first'), (101, u'do'), (100, u'shall'), (98, u'can'), (96, u'own'), (93, u'towards'), (92, u'what'), (91, u'saw'), (89, u'most'), (88, u'those'), (88, u'It'), (82, u'even'), (82, u'other'), (81, u'then'), (81, u'whom'), (80, u'found'), (78, u'Project'), (77, u'where'), (77, u'such'), (76, u'them'), (76, u'man'), (75, u'time'), (73, u'being'), (73, u'very'), (73, u'father'), (72, u'felt'), (72, u'must'), (71, u'\u201cI'), (70, u'This'), (68, u'said'), (68, u'She'), (66, u'many'), (66, u'made'), (65, u'life'), (65, u'thought'), (65, u'up'), (64, u'You'), (63, u'eyes'), (63, u'has'), (63, u'still'), (63, u'again'), (62, u'dear'), (62, u'same'), (60, u'mind'), (60, u'never'), (60, u'became'), (60, u'few'), (59, u'passed'), (58, u'while'), (58, u'also'), (58, u'like'), (57, u'ever'), (57, u'feelings'), (56, u'We'), (56, u'heard'), (56, u'soon'), (55, u'over'), (55, u'how'), (55, u'night'), (55, u'after'), (54, u'human'), (54, u'day'), (53, u'little'), (53, u'about'), (53, u'work'), (53, u'miserable'), (53, u'Gutenberg-tm'), (52, u'me;'), (52, u'appeared'), (51, u'there'), (50, u'see'), (50, u'cannot')]`
