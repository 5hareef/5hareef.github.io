---
title: "Redis - Client"
date: Tue Dec 17 13:04:59 GMT 2024
categories: [redis,cluster]
tags: [redis]
image:
  path: /assets/img/redis/redis_cluster.png
  alt: Redis Cluster
---

# Redis Cluster

Redis Cluster is a distributed implementation of Redis that automatically partitions data across multiple nodes and provides fault tolerance. Here’s a guide to understanding various Redis Cluster commands and what they do.

### **1. `CLUSTER NODES`**

This command provides detailed information about all the nodes in the cluster. When you run it, Redis returns a list of all cluster nodes, their roles, and their states.

**Example**

```bash
127.0.0.1:6379> cluster nodes
004a2ba5d1a19fe28ef3f6397261ee7246f7cbe0 172.27.0.3:6379@16379 master - 0 1734437046074 3 connected 10923-16383
59a39b18368aac76e9e373d928f4dd10783193f5 172.27.0.6:6379@16379 slave 004a2ba5d1a19fe28ef3f6397261ee7246f7cbe0 0 1734437045000 3 connected
2a23e44ab6ec167219b74da9f3c7dc7af79a6ea8 172.27.0.7:6379@16379 slave a7425fa5cb1c211642276cfa90e54930da8568bf 0 1734437048146 2 connected
5a6a43f5a5f6216c10a5fcee212accfa1cebd365 172.27.0.5:6379@16379 myself,master - 0 1734437047000 1 connected 0-5460
e60f7cefdc90eb721944009e22d30b8417ff1553 172.27.0.4:6379@16379 slave 5a6a43f5a5f6216c10a5fcee212accfa1cebd365 0 1734437045038 1 connected
a7425fa5cb1c211642276cfa90e54930da8568bf 172.27.0.2:6379@16379 master - 0 1734437047109 2 connected 5461-10922
```

**Where:**

- **Node ID**: A unique identifier for each node (e.g., `c37ba469d276...`).
- **Address**: The IP and port of the node (e.g., `127.0.0.1:6379`).
    - The address of the node. `@16379` represents the inter-node communication port.
- **Role**: Whether the node is a `master` or `slave` (replica).
- **Slots**: Range of hash slots the node is responsible for (e.g., `0-5460`).

**Use Cases**

- To check the cluster topology and the role of each node.
- To debug issues like unreachable nodes or incorrect configuration.

### **2. `redis-cli --cluster check 127.0.0.1:6379`**

This command is part of the Redis CLI's cluster management utility. It verifies the health of the cluster by connecting to a specific node and performing a series of checks.

**Key points**

- Ensures all nodes in the cluster are reachable.
- Verifies that slots are properly distributed and covered.
- Checks for misconfigurations, such as missing replicas.

**Output**

```bash
❯ redis-cli --cluster check 127.0.0.1:6379
127.0.0.1:6379 (5a6a43f5...) -> 0 keys | 5461 slots | 1 slaves.
172.27.0.3:6379 (004a2ba5...) -> 0 keys | 5461 slots | 1 slaves.
172.27.0.2:6379 (a7425fa5...) -> 0 keys | 5462 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: 5a6a43f5a5f6216c10a5fcee212accfa1cebd365 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 004a2ba5d1a19fe28ef3f6397261ee7246f7cbe0 172.27.0.3:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 59a39b18368aac76e9e373d928f4dd10783193f5 172.27.0.6:6379
   slots: (0 slots) slave
   replicates 004a2ba5d1a19fe28ef3f6397261ee7246f7cbe0
S: 2a23e44ab6ec167219b74da9f3c7dc7af79a6ea8 172.27.0.7:6379
   slots: (0 slots) slave
   replicates a7425fa5cb1c211642276cfa90e54930da8568bf
S: e60f7cefdc90eb721944009e22d30b8417ff1553 172.27.0.4:6379
   slots: (0 slots) slave
   replicates 5a6a43f5a5f6216c10a5fcee212accfa1cebd365
M: a7425fa5cb1c211642276cfa90e54930da8568bf 172.27.0.2:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

**Use Cases**

- To confirm that your cluster is healthy after setup.
- To troubleshoot slot misconfigurations or network issues.

### **3. `CLUSTER SLOTS`**

This command displays how hash slots are distributed among the nodes in the cluster. It’s an overview of which slots belong to which master and their associated replicas.

**Example**

```bash
127.0.0.1:6379> cluster slots
1) 1) (integer) 0
   2) (integer) 5460
   3) 1) "172.27.0.5"
      2) (integer) 6379
      3) "5a6a43f5a5f6216c10a5fcee212accfa1cebd365"
      4) (empty array)
   4) 1) "172.27.0.4"
      2) (integer) 6379
      3) "e60f7cefdc90eb721944009e22d30b8417ff1553"
      4) (empty array)
...
```

**Where**

- **Slot Range**: The hash slots managed by each master node (e.g., `0-5460`).
- **Node Address**: The master node responsible for the slots.
- **Replica Nodes**: If replicas are set up, their addresses are also listed.

**Use Cases**

- To visualize how data is distributed across the cluster.
- To locate specific slots for debugging.

### **4. `CLUSTER INFO`**

This command provides statistics and status information about the entire cluster.

**Example**

```bash
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:297
cluster_stats_messages_pong_sent:294
cluster_stats_messages_sent:591
cluster_stats_messages_ping_received:289
cluster_stats_messages_pong_received:297
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:591
total_cluster_links_buffer_limit_exceeded:0
```

**Key Metrics**

- **`cluster_state`**: Shows if the cluster is healthy (`ok`) or has issues (`fail`).
- **`cluster_slots_assigned`**: Total hash slots assigned to nodes (should always be `16384` in a functional cluster).
- **`cluster_slots_pfail/fail`**: Slots with nodes in pre-failure or failure state.
- **`cluster_known_nodes`**: Number of nodes in the cluster.
- **`cluster_size`**: Number of master nodes.

**Use Cases**

- To monitor cluster health.
- To quickly see if any slots or nodes are failing.

### **5. `CLUSTER MYID`**

This command returns the unique ID of the node you’re connected to.

**Example**

```bash
127.0.0.1:6379> cluster myid
"5a6a43f5a5f6216c10a5fcee212accfa1cebd365"
```

**Use Cases**

- To identify the current node's role in the cluster.
- Useful in debugging to match the node ID with the output of `CLUSTER NODES`.

### **6. `CLUSTER REPLICAS node_id`**

This command lists all the replicas for a specific master node in the cluster. You provide the master’s node ID as an argument.

**Example**

```bash
127.0.0.1:6379> cluster replicas 5a6a43f5a5f6216c10a5fcee212accfa1cebd365
1) "e60f7cefdc90eb721944009e22d30b8417ff1553 172.27.0.4:6379@16379 slave 5a6a43f5a5f6216c10a5fcee212accfa1cebd365 0 1734437164427 1 connected"
```

**Where**

- **Replica Node ID**: The unique identifier for the replica.
- **Address**: The IP and port of the replica.

**Use cases**

- To identify the replicas of a specific master.
- Useful for setting up monitoring or verifying replication.

### **7. `CLUSTER KEYSLOT key_name`**

This command calculates the hash slot for a specific key. Redis Cluster uses the hash slot to determine which node stores the key.

**Example**

```bash
127.0.0.1:6379> set name cloudsimplify
-> Redirected to slot [5798] located at 172.27.0.2:6379
OK
172.27.0.2:6379> cluster keyslot name
(integer) 5798
```

Redis calculates a CRC16 hash of the key, which is then mapped to one of the 16,384 slots.

**Use Cases**

- To debug where a key is stored in the cluster.
- To understand how Redis partitions data.

### **8. `CLUSTER GETKEYSINSLOT slot_num count`**

This command lists keys that belong to a specific hash slot. You also specify how many keys to return (`count`).

**Example**

```bash
172.27.0.2:6379> set age 1
-> Redirected to slot [741] located at 172.27.0.5:6379
OK
172.27.0.5:6379> keys *
1) "age"
172.27.0.5:6379> cluster getkeysinslot 741 1
1) "age"
```

### **Use Cases**

- To find out which keys are stored in a particular slot.
- Useful for debugging slot-specific data or migration issues.

Redis Cluster commands provide powerful tools to manage and monitor distributed data. By understanding these commands:

- You can ensure your cluster is properly configured and healthy.
- You can locate specific keys, slots, and nodes for troubleshooting.
- You can gain deeper insights into how Redis partitions and replicates data.