---
title: "Redis Strings"
date: Tue Jul 30 17:12:48 IST 2024
categories: [redis,strings]
tags: [redis]
image:
  path: /assets/img/redis/redis_strings.png
  alt: Redis Strings
---

# Redis - Strings

## Commands

❯ Increments the integer value of a key by one. Uses 0 as initial value if the key doesn't exist.

❯ Syntax: `incr [key]` 

```bash
127.0.0.1:6379> set customer:1000:balance 100
OK
127.0.0.1:6379> get customer:1000:balance
"100"
127.0.0.1:6379> incr customer:1000:balance
(integer) 101
127.0.0.1:6379> get customer:1000:balance
"101"
```

❯ Increments the integer value of a key by a number. Uses 0 as initial value if the key doesn't exist.

❯ Syntax: `incrby [key] [increment]` 

```bash
127.0.0.1:6379> set customer:1000:balance 100
OK
127.0.0.1:6379> get customer:1000:balance
"100"
127.0.0.1:6379> incr customer:1000:balance
(integer) 101
127.0.0.1:6379> incrby customer:1000:balance 9
(integer) 110
127.0.0.1:6379> get customer:1000:balance
"110"
```

❯ Decrements the integer value of a key by one. Uses 0 as initial value if the key doesn't exist.

❯ Syntax: `decr [key]` 

```bash
127.0.0.1:6379> set customer:1000:balance 100
OK
127.0.0.1:6379> get customer:1000:balance
"100"
127.0.0.1:6379> decr customer:1000:balance
(integer) 99
127.0.0.1:6379> get customer:1000:balance
"99"
```

❯ Decrements a number from the integer value of a key. Uses 0 as initial value if the key doesn't exist.

❯ Syntax: `decrby [key] [decrement]` 

```bash
127.0.0.1:6379> set customer:1000:balance 100
OK
127.0.0.1:6379> get customer:1000:balance
"100"
127.0.0.1:6379> decr customer:1000:balance
(integer) 99
127.0.0.1:6379> decrby customer:1000:balance 9
(integer) 90
127.0.0.1:6379> get customer:1000:balance
"90"
```

❯ Increment the floating point value of a key by a number. Uses 0 as initial value if the key doesn't exist.

❯ Syntax: `incrbyfloat [key] [increment/decrement]` 

```bash
127.0.0.1:6379> set customer:1000:balance 12.45
OK
127.0.0.1:6379> get customer:1000:balance
"12.45"
127.0.0.1:6379> type customer:1000:balance
string
127.0.0.1:6379> incr customer:1000:balance
(error) ERR value is not an integer or out of range
127.0.0.1:6379> incrbyfloat customer:1000:balance 0.55
"13"
127.0.0.1:6379> get customer:1000:balance
"13"
127.0.0.1:6379> incrbyfloat customer:1000:balance -2.65
"10.35"
127.0.0.1:6379> get customer:1000:balance
"10.35"
```

❯ Returns the length of a string value.

❯ Syntax: `strlen [key]` 

```bash
127.0.0.1:6379> set name "hello world"
OK
127.0.0.1:6379> get name
"hello world"
127.0.0.1:6379> strlen name
(integer) 11
```

❯ Appends a string to the value of a key. Creates the key if it doesn't exist.

❯ Syntax: `append [key] [value]` 

```bash
127.0.0.1:6379> set name "hello"
OK
127.0.0.1:6379> append name " cloudsimplify"
(integer) 19
127.0.0.1:6379> get name
"hello cloudsimplify"
```

❯ Atomically creates or modifies the string values of one or more keys.

❯ Syntax: `mset [key1] [value1] [key2] [value2] ...` 

❯ Atomically returns the string values of one or more keys.

❯ Syntax: `mget [key1] [key2] ...` 

❯ Atomically modifies the string values of one or more keys only when all keys don't exist.

❯ Syntax: `msetnx [key1] [value1] [key2] [value2] ...`

```bash
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 k4 v4
OK
127.0.0.1:6379> mget k1 k2 k3 k4
1) "v1"
2) "v2"
3) "v3"
4) "v4"
127.0.0.1:6379> mset k1 v10 k2 v20 k3 v3 k4 v4
OK
127.0.0.1:6379> mget k1 k2 k3 k4
1) "v10"
2) "v20"
3) "v3"
4) "v4"
127.0.0.1:6379> msetnx k1 v100 k2 v200 k3 v3 k4 v4 k5 v5 # Failed due to keys k1,k2,k3,k4 already exist
(integer) 0
127.0.0.1:6379> mget k1 k2 k3 k4 k5
1) "v10"
2) "v20"
3) "v3"
4) "v4"
5) (nil)
127.0.0.1:6379> msetnx k5 v5
(integer) 1
127.0.0.1:6379> mget k1 k2 k3 k4 k5
1) "v10"
2) "v20"
3) "v3"
4) "v4"
5) "v5"
```

❯ Returns the previous string value of a key after setting it to a new value

❯ Syntax: `getset [key] [value]` 

```bash
127.0.0.1:6379> set app:config:counter 10
OK
127.0.0.1:6379> decr app:config:counter
(integer) 9
127.0.0.1:6379> decr app:config:counter
(integer) 8
127.0.0.1:6379> getset app:config:counter 10     # Reset the counter value
"8"
127.0.0.1:6379> get app:config:counter
"10"
127.0.0.1:6379>
```

❯ Returns a substring of the string stored at a key

❯ Syntax: `getrange [key] [start-index] [end-index]` 

```bash
127.0.0.1:6379> set website "cloudsimplify.me"
OK
127.0.0.1:6379> getrange website 0 -1
"cloudsimplify.me"
127.0.0.1:6379> getrange website 0 12
"cloudsimplify"
```

❯ Overwrites a part of a string value with another by an offset. Creates the key if it doesn't exist

❯ Syntax: `setrange [key] [offset] [value]`

```bash
127.0.0.1:6379> set name "hello redis"
OK
127.0.0.1:6379> get name
"hello redis"
127.0.0.1:6379> setrange name 6 "cloudsimplify"
(integer) 19
127.0.0.1:6379> get name
"hello cloudsimplify"
```

### Redis Strings Encoding Types

- **`int`**: Used for strings that represent 64-bit signed integers.
- **`embstr`**: Used for strings that are 44 bytes or shorter; this encoding is more efficient for memory and performance.
- **`raw`**: Used for strings longer than 44 bytes.

❯ Returns the internal encoding of an object.

❯ Syntax: `object encoding [key]` 

```bash
127.0.0.1:6379> set number 12324353252
OK
127.0.0.1:6379> object encoding number
"int"
127.0.0.1:6379> set name "cloudsimplify"
OK
127.0.0.1:6379> object encoding name
"embstr"
127.0.0.1:6379> set text "object encoding returns the internal encoding for the redis object stored at key"
OK
127.0.0.1:6379> object encoding text
"raw"
```

❯ Serialized JSON data using set/get

```bash
127.0.0.1:6379> set json '{"fname" : "cloudsimplify", "lname" : "me"}'
OK
127.0.0.1:6379> get json
"{\"fname\" : \"cloudsimplify\", \"lname\" : \"me\"}"
```

Iterates over the key names in the database

Syntax: `scan [cursor/pagination] match [pattern(regex) count [count] type [type]` 

```bash
127.0.0.1:6379> debug populate 50
OK
127.0.0.1:6379> dbsize
(integer) 50
127.0.0.1:6379> scan 0     # Cursor start with 0
1) "2"                     # Next cursor/page number
2)  1) "key:10"            # Return 10 records (default)
    2) "key:46"
    3) "key:20"
    4) "key:17"
    5) "key:31"
    6) "key:35"
    7) "key:39"
    8) "key:9"
    9) "key:37"
   10) "key:22"
127.0.0.1:6379> scan 2
1) "22"
2)  1) "key:25"
    2) "key:41"
    3) "key:2"
    4) "key:11"
    5) "key:4"
    6) "key:26"
    7) "key:3"
    8) "key:1"
    9) "key:48"
   10) "key:28"
127.0.0.1:6379> scan 0 count 5    # count return 5 records instead of default 10 records
1) "24"
2) 1) "key:10"
   2) "key:46"
   3) "key:20"
   4) "key:17"
   5) "key:31"
127.0.0.1:6379> scan 0 count 5 match "key:1*"   # Scan only matching keys
1) "24"
2) 1) "key:10"
   2) "key:17"
127.0.0.1:6379> hset employee name "cloudsimplify" age "20"
(integer) 2
127.0.0.1:6379> scan 0 count 5 type hash        # scan only keys of type hash
1) "24"
2) (empty array)
127.0.0.1:6379> scan 24 count 5 type hash
1) "2"
2) (empty array)
127.0.0.1:6379> scan 2 count 5 type hash
1) "42"
2) (empty array)
127.0.0.1:6379> scan 42 count 5 type hash
1) "22"
2) (empty array)
127.0.0.1:6379> scan 22 count 5 type hash
1) "17"
2) 1) "employee"
```