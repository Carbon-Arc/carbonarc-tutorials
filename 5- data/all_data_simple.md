# Bulk Data - Getting started

## Prerequisites
- `carbonarc` sdk and other dependencies in local python environment

```
python3.10 -m venv .venv
source .venv/bin/activate
pip install git+https://github.com/Carbon-Arc/carbonarc
pip install python-dotenv
```

## Setup

1. create .env file in the same directory as this notebook or export `API_AUTH_TOKEN` env variable
2. If using .env add the following lines to the .env :

```
API_AUTH_TOKEN=<api auth token from (https://platform.carbonarc.co/profile)>
```


```python
# Import required dependencies
import os
from dotenv import load_dotenv
from carbonarc import APIClient
```


```python
## Read in environment variables
load_dotenv()
API_AUTH_TOKEN=os.getenv("API_AUTH_TOKEN")
```


```python
# Create API Client
assert API_AUTH_TOKEN, "API_AUTH_TOKEN must be set in environment variables"
client = APIClient(API_AUTH_TOKEN)
```


```python
## Get insights data identifiers
data_identifiers = client.data.get_alldata_data_identifiers()
print("Data Identifiers:")
for data_id in data_identifiers["data"]:
    print(data_id["data_identifier"], ": ", data_id["description"])
```


```python
## Getting all manifests files for a given data identifier
data_identifier = "vehicle_registration_data"
manifests = client.data.get_alldata_manifest(data_identifier)
print(f"\nManifest for {data_identifier}:")
for manifest in manifests["files"]:
    print(manifest)
```


```python
# Download the file from manifest url
manifest_file_url="https://platform.carbonarc.co/api/v2/data/alldata/vehicle_registration_data/part-00000-a92189b3-b568-4068-9e04-0a34f59a3a88?drop_partition_id=1744757997"
output_dir="./output/alldata/vehicle_registration_data"
output_file_path = os.path.join(output_dir, "part-00001.parquet")

os.makedirs(output_dir, exist_ok=True)

print(f"Downloading manifest file to: {output_file_path}")
client.data.download_alldata_to_file(manifest_file_url, output_file_path)
```
