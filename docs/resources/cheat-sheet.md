# Redis Cheat Sheet

Quick reference for the most common Redis commands.

## Connection & Server

```bash
# Test connection
redis-cli ping

# Connect to specific server
redis-cli -h hostname -p 6379

# Connect with authentication
redis-cli -a password

# Get server info
redis-cli info

# Check Redis version
redis-cli info server

# Monitor commands in real-time
redis-cli monitor

# Check memory usage
redis-cli info memory
```

## String Operations

```bash
# Set/Get values
SET key "value"
GET key
MSET key1 "value1" key2 "value2"
MGET key1 key2

# Set with expiration
SETEX key 60 "value"        # Expires in 60 seconds
SET key "value" EX 60       # Same as above

# Increment/Decrement
INCR counter
INCRBY counter 5
DECR counter
DECRBY counter 3

# Append to string
APPEND key "more_text"

# Get string length
STRLEN key

# Check if key exists
EXISTS key

# Get key type
TYPE key

# Delete key
DEL key
```

## Hash Operations

```bash
# Set/Get hash fields
HSET user:123 name "Alice"
HGET user:123 name
HMSET user:123 name "Alice" age 25 email "alice@example.com"
HMGET user:123 name age

# Get all fields and values
HGETALL user:123

# Get only field names or values
HKEYS user:123
HVALS user:123

# Check if field exists
HEXISTS user:123 name

# Delete field
HDEL user:123 email

# Get number of fields
HLEN user:123

# Increment hash field
HINCRBY user:123 age 1
```

## List Operations

```bash
# Add to list
LPUSH mylist "item1"        # Add to left (beginning)
RPUSH mylist "item2"        # Add to right (end)

# Get from list
LPOP mylist                 # Remove and get from left
RPOP mylist                 # Remove and get from right

# Get range
LRANGE mylist 0 -1         # Get all items
LRANGE mylist 0 4          # Get first 5 items

# Get list length
LLEN mylist

# Get item at index
LINDEX mylist 0

# Set item at index
LSET mylist 0 "new_value"

# Insert item
LINSERT mylist BEFORE "item1" "new_item"

# Remove items
LREM mylist 1 "item1"      # Remove 1 occurrence of "item1"

# Trim list
LTRIM mylist 0 99          # Keep only first 100 items
```

## Set Operations

```bash
# Add/Remove members
SADD myset "member1"
SREM myset "member1"

# Check membership
SISMEMBER myset "member1"

# Get all members
SMEMBERS myset

# Get random member
SRANDMEMBER myset

# Pop random member
SPOP myset

# Set operations
SINTER set1 set2           # Intersection
SUNION set1 set2           # Union
SDIFF set1 set2            # Difference

# Store set operations result
SINTERSTORE result set1 set2
SUNIONSTORE result set1 set2
SDIFFSTORE result set1 set2

# Get set size
SCARD myset

# Move member between sets
SMOVE source_set dest_set "member"
```

## Sorted Set Operations

```bash
# Add with score
ZADD leaderboard 100 "player1"
ZADD leaderboard 150 "player2"

# Get by rank
ZRANGE leaderboard 0 -1 WITHSCORES    # All members with scores
ZREVRANGE leaderboard 0 2             # Top 3 members

# Get by score
ZRANGEBYSCORE leaderboard 100 200
ZREVRANGEBYSCORE leaderboard 200 100

# Get member rank
ZRANK leaderboard "player1"           # 0-based rank (ascending)
ZREVRANK leaderboard "player1"        # 0-based rank (descending)

# Get score
ZSCORE leaderboard "player1"

# Increment score
ZINCRBY leaderboard 10 "player1"

# Remove member
ZREM leaderboard "player1"

# Remove by rank
ZREMRANGEBYRANK leaderboard 0 2       # Remove bottom 3

# Remove by score
ZREMRANGEBYSCORE leaderboard 0 100

# Count members
ZCARD leaderboard

# Count by score range
ZCOUNT leaderboard 100 200
```

## Expiration & TTL

```bash
# Set expiration
EXPIRE key 60              # Expire in 60 seconds
EXPIREAT key 1693939200    # Expire at timestamp

# Check time to live
TTL key                    # Returns seconds (-1 = no expiration, -2 = expired)

# Remove expiration
PERSIST key

# Set expiration when creating
SETEX key 60 "value"       # String with expiration
PSETEX key 60000 "value"   # String with expiration in milliseconds

# Get TTL in milliseconds
PTTL key
```

## Key Management

```bash
# List keys (use carefully in production)
KEYS pattern               # e.g., KEYS "user:*"

# Scan keys (safer for production)
SCAN 0 MATCH "user:*" COUNT 100

# Rename key
RENAME oldkey newkey
RENAMENX oldkey newkey     # Rename only if newkey doesn't exist

# Check key existence
EXISTS key
EXISTS key1 key2 key3      # Check multiple keys

# Get random key
RANDOMKEY

# Copy key
COPY source dest

# Get key information
TYPE key
OBJECT ENCODING key
MEMORY USAGE key

# Touch key (update last access time)
TOUCH key
```

## Transaction Commands

```bash
# Start transaction
MULTI

# Execute transaction
EXEC

# Discard transaction
DISCARD

# Watch keys for changes
WATCH key1 key2

# Unwatch all keys
UNWATCH
```

## Pub/Sub Commands

```bash
# Publish message
PUBLISH channel "message"

# Subscribe to channels
SUBSCRIBE channel1 channel2

# Subscribe to patterns
PSUBSCRIBE pattern*

# Unsubscribe
UNSUBSCRIBE channel1
PUNSUBSCRIBE pattern*

# List channels
PUBSUB CHANNELS
PUBSUB CHANNELS pattern*

# Count subscribers
PUBSUB NUMSUB channel1 channel2
```

## Database Operations

```bash
# Select database
SELECT 0                   # Switch to database 0 (default)

# Flush database
FLUSHDB                    # Clear current database
FLUSHALL                   # Clear all databases

# Get database size
DBSIZE

# Save database
SAVE                       # Synchronous save
BGSAVE                     # Background save

# Get last save time
LASTSAVE

# Swap databases
SWAPDB 0 1                 # Swap database 0 with database 1
```

## Configuration

```bash
# Get configuration
CONFIG GET parameter       # e.g., CONFIG GET "maxmemory"
CONFIG GET "*"            # Get all configuration

# Set configuration
CONFIG SET parameter value # e.g., CONFIG SET maxmemory 1gb

# Reset statistics
CONFIG RESETSTAT

# Rewrite config file
CONFIG REWRITE
```

## Server Information

```bash
# Get all server info
INFO

# Get specific sections
INFO memory
INFO stats
INFO replication
INFO cpu
INFO clients
INFO server

# Get command statistics
INFO commandstats

# Get keyspace info
INFO keyspace
```

## Client Management

```bash
# List connected clients
CLIENT LIST

# Kill client connection
CLIENT KILL ip:port
CLIENT KILL id client_id

# Set client name
CLIENT SETNAME name

# Get client name
CLIENT GETNAME

# Pause all clients
CLIENT PAUSE timeout
```

## Memory Commands

```bash
# Get memory usage of key
MEMORY USAGE key

# Get memory statistics
MEMORY STATS

# Memory doctor (analyze memory usage)
MEMORY DOCTOR

# Purge memory
MEMORY PURGE
```

## Performance Monitoring

```bash
# Monitor commands in real-time
MONITOR

# Get slow log
SLOWLOG GET number
SLOWLOG RESET
SLOWLOG LEN

# Latency monitoring
LATENCY LATEST
LATENCY HISTORY event
LATENCY RESET
```

## Debugging Commands

```bash
# Basic diagnostics
PING
ECHO "message"
TIME

# Debug commands
DEBUG OBJECT key
DEBUG SEGFAULT          # Crash server (debugging only)

# Role information
ROLE                    # Master/slave role info
```

## Lua Scripting

```bash
# Execute Lua script
EVAL script numkeys key1 key2 arg1 arg2

# Execute cached script
EVALSHA sha1 numkeys key1 key2 arg1 arg2

# Load script
SCRIPT LOAD script

# Check if script exists
SCRIPT EXISTS sha1

# Kill running script
SCRIPT KILL

# Flush script cache
SCRIPT FLUSH
```

## Common Error Solutions

**Connection Refused:**
```bash
# Check if Redis is running
redis-cli ping
# Start Redis if needed
redis-server
```

**Out of Memory:**
```bash
# Check memory usage
INFO memory
# Set memory limit
CONFIG SET maxmemory 1gb
CONFIG SET maxmemory-policy allkeys-lru
```

**Authentication Error:**
```bash
# Connect with password
redis-cli -a your_password
```

**Slow Performance:**
```bash
# Check slow queries
SLOWLOG GET 10
# Monitor commands
MONITOR
# Check memory usage
INFO memory
```

## Important Commands Summary

**Most Used Commands:**
```bash
# Basic operations
SET key value
GET key
DEL key
EXISTS key
EXPIRE key seconds

# Hash operations
HSET key field value
HGET key field
HGETALL key

# List operations
LPUSH key value
RPUSH key value
LRANGE key start stop

# Set operations
SADD key member
SMEMBERS key

# Sorted set operations
ZADD key score member
ZRANGE key start stop WITHSCORES
```

!!! tip "Production Best Practices"
    - Use `SCAN` instead of `KEYS` in production
    - Always set expiration on temporary data
    - Monitor memory usage with `INFO memory`
    - Use `SLOWLOG` to identify slow queries
    - Set appropriate `maxmemory` and eviction policy

!!! warning "Dangerous Commands"
    - Never use `FLUSHALL` or `FLUSHDB` in production
    - Avoid `KEYS *` on large databases
    - Don't forget to set authentication
    - Be careful with `DEBUG` commands
    - Use `SCAN` for key iteration instead of `KEYS`
