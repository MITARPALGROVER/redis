# Useful Redis Tools

Essential tools and utilities to work with Redis more effectively.

## Redis CLI Tools

### Redis CLI
The official command-line interface for Redis.

```bash
# Basic connection
redis-cli

# Connect to remote server
redis-cli -h hostname -p 6379

# Connect with authentication
redis-cli -a password

# Execute single command
redis-cli SET key value

# Interactive mode with password
redis-cli -a password
```

**Key Features:**

- Interactive command execution
- Batch command execution
- Monitoring and debugging
- Import/export capabilities

### Redis Benchmark
Built-in performance testing tool.

```bash
# Basic benchmark
redis-benchmark

# Test specific operations
redis-benchmark -t set,get -n 100000

# Test with different data sizes
redis-benchmark -d 1024

# Test with pipeline
redis-benchmark -P 16

# Custom benchmark
redis-benchmark -c 50 -n 10000 -t set
```

**Use Cases:**

- Performance testing
- Hardware evaluation
- Configuration optimization
- Load testing

## GUI Management Tools

### RedisInsight
Official Redis GUI by Redis Ltd.

**Features:**

- Visual key browser
- Real-time monitoring
- Query builder
- Memory analysis
- Cluster management

**Installation:**

- Download from Redis website
- Available for Windows, Mac, Linux
- Web-based interface
- Free to use

### Redis Desktop Manager
Popular third-party GUI tool.

**Features:**

- Cross-platform
- Key management
- Terminal integration
- SSH tunneling support
- Multiple database support

**Note:** Now requires license for newer versions.

### Another Redis Desktop Manager
Free open-source alternative.

**Features:**

- Similar to Redis Desktop Manager
- Completely free
- Regular updates
- Cross-platform support

## Monitoring Tools

### Redis Stat
Real-time Redis monitoring tool.

```bash
# Install via npm
npm install -g redis-stat

# Basic monitoring
redis-stat

# Monitor specific server
redis-stat --server localhost:6379

# Web interface
redis-stat --server localhost:6379 --daemon
```

**Features:**

- Real-time statistics
- Web dashboard
- Historical data
- Alert capabilities

### Redis Monitor
Built-in monitoring with redis-cli.

```bash
# Monitor all commands
redis-cli monitor

# Monitor with timestamp
redis-cli monitor | while read line; do echo "$(date): $line"; done
```

**Use Cases:**

- Debug application queries
- Performance analysis
- Security monitoring
- Development debugging

## Development Tools

### Redis Modules
Extend Redis functionality.

**Popular Modules:**

- **RedisJSON:** JSON data type support
- **RedisSearch:** Full-text search
- **RedisGraph:** Graph database
- **RedisTimeSeries:** Time series data
- **RedisBloom:** Probabilistic data structures

**Installation Example:**
```bash
# Load module
redis-server --loadmodule /path/to/module.so

# Or in redis.conf
loadmodule /path/to/module.so
```

### Redis Stack
All-in-one Redis package with modules.

**Includes:**

- Redis core
- RedisJSON
- RedisSearch
- RedisGraph
- RedisTimeSeries
- RedisBloom

**Installation:**
```bash
# Docker
docker run -d -p 6379:6379 redis/redis-stack:latest

# Or download installer
```

## Backup and Migration Tools

### Redis RDB Tools
Analyze and manipulate RDB files.

```bash
# Install
pip install rdbtools

# Parse RDB file
rdb --command json /path/to/dump.rdb

# Memory analysis
rdb -c memory /path/to/dump.rdb

# Filter by key pattern
rdb --command json /path/to/dump.rdb --key "user:*"
```

**Use Cases:**

- Memory analysis
- Data migration
- Backup analysis
- Performance optimization

### Redis Dump
Simple backup and restore tool.

```bash
# Install
npm install -g redis-dump

# Backup database
redis-dump > backup.json

# Restore database
cat backup.json | redis-load
```

## Configuration Management

### Redis Configuration Generator
Online tool to generate redis.conf files.

**Features:**

- Interactive configuration
- Best practice recommendations
- Environment-specific settings
- Download ready config files

### Redis Sentinel
High availability solution.

```bash
# Start Sentinel
redis-sentinel /path/to/sentinel.conf

# Sentinel configuration example
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
```

**Use Cases:**

- Automatic failover
- Master discovery
- Configuration management
- High availability

## Testing Tools

### Redis Test Suite
Built-in testing framework.

```bash
# Run all tests
make test

# Run specific test
./runtest --single unit/auth

# Run with specific tags
./runtest --tags basic
```

### Custom Testing Scripts
Create your own test scripts.

```bash
#!/bin/bash
# Simple Redis test script

echo "Testing Redis connection..."
redis-cli ping

echo "Testing basic operations..."
redis-cli set test_key "test_value"
redis-cli get test_key
redis-cli del test_key

echo "Test completed!"
```

## Cloud Tools

### Redis Cloud CLI
Manage Redis Cloud instances.

```bash
# Install
pip install rediscloud-cli

# Login
redis-cloud login

# List databases
redis-cloud database list

# Create database
redis-cloud database create
```

### AWS ElastiCache CLI
Manage AWS Redis instances.

```bash
# List clusters
aws elasticache describe-cache-clusters

# Create cluster
aws elasticache create-cache-cluster \
  --cache-cluster-id my-cluster \
  --engine redis
```

## Performance Tools

### Memory Analyzer
Built-in memory analysis.

```bash
# Get memory usage
redis-cli info memory

# Analyze specific key
redis-cli memory usage key_name

# Find biggest keys
redis-cli --bigkeys

# Memory doctor
redis-cli memory doctor
```

### Latency Monitoring
Built-in latency tracking.

```bash
# Monitor latency
redis-cli --latency

# Latency history
redis-cli --latency-history

# Latency distribution
redis-cli --latency-dist
```

## Scripting Tools

### Redis Lua Scripts
Server-side scripting support.

```lua
-- Example Lua script
local current = redis.call('GET', KEYS[1])
if current == false then
    current = 0
end
local new_value = current + ARGV[1]
redis.call('SET', KEYS[1], new_value)
return new_value
```

```bash
# Execute script
redis-cli eval "lua_script" 1 key_name 10
```

### Batch Processing Tools
Process multiple commands efficiently.

```bash
# Pipe commands from file
cat commands.txt | redis-cli --pipe

# CSV import
redis-cli --csv < data.csv
```

## Security Tools

### Redis ACL Manager
Manage user permissions (Redis 6+).

```bash
# List users
redis-cli acl list

# Create user
redis-cli acl setuser username on >password +@read ~cached:*

# Check permissions
redis-cli acl whoami
```

### SSL/TLS Tools
Secure Redis connections.

```bash
# Connect with TLS
redis-cli --tls --cert cert.pem --key key.pem --cacert ca.pem

# Generate certificates
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```

## Useful Scripts

### Health Check Script
Monitor Redis health.

```bash
#!/bin/bash
# Redis health check

REDIS_HOST="localhost"
REDIS_PORT="6379"

# Test connection
if redis-cli -h $REDIS_HOST -p $REDIS_PORT ping > /dev/null 2>&1; then
    echo "✅ Redis is responding"
else
    echo "❌ Redis is not responding"
    exit 1
fi

# Check memory usage
MEMORY_USAGE=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT info memory | grep used_memory_human | cut -d: -f2 | tr -d '\r')
echo "📊 Memory usage: $MEMORY_USAGE"

# Check connected clients
CLIENTS=$(redis-cli -h $REDIS_HOST -p $REDIS_PORT info clients | grep connected_clients | cut -d: -f2 | tr -d '\r')
echo "👥 Connected clients: $CLIENTS"
```

### Backup Script
Automated Redis backup.

```bash
#!/bin/bash
# Redis backup script

REDIS_HOST="localhost"
REDIS_PORT="6379"
BACKUP_DIR="/backup/redis"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
redis-cli -h $REDIS_HOST -p $REDIS_PORT bgsave

# Wait for backup to complete
while [ $(redis-cli -h $REDIS_HOST -p $REDIS_PORT lastsave) -eq $(redis-cli -h $REDIS_HOST -p $REDIS_PORT lastsave) ]; do
    sleep 1
done

# Copy RDB file
cp /var/lib/redis/dump.rdb $BACKUP_DIR/dump_$DATE.rdb

echo "Backup completed: $BACKUP_DIR/dump_$DATE.rdb"
```

## Quick Setup Commands

### Development Environment
```bash
# Start Redis in development mode
redis-server --daemonize yes --loglevel verbose --logfile redis.log

# Start with custom config
redis-server /path/to/dev-redis.conf
```

### Production Environment
```bash
# Start with production config
redis-server /etc/redis/redis.conf

# Start with Sentinel
redis-sentinel /etc/redis/sentinel.conf
```

### Docker Setup
```bash
# Basic Redis container
docker run -d --name redis -p 6379:6379 redis:latest

# Redis with persistence
docker run -d --name redis -p 6379:6379 -v redis-data:/data redis:latest

# Redis Stack
docker run -d --name redis-stack -p 6379:6379 redis/redis-stack:latest
```

These tools will help you manage, monitor, and optimize your Redis installations effectively. Choose the ones that best fit your workflow and requirements.
