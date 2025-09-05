# Redis Clustering

Redis clustering is like having multiple Redis servers work together as one big team. Instead of one server doing all the work, you have several servers sharing the load, making everything faster and more reliable.

## What is Redis Clustering?

Think of Redis clustering like:

- **Restaurant Chain:** Multiple locations serving the same menu
- **Library System:** Multiple branches with different books, but one catalog
- **Team Project:** Each person handles different parts, but work together
- **Post Office:** Multiple branches, each handles certain zip codes

In Redis clustering, data is automatically split across multiple servers (called nodes).

## Why Use Clustering?

### Problems with Single Redis Server

```
+-------------------+
|   Single Redis    |  <- All data here (risky!)
|     Server        |  <- All requests here (slow!)
+-------------------+
```

**Problems:**

- If server crashes, you lose everything
- One server handles all requests (can get overwhelmed)
- Limited by one machine's memory and processing power

### Solution with Clustering

```
+-------------+  +-------------+  +-------------+
|   Node 1    |  |   Node 2    |  |   Node 3    |
|  Keys A-F   |  |  Keys G-M   |  |  Keys N-Z   |
+-------------+  +-------------+  +-------------+
```

**Benefits:**

- Data is spread across multiple servers
- If one server fails, others keep working
- More servers = can handle more requests
- Can add more servers when you need them

## How Data is Distributed

Redis uses "hash slots" to decide which server stores which data:

### Hash Slots Explained

```redis
# Redis calculates which "slot" a key belongs to
# There are 16,384 slots total (0 to 16,383)

# Example key assignments:
user:123    → Slot 5,432  → Node 1
user:456    → Slot 12,891 → Node 3  
user:789    → Slot 2,107  → Node 1
```

**How it works:**

1. Redis takes your key (like "user:123")
2. Calculates a hash number (like 5,432)
3. That number determines which server stores the data
4. All requests for that key always go to the same server

## Basic Cluster Setup Example

### Simple 3-Node Cluster

```bash
# Create directories for 3 nodes
mkdir cluster-test
cd cluster-test
mkdir 7000 7001 7002

# Create config for Node 1 (7000)
echo "port 7000
cluster-enabled yes
cluster-config-file nodes-7000.conf
cluster-node-timeout 5000" > 7000/redis.conf

# Create config for Node 2 (7001)
echo "port 7001
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 5000" > 7001/redis.conf

# Create config for Node 3 (7002)
echo "port 7002
cluster-enabled yes
cluster-config-file nodes-7002.conf
cluster-node-timeout 5000" > 7002/redis.conf
```

### Starting the Nodes

```bash
# Start each node in separate terminals
redis-server 7000/redis.conf
redis-server 7001/redis.conf
redis-server 7002/redis.conf

# Create the cluster
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 --cluster-replicas 0
```

**Config Explanation:**

- `cluster-enabled yes` = Turn on cluster mode
- `cluster-config-file` = Where Redis saves cluster info
- `cluster-node-timeout` = How long to wait before marking a node as down
- `--cluster-replicas 0` = No backup copies (for simplicity)

## Python Client for Clusters

```python
from rediscluster import RedisCluster

# Connect to the cluster
startup_nodes = [
    {"host": "127.0.0.1", "port": "7000"},
    {"host": "127.0.0.1", "port": "7001"},
    {"host": "127.0.0.1", "port": "7002"}
]

# Create cluster connection
rc = RedisCluster(startup_nodes=startup_nodes, decode_responses=True)

def test_cluster_operations():
    """Test basic operations on a Redis cluster"""
    
    print("=== TESTING CLUSTER OPERATIONS ===")
    
    # Store data (automatically goes to correct node)
    rc.set("user:alice", "Alice Smith")
    rc.set("user:bob", "Bob Jones") 
    rc.set("user:charlie", "Charlie Brown")
    
    print("✅ Stored 3 users in cluster")
    
    # Retrieve data (automatically finds correct node)
    alice = rc.get("user:alice")
    bob = rc.get("user:bob")
    charlie = rc.get("user:charlie")
    
    print(f"📋 Retrieved: {alice}, {bob}, {charlie}")
    
    # Add to different data structures
    rc.lpush("messages:alice", "Hello!")
    rc.lpush("messages:alice", "How are you?")
    rc.zadd("scores", {"alice": 100, "bob": 85, "charlie": 92})
    
    # Get the data back
    messages = rc.lrange("messages:alice", 0, -1)
    scores = rc.zrange("scores", 0, -1, withscores=True)
    
    print(f"💬 Alice's messages: {messages}")
    print(f"🏆 Game scores: {scores}")
    
    return True

def show_cluster_info():
    """Show which node handles which keys"""
    
    print("\n=== CLUSTER KEY DISTRIBUTION ===")
    
    keys_to_test = ["user:alice", "user:bob", "user:charlie", "messages:alice", "scores"]
    
    for key in keys_to_test:
        # Get cluster info for this key
        slot = rc.cluster_keyslot(key)
        nodes = rc.cluster_nodes()
        
        # Find which node handles this slot
        for node_id, node_info in nodes.items():
            if 'master' in node_info['flags']:
                slot_range = node_info.get('slots', [])
                for slot_start, slot_end in slot_range:
                    if slot_start <= slot <= slot_end:
                        host = node_info['host']
                        port = node_info['port']
                        print(f"🔑 Key '{key}' → Slot {slot} → Node {host}:{port}")
                        break

def handle_node_failure_demo():
    """Demo what happens when a node goes down"""
    
    print("\n=== NODE FAILURE HANDLING ===")
    
    try:
        # Try to access data
        result = rc.get("user:alice")
        print(f"✅ Successfully retrieved: {result}")
        
        # The cluster automatically handles node failures
        # If a node goes down, Redis will:
        # 1. Mark it as failed
        # 2. Redirect requests to other nodes
        # 3. Try to maintain data availability
        
        print("ℹ️  If a node fails, cluster redirects to available nodes")
        
    except Exception as e:
        print(f"❌ Error accessing cluster: {e}")

# Run demos
test_cluster_operations()
show_cluster_info()
handle_node_failure_demo()
```

**Sample Output:**
```
=== TESTING CLUSTER OPERATIONS ===
✅ Stored 3 users in cluster
📋 Retrieved: Alice Smith, Bob Jones, Charlie Brown
💬 Alice's messages: ['How are you?', 'Hello!']
🏆 Game scores: [('bob', 85.0), ('charlie', 92.0), ('alice', 100.0)]

=== CLUSTER KEY DISTRIBUTION ===
🔑 Key 'user:alice' → Slot 8734 → Node 127.0.0.1:7001
🔑 Key 'user:bob' → Slot 6257 → Node 127.0.0.1:7000
🔑 Key 'user:charlie' → Slot 2467 → Node 127.0.0.1:7000
🔑 Key 'messages:alice' → Slot 11078 → Node 127.0.0.1:7002
🔑 Key 'scores' → Slot 8852 → Node 127.0.0.1:7001

=== NODE FAILURE HANDLING ===
✅ Successfully retrieved: Alice Smith
ℹ️  If a node fails, cluster redirects to available nodes
```

## Code Explanation

Let me explain the important parts:

### Cluster Connection

```python
startup_nodes = [
    {"host": "127.0.0.1", "port": "7000"},
    {"host": "127.0.0.1", "port": "7001"},
    {"host": "127.0.0.1", "port": "7002"}
]
rc = RedisCluster(startup_nodes=startup_nodes, decode_responses=True)
```

**What this does:**

- `startup_nodes` = List of cluster nodes to connect to
- `RedisCluster()` = Special client that understands clustering
- Client automatically discovers all nodes in the cluster
- You only need to provide a few node addresses to start

### Automatic Key Routing

```python
rc.set("user:alice", "Alice Smith")  # Goes to correct node automatically
alice = rc.get("user:alice")         # Finds data on correct node automatically
```

**How it works:**

1. Client calculates which slot the key belongs to
2. Client knows which node handles that slot
3. Client sends command directly to correct node
4. No need to manually specify which server to use

### Slot Calculation

```python
slot = rc.cluster_keyslot(key)  # Calculate slot number for a key
```

**What happens:**

- Redis uses CRC16 hash algorithm
- Takes key name and calculates a number 0-16,383
- Same key always gets same slot number
- Slot number determines which node stores the data

## Cluster Management Commands

### Check Cluster Status

```redis
# Connect to any cluster node
redis-cli -c -p 7000

# Check cluster status
127.0.0.1:7000> CLUSTER INFO
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:3
cluster_size:3

# See all nodes
127.0.0.1:7000> CLUSTER NODES
a1b2c3... 127.0.0.1:7000@17000 myself,master - 0 0 1 connected 0-5460
d4e5f6... 127.0.0.1:7001@17001 master - 0 1693939201 2 connected 5461-10922
g7h8i9... 127.0.0.1:7002@17002 master - 0 1693939202 3 connected 10923-16383
```

**Explanation:**

- `cluster_state:ok` = Cluster is working properly
- `cluster_slots_assigned:16384` = All slots are assigned to nodes
- `cluster_known_nodes:3` = 3 nodes in the cluster
- Each node shows its slot range (like 0-5460)

### Adding a New Node

```python
def add_new_node_demo():
    """Example of adding a new node to cluster"""
    
    print("=== ADDING NEW NODE TO CLUSTER ===")
    
    # Steps to add a new node:
    print("1. Start new Redis instance on port 7003")
    print("2. Add it to cluster:")
    print("   redis-cli --cluster add-node 127.0.0.1:7003 127.0.0.1:7000")
    print("3. Rebalance slots:")
    print("   redis-cli --cluster rebalance 127.0.0.1:7000")
    
    print("\nℹ️  After adding, data will be redistributed automatically")

add_new_node_demo()
```

**Sample Output:**
```
=== ADDING NEW NODE TO CLUSTER ===
1. Start new Redis instance on port 7003
2. Add it to cluster:
   redis-cli --cluster add-node 127.0.0.1:7003 127.0.0.1:7000
3. Rebalance slots:
   redis-cli --cluster rebalance 127.0.0.1:7000

ℹ️  After adding, data will be redistributed automatically
```

## High Availability with Replicas

### Adding Backup Nodes

```bash
# Create cluster with backups (replicas)
redis-cli --cluster create \
  127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1
```

**What this creates:**

- 3 master nodes (handle reads and writes)
- 3 replica nodes (backup copies)
- Each master has 1 replica
- If a master fails, its replica takes over

```python
def explain_replicas():
    """Explain how replicas work"""
    
    print("=== HOW REPLICAS WORK ===")
    print("Master Nodes (handle requests):")
    print("  🖥️  Node 7000: Slots 0-5460")
    print("  🖥️  Node 7001: Slots 5461-10922") 
    print("  🖥️  Node 7002: Slots 10923-16383")
    print()
    print("Replica Nodes (backup copies):")
    print("  💾 Node 7003: Backup of Node 7000")
    print("  💾 Node 7004: Backup of Node 7001")
    print("  💾 Node 7005: Backup of Node 7002")
    print()
    print("If Node 7000 fails:")
    print("  ❌ Node 7000 goes down")
    print("  ✅ Node 7003 becomes new master")
    print("  ✅ Cluster keeps working!")

explain_replicas()
```

**Sample Output:**
```
=== HOW REPLICAS WORK ===
Master Nodes (handle requests):
  🖥️  Node 7000: Slots 0-5460
  🖥️  Node 7001: Slots 5461-10922
  🖥️  Node 7002: Slots 10923-16383

Replica Nodes (backup copies):
  💾 Node 7003: Backup of Node 7000
  💾 Node 7004: Backup of Node 7001
  💾 Node 7005: Backup of Node 7002

If Node 7000 fails:
  ❌ Node 7000 goes down
  ✅ Node 7003 becomes new master
  ✅ Cluster keeps working!
```

## Important Limitations

### Multi-Key Operations

```python
def cluster_limitations_demo():
    """Show what works and what doesn't in clusters"""
    
    print("=== CLUSTER LIMITATIONS ===")
    
    # ✅ This works - single key operations
    rc.set("user:123", "Alice")
    rc.get("user:123")
    print("✅ Single key operations work fine")
    
    # ✅ This works - keys with same hash tag
    rc.mset({"user:{123}:name": "Alice", "user:{123}:age": "25"})
    print("✅ Multi-key operations work with hash tags {123}")
    
    # ❌ This might not work - keys on different nodes
    try:
        rc.mset({"user:alice": "Alice", "user:bob": "Bob"})
        print("✅ Multi-key operation succeeded")
    except Exception as e:
        print(f"❌ Multi-key operation failed: {e}")
    
    print("\nℹ️  Use hash tags {} to keep related keys on same node")

cluster_limitations_demo()
```

**Sample Output:**
```
=== CLUSTER LIMITATIONS ===
✅ Single key operations work fine
✅ Multi-key operations work with hash tags {123}
❌ Multi-key operation failed: Keys in request don't hash to the same slot

ℹ️  Use hash tags {} to keep related keys on same node
```

### Hash Tags Explained

```python
def hash_tag_examples():
    """Show how to use hash tags properly"""
    
    print("=== HASH TAG EXAMPLES ===")
    
    # All these keys will be on the same node (because of {user123})
    user_keys = {
        "user:{user123}:profile": "Alice Smith",
        "user:{user123}:settings": "dark_mode=true",
        "user:{user123}:friends": "bob,charlie"
    }
    
    rc.mset(user_keys)  # This works because all keys hash to same slot
    
    print("✅ Stored user data with hash tag {user123}")
    print("ℹ️  All keys: user:{user123}:* are on the same node")
    
    # You can now do multi-key operations on these keys
    user_data = rc.mget(list(user_keys.keys()))
    print(f"📋 Retrieved all user data: {user_data}")

hash_tag_examples()
```

**Sample Output:**
```
=== HASH TAG EXAMPLES ===
✅ Stored user data with hash tag {user123}
ℹ️  All keys: user:{user123}:* are on the same node
📋 Retrieved all user data: ['Alice Smith', 'dark_mode=true', 'bob,charlie']
```

## When to Use Clustering

### Good for Clustering:
- **Large datasets** that don't fit on one server
- **High traffic** applications with many concurrent users
- **Need high availability** (can't afford downtime)
- **Horizontal scaling** (add more servers instead of bigger servers)

### Not Good for Clustering:
- **Small applications** with low traffic
- **Complex transactions** across multiple keys
- **Lua scripts** that need access to keys on different nodes
- **Learning/development** (adds complexity)

## Summary

Redis clustering spreads data across multiple servers:

1. **Data is split** into 16,384 hash slots
2. **Each node** handles a range of slots
3. **Client automatically** routes requests to correct node
4. **Replicas provide backup** in case nodes fail
5. **Hash tags** keep related keys together

**What you learned:**

- How clustering distributes data across nodes
- Basic cluster setup and configuration
- Python client for cluster operations
- Key routing and slot calculation
- High availability with replicas
- Limitations and hash tag solutions

**Perfect for:** Large-scale applications that need to handle more data and traffic than a single Redis server can manage.
