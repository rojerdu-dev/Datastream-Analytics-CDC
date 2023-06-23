# Datastream Analytics Pipeline 
This project aims to build a reliable and low-latency data replication and analytics pipeline using various GCP services: 
* Datastream for Change Data Capture (CDC) replication
* Cloud Storage for storing CDC events
* Pub/Sub for triggering downstream processes
* Dataflow for transforming and loading data into BigQuery
* BigQuery for analytics and queries

# Data Source 
* This project utilizes Oracle's [world_x database](https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysql-shell-tutorial-javascript-download.html), which is a comprehensive colelction of data about countries, cities, languages, and other geographical information. 

# Architecture Diagram 
![Alt text](https://github.com/rojerdu-dev/Datastream-Analytics-CDC/blob/main/Project%20Diagram.png)
_Diagram made using [Excalidraw](https://excalidraw.com/)_

# 3 Phases 
* Phase 1 - Create Cloud SQL database (MySQL) and connect it to your local MySQL instance
* Download world_x database from Oracle's website and run SQL script to load database into local MySQL instance

* Phase 2.1 - Stream data from Cloud SQL into Cloud Storage using Datastream change data capture
* Define Datastream stream (database1-to-cloud-storage) and specify region, source type (MySQL), and destination (Cloud Storage).
* The stream also requires defining connection settings, select database objects (world_x database) to stream to or exclude from the destination (Cloud Storage)
* Define destination (Cloud Storage bucket) and file format (Avro)

* Phase 2.2 - Configure Pub/Sub Notifications for Cloud Storage bucket (destination in previous phase)
* Create Pub/Sub topic for destination and then create subscription to notify Dataflow once CDC is complete

* Phase 3 - Build Dataflow pipeline to move data from Cloud Storage into BigQuery
* Use Dataflow job template (Datastream to BigQuery) to create and deploy data pipeline 
* Dataflow writes the data to a data sink (BigQuery)

# Hiccups 
My first 6 pipelines in Dataflow failed because of a 503 Service Unavailable error. 
Apparently, this service (Dataflow) was unavailable in my region (us-west2), which I was hardly able to believe. 
My 7th attempt at running the Dataflow job succeeded when I changed the job's region to us-central 1.


