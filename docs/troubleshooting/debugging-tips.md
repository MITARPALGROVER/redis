# Redis Debugging Tips

Essential debugging techniques to quickly identify and fix Redis issues.

## Basic Debugging Commands

### Check if Redis is Working

**First Steps:**
```bash
# Test basic connectivity
redis-cli ping
# Expected: PONG

# Check Redis version and basic info
redis-cli info server

# Test with specific host/port
redis-cli -h 127.0.0.1 -p 6379 ping
```

### Get Current Status

**Server Status Commands:**
```bash
# Overall server information
redis-cli info

# Memory usage details
redis-cli info memory

# Client connections
redis-cli info clients

# Statistics
redis-cli info stats
```

## Debugging Data Issues

### Inspect Keys and Values

**Key Investigation:**
```bash
# Check if key exists
redis-cli exists "user:123"

# Get key type
redis-cli type "user:123"

# Check TTL (time to live)
redis-cli ttl "user:123"
# -1 = no expiration, -2 = expired/doesn't exist

# Get key size
redis-cli memory usage "user:123"
```

### Search for Keys

!!! warning "Use SCAN Instead of KEYS"
    ```bash
    # Don't use this on production (blocks Redis)
    redis-cli keys "*"
    
    # Use SCAN instead (non-blocking)
    redis-cli scan 0 match "user:*" count 100
    
    # Find keys by pattern
    redis-cli scan 0 match "session:*"
    redis-cli scan 0 match "*:cache"
    ```

### Examine Data Structures

**Debug Different Data Types:**
```bash
# String values
redis-cli get "user:name"

# Hash fields
redis-cli hgetall "user:123"
redis-cli hkeys "user:123"      # Just field names
redis-cli hvals "user:123"      # Just values

# List contents
redis-cli lrange "queue:tasks" 0 -1  # All items
redis-cli llen "queue:tasks"          # List length

# Set members
redis-cli smembers "online:users"
redis-cli scard "online:users"        # Set size

# Sorted set
redis-cli zrange "leaderboard" 0 -1 withscores
redis-cli zcard "leaderboard"         # Set size
```

## Real-time Debugging

### Monitor Commands in Real-time

**Live Command Monitoring:**
```bash
# Watch all commands as they happen
redis-cli monitor

# Example output:
# 1693939200.123456 [0 127.0.0.1:54321] "SET" "user:123" "Alice"
# 1693939201.234567 [0 127.0.0.1:54322] "GET" "user:123"
```

### Track Client Connections

**Debug Connection Issues:**
```bash
# List connected clients
redis-cli client list

# Kill specific client by ID
redis-cli client kill id 12345

# Kill clients by pattern
redis-cli client kill addr 192.168.1.100:54321
```

## Python Debugging Techniques

### Enable Redis Command Logging

**Log All Redis Commands:**
```python
import redis
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('redis')

redis_client = redis.Redis(
    host='localhost',
    port=6379,
    decode_responses=True
)

# Now all Redis commands will be logged
redis_client.set("debug:key", "debug:value")
redis_client.get("debug:key")
```

### Connection Pool Debugging

**Debug Connection Pool Issues:**
```python
import redis

# Create pool with debugging
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=10,
    decode_responses=True
)

redis_client = redis.Redis(connection_pool=pool)

def debug_connection_pool():
    """Check connection pool status"""
    pool_info = {
        "created_connections": pool.created_connections,
        "available_connections": len(pool._available_connections),
        "in_use_connections": len(pool._in_use_connections)
    }
    print(f"Pool status: {pool_info}")
    return pool_info

# Test connection pool
debug_connection_pool()
redis_client.set("test", "value")
debug_connection_pool()
```

### Exception Handling for Debugging

**Comprehensive Error Handling:**
```python
import redis
import traceback

def debug_redis_operation():
    """Example of debugging Redis operations"""
    
    try:
        redis_client = redis.Redis(host='localhost', decode_responses=True)
        
        # Test connection
        redis_client.ping()
        print("✅ Connection successful")
        
        # Test operation
            redis_client.set("debug:test", "test_value")
            value = redis_client.get("debug:test")
            print(f"✅ Operation successful: {value}")
            
        except redis.ConnectionError as e:
            print(f"❌ Connection Error: {e}")
            print("Check if Redis server is running")
            
        except redis.AuthenticationError as e:
            print(f"❌ Authentication Error: {e}")
            print("Check Redis password configuration")
            
        except redis.ResponseError as e:
            print(f"❌ Redis Response Error: {e}")
            print("Check command syntax and data types")
            
        except Exception as e:
            print(f"❌ Unexpected Error: {e}")
            print("Full traceback:")
            traceback.print_exc()
    
    debug_redis_operation()
    ```

## Performance Debugging

### Identify Slow Operations

**Enable Slow Log:**
```bash
# Set threshold for slow log (microseconds)
redis-cli config set slowlog-log-slower-than 10000  # 10ms

# Check slow operations
redis-cli slowlog get 5

# Example output:
# 1) 1) (integer) 14           # Slow log entry ID
#    2) (integer) 1693939200   # Timestamp
#    3) (integer) 12000        # Execution time (microseconds)  
#    4) 1) "GET"               # Command
#       2) "slow:key"          # Arguments
```

### Memory Usage Debugging

**Debug Memory Issues:**
```bash
# Get memory breakdown
redis-cli info memory

# Find largest keys
redis-cli --bigkeys

# Analyze specific key memory usage
redis-cli memory usage "large:key"

# Get memory usage by database
redis-cli info keyspace
```

### CPU Usage Debugging

**Check CPU Performance:**
```bash
# Monitor Redis CPU usage
redis-cli info cpu

# Check command statistics
redis-cli info commandstats

# Example output shows most used commands:
# cmdstat_get:calls=1000,usec=50000,usec_per_call=50.00
# cmdstat_set:calls=800,usec=40000,usec_per_call=50.00
```

## Network Debugging

### Test Network Connectivity

**Network Troubleshooting:**
```bash
# Test from different machines
redis-cli -h remote.redis.server.com -p 6379 ping

# Check if port is accessible
telnet redis.server.com 6379

# Test with timeout
redis-cli -h redis.server.com --latency-history
```

### Debug Latency Issues

**Measure Redis Latency:**
```bash
# Basic latency test
redis-cli --latency -h redis.server.com

# Detailed latency histogram
redis-cli --latency-dist -h redis.server.com

# Track latency over time
redis-cli --latency-history -i 1
```

## Configuration Debugging

### Check Current Configuration

**Inspect Redis Configuration:**
```bash
# Get all configuration
redis-cli config get "*"

# Check specific settings
redis-cli config get "maxmemory*"
redis-cli config get "save"
redis-cli config get "*timeout*"

# Check if configuration file is being used
redis-cli info server | grep config_file
```

### Validate Configuration Changes

!!! warning "Test Configuration Changes"
    ```bash
    # Get current value
    redis-cli config get maxmemory
    
    # Change value temporarily
    redis-cli config set maxmemory 1gb
    
    # Verify change
    redis-cli config get maxmemory
    
    # Revert if needed
    redis-cli config set maxmemory 0  # No limit
    ```

## Debugging Scripts and Tools

### Simple Health Check Script

**Create a Health Check:**
```python
import redis
import time
import sys

def redis_health_check():
    """Comprehensive Redis health check"""
    
    print("🔍 Redis Health Check")
    print("=" * 50)
    
    try:
        # Connect to Redis
        r = redis.Redis(host='localhost', port=6379, decode_responses=True)
        
        # Test 1: Basic connectivity
        start = time.time()
        response = r.ping()
        latency = (time.time() - start) * 1000
        
        if response:
            print(f"✅ Connection: OK (latency: {latency:.2f}ms)")
        else:
            print("❌ Connection: Failed")
            return False
        
        # Test 2: Memory usage
        memory_info = r.info('memory')
        used_memory = memory_info['used_memory_human']
        max_memory = memory_info.get('maxmemory_human', 'No limit')
        print(f"📊 Memory: {used_memory} / {max_memory}")
        
        # Test 3: Connected clients
        clients_info = r.info('clients')
        connected_clients = clients_info['connected_clients']
        print(f"👥 Connected clients: {connected_clients}")
        
        # Test 4: Basic operations
        test_key = "health:check:" + str(int(time.time()))
        r.set(test_key, "test_value", ex=60)  # Expires in 60 seconds
        value = r.get(test_key)
        
        if value == "test_value":
            print("✅ Read/Write: OK")
            r.delete(test_key)  # Cleanup
        else:
            print("❌ Read/Write: Failed")
            return False
        
        print("🎉 Redis is healthy!")
        return True
        
    except redis.ConnectionError:
        print("❌ Cannot connect to Redis server")
            return False
        except Exception as e:
            print(f"❌ Health check failed: {e}")
            return False
    
    if __name__ == "__main__":
        is_healthy = redis_health_check()
        sys.exit(0 if is_healthy else 1)
    ```

## Common Debug Scenarios

### Debugging Cache Misses

!!! note "Track Cache Effectiveness"
    ```python
    def debug_cache_performance():
        """Debug cache hit/miss ratio"""
        
        redis_client = redis.Redis(host='localhost', decode_responses=True)
        
        # Get current stats
        info = redis_client.info('stats')
        hits = info['keyspace_hits']
        misses = info['keyspace_misses']
        
        if hits + misses > 0:
            hit_rate = (hits / (hits + misses)) * 100
            print(f"Cache hit rate: {hit_rate:.2f}%")
            print(f"Total hits: {hits}")
            print(f"Total misses: {misses}")
        else:
            print("No cache operations recorded yet")
    
    debug_cache_performance()
    ```

### Debugging Data Inconsistency

!!! warning "Check Data Consistency"
    ```python
    def debug_data_consistency():
        """Check for data consistency issues"""
        
        redis_client = redis.Redis(host='localhost', decode_responses=True)
        
        # Check for orphaned keys
        print("🔍 Checking for orphaned session data...")
        
        # Find all session keys
        session_keys = []
        cursor = 0
        while True:
            cursor, keys = redis_client.scan(cursor, match="session:*", count=100)
            session_keys.extend(keys)
            if cursor == 0:
                break
        
        print(f"Found {len(session_keys)} session keys")
        
        # Check which sessions have expired
        expired_sessions = []
        for key in session_keys[:10]:  # Check first 10 for demo
            ttl = redis_client.ttl(key)
            if ttl == -2:  # Key doesn't exist
                expired_sessions.append(key)
        
        if expired_sessions:
            print(f"Found {len(expired_sessions)} expired sessions")
        else:
            print("✅ No expired sessions found")
    
    debug_data_consistency()
    ```

## Quick Debug Checklist

!!! success "Debugging Checklist"
    - Test basic connectivity with `redis-cli ping`
    - Check server status with `redis-cli info`
    - Monitor commands with `redis-cli monitor`
    - Check slow operations with `slowlog get`
    - Verify memory usage with `info memory`
    - Test network latency with `--latency`
    - Check configuration with `config get "*"`
    - Use proper exception handling in code
    - Enable logging for Redis operations
    - Monitor cache hit rates

!!! danger "Debug Mode Warning"
    - Never use `KEYS *` on production
    - Don't leave `MONITOR` running on production
    - Be careful with `CLIENT KILL` commands
    - Always test configuration changes first
    - Don't forget to clean up debug data
