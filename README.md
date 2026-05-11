# 🐘 Big Data Stack for Mac Apple Silicon (M1 / M2 / M3 / M4)

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Apple%20Silicon%20ARM64-black?logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Required-2496ED?logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Hadoop-3.3.1-66CCFF?logo=apachehadoop&logoColor=white" />
  <img src="https://img.shields.io/badge/Hive-2.3.2-FDEE21?logo=apachehive&logoColor=black" />
  <img src="https://img.shields.io/badge/Spark-latest-E25A1C?logo=apachespark&logoColor=white" />
  <img src="https://img.shields.io/badge/DBeaver-Required-372923?logo=dbeaver&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-green" />
</p>

<p align="center">
  A <strong>Cloudera-like Big Data environment</strong> that runs natively on Mac Apple Silicon.<br/>
  No VM. No other OS. Just Docker Desktop + DBeaver and one command.
</p>

---

## ✨ Why this project?

Cloudera QuickStart VM is x86-only and **cannot run on Mac M1/M2/M3/M4**. This project gives you the same experience — Hadoop, Hive, HBase, Spark, and Zeppelin notebooks — fully working on Apple Silicon via Docker + DBeaver.

---

## 📦 Stack

| Service | Version | Role | Access |
|---|---|---|---|
| **Hadoop HDFS** | 3.3.1 | Distributed file system | [localhost:9870](http://localhost:9870) |
| **YARN** | 3.3.1 | Resource & job manager | [localhost:8088](http://localhost:8088) |
| **Hive** | 2.3.2 | SQL on Hadoop | Via DBeaver |
| **HBase** | latest | NoSQL wide-column store | [localhost:16010](http://localhost:16010) |
| **Spark** | latest | Fast in-memory processing | [localhost:8080](http://localhost:8080) |
| **Zeppelin** | 0.11.0 | Notebook UI (Hue alternative) | [localhost:9091](http://localhost:9091) |
| **Cloudbeaver** | latest | Web based SQL UI | [localhost:8090](http://localhost:8090) |
| **Zookeeper** | 3.9 | Coordination service | — |
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

### Connect to HBase
1. Click **"New Database Connection"**
2. Search **"Phoenix"** → Select **Apache Phoenix**
3. Fill in:
   - Host: `localhost`
   - Port: `8765`
4. Click **Finish** ✅

---

## 📓 Zeppelin Notebooks (Hue Alternative)

Open: **http://localhost:9091**

Zeppelin gives you:
- 📝 Interactive notebooks (like Jupyter)
- 🐝 Hive SQL interpreter
- ⚡ Spark interpreter
- 📊 Built-in charts and visualizations

### Connect Zeppelin to Hive
1. Go to **http://localhost:9091**
2. Click top right menu → **Interpreter**
3. Search **"jdbc"**
4. Set:
   - `default.url` → `jdbc:hive2://hive-server:10000`
   - `default.driver` → `org.apache.hive.jdbc.HiveDriver`
5. Click **Save** ✅

### Connect Zeppelin to Spark
1. Click top right menu → **Interpreter**
2. Search **"spark"**
3. Set:
   - `master` → `spark://spark-master:7077`
4. Click **Save** ✅

---

## 💡 Usage Examples

### Hive SQL (in DBeaver or Zeppelin)
```sql
CREATE DATABASE mydb;
USE mydb;

CREATE TABLE employees (
  id     INT,
  name   STRING,
  dept   STRING,
  salary DOUBLE
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';

INSERT INTO employees VALUES (1, 'Alice', 'Engineering', 95000);
INSERT INTO employees VALUES (2, 'Bob', 'Marketing', 75000);

SELECT dept, AVG(salary) as avg_salary
FROM employees
GROUP BY dept;
```

### HDFS Commands
```bash
# List files
docker exec namenode hdfs dfs -ls /

# Create directory
docker exec namenode hdfs dfs -mkdir /mydata

# Upload a file
docker cp localfile.csv namenode:/tmp/
docker exec namenode hdfs dfs -put /tmp/localfile.csv /mydata/
```

### HBase Shell
```bash
docker exec -it hbase hbase shell
```
```ruby
create 'students', 'info', 'grades'
put 'students', 'student1', 'info:name', 'Alice'
put 'students', 'student1', 'grades:math', '95'
get 'students', 'student1'
scan 'students'
```

### PySpark
```bash
docker exec -it spark-master pyspark --master spark://spark-master:7077
```
```python
df = spark.read.text("hdfs://namenode:9000/mydata/file.txt")
from pyspark.sql.functions import explode, split, col
words = df.select(explode(split(col("value"), " ")).alias("word"))
words.groupBy("word").count().orderBy("count", ascending=False).show()
```

---

## 🔄 Daily Usage

```bash
# Every time you start your Mac:
cd ~/bigdata-mac-apple-silicon
open -a Docker
docker-compose up -d

# When done:
docker-compose down
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

### Zeppelin not loading?
```bash
docker logs zeppelin 2>&1 | tail -10
```

### Complete reset
```bash
docker-compose down -v
docker system prune -f
docker-compose up -d
```

### Architecture warning `linux/amd64` on M4?
Normal — Docker uses Rosetta 2 for amd64 images. They work fine.

---

## 🗺️ Port Reference

| Port  | Service                     |
|-------|-----------------------------|
| 9870  | HDFS NameNode UI            |
| 8088  | YARN ResourceManager UI     |
| 8080  | Spark Master UI             |
| 8081  | Spark Worker UI             |
| 9091  | **Zeppelin Notebook UI**    |
| 8090  | Cloudbeaver Web UI          |
| 16010 | HBase Master UI             |
| 10000 | Hive JDBC (DBeaver)         |
| 8188  | MapReduce History Server    |
| 9864  | HDFS DataNode UI            |
| 8042  | YARN NodeManager UI         |
| 9000  | HDFS RPC                    |
| 9083  | Hive Metastore Thrift       |
| 2181  | Zookeeper                   |
| 7077  | Spark Master RPC            |
| 9090  | HBase Thrift                |

---

## 📁 Project Structure

```
bigdata-mac-apple-silicon/
├── docker-compose.yml          # All services
├── hadoop.env                  # Hadoop configuration
├── .github/
│   └── workflows/
│       └── validate.yml        # GitHub Actions CI
├── .gitignore
├── LICENSE
├── CONTRIBUTING.md
└── README.md
```

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs welcome!

---

## 📄 License

[MIT](LICENSE) — free to use, modify, and share.

---

<p align="center">Made for Mac Apple Silicon users who miss Cloudera 🍎🐘</p>
