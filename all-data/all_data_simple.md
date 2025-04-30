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
API_AUTH_TOKEN=os.getenv("API_AUTH_TOKEN", None)
```


```python
# Create API Client
client = APIClient(API_AUTH_TOKEN)
```


```python
## Get insights data identifiers
data_identifiers = client.get_alldata_data_identifiers()
print("Data Identifiers:")
for data_id in data_identifiers["data"]:
    print(data_id["data_identifier"], ": ", data_id["description"])
```

    Data Identifiers:
    card_us_detail_data :  Card - US Detailed Panel Row Level Data
    pos_instore_online_data :  POS Transaction Bulk Data
    permit_data :  Permit Row Level Data
    medicare_claims_data :  Medicare Claims Data
    card_us_can_general_data :  Card - US and CAN General Panel Row Level Data
    clickstream_data :  Clickstream Data
    trade_export_data :  Trade Export Data
    trade_import_data :  Trade Import Data
    card_eu_data :  Card - EU Panel Row Level Data
    pos_cse_data :  POS Corner Store Transaction Data
    pos_skupos_data :  POS Convenience Store Transaction Data
    ecommerce_metrics_data :  Ecommerce Metrics Row Level Data
    vehicle_registration_data :  Vehicle Registration Row Level Data



```python
## Getting all manifests files for a given data identifier
data_identifier = "vehicle_registration_data"
manifests = client.get_alldata_manifest(data_identifier)
print(f"\nManifest for {data_identifier}:")
for manifest in manifests["files"]:
    print(manifest)
```

    
    Manifest for vehicle_registration_data:
    {'url': 'https://platform.carbonarc.co/api/v2/data/alldata/vehicle_registration_data/part-00000-a92189b3-b568-4068-9e04-0a34f59a3a88?drop_partition_id=1744757997', 'format': 'parquet', 'records': 4471504, 'size_bytes': 31863469, 'modification_time': '2025-04-15T23:04:44'}
    {'url': 'https://platform.carbonarc.co/api/v2/data/alldata/vehicle_registration_data/part-00000-b571f9bb-10c0-4716-87ce-0ea5a58e20af?drop_partition_id=1740512141', 'format': 'parquet', 'records': 55103110, 'size_bytes': 533682045, 'modification_time': '2025-03-12T22:57:36'}
    {'url': 'https://platform.carbonarc.co/api/v2/data/alldata/vehicle_registration_data/part-00000-d2ef64bc-680d-4378-83d6-cb146d58005e?drop_partition_id=1740512141', 'format': 'parquet', 'records': 49666721, 'size_bytes': 477762416, 'modification_time': '2025-03-12T22:57:06'}
    {'url': 'https://platform.carbonarc.co/api/v2/data/alldata/vehicle_registration_data/part-00000-69ef360f-356a-425d-91ad-eeddd9bba4d9?drop_partition_id=1741819689', 'format': 'parquet', 'records': 7406839, 'size_bytes': 54818210, 'modification_time': '2025-03-12T22:54:24'}



```python
# Download the file from manifest url
manifest_file_url="https://platform.carbonarc.co/api/v2/data/alldata/vehicle_registration_data/part-00000-a92189b3-b568-4068-9e04-0a34f59a3a88?drop_partition_id=1744757997"
output_dir="./output/alldata/vehicle_registration_data"
output_file_path = os.path.join(output_dir, "part-00001.parquet")

os.makedirs(output_dir, exist_ok=True)

print(f"Downloading manifest file to: {output_file_path}")
client.download_alldata_to_file(manifest_file_url, output_file_path)

```

    Downloading manifest file to: ./output/alldata/vehicle_registration_data/part-00001.parquet

