# ⚡ Quick Start Guide

The most important commands — perfect for beginners!

---

## 1️⃣ Start the Stack

```bash
cd ~/bigdata-mac-apple-silicon
open -a Docker
docker-compose up -d
```

---

## 2️⃣ Check Everything is Running

```bash
docker-compose ps
```
All services should show **Up**.

---

## 3️⃣ Use Hive (via DBeaver)

1. Open DBeaver
2. Select the Hive connection
3. Open SQL Editor
4. Write your query and press `Ctrl+Enter`

```sql
SHOW DATABASES;
CREATE DATABASE test;
USE test;
CREATE TABLE mytable (id INT, name STRING);
INSERT INTO mytable VALUES (1, 'Alice');
SELECT * FROM mytable;
```

---

## 4️⃣ Use HDFS

```bash
# Upload a file
docker cp myfile.csv namenode:/tmp/
docker exec namenode hdfs dfs -mkdir /data
docker exec namenode hdfs dfs -put /tmp/myfile.csv /data/

# List files
docker exec namenode hdfs dfs -ls /data/
```

---

## 5️⃣ Use HBase

```bash
docker exec -it hbase hbase shell
```
```ruby
create 'mytable', 'cf'
put 'mytable', 'row1', 'cf:name', 'Alice'
scan 'mytable'
exit
```

---

## 6️⃣ Use Spark

```bash
docker exec -it spark-master pyspark --master spark://spark-master:7077
```
```python
df = spark.createDataFrame([("Alice", 95), ("Bob", 88)], ["name", "marks"])
df.show()
```

---

## 7️⃣ Use Zeppelin Notebooks

Open: **http://localhost:9091**

---

## 8️⃣ Stop the Stack

```bash
docker-compose down
```

---

## 🌐 All Web UIs

| URL | Service |
|---|---|
| http://localhost:9870 | Hadoop HDFS |
| http://localhost:8088 | YARN Jobs |
| http://localhost:8080 | Spark |
| http://localhost:16010 | HBase |
| http://localhost:9091 | Zeppelin |
| http://localhost:8090 | Cloudbeaver |

---

> 📖 Full detailed guide: [USAGE.md](USAGE.md)
