# Real-Time Data Engineering Project – Streaming ETL Pipeline

## 🚀 Project Overview  
This project implements a real-time streaming ETL pipeline to ingest, process, and store user activity data. Data is produced to Kafka, processed and transformed using Apache Spark Structured Streaming, and the cleaned data is written into both MongoDB and AWS Redshift for analytics and storage.

## 🛠️ Tech Stack  
- **Apache Kafka:** Distributed messaging system for ingesting user activity events  
- **Apache Spark Structured Streaming:** Real-time processing and transformation  
- **MongoDB:** NoSQL database for storing structured and semi-structured data  
- **AWS Redshift:** Data warehouse for analytical queries and reporting  
- **Python:** Used for the Kafka producer and Spark streaming logic  


## ✅ Architecture  

### 📌 Data Flow:
1. **Kafka Producer:** Sends real-time user activity JSON events to the Kafka topic `user_activity`.
2. **Spark Streaming:** Reads messages from Kafka, parses and transforms the data.
3. **Data Storage:**  
   - Transformed data is written to both **MongoDB** and **Redshift**.

---

## 🔨 Implementation Steps  

### Step 1: Kafka Data Producer  
Simulate and send real-time events to Kafka.

### Step 2: Spark Structured Streaming ETL  

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, TimestampType

# Initialize SparkSession
spark = SparkSession.builder \
    .appName("StreamingETLPipeline") \
    .getOrCreate()

# Define schema for incoming JSON
schema = StructType([
    StructField("user_id", StringType()),
    StructField("activity_type", StringType()),
    StructField("device", StringType()),
    StructField("timestamp", TimestampType())
])

# Read data from Kafka
kafka_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "user_activity") \
    .option("startingOffsets", "latest") \
    .load()

# Parse JSON from Kafka message value
parsed_df = kafka_df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select("data.*")

# Example transformation: keep only login events
transformed_df = parsed_df.filter(col("activity_type") == "login")

# Write transformed data to MongoDB
mongo_query = transformed_df.writeStream \
    .format("mongodb") \
    .option("uri", "mongodb://localhost:27017/streamingdb.login_events") \
    .outputMode("append") \
    .start()

# Write transformed data to Redshift
redshift_query = transformed_df.writeStream \
    .format("jdbc") \
    .option("url", "jdbc:redshift://your-redshift-cluster-url:5439/dev") \
    .option("dbtable", "public.login_events") \
    .option("user", "your_username") \
    .option("password", "your_password") \
    .outputMode("append") \
    .start()

# Await termination
mongo_query.awaitTermination()
redshift_query.awaitTermination()
