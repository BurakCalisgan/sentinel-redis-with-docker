# Redis Sentinel Cluster with Docker Compose

This repository provides a **fully functional Redis Sentinel** setup running on Docker Compose. It includes one master, two replicas, three sentinels, and optional Redis Insight for UI.

---

## Project Structure

```
sentinel-redis-with-docker/
├── docker-compose.yml
├── conf/
│   ├── master/
│   │   └── redis.conf
│   ├── replica/
│   │   └── redis.conf
│   └── sentinel/
│       └── sentinel.conf
└── README.md
```

- `conf/master/redis.conf` → Master Redis configuration  
- `conf/replica/redis.conf` → Replica configuration (both replicas use the same file)  
- `conf/sentinel/sentinel.conf` → Sentinel configuration (all three use the same file)  

---

## Configuration (reference)

### Master (`conf/master/redis.conf`)
```
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
```

### Replica (`conf/replica/redis.conf`)
```
bind 0.0.0.0
protected-mode no
port 6379

requirepass S3cureRedis!2025
masterauth S3cureRedis!2025
replicaof redis-master 6379

appendonly yes
appendfsync everysec
tcp-keepalive 60
maxclients 10000
loglevel notice
dir /data
```

### Sentinel (`conf/sentinel/sentinel.conf`)
```
bind 0.0.0.0
port 26379
protected-mode no
dir /data

sentinel monitor mymaster redis-master 6379 2
sentinel auth-pass mymaster S3cureRedis!2025
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
sentinel parallel-syncs mymaster 1
```

> All sentinels use the same internal port (26379). Host ports are mapped uniquely via Docker compose (26379/26380/26381).

---

## Services & Ports

| Service        | Role        | Host Port | Container Port |
|----------------|-------------|-----------|----------------|
| redis-master   | Master      | 6379      | 6379           |
| redis-replica1 | Replica     | 6380      | 6379           |
| redis-replica2 | Replica     | 6381      | 6379           |
| sentinel1      | Sentinel    | 26379     | 26379          |
| sentinel2      | Sentinel    | 26380     | 26379          |
| sentinel3      | Sentinel    | 26381     | 26379          |
| redis-insight  | UI          | 5540      | 5540           |

---

## Run

```
docker compose up -d
```

Verify containers:

```
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Status}}"
```

---

## Quick Tests

### 1) Health
```
redis-cli -h 127.0.0.1 -p 6379 -a S3cureRedis!2025 ping
redis-cli -h 127.0.0.1 -p 6380 -a S3cureRedis!2025 ping
redis-cli -h 127.0.0.1 -p 6381 -a S3cureRedis!2025 ping
```

### 2) Replication
```
redis-cli -h 127.0.0.1 -p 6379 -a S3cureRedis!2025 set city "Istanbul"
redis-cli -h 127.0.0.1 -p 6380 -a S3cureRedis!2025 get city
redis-cli -h 127.0.0.1 -p 6381 -a S3cureRedis!2025 get city
```

### 3) Sentinel Status
```
redis-cli -h 127.0.0.1 -p 26379 SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
redis-cli -h 127.0.0.1 -p 26379 SENTINEL SLAVES mymaster
```

### 4) Manual Failover
```
redis-cli -h 127.0.0.1 -p 26379 SENTINEL FAILOVER mymaster
```

### 5) Automatic Failover
```
docker stop redis-master
sleep 10
redis-cli -h 127.0.0.1 -p 26379 SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
docker start redis-master
```

---

## Notes

- Default password: `S3cureRedis!2025`
- Redis Insight UI: `http://localhost:5540`
- `/data` folders are mounted volumes and should be ignored by Git.
- If you see `WRONGTYPE` on `GET city`, it means `city` exists with a non-string type (e.g., a Set). Delete and re-create:
  ```
  redis-cli -h 127.0.0.1 -p <port> -a S3cureRedis!2025 type city
  redis-cli -h 127.0.0.1 -p <port> -a S3cureRedis!2025 del city
  redis-cli -h 127.0.0.1 -p <port> -a S3cureRedis!2025 set city "Istanbul"
  ```

---

## Test Checklist

- Cluster health ✅
- Replication sync ✅
- Sentinel monitoring ✅
- Manual & automatic failover ✅
- Data consistency ✅
