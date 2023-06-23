# Datastream Analytics Pipeline 
This project aims to build a reliable and low-latency data replication and analytics pipeline using various GCP services: 
* Datastream for Change Data Capture (CDC) replication (more on this below)
* Cloud Storage for storing CDC events
* Pub/Sub for triggering downstream processes
* Dataflow for transforming and loading data into BigQuery
* BigQuery for final log and metadata info, as well as any final queries or analytics

# Change Data Capture (CDC) - Why Is This Important? 
* According to [Confluent](https://www.confluent.io/learn/change-data-capture/), CDC is a pattern that tracks all changes in a data source (databases, data warehouses, etc.), so they can be captured in destination systems. This pattern allows organizations to achieve data integrity and consistency across all systems and deployment environments.
* In short, this project will use CDC through various GCP services to achieve data integrity and consistency across all systems.

# Data Source 
* This project utilizes Oracle's [world_x database](https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysql-shell-tutorial-javascript-download.html), which is a comprehensive collection of data about countries, cities, languages, and other geographical information. 

# Architecture Diagram 
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Architecture%20Diagram.png)
_Diagram made using [Excalidraw](https://excalidraw.com/)_

# Process  
**Preparation** 
* Make sure you have access to GCP
* Setup a local MySQL instance
* Run this repo's shell script to download and unzip world_x database from Oracle's website

  `bash get_db.sh`


**Phase 1 - Connect Cloud SQL Database to Local MySQL Instance**
* Create Cloud SQL database (MySQL, db name: database1) and connect it to your local MySQL instance
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/1.%20Cloud%20SQL%20Connection/2.%20Cloud%20SQL%20Instance%20edited.png)
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/1.%20Cloud%20SQL%20Connection/4.%20Connect%20to%20Cloud%20edited.png)

* In MySQL, run SQL script to load world_x database into local MySQL instance
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/1.%20Cloud%20SQL%20Connection/5.%20Run%20SQL%20Script%20edited.png)

**Phase 2.1 - Stream Data From Cloud SQL To Cloud Storage (CDC)**
* Setup Cloud Storage Bucket
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/2.%20Cloud%20Storage%20%26%20Datastream/1.%20Setup%20Cloud%20Storage%20Bucket.png)
* Define new Datastream stream (database1-to-cloud-storage), specify region, source type (MySQL), and destination (Cloud Storage)
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/2.%20Cloud%20Storage%20%26%20Datastream/2.%20Define%20Stream%20Details.png)
* Stream also requires defining connection settings and selecting database objects (world_x database) to stream to or exclude from destination (Cloud Storage)
* Define destination (Cloud Storage bucket) and file format (Avro)
Stream data from Cloud SQL into Cloud Storage using Datastream change data capture.
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/2.%20Cloud%20Storage%20%26%20Datastream/8.%20Stream%20Ready.png)

**Phase 2.2 - Pub/Sub Notification & Subscription** 
* Configure Pub/Sub topic for destination (Cloud Storage bucket)
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/3.%20PubSub/1.%20PubSub%20Topic.png)
* Create subscription to notify Dataflow once CDC is complete
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/3.%20PubSub/2.%20PubSub%20Subscription.png)


**Phase 3 - Pipeline To Move Data From Cloud Storage Into BigQuery** 
* In BigQuery, create a new dataset called Analytics_Stream
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/4.%20Dataflow%20%26%20BigQuery/1.%20BQ%20-%20Create%20Dataset%20Analytics%20Stream.png)
* In Dataflow, create job from template (Datastream to BigQuery)
* In the job's required parameters, select file location in Cloud Storage, Pub/Sub subscription on the bucket, output file format (Avro), and name of the dataset to contain staging tables (analytics_stream), and dead letter queue directory in bucket
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/4.%20Dataflow%20%26%20BigQuery/2.%20DataFlow%20Job%20Template.png)
* Run job in Dataflow deploy data pipeline and pipeline graph
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/4.%20Dataflow%20%26%20BigQuery/3.%20DataFlow%20Pipeline%20Graph.png)
* Dataflow writes the data to a data sink (BigQuery)
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/4.%20Dataflow%20%26%20BigQuery/5.%20Log%20Tables%20in%20Analytics%20Stream.png)
* Also streamed into Cloud Storage
* ![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Images/4.%20Dataflow%20%26%20BigQuery/7.%20Streamed%20into%20Cloud%20Storage.png)

# Final Logs & Metadata in BigQuery 

![city_log](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Logs/schemas_and_previews/city_log_preview.png) 
![country_log](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Logs/schemas_and_previews/country_log_preview.png)
![countryinfo_log](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Logs/schemas_and_previews/countryinfo_preview.png) 
![countrylanguage](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Logs/schemas_and_previews/countrylanguage_preview.png)

# Hiccups  
A 503 Service Unavailable error caused my first 6 Dataflow pipeline jobs to fail. 

I did check the availability of the service in my selected region, us-west 2.

[Google Cloud](https://cloud.google.com/about/locations) officially states that Dataflow is indeed available in us-west 2. 

However, my Dataflow job succeeded on the 7th try only when I changed the region to us-central 1.

So pro tip when using Dataflow - change the region if the job fails because of a 503 Service Unavailable Error.


