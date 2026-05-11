# 🐘 Big Data Stack for Mac Apple Silicon (M1 / M2 / M3 / M4)

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Apple%20Silicon%20ARM64-black?logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Required-2496ED?logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Hadoop-3.3.1-66CCFF?logo=apachehadoop&logoColor=white" />
  <img src="https://img.shields.io/badge/Hive-2.3.2-FDEE21?logo=apachehive&logoColor=black" />
  <img src="https://img.shields.io/badge/Spark-3.4-E25A1C?logo=apachespark&logoColor=white" />
  <img src="https://img.shields.io/badge/DBeaver-Required-372923?logo=dbeaver&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-green" />
</p>

<p align="center">
  A <strong>Cloudera-like Big Data environment</strong> that runs natively on Mac Apple Silicon.<br/>
  No VM. No other OS. Just Docker Desktop + DBeaver and one command.
</p>

---

## ✨ Why this project?

Cloudera QuickStart VM is x86-only and **cannot run on Mac M1/M2/M3/M4**. This project gives you the same experience — Hadoop, Hive, HBase, Spark — fully working on Apple Silicon via Docker + DBeaver.

---

## 📦 Stack

| Service | Version | Role | Web UI |
|---|---|---|---|
| **Hadoop HDFS** | 3.3.1 | Distributed file system | [localhost:9870](http://localhost:9870) |
| **YARN** | 3.3.1 | Resource & job manager | [localhost:8088](http://localhost:8088) |
| **Hive** | 2.3.2 | SQL on Hadoop | Via DBeaver |
| **HBase** | latest | NoSQL wide-column store | [localhost:16010](http://localhost:16010) |
| **Spark** | latest | Fast in-memory processing | [localhost:8080](http://localhost:8080) |
| **Zookeeper** | 3.9 | Coordination service | — |
| **Cloudbeaver** | latest | Web based SQL UI | [localhost:8090](http://localhost:8090) |
| **PostgreSQL** | 12 | Hive metastore backend | — |

---

## ⚡ Prerequisites

### 1. Docker Desktop (Required)
Download Apple Silicon version: [Docker Desktop for Mac](https://desktop.docker.com/mac/main/arm64/Docker.dmg)

After installing → **Settings → Resources** set:
- CPUs: **5**
- Memory: **10 GB**
- Swap: **2 GB**
- Click **Apply & Restart**

### 2. DBeaver Community (Required — your main SQL UI)
```bash
brew install --cask dbeaver-community
```
Or download from: https://dbeaver.io/download/

> DBeaver is your main interface — like Cloudera's Hue but works natively on Mac M4!

---

## 🚀 Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/Henzlo/bigdata-mac-apple-silicon.git
cd bigdata-mac-apple-silicon

# 2. Start Docker Desktop
open -a Docker

# 3. Start the stack
docker-compose up -d

# 4. Check all services are running
docker-compose ps
```

> **First run:** Takes 5–10 minutes to download images (~3–4 GB).
> After that, startup takes about 60–90 seconds.

---

## 🖥️ DBeaver Setup (Do this once)

### Connect to Hive

1. Open **DBeaver**
2. Click **"New Database Connection"** (+ icon)
3. Search **"Hive"** → Select **Apache Hive 2**
4. Fill in:
   - Host: `localhost`
   - Port: `10000`
   - Leave username/password blank
5. Click **"Test Connection"** → DBeaver auto-downloads the driver
6. Click **Finish** ✅

### Connect to HBase (via Phoenix)

1. Click **"New Database Connection"**
2. Search **"Phoenix"** → Select **Apache Phoenix**
3. Fill in:
   - Host: `localhost`
   - Port: `8765`
4. Click **"Test Connection"** → Click **Finish** ✅

### Connect to Spark SQL

1. Click **"New Database Connection"**
2. Search **"Hive"** → Select **Apache Hive 2**
3. Fill in:
   - Host: `localhost`
   - Port: `10000`
   - Database: `default`
4. Click **Finish** ✅

---

## 💡 Usage Examples

### Hive SQL

```sql
-- Create a database
CREATE DATABASE mydb;
USE mydb;

-- Create a table
CREATE TABLE employees (
  id     INT,
  name   STRING,
  dept   STRING,
  salary DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';

-- Insert data
INSERT INTO employees VALUES (1, 'Alice', 'Engineering', 95000);
INSERT INTO employees VALUES (2, 'Bob', 'Marketing', 75000);

-- Query
SELECT dept, AVG(salary) as avg_salary
FROM employees
GROUP BY dept;
```

---

### HDFS Commands

```bash
# List files
docker exec namenode hdfs dfs -ls /

# Create directory
docker exec namenode hdfs dfs -mkdir /mydata

# Upload a file from your Mac
docker cp localfile.csv namenode:/tmp/
docker exec namenode hdfs dfs -put /tmp/localfile.csv /mydata/

# Read a file
docker exec namenode hdfs dfs -cat /mydata/localfile.csv
```

---

### HBase Shell

```bash
docker exec -it hbase hbase shell
```

```ruby
# Create a table
create 'students', 'info', 'grades'

# Insert data
put 'students', 'student1', 'info:name', 'Alice'
put 'students', 'student1', 'grades:math', '95'

# Read a row
get 'students', 'student1'

# Scan all rows
scan 'students'
```

---

### PySpark

```bash
docker exec -it spark-master pyspark --master spark://spark-master:7077
```

```python
# Word count example
df = spark.read.text("hdfs://namenode:9000/mydata/file.txt")
from pyspark.sql.functions import explode, split, col
words = df.select(explode(split(col("value"), " ")).alias("word"))
words.groupBy("word").count().orderBy("count", ascending=False).show()
```

---

## 🔄 Daily Usage

### Every time you start your Mac:
```bash
cd ~/bigdata-mac-apple-silicon
open -a Docker          # Start Docker Desktop
docker-compose up -d    # Start the stack
```

### Then open DBeaver and start working! ✅

### When done for the day:
```bash
docker-compose down     # Stop stack (keeps all your data)
```

---

## 🛑 Managing the Stack

```bash
# Start
docker-compose up -d

# Stop (keeps all your data)
docker-compose down

# Stop and DELETE all data (fresh start)
docker-compose down -v

# Restart a single service
docker-compose restart hive-server

# View logs
docker-compose logs -f hive-server
docker-compose logs -f namenode
```

---

## 🔧 Troubleshooting

### Hive connection refused in DBeaver?
HiveServer2 takes 3-5 minutes to fully start. Wait and retry.
```bash
docker logs hive-server 2>&1 | tail -10
```

### Services not starting?
```bash
docker-compose logs -f namenode
docker-compose logs -f hive-metastore
```

### Complete reset (if everything is broken)
```bash
docker-compose down -v
docker system prune -f
docker-compose up -d
```

### Architecture warning `linux/amd64` on M4?
Normal and expected — Docker uses Rosetta 2 for amd64 images.

---

## 🗺️ Port Reference

| Port  | Service                     |
|-------|-----------------------------|
| 9870  | HDFS NameNode UI            |
| 8088  | YARN ResourceManager UI     |
| 8080  | Spark Master UI             |
| 8081  | Spark Worker UI             |
| 16010 | HBase Master UI             |
| 10000 | Hive JDBC (DBeaver)         |
| 8090  | Cloudbeaver Web UI          |
| 8188  | MapReduce History Server    |
| 9864  | HDFS DataNode UI            |
| 8042  | YARN NodeManager UI         |
| 9000  | HDFS RPC                    |
| 9083  | Hive Metastore Thrift       |
| 2181  | Zookeeper                   |
| 7077  | Spark Master RPC            |
| 9090  | HBase Thrift                |

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs welcome!

---

## 📄 License

[MIT](LICENSE) — free to use, modify, and share.

---

<p align="center">Made for Mac Apple Silicon users who miss Cloudera 🍎🐘</p>
