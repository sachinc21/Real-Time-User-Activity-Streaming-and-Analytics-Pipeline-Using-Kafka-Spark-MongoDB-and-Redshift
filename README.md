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

## 🔨 Pipeline Deep Dive 

### 1: Kafka Data Producer  
Simulate and send real-time events to Kafka.
**Code**: `kafka_producer.py`
```python
# Key Features:
# - Circular reading for infinite streaming
# - Error handling and graceful shutdown

import json
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers='localhost:9092',
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

with open("UserActivity.json") as f:
    records = json.load(f)  # Sample data: 1000+ user activities

while True:
    for record in records:
        try:
            producer.send('user_activity_topic', record)
            print(f"Sent: {record['email'][:5]}...")  # Truncate PII
            time.sleep(0.5)  # Simulate real-time
        except Exception as e:
            print(f"Error: {e}")
            break
```

**Producer**
![Screenshot from 2025-05-21 13-02-44](https://github.com/user-attachments/assets/3653b0eb-6ec8-413c-b989-96b0927b785f)

**Consumer**
![image](https://github.com/user-attachments/assets/8ad98d4e-68b2-456c-a459-7de9e0560e6b)

### 2: Spark Structured Streaming ETL  

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, from_json, to_timestamp, current_timestamp, window, hour
from pyspark.sql.types import StructType, StructField, StringType

from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("KafkaToMongoAndRedshift") \
    .config("spark.jars.packages",
            "org.mongodb.spark:mongo-spark-connector_2.12:3.0.1,"
            "org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0") \
    .config("spark.jars", "/home/sachin/Downloads/redshift-jdbc42-2.1.0.32/redshift-jdbc42-2.1.0.32.jar") \
    .config("spark.mongodb.write.connection.uri", "mongodb://127.0.0.1/") \
    .getOrCreate()

# Kafka stream
kafka_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "user_activity_topic") \
    .option("startingOffsets", "earliest") \
    .load()
    
# Define schema for incoming Kafka messages
schema = StructType([
    StructField("id", StringType(), True),
    StructField("activity", StringType(), True),
    StructField("timestamp", StringType(), True),  
    StructField("email", StringType(), True),
    StructField("gender", StringType(), True),
    StructField("device", StringType(), True),
    StructField("user_ip", StringType(), True),
    StructField("session_id", StringType(), True),
    StructField("user_agent", StringType(), True),
    StructField("location", StringType(), True)
])

# Parse JSON value
parsed_df = kafka_df.selectExpr("CAST(value AS STRING)") \
    .withColumn("parsed", from_json(col("value"), schema)) \
    .select("parsed.*") \
    .withColumn("activity_time", to_timestamp(col("timestamp"), "yyyy-MM-dd HH:mm:ss")) \
    .withColumn("processing_time", current_timestamp()) \
    .filter(col("activity").isNotNull() & (col("activity") != ""))

# Convert 'timestamp' string to proper timestamp format
parsed_df = parsed_df.withColumn("activity_time", to_timestamp(col("timestamp"), "yyyy-MM-dd HH:mm:ss"))

# Add a new column for when Spark processes the event
parsed_df = parsed_df.withColumn("processing_time", current_timestamp())

# Filter out rows with null or empty activity
parsed_df = parsed_df.filter(col("activity").isNotNull() & (col("activity") != ""))

# Normalize email addresses for consistent grouping
from pyspark.sql.functions import col, lower, trim

parsed_df = parsed_df.withColumn("email", lower(trim(col("email"))))

# Flag local or empty IPs for debugging or filtering
parsed_df = parsed_df.withColumn(
    "is_valid_ip",
    (col("user_ip").isNotNull()) & (~col("user_ip").like("127.%")) & (col("user_ip") != "")
)


```
### 3: MongoDB sink
```python
from pymongo import MongoClient
from pyspark.sql.streaming import StreamingQuery
from pyspark.sql import Row

# MongoDB writer for raw data
class MongoDBForeachWriter:
    def __init__(self, db_name, collection_name):
        self.client = None
        self.db_name = db_name
        self.collection_name = collection_name

    def open(self, partition_id, epoch_id):
        """Establish MongoDB connection for each partition"""
        self.client = MongoClient("mongodb://localhost:27017/")
        return True

    def process(self, row: Row):
        """Insert row into MongoDB"""
        data = row.asDict()
        print(f"Inserting row into MongoDB: {data}")
        self.client[self.db_name][self.collection_name].insert_one(data)

    def close(self, error):
        """Close MongoDB connection"""
        if self.client:
            self.client.close()

# Await termination
mongo_query.awaitTermination()
```

### 4: Redshift sink
```python


import os

# Redshift connection details
redshift_url = "jdbc:redshift://user-activity.876820567780.ap-southeast-2.redshift-serverless.amazonaws.com:5439/dev"
redshift_properties = {
    "user": os.environ.get("REDSHIFT_USER"),
    "password": os.environ.get("REDSHIFT_PASSWORD"),
    "driver": "com.amazon.redshift.jdbc42.Driver"
}

# Write to Redshift using foreachBatch
def write_to_redshift(batch_df, batch_id):
    batch_df.write \
        .jdbc(url=redshift_url,
              table=redshift_table,
              mode="append",
              properties=redshift_properties)

redshift_query = parsed_df.writeStream \
    .foreachBatch(write_to_redshift) \
    .outputMode("append") \
    .option("checkpointLocation", "/tmp/portfolio_checkpoints/user_activity_redshift") \
    .start()

#Await termination
redshift_query.awaitTermination()
```
