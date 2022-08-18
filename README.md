# wikivitals-lvl5-dataset
A dataset built from Wikivitals articles


## Description of the dataset

This dataset is composed of one-hot representations of Wikipedia's vital level 5 articles (retrieved from an archive dated April 12, 2022). 
A graph structure was added by collecting HTML links between the vital articles present in the body of the articles. Vital articles are the nodes of the graph and HTML links define directed links between these nodes. 
Vital articles have been grouped into sections by different Wikipedia contributors; these sections and their subsections have been used to assign to each article 3 levels of classification, the highest level defining a coarse classification and the lowest a fine classification of vital articles.

## General values

Number of nodes: 48512
Number of directed edges: 2297782
Number of edges in the undirected graph: 4132534

Number of features: 4000
Number of labels:
- level 0 (coarse classification): 11
- level 1 (intermediary classification): 32
- level 2 (fine classification): 230

Edge homophily (= number of edges connecting nodes of the same class / total number of edges):
- for class level 0: 0.34
- for class level 1: 0.24
- for class level 2: 0.15

Average outgoing degree: 85.19 (141.23)
Max & min degree: 7720 & 0
Number of isolated nodes: 16

## How to reconstruct

**Step 1:** Download the content of this repository

**Step 2:** run "python build.py"
Five files will be created in the root of this repository:
- wikivitals_features_one_hot.txt
- links_filtered.txt
- label_0.txt
- label_1.txt
- label_2.txt
A description of the contents of these files is given in the following sections.

**(optional) Step 3:** run "python build_npz.py"
An .npz file (wikivitals.npz) will be created at the root of the repository
 
## Features

The features used to represent the articles come from one-hot representations of the article abstracts (first paragraph before the first section title in the body of the article), the article titles and the headers present in the body of the article (section and subsection titles).

Some useful informations:
- The stemmer of nltk was used 
- The most frequent unigrams and bigrams (frequency > 0.001 for abstracts and headers, frequency > 0.0001 for titles) have been extracted (note: frequency means we only keep unigrams or bigrams that are present in at least frequency*total_number_of_articles_in_the_corpus articles in the corpus)
- English stopwords (from nltk.corpus) have been ignored
- The 4000 features most predictive of level 2 labels (fine classification) in the chi2 sense were retained to reduce the size of the representations.

In wikivitals_features_one_hot.txt files, lines have the following format:

```
3449450	abs_group:1 abs_includ:1 abs_island:1 abs_islet:1 abs_provinc:1 hea_histori:1 tit_island:1
```

which means that the article whose wikiid is 3449450 is represented by 7 features out of 4000, 5 of them come from the abstract (the ones starting with 'abs'), 1 comes from the headers (the one starting with 'hea') and 1 comes from the title (the one starting with 'tit'). 'group', 'includ', 'island', 'islet', 'provinc', 'histori', 'island' are the stems corresponding to these features.
WARNING: the wikiid is followed by a trailing character ('\t') and features are separated by spaces. 


## Edges

The directed edges correspond to HTML links between vital articles in the namespace 0 (lien). 
The links were collected in the body of the articles and duplicates were removed.

In links_filtered.txt, lines have the following format:

```
100005	674	1.0
```

which means that there is a link between the article whose wikiid is '100005' and the article whose wikiid is '674' (the weight assigned to the link is always equal to 1.0). 
WARNING: the three values on each line are separated by trailing characters ('\t').

[[test|'./assets/edges_loglogdegrees.png']]


## Labels

Vital articles have been grouped into sections by different Wikipedia contributors; these sections and their subsections have been used to assign to each article 3 levels of classification, the highest level defining a coarse classification and the lowest a fine classification of vital articles.

See file 'class_hierarchy.txt' for class hierarchy and number of articles per class.





