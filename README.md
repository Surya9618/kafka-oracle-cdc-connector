# 🚀 Kafka Oracle CDC Connector POC

> End-to-end **Oracle Change Data Capture (CDC)** pipeline using Confluent Oracle CDC Connector.

---

## 📌 Overview

This project demonstrates how to stream **real-time database changes** from Oracle into Kafka using the **Confluent Oracle CDC Source Connector**.

Instead of traditional polling (JDBC), this setup captures **every INSERT, UPDATE, and DELETE** directly from Oracle redo logs and publishes them into Kafka topics with minimal latency.

---

## 🏗️ Architecture

```
Oracle DB → Redo Logs → LogMiner → Kafka Connect (CDC) → Kafka Topics → Consumers
```

---

## 🎯 What This POC Covers

- Oracle CDC setup using LogMiner
- Enabling ARCHIVELOG & supplemental logging
- Creating CDC user with required privileges
- Kafka Connect Oracle CDC connector deployment
- Streaming real-time events into Kafka
- Validating INSERT / UPDATE / DELETE flows

---

## ⚙️ Prerequisites

- Docker & Docker Compose
- Kafka + Kafka Connect
- Oracle DB container/image
- Confluent Oracle CDC connector plugin
- (Optional) Schema Registry for Avro

---

## 🛠️ Setup Steps

### 1. Start Services

```bash
docker compose up -d
```

### 2. Wait for Oracle to Start(login steps were provided in blog mentioned at the end)

```bash
docker logs -f oracle
```

### 3. Run Initialization Script

```bash
docker exec -it oracle bash
sh /app/initial_setup.sh
```

---

## 🔌 Deploy CDC Connector

```bash
curl -X POST http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d @cdc_customers_connector.json
```

---

## 📊 Monitor Data Flow

Consume Kafka topic:

```bash
kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic ORCLPDB1.C__DBUSER.CUSTOMERS \
  --from-beginning
```

---

## 🧪 Test Operations

### INSERT
```sql
INSERT INTO CUSTOMERS (first_name, last_name, email, gender, club_status, comments)
VALUES ('Alex', 'Johnson', 'alex.johnson@test.com', 'Male', 'gold', 'New CDC insert test');

INSERT INTO CUSTOMERS (first_name, last_name, email, gender, club_status, comments)
VALUES ('Emily', 'Clark', 'emily.clark@test.com', 'Female', 'silver', 'Another CDC insert');

COMMIT;
```

### UPDATE
```sql
UPDATE CUSTOMERS
SET club_status = 'platinum',
    update_ts = CURRENT_TIMESTAMP
WHERE first_name = 'Alex';

COMMIT;
```

### DELETE
```sql
DELETE FROM CUSTOMERS
WHERE first_name = 'Emily';

COMMIT;
```

---

## 🔍 Expected Output

Look for operation types(op_type):

- **I** → Insert  
- **U** → Update  
- **D** → Delete  

Example:
```json
"op_type": { "string": "D" }
```

---


## 🧰 Useful Commands

```bash
# Check connector status
curl http://localhost:8083/connectors/<name>/status

# Restart connector
curl -X POST http://localhost:8083/connectors/<name>/restart

# Restart task
curl -X POST http://localhost:8083/connectors/<name>/tasks/0/restart
```

```bash
# Login from inside the container (best & easiest)
sudo docker exec -it oracle bash
sqlplus / as sysdba
```

```bash
# Login to CDB (ORCLCDB) using password
sqlplus sys/surya123@//localhost:1521/ORCLCDB as sysdba
```

```bash
# Login to PDB (your actual working DB)
sqlplus sys/surya123@//localhost:1521/ORCLPDB1 as sysdba
```

```bash
# Login as your CDC user(recommended)
sqlplus C##DBUSER/userpass123@//localhost:1521/ORCLPDB1
```

### Verify where you are upon log in
```sql
SHOW CON_NAME;
```
---

## ✅ Conclusion

This POC demonstrates how Oracle can be transformed into a **real-time event source** using CDC and Kafka. As a next step, you can plug in a **JDBC Sink Connector** to replicate data into another database and build a full streaming pipeline.

---

## 👨‍💻 Author

Surya Teja
