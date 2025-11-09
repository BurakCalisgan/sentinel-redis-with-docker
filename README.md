# Redis Sentinel Cluster (Docker Compose)

This repository provides a **fully functional Redis Sentinel setup** using Docker Compose.

---

## 🧩 Architecture

| Component      | Role        | Internal Port | Host Port | Description            |
|----------------|-------------|---------------|------------|------------------------|
| redis-master   | Master       | 6379          | 6379       | Primary node           |
| redis-replica1 | Replica      | 6379          | 6380       | Follows master         |
| redis-replica2 | Replica      | 6379          | 6381       | Follows master         |
| sentinel1      | Sentinel     | 26379         | 26379      | Monitors cluster       |
| sentinel2      | Sentinel     | 26379         | 26380      | Monitors cluster       |
| sentinel3      | Sentinel     | 26379         | 26381      | Monitors cluster       |
| redis-insight  | UI Dashboard | 5540          | 5540       | Web management console |

> Master, replicas and sentinels all share common config files (`redis.conf`, `sentinel.conf`).

---

## ⚙️ Configuration Files

### `redis.conf`

bind 0.0.0.0
protected-mode no
port 6379

requirepass S3cureRedis!2025
masterauth S3cureRedis!2025

appendonly yes
appendfsync everysec
tcp-keepalive 60
maxclients 10000
loglevel notice
dir /data

---

### `sentinel.conf`

bind 0.0.0.0
port 26379
protected-mode no
dir /data

sentinel monitor mymaster redis-master 6379 2
sentinel auth-pass mymaster S3cureRedis!2025
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1

---

## 🚀 How to Run

docker compose up -d

Verify all containers are running:

docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"

---

## 🧪 Quick Tests

### 1️⃣ Health
redis-cli -h 127.0.0.1 -p 6379 -a S3cureRedis!2025 ping
redis-cli -h 127.0.0.1 -p 6380 -a S3cureRedis!2025 ping
redis-cli -h 127.0.0.1 -p 6381 -a S3cureRedis!2025 ping

### 2️⃣ Replication
redis-cli -h 127.0.0.1 -p 6379 -a S3cureRedis!2025 set city "Istanbul"
redis-cli -h 127.0.0.1 -p 6380 -a S3cureRedis!2025 get city
redis-cli -h 127.0.0.1 -p 6381 -a S3cureRedis!2025 get city

### 3️⃣ Sentinel Status
redis-cli -h 127.0.0.1 -p 26379 SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
redis-cli -h 127.0.0.1 -p 26379 SENTINEL SLAVES mymaster

### 4️⃣ Manual Failover
redis-cli -h 127.0.0.1 -p 26379 SENTINEL FAILOVER mymaster

### 5️⃣ Automatic Failover
Stop master:
docker stop redis-master
sleep 10
redis-cli -h 127.0.0.1 -p 26379 SENTINEL GET-MASTER-ADDR-BY-NAME mymaster

Restart master:
docker start redis-master

---

## 🧠 Notes

- Default password: S3cureRedis!2025
- Redis Insight UI: http://localhost:5540
- All `data/` folders are mounted volumes and ignored in Git via `.gitignore`.

---

## ✅ Test Summary

1. Cluster health check ✅  
2. Replication sync ✅  
3. Role verification ✅  
4. Sentinel connectivity ✅  
5. Manual failover ✅  
6. Automatic failover ✅  
7. Data consistency ✅  

---

© 2025 – Redis Sentinel Cluster by Burak Çalışgan
