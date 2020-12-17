# Team NA Coders


## Project team:

* [Nathaniel Rupsis](https://github.com/rupsis/) (team captain)
* [Hoa Le](https://github.com/hle027)


## Project details:

*This is the final course project for CS 410 - Text Retrieval & Mining course at the University of Illinois at Urbana-Champaign.*

For the project our team decided to participate in the CORD-19 Open Research Dataset IR Competition. The CORD-19 dataset contains over 57000 scholarly articles available for the global research community. Our goal is to build a live system that supports search on this massive dataset.

Our team is prepared to research, test, and implement state of the art search methods and techniques. For this project, we'll be using python, and focusing on implementing and tuning the methods mentioned in the proposal(query expansion, feedback, rank fusion, etc). Time permitting, we'll test out and incorporate additional retrieval methods listed [here](https://en.wikipedia.org/wiki/Category:Information_retrieval_techniques), such as [Compound term processing](https://en.wikipedia.org/wiki/Compound-term_processing) and [Contextual Searching](https://en.wikipedia.org/wiki/Contextual_searching).

The complete project proposal can be found [here](https://github.com/rupsis/CourseProject/blob/main/NA_Coders_Final_Project_Proposal.pdf).

## Video Presentation:
Link to our video presentation for this project can be found here: https://mediaspace.illinois.edu/media/t/1_cgdhmw3n

## Prerequisites
### Packages
*	Metapy

We use [metapy](https://github.com/meta-toolkit/metapy)  to build and evaluate our search engine. 
If you have not installed metapy so far, use the following commands to get started.

```bash
# Ensure your pip is up to date
pip install --upgrade pip

# install metapy!
pip install metapy pytoml
```

* argparse

The argparse module makes it easy to write user friendly command line interfaces. 
 argparse should work on Python 2.3 and newer.

```bash
# Try one of the following installation methods:
pip install argparse
python setup.py install
easy_install argparse
putting argparse.py in some directory listed in sys.path should also work
```

# Running the program

Directory:
```
| - /test
        \- line.toml
        ... data
| - /train
        \- line.toml
        ... data
```
A test and train directory need to exist with a `line.toml` file which specifies the metaData. 

Note: the test and train dataset are too large to be uploaded to this repo. Please see the link below to download the dataset. You need to put `line.toml` file in the downloaded train dataset in order for `build_courpse.py` to create the index.

The test files can be obtained from https://drive.google.com/file/d/1FCW8fmcneow5yyDgApkPIGM-r2x6OFkm/view?usp=sharing

The train files can be obtained from https://drive.google.com/file/d/1E_Y-MkNvoOYoCZUZZa8JJ3ExiHYTfKTo/view?usp=sharing
```
// line.toml

type = "line-corpus"

metadata = [{name = "uid", type = "string"}]

```

To build the index, simply run:
```
// remove the index else the evaluator will use a cached version
rm -rf idx/ && python3 build_courpse.py train 
```

where the first argument is either "test" or "train" (need to have corresponding test / train files fro MetaPy to work)

To run the evaluator:
```
python search_eval.py config_train.toml
```

where the config argument is either `config_train.toml` or `config_test.toml`

## File & Code Walkthrough
### search_eval.py
This is our main file containing most of the crucial functions used for creating ranking algorithms and searching an index as well as evaluate and writing the results.
Below is the list of functions in the search_eval.py file:

#### expand_query(query):
*	Description: This function is a simple approach to query expansion. It expands on some important keywords such as covid-19, coronavirus, test, mark, spread, etc… It also contains some pre-processing methods such as removing punctuation, converting to lower case, filtering out common words, etc…
*	Parameter: query
*	What the function returns: return list of query words containing new keywords.

#### load_queries(): 
*	Description: This function retrieves query from the xml file. 
*	Parameter: None
*	What the function returns: return tuple array of queryID and query pairs.

#### load_ranker(cfg_file):
*	Description: This function returns the Ranker object to evaluate 
*	Parameter: cfg_file is the path to a configuration file used to load the index. 
*	What the function returns: Ranker object.

#### saveResults(prediction_results):
*	Description: This function writes the results to “predictions.txt” file.
*	Parameter: prediction_results (a list of 3 elements returned from running ranking algorithms)
*	What the function returns: return list of query words containing new keywords.

#### runQueries(queries) :
*	Description: This is our main function for creating inverted index and ranking algorithms. 
*	Parameter: it takes the output which is the query from load_queries function as input.
*	What the function returns: The function saveResults mentioned above is called to store the output.

### build_courpse.py
This function contains code to extract information such as title, abstract, introduction, etc… from metadata.csv using example code. It also writes the data to .dat file and create metadata file for metapy.

### config.toml
We have two separate config file : config_test and config_train. Each file contains general settings that deal with paths related to the its corresponding dataset. We specified some details about the corpus and its analyzer. For example:
dataset = "train"    # name of the dataset
corpus = "line.toml"      # type of corpus
query-judgements = "qrels.txt"   # path to query judgement file
[[analyzers]]
method = "ngram-word" # set of co-occurring words
ngram = 1 # specified to one word aka unigram
filter = "default-unigram-chain"

### stopwords.txt
A text file contains list of common English words that do not add much meaning to a sentence. For example: “the”,”a”,”an”,etc…They can be safely ignored, and this is part of pre-processing to filter out useless information.

### train & test folders
Contain the cord-19 dataset and metadata.csv file. The train data also contains query relevance file.
predictions.txt
This is a text file that has the same format as relevance judgment file which is (query_ID doc_uid relevance_score). It contains our final top 1000 documents per query and is used for computing nDCG score on the leaderboard.

## Implementation Overview

### Task Definitions
1.	Collecting data

i.	Unzip and extract data from the provided train.zip 

ii.	Collect and store information from metadata.csv such as title, abstract, introduction, etc… 

2.	Building a corpus

i.	Create train/test.dat file. Each line in .dat file represents one document.

ii.	Create metadata.dat file containing the collected information from step 1 part ii.

iii.	Create line.toml configuration file containing corpus input formats. 

3.	Implementing simple query expansion

i.	Collect query from xml file. Query are stored in a tuple array (queryID, query)

ii.	Create query expansion function with some preprocessing methods.

4.	Create an inverted index

i.	Set up the config file. This file contains fields that need to be specified and makes references/pathing to several files.

ii.	Write code to create inverted index.

5.	Searching the index

i.	Create bm25 ranker.

ii.	Run query to search the index and returns top 1000 documents per query as sorted vector pairs (doc_id, double). 

iii.	Store the query-ids, results and the corresponding uid (document_id retrieved from metadata file) and write to predictions.txt file.

6.	Optimize nDCG score

i.	Submit predictions.txt file and check the nDCG score on the leaderboard. Address potential problems and identify methods to improve nDCG score.

## Potential Issues Encounter

If `UnicodeEncodeError: 'charmap' codec can't encode characters error` appears, running python3 seems to fix that. One solution which we will add to our code is to write `(encoding = “utf-8”)` when reading .json or .dat file. 
We ran into some disk problem when implementing BERT-large method for reranking. We decided to exclude it from our final code to avoid this potential issue.

## Key Findings & Potential Improvements

Even though we beat the baseline and made it to leaderboard, we think that the result could be further enhanced. Below is what we identify that might help with improving the result:
1.	Add more information to the index (actual documents)
2.	Weight the keywords better since relevancy will definitely be based on the keywords (covid, alcohol, spread, African American, health, etc…)
3.	Further expanding on query. Query expansion has helped enhancing our result greatly. 
4.	Implementing BERT large for reranking. We ran into disk problems when trying to implement it. 


## Citation
https://github.com/meta-toolkit/metapy

Bhavya, Dec (2020) MP 2.4 [Source code] https://github.com/CS410Fall2020/MP2.4

## Contact

nrupsis@gmail.com

hle027@gmail.com

