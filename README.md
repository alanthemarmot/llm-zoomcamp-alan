# llm-zoomcamp
repo for my llm datatalks zoomcamp work

#Module 1: Introduction

## 1.1 Introduction

* llm
* RAG
* RAG Architecture
* Course Outcome



## 1.2 Preparing the Environment

Video - codespaces (https://www.youtube.com/watch?v=ozCpmkbJNJE&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R&index=2)

* Open in VSCode codespaces
* pip install tdqm notebook==7.1.2 openai elasticsearch scikit-learn pandas
* get key for openai
* add to environment.... export OPENAI_API_KEY="....."
* open jupyter notebook
* notebook - 1.2 - Environment Setup
* Code in notebook, get openai api working, get anaconda/miniconda working as an alternative
* wget https://repo.anaconda.com/archive/Anaconda3-2024.02-1-Linux-x86_64.sh or miniconda install... only needed if not working on codespaces/locally
* with conda you dont need to install most of the packages, already installed etc.


## 1.3 Retrieval

Video - (https://www.youtube.com/watch?v=olvem333Bqo&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R&index=3)

* We will use the implemntation in the [build-your-own-search-engine workshop] https://github.com/alexeygrigorev/build-your-own-search-engine 
* [minsearch] (https://github.com/alexeygrigorev/minsearch)
* !wget https://raw.githubusercontent.com/alexeygrigorev/minsearch/main/minsearch.py
* Use the minsearch library to look through the cosumts.json file (look into the parameters of the minsearch functions to understand usage)


## 1.4 Generating answers with ChatGPT over the document

Video - (https://www.youtube.com/watch?v=qz316T3U49Q&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R&index=4)


## 1.4.2 OPENAI Alternatives

* Video: https://www.youtube.com/watch?v=HObjFso2UJE&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R&index=5
* List of alternatives:  https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-intro/open-ai-alternatives.md


## 1.5 The RAG Flow Cleaning and Modularizing Code

* Video: https://www.youtube.com/watch?v=vkTiVwwch6A&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R&index=6


## 1.6 Search with Elasticsearch

* Video: https://www.youtube.com/watch?v=1lgbR5wMvsI&list=PL3MmuxUbc_hIB4fSqLy_0AfTjVLpgjV3R&index=7

* Replacing the search script with Elasticsearch, runs in memory etc.
* Run Elasticsearch
* Index the documents
* Replace MinSearch with Elasticsearch

Running ElasticSearch in new terminal:

```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

```python
{
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"} 
        }
    }
}
```

```python
{
    "size": 5,
    "query": {
        "bool": {
            "must": {
                "multi_match": {
                    "query": query,
                    "fields": ["question^3", "text", "section"],
                    "type": "best_fields"
                }
            },
            "filter": {
                "term": {
                    "course": "data-engineering-zoomcamp"
                }
            }
        }
    }
}
```