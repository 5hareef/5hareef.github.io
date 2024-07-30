---
title: "Redis Data Management"
date: Tue Jul 30 15:51:53 IST 2024
categories: [redis,basic]
tags: [redis]
image:
  path: /assets/img/redis/redis_data_management.png
  alt: Redis Data Management
---

# Redis Data Management

- Redis uses a key-based data structure.
- It belongs to the family of databases called `key-value stores`.
- Data is stored as `key = value`.
- Values can be:
    - String
    - List
    - Hash
    - Set
    - And more...
- You can only retrieve data if you know the exact `key` used to store it.

## Commands

### Keys Naming Conventions

- **Keep it simple but robust**: Choose clear and meaningful names for your keys.
- **Avoid very short keys**: They can be unclear and hard to manage.
- **Use a logical schema**: Design your key names based on your database structure. For example, use `object-id:id`.
- **Maximum key size**: Keys can be up to 512MB in size.
- **Binary safe**: Redis keys can be any binary sequence, meaning they can include any characters.
- **Empty string**: An empty string is also a valid key in Redis.

![redis_naming_conventon.png](assets/img/redis/redis_naming_conventon.png)
_Redis Naming Convention_

❯ Set key

❯ Syntax: `set [key] [value]`

```bash
127.0.0.1:6379> set cloudsimplify:user:100:name github
OK
```

❯ Get key

❯ Syntax: `get [key]`

```bash
127.0.0.1:6379> get cloudsimplify:user:100:name
"github"
```

❯ Delete keys (Integer 1 means operation was successful)

❯ Syntax: `del [key...]`

```bash
127.0.0.1:6379> del cloudsimplify:user:100:name
(integer) 1
```

❯ Check if a key exists or not (Integer 0 means key not exists)

❯ Syntax: `exists [key...]`

```bash
127.0.0.1:6379> set cloudsimplify:user:100:name github
OK
127.0.0.1:6379> exists cloudsimplify:user:100:name
(integer) 1
127.0.0.1:6379> del cloudsimplify:user:100:name
(integer) 1
127.0.0.1:6379> exists cloudsimplify:user:100:name
(integer) 0
```

### How Redis Handles Key Expirations?

- **Normal keys**: These are created without any expiration time.
- **Expired keys**: These have expiration information stored as a Unix timestamp (in milliseconds).

### Redis expires keys in two ways:

1. **Passive Expiration**:
    - A key expires passively when a client tries to access it and it's found to have timed out.
2. **Active Expiration**:
    - Redis actively checks for expired keys 10 times per second by:
        - Testing 20 random keys that have expiration times.
        - Deleting any keys that are found to be expired.
        - If more than 25% of the tested keys are expired, Redis repeats the process.

❯ Define key with expiration

❯ Syntax: `set [key] [value] ex [ttl]` 

```bash
127.0.0.1:6379> set cloudsimplify:user:100:password secretpasswd ex 86400
OK
```

❯ Check ttl on key in sec

❯ Syntax: `ttl [key]` 

```bash
127.0.0.1:6379> ttl cloudsimplify:user:100:password
(integer) 86252
```

❯ Set expiration on key in sec

❯ Syntax: `expire [key] [ttl]` 

```bash
127.0.0.1:6379> expire cloudsimplify:user:100:password 300
(integer) 1
127.0.0.1:6379> ttl cloudsimplify:user:100:password
(integer) 297
```

❯ Set expiration on key in ms

❯ Syntax: `pexpire [key] [ttl(ms)]` 

```bash
127.0.0.1:6379> pexpire cloudsimplify:user:100:password 300000
(integer) 1
```

❯ Check ttl on key in ms

❯ Syntax: `pttl [key]` 

```bash
127.0.0.1:6379> pttl cloudsimplify:user:100:password
(integer) 297541
```

❯ Sets the string value and expiration time of a key. Creates the key if it doesn't exist

❯ Syntax: `setex [key] [ttl] [value]` 

```bash
127.0.0.1:6379> setex cloudsimplify:user:100:password 300 secretpasswd
OK
127.0.0.1:6379> ttl cloudsimplify:user:100:password
(integer) 297
```

❯ Sets both string value and expiration time in milliseconds of a key. The key is created if it doesn't exist

❯ Syntax: `psetex [key] [ttl(ms)] [value]` 

```bash
127.0.0.1:6379> psetex cloudsimplify:user:100:password 300000 secretpasswd
OK
127.0.0.1:6379> pttl cloudsimplify:user:100:password
(integer) 296337
127.0.0.1:6379> ttl cloudsimplify:user:100:password
(integer) 294
127.0.0.1:6379>
```

❯ Remove the expiration from the key (Integer 1 means expiration removed)

❯ Syntax: `persist [key]` 

```bash
127.0.0.1:6379> setex cloudsimplify:user:100:token 300 sometoken
OK
127.0.0.1:6379> ttl cloudsimplify:user:100:token
(integer) 287
127.0.0.1:6379> persist cloudsimplify:user:100:token
(integer) 1
127.0.0.1:6379> ttl cloudsimplify:user:100:token
(integer) -1
```

### Key Spaces

- **Key Spaces in Redis**: These are similar to database namespaces or schemas.
    - You can have the same key name in different key spaces.
    - You can manage keys separately in each key space.
    - Key spaces are indexed starting from 0.
    - Use the command `select [index]` to choose a key space.
    - This allows you to `select`, `get`, and control different parts of the database independently.

❯ Select key space

❯ Syntax: `select [index]` 

```bash
127.0.0.1:6379> set name cloudsimplify
OK
127.0.0.1:6379> get name
"cloudsimplify"
127.0.0.1:6379> select 1
OK
127.0.0.1:6379[1]> get name                 # You're now in key space index #1
(nil)
127.0.0.1:6379[1]> set name github
OK
127.0.0.1:6379[1]> get name
"github"
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> get name
"cloudsimplify"
```

### Key Patterns Matching

- **Key Patterns**:
    - `h?llo` matches `hello`, `hallo`, and `hxllo`.
    - `h*llo` matches `hllo` and `heeeello`.
    - `h[ae]llo` matches `hello` and `hallo`, but not `hillo`.
    - `h[^e]llo` matches `hallo`, `hbllo`, etc., but not `hello`.
    - `h[a-b]llo` matches `hallo` and `hbllo`.
- **Performance**:
    - Redis can scan a database with 1 million keys in 40 milliseconds on an entry-level laptop.
- **Usage Considerations**:
    - Use the `KEYS` command with extreme care in production environments, as it can severely impact performance when used on large databases.
    - Consider using the `SCAN` command as an alternative.

❯ Get all available keys

❯ Syntax: `keys *` 

```bash
127.0.0.1:6379> set hello 1
OK
127.0.0.1:6379> set hillo 2
OK
127.0.0.1:6379> set hallo 3
OK
127.0.0.1:6379> set hbllo 4
OK
127.0.0.1:6379> set hllo 5
OK
127.0.0.1:6379> set heeello 6
OK
127.0.0.1:6379> keys *
1) "hillo"
2) "hllo"
3) "hallo"
4) "hbllo"
5) "hello"
6) "heeello"
127.0.0.1:6379> keys h?llo
1) "hillo"
2) "hallo"
3) "hbllo"
4) "hello"
127.0.0.1:6379> keys * h*llo
(error) ERR wrong number of arguments for 'keys' command
127.0.0.1:6379> keys h*llo
1) "hillo"
2) "hllo"
3) "hallo"
4) "hbllo"
5) "hello"
6) "heeello"
127.0.0.1:6379> keys h[ae]llo
1) "hallo"
2) "hello"
127.0.0.1:6379> keys h[^e]llo
1) "hillo"
2) "hallo"
3) "hbllo"
127.0.0.1:6379> keys h[a-b]llo
1) "hallo"
2) "hbllo"
```

❯ Renames a key and overwrites the destination.

❯ Syntax: `rename [key] [newkey]` 

```bash
127.0.0.1:6379> set fname cloudsimplify
OK
127.0.0.1:6379> set lname me
OK
127.0.0.1:6379> rename fname newfname
OK
127.0.0.1:6379> get fname
(nil)
127.0.0.1:6379> get newfname
"cloudsimplify"
127.0.0.1:6379> set fname github
OK
127.0.0.1:6379> get fname
"github"
127.0.0.1:6379> rename newfname fname # If newkey already exists it is overwritten, when this happens RENAME executes an implicit DEL operation
OK
127.0.0.1:6379> get fname             # fname is now overwritten to newfname value
"cloudsimplify"
```

❯ Renames a key only when the target key name doesn't exist.

❯ Syntax: `renamenx [key] [newkey]` 

```bash
127.0.0.1:6379> set fname cloudsimplify
OK
127.0.0.1:6379> set lname me
OK
127.0.0.1:6379> renamenx fname newfname
(integer) 1
127.0.0.1:6379> get fname
(nil)
127.0.0.1:6379> get newfname
"cloudsimplify"
127.0.0.1:6379> set fname github
OK
127.0.0.1:6379> renamenx newfname fname   # returns an error cause key fname exist
(integer) 0
127.0.0.1:6379> get fname
"github"
```

❯ Asynchronously deletes one or more keys

❯ Syntax: `unlink [key...]` 

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> set k3 v3
OK
127.0.0.1:6379> unlink k1 k2
(integer) 2
127.0.0.1:6379> get k1
(nil)
127.0.0.1:6379> get k2
(nil)
127.0.0.1:6379> get k3
"v3"
```

❯ Determines the type of value stored at a key

❯ Syntax: `type [key]` 

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> lpush numbers 1 2 3 4 5
(integer) 5
127.0.0.1:6379> hset users fname cloudsimplify lname me
(integer) 2
127.0.0.1:6379> sadd cars mazda bmw audi
(integer) 3
127.0.0.1:6379> type k1
string
127.0.0.1:6379> type users
hash
127.0.0.1:6379> type numbers
list
127.0.0.1:6379> type cars
set
```