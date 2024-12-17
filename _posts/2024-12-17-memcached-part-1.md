---
title: "Memcached - Part-1"
date: Tue Dec 17 14:35:45 GMT 2024
categories: [memcached,operation]
tags: [memcached]
image:
  path: /assets/img/memcached/memcached-part-1.png
  alt: Memcached Part-1
---

# Memcached - Part 1

### **Basic Operations:**

Use a Memcached client or `nc` to store and retrieve data.

- The **`set`** command is used to store a key-value pair in Memcached. Syntax:
    
    ```bash
    set <key> <flags> <exptime> <bytes> [noreply]
    <data>
    ```
    
    | Parameter | Description |
    | --- | --- |
    | `key` | The unique identifier for the data  |
    | `flags` | Arbitrary 32-bit integer set by the client (commonly used for metadata; defaults to `0`  |
    | `expire_time` | Expiration time of the key in seconds: 
    |               | `0` means item will never expire. |
    | `positive value` | means expiration time in seconds from now |
    | `bytes` | The size of the value in bytes |
    | `noreply` | Suppresses the `STORED` response for this command (Optional) |
    | `data` | The actual value to be stored. |
    
    - Example:
    
    ```bash
    ❯ echo -e "set name 0 60 5\r\ncache\r\nquit" | nc localhost 11211
    STORED
    ```
    
- The **`get`** command retrieves the value of a given key. Syntax:
    
    ```bash
    get <key> [<key> ...]
    
    # `key`: The key(s) to retrieve. 
    # Note: You can request multiple keys separated by spaces.
    ```
    
    - Example:
    
    ```bash
    # If key exits:
    ❯ echo -e "get name\r\nquit" | nc localhost 11211
    VALUE name 0 5    # VALUE <key> <flags> <bytes>
    cache             # <data>
    END
    
    # If the key does not exist:
    ❯ echo -e "get name\r\nquit" | nc localhost 11211
    END
    ```
    

### Memcached Advanced Operation

- The **`CAS`** stands for **`Check-And-Set`** (or Compare-And-Swap). It is a Memcached operation used to ensure that data is only updated if it hasn't been modified by another client since it was retrieved. This helps in scenarios where multiple clients might simultaneously update the same key, preventing overwrites or stale updates. Syntax:
    
    ```bash
    cas <key> <flags> <expire_time> <bytes> <cas_token> [noreply]
    <data>
    ```
    
    | Parameter | Description |
    | --- | --- |
    | `key` | The key to be updated |
    | `flags` | Metadata associated with the item (similar to `set`) |
    | `expire_time` | Expiration time for the key |
    | `bytes` | Size of the value in bytes |
    | `cas_token` | The unique token retrieved via the `gets` command |
    | `noreply` | Suppresses the response from the server (Optional) |
    | `data` | The new value to store |

    - Example
    
    ```bash
    # Store a value
    ❯ echo -e "set name 0 300 5\r\ncache\r\nquit" | nc localhost 11211
    STORED
    
    # Retrieve it with gets
    ❯ echo -e "gets name\r\nquit" | nc localhost 11211
    VALUE name 0 5 132937     # Here CAS token = 132937
    cache
    END
    
    # Update the value with cas
    ❯ echo -e "cas name 0 300 5 132937\r\nhello\r\nquit" | nc localhost 11211
    STORED
    ❯ echo -e "get name\r\nquit" | nc localhost  11211
    VALUE name 0 5
    hello
    END
    
    # If another client updates the value after your CAS operation
    ❯ echo -e "cas name 0 300 5 132937\r\nhello\r\nquit" | nc localhost 11211
    EXISTS
    ```
    
- The **`lru_crawler metadump`** command in Memcached triggers the **`LRU (Least Recently Used) crawler`** to dump metadata about all the keys currently stored in the cache. This is particularly useful for inspecting what is stored in Memcached without retrieving the full key values.
    
    The `lru_crawler metadump all` command:
    
    - Dumps metadata for **all items** across all slabs.
    - Does **not dump the values** of the keys.
    - Outputs key statistics such as size, expiration, and CAS (Check-And-Set) tokens.
    - Example:
    
    ```bash
    ❯ echo "lru_crawler metadump all" | nc -w5 localhost 11211
    key=key_1 exp=16777221 la=16777122 cas=5 fetch=no cls=3 size=50 flags=3
    key=key_2 exp=16777240 la=16777100 cas=7 fetch=yes cls=4 size=70 flags=2
    END
    ```
    
    | Fields | Description |
    | --- | --- |
    | `key` | The name of the key stored in Memcached |
    | `exp` | The expiration time of the key as a Unix timestamp. `0` indicates no expiration (key persists indefinitely) |
    | `la` | The last time the key was accessed, in Unix timestamp format. Useful for identifying stale or rarely accessed items |
    | `cas` | The CAS (Check-And-Set) token associated with the key. Helps track changes to the item for concurrency control |
    | `fetch` | Indicates whether the item has been fetched recently (`yes`) or not (`no`) |
    | `cls` | The slab class in which the key is stored. Slabs are memory chunks grouped by object sizes |
    | `size` | The total size of the key and its metadata, in bytes |
    | `END` | Indicates the end of the output |

### Memcached Statistics

1. The `stats` command in Memcached provides a detailed set of statistics about the server's performance and usage.
    
    ```bash
    ❯ echo -e "stats\r\nquit" | nc localhost 11211
    STAT pid 1
    STAT uptime 92791
    STAT time 1733841314
    STAT version 1.6.32
    STAT max_connections 2048
    STAT curr_connections 10
    STAT total_connections 27856
    STAT cmd_get 9573
    STAT cmd_set 1920
    STAT get_hits 1844
    STAT get_misses 7729
    STAT bytes_read 64620281
    STAT bytes_written 75356348
    STAT bytes 633128
    STAT curr_items 12
    STAT total_items 1897
    STAT evictions 0
    ...
    END
    ```
    
    **Key metrics to observe:**
    
    | Key | Description |
    | --- | --- |
    | `pid` | The process ID of the memcached server process. |
    | `uptime` | The amount of time (in seconds) the server has been running since its last restart. |
    | `time` | The current UNIX timestamp according to the server. |
    | `version` | The version of memcached that is running. |
    | `curr_items` | The current number of items stored in the cache. This gives a snapshot of the cache's usage at the moment. |
    | `total_items` | The total number of items that have been stored in the cache since the server started. This helps understand the cache's throughput over time. |
    | `bytes` | The current number of bytes used by all the items in the cache. |
    | `curr_connections` | The current number of open connections to the memcached server. |
    | `total_connections` | The total number of connections that have been opened since the server started. |
    | `cmd_get` | The cumulative number of retrieval commands (get) issued to the server. Helps gauge read load. |
    | `cmd_set` | The cumulative number of storage commands (set) issued to the server. Helps gauge write load. |
    | `get_hits` | The number of successful get commands (cache hits). Indicates how often data is successfully retrieved from the cache. |
    | `get_misses` | The number of get commands that failed to find the requested data (cache misses). Helps measure the cache hit rate. |
    | `evictions` | The number of items removed from the cache to free up memory for new items. |
    | `bytes_read` | The total number of bytes read by the server from network connections. Useful for monitoring network I/O. |
    | `bytes_written` | The total number of bytes written by the server to network connections. Also useful for monitoring network I/O. |
2. The `stats items` command in Memcached provides detailed statistics about each slab class. Memcached organizes items into slab classes based on their size, which helps in efficient memory allocation and management.
    
    ```bash
    ❯  echo -e "stats items\r\nquit" | nc localhost 11211
    STAT items:15:number 1
    STAT items:15:age 1679
    STAT items:15:evicted 0
    STAT items:15:evicted_nonzero 0
    STAT items:15:evicted_time 0
    STAT items:15:outofmemory 0
    STAT items:15:tailrepairs 0
    STAT items:15:reclaimed 175
    ...
    ```
    
    **Key metrics to observe:**
    
    | Key | Description |
    | --- | --- |
    | `items:<slab_id>:number` | The number of items currently stored in the specified slab class. Each `slab_id` corresponds to a slab class handling items of a particular size range. |
    | `items:<slab_id>:age` | The age (in seconds) of the oldest item in the specified slab class. This indicates how long items have been in the cache without being evicted. |
    | `items:<slab_id>:evicted` | The total number of items evicted from the specified slab class since the server started. High eviction rates may indicate that the cache size is too small. |
    | `items:<slab_id>:evicted_nonzero` | The number of items evicted from the specified slab class that had a non-zero expiration time. This can help distinguish between items evicted due to memory pressure versus those evicted because they expired. |
    | `items:<slab_id>:evicted_time` | The time (in seconds) since the last item was evicted from the specified slab class. This can help you understand the recency of evictions. |
    | `items:<slab_id>:outofmemory` | The number of times an item could not be stored in the specified slab class because the class was out of memory. This helps identify if memory allocation issues are affecting performance. |
    | `items:<slab_id>:reclaimed` | The number of expired items reclaimed in the specified slab class. Reclaimed items are those that were overwritten by new items because they had expired. |

### Memcached Key Dump

Memcached does not provide a built-in command to dump all keys. However, the keys can be retrieved by iterating through the slab classes and dumping the keys for each slab. This can be done using a combination of the `stats items` (covered above) and `stats cachedump` commands.

Below is a step-by-step command using netcat to dump all memcached keys.

```bash
# Get the list of slab classes
echo -e "stats items\r\nquit"  | nc localhost 11211
STAT items:12:number 2
STAT items:12:number_hot 0
STAT items:12:number_warm 0
STAT items:12:number_cold 2
...

# For each slab class, use stats cachedump to retrieve keys:
# syntax: stats cachedump <slab_id> <limit>
# where
# <slab_id>: The ID of the slab class.
# <limit>: The maximum number of keys to retrieve (use 0 to get all keys).
echo -e "stats cachedump 12 0\r\nquit" | nc localhost 11211
ITEM xxxxxxxxxxxxxxxxxxxxxxxxxxxxx [902 b; 1736272214 s]
ITEM xxxxxxxxxxxxxxxxxxxxxxxxxxxxx [882 b; 1736271757 s]
END
```

### Reference

[memcached/doc/protocol.txt at master · memcached/memcached](https://github.com/memcached/memcached/blob/master/doc/protocol.txt)
