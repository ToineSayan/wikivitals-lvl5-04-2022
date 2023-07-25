# [dataset] wikivitals-lvl5-04-2022
A dataset built from vitals articles of Wikipedia - Level 5 in April 2022.

Used in [Fair Evaluation of Graph Markov Neural Networks](https://arxiv.org/abs/2304.01235) (under review)


## Description of the dataset

This dataset is composed of one-hot representations of [Wikipedia's vital level 5 articles](https://en.wikipedia.org/wiki/Wikipedia:Vital_articles/Level/5) (retrieved from [an archive dated April 01, 2022 (no more available from this website)](https://dumps.wikimedia.org/enwiki/)). <br/>
A graph structure was added by collecting HTML links between the vital articles present in the body of the articles. Vital articles are the nodes of the graph and HTML links define directed edges between these nodes. <br/>
Vital articles have been grouped into sections by different Wikipedia contributors; these sections and their subsections have been used to assign to each article 3 levels of classification, the highest level defining a coarse classification and the lowest a fine classification of vital articles.

## General values

| #Nodes | #Edges | #Features | #Labels | Edge homophily<sup>1</sup> | Outgoing degree |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 48,512 | *directed:* 2,297,782 <br/> *undirected:* 4,132,534 | 4000 | *lvl 0 (coarse):* 11 <br/> *lvl 1 (intermediary):* 32 <br/> *lvl 2 (fine):* 251 |  *lvl 0 (coarse):* 0.34 <br/> *lvl 1 (intermediary):* 0.24 <br/> *lvl 2 (fine):* 0.15 | *avg.:* 85.19 (141.23) <br/> *max:* 7,720 <br/> *min:* 0 (16 nodes) | 

<sup>1</sup> (Edge) homophily = number of edges connecting nodes of the same class / total number of edges


## How to use

```python
main_file = "./dataset/wikivitals-lvl5-04-2022.json"
additional_file = "./dataset/wikivitals-lvl5-04-2022-binary-representations.json"

import json

corpus = {}
with open(main_file, 'r', encoding='utf8') as f:
    for row in f:
        entry = json.loads(row)
        corpus[entry["id"]] = entry
    
with open(additional_file, 'r', encoding='utf8') as add_f:
    for row in add_f:
        entry = json.loads(row)
        corpus[entry["id"]]["binary_features"] = entry["binary_features"]
```

Below an entry of the corpus loaded:

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
    ],
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
 
## Features

### General information
The features used to represent the articles come from one-hot representations of the article abstracts (first paragraph before the first section title in the body of the article), the article titles and the headers present in the body of the article (section and subsection titles).

Some useful informations:
- The stemmer SnowballStemmer of nltk was used 
- The most frequent unigrams and bigrams (frequency > 0.001 for abstracts and headers, frequency > 0.0001 for titles) have been extracted (note: frequency means we only keep unigrams or bigrams that are present in at least frequency*total_number_of_articles_in_the_corpus articles in the corpus)
- English stopwords (from nltk.corpus) have been ignored
- The 4000 features most predictive of level 2 labels (fine classification) in the chi2 sense were retained to reduce the size of the representations.

```python
# Stemmer - Vectorizer

import nltk.stem
from sklearn.feature_extraction.text import CountVectorizer
from nltk.corpus import stopwords

english_stemmer = nltk.stem.SnowballStemmer('english', ignore_stopwords=True)
class StemmedCountVectorizer(CountVectorizer):
    def build_analyzer(self):
        analyzer = super(StemmedCountVectorizer, self).build_analyzer()
        return lambda doc: ([english_stemmer.stem(w) for w in analyzer(doc)])

vectorizer = StemmedCountVectorizer(stop_words=stopwords.words('english'), ngram_range=(1, 2), min_df=0.001)
```

## Edges

### List of edges

The directed edges correspond to HTML links between vital articles in the [namespace 0](https://en.wikipedia.org/wiki/Wikipedia:What_is_an_article%3F#Namespace). 

Links between articles were collected in the body of vital articles, filtered to keep only those that point to another vial article, and duplicates were removed. 

The list of edges has been saved in files prefixed with '\_\_links\_filtered\_' in the folder './dataset/main/'.

### Edges distribution

![Edges distribution](/assets/edges_loglogdegree.png)


## Labels

### Labels extraction

Vital articles have been grouped into sections by different Wikipedia contributors; these sections and their subsections have been used to assign to each article 3 levels of classification, the highest level defining a coarse classification and the lowest a fine classification of vital articles.

Levels 0 and 1 corresponds to sections and subsections on the main page of Vital articles ([this page](https://en.wikipedia.org/wiki/Wikipedia:Vital_articles/Level/5) for the current version, but don't forget we used an archive from April 1st, 2022 to construct the dataset).

Level 2 was built by collecting the highest level headers in the pages listing the vital articles of each section corresponding to level 1 (for example [this page](https://en.wikipedia.org/wiki/Wikipedia:Vital_articles/Level/5/Physical_sciences/Astronomy) for a current version, but don't forget we used an archive from April 1st, 2022 to construct the dataset)

### Labels of nodes

The labels of each nodes have been saved in the following file in the folder './dataset/main/' : __class0.txt, __class1.txt, __class2.txt.


### Full hierarchy of labels
```
Arts  (3310 articles)
		 Arts  (3310 articles)
				 Architecture (249 articles)
				 Cultural venues (131 articles)
				 Fictional characters (134 articles)
				 General (5 articles)
				 Literature (989 articles)
				 Modern visual arts (301 articles)
				 Music (803 articles)
				 Performing arts (198 articles)
				 Visual arts (500 articles)
Biological and health sciences  (4681 articles)
		 Animals  (2396 articles)
				 Agnatha (17 articles)
				 Amphibians (55 articles)
				 Animal breeds and hybrids (149 articles)
				 Arachnids (52 articles)
				 Arthropoda, others (35 articles)
				 Birds (382 articles)
				 Cnidarians (22 articles)
				 Crustaceans (84 articles)
				 Echinoderms (15 articles)
				 Fishes (299 articles)
				 General classifications (13 articles)
				 Individual animals (94 articles)
				 Insects (196 articles)
				 Invertebrata, others (52 articles)
				 Mammals (539 articles)
				 Mollusks (69 articles)
				 Porifera (5 articles)
				 Proto-mammals (18 articles)
				 Reptiles (297 articles)
				 Reptiliomorphs (3 articles)
		 Biology  (886 articles)
				 Anatomy and morphology (267 articles)
				 Biochemistry and molecular biology (160 articles)
				 Biological processes and physiology (111 articles)
				 Biology basics (47 articles)
				 Botany (9 articles)
				 Cell biology (78 articles)
				 Ecology (39 articles)
				 Evolutionary biology (92 articles)
				 Genetics (41 articles)
				 Zoology (42 articles)
		 Health  (791 articles)
				 Drugs and medication (121 articles)
				 Health and fitness (93 articles)
				 Medicine (199 articles)
				 Morbidity (378 articles)
		 Plants  (608 articles)
				 Fungi (36 articles)
				 Other organisms (69 articles)
				 Plants (503 articles)
Everyday life  (2422 articles)
		 Everyday life  (1191 articles)
				 Clothing and fashion (192 articles)
				 Cooking, food and drink (654 articles)
				 Family and kinship (97 articles)
				 General (3 articles)
				 Home living (18 articles)
				 Household items (74 articles)
				 Sexuality and gender (139 articles)
				 Stages of life (14 articles)
		 Sports, games and recreation  (1231 articles)
				 Entertainment (518 articles)
				 Sports (546 articles)
				 Sports organizations (167 articles)
Geography  (5318 articles)
		 Cities  (2030 articles)
				 Africa (226 articles)
				 Americas (387 articles)
				 Asia (873 articles)
				 Europe and Russia (440 articles)
				 Non-city settlements (17 articles)
				 Oceania (43 articles)
				 Urban studies and planning (44 articles)
		 Countries  (1386 articles)
				 Countries (221 articles)
				 General (19 articles)
				 Regions and country subdivisions (1146 articles)
		 Physical  (1902 articles)
				 Basics (78 articles)
				 Bodies of water (529 articles)
				 Geography by country (52 articles)
				 Islands (490 articles)
				 Isthmuses (10 articles)
				 Land relief (226 articles)
				 Mountain peaks (128 articles)
				 Other hydrological features (109 articles)
				 Other terrestrial features (89 articles)
				 Parks, preserves, and World Heritage Sites (105 articles)
				 Peninsulas and capes (86 articles)
History  (2979 articles)
		 History  (2979 articles)
				 19th century (238 articles)
				 20th century (563 articles)
				 21st century (172 articles)
				 Ancient history (300 articles)
				 Basics (36 articles)
				 Early modern history (270 articles)
				 Historical cities, towns and archaeological sites (104 articles)
				 History by city (39 articles)
				 History by continent and region (42 articles)
				 History by country and subdivision (420 articles)
				 History by ethnicity (8 articles)
				 History by topic (323 articles)
				 Post-classical history (441 articles)
				 Prehistory (23 articles)
Mathematics  (1126 articles)
		 Mathematics  (1126 articles)
				 Algebra (74 articles)
				 Applied mathematics (36 articles)
				 Basics (114 articles)
				 Calculus and analysis (177 articles)
				 Discrete mathematics (140 articles)
				 Foundations (160 articles)
				 Functions (88 articles)
				 Geometry (139 articles)
				 Number theory (50 articles)
				 Statistics and probability (85 articles)
				 Theoretical computer science (63 articles)
People  (15575 articles)
		 Artists, musicians, and composers  (2310 articles)
				 Musicians and composers (1503 articles)
				 Visual artists (807 articles)
		 Entertainers, directors, producers, and screenwriters  (2342 articles)
				 Comedians (219 articles)
				 Dancers and choreographers (200 articles)
				 Entertainers (1424 articles)
				 Other entertainment and fields (298 articles)
				 Television hosts and personalities (201 articles)
		 Military personnel, revolutionaries, and activists  (1012 articles)
				 Military leaders, personnel, and theorists (470 articles)
				 Rebels, revolutionaries and activists (542 articles)
		 Miscellaneous  (1186 articles)
				 Body modification (10 articles)
				 Businesspeople (369 articles)
				 Case studies (25 articles)
				 Chefs, bartenders and winemakers (40 articles)
				 Cosmetics people (10 articles)
				 Crime (250 articles)
				 Explorers (275 articles)
				 Fandom, conventions and festivals (10 articles)
				 Health and fitness (6 articles)
				 Hobbyists (10 articles)
				 Law enforcement and fire service (60 articles)
				 Micronations (2 articles)
				 Other (11 articles)
				 Pseudoscientists (40 articles)
				 Sex work (20 articles)
				 Socialites (40 articles)
				 Tradespeople (8 articles)
		 Philosophers, historians, political and social scientists  (1335 articles)
				 Historians (175 articles)
				 Philosophers (295 articles)
				 Social scientists (865 articles)
		 Politicians and leaders  (2452 articles)
				 Ancient (280 articles)
				 Early modern period [about 1400-1814] (383 articles)
				 Modern [about 1815-1945] (501 articles)
				 Post-1945 (931 articles)
				 Post-classical (357 articles)
		 Religious figures  (500 articles)
				 Buddhism (60 articles)
				 Christianity (241 articles)
				 Hinduism (37 articles)
				 Islam (90 articles)
				 Judaism (29 articles)
				 Others (43 articles)
		 Scientists, inventors, and mathematicians  (1108 articles)
				 Chemistry (79 articles)
				 Computer scientists and programmers (51 articles)
				 Inventors and engineers (185 articles)
				 Life sciences (221 articles)
				 Mathematicians (129 articles)
				 Physical geography (47 articles)
				 Physics and astronomy (338 articles)
				 Pre-modern figures (58 articles)
		 Sports figures  (1210 articles)
				 American football (40 articles)
				 Ancient sports (5 articles)
				 Association football (110 articles)
				 Athletics (91 articles)
				 Auto racing (40 articles)
				 Baseball (56 articles)
				 Basketball (60 articles)
				 Cricket (60 articles)
				 Cycling (20 articles)
				 Figure skating (30 articles)
				 Golf (35 articles)
				 Gymnastics (25 articles)
				 Ice hockey (45 articles)
				 Martial arts (85 articles)
				 Other individual sports (151 articles)
				 Other team sports (59 articles)
				 Rugby (30 articles)
				 Sports business-people, coaches, and referees (152 articles)
				 Swimming (20 articles)
				 Tabletop games (45 articles)
				 Tennis (51 articles)
		 Writers and journalists  (2120 articles)
				 Journalists (395 articles)
				 Writers (1725 articles)
Philosophy and religion  (1408 articles)
		 Philosophy and religion  (1408 articles)
				 Abrahamic religions (411 articles)
				 Eastern religions (194 articles)
				 Mythology (365 articles)
				 Other religions (71 articles)
				 Philosophy (197 articles)
				 Religion and spirituality (170 articles)
Physical sciences  (4290 articles)
		 Astronomy  (886 articles)
				 Astronomical objects (302 articles)
				 Astronomy basics (35 articles)
				 Celestial mechanics (72 articles)
				 Celestial sphere (122 articles)
				 Galactic and extragalactic astronomy (89 articles)
				 Observational astronomy (49 articles)
				 Physical cosmology (51 articles)
				 Planetary science (50 articles)
				 Stellar astronomy (116 articles)
		 Basics and measurement  (360 articles)
				  Science in countries (15 articles)
				 Measurement (302 articles)
				 Science basics (43 articles)
		 Chemistry  (1207 articles)
				 Acids and bases (54 articles)
				 Alloys and ceramic compounds (39 articles)
				 Analysis (40 articles)
				 Basics (136 articles)
				 Chemical elements (204 articles)
				 Chemical engineering (12 articles)
				 Molecular compounds (61 articles)
				 Organic compounds (194 articles)
				 Reactions and synthesis (81 articles)
				 Salts and ions (220 articles)
				 Separation processes (56 articles)
				 Theory (110 articles)
		 Earth science  (849 articles)
				 Air (147 articles)
				 Biological materials (3 articles)
				 Earth (629 articles)
				 Earth science basics (1 articles)
				 Water (69 articles)
		 Physics  (988 articles)
				 Atomic, molecular and optical physics (93 articles)
				 Color (53 articles)
				 Condensed matter physics (78 articles)
				 Electromagnetism (121 articles)
				 Mechanics (273 articles)
				 Nuclear physics (25 articles)
				 Particle physics (111 articles)
				 Physics basics (62 articles)
				 Theory of relativity (35 articles)
				 Thermodynamics (89 articles)
				 Waves (48 articles)
Society and social sciences  (4255 articles)
		 Culture  (2075 articles)
				 Communication (51 articles)
				 Culture (132 articles)
				 Education (353 articles)
				 Ethnology and Anthropology (120 articles)
				 Journalism and mass media (825 articles)
				 Language (594 articles)
		 Politic and economic  (1825 articles)
				 Business and economics (668 articles)
				 Law (567 articles)
				 Organizations (250 articles)
				 Politics and government (230 articles)
				 War and military (110 articles)
		 Social studies  (355 articles)
				 Basics (17 articles)
				 Psychology (138 articles)
				 Society (140 articles)
				 Sociology (60 articles)
Technology  (3148 articles)
		 Technology  (3148 articles)
				 Agriculture (237 articles)
				 Astronomical technology (72 articles)
				 Biotechnology (48 articles)
				 Computing and information technology (741 articles)
				 Electronics (117 articles)
				 Energy (172 articles)
				 General (89 articles)
				 Industry (201 articles)
				 Infrastructure (234 articles)
				 Machinery and tools (271 articles)
				 Medical technology (17 articles)
				 Military technology (303 articles)
				 Navigation and timekeeping (70 articles)
				 Optical technology (57 articles)
				 Space (68 articles)
				 Transportation (451 articles)
```

