# Global Language Rankings Data Pipeline 
* This project's purpose is to stream data about the world (city, country, languages, etc) for analytics to show the most widely spoken languages in the world.
* The primary data source is Oracle's [world_x]([https://pages.github.com/](https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysql-shell-tutorial-javascript-download.html)https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/mysql-shell-tutorial-javascript-download.html) Database, which contains structured data related to countries, cities, customers, products, and orders for educational and demonstration purposes.

# Architecture Diagram 
![Alt text](https://github.com/rojerdu-dev/Global-Language-Rankings/blob/main/GCP%20Proj1%20-%20Architecture%20Diagram.png?raw=true)
_Diagram made using [Excalidraw](https://excalidraw.com/)_

Technical Goals 
1. Create Cloud SQL database and connect to local MySQL instance
2. Load world_x database into local MySQL (now connected to Cloud SQL)
3. Stream data from Cloud SQL to Cloud Storage with Datastream Change Data Capture
4. Pub/Sub notifications for Cloud Storage bucket
5. Create streaming pipeline with Dataflow
6. Final analytics in BigQuery

# Visualization 
![Alt text](https://github.com/rojerdu-dev/Global-Language-Rankings/blob/main/visualization-1.png)
