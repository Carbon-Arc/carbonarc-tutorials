# Bulk Data Tutorials

## Prerequisites
- Python 3.x
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
DATA_IDENTIFIERS = {
    "pos_instore_online_data": {
        "outputdir": "output/pos/instore_online",
        "filters": {},
    }
}
```


```python
# Create API Client
api_client = APIClient(API_AUTH_TOKEN)
```


```python
# Download data for each data identifier
for data_id, data in DATA_IDENTIFIERS.items():
    # print(f"Downloading data for {data_id}")
    outputdir = data["outputdir"]
     
    # Get data manifest, this will contain all the files that can be downloaded
    # You can track the downloaded files to maintain ingestion state
    manifest = api_client.get_alldata_data_manifest(data_id)
    print(f"Data id: {data_id}, total files: {len(manifest['files'])}")
    # print(f"Manifest: {manifest}")
    
    # Download all files in the manifest, this can be done in parallel to speed up the process
    for file in manifest["files"]:
        # Download the file to the output directory
        print(f"Downloading file {file}...")
        print(f"{file['size_bytes']/1024/1024} MB")
        # Uncomment the line below to download files locally
        # api_client.download_alldata_file(file["url"], outputdir)
        
        # Download the file to S3
        api_client.download_alldata_data_file(
            file["url"],
            "."
        )
        
    print(f"Downloaded all files for {data_id}")
```
