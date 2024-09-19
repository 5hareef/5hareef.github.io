---
title: "Redis Sets"
date: Thu Sep 19 13:23:51 IST 2024
categories: [redis,sets]
tags: [redis]
image:
  path: /assets/img/redis/redis_sets.png
  alt: Redis Sets
---

# Redis - Sets

## Redis Sets

- **Sets** are collections of strings that are **unordered** and **unique**.
- They support standard mathematical operations like:
    - Intersection
    - Difference
    - Union
- Sets can be used to check if a member exists.
- A set can contain up to around 4 billion elements.

## Commands

```bash
sadd, smembers, sismember, smismember, scard, srem, spop, srandmember
```

❯ Adds one or more members to a set. Creates the key if it doesn't exist

❯ Syntax: `sadd [key] [member ...]` 

❯ Returns all members of a set

❯ Syntax: `smembers [key]`

❯ Determines whether a member belongs to a set

❯ Syntax: `sismember [key] [member]`

❯ Determines whether multiple members belong to a set

❯ Syntax: `smismember [key] [member ...]`

❯ Returns the number of members in a set

❯ Syntax: `scard [key]`

❯ Removes one or more members from a set. Deletes the set if the last member was removed

❯ Syntax: `srem [key] [member ...]`

❯ Returns one or more random members from a set after removing them. Deletes the set if the last member was popped

❯ Syntax: `spop [key] [count]` 

```bash
127.0.0.1:6379> sadd cars mazda bmw mercedes volkswagen toyota
(integer) 5
127.0.0.1:6379> smembers cars
1) "volkswagen"
2) "bmw"
3) "mazda"
4) "mercedes"
5) "toyota"
127.0.0.1:6379> sismember cars audi
(integer) 0                          # 0 means member does not exist
127.0.0.1:6379> sismember cars mercedes
(integer) 1
127.0.0.1:6379> scard cars
(integer) 5                          # Total members in a set
127.0.0.1:6379> srem cars toyota
(integer) 1
127.0.0.1:6379> smembers cars
1) "volkswagen"
2) "bmw"
3) "mazda"
4) "mercedes"
127.0.0.1:6379> spop cars
"bmw"
127.0.0.1:6379> scard cars
(integer) 3
127.0.0.1:6379> smembers cars
1) "volkswagen"
2) "mazda"
3) "mercedes"
127.0.0.1:6379> spop cars 3
1) "volkswagen"
2) "mazda"
3) "mercedes"
127.0.0.1:6379> smembers cars
(empty array)
127.0.0.1:6379> scard cars
(integer) 0
```

❯ Get one or multiple random members from a set

❯ Syntax: `srandmember [key] [count]` 

```bash
127.0.0.1:6379> sadd number 10 20 30 40 50 60 70 80 90 100
(integer) 10
127.0.0.1:6379> smembers number
 1) "10"
 2) "20"
 3) "30"
 4) "40"
 5) "50"
 6) "60"
 7) "70"
 8) "80"
 9) "90"
10) "100"
127.0.0.1:6379> srandmember number
"50"
127.0.0.1:6379> srandmember number
"100"
127.0.0.1:6379> srandmember number 2
1) "100"
2) "40"
127.0.0.1:6379>
```

❯ Moves a member from one set to another

❯ Syntax: `smove [source-key] [destination-key] [member-to-move-from-src]` 

```bash
127.0.0.1:6379> sadd order:pending 12 13 14 15
(integer) 4
127.0.0.1:6379> sadd order:completed 9 10 11
(integer) 3
127.0.0.1:6379> smembers order:pending
1) "12"
2) "13"
3) "14"
4) "15"
127.0.0.1:6379> smembers order:completed
1) "9"
2) "10"
3) "11"
127.0.0.1:6379> smove order:pending order:completed 12
(integer) 1
127.0.0.1:6379> smembers order:completed
1) "9"
2) "10"
3) "11"
4) "12"
```

❯ Returns the union of multiple sets.

❯ Syntax: `sunion [key1] [key2] ...` 

```bash
127.0.0.1:6379> sadd number:odd 1 3 5 7 9
(integer) 5
127.0.0.1:6379> sadd number:even 2 4 6 8 10
(integer) 5
127.0.0.1:6379> smembers number:odd
1) "1"
2) "3"
3) "5"
4) "7"
5) "9"
127.0.0.1:6379> smembers number:even
1) "2"
2) "4"
3) "6"
4) "8"
5) "10"
127.0.0.1:6379> sunion number:odd number:even
 1) "1"
 2) "2"
 3) "3"
 4) "4"
 5) "5"
 6) "6"
 7) "7"
 8) "8"
 9) "9"
10) "10"
127.0.0.1:6379>
```

❯ Stores the union of multiple sets in a key

❯ Syntax: `sunionstore [store_result_in_this_destination_set_key] [key1] [key2] ...` 

```bash
127.0.0.1:6379> sadd task:pending e f g h i
(integer) 5
127.0.0.1:6379> sadd task:completed a b c d
(integer) 4
127.0.0.1:6379> smembers task:pending
1) "f"
2) "h"
3) "g"
4) "e"
5) "i"
127.0.0.1:6379> smembers task:completed
1) "b"
2) "c"
3) "d"
4) "a"
127.0.0.1:6379> sunionstore all_tasks task:pending task:completed
(integer) 9
127.0.0.1:6379> smembers all_tasks
1) "b"
2) "f"
3) "d"
4) "i"
5) "h"
6) "g"
7) "e"
8) "c"
9) "a"
127.0.0.1:6379>
```

❯ Returns the intersect of multiple sets

❯ Syntax: `sinter [key1] [key2] ...`

❯ Stores the intersect of multiple sets in a key

❯ Syntax: `sinterstore [store_intersect_result_in_this_set_key] [key1] [key2] ...` 

```bash
127.0.0.1:6379> sadd streaming:netflix avengers mummy hellboy
(integer) 3
127.0.0.1:6379> sadd streaming:amazon mummy logan
(integer) 2
127.0.0.1:6379> sadd streaming:hulu logan mummy outlander
(integer) 3
127.0.0.1:6379> sinter streaming:netflix streaming:amazon streaming:hulu
1) "mummy"
127.0.0.1:6379> sinterstore streaming:common streaming:netflix streaming:amazon streaming:hulu
(integer) 1
127.0.0.1:6379> smembers streaming:common
1) "mummy"
```