# Redis Performance Tuning

Quick guide to make your Redis faster and handle more load.

## Memory Optimization

### Configure Memory Limits

!!! warning "Default Behavior"
    Redis uses all available system memory by default, which can crash your system.

**Set Memory Limits:**
```bash
# Set maximum memory (adjust for your system)
redis-cli config set maxmemory 1gb

# Set eviction policy
redis-cli config set maxmemory-policy allkeys-lru
```

### Memory Policies Explained

**Eviction Policies:*
- **allkeys-lru**: Remove least recently used keys (best for cache)
- **allkeys-lfu**: Remove least frequently used keys
- **volatile-lru**: Remove LRU keys with expire time only
- **noeviction**: Don't remove anything, return errors (default)

**Choose the Right Policy:**
```python
# For caching - use allkeys-lru
redis_client.config_set('maxmemory-policy', 'allkeys-lru')

# For session storage - use volatile-lru  
redis_client.config_set('maxmemory-policy', 'volatile-lru')
```

### Monitor Memory Usage

**Check Memory Usage:**
```bash
# Get detailed memory info
redis-cli info memory

# Key fields to watch:
# used_memory_human: Current memory usage
# used_memory_peak_human: Peak memory usage  
# maxmemory_human: Memory limit
```

## Connection Optimization

### Use Connection Pooling

**Avoid This:**
```python
# Don't create new connection for each request
def get_user(user_id):
    redis_client = redis.Redis(host='localhost')  # BAD!
    return redis_client.get(f"user:{user_id}")
```

**Do This Instead:**
```python
# Create connection pool once
redis_pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=20,
    decode_responses=True
)

# Reuse connections
redis_client = redis.Redis(connection_pool=redis_pool)

def get_user(user_id):
    return redis_client.get(f"user:{user_id}")
```

### Optimize Connection Settings

**Better Connection Settings:**
```python
redis_client = redis.Redis(
    host='localhost',
    port=6379,
    socket_keepalive=True,           # Keep connections alive
    socket_keepalive_options={},
    health_check_interval=30,        # Check connection health
    retry_on_timeout=True,           # Retry on timeout
    decode_responses=True
)
```

## Command Optimization

### Use Pipelines for Multiple Commands

**Slow Way:**
```python
# Each command is a separate network round-trip
redis_client.set("key1", "value1")  # Round-trip 1
redis_client.set("key2", "value2")  # Round-trip 2  
redis_client.set("key3", "value3")  # Round-trip 3
```

**Fast Way:**
```python
# Send all commands at once
pipe = redis_client.pipeline()
pipe.set("key1", "value1")
pipe.set("key2", "value2") 
pipe.set("key3", "value3")
results = pipe.execute()  # Single round-trip
```

### Batch Operations

**Use Batch Commands When Available:**
```python
# Instead of multiple SET commands
redis_client.mset({
    "user:1": "Alice",
    "user:2": "Bob", 
    "user:3": "Charlie"
})

# Instead of multiple GET commands
values = redis_client.mget(["user:1", "user:2", "user:3"])
```

### Avoid Expensive Commands

!!! danger "Avoid on Production"
    ```bash
    # These commands can block Redis
    KEYS *          # Scans entire database
    FLUSHALL        # Deletes everything
    SORT            # Can be very slow
    ```

**Use Alternatives:**
```bash
# Instead of KEYS *, use SCAN
redis-cli scan 0 match "user:*" count 100

# Use specific key patterns
redis-cli keys "user:*"  # Only if you need few keys
```

## Data Structure Optimization

### Choose Right Data Types

**Performance by Data Type:**
```python
# Strings - fastest for simple values
redis_client.set("counter", "1")

# Hashes - efficient for objects
redis_client.hset("user:123", mapping={
    "name": "Alice",
    "email": "alice@example.com"
})

# Lists - good for queues
redis_client.lpush("tasks", "task1")

# Sets - fast membership testing
redis_client.sadd("online_users", "user123")
```

### Optimize Key Names

**Efficient Key Naming:**
```python
# Good - short and descriptive
"u:123"           # user:123
"s:456"           # session:456
"c:product:789"   # cache:product:789

# Avoid - too long
"very_long_descriptive_key_name_for_user_data:123"
```

### Set Appropriate Expiration

**Use TTL for Temporary Data:**
```python
# Cache data with expiration
redis_client.setex("cache:product:123", 3600, product_data)  # 1 hour

# Session data
redis_client.setex("session:abc123", 1800, session_data)     # 30 minutes

# Temporary locks
redis_client.setex("lock:process", 60, "locked")             # 1 minute
```

## Configuration Tuning

### Persistence Settings

!!! warning "Persistence vs Performance"
    Persistence saves data to disk but slows down Redis.

**For High Performance (Cache Mode):**
```bash
# Disable persistence completely
redis-cli config set save ""
redis-cli config set appendonly no
```

**For Balanced Performance:**
```bash
# Less frequent saves
redis-cli config set save "900 1 300 10 60 10000"

# Enable AOF with less frequent sync
redis-cli config set appendonly yes
redis-cli config set appendfsync everysec
```

### Network Settings

**Optimize Network:**
```bash
# Increase output buffer size
redis-cli config set client-output-buffer-limit "normal 0 0 0"

# TCP keepalive
redis-cli config set tcp-keepalive 300

# Disable protected mode if on trusted network
redis-cli config set protected-mode no
```

## Monitoring Performance

### Key Metrics to Watch

**Essential Metrics:**
```bash
# Commands per second
redis-cli info stats | grep instantaneous_ops_per_sec

# Memory usage
redis-cli info memory | grep used_memory_human

# Connected clients
redis-cli info clients | grep connected_clients

# Hit rate (for cache usage)
redis-cli info stats | grep keyspace_hits
redis-cli info stats | grep keyspace_misses
```

### Identify Slow Commands

**Enable and Check Slow Log:**
```bash
# Log commands slower than 10ms
redis-cli config set slowlog-log-slower-than 10000

# Check slow commands
redis-cli slowlog get 10

# Reset slow log
redis-cli slowlog reset
```

### Real-time Monitoring

**Monitor Live Performance:**
```bash
# Watch commands in real-time
redis-cli monitor

# Watch stats continuously
redis-cli --latency

# Memory usage over time
redis-cli --latency-history -i 1
```

## Performance Testing

### Benchmark Your Setup

**Built-in Benchmark Tool:**
```bash
# Basic benchmark
redis-benchmark

# Test specific operations
redis-benchmark -t set,get -n 100000 -q

# Test with pipeline
redis-benchmark -t set,get -n 100000 -P 16 -q
```

### Python Performance Testing

**Simple Performance Test:**
```python
import time
import redis

redis_client = redis.Redis(host='localhost', decode_responses=True)

def test_performance():
    # Test 10,000 SET operations
    start_time = time.time()
    
    for i in range(10000):
        redis_client.set(f"test:key:{i}", f"value:{i}")
    
    end_time = time.time()
    ops_per_second = 10000 / (end_time - start_time)
    
    print(f"SET operations per second: {ops_per_second:.2f}")
    
    # Test with pipeline
    start_time = time.time()
    pipe = redis_client.pipeline()
    
    for i in range(10000):
        pipe.set(f"test:pipe:{i}", f"value:{i}")
    
    pipe.execute()
    end_time = time.time()
    
    ops_per_second = 10000 / (end_time - start_time)
    print(f"Pipeline SET operations per second: {ops_per_second:.2f}")

test_performance()
```

## Quick Performance Checklist

!!! success "Performance Optimization Checklist"
    - Set appropriate memory limits
    - Choose right eviction policy
    - Use connection pooling
    - Use pipelines for multiple commands
    - Choose efficient data structures
    - Set TTL on temporary data
    - Monitor key performance metrics
    - Avoid expensive commands in production
    - Use batch operations when possible
    - Configure persistence based on needs

!!! warning "Common Performance Killers"
    - Creating new connections for each request
    - Using KEYS * on large databases
    - Not setting memory limits
    - Storing very large values (>1MB)
    - Not using pipelines for multiple operations
    - Blocking operations without timeouts
