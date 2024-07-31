---
title: "Redis Lists"
date: Wed Jul 31 12:50:37 IST 2024
categories: [redis,lists]
tags: [redis]
image:
  path: /assets/img/redis/redis_lists.png
  alt: Redis Strings
---

# Redis - Lists

- Lists are a flexible data structure in Redis.
- A list is a simple collection of elements.
- It stores a sequence of objects in an ordered manner.
- The order of elements is based on the sequence in which they are inserted.
- Lists can be encoded and optimized for memory.
- From a programming perspective, a list is similar to an array.
- Elements in a list are strings.
- A list can contain more than 4 billion elements.

## Commands

❯ Prepends one or more elements to a list. Creates the key if it doesn't exist

❯ Syntax: `lpush [key] [element ...]` 

❯ Appends one or more elements to a list. Creates the key if it doesn't exist.

❯ Syntax: `rpush [key] [element ...]` 

❯ Returns a range of elements from a list.

❯ Syntax: `lrange [key] [start-index] [end-index]` 

❯ Returns an element from a list by its index

❯ Syntax: `lindex [key] [element-index]`

```bash
127.0.0.1:6379> lpush cars mazda bmw audi
(integer) 3
127.0.0.1:6379> lrange cars 0 -1
1) "audi"
2) "bmw"
3) "mazda"
127.0.0.1:6379> rpush cars toyota skoda mercedes
(integer) 6
127.0.0.1:6379> lrange cars 0 -1
1) "audi"
2) "bmw"
3) "mazda"
4) "toyota"
5) "skoda"
6) "mercedes"
127.0.0.1:6379> lindex cars 2
"mazda"
127.0.0.1:6379> lindex cars -1
"mercedes"
127.0.0.1:6379> lindex cars -2
"skoda"
```

![redis_lists_image_illustration.png](assets/img/redis/redis_lists_image_illustration.png)
_Redis Lists Illustration_

❯ Inserts an element before or after another element in a list

❯ Syntax: `linsert [key] before|after [pivot] [element]` 

**Note:** The `pivot` is the value we need to check first in the list based on the `before|after` clause, and the `element` is the actual value we need to add.

```bash
127.0.0.1:6379> linsert cars before mazda honda
(integer) 7
127.0.0.1:6379> lrange cars 0 -1
1) "audi"
2) "bmw"
3) "honda"
4) "mazda"
5) "toyota"
6) "skoda"
7) "mercedes"
127.0.0.1:6379> linsert cars after skoda volkswagen
(integer) 8
127.0.0.1:6379>
127.0.0.1:6379> lrange cars 0 -1
1) "audi"
2) "bmw"
3) "honda"
4) "mazda"
5) "toyota"
6) "skoda"
7) "volkswagen"
8) "mercedes"
127.0.0.1:6379>
```

❯ Returns the first elements in a list after removing it. Deletes the list if the last element was popped

❯ Syntax: `lpop [key] [count]` 

❯ Returns and removes the last elements of a list. Deletes the list if the last element was popped

❯ Syntax: `rpop [key] [count]`

**Note:** `count` tell how many elements to delete from either left (`lpop`) or right (`rpop`)

```bash
127.0.0.1:6379> lrange cars 0 -1
1) "audi"
2) "bmw"
3) "honda"
4) "mazda"
5) "toyota"
6) "skoda"
7) "volkswagen"
8) "mercedes"
127.0.0.1:6379> lpop cars
"audi"
127.0.0.1:6379> lrange cars 0 -1
1) "bmw"
2) "honda"
3) "mazda"
4) "toyota"
5) "skoda"
6) "volkswagen"
7) "mercedes"
127.0.0.1:6379> rpop cars
"mercedes"
127.0.0.1:6379> lrange cars 0 -1
1) "bmw"
2) "honda"
3) "mazda"
4) "toyota"
5) "skoda"
6) "volkswagen"
127.0.0.1:6379> lpop cars 2
1) "bmw"
2) "honda"
127.0.0.1:6379> lrange cars 0 -1
1) "mazda"
2) "toyota"
3) "skoda"
4) "volkswagen"
```

❯ Removes elements from both ends a list. Deletes the list if all elements were trimmed

❯ Syntax: `ltrim [key] [start-index] [end-index]` 

**Note:** Anything that resides between the `start` and `end` range will be trimmed

```bash
127.0.0.1:6379> lpush numbers 1 2 3 4 5 6 7 8 9 10
(integer) 10
127.0.0.1:6379> lrange numbers 0 -1
 1) "10"
 2) "9"
 3) "8"
 4) "7"
 5) "6"
 6) "5"
 7) "4"
 8) "3"
 9) "2"
10) "1"
127.0.0.1:6379> ltrim numbers 2 6
OK
127.0.0.1:6379> lrange numbers 0 -1
1) "8"
2) "7"
3) "6"
4) "5"
5) "4"
127.0.0.1:6379>
```

❯ Sets the value of an element in a list by its index

❯ Syntax: `lset [key] [index-to-replace] [element]` 

```bash
127.0.0.1:6379> lpush cars mazda bmw audi skoda toyota nissan
(integer) 6
127.0.0.1:6379> lrange cars 0 -1
1) "nissan"
2) "toyota"
3) "skoda"
4) "audi"
5) "bmw"
6) "mazda"
127.0.0.1:6379> lset cars 2 mercedes
OK
127.0.0.1:6379> lrange cars 0 -1
1) "nissan"
2) "toyota"
3) "mercedes"
4) "audi"
5) "bmw"
6) "mazda"
127.0.0.1:6379> lset cars -1 volkswagen
OK
127.0.0.1:6379> lrange cars 0 -1
1) "nissan"
2) "toyota"
3) "mercedes"
4) "audi"
5) "bmw"
6) "volkswagen"
```

❯ Returns the length of a list

❯ Syntax: `llen [key]` 

```bash
127.0.0.1:6379> lpush cars mazda bmw audi skoda toyota nissan
(integer) 6
127.0.0.1:6379> lrange cars 0 -1
1) "nissan"
2) "toyota"
3) "skoda"
4) "audi"
5) "bmw"
6) "mazda"
127.0.0.1:6379> llen cars
(integer) 6
```

❯ Returns the index of matching elements in a list

❯ Syntax: `lpos [key] [element] rank [num] count [num] maxlen [num]`

Basically, it finds the occurrence of a matching element in a list, where `count` specifies the number of matches you want, `rank` determines which occurrence of the element to focus on, and `maxlen` limits the comparison to a specified maximum number of list items.

```bash
127.0.0.1:6379> lpush letters a a c d f g d h i f f
(integer) 11
127.0.0.1:6379> lrange letters 0 -1
 1) "f"
 2) "f"
 3) "i"
 4) "h"
 5) "d"
 6) "g"
 7) "f"
 8) "d"
 9) "c"
10) "a"
11) "a"
127.0.0.1:6379> lpos letters "f"
(integer) 0
127.0.0.1:6379> lpos letters "f" count 0
1) (integer) 0
2) (integer) 1
3) (integer) 6
127.0.0.1:6379> lpos letters "f" count 1
1) (integer) 0
127.0.0.1:6379> lpos letters "f" rank 2
(integer) 1
127.0.0.1:6379> lpos letters "f" count 1 rank 2
1) (integer) 1
127.0.0.1:6379> lpos letters "a" count 2  maxlen 10
1) (integer) 9
```

❯ Removes elements from a list. Deletes the list if the last element was removed

❯ Syntax: `lrem [key] [count] [element]`

Where: 

- count > 0 remove element from head to tail
- count < 0 remove element from tail to head
- count = 0 remove all elements equal to element

```bash
127.0.0.1:6379> lpush mykey "one" "two" "three" "one" "three" "four"
(integer) 6
127.0.0.1:6379> lrange mykey 0 -1
1) "four"
2) "three"
3) "one"
4) "three"
5) "two"
6) "one"
127.0.0.1:6379> lrem mykey 1 "four"   # Remove element "four" from head
(integer) 1
127.0.0.1:6379> lrange mykey 0 -1
1) "three"
2) "one"
3) "three"
4) "two"
5) "one"
127.0.0.1:6379> lrem mykey -1 "two"   # Remove element "two" from tail
(integer) 1
127.0.0.1:6379> lrange mykey 0 -1
1) "three"
2) "one"
3) "three"
4) "one"
127.0.0.1:6379> lrem mykey 0 "three"   # Remove all elements equal to "three"
(integer) 2
127.0.0.1:6379> lrange mykey 0 -1
1) "one"
2) "one"
```