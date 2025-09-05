# Redis Replication

Redis replication is like having a photocopier that automatically makes copies of all your data. You have one main Redis server (master) and one or more backup servers (replicas) that keep exact copies of everything.

## What is Replication?

Think of replication like:

- **Backup Drives:** Main computer + backup drive with same files
- **Library Copies:** Main library + branch libraries with same books
- **Mirror Websites:** Main website + mirror sites with same content
- **Team Notes:** One person writes, others copy the same notes

The main server handles writes, and backup servers automatically get all the same data.

## Why Use Replication?

### Benefits of Replication

```
Master Server          Replica Servers
+-------------+        +-------------+  +-------------+
|   Main      |   ---> |   Copy 1    |  |   Copy 2    |
|   Data      |        |   Same Data |  |   Same Data |
+-------------+        +-------------+  +-------------+
     ^                        |               |
     |                        v               v
  All Writes            Read Requests   Read Requests
```

**Benefits:**

- **High Availability:** If main server fails, backup takes over
- **Read Performance:** Multiple servers can handle read requests
- **Data Safety:** Multiple copies prevent data loss
- **Geographic Distribution:** Servers in different locations

## Basic Replication Setup

### Master Configuration

```bash
# redis-master.conf
port 6379
bind 0.0.0.0
# Allow replicas to connect
# No special master configuration needed
```

### Replica Configuration

```bash
# redis-replica.conf  
port 6380
bind 0.0.0.0
# Connect to master
replicaof 127.0.0.1 6379
```

### Starting Master and Replica

```bash
# Start master
redis-server redis-master.conf

# Start replica (in another terminal)
redis-server redis-replica.conf
```

**Config Explanation:**

- `replicaof 127.0.0.1 6379` = Connect to master at this address
- Replica automatically copies all data from master
- Replica stays in sync with master automatically

## Python Example with Replication

```python
import redis
import time

# Connect to master (for writes)
master = redis.Redis(host='127.0.0.1', port=6379, decode_responses=True)

# Connect to replica (for reads)
replica = redis.Redis(host='127.0.0.1', port=6380, decode_responses=True)

def test_replication():
    """Test how replication works"""
    
    print("=== TESTING REPLICATION ===")
    
    # Write to master
    master.set("user:123", "Alice Smith")
    master.set("user:456", "Bob Jones")
    print("✅ Wrote data to master")
    
    # Small delay for replication
    time.sleep(0.1)
    
    # Read from replica
    user123 = replica.get("user:123")
    user456 = replica.get("user:456")
    
    print(f"📖 Read from replica: {user123}, {user456}")
    
    # Add more data to master
    master.lpush("messages", "Hello World")
    master.lpush("messages", "How are you?")
    print("✅ Added messages to master")
    
    time.sleep(0.1)
    
    # Read messages from replica
    messages = replica.lrange("messages", 0, -1)
    print(f"📖 Messages from replica: {messages}")

def read_write_split_example():
    """Example of using master for writes, replica for reads"""
    
    print("\n=== READ/WRITE SPLIT EXAMPLE ===")
    
    # Function to save user data (writes go to master)
    def save_user(user_id, name, email):
        master.hset(f"user:{user_id}", mapping={
            "name": name,
            "email": email,
            "last_updated": time.time()
        })
        print(f"💾 Saved user {user_id} to master")
    
    # Function to get user data (reads come from replica)
    def get_user(user_id):
        user_data = replica.hgetall(f"user:{user_id}")
        if user_data:
            return {k: v for k, v in user_data.items()}
        return None
    
    # Test the split
    save_user("123", "Alice", "alice@example.com")
    save_user("456", "Bob", "bob@example.com")
    
    time.sleep(0.1)  # Wait for replication
    
    # Read from replica
    alice = get_user("123")
    bob = get_user("456")
    
    print(f"👤 Alice from replica: {alice}")
    print(f"👤 Bob from replica: {bob}")

# Run tests
test_replication()
read_write_split_example()
```

**Sample Output:**
```
=== TESTING REPLICATION ===
✅ Wrote data to master
📖 Read from replica: Alice Smith, Bob Jones
✅ Added messages to master
📖 Messages from replica: ['How are you?', 'Hello World']

=== READ/WRITE SPLIT EXAMPLE ===
💾 Saved user 123 to master
💾 Saved user 456 to master
👤 Alice from replica: {'name': 'Alice', 'email': 'alice@example.com', 'last_updated': '1693939200.123'}
👤 Bob from replica: {'name': 'Bob', 'email': 'bob@example.com', 'last_updated': '1693939201.456'}
```

## Replication Commands

### Check Replication Status

```redis
# On master - see connected replicas
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=123,lag=0

# On replica - see master connection
127.0.0.1:6380> INFO replication  
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
```

**Explanation:**

- `role:master` = This server is the main server
- `connected_slaves:1` = 1 replica is connected
- `master_link_status:up` = Replica is connected to master

### Manual Replication Control

```redis
# On replica - temporarily stop replication
127.0.0.1:6380> REPLICAOF NO ONE
OK

# On replica - resume replication
127.0.0.1:6380> REPLICAOF 127.0.0.1 6379
OK
```

## Failover Example

```python
def simulate_failover():
    """Example of handling master failure"""
    
    print("=== FAILOVER SIMULATION ===")
    
    try:
        # Try to write to master
        master.set("test_key", "test_value")
        print("✅ Master is working")
        
    except redis.ConnectionError:
        print("❌ Master is down!")
        print("🔄 Promoting replica to master...")
        
        # Promote replica to master
        replica.replicaof()  # Become independent
        
        # Now use replica as new master
        replica.set("emergency_key", "emergency_value")
        print("✅ Replica is now handling writes")

# This would be used in real failover scenarios
print("ℹ️  Failover example (master is working, so no failover needed)")
```

**Sample Output:**
```
=== FAILOVER SIMULATION ===
✅ Master is working
ℹ️  Failover example (master is working, so no failover needed)
```

## Summary

Replication creates automatic backup copies:

1. **Master** handles all write operations
2. **Replicas** automatically copy all data from master
3. **Replicas** can handle read operations
4. **Failover** possible if master goes down

**What you learned:**

- Basic master/replica setup
- Read/write split pattern
- Replication monitoring commands
- Failover concepts

**Perfect for:** Applications that need high availability and want to distribute read load across multiple servers.
