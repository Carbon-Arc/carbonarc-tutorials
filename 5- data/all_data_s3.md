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
from io import BytesIO

import boto3
from carbonarc import APIClient
from carbonarc.manager import HttpRequestManager

```


```python
API_AUTH_TOKEN=os.getenv("API_AUTH_TOKEN")
S3_BUCKET=os.getenv("S3_BUCKET")
```


```python
def download_alldata_to_s3(
    file_url: str, 
    request_manager: HttpRequestManager,
    s3_bucket: str, 
    s3_key_prefix: str, 
    chunk_size: int = 5 * 1024 * 1024,  # Default to 100MB
):
    print(f"Downloading file {file_url} to S3...")
    
    # Ensure chunk size is at least 5MB (AWS requirement for multipart uploads)
    if chunk_size < 5 * 1024 * 1024:
        chunk_size = 5 * 1024 * 1024
        print("Chunk size adjusted to 5MB to meet AWS minimum part size requirement")
    
    # Make the request
    response = request_manager.get_stream(file_url)
    response.raise_for_status()
    
    # Extract filename from response headers
    filename = response.headers["Content-Disposition"].split("filename=")[1].strip('"')
    
    # Create the full S3 key (path + filename)
    s3_key = f"{s3_key_prefix.rstrip('/')}/{filename}"
    
    # Initialize S3 client with environment variables
    s3_client = boto3.client(
        's3',
        aws_access_key_id=os.environ.get('AWS_ACCESS_KEY_ID'),
        aws_secret_access_key=os.environ.get('AWS_SECRET_ACCESS_KEY'),
        aws_session_token=os.environ.get('AWS_SESSION_TOKEN'),  # Optional, for temporary credentials
        region_name=os.environ.get('AWS_REGION', 'us-east-1')  # Default to us-east-1 if not specified
    )
    
    # Check if file is small enough for direct upload
    content_length = int(response.headers.get('Content-Length', 0))
    
    # If file is small (less than 10MB) or content length is unknown, use simple upload
    if content_length > 0 and content_length < 10 * 1024 * 1024:
        print(f"File is small ({content_length} bytes), using simple upload")
        content = response.content
        s3_client.put_object(
            Bucket=s3_bucket,
            Key=s3_key,
            Body=content
        )
        print(f"File uploaded successfully to s3://{s3_bucket}/{s3_key}")
        return f"s3://{s3_bucket}/{s3_key}"
    
    # For larger files, use multipart upload
    print(f"Initiating multipart upload to s3://{s3_bucket}/{s3_key}")
    multipart_upload = s3_client.create_multipart_upload(
        Bucket=s3_bucket,
        Key=s3_key
    )
    
    upload_id = multipart_upload['UploadId']
    parts = []
    part_number = 1
    
    try:
        # Use a buffer to collect chunks until we have at least 5MB
        buffer = BytesIO()
        buffer_size = 0
        
        for chunk in response.iter_content(chunk_size=1024 * 1024):  # Read in 1MB chunks
            if not chunk:
                continue
                
            # Add the chunk to our buffer
            buffer.write(chunk)
            buffer_size += len(chunk)
            
            # If we have at least 5MB (or this is the last chunk), upload the part
            if buffer_size >= chunk_size:
                # Reset buffer position to beginning for reading
                buffer.seek(0)
                
                # Upload the part
                part = s3_client.upload_part(
                    Bucket=s3_bucket,
                    Key=s3_key,
                    PartNumber=part_number,
                    UploadId=upload_id,
                    Body=buffer.read()
                )
                
                # Add the part info to our parts list
                parts.append({
                    'PartNumber': part_number,
                    'ETag': part['ETag']
                })
                
                print(f"Uploaded part {part_number} ({buffer_size} bytes)")
                part_number += 1
                
                # Reset the buffer
                buffer = BytesIO()
                buffer_size = 0
        
        # Upload any remaining data as the final part (can be less than 5MB)
        if buffer_size > 0:
            buffer.seek(0)
            part = s3_client.upload_part(
                Bucket=s3_bucket,
                Key=s3_key,
                PartNumber=part_number,
                UploadId=upload_id,
                Body=buffer.read()
            )
            
            parts.append({
                'PartNumber': part_number,
                'ETag': part['ETag']
            })
            
            print(f"Uploaded final part {part_number} ({buffer_size} bytes)")
        
        # Complete the multipart upload only if we have parts
        if parts:
            s3_client.complete_multipart_upload(
                Bucket=s3_bucket,
                Key=s3_key,
                UploadId=upload_id,
                MultipartUpload={'Parts': parts}
            )
            
            print(f"File uploaded successfully to s3://{s3_bucket}/{s3_key}")
        else:
            # No parts were uploaded, likely an empty file
            s3_client.abort_multipart_upload(
                Bucket=s3_bucket,
                Key=s3_key,
                UploadId=upload_id
            )
            
            # Upload an empty file instead
            s3_client.put_object(Bucket=s3_bucket, Key=s3_key, Body=b'')
            print(f"Empty file uploaded to s3://{s3_bucket}/{s3_key}")
            
        return f"s3://{s3_bucket}/{s3_key}"
        
    except Exception as e:
        # Abort the multipart upload if something goes wrong
        s3_client.abort_multipart_upload(
            Bucket=s3_bucket,
            Key=s3_key,
            UploadId=upload_id
        )
        print(f"Multipart upload aborted due to: {str(e)}")
        raise
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
assert API_AUTH_TOKEN, "API_AUTH_TOKEN must be set in environment variables"
api_client = APIClient(API_AUTH_TOKEN)
```


```python
# Download data for each data identifier
assert S3_BUCKET, "S3_BUCKET must be set in environment variables"
for data_id, data in DATA_IDENTIFIERS.items():
    # print(f"Downloading data for {data_id}")
    params = data["filters"]
    outputdir = data["outputdir"]
     
    # Get data manifest, this will contain all the files that can be downloaded
    # You can track the downloaded files to maintain ingestion state
    manifest = api_client.data.get_alldata_manifest(data_id)
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
        download_alldata_to_s3(
            file["url"],
            api_client.data.request_manager,
            s3_bucket=S3_BUCKET,
            s3_key_prefix=outputdir,

        )
        
    print(f"Downloaded all files for {data_id}")
```
