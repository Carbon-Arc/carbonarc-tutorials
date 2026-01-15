# Carbon Arc Tutorials

This repository contains code for [Carbon Arc](https://github.com/Carbon-Arc/carbonarc) tutorials.

## Setup

### 1. Install Dependencies

```bash
pip install carbonarc python-dotenv pandas boto3 pyarrow
```

### 2. Configure Environment Variables

Copy the example environment file and add your credentials:

```bash
cp .env.example .env
```

Edit `.env` with your values:

- **API_AUTH_TOKEN**: Get your API token from https://app.carbonarc.co/my/profile
- **API_BASE_URL**: Use `https://api.carbonarc.co` for production
- **AWS credentials**: Required for Bulk API access (see [Bulk Data Access](#bulk-data-access))

### 3. Run Notebooks

Open any notebook in Jupyter and run the cells. Each notebook loads credentials from `.env` automatically.

## What's in This Repository?

### `1-platform/`
Platform guides for getting started with Carbon Arc:
* **account.ipynb** - Account setup and API client initialization

### `2-explorer/`
Framework guides for building and using the Explorer:
* **build.ipynb** - Build and configure a framework

### `3-ontology/`
Ontology guides for understanding the data model:
* **entities.ipynb** - Explore entities in the ontology
* **insights.ipynb** - Explore insights in the ontology

### `4-data/`
Data guides for working with bulk data:
* **data_library.ipynb** - Explore the complete Data Library (55 datasets across 6 categories)
* **deep_dive_bulk.ipynb** - Deep dive into bulk data exploration
* **graph_data_local.ipynb** - Work with graph data locally
* **bulk_data_coverage.ipynb** - Analyze dataset coverage by reading parquet files from S3
* **bulk_data_coverage_athena.ipynb** - Analyze dataset coverage using AWS Athena for large datasets

#### Bulk Data Access

The `bulk_data_coverage*.ipynb` notebooks demonstrate how to analyze row-level bulk data directly from S3 buckets. This is for customers with **Bulk API** access to curated data buckets.

To purchase access to bulk datasets, contact sales at **sales@carbonarc.co**.