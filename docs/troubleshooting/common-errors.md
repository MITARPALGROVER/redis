# Common Redis Errors

This guide covers the most frequent Redis errors beginners encounter and how to fix them quickly.

## Connection Errors

### "Connection refused" Error

**Error Message:**
```
redis.exceptions.ConnectionError: Error 10061 connecting to localhost:6379. 
Connection refused.
```

**What This Means:** Redis server is not running or not accepting connections on the specified port.

**Quick Fix:**
```bash
# Check if Redis is running
redis-cli ping

# If not running, start Redis
redis-server

# Or start with config file
redis-server redis.conf
```

### "Authentication required" Error

**Error Message:**
```
redis.exceptions.AuthenticationError: Authentication required.
```

**What This Means:** Redis requires a password but none was provided.

**Quick Fix:**
```python
# Add password to connection
redis_client = redis.Redis(
    host='localhost',
    port=6379,
    password='your_password',  # Add this line
    decode_responses=True
)
```

### "Connection timeout" Error

**Error Message:**
```
redis.exceptions.TimeoutError: Timeout reading from socket
```

!!! warning "Common Causes"
    - Network issues
    - Redis server overloaded
    - Firewall blocking connection
    - Wrong host/port

**Quick Fix:**
```python
# Increase timeout
redis_client = redis.Redis(
    host='localhost',
    port=6379,
    socket_timeout=30,  # Increase from default 5 seconds
    socket_connect_timeout=30
)
```

## Memory Errors

### "Out of memory" Error

**Error Message:**
```
OOM command not allowed when used memory > 'maxmemory'
```

**What This Means:** Redis has reached its memory limit and cannot store more data.

**Quick Fixes:**
```bash
# Check current memory usage
redis-cli info memory

# Increase memory limit (temporarily)
redis-cli config set maxmemory 1gb

# Or remove old data
redis-cli flushdb
```

### Memory Policy Issues

**Memory Policies:**
```bash
# Check current policy
redis-cli config get maxmemory-policy

# Set better policy for cache usage
redis-cli config set maxmemory-policy allkeys-lru
```

## Data Type Errors

### "Wrong type" Error

**Error Message:**
```
redis.exceptions.ResponseError: WRONGTYPE Operation against a key 
holding the wrong kind of value
```

**What This Means:** Trying to use string commands on a list, or list commands on a string, etc.

**Quick Fix:**
```python
# Check what type the key is
key_type = redis_client.type("my_key")
print(f"Key type: {key_type}")

# Delete and recreate with correct type
redis_client.delete("my_key")
redis_client.set("my_key", "correct_value")
```

### Key Not Found Issues

**Common Mistake:**
```python
# This returns None if key doesn't exist
value = redis_client.get("nonexistent_key")  
print(value)  # None

# Better approach
if redis_client.exists("my_key"):
    value = redis_client.get("my_key")
    print(f"Value: {value}")
else:
    print("Key does not exist")
```

## Command Errors

### "Unknown command" Error

**Error Message:**
```
redis.exceptions.ResponseError: ERR unknown command 'SOME_COMMAND'
```

!!! warning "Common Causes"
    - Typo in command name
    - Using command from newer Redis version
    - Command was renamed/disabled

**Quick Fix:**
```bash
# Check Redis version
redis-cli info server

# List all available commands
redis-cli command

# Check if command exists
redis-cli command info SET
```

### "Syntax error" in Commands

**Common Syntax Errors:**
```python
# Wrong - missing quotes
redis_client.set(user:123, "John")  # Error!

# Correct
redis_client.set("user:123", "John")

# Wrong - incorrect parameters
redis_client.lpush("mylist")  # Missing value!

# Correct  
redis_client.lpush("mylist", "item1")
```

## Configuration Errors

### "Permission denied" Error

**Error Message:**
```
Permission denied writing config file
```

**What This Means:** Redis cannot write to config file due to file permissions.

**Quick Fix:**
```bash
# On Windows - run as administrator
# On Linux/Mac
sudo chown redis:redis /etc/redis/redis.conf
sudo chmod 644 /etc/redis/redis.conf
```

### Port Already in Use

**Error Message:**
```
Could not bind to port 6379: Address already in use
```

**Quick Fixes:**
```bash
# Find what's using port 6379
netstat -tulpn | grep 6379

# Kill existing Redis process
pkill redis-server

# Or use different port
redis-server --port 6380
```

## Python-Specific Errors

### Import Error

**Error Message:**
```
ModuleNotFoundError: No module named 'redis'
```

**Quick Fix:**
```bash
# Install redis package
pip install redis

# Or with specific version
pip install redis==4.5.4
```

### Encoding Issues

**Common Problem:**
```python
# This returns bytes
value = redis_client.get("key")  # b'hello'

# Fix: use decode_responses=True
redis_client = redis.Redis(
    host='localhost',
    port=6379,
    decode_responses=True  # This fixes encoding
)
```

## Performance Issues

### Slow Queries

**Signs of Slow Queries:**

- Commands taking too long
- High CPU usage
- Connection timeouts

**Quick Diagnosis:**
```bash
# Enable slow log
redis-cli config set slowlog-log-slower-than 10000

# Check slow queries
redis-cli slowlog get 10

# Monitor commands in real-time
redis-cli monitor
```

### Too Many Connections

**Error Message:**
```
ERR max number of clients reached
```

**Quick Fix:**
```bash
# Check current connections
redis-cli info clients

# Increase max clients (temporarily)
redis-cli config set maxclients 1000

# Close unused connections in your code
redis_client.close()
```

## Quick Diagnostic Commands

**Essential Debug Commands:**
```bash
# Check if Redis is working
redis-cli ping

# Get server info
redis-cli info

# Check memory usage
redis-cli info memory

# List all keys (careful on production!)
redis-cli keys "*"

# Check specific key
redis-cli type "my_key"
redis-cli ttl "my_key"

# Monitor commands
redis-cli monitor
```

## Error Prevention Tips

!!! tip "Best Practices"
    - Always check if keys exist before operations
    - Use proper data types for operations
    - Handle connection errors gracefully
    - Set appropriate timeouts
    - Monitor memory usage regularly
    - Use connection pooling for multiple requests

!!! warning "What to Avoid"
    - Don't use `KEYS *` on production
    - Don't store huge values (>100MB)
    - Don't ignore memory limits
    - Don't forget to close connections
    - Don't use blocking operations without timeouts
