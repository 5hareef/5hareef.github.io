---
title: "Redis Hashes"
date: Wed Jul 31 19:22:12 IST 2024
categories: [redis,hashes]
tags: [redis]
image:
  path: /assets/img/redis/redis_hashes.png
  alt: Redis Hashes
---

# Redis - Hashes

- **Hashes** are collections of fields-value pairs.
- Example: `field1:value1`, `field2:value2`, `field3:value3`.
- They are similar to JSON objects.
- Hashes represent the mapping relationship between fields and values.
- **Elements in a hash**:
    - Are strings.
    - Consist of fields paired with their values.
- Hashes are schema less.
- A hash can contain more than 4 billion elements.

## Commands

❯ Creates or modifies the value of a field in a hash

❯ Syntax: `hset [key] [field1] [value1] [field2] [value2] ...`

❯ Returns the value of a field in a hash

❯ Syntax: `hget [key] [field]`

❯ Returns all fields and values in a hash

❯ Syntax: `hgetall [key]` 

```bash
127.0.0.1:6379> hset certificate name "cloudsimplify" organisation "IT" age 5
(integer) 3
127.0.0.1:6379> hget certificate name
"cloudsimplify"
127.0.0.1:6379> hgetall certificate
1) "name"
2) "cloudsimplify"
3) "organisation"
4) "IT"
5) "age"
6) "5"
127.0.0.1:6379> hset certificate age 3    
(integer) 0                                      # 0 means change is made but no new field is added 
127.0.0.1:6379> hset certificate active true
(integer) 1
127.0.0.1:6379> hgetall certificate
1) "name"
2) "cloudsimplify"
3) "organisation"
4) "IT"
5) "age"
6) "3"
7) "active"
8) "true"
```

❯ Returns the values of all fields in a hash

❯ Syntax: `hmget [key] [field ...]` 

```bash
127.0.0.1:6379> hset certificate name "cloudsimplify" organisation "IT" age 5
(integer) 3
127.0.0.1:6379> hmget certificate name organisation
1) "cloudsimplify"
2) "IT"
```

❯ Returns the number of fields in a hash

❯ Syntax: `hlen [key]`

```bash
127.0.0.1:6379> hset certificate name "cloudsimplify" organisation "IT" age 5
(integer) 3
127.0.0.1:6379> hlen certificate
(integer) 4
```

❯ Deletes one or more fields and their values from a hash. Deletes the hash if no fields remain

❯ Syntax: `hdel [key] [field ...]` 

```bash
127.0.0.1:6379> hset certificate name "cloudsimplify" organisation "IT" age 5
(integer) 3
127.0.0.1:6379> hgetall certificate
1) "name"
2) "cloudsimplify"
3) "organisation"
4) "IT"
5) "age"
6) "3"
7) "active"
8) "true"
127.0.0.1:6379> hdel certificate organisation
(integer) 1
127.0.0.1:6379> hgetall certificate
1) "name"
2) "cloudsimplify"
3) "age"
4) "3"
5) "active"
6) "true"
127.0.0.1:6379> hdel certificate age active
(integer) 2
127.0.0.1:6379> hgetall certificate
1) "name"
2) "cloudsimplify"
```

❯ Determines whether a field exists in a hash

❯ Syntax: `hexists [key] [field]` 

```bash
127.0.0.1:6379> hset user name "cloudsimplify" role "admin" status "active"
(integer) 3
127.0.0.1:6379> hgetall user
1) "name"
2) "cloudsimplify"
3) "role"
4) "admin"
5) "status"
6) "active"
127.0.0.1:6379> hexists user email
(integer) 0
127.0.0.1:6379> hset user email "cloudsimplify@mydomain.com"
(integer) 1
127.0.0.1:6379> hexists user email
(integer) 1
127.0.0.1:6379>
```

❯ Returns all fields in a hash

❯ Syntax: `hkeys [key]`

❯ Returns all values in a hash

❯ Syntax: `hvals [key]` 

```bash
127.0.0.1:6379> hset user:100 name "admin" role "admin" status "active" email "admin@cloudsimplify.me"
(integer) 4
127.0.0.1:6379> hkeys user:100
1) "name"
2) "role"
3) "status"
4) "email"
127.0.0.1:6379> hvals user:100
1) "admin"
2) "admin"
3) "active"
4) "admin@cloudsimplify.me"
127.0.0.1:6379>
```

❯ Increments the integer value of a field in a hash by a number. Uses 0 as initial value if the field doesn't exist

❯ Syntax: `hincrby [key] [field] [increment/decrement]`

❯ Increments the floating point value of a field by a number. Uses 0 as initial value if the field doesn't exist

❯ Syntax: `hincrbyfloat [key] [field] [increment/decrement]` 

```bash
127.0.0.1:6379> hset user:100 name "admin" role "admin" status "active" email "admin@cloudsimplify.me"
(integer) 4
127.0.0.1:6379> hincrby user:100 rank 5
(integer) 5
127.0.0.1:6379> hgetall user:100
 1) "name"
 2) "admin"
 3) "role"
 4) "admin"
 5) "status"
 6) "active"
 7) "email"
 8) "admin@cloudsimplify.me"
 9) "rank"
10) "5"
127.0.0.1:6379> hincrby user:100 rank -2
(integer) 3
127.0.0.1:6379> hget user:100 rank
"3"
127.0.0.1:6379> hincrbyfloat user:100 score 45.35
"45.35"
127.0.0.1:6379> hgetall user:100
 1) "name"
 2) "admin"
 3) "role"
 4) "admin"
 5) "status"
 6) "active"
 7) "email"
 8) "admin@cloudsimplify.me"
 9) "rank"
10) "3"
11) "score"
12) "45.35"
127.0.0.1:6379> hincrbyfloat user:100 score -12.30
"33.05"
127.0.0.1:6379> hgetall user:100
 1) "name"
 2) "admin"
 3) "role"
 4) "admin"
 5) "status"
 6) "active"
 7) "email"
 8) "admin@cloudsimplify.me"
 9) "rank"
10) "3"
11) "score"
12) "33.05"
```

❯ Returns one or more random fields from a hash

❯ Syntax: `hrandfield [key] [count] [withvalues]` 

> The `count` parameter returns a random number of fields equal to the specified value. If `count` is positive, the returned fields are distinct. If `count` is negative, it may return duplicate fields. The `WithValues` option returns random fields along with their values.
> 

```bash
127.0.0.1:6379> hgetall user:100
 1) "name"
 2) "admin"
 3) "role"
 4) "admin"
 5) "status"
 6) "active"
 7) "email"
 8) "admin@cloudsimplify.me"
 9) "rank"
10) "3"
11) "score"
12) "33.05"
127.0.0.1:6379> hrandfield user:100
"email"
127.0.0.1:6379> hrandfield user:100
"rank"
127.0.0.1:6379> hrandfield user:100 2
1) "email"
2) "rank"
127.0.0.1:6379> hrandfield user:100 -2
1) "name"
2) "email"
127.0.0.1:6379> hrandfield user:100 -2
1) "role"
2) "role"
127.0.0.1:6379> hrandfield user:100 2 withvalues
1) "status"
2) "active"
3) "rank"
4) "3"
```