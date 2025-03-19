# Introduction

## Prerequisites
- `carbonarc` library

## Setup

1. create .env file in the same directory as this notebook
2. add the following lines to the .env file:

```
API_AUTH_TOKEN=<api auth token from (https://platform.carbonarc.co/profile)>
```


```python
# Import required dependencies
import os
from carbonarc import APIClient
```


```python
# Get client_id and api key from https://platform.carbonarc.co/profile and add to environment variable
API_AUTH_TOKEN=os.getenv("API_AUTH_TOKEN", None)
```


```python
# Create API Client
api_client = APIClient(API_AUTH_TOKEN)
```

Explore Ontology


```python
# Get ontology representations
entity_representations = api_client.get_ontology_entity_representations()
```


```python
# Get ontology entities
entities = api_client.get_ontology_entities()
```
