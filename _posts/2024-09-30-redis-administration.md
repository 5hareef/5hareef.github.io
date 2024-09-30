---
title: "Redis - HyperLogLog, Piping, Protocol, Administration"
date: Mon Sep 30 19:37:29 IST 2024
categories: [redis,hyperloglog,redis-administration,redis-piping]
tags: [redis]
image:
  path: /assets/img/redis/redis_administration.png
  alt: Redis Administration
---

# Redis - HyperLogLog, Piping, Protocol, Administration

## HyperLogLog

- It is a data structure that gives approximate results.
- It's used to estimate the count of unique items in a set.
- Stored as a Redis string.
- Works similarly to `Sets` but uses less memory.
- Not 100% accurate, but saves a lot of memory.
- Helps you keep track of counts for millions of items very efficiently.
- Uses just three commands.
    - `PFADD`
    - `PFCOUNT`
    - `PFMERGE`

### Commands

❯ Adds elements to a HyperLogLog key. Creates the key if it doesn't exist

❯ Syntax: `pfadd [key] [members]`

❯ Returns the approximated cardinality of the set(s) observed by the HyperLogLog key(s)

❯ Syntax: `pfcount [key1] ...`

❯ Merges one or more HyperLogLog values into a single key

❯ Syntax: `pfmerge [result_hll_key] [key1] [key2] ...`

```bash
127.0.0.1:6379> pfadd cloudsimplify:home_page:ips 12.12.12.12 34.12.0.5 90.90.125.127
(integer) 1
127.0.0.1:6379> pfadd cloudsimplify:blog_page:ips 56.234.202.101 89.233.233.78
(integer) 1
127.0.0.1:6379> pfadd cloudsimplify:contact_page:ips 12.12.12.12 34.12.0.5 90.90.125.127
(integer) 1
127.0.0.1:6379> pfcount cloudsimplify:home_page:ips cloudsimplify:blog_page:ips cloudsimplify:contact_page:ips
(integer) 5
127.0.0.1:6379> pfmerge cloudsmplify:visitor:unique:ips cloudsimplify:home_page:ips cloudsimplify:blog_page:ips cloudsimplify:contact_page:ips
OK
127.0.0.1:6379> pfcount cloudsmplify:visitor:unique:ips
(integer) 5
```

> ⚠️ **Can we view the HyperLogLog data?**
> 
> *No, it does not store the data but only the cardinality* 

---

## Pipeling

The `redis --pipe` feature allows for efficient bulk data transfer to a Redis server by using pipelining. Instead of sending and waiting for each command individually (SYNC), it sends multiple commands at once in a single network round trip (ASYNC), improving performance. This is particularly useful for importing or exporting large datasets. The process minimizes latency and network overhead by batching commands, making it faster than sending commands one by one.

```bash
curl -sL https://datahub.io/core/country-list/_r/-/data.csv \
| awk -F',' 'NR>1 && length($2)==2 {print "SET country:"$2 " " "\""$1"\""}' \
| redis-cli --pipe

All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 233
```

---

## RESP Protocol

- Redis clients talk to the Redis server using a protocol called **RESP** (REdis Serialization Protocol).
- It's easy to use, quick to process, and readable by humans.
- RESP can handle different types of data like:
    - Strings
    - Numbers
    - Lists
    - Booleans
- RESP is binary safe, meaning it can handle all kinds of data.
- The type of data in RESP is identified by the first character:
    - For **simple strings**, it starts with `+`
    - For **errors**, it starts with `-`
    - For **integers**, it starts with `:`
    - For **bulk strings**, it starts with `$`

![redis_resp_protocol.png](assets/img/redis/redis_resp_protocol.png)
_Redis RESP Protocol_

```bash
echo -e "PING\r\nQUIT\r\n" | nc localhost 6379
+PONG
+OK
```

[Redis serialization protocol specification](https://redis.io/docs/latest/develop/reference/protocol-spec/)

---

## Administration

❯ `OBJECT REFCOUNT` ➖ This command tells you how many parts of Redis are currently **referring** to the same key's value in memory (i.e., the **reference count**).  It helps you understand how many times the data is being shared or reused across different keys or structures. Ex:

```bash
127.0.0.1:6379> RPUSH mylist "Hello"
(integer) 1
127.0.0.1:6379> object refcount mylist
(integer) 1
```

> ℹ️ **Notes:**
> - The reference count (`OBJECT REFCOUNT`) will only show a value greater than `1` if **multiple references exist within Redis to the same memory location**.
> - This happens if Redis is reusing the **same object** internally, such as with **shared objects** in lists, sets, or other data structures.
> - **Copy-On-Write**: During a forked background process (like during snapshotting), Redis uses a copy-on-write mechanism where reference counts for objects in memory can increase temporarily.

❯ `OBJECT ENCODING` ➖ This command tells you how the data for a specific key is **stored in memory**. Redis can store data in different formats (e.g., `embstr`, `ziplist`, etc.) to optimize memory use.  It helps you understand Redis' internal memory optimization, which can improve performance and save memory. Ex:

```bash
127.0.0.1:6379> RPUSH mylist "Hello"
(integer) 1
127.0.0.1:6379> object encoding mylist
"quicklist"
```

❯ `OBJECT IDLETIME` ➖ This command tells you how many **seconds** a key has been idle (i.e., how long it's been since the key was last accessed). You can use it to check if a key hasn't been used for a while, which is helpful if you want to know which keys are `cold` (not being used). Ex:

```bash
127.0.0.1:6379> RPUSH mylist "Hello"
(integer) 1
127.0.0.1:6379> object idletime mylist
(integer) 235
```

❯ `DUMP` ➖ Exports the serialized version of a key's value from Redis. You can use `DUMP` to get a binary representation of the value stored at a specific key, which can be used to **back up** or **transfer** that key's data to another Redis instance.

```bash
127.0.0.1:6379> set name cloudsimplify
OK
127.0.0.1:6379> get name
"cloudsimplify"
127.0.0.1:6379> dump name
"\x00\rcloudsimplify\t\x00\x12\xe6\tu7\xcb\xab'"
```

❯ `RESTORE` ➖ Takes the binary data created by `DUMP` and **restores** it into a Redis key. You can use `RESTORE` to **import** or **recover** data that was previously dumped. It recreates the key with its value exactly as it was, including its expiration time (if applicable).

❯ Syntax: `restore [key] [ttl] [serialized_value]` 

```bash
127.0.0.1:6379> set name cloudsimplify
OK
127.0.0.1:6379> get name
"cloudsimplify"
127.0.0.1:6379> dump name
"\x00\rcloudsimplify\t\x00\x12\xe6\tu7\xcb\xab'"
127.0.0.1:6379> restore newname 0 "\x00\rcloudsimplify\t\x00\x12\xe6\tu7\xcb\xab'"
OK
127.0.0.1:6379> get newname
"cloudsimplify"
127.0.0.1:63
```

❯ `INFO` ➖ Provides detailed information about the Redis server, such as memory usage, number of connected clients, key statistics, and more. Useful for **monitoring** the server’s performance and health

```bash
127.0.0.1:6379> info
# Server
redis_version:6.2.14
redis_git_sha1:00000000
redis_git_dirty:0
...

# Clients
connected_clients:1
cluster_connections:0
maxclients:10000
...

# Memory
used_memory:1079200
used_memory_human:1.03M
...
127.0.0.1:6379> info memory        # If you want to see specific sections, like memory info:
# Memory
used_memory:1077200
used_memory_human:1.03M
used_memory_rss:4243456
used_memory_rss_human:4.05M
...
```

❯ `PING` ➖ Tests the connection to the Redis server. To check if the Redis server is running and responding.

```bash
$ redis-cli ping
PONG
```

❯ `DBSIZE` ➖ Returns the total number of keys in the current database. To check how many keys are stored in a specific Redis database.

```bash
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> set name cloudsimplify
OK
127.0.0.1:6379> dbsize
(integer) 1
```

❯ `DEBUG POPULATE` ➖ is a **testing tool** that lets you quickly create a large number of random keys with random values in Redis. It’s useful for **performance testing**, **memory checks**, and **benchmarking**.

```bash
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> debug populate 10
OK
127.0.0.1:6379> dbsize
(integer) 10
```

❯ `SELECT` ➖ lets you choose which Redis database to work with. Redis has multiple databases (by default 16), and each one is independent. 

❯ Syntax: `select [database_number]` 

```bash
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> set name cloudsimplify
OK
127.0.0.1:6379> dbsize
(integer) 1
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
127.0.0.1:6379[1]> set name cloudsimplify
OK
127.0.0.1:6379[1]> dbsize
(integer) 1
```

❯ `FLUSHALL` ➖ Deletes **all keys** in **all databases**. To completely wipe out the entire Redis server data.

```bash
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> debug populate 100
OK
127.0.0.1:6379> dbsize
(integer) 100
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
127.0.0.1:6379[1]> debug populate 500
OK
127.0.0.1:6379[1]> dbsize
(integer) 500
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
```

❯ `FLUSHDB` ➖ Deletes **all keys** in the **current database**. To wipe out just the data in a single database, not the entire server.

```bash
127.0.0.1:6379> dbsize
(integer) 0
127.0.0.1:6379> debug populate 100
OK
127.0.0.1:6379> dbsize
(integer) 100
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
127.0.0.1:6379[1]> debug populate 500
OK
127.0.0.1:6379[1]> dbsize
(integer) 500
127.0.0.1:6379[1]> flushdb
OK
127.0.0.1:6379[1]> dbsize
(integer) 0
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> dbsize
(integer) 100
127.0.0.1:6379>
```

❯ `MONITOR` ➖ Streams back every command that is being executed on the Redis server in real-time. Useful for **debugging** or seeing what commands are being sent to the Redis server live.

![redis_monitor.gif](assets/img/redis/redis_monitor.gif)

❯ `CONFIG` ➖ Manages the Redis server's configuration. You can get, set, or rewrite the server configuration parameters. To change or view the configuration of the Redis server without restarting it.

```bash
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "0"
127.0.0.1:6379> config set maxmemory 128mb
OK
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "134217728"
```

❯ `SHUTDOWN` ➖ Stops the Redis server. It will save the current dataset to disk (if configured to do so) before shutting down. To gracefully shut down the Redis server.

- `SHUTDOWN SAVE`: Saves the dataset to disk before shutting down, ensuring all recent changes are persisted.
- `SHUTDOWN NOSAVE`: Immediately shuts down the server without saving, which is faster but may lead to data loss of unsaved changes.

![redis_shutdown_command.gif](assets/img/redis/redis_shutdown_command.gif)
