# Real-Time-User-Activity-Streaming-and-Analytics-Pipeline-Using-Kafka-Spark-MongoDB-and-Redshift

Real-time ETL pipeline streaming user activity data via Kafka, transforming it with Spark Structured Streaming. Stores raw data in MongoDB for fast lookups and loads transformed data into AWS Redshift for analytics. Demonstrates a scalable, end-to-end streaming solution for user behavior insights.

# Real-Time Data Engineering Project – Streaming ETL Pipeline

## 🚀 Project Overview  
This project implements a real-time streaming ETL pipeline to ingest, process, and store user activity data. Data is produced to Kafka, processed and transformed using Apache Spark Structured Streaming, raw data is stored in MongoDB, and aggregated data is loaded into AWS Redshift for analytics. This setup enables real-time insights into user behavior.

## 🛠️ Tech Stack  
- **Apache Kafka:** Distributed messaging system for data ingestion  
- **Apache Spark Structured Streaming:** Real-time data processing and transformations  
- **MongoDB:** NoSQL database for raw data storage and fast lookups  
- **AWS Redshift:** Data warehouse for analytics and reporting  
- **Python:** Kafka producer and pipeline orchestration  

## ✅ Architecture  
The pipeline follows the typical ETL pattern:  
- **Extract:** Simulated user activity JSON data streamed into Kafka topics  
- **Transform:** Spark streaming job reads Kafka, parses JSON, applies cleaning and enrichment  
- **Load:** Raw data written to MongoDB; transformed data loaded into Redshift  

### 📌 Data Flow Overview:  
1. **Data Ingestion:** Kafka producer sends user activity events to Kafka cluster.  
2. **Stream Processing:** Spark Structured Streaming consumes Kafka data, performs transformations.  
3. **Storage:**  
   - Raw data saved in MongoDB collections for quick querying.  
   - Aggregated and cleaned data loaded into Redshift tables for SQL analytics.  
4. **Analytics:** Query Redshift to derive insights such as user activity trends and device usage.

## 🔨 Implementation Steps  

### Step 1: Kafka Data Producer  
- Generate and send user activity JSON events continuously to Kafka topics.

### Step 2: Spark Streaming Processing  
- Read Kafka topic data using Spark Structured Streaming.  
- Parse JSON and apply transformations like filtering, enrichment.  
- Write raw data to MongoDB and transformed data to Redshift.

### Step 3: Data Storage and Querying  
- Verify raw events in MongoDB collections.  
- Use SQL queries on Redshift to analyze user behavior.

---

## 📸 Screenshots  

### 1. Architecture Diagram  
![Architecture](./screenshots/architecture_diagram.png)  

### 2. Kafka Producer Output  
![Kafka Producer Console](./screenshots/kafka_producer_output.png)  

### 3. Spark Streaming Logs  
![Spark Streaming Console](./screenshots/spark_streaming_output.png)  

### 4. MongoDB Sample Data  
![MongoDB Data](./screenshots/mongodb_sample.png)  

### 5. AWS Redshift Query Results  
![Redshift Analytics](./screenshots/redshift_query_results.png)  

---

## 🌱 Key Learnings  
- Building scalable real-time ETL pipelines with open-source tools  
- Handling JSON data ingestion and transformation in streaming workflows  
- Integrating NoSQL and data warehouse storage for different analytics needs  
- Writing performant Spark Structured Streaming jobs  
- Querying large datasets efficiently in AWS Redshift  

---

## Author  
Sachin.C  
Data Engineering Enthusiast  
[LinkedIn](https://www.linkedin.com/in/yourprofile) | [GitHub](https://github.com/yourusername)

