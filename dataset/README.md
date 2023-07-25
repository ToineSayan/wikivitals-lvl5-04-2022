# Dataset

## Main file: wikivitals-lvl-5-04-2022.json

Each line in this file corresponds to a single item in the wikivitals-lvl-5-04-2022 dataset, in the form of a dictionary whose structure is described below.


* **id** (str): wikipedia id of the article. 
* **abstract** (str): preprocessed text of the abstract (can be empty, some articles have no abstract)
* **label** (list of str): list of 3 labels containing, in this order, the level 0 label ("coarse granularity" label), the level 1 label ("intermediate granularity" label) and finally the level 2 label ("fine granularity" label).
* **title** (str): article title (note: each title in the dataset is unique)
* **headers** (list of str): article section and sub-section titles
* **outcoming_links_filtered** (list of str): list ids of articles in the wikivitals-lvl-5-04-2022 corpus for which there is an outgoing link in the body of the article pointing to these articles


Below is an example of an article in the corpus:

```python
{
    "id": "10152440", # current version of the article: https://en.wikipedia.org/?curid=10152440
    "abstract": [
        "Dallasaurus Dallas lizard is a basal mosasauroid from the Upper Cretaceous of North America. Along with Russellosaurus Dallasaurus is one of the two oldest mosasauroid taxa currently known from North America. This small semi aquatic lizard measured less than a meter in length  compared to such gigantic derived mosasaurs as Tylosaurus and Mosasaurus each exceeding 14 meters."
    ], 
    "label": [
        "Biological and health sciences", # level 0 label: "coarse granularity" label
        "Animals", # level 1 label: "intermediate granularity" label
        "Reptiles" # level 2 label: "fine granularity" label
    ], 
    "title": "Dallasaurus", 
    "headers": [
        "Specimens", 
        "Anatomy", 
        "Classification", 
        "References", 
        "Sources"
    ], 
    "outcoming_links_filtered": [
        "800373", # Mosasaur
        "32031", # University of Texas at Austin
        "210294", # Ulna
        "554469", # Stratum
        "331755", # Transitional fossil
        "2051142", # Mosasaurus
        "2469649", # Tylosaurus
        "38493", # Genus
        "44211", # Shale
        "227807", # Humerus
        "166945", # Cartilage
        "21139", # North America
        "29810", # Texas
        "585732" # Radius (bone)
    ]
}
```

## Additional file: wikivitals-lvl-5-04-2022-binary-representations.json

Each line in this file corresponds to a single item in the wikivitals-lvl-5-04-2022 dataset, in the form of a dictionary whose structure is described below.

* **id** (str): wikipedia id of the article. 
* **binary_features** (list of str): list of features present at least once in the abstract, headers or title of the article (can be used to build binarized representations of articles)

There are a total of 4000 features (unigrams and bigrams) extracted from the articles in the corpus.
Feature names have the following format: 
* for unigrams: *{prefix}*_*{unigram_stem}*
* for bigrams: *{prefix}*_*{bigram_stem_1}*_*{bigram_stem_1}*

*{prefix}* indicates the origin of the n-gram: *abs* for abstract, *hea* for section and sub-section titles, *tit* for title

Below is an example of an article in the corpus:

```python
{
    "id": "10152440", # current version of the article: https://en.wikipedia.org/?curid=10152440
    "binary_features": [
        "abs_america", 
        "abs_aquat", 
        "abs_basal", 
        "abs_cretac", 
        "abs_current", 
        "abs_deriv", 
        "abs_known", 
        "abs_length",
        "abs_lizard", 
        "abs_measur", 
        "abs_meter", 
        "abs_north", 
        "abs_north_america", # a bigram from the abstract
        "abs_oldest", 
        "abs_small", 
        "abs_two", 
        "hea_classif", # a unigram from a section or sub-section title
        "hea_sourc"
    ]
}
```