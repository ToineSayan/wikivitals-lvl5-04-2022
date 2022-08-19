## Files 

- \_\_wikivitals_features_one_hot_*X*.txt: one-hot representations of the nodes (= vital articles)
- \_\_wikivitals_vocabulary.txt: list of the 4000 feature names
- \_\_links_filtered_*X*.txt: list of the  directed edges (= HTML links between 2 vital articles)
- \_\_classes*i*.txt: labels of level *i* of each node

## \_\_wikivitals_vocabulary.txt - Features

Each feature starts with a 3 letter prefix that indicates the origin of the stem:
- 'abs' for abstracts (.../4000 features (xx%))
- 'tit' for titles (.../4000 features (xx%))
- 'hea' for headers in the articles (.../4000 features (xx%))

For example:
- 'abs_group' is a feature from the stemming of abstracts (prefix 'abs') and corresponds to the unigram whose stem is 'group'.
- 'hea_personal_lif' is a feature from the stemming of headers (prefix 'hea') and corresponds to the bigram whose stems are ('personal','lif').

There are 4000 features.


## \_\_wikivitals_features_one_hot_*X*.txt files - Node representations

Each line of these files lists the features present (and only those) in the article designated by its wikiid.

A typical line of these files is reproduced below. It is composed of a wikiid followed by a trailing character ('\t') and a list of features concatenated to a document presence indicator (':1') and separated by spaces. 

```
3449450	abs_group:1 abs_includ:1 abs_island:1 abs_islet:1 abs_provinc:1 hea_histori:1 tit_island:1
```

## \_\_links_filtered_*X*.txt - Directed edges

A typical line of these files is reproduced below. It is composed of 2 wikiid and a weight (always 1.0) separated by a trailing character ('\t'). 

```
100005	674	1.0
```

## \_\_classes*i*.txt - Node labels

A typical line of these files is reproduced below. It is composed of a wikiid and a class name separated by a trailing character ('\t'). 

```
171038	People ->- Artists, musicians, and composers
```
