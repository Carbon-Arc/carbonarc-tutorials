# Bulk Data - Ingesting alldata for a data identifier to local filesystem

This notebook walks through a tutorial to pull data for `vehicle_registration_data` identifier.

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
from urllib.parse import urlparse, parse_qs
from datetime import datetime, timezone
```


```python
## Read in environment variables and setup constants
load_dotenv()
API_AUTH_TOKEN=os.getenv("API_AUTH_TOKEN", None)
data_identifier = "vehicle_registration_data"
```


```python
# Create API Client
client = APIClient(API_AUTH_TOKEN)
```


```python
# Helper function to create local path from URL
def create_local_path(url: str, base_dir: str = "./data") -> tuple:
    parsed = urlparse(url)

    # Get the table name (second last part of the path)
    path_parts = parsed.path.strip("/").split("/")
    table_name = path_parts[-2]
    filename = path_parts[-1]

    # Parse query parameters
    query_params = parse_qs(parsed.query)
    param_dirs = [f"{key}={value[0]}" for key, value in query_params.items()]

    # Construct full local path
    local_dir = os.path.join(base_dir, table_name, *param_dirs)
    local_path = os.path.join(local_dir, f"{filename}.parquet")

    return local_dir, local_path

def download_manifest_files(manifest):
    manifest_files = manifest.get("files", [])
    print(f"Downloading {len(manifest_files)} files")
    for file_info in manifest_files:
        manifest_file_url = file_info["url"]
        local_dir, local_path = create_local_path(manifest_file_url)
        # skip download if file exists
        if os.path.exists(local_path):
            print(f"File '{local_path}' exists. Skipping download.")
            continue

        # make sure local_dir exists
        os.makedirs(local_dir, exist_ok=True)
        print(f"Downloading {manifest_file_url} to: {local_path}")
        client.download_alldata_to_file(manifest_file_url, local_path)  
```


```python
# Download all history for give data identifier

last_ingest_time = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S')
print(last_ingest_time)

manifest = client.get_alldata_manifest(data_identifier, created_since=None)
print(f"\nManifest for {data_identifier}:")
download_manifest_files(manifest)
```

    2025-04-30T21:38:00
    
    Manifest for vehicle_registration_data:
    Downloading 4 files
    File './data/vehicle_registration_data/drop_partition_id=1744757997/part-00000-a92189b3-b568-4068-9e04-0a34f59a3a88.parquet' exists. Skipping download.
    File './data/vehicle_registration_data/drop_partition_id=1740512141/part-00000-b571f9bb-10c0-4716-87ce-0ea5a58e20af.parquet' exists. Skipping download.
    File './data/vehicle_registration_data/drop_partition_id=1740512141/part-00000-d2ef64bc-680d-4378-83d6-cb146d58005e.parquet' exists. Skipping download.
    File './data/vehicle_registration_data/drop_partition_id=1741819689/part-00000-69ef360f-356a-425d-91ad-eeddd9bba4d9.parquet' exists. Skipping download.



```python
# Downloading files created since last ingestions, this needs last ingestion time
manifest = client.get_alldata_manifest(data_identifier, created_since=last_ingest_time)
print(f"\nManifest for {data_identifier}:")
download_manifest_files(manifest)
```

    
    Manifest for vehicle_registration_data:
    Downloading 0 files



```python

```
