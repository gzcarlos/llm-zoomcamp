## How to responds to questions

> Before anything else execute this command to install all dependencies 
```bash
pip install requests jupyter elasticsearch tqdm openai tiktoken
```

### Q1 Running Elastic
Install and run elasticsearch engine locally using docker with this command
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
Then run the command suggested:
```bash
curl localhost:9200
```
And look under `version` the key `build_hash` and copy the value.

### Q2 Indexing the data

Execute this commands

Get all `documents`
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

Connect to Elastic Search instance already running with the command of `Q1`
```python
from elasticsearch import Elasticsearch
# create client for elastic search service running locally
es_client = Elasticsearch('http://localhost:9200') 
```

Create an index in Elastic Search to save all the `documents`
```python
# create an index to use in elastic search
index_settings = {
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
# give it a name
index_name = "course-questions"

es_client.indices.create(index=index_name, body=index_settings)
```

Load all documents (using tqdm to see a progress bar)
```python
from tqdm.auto import tqdm
# index all documents
for doc in tqdm(documents):
    es_client.index(index=index_name, document=doc)
```

### Q3 Searching

Define function to execte the search on a query
```python
def elastic_search(query):
    search_query = {
        "size": 5,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^4", "text"],
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

    response = es_client.search(index=index_name, body=search_query)
    
    result_docs = []
    print(response)
    
    for hit in response['hits']['hits']:
        result_docs.append(hit['_source'])
    
    return result_docs
```



### Q4 Filtering

Modify the function to make look in the specified course

```python
def elastic_search(query):
    search_query = {
        "size": 5,
        "query": {
            "bool": {
                "must": {
                    "multi_match": {
                        "query": query,
                        "fields": ["question^4", "text"],
                        "type": "best_fields"
                    }
                },
                "filter": {
                    "term": {
                        "course": "machine-learning-zoomcamp" #"data-engineering-zoomcamp"
                    }
                }
            }
        }
    }

    response = es_client.search(index=index_name, body=search_query)
    
    result_docs = []
    print(response)
    
    for hit in response['hits']['hits']:
        result_docs.append(hit['_source'])
    
    return result_docs
```

### Q5 Building a prompt

Set the OpenAI API Key obtained from [Platform OpenAI](https://platform.openai.com/)
```bash
export OPENAI_API_KEY=[the key generated]
```

Import the OpenAI pacakge
```python
from openai import OpenAI
import os
# create client
client = OpenAI()
#confirm the API Key is in the envorinment variables
os.environ['OPENAI_API_KEY']
```

Define the function to build the `prompt`s:
```python
def build_prompt(query, search_results):
    prompt_template = """
You're a course teaching assistant. Answer the QUESTION based on the CONTEXT from the FAQ database.
Use only the facts from the CONTEXT when answering the QUESTION.

QUESTION: {question}

CONTEXT:
{context}
""".strip()
    context_template = """
Q: {question}
A: {text}
""".strip()

    context = ""
    
    for doc in search_results:
        context = context +  context_template.format(question=doc['question'], text=doc['text']) +"\n\n"
    
    prompt = prompt_template.format(question=query, context=context).strip()
    return prompt
```

Have a look at the `prompt`
```python
prompt = build_prompt(q3, elastic_search(q3))
print(prompt)
```

See the length of the `prompt`
```python
len(prompt)
```

### Q6 Tokens

Import `tiktokens` and set the enconding
```python
import tiktoken
encoding = tiktoken.encoding_for_model("gpt-4o")
```

Then get the tokens of the `prompt`
```python
len(encoding.encode(prompt))
```