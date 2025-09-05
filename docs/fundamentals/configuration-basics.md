# Configuration Basics

Redis is highly configurable, allowing you to tune it for different use cases - from simple caching to high-performance databases. Let's master Redis configuration step by step!

## Understanding Redis Configuration

### What is Redis Configuration?

Redis configuration controls how Redis behaves. Think of it like settings on your phone - you can adjust them to make Redis work exactly how you need it.

```redis
# Check current configuration
127.0.0.1:6379> CONFIG GET "*"
# This shows ALL current settings (there are many!)

# Check specific setting
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
# 0 means no memory limit set

# Change setting temporarily (until restart)
127.0.0.1:6379> CONFIG SET maxmemory 1073741824
OK
# Set 1GB memory limit

# Verify the change
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "1073741824"
```

### Configuration Methods

Redis can be configured in three ways:

| Method | When Changes Apply | Survives Restart? | Use Case |
|--------|-------------------|-------------------|----------|
| **CONFIG command** | Immediately | No | Testing, temporary changes |
| **Config file** | At startup | Yes | Permanent settings |
| **Command line** | At startup | No | Override config file |

---

## Configuration File Basics

### Finding Your Config File

**Where is redis.conf located?**

```bash
# Linux/macOS typical locations:
/etc/redis/redis.conf
/usr/local/etc/redis.conf
/opt/redis/redis.conf

# Windows (if using Redis on Windows):
C:\Program Files\Redis\redis.windows.conf
C:\redis\redis.conf

# Find config file location from Redis
127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/var/lib/redis"
# Config file is usually in same directory or nearby
```

### Basic Config File Structure

```bash
# Example redis.conf structure
# Lines starting with # are comments
# Settings are: parameter value

# Network settings
bind 127.0.0.1
port 6379

# Memory settings
maxmemory 2gb
maxmemory-policy allkeys-lru

# Persistence settings
save 900 1
appendonly yes

# Security settings
requirepass mypassword

# Logging
loglevel notice
logfile /var/log/redis/redis-server.log
```

**How to read config files:**
- Lines starting with `#` are comments (ignored)
- Format is: `setting_name value`
- Some settings can have multiple values
- Case sensitive!

---

## Essential Configuration Settings

### 1. Network Configuration

**Who can connect to Redis and how?**

```redis
# Check current network settings
127.0.0.1:6379> CONFIG GET bind
1) "bind"
2) "127.0.0.1"
# Only local connections allowed

127.0.0.1:6379> CONFIG GET port
1) "port"
2) "6379"
# Redis listens on port 6379

# Allow connections from any IP (careful - security risk!)
127.0.0.1:6379> CONFIG SET bind "0.0.0.0"
OK

# Change port (requires restart to take effect)
# Add to redis.conf: port 6380
```

**Common network configurations:**

```bash
# Config file examples:

# 1. Local development (default)
bind 127.0.0.1
port 6379

# 2. Allow specific IPs
bind 127.0.0.1 192.168.1.100 192.168.1.101
port 6379

# 3. Allow all connections (use with authentication!)
bind 0.0.0.0
port 6379
requirepass your_strong_password

# 4. Use custom port
bind 127.0.0.1
port 6380
```

### 2. Memory Configuration

**How much memory can Redis use?**

```redis
# Check current memory settings
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "0"
# 0 = unlimited (dangerous in production!)

127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
# noeviction = return errors when memory full

# Set memory limit (2GB)
127.0.0.1:6379> CONFIG SET maxmemory 2147483648
OK

# Set what happens when memory is full
127.0.0.1:6379> CONFIG SET maxmemory-policy allkeys-lru
OK

# Check current memory usage
127.0.0.1:6379> INFO memory | grep used_memory_human
used_memory_human:1.25M
```

**Memory policy options explained:**

| Policy | What It Does | Best For |
|--------|-------------|----------|
| `noeviction` | Return errors when memory full | Databases (never lose data) |
| `allkeys-lru` | Remove least recently used keys | General caching |
| `allkeys-lfu` | Remove least frequently used keys | Smart caching |
| `volatile-lru` | Remove LRU keys that have TTL | Mixed workloads |
| `volatile-ttl` | Remove keys with shortest TTL | Time-based cache |
| `allkeys-random` | Remove random keys | Simple cache |

**Example configurations:**

```bash
# Cache server (can lose data)
maxmemory 4gb
maxmemory-policy allkeys-lru

# Database server (never lose data)  
maxmemory 8gb
maxmemory-policy noeviction

# Session store (only temporary data)
maxmemory 2gb
maxmemory-policy volatile-lru
```

### 3. Persistence Configuration

**How to save your data to disk?**

```redis
# Check current persistence settings
127.0.0.1:6379> CONFIG GET save
1) "save"
2) "900 1 300 10 60 10000"
# RDB snapshots enabled

127.0.0.1:6379> CONFIG GET appendonly
1) "appendonly"
2) "no"
# AOF disabled

# Enable AOF persistence
127.0.0.1:6379> CONFIG SET appendonly yes
OK

# Configure AOF sync frequency
127.0.0.1:6379> CONFIG SET appendfsync everysec
OK

# Check persistence status
127.0.0.1:6379> INFO persistence | grep -E "(rdb_last_save_time|aof_enabled)"
rdb_last_save_time:1705751400
aof_enabled:1
```

**Persistence configuration examples:**

```bash
# Fast cache (no persistence)
save ""
appendonly no

# Balanced (snapshots only)
save "900 1 300 10 60 10000"
appendonly no
dbfilename dump.rdb

# Maximum durability (AOF only)
save ""
appendonly yes
appendfsync everysec
appendfilename "appendonly.aof"

# Production (both RDB + AOF)
save "900 1 300 10 60 10000"
appendonly yes
appendfsync everysec
aof-use-rdb-preamble yes
```

### 4. Security Configuration

**How to protect your Redis?**

```redis
# Check if password is set
127.0.0.1:6379> CONFIG GET requirepass
1) "requirepass"
2) ""
# Empty = no password set (dangerous!)

# Set password
127.0.0.1:6379> CONFIG SET requirepass "my_secure_password_123"
OK

# Now you need to authenticate
127.0.0.1:6379> PING
(error) NOAUTH Authentication required.

127.0.0.1:6379> AUTH my_secure_password_123
OK

127.0.0.1:6379> PING
PONG

# Check dangerous commands
127.0.0.1:6379> CONFIG GET "rename-command"
1) "rename-command"
2) ""
```

**Security configuration examples:**

```bash
# Basic security
requirepass your_very_strong_password_here
bind 127.0.0.1

# Enhanced security
requirepass your_very_strong_password_here
bind 127.0.0.1
protected-mode yes

# Disable dangerous commands
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""
rename-command CONFIG "CONFIG_a8b2c3d4e5f6"

# Enable TLS (Redis 6.0+)
port 0
tls-port 6379
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
```

---

## Configuration for Different Use Cases

### 1. Development Environment

**Settings for local development:**

```bash
# redis-dev.conf
# Easy to use, not secure (that's OK for dev)

bind 127.0.0.1
port 6379
# No password for easy development

# Small memory limit to avoid eating laptop RAM
maxmemory 512mb
maxmemory-policy allkeys-lru

# Minimal persistence (faster restarts)
save ""
appendonly no

# Verbose logging for debugging
loglevel debug
logfile /tmp/redis-dev.log
```

**Starting Redis with dev config:**
```bash
redis-server /path/to/redis-dev.conf
```

### 2. Production Cache Server

**Settings for high-performance caching:**

```bash
# redis-cache.conf
# Fast, can lose data, handles high load

bind 0.0.0.0
port 6379
requirepass "cache_password_xyz789"

# Large memory for cache
maxmemory 8gb
maxmemory-policy allkeys-lru

# No persistence (cache can be rebuilt)
save ""
appendonly no

# Performance optimizations
tcp-keepalive 300
timeout 300

# Reasonable logging
loglevel notice
logfile /var/log/redis/cache.log
```

### 3. Production Database Server

**Settings for reliable data storage:**

```bash
# redis-database.conf  
# Never lose data, reliable, secure

bind 127.0.0.1 10.0.1.100  # App servers only
port 6379
requirepass "database_password_abc123"

# Conservative memory settings
maxmemory 4gb
maxmemory-policy noeviction  # Never auto-delete data

# Full persistence
save "900 1 300 10 60 10000"
appendonly yes
appendfsync everysec
aof-use-rdb-preamble yes

# Data directories
dir /var/lib/redis
dbfilename dump.rdb
appendfilename appendonly.aof

# Enhanced security
protected-mode yes
rename-command FLUSHDB ""
rename-command FLUSHALL ""

# Production logging
loglevel notice
logfile /var/log/redis/database.log
syslog-enabled yes
```

### 4. Session Store

**Settings for web session storage:**

```bash
# redis-sessions.conf
# Balance of speed and reliability

bind 127.0.0.1
port 6379  
requirepass "sessions_password_def456"

# Moderate memory for sessions
maxmemory 2gb
maxmemory-policy volatile-lru  # Only remove expired sessions

# Light persistence (sessions can be lost occasionally)
save "300 10"
appendonly no

# Session optimization
timeout 0  # Don't timeout connections
tcp-keepalive 60

# Standard logging
loglevel notice
logfile /var/log/redis/sessions.log
```

---

## Working with Configuration

### Checking Current Configuration

```redis
# View all settings (lots of output!)
127.0.0.1:6379> CONFIG GET "*"

# View specific setting
127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "1073741824"

# View multiple related settings
127.0.0.1:6379> CONFIG GET "*memory*"
1) "maxmemory"
2) "1073741824"
3) "maxmemory-policy"
4) "allkeys-lru"
5) "maxmemory-samples"
6) "5"

# Search for settings by pattern
127.0.0.1:6379> CONFIG GET "*timeout*"
1) "timeout"
2) "0"
3) "tcp-keepalive"
4) "300"
```

### Making Temporary Changes

```redis
# Changes that don't survive restart
127.0.0.1:6379> CONFIG SET loglevel debug
OK

127.0.0.1:6379> CONFIG SET maxmemory 2147483648
OK

# Verify changes
127.0.0.1:6379> CONFIG GET loglevel
1) "loglevel"
2) "debug"

# Reset to default (if you know the default)
127.0.0.1:6379> CONFIG SET loglevel notice
OK
```

### Making Permanent Changes

**Method 1: Edit config file and restart**
```bash
# Edit redis.conf
sudo nano /etc/redis/redis.conf

# Add or change settings
maxmemory 2gb
maxmemory-policy allkeys-lru

# Restart Redis
sudo systemctl restart redis
# or
redis-server /etc/redis/redis.conf
```

**Method 2: Save runtime config to file**
```redis
# Make changes with CONFIG SET
127.0.0.1:6379> CONFIG SET maxmemory 2147483648
OK

# Save current config to file (overwrites redis.conf!)
127.0.0.1:6379> CONFIG REWRITE
OK
# Warning: This overwrites your config file!
```

---

## Configuration Validation and Testing

### Testing Configuration Changes

```redis
# Test memory settings
127.0.0.1:6379> CONFIG SET maxmemory 1048576  # 1MB
OK

# Fill memory to test eviction
127.0.0.1:6379> SET test1 "x"
OK
127.0.0.1:6379> SET test2 "x"  
OK
# ... add more data until eviction happens

# Check which keys remain
127.0.0.1:6379> KEYS "*"
1) "test2"
# test1 was evicted (LRU policy working)
```

### Validating Security Settings

```bash
# Test authentication
redis-cli -p 6379
127.0.0.1:6379> PING
(error) NOAUTH Authentication required.  # Good!

redis-cli -p 6379 -a your_password
127.0.0.1:6379> PING
PONG  # Authentication working

# Test bind settings
# Try connecting from different IP
redis-cli -h your_redis_ip -p 6379
# Should succeed or fail based on bind setting
```

### Common Configuration Errors

```redis
# Error 1: Invalid memory format
127.0.0.1:6379> CONFIG SET maxmemory "2GB"
(error) ERR invalid memory value  # Use bytes: 2147483648

# Error 2: Invalid policy name
127.0.0.1:6379> CONFIG SET maxmemory-policy "lru"
(error) ERR invalid maxmemory policy  # Use: allkeys-lru

# Error 3: Read-only settings
127.0.0.1:6379> CONFIG SET port 6380
(error) ERR Unsupported CONFIG parameter: port  # Requires restart
```

---

## Configuration Monitoring

### Important Settings to Monitor

```redis
# Memory settings
127.0.0.1:6379> CONFIG GET maxmemory
127.0.0.1:6379> INFO memory | grep used_memory_human

# Security settings  
127.0.0.1:6379> CONFIG GET requirepass
127.0.0.1:6379> CONFIG GET bind

# Persistence settings
127.0.0.1:6379> CONFIG GET save
127.0.0.1:6379> CONFIG GET appendonly

# Performance settings
127.0.0.1:6379> CONFIG GET timeout
127.0.0.1:6379> CONFIG GET tcp-keepalive
```

### Configuration Health Check Script

```bash
#!/bin/bash
# redis-config-check.sh

echo "=== Redis Configuration Health Check ==="

# Check if Redis is running
redis-cli ping > /dev/null 2>&1
if [ $? -ne 0 ]; then
    echo "❌ Redis is not responding"
    exit 1
fi

echo "✅ Redis is running"

# Check memory settings
MAX_MEM=$(redis-cli CONFIG GET maxmemory | tail -1)
if [ "$MAX_MEM" = "0" ]; then
    echo "⚠️  Warning: No memory limit set"
else
    echo "✅ Memory limit: $(($MAX_MEM / 1024 / 1024))MB"
fi

# Check authentication
PASS=$(redis-cli CONFIG GET requirepass | tail -1)
if [ -z "$PASS" ]; then
    echo "⚠️  Warning: No password set"
else
    echo "✅ Authentication enabled"
fi

# Check persistence
AOF=$(redis-cli CONFIG GET appendonly | tail -1)
SAVE=$(redis-cli CONFIG GET save | tail -1)

if [ "$AOF" = "yes" ] || [ -n "$SAVE" ]; then
    echo "✅ Persistence enabled"
else
    echo "⚠️  Warning: No persistence configured"
fi

echo "=== Health Check Complete ==="
```

---

## 🧪 Practical Configuration Exercises

### Exercise 1: Development Setup

```redis
# Configure Redis for local development
CONFIG SET maxmemory 268435456     # 256MB
CONFIG SET maxmemory-policy allkeys-lru
CONFIG SET save ""                 # No snapshots
CONFIG SET appendonly no           # No AOF
CONFIG SET loglevel debug          # Verbose logging

# Test the configuration
SET test:1 "value1"
SET test:2 "value2"  
INFO memory
```

### Exercise 2: Production Cache

```redis
# Configure for production caching
CONFIG SET maxmemory 1073741824    # 1GB
CONFIG SET maxmemory-policy allkeys-lru  
CONFIG SET timeout 300             # 5 minute timeout
CONFIG SET tcp-keepalive 60        # Keep connections alive

# Test eviction policy
# Fill memory and see what gets evicted
```

### Exercise 3: Security Hardening

```redis
# Add authentication
CONFIG SET requirepass "secure_password_123"

# Test authentication required
AUTH secure_password_123

# In config file, add:
# bind 127.0.0.1
# protected-mode yes
# rename-command FLUSHALL ""
```

---

## 📋 Configuration Checklist

### Before Going to Production

**Security:**
- [ ] Set strong password (`requirepass`)
- [ ] Bind to specific IPs (`bind`)
- [ ] Enable protected mode (`protected-mode yes`)
- [ ] Rename dangerous commands
- [ ] Consider TLS encryption

**Memory:**
- [ ] Set memory limit (`maxmemory`)
- [ ] Choose appropriate eviction policy
- [ ] Monitor memory usage regularly

**Persistence:**
- [ ] Choose RDB, AOF, or both
- [ ] Configure save intervals
- [ ] Test backup and recovery
- [ ] Monitor disk space

**Performance:**
- [ ] Set appropriate timeouts
- [ ] Configure TCP keepalive
- [ ] Optimize for your workload
- [ ] Monitor slow commands

**Monitoring:**
- [ ] Configure logging (`loglevel`, `logfile`)
- [ ] Set up health checks
- [ ] Monitor key metrics
- [ ] Plan for capacity growth

---

## 🎉 Configuration Mastery Complete!

You now understand:
- ✅ **How Redis configuration works** and different methods to change it
- ✅ **Essential settings** for network, memory, persistence, and security
- ✅ **Configuration patterns** for different use cases
- ✅ **Testing and validation** of configuration changes
- ✅ **Production considerations** and best practices
- ✅ **Monitoring and maintenance** of Redis configuration

## 🚀 What's Next?

Now that you understand how to configure Redis, let's dive into the core concepts and learn about Redis commands and operations in detail.

The next section will cover:
- Overview of all Redis commands
- String operations in depth
- Working with different data types
- Command patterns and best practices

---

**Up Next**: [Commands Overview](../core-concepts/commands-overview.md) - Master Redis commands!

[Continue to Commands Overview →](../core-concepts/commands-overview.md){ .md-button .md-button--primary }
