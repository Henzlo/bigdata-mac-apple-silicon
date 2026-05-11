# 📖 Usage Guide — Big Data Stack for Mac M4

Complete step-by-step guide for using every service in the stack.

---

## 📋 Table of Contents
1. [Start & Stop the Stack](#start--stop-the-stack)
2. [Hive SQL — via DBeaver](#hive-sql)
3. [HDFS — File Storage](#hdfs)
4. [HBase — NoSQL Database](#hbase)
5. [Spark — Data Processing](#spark)
6. [Zeppelin — Notebooks](#zeppelin)
7. [Web UIs](#web-uis)
8. [Troubleshooting](#troubleshooting)

---

## 🚀 Start & Stop the Stack

### Start
```bash
cd ~/bigdata-mac-apple-silicon
open -a Docker
docker-compose up -d
```

### Check status
```bash
docker-compose ps
```
All 14 services should show **Up**.

### Stop (keeps your data)
```bash
docker-compose down
```

### Fresh start (deletes all data)
```bash
docker-compose down -v
docker-compose up -d
```

---

## 🐝 Hive SQL

Hive lets you run SQL queries on data stored in HDFS.

### Step 1 — Open DBeaver
Open the DBeaver app on your Mac.

### Step 2 — Connect to Hive (first time only)
1. Click **+** → New Database Connection
2. Search **"Hive"** → Select **Apache Hive 2**
3. Host: `localhost`, Port: `10000`
4. Click **Test Connection** → **Finish**

### Step 3 — Open SQL Editor
1. Expand the Hive connection in the left panel
2. Right click → **SQL Editor** → **New SQL Script**

### Step 4 — Run Queries

```sql
-- Create a database
CREATE DATABASE college;
USE college;

-- Create a table
CREATE TABLE students (
  id     INT,
  name   STRING,
  age    INT,
  marks  DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';

-- Insert data
INSERT INTO students VALUES (1, 'Alice', 20, 95.5);
INSERT INTO students VALUES (2, 'Bob', 21, 88.0);
INSERT INTO students VALUES (3, 'Charlie', 22, 76.5);

-- Query all data
SELECT * FROM students;

-- Average marks
SELECT AVG(marks) as average FROM students;

-- Filter students with marks above 80
SELECT name, marks FROM students WHERE marks > 80;

-- Group by and count
SELECT age, COUNT(*) as total FROM students GROUP BY age;
```

Press **Ctrl+Enter** to run a query.

---

## 📁 HDFS — File Storage

HDFS (Hadoop Distributed File System) stores your big data files.

### Basic Commands

```bash
# List root directory
docker exec namenode hdfs dfs -ls /

# Create a directory
docker exec namenode hdfs dfs -mkdir /mydata
docker exec namenode hdfs dfs -mkdir /mydata/csv

# Upload a file from your Mac to HDFS
docker cp /path/to/yourfile.csv namenode:/tmp/
docker exec namenode hdfs dfs -put /tmp/yourfile.csv /mydata/csv/

# List files in a directory
docker exec namenode hdfs dfs -ls /mydata/csv/

# Read a file
docker exec namenode hdfs dfs -cat /mydata/csv/yourfile.csv

# Download file from HDFS to Mac
docker exec namenode hdfs dfs -get /mydata/csv/yourfile.csv /tmp/
docker cp namenode:/tmp/yourfile.csv ~/Downloads/

# Delete a file
docker exec namenode hdfs dfs -rm /mydata/csv/yourfile.csv

# Delete a directory
docker exec namenode hdfs dfs -rm -r /mydata

# Check disk usage
docker exec namenode hdfs dfs -du -h /
```

### Using Web UI
Open **http://localhost:9870** → Click **Utilities** → **Browse the file system**

---

## 🗄️ HBase — NoSQL Database

HBase is a NoSQL wide-column store — great for real-time read/write access to large datasets.

### Open HBase Shell
```bash
docker exec -it hbase hbase shell
```

### Basic Commands

```ruby
# Create a table with column families
create 'students', 'info', 'grades'

# Insert data (put 'table', 'rowkey', 'family:column', 'value')
put 'students', 'row1', 'info:name', 'Alice'
put 'students', 'row1', 'info:age', '20'
put 'students', 'row1', 'grades:math', '95'
put 'students', 'row1', 'grades:science', '88'

put 'students', 'row2', 'info:name', 'Bob'
put 'students', 'row2', 'info:age', '21'
put 'students', 'row2', 'grades:math', '78'

# Get a specific row
get 'students', 'row1'

# Scan all rows
scan 'students'

# Scan specific column
scan 'students', {COLUMNS => 'info:name'}

# Count rows
count 'students'

# Delete a specific cell
delete 'students', 'row1', 'grades:math'

# Delete an entire row
deleteall 'students', 'row1'

# List all tables
list

# Describe a table
describe 'students'

# Drop a table
disable 'students'
drop 'students'

# Exit shell
exit
```

### Using Web UI
Open **http://localhost:16010** → HBase Master UI

---

## ⚡ Spark — Data Processing

Spark is a fast in-memory data processing engine.

### PySpark Shell (Python)
```bash
docker exec -it spark-master pyspark --master spark://spark-master:7077
```

### Basic PySpark Examples

```python
# Simple RDD example
data = [1, 2, 3, 4, 5]
rdd = sc.parallelize(data)
print(rdd.collect())
print(rdd.sum())
print(rdd.mean())

# Word count
text = sc.parallelize(["hello world", "hello spark", "world of spark"])
words = text.flatMap(lambda line: line.split(" "))
word_count = words.map(lambda word: (word, 1)).reduceByKey(lambda a, b: a + b)
print(word_count.collect())

# DataFrame example
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("test").getOrCreate()

data = [("Alice", 95), ("Bob", 88), ("Charlie", 76)]
df = spark.createDataFrame(data, ["name", "marks"])

# Show data
df.show()

# Filter
df.filter(df.marks > 80).show()

# Average
df.agg({"marks": "avg"}).show()

# Sort
df.orderBy("marks", ascending=False).show()

# Read from HDFS
df = spark.read.csv("hdfs://namenode:9000/mydata/yourfile.csv", header=True)
df.show()
```

### Scala Spark Shell
```bash
docker exec -it spark-master spark-shell --master spark://spark-master:7077
```

```scala
// DataFrame example
val data = Seq(("Alice", 95), ("Bob", 88), ("Charlie", 76))
val df = spark.createDataFrame(data).toDF("name", "marks")
df.show()
df.filter($"marks" > 80).show()
```

### Spark Submit (Run a Script)
```bash
docker exec spark-master spark-submit \
  --master spark://spark-master:7077 \
  /path/to/your/script.py
```

### Using Web UI
Open **http://localhost:8080** → Spark Master UI

---

## 📓 Zeppelin — Notebooks

Zeppelin is a web-based notebook UI — similar to Jupyter but for Big Data.

### Open Zeppelin
```
http://localhost:9091
```

### Create a New Notebook
1. Click **"Create new note"**
2. Give it a name e.g. `MyFirstNotebook`
3. Click **Create**

### Run Hive SQL in Zeppelin

```sql
%jdbc(hive)
SHOW DATABASES;
```

```sql
%jdbc(hive)
USE college;
SELECT * FROM students;
```

### Run PySpark in Zeppelin

```python
%pyspark
data = [("Alice", 95), ("Bob", 88), ("Charlie", 76)]
df = spark.createDataFrame(data, ["name", "marks"])
df.show()
```

### Run Scala Spark in Zeppelin

```scala
%spark
val data = Seq(("Alice", 95), ("Bob", 88))
val df = spark.createDataFrame(data).toDF("name", "marks")
df.show()
```

### Run Shell Commands in Zeppelin

```bash
%sh
hdfs dfs -ls /
docker ps
```

### Run Markdown in Zeppelin

```
%md
# My Big Data Analysis
This notebook analyzes student data using **Hive** and **Spark**.
```

---

## 🌐 Web UIs

| Service | URL | What you can see |
|---|---|---|
| **HDFS NameNode** | http://localhost:9870 | File system, storage, DataNodes |
| **YARN** | http://localhost:8088 | Running jobs, resource usage |
| **Spark Master** | http://localhost:8080 | Spark workers, applications |
| **Spark Worker** | http://localhost:8081 | Worker details, executors |
| **HBase** | http://localhost:16010 | Tables, regions, servers |
| **Zeppelin** | http://localhost:9091 | Notebooks, interpreters |
| **Cloudbeaver** | http://localhost:8090 | Web SQL editor |
| **History Server** | http://localhost:8188 | Completed MapReduce jobs |
| **NodeManager** | http://localhost:8042 | Node resource details |

---

## 🔧 Troubleshooting

### Hive not connecting in DBeaver?
```bash
# Check if hive-server is ready (takes 3-5 minutes)
docker logs hive-server 2>&1 | tail -10
```
Wait until you see `Started ThriftBinaryCLIService on port 10000`

### Service crashed or not responding?
```bash
# Restart a specific service
docker-compose restart hive-server
docker-compose restart zeppelin
docker-compose restart namenode
```

### Check logs of any service
```bash
docker logs hive-server 2>&1 | tail -20
docker logs namenode 2>&1 | tail -20
docker logs zeppelin 2>&1 | tail -20
```

### Out of memory errors?
- Open Docker Desktop → Settings → Resources
- Increase Memory to **12 GB**
- Click Apply & Restart

### Complete reset (if everything is broken)
```bash
docker-compose down -v
docker system prune -f
docker-compose up -d
```

### Check how much disk Docker is using
```bash
docker system df
```
