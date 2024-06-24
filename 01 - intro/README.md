# llm-zoomcamp
repo for my llm datatalks zoomcamp work - Week 1

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

## 1.7 Homework

In this homework, we'll learn more about search and use Elastic Search for practice. 

> It's possible that your answers won't match exactly. If it's the case, select the closest one.

## Q1. Running Elastic 

Run Elastic Search 8.4.3, and get the cluster information. If you run it on localhost, this is how you do it:

```bash
curl localhost:9200
```

What's the `version.build_hash` value?


## Getting the data

Now let's get the FAQ data. You can run this snippet:

```python
import requests 

docs_url = 'https://github.com/DataTalksClub/llm-zoomcamp/blob/main/01-intro/documents.json?raw=1'
docs_response = requests.get(docs_url)
documents_raw = docs_response.json()

documents = []

for course in documents_raw:
    course_name = course['course']

    for doc in course['documents']:
        doc['course'] = course_name
        documents.append(doc)
```

Note that you need to have the `requests` library:

```bash
pip install requests
```

## Q2. Indexing the data

Index the data in the same way as was shown in the course videos. Make the `course` field a keyword and the rest should be text. 

Don't forget to install the ElasticSearch client for Python:

```bash
pip install elasticsearch
```

Which function do you use for adding your data to elastic?

* `insert`
* `index`
* `put`
* `add`

## Q3. Searching

Now let's search in our index. 

We will execute a query "How do I execute a command in a running docker container?". 

Use only `question` and `text` fields and give `question` a boost of 4, and use `"type": "best_fields"`.

What's the score for the top ranking result?

* 94.05
* 84.05
* 74.05
* 64.05

Look at the `_score` field.

## Q4. Filtering

Now let's only limit the questions to `machine-learning-zoomcamp`.

Return 3 results. What's the 3rd question returned by the search engine?

* How do I debug a docker container?
* How do I copy files from a different folder into docker container’s working directory?
* How do Lambda container images work?
* How can I annotate a graph?

## Q5. Building a prompt

Now we're ready to build a prompt to send to an LLM. 

Take the records returned from Elasticsearch in Q4 and use this template to build the context. Separate context entries by two linebreaks (`\n\n`)
```python
context_template = """
Q: {question}
A: {text}
""".strip()
```

Now use the context you just created along with the "How do I execute a command in a running docker container?" question 
to construct a prompt using the template below:

```
prompt_template = """
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: {question}

CONTEXT:
{context}
""".strip()
```

What's the length of the resulting prompt? (use the `len` function)

* 962
* 1462
* 1962
* 2462

## Q6. Tokens

When we use the OpenAI Platform, we're charged by the number of 
tokens we send in our prompt and receive in the response.

The OpenAI python package uses `tiktoken` for tokenization:

```bash
pip install tiktoken
```

Let's calculate the number of tokens in our query: 

```python
encoding = tiktoken.encoding_for_model("gpt-4o")
```

Use the `encode` function. How many tokens does our prompt have?

* 122
* 222
* 322
* 422

Note: to decode back a token into a word, you can use the `decode_single_token_bytes` function:

```python
encoding.decode_single_token_bytes(63842)
```

## Bonus: generating the answer (ungraded)

Let's send the prompt to OpenAI. What's the response?  

Note: you can replace OpenAI with Ollama. See module 2.

## Bonus: calculating the costs (ungraded)

Suppose that on average per request we send 150 tokens and receive back 250 tokens.

How much will it cost to run 1000 requests?

You can see the prices [here](https://openai.com/api/pricing/)

On June 17, the prices for gpt4o are:

* Input: $0.005 / 1K tokens
* Output: $0.015 / 1K tokens

You can redo the calculations with the values you got in Q6 and Q7.


## Submit the results

* Submit your results here: https://courses.datatalks.club/llm-zoomcamp-2024/homework/hw1
* It's possible that your answers won't match exactly. If it's the case, select the closest one.

