# Feature Store Freshness Benchmarks


In this benchmark, we measure the best-case, average-case, and worst-case performance of feature stores in going from 
raw event data in a message bus (Kafka, Polars, etc) to a stream processing feature pipeline computing (or recomputing) 
the updated or new feature value, and then writing that feature value to the online feature store. That is, the time 
stops when the computed feature becomes available in the online feature store for reading.

- [java-bytewax](https://github.com/featurestoreorg/featurestore-benchmarks/tree/main/fs-feature-freshness-benchmark/java-bytewax):
Feature Store Freshness using Bytewax event simulator and java application for fetching feature vectors


## Benchmarked Feature Stores

The current feature stores benchmarked here are:

 * Hopsworks Feature Store

Coming Soon
 * [AWS Sagemaker]
 * [GCP Vertex]


# How to run the Benchmarks

## Setup
Install the required python libraries
```console
pip install hopsworks
pip install bytewax==0.17.2
```

## Clone tutorials repository
```bash
git clone https://github.com/featurestoreorg/featurestore-benchmarks
cd ./featurestore-benchmarks/fs-feature-freshness-benchmark/java-bytewax
```

## Build the Java client
```console
mvn clean compile assembly:single
# Jar will be created as target/bytewaxlatencybenchmark-1.0-SNAPSHOT-jar-with-dependencies.jar
```

## Define environment variables
You need  to have Hopsworks cluster host address, hopsworks project name and
[api key](https://docs.hopsworks.ai/3.3/user_guides/projects/api_key/create_api_key/)

Once you have the above, define the following environment variables:

**Console 1: Define variables**
```console
export FEATURE_GROUP_NAME=clicks
export FEATURE_GROUP_VERSION=1
export HOPSWORKS_HOST=REPLACE_WITH_YOUR_HOPSWOKRKS_CLUSTER_HOST
export HOPSWORKS_API_KEY=REPLACE_WITH_YOUR_HOPSWORKS_API_KEY
export HOPSWOERKS_PROJECT_NAME=REPLACE_WITH_YOUR_HOPSWOERKS_PROJECT_NAME
```

## Create a feature group using the HSFS APIs.
Full documentation how to create feature group using HSFS APIs can be found [here](https://docs.hopsworks.ai/3.3/user_guides/fs/feature_group/create/).

**Console 1: Create the feature group**
```console
python3 ./bytewax_scripts/recreate_fg.py
```

## Start the benchmarking tool
To get necessary environment variables in Feature Store UI go to Storage Connectors -> FEATURE_STORENAME_USER_onlinefeaturestore. Then click to edit button and Select following variables:

**Console 2: Start the benchmarking tool**
```console
export DB_URL="REPLACE_WITH_YOUR_ONLINE_FEATURE_STORE_CONNECTOR_URL" # You might need to change the IP to an IP that you can reach from your benchmark machine
export USER="REPLACE_WITH_YOUR_ONLINE_FEATURE_STORE_CONNECTOR_USER"
export PASS="REPLACE_WITH_YOUR_ONLINE_FEATURE_STORE_CONNECTOR_PASSWORD"

java -jar bytewaxlatencybenchmark-1.0-SNAPSHOT-jar-with-dependencies.jar BATCH_SIZE START_ID ROUNDS
# BATCH_SIZE - Maximum number of ids to be fetched in one request. Default is 50.
# START_ID - Id of the first record to be fetched. Default is 1.
# ROUNDS - How many batches to fetch. Default is 100.
```

## Simulate click events and write to online feature store.
**Console 1: Start sending records**
```console
cd bytewax_scripts
export HOPSWORKS_HOST=REPLACE_WITH_YOUR_HOPSWOKRKS_CLUSTER_HOST
export HOPSWORKS_API_KEY=REPLACE_WITH_YOUR_HOPSWORKS_API_KEY
export HOPSWOERKS_PROJECT_NAME=REPLACE_WITH_YOUR_HOPSWOERKS_PROJECT_NAME
export FEATURE_GROUP_NAME=clicks
export FEATURE_GROUP_VERSION=1

python3 -m bytewax.run "click_events:get_flow('$FEATURE_GROUP_NAME', $FEATURE_GROUP_VERSION, '$HOPSWORKS_HOST', '$HOPSWOERKS_PROJECT_NAME', '$HOPSWORKS_API_KEY')"
```

# Benchmark Results
## Benchmark Setup
![cluster_specs](https://github.com/featurestoreorg/featurestore-benchmarks/assets/4143920/a1df206e-6ec5-46dc-8a89-0c79d401cb2e)
Hardware setup for feature stores used in benchmarks

## Results
**Hopsworks** supports [streaming ingestion to the feature store using Flink, Beam, Spark, and Bytewax](https://docs.hopsworks.ai/3.3/user_guides/fs/feature_group/create/#streaming-write-api)

![latency_graph](https://github.com/featurestoreorg/featurestore-benchmarks/assets/4143920/bdc575e4-39a3-4d48-a7da-0a6b9273ec8e)
![latencies](https://github.com/featurestoreorg/featurestore-benchmarks/assets/4143920/43f37954-64ab-457b-b347-e9ece5cf19f9)

**AWS Sagemaker** supports [streaming ingestion to the feature store](https://aws.amazon.com/blogs/machine-learning/using-streaming-ingestion-with-amazon-sagemaker-feature-store-to-make-ml-backed-decisions-in-near-real-time/) 
[Results coming soon]

**Legacy GCP Vertex** provided a [REST API for streaming ingestion to the feature store](https://cloud.google.com/vertex-ai/docs/featurestore/ingesting-stream). 
**New GCP Vertex**  [synchronizes offline tables in Big Query with an online table (either a low latency store or BigTable) periodically](https://cloud.google.com/vertex-ai/docs/featurestore/latest/sync-data)


**Databricks** [synchronizes offline tables with an online store periodically](https://docs.databricks.com/en/machine-learning/feature-store/publish-features.html#publish-streaming-features-to-an-online-store) but also supports [streaming ingestion](https://docs.databricks.com/en/machine-learning/feature-store/publish-features.html#publish-streaming-features-to-an-online-store). [See API](https://api-docs.databricks.com/python/feature-store/latest/feature_store.client.html). 
Databricks will be included here when they provide an online feature serving API that enables feature freshness to be measured. 

