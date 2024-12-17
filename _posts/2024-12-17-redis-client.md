---
title: "Redis - Client"
date: Tue Dec 17 12:56:52 GMT 2024
categories: [redis,client]
tags: [redis]
image:
  path: /assets/img/redis/redis_client.png
  alt: Redis Client
---

# Redis Client

### Difference between `redis-cli scan`, `redis-cli --scan`, and `redis-cli keys`:

| **Command** | **`redis-cli keys`** | **`redis-cli scan`** | **`redis-cli --scan`** |
| --- | --- | --- | --- |
| **Function** | Fetches all keys matching a pattern | Iteratively scans keys in batches | Shortcut for `SCAN` to list all keys |
| **Blocking/Non-Blocking** | Blocking (can affect Redis performance) | Non-blocking (safe for production) | Non-blocking (safe for production) |
| **Performance** | Slow on large datasets | Fast and efficient for large datasets | Fast and efficient for large datasets |
| **Use Case** | Small datasets or quick lookups | Large datasets, production-safe | Large datasets, user-friendly |
| **Cursor Handling** | Not applicable | You need to manually manage the cursor | Cursor handled automatically |
| **Pattern Matching** | Supports patterns | Supports patterns | Supports patterns |

### **Redis Patterns Use Glob-Style Matching, Not Regular Expressions**

- The `-scan` command with `-pattern` uses **glob-style pattern matching**, which is simpler than regular expressions.
- Redis expects patterns with wildcards like:
    - `?`: Matches a single character.
    - ``: Matches zero or more characters.
    - `[abc]`: Matches any character within the brackets.

**Example of Valid Patterns:**

- `"key*"`: Matches all keys starting with "key".
- `"??abc"`: Matches keys with exactly 2 characters before "abc".
- `"user[0-9]*"`: Matches keys starting with "user" followed by a number.

### **If Regex is Necessary**

- Redis itself doesn't support regex for `SCAN`. However, you can filter results using external tools like `grep` or use scripting (e.g., in Python).
- Example using `grep` to filter keys matching the regex:
    
    ```bash
    redis-cli --scan | grep -P '^([a-zA-Z0-9]){7}$'
    ```
    

Explanation:

- `redis-cli --scan`: Retrieves keys from Redis.
- `grep -E '^([a-zA-Z0-9]){7}$'`: Filters results to match the regex.

### `redis-cli --scan` examples

1. Populate some keys
    
    ```bash
    127.0.0.1:6379> debug populate 10 user:id
    OK
    127.0.0.1:6379> keys *
     1) "user:id:6"
     2) "user:id:7"
     3) "user:id:2"
     4) "user:id:3"
     5) "user:id:0"
     6) "user:id:5"
     7) "user:id:9"
     8) "user:id:8"
     9) "user:id:4"
    10) "user:id:1"
    127.0.0.1:6379> debug populate 10 employee:id
    OK
    127.0.0.1:6379> keys *
     1) "user:id:6"
     2) "user:id:7"
     3) "user:id:2"
     4) "employee:id:9"
     5) "employee:id:8"
     6) "user:id:3"
     7) "user:id:0"
     8) "employee:id:5"
     9) "user:id:5"
    10) "employee:id:4"
    11) "user:id:9"
    12) "employee:id:0"
    13) "user:id:8"
    14) "employee:id:1"
    15) "employee:id:3"
    16) "employee:id:7"
    17) "employee:id:2"
    18) "user:id:1"
    19) "user:id:4"
    20) "employee:id:6"
    127.0.0.1:6379> quit
    ```
    
2. Bulk delete keys using pipe based on pattern
    
    ```bash
    ❯ redis-cli --scan --pattern "employee:id*" | awk '{print "UNLINK " $0}' | redis-cli --pipe
    All data transferred. Waiting for the last reply...
    Last reply received from server.
    errors: 0, replies: 10
    ```
    
3. Bulk get keys
    
    ```bash
    ❯ for key in $(redis-cli --scan --pattern "user:id*"); do
      echo "${key} $(redis-cli GET ${key})"
    done
    
    user:id:6 value:6
    user:id:3 value:3
    user:id:8 value:8
    user:id:1 value:1
    user:id:4 value:4
    user:id:7 value:7
    user:id:0 value:0
    user:id:2 value:2
    user:id:9 value:9
    user:id:5 value:5
    ```