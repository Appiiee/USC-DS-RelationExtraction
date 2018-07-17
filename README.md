## Entity and Relation Extraction with Knowledge Bases

This repository includes recent models and data for **sentence-level extraction of entities and relations** *using knowledge bases (i.e., distant supervision)*. In particular, it contains the source code for WWW'17 paper *[CoType: Joint Extraction of Typed Entities and Relations with Knowledge Bases](https://arxiv.org/pdf/1610.08763.pdf)*.

**Task Setting**: Given a text corpus with entity mentions *detected* and *heuristically labeled* by distant supervision, teh task aims to identify relation types/labels between a pair of entity mentions based on the sentence context where they co-occur.

**Run on your own data**: Code for producing the JSON files from a raw corpus for running CoType and baseline models is [here](https://github.com/shanzhenren/StructMineDataPipeline).


## Performance
Performance comparison with several *relation extraction* systems over KBP 2013 dataset (**sentence-level extraction**). 

Method | Precision | Recall | F1 
-------|-----------|--------|----
Mintz (our implementation, [Mintz et al., 2009](http://web.stanford.edu/~jurafsky/mintz.pdf)) | 0.296 | 0.387 | 0.335 
LINE + Dist Sup ([Tang et al., 2015](https://arxiv.org/pdf/1503.03578.pdf)) | **0.360** | 0.257 | 0.299 
MultiR ([Hoffmann et al., 2011](http://raphaelhoffmann.com/publications/acl2011.pdf)) | 0.325 | 0.278 | 0.301 
FCM + Dist Sup ([Gormley et al., 2015](http://www.aclweb.org/anthology/D15-1205)) | 0.151 | **0.498** | 0.300 
**CoType** ([Ren et al., 2017](https://arxiv.org/pdf/1610.08763v1.pdf)) | 0.348 | 0.406 | **0.369**

## Dependencies

We will take Ubuntu for example.

* python 2.7
* Python library dependencies
```
$ pip install pexpect ujson tqdm
```

* [stanford coreNLP 3.7.0](http://stanfordnlp.github.io/CoreNLP/) and its [python wrapper](https://github.com/stanfordnlp/stanza). Please put the library under `CoType/code/DataProcessor/'.

```
$ cd code/DataProcessor/
$ git clone git@github.com:stanfordnlp/stanza.git
$ cd stanza
$ pip install -e .
$ wget http://nlp.stanford.edu/software/stanford-corenlp-full-2016-10-31.zip
$ unzip stanford-corenlp-full-2016-10-31.zip
```
* [eigen 3.2.5](http://bitbucket.org/eigen/eigen/get/3.2.5.tar.bz2) (already included). 


## Data
We [process](https://github.com/shanzhenren/StructMineDataPipeline) (using our [data pipeline](https://github.com/shanzhenren/StructMineDataPipeline)) three public datasets to our JSON format. We ran [Stanford NER](https://nlp.stanford.edu/software/CRF-NER.shtml) on training set to detect entity mentions, and performed distant supervision using [DBpediaSpotlight](https://github.com/dbpedia-spotlight/dbpedia-spotlight) to assign type labels:

   * **BioInfer**: 100k PubMed paper abstracts as training data and 1,530 manually labeled biomedical paper abstracts from [PubMed-BioInfer](http://mars.cs.utu.fi/BioInfer/) ([Pyysalo et al., 2007](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-8-50)) as test data. It consists of 94 relation types (protein-protein interactions) and over 2,000 entity types (from MESH ontology). ([Download JSON](https://drive.google.com/drive/folders/0B--ZKWD8ahE4RmFBTjR6aUJjTkU?usp=sharing))
   * **NYT-sent** ([Riedel et al., 2011](https://pdfs.semanticscholar.org/db55/0f7af299157c67d7f1874bf784dca10ce4a9.pdf)): 1.18M sentences sampled from 294K New York Times news articles ([original data](http://iesl.cs.umass.edu/riedel/ecml/)). In their sentence-level evaluation, 395 sentences are manually annotated with 24 relation types and 47 entity types. ([Download JSON](https://drive.google.com/drive/folders/0B--ZKWD8ahE4UktManVsY1REOUk?usp=sharing))
   * **Wiki-KBP**: the training corpus contains 1.5M sentences sampled from 780k [Wikipedia articles](https://github.com/xiaoling/figer) ([Ling & Weld, 2012](http://xiaoling.github.io/pubs/ling-aaai12.pdf)) plus ~7,000 sentences from 2013 KBP corpus. Test data consists of 14k system-labeled sentences from [2013 KBP slot filling](http://surdeanu.info/kbp2013/) assessment results. It has 13 relation types and 126 entity types after filtering of numeric value relations. ([Download JSON](https://drive.google.com/drive/folders/0B--ZKWD8ahE4RjFLUkVQTm93WVU?usp=sharing))

Please put the data files in corresponding subdirectories under `CoType/data/source`


## Makefile
We have included compilied binaries. If you need to re-compile `retype.cpp` under your own g++ environment
```
$ cd CoType/code/Model/retype; make
```

## Default Run
Run CoType for the task of *Relation Extraction* on the Wiki-KBP dataset

Start the Stanford corenlp server for the python wrapper.
```
$ java -mx4g -cp "code/DataProcessor/stanford-corenlp-full-2016-10-31/*" edu.stanford.nlp.pipeline.StanfordCoreNLPServer
```

Feature extraction, embedding learning on training data, and evaluation on test data.
```
$ ./run.sh  
```

For relation classification, the "none"-labeled instances need to be first removed from train/test JSON files. The hyperparamters for embedding learning are included in the run.sh script.

## Parameters - run.sh
Dataset to run on.
```
Data="KBP"
```
- Hyperparameters for *relation extraction*:
```
- KBP: -negative 3 -iters 400 -lr 0.02 -transWeight 1.0
- NYT: -negative 5 -iters 700 -lr 0.02 -transWeight 7.0
- BioInfer: -negative 5 -iters 700 -lr 0.02 -transWeight 7.0
```
Hyperparameters for *relation classification* are included in the run.sh script.

## Evaluation
Evaluates relation extraction performance (precision, recall, F1): produce predictions along with their confidence score; filter the predicted instances by tuning the thresholds.
```
$ python code/Evaluation/emb_test.py extract KBP retype cosine 0.0
$ python code/Evaluation/tune_threshold.py extract KBP emb retype cosine
```

## Prediction
The last command in *run.sh* generates json file for predicted results, in the same format as test.json in data/source/$DATANAME, except that we only output the predicted relation mention labels. Replace the second parameter with whatever threshold you would like.
```
$ python code/Evaluation/convertPredictionToJson.py $Data 0.0
```

## Reference
Please cite the following paper if you find the codes and datasets useful:
```
@inproceedings{ren2017cotype,
 author = {Ren, Xiang and Wu, Zeqiu and He, Wenqi and Qu, Meng and Voss, Clare R. and Ji, Heng and Abdelzaher, Tarek F. and Han, Jiawei},
 title = {CoType: Joint Extraction of Typed Entities and Relations with Knowledge Bases},
 booktitle = {Proceedings of the 26th International Conference on World Wide Web},
 year = {2017},
 pages = {1015--1024},
} 

```
