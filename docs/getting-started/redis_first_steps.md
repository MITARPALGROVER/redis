# First Steps with Redis

Congratulations! You have Redis installed. Now let's take your first steps and get comfortable with the basics. Think of this as your "Hello World" moment with Redis.

## Starting Your Redis Journey

### Step 1: Starting the Redis Server

The Redis server needs to be running before you can connect to it. Here's how to start it, depending on your system:

#### Docker
If you installed Redis using Docker, you have two options:

**Option 1**: If you already created a Redis container before, this starts it again:
```bash title="Docker"
docker start my-redis
```

**OR**

**Option 2**: If you haven't created a container yet, this creates and starts a new one:
```bash title="Docker"
docker run --name my-redis -p 6379:6379 -d redis:latest
```

**What these parameters mean:**

- `--name my-redis`: Gives your container a name for easy reference
- `-p 6379:6379`: Maps port 6379 from inside the container to port 6379 on your computer

    - **First 6379**: Port on your computer (host)
    - **Second 6379**: Port inside the Docker container
    - **Why both same?** Redis uses port 6379 by default, so we keep it consistent

- `-d`: Runs Redis in the background (detached mode)
- `redis:latest`: Uses the latest Redis version

!!! info "Port Mapping Explained"
    Think of Docker containers like separate computers. The `-p 6379:6379` creates a "bridge" so when you connect to port 6379 on your computer, it forwards the connection to port 6379 inside the Redis container. This is why you can use `redis-cli` from your computer to talk to Redis running inside Docker.

#### macOS (Homebrew)
If you installed Redis using Homebrew on a Mac, you have two options:

**Option 1 (Recommended)**: Start Redis as a background service - keeps running even if you close terminal:
```bash title="macOS (Homebrew)"
brew services start redis
```

**OR**

**Option 2**: Start Redis manually in your current terminal window - stops when you close terminal:
```bash title="macOS (Homebrew)"
redis-server
```

#### Linux
If you installed Redis on Linux, you have two options:

**Option 1 (Recommended)**: Start Redis as a background service:
```bash title="Linux"
sudo systemctl start redis
```

**OR**

**Option 2**: Start Redis manually in your current terminal window (for quick tests or development):
```bash title="Linux"
redis-server
```

#### Windows
If you installed Redis on Windows:

**Step 1**: Move to the folder where you installed Redis:
```cmd title="Windows"
cd C:\Redis
```

**OR**

**Step 2**: Start the Redis server:
```cmd title="Windows"
redis-server.exe
```

!!! tip "Background vs Foreground"
    - **Service/Background**: Redis runs in the background, terminal is free to use
    - **Manual/Foreground**: Redis runs in the current terminal window, shows logs

### Step 2: Verify Redis is Running

Now let's check if Redis is running properly. The command depends on how you installed Redis:

#### For Docker Users:
If you're using Docker, you need to run the command **inside** the Docker container:

```bash
docker exec -it my-redis redis-cli ping
```

**Let's break down this command:**

- `docker exec`: Execute a command in a running Docker container
- `-it`: Two flags combined:
  - `-i`: Interactive mode (keep input open)
  - `-t`: Allocate a pseudo-terminal (makes it look like a real terminal)
- `my-redis`: The name of your Redis container
- `redis-cli ping`: The actual Redis command to run inside the container

**Think of it like this:** You're telling Docker "Go into my Redis container and run the `redis-cli ping` command for me"

---

#### For Native Installation (Windows, macOS, Linux):
If you installed Redis directly on your system:

```bash
redis-cli ping
```

**Expected response for both**: `PONG`

!!! info "Why Different Commands?"
    - **Docker**: `redis-cli` is inside the container, so you need `docker exec` to run it
    - **Native**: `redis-cli` is installed directly on your system

!!! success "It's Alive!"
    If you see `PONG`, Redis is running and ready to accept connections!

---

## Understanding Redis Server Information

Before we start storing data, let's learn about our Redis instance:

### Basic Server Information

```redis
# Connect to Redis
redis-cli

# Get basic server info
127.0.0.1:6379> INFO server
```

You'll see output like this:
```
# Server
redis_version:8.2.1
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:123456789
redis_mode:standalone
os:Linux 5.4.0-74-generic x86_64
arch_bits:64
uptime_in_seconds:3600
```

### Detailed Field Explanations

| Field | What It Means | Detailed Explanation |
|-------|---------------|---------------------|
| `redis_version` | Your Redis version (e.g., 8.2.1) | **Why it matters**: Different versions have different features. Version 7.x has functions, 6.x has modules, 5.x has streams. Always check compatibility when following tutorials. |
| `redis_mode` | How Redis is configured | **standalone**: Single Redis server (most common for beginners)<br/>**cluster**: Multiple Redis servers working together<br/>**sentinel**: High-availability setup with automatic failover |
| `os` | Operating system Redis is running on | **Examples**: `Linux 5.4.0`, `Darwin 21.6.0` (macOS), `Windows 10`<br/>**Why useful**: Helps with performance tuning and troubleshooting OS-specific issues |
| `arch_bits` | CPU architecture (32-bit or 64-bit) | **64**: Can handle larger datasets and memory<br/>**32**: Limited to ~3GB memory<br/>**Modern systems**: Almost always 64-bit |
| `uptime_in_seconds` | How long Redis has been running | **Examples**: `3600` = 1 hour, `86400` = 1 day<br/>**Use case**: Check server stability - longer uptime usually means stable operation |

---

### Memory Information

```redis
127.0.0.1:6379> INFO memory
```

You'll see output like this:
```
# Memory
used_memory:1048576
used_memory_human:1.00M
used_memory_rss:8912896
used_memory_rss_human:8.50M
maxmemory:0
maxmemory_human:0B
mem_fragmentation_ratio:8.50
```

### Detailed Memory Field Explanations

| Field | What It Shows | Detailed Explanation |
|-------|---------------|---------------------|
| `used_memory_human` | Memory Redis is actually using for data | **Example**: `1.00M` = 1 megabyte<br/>**What it includes**: Your stored keys, values, and Redis overhead<br/>**Good range**: Depends on your data, but should grow predictably with your dataset |
| `used_memory_rss_human` | Total memory Redis process uses (from OS perspective) | **Example**: `8.50M` = 8.5 megabytes<br/>**What it includes**: Redis data + operating system overhead + memory fragmentation<br/>**Why higher**: OS allocates memory in chunks, not exact amounts |
| `maxmemory_human` | Maximum memory limit (if set) | **0B**: No limit set (Redis can use all available RAM)<br/>**128M**: Limited to 128 megabytes<br/>**Why set limits**: Prevents Redis from crashing your server by using all memory |
| `mem_fragmentation_ratio` | Memory efficiency indicator | **Good**: 1.0-1.5 (very efficient)<br/>**OK**: 1.5-2.0 (acceptable)<br/>**Concerning**: >2.0 (wasting memory)<br/>**How to fix**: Restart Redis to defragment memory |

---

### Memory Monitoring Best Practices

#### Check Memory Regularly
```redis
# Quick memory check
127.0.0.1:6379> INFO memory | grep used_memory_human
used_memory_human:1.00M

# Check if you're hitting limits
127.0.0.1:6379> INFO memory | grep maxmemory_human  
maxmemory_human:128.00M
```

#### Warning Signs to Watch For
- **Fragmentation ratio > 2.0**: Consider restarting Redis
- **Used memory near max limit**: Add more memory or reduce data
- **Memory growing unexpectedly**: Check for memory leaks in your application

#### Sample Memory Health Check
```redis
# Get all key memory info at once
127.0.0.1:6379> INFO memory
# Look for these key indicators:
# - used_memory_human: Should match your expectations
# - mem_fragmentation_ratio: Should be < 2.0
# - maxmemory_human: Should be set in production
```

!!! tip "Memory Management Tips"
    - **Development**: Usually don't need memory limits
    - **Production**: Always set `maxmemory` to prevent crashes  
    - **High fragmentation**: Restart Redis during low-traffic periods
    - **Growing memory**: Use `MEMORY USAGE keyname` to find large keys

---

## Your First Redis Commands

Let's start with the most basic Redis operations:

### The Classic "Hello World"

```redis
# Store a simple value
127.0.0.1:6379> SET greeting "Hello Redis!"
OK

# Retrieve the value
127.0.0.1:6379> GET greeting
"Hello Redis!"
```

!!! example "Try This Yourself"
    1. Open `redis-cli`
    2. Type the `SET` command exactly as shown
    3. Type the `GET` command
    4. You should see your greeting message!

### Storing Different Types of Data

```redis
# Store text
127.0.0.1:6379> SET username "john_doe"
OK

# Store numbers (Redis treats them as strings)
127.0.0.1:6379> SET age "25"
OK

# Store JSON-like data
127.0.0.1:6379> SET user_profile '{"name":"John","city":"NYC"}'
OK
```

### Checking What's in Redis

```redis
# List all keys
127.0.0.1:6379> KEYS *
1) "greeting"
2) "username" 
3) "age"
4) "user_profile"

# Check if a key exists
127.0.0.1:6379> EXISTS username
(integer) 1

# Check if a non-existent key exists
127.0.0.1:6379> EXISTS nonexistent
(integer) 0
```

### Working with Numbers

```redis
# Set a counter
127.0.0.1:6379> SET page_views 0
OK

# Increment the counter
127.0.0.1:6379> INCR page_views
(integer) 1

# Increment by a specific amount
127.0.0.1:6379> INCRBY page_views 10
(integer) 11

# Decrement
127.0.0.1:6379> DECR page_views
(integer) 10
```

---

## Working with Expiration

One of Redis's superpowers is automatic data expiration:

### Setting Expiration Time

```redis
# Set a key that expires in 60 seconds
127.0.0.1:6379> SETEX temporary_data 60 "This will disappear"
OK

# Or set expiration after creating the key
127.0.0.1:6379> SET another_key "some value"
OK
127.0.0.1:6379> EXPIRE another_key 30
(integer) 1
```

**What do these numbers mean?**

- **60**: This means 60 **seconds** (1 minute)
  - The key `temporary_data` will automatically disappear after 60 seconds
  - Think of it like a timer: "Delete this data in 60 seconds"
  
- **30**: This means 30 **seconds** 
  - The key `another_key` will automatically disappear after 30 seconds
  - It's like setting a shorter timer: "Delete this data in 30 seconds"

**Real-world example:**
```redis
# Cache user session for 15 minutes (900 seconds)
SETEX user:session:abc123 900 "session_data"

# Cache product info for 1 hour (3600 seconds)  
SETEX product:123 3600 "product_details"

# Temporary verification code for 5 minutes (300 seconds)
SETEX verify:email:user123 300 "verification_code_xyz"
```

### Checking Time-to-Live

```redis
# Check how many seconds until expiration
127.0.0.1:6379> TTL temporary_data
(integer) 45

# Check again after some time
127.0.0.1:6379> TTL temporary_data
(integer) 20

# After expiration, the key is gone
127.0.0.1:6379> GET temporary_data
(nil)
```

!!! note "TTL Return Values"
    - **Positive number**: Seconds until expiration
    - **-1**: Key exists but has no expiration
    - **-2**: Key does not exist

---

## Cleaning Up

### Removing Specific Keys

```redis
# Delete a single key
127.0.0.1:6379> DEL username
(integer) 1

# Delete multiple keys
127.0.0.1:6379> DEL greeting age user_profile
(integer) 3
```

### Clearing All Data (Use Carefully!)

```redis
# Clear all data in the current database
127.0.0.1:6379> FLUSHDB
OK

# Clear all data in all databases (DANGEROUS!)
127.0.0.1:6379> FLUSHALL
OK
```

!!! warning "Be Careful with FLUSH Commands"
    These commands permanently delete data. Only use them in development/testing!

---

## Monitoring Redis Activity

### Real-time Command Monitoring

Open two terminals:

**Terminal 1** (Monitor):
```bash
redis-cli MONITOR
```

**Terminal 2** (Execute commands):
```bash
redis-cli
127.0.0.1:6379> SET test "monitoring example"
127.0.0.1:6379> GET test
```

You'll see all commands appear in Terminal 1 in real-time!

---

### Basic Statistics

```redis
# Get overall statistics
127.0.0.1:6379> INFO stats

# See connected clients
127.0.0.1:6379> CLIENT LIST

# Check database size
127.0.0.1:6379> DBSIZE
```

**What do these commands do?**

#### 1. INFO stats - Redis Performance Statistics
This command shows you how busy and healthy your Redis server is:

**Sample Output:**
```
# Stats
total_connections_received:100
total_commands_processed:5000
instantaneous_ops_per_sec:15
total_net_input_bytes:1024000
total_net_output_bytes:2048000
rejected_connections:0
expired_keys:25
evicted_keys:0
```

**Key Fields Explained:**

- **total_connections_received**: How many clients have connected since Redis started
    - Example: `100` = 100 different connections made
    - **Why useful**: Shows if your app is connecting properly
  
- **total_commands_processed**: Total number of Redis commands executed
    - Example: `5000` = 5,000 GET, SET, etc. commands run
    - **Why useful**: Measures Redis activity level
  
- **instantaneous_ops_per_sec**: Commands being processed right now (per second)
    - Example: `15` = 15 commands happening per second currently
    - **Why useful**: Shows current load - higher = busier server
  
- **expired_keys**: Keys that automatically disappeared due to TTL
    - Example: `25` = 25 keys expired and were deleted
    - **Why useful**: Shows if your expiration strategy is working

#### 2. CLIENT LIST - Who's Connected Right Now
This shows all active connections to your Redis server:

**Sample Output:**
```
id=3 addr=127.0.0.1:54320 name= age=60 idle=10 flags=N db=0 sub=0 psub=0
id=4 addr=192.168.1.100:43210 name=webapp age=120 idle=5 flags=N db=0 sub=0 psub=0
```

**What each part means:**

- **id=3**: Unique connection ID (Redis assigns this)
- **addr=127.0.0.1:54320**: IP address and port of the client
    - `127.0.0.1` = local computer (your machine)
    - `192.168.1.100` = another computer on network
  
- **age=60**: How long this connection has been open (60 seconds)
- **idle=10**: How long since this client sent a command (10 seconds)
- **db=0**: Which Redis database they're using (0 is default)

**Real-world use cases:**

- **Debugging**: "Why is Redis slow?" - Check if too many clients connected
- **Security**: "Who's accessing my Redis?" - See all IP addresses
- **Monitoring**: "Is my app still connected?" - Look for your app's connection

#### 3. DBSIZE - How Much Data Do I Have?
This tells you exactly how many keys (pieces of data) are stored:

**Sample Output:**
```
(integer) 1547
```

**What this means:**

- **1547**: You have 1,547 different keys stored in Redis
- Each key could be: a string, number, list, hash, etc.
- **Empty database**: Returns `(integer) 0`

**Practical examples:**
```redis
# Start with empty database
127.0.0.1:6379> DBSIZE
(integer) 0

# Add some data
127.0.0.1:6379> SET user:1 "John"
127.0.0.1:6379> SET user:2 "Jane" 
127.0.0.1:6379> SET counter 100

# Check size again
127.0.0.1:6379> DBSIZE
(integer) 3
```

**Why this is useful:**

- **Performance**: More keys = potentially slower operations
- **Memory planning**: Estimate how much data you're storing
- **Debugging**: "Did my data import work?" - Check if key count increased
- **Monitoring**: Track data growth over time

**Quick Health Check Routine:**
```redis
# 1. Check how busy Redis is
INFO stats

# 2. See who's connected
CLIENT LIST

# 3. Count total data
DBSIZE

# 4. Check memory usage
INFO memory
```

---

## Basic Configuration

### Viewing Current Configuration

```redis
# See all configuration
127.0.0.1:6379> CONFIG GET "*"

# See specific configuration
127.0.0.1:6379> CONFIG GET "maxmemory*"
```

### Making Simple Changes

```redis
# Set a memory limit (128MB)
127.0.0.1:6379> CONFIG SET maxmemory 134217728

# Set password (for security)
127.0.0.1:6379> CONFIG SET requirepass "mypassword"

# After setting password, you need to authenticate
127.0.0.1:6379> AUTH mypassword
```

!!! tip "Configuration Persistence"
    Changes made with `CONFIG SET` are temporary. To make them permanent, use `CONFIG REWRITE` or edit the Redis configuration file.

---

## Starting and Stopping Redis Safely

### Graceful Shutdown

```redis title="From Redis CLI"
127.0.0.1:6379> SHUTDOWN
```

```bash title="Docker"
docker stop my-redis
```

```bash title="Linux Service"
sudo systemctl stop redis
```

```bash title="macOS Service"
brew services stop redis
```

```cmd title="Windows"
# If Redis is running in a terminal window, press Ctrl+C
# Or if you need to force stop the process:
taskkill /F /IM redis-server.exe
```

### Restart Redis

```bash title="Docker"
docker restart my-redis
```

```bash title="Linux Service"
sudo systemctl restart redis
```

```bash title="macOS Service"
brew services restart redis
```

```cmd title="Windows"
# Navigate to Redis folder and start again
cd C:\Redis
redis-server.exe
```

**Best Practice for Windows:**
```cmd
# 1. Stop Redis gently
# Press Ctrl+C in the Redis terminal window

# 2. Wait for "Redis is now ready to exit, bye bye..." message

# 3. Start Redis again
cd C:\Redis
redis-server.exe
```

---

## Practice Exercises

Try these exercises to solidify your understanding:

### Exercise 1: User Profile Storage
```redis
# Store a user's information
SET user:1001:name "Alice Johnson"
SET user:1001:email "alice@example.com"  
SET user:1001:age "28"

# Retrieve the information
GET user:1001:name
GET user:1001:email
GET user:1001:age
```

### Exercise 2: Temporary Cache
```redis
# Create a cache entry that expires in 5 minutes
SETEX cache:product:123 300 "Product details here"

# Check how much time is left
TTL cache:product:123

# Wait a minute and check again
TTL cache:product:123
```

### Exercise 3: Simple Counter
```redis
# Initialize counters
SET visitors:today 0
SET visitors:total 1000

# Simulate some visits
INCR visitors:today
INCR visitors:total
INCRBY visitors:today 5

# Check the results
GET visitors:today
GET visitors:total
```

---

## Congratulations!

You've successfully:

- Started and connected to Redis
- Executed your first Redis commands
- Stored and retrieved data
- Worked with expiration
- Monitored Redis activity
- Learned basic server management
