# Bulk Data S3 Upload

## Prerequisites
- `carbonarc` library
- `boto3` library

## Setup

1. create .env file in the same directory as this notebook
2. add the following lines to the .env file:
```
API_AUTH_TOKEN=<api auth token from (https://platform.carbonarc.co/profile)>
AWS_ACCESS_KEY_ID=<aws access key id>
AWS_SECRET_ACCESS_KEY=<aws secret access key>
AWS_S3_BUCKET=<aws s3 bucket name>

```




```python
# Import required dependencies
import os

import boto3
from carbonarc import CarbonArcClient
```


```python
API_AUTH_TOKEN=os.getenv("API_AUTH_TOKEN")
S3_BUCKET=os.getenv("S3_BUCKET")
```


```python

```


```python
DATA_IDENTIFIERS = {
    "CA0031": {
        "outputdir": "output/CA0031",
        "filters": {},
    }
}
```


```python
# Create API Client
assert API_AUTH_TOKEN, "API_AUTH_TOKEN must be set in environment variables"
api_client = CarbonArcClient(API_AUTH_TOKEN)
```


```python
# Initialize S3 client with environment variables
s3_client = boto3.client(
    's3',
    aws_access_key_id=os.environ.get('AWS_ACCESS_KEY_ID'),
    aws_secret_access_key=os.environ.get('AWS_SECRET_ACCESS_KEY'),
    aws_session_token=os.environ.get('AWS_SESSION_TOKEN'),  # Optional, for temporary credentials
    region_name=os.environ.get('AWS_REGION', 'us-east-1')  # Default to us-east-1 if not specified
)
```


```python
# Download data for each data identifier
assert isinstance(S3_BUCKET, str), "S3_BUCKET must be set in environment variables"


for data_id, data in DATA_IDENTIFIERS.items():
    # print(f"Downloading data for {data_id}")
    params = data["filters"]
    outputdir = data["outputdir"]
     
    # Get data manifest, this will contain all the files that can be downloaded
    # You can track the downloaded files to maintain ingestion state
    manifest = api_client.data.get_bulk_data_manifest(data_id)
    print(f"Data id: {data_id}, total files: {len(manifest['files'])}")
    # print(f"Manifest: {manifest}")
    
    # Download all files in the manifest, this can be done in parallel to speed up the process
    for file in manifest["files"]:
        # Download the file to the output directory
        print(f"Downloading file {file}...")
        print(f"{file['size_bytes']/1024/1024} MB")
        # Uncomment the line below to download files locally
        # api_client.download_bulk_data_file(file["url"], outputdir)
        
        # Download the file to S3
        api_client.data.download_bulk_data_to_s3(
            s3_client,
            file["url"],
            s3_bucket=S3_BUCKET,
            s3_key_prefix=outputdir,

        )
        
    print(f"Downloaded all files for {data_id}")
```
