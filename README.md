# 🐘 Big Data Stack for Mac Apple Silicon (M1 / M2 / M3 / M4)

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Apple%20Silicon%20ARM64-black?logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Required-2496ED?logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Hadoop-3.3.1-66CCFF?logo=apachehadoop&logoColor=white" />
  <img src="https://img.shields.io/badge/Hive-2.3.2-FDEE21?logo=apachehive&logoColor=black" />
  <img src="https://img.shields.io/badge/Spark-3.4-E25A1C?logo=apachespark&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-green" />
</p>

<p align="center">
  A <strong>Cloudera-like Big Data environment</strong> that runs natively on Mac Apple Silicon.<br/>
  No VM. No other OS. Just Docker Desktop and one command.
</p>

---

## ✨ Why this project?

Cloudera QuickStart VM is x86-only and **cannot run on Mac M1/M2/M3/M4**. This project gives you the same experience — Hadoop, Hive, HBase, Spark, and a Hue web UI — fully working on Apple Silicon via Docker.

---

## 📦 Stack

| Service | Version | Role | Web UI |
|---|---|---|---|
| **Hue** | latest | ⭐ Browser UI (like Cloudera Manager) | [localhost:8888](http://localhost:8888) |
| **Hadoop HDFS** | 3.3.1 | Distributed file system | [localhost:9870](http://localhost:9870) |
| **YARN** | 3.3.1 | Resource & job manager | [localhost:8088](http://localhost:8088) |
| **Hive** | 2.3.2 | SQL on Hadoop | [localhost:10002](http://localhost:10002) |
| **HBase** | latest | NoSQL wide-column store | [localhost:16010](http://localhost:16010) |
| **Spark** | 3.4 | Fast in-memory processing | [localhost:8080](http://localhost:8080) |
| **Zookeeper** | 3.8 | Coordination service | — |
| **PostgreSQL** | 14 | Hive metastore + Hue backend | — |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Your Mac M4 Browser                │
│              http://localhost:8888 (Hue)            │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                   Docker Network                    │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ NameNode │  │ DataNode │  │ ResourceManager  │  │
│  │  (HDFS)  │  │  (HDFS)  │  │    (YARN)        │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │   Hive   │  │  HBase   │  │  Spark Master    │  │
│  │  Server  │  │          │  │  + Worker        │  │
│  └──────────┘  └──────────┘  └──────────────────┘  │
│                                                     │
│  ┌──────────┐  ┌──────────┐                         │
│  │Zookeeper │  │PostgreSQL│  (Hive metastore + Hue) │
│  └──────────┘  └──────────┘                         │
└─────────────────────────────────────────────────────┘
```

---

## ⚡ Prerequisites

- Mac with **Apple Silicon** (M1, M2, M3, or M4)
- **16 GB RAM** recommended
- **Docker Desktop** for Apple Silicon → [Download here](https://desktop.docker.com/mac/main/arm64/Docker.dmg)
- ~**10 GB** free disk space (for Docker images)

### ⚙️ Docker Desktop Settings (important!)
Open Docker Desktop → **Settings → Resources** and set:
- CPUs: **5**
- Memory: **10 GB**
- Swap: **2 GB**
- Click **Apply & Restart**

---

## 🚀 Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/YOUR_USERNAME/bigdata-mac-apple-silicon.git
cd bigdata-mac-apple-silicon

# 2. Start everything
docker-compose up -d

# 3. Check all services are running
docker-compose ps

# 4. Open Hue in your browser
open http://localhost:8888
```

> **First run:** Docker downloads all images (~3–4 GB). This takes 5–10 minutes.
> After that, startup takes about 60 seconds.

**Default Hue login:** `admin` / `admin`

---

## 🌐 Hue — Your Main Interface

Hue runs at **http://localhost:8888** and gives you:

- 📁 **File Browser** — upload/download files to/from HDFS
- 📝 **Hive Editor** — write SQL and run queries
- 🗄️ **HBase Browser** — view and edit HBase tables
- ⚙️ **Job Browser** — monitor MapReduce and YARN jobs
- 🔥 **Spark Notebooks** — run PySpark interactively

---

## 💡 Usage Examples

### Hive SQL (via Hue Editor or terminal)

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

-- Query
SELECT dept, AVG(salary) as avg_salary
FROM employees
GROUP BY dept;
```

Or via terminal:
```bash
docker exec -it hive-server bash
/opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
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
text = spark.read.text("hdfs://namenode:9000/mydata/file.txt")
from pyspark.sql.functions import explode, split, col
words = text.select(explode(split(col("value"), " ")).alias("word"))
words.groupBy("word").count().orderBy("count", ascending=False).show()
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
docker-compose restart hue

# View logs
docker-compose logs -f hue
docker-compose logs -f hive-server
docker-compose logs -f namenode
```

---

## 🔧 Troubleshooting

### Hue not loading at localhost:8888?
Hue waits for Hive and HDFS to be ready. Wait 2–3 more minutes after `docker-compose up -d`, then refresh.

### A service shows "Exit" in `docker-compose ps`?
```bash
docker-compose logs <service-name>
docker-compose restart <service-name>
```

### "No space left on device" error?
Open Docker Desktop → Settings → Resources → increase **Disk image size**.

### Architecture warning `linux/amd64` on M4?
```
WARNING: The requested image's platform (linux/amd64) does not match...
```
This is **normal and expected**. Docker uses Rosetta 2 to run amd64 images. They work fine — just slightly slower than native ARM64. Native ARM64 images are used wherever possible.

### Complete reset (if everything is broken)
```bash
docker-compose down -v
docker system prune -f
docker-compose up -d
```

---

## 📁 Project Structure

```
bigdata-mac-apple-silicon/
├── docker-compose.yml          # All services defined here
├── hadoop.env                  # Hadoop/YARN/MapReduce config
├── hue-config/
│   └── hue.ini                 # Hue connections to Hive, HBase, HDFS
├── .github/
│   └── workflows/
│       └── validate.yml        # GitHub Actions CI
├── .gitignore
├── LICENSE
├── CONTRIBUTING.md
└── README.md
```

---

## 🗺️ Port Reference

| Port  | Service                     |
|-------|-----------------------------|
| 8888  | **Hue Web UI** ⭐            |
| 9870  | HDFS NameNode UI            |
| 8088  | YARN ResourceManager UI     |
| 8080  | Spark Master UI             |
| 8081  | Spark Worker UI             |
| 16010 | HBase Master UI             |
| 10002 | HiveServer2 UI              |
| 8188  | MapReduce History Server    |
| 9864  | HDFS DataNode UI            |
| 8042  | YARN NodeManager UI         |
| 9000  | HDFS RPC                    |
| 10000 | Hive JDBC                   |
| 9083  | Hive Metastore Thrift       |
| 2181  | Zookeeper                   |
| 7077  | Spark Master RPC            |
| 9090  | HBase Thrift                |

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). PRs are welcome — especially ARM64-native image alternatives and new service additions.

---

## 📄 License

[MIT](LICENSE) — free to use, modify, and share.

---

<p align="center">Made for Mac Apple Silicon users who miss Cloudera 🍎🐘</p>
