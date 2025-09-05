# Memory and Persistence

Understanding how Redis manages memory and persists data is crucial for building reliable applications. Let's explore Redis's memory model and persistence options!

## How Redis Uses Memory

Redis is an **in-memory database**, which means all your data is stored in RAM for lightning-fast access. But what does this really mean?

### Memory vs Disk Storage

| Aspect | Traditional Database | Redis |
|--------|---------------------|--------|
| **Data Location** | Hard disk/SSD | RAM (memory) |
| **Access Speed** | Milliseconds | Microseconds |
| **Data Survival** | Survives restarts | Requires persistence setup |
| **Storage Cost** | Cheap | More expensive |
| **Capacity** | Very large (TBs) | Limited by RAM (GBs) |

### Why Memory Makes Redis Fast

```redis
# Traditional database query: ~10-100ms
SELECT * FROM users WHERE id = 1001;

# Redis equivalent: ~0.1ms
127.0.0.1:6379> GET user:1001
```

The speed difference comes from:

- **No disk I/O** - data is already in memory
- **Simple data structures** - optimized for speed
- **No query parsing** - direct key-value access

---

## Redis Memory Management

### Memory Allocation

When you store data in Redis:

```redis
# Each operation allocates memory
127.0.0.1:6379> SET user:1001 "John Doe"           # ~64 bytes
127.0.0.1:6379> HSET profile:1001 name "John"      # ~96 bytes  
127.0.0.1:6379> LPUSH queue:tasks "task1"          # ~80 bytes
```

Redis allocates memory for:

- **Keys** (the identifiers)
- **Values** (the actual data)
- **Data structure overhead** (pointers, metadata)
- **Expiration information** (if TTL is set)

### Checking Memory Usage

```redis
# Overall memory statistics
127.0.0.1:6379> INFO memory
# Memory
used_memory:1048576
used_memory_human:1.00M
used_memory_rss:2097152
used_memory_peak:1200000
used_memory_peak_human:1.14M

# Memory usage of specific key
127.0.0.1:6379> MEMORY USAGE user:1001
(integer) 64

# Memory usage with detailed breakdown
127.0.0.1:6379> MEMORY USAGE user:1001 SAMPLES 5
(integer) 64
```

### Key Memory Metrics

| Metric | Description | Why It Matters |
|--------|-------------|----------------|
| `used_memory` | Memory used by Redis | Your actual data size |
| `used_memory_rss` | Memory used by OS | Includes fragmentation |
| `used_memory_peak` | Highest memory usage | Plan for peak loads |
| `mem_fragmentation_ratio` | Fragmentation level | Memory efficiency |

```redis
# Calculate fragmentation ratio
127.0.0.1:6379> INFO memory | grep fragmentation
mem_fragmentation_ratio:1.25
```

**Fragmentation ratios:**

- **< 1.0**: Memory swapping (bad)
- **1.0-1.5**: Normal range (good)
- **> 1.5**: High fragmentation (consider optimization)

---

## Memory Policies

What happens when Redis runs out of memory? You can configure different policies:

### Available Memory Policies

```redis
# Check current policy
127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"

# Set memory limit (128MB)
127.0.0.1:6379> CONFIG SET maxmemory 134217728

# Set eviction policy
127.0.0.1:6379> CONFIG SET maxmemory-policy allkeys-lru
```

### Eviction Policies

| Policy | Description | Best For |
|--------|-------------|----------|
| **noeviction** | Return errors when memory limit reached | Databases where all data is critical |
| **allkeys-lru** | Remove least recently used keys | General caching |
| **allkeys-lfu** | Remove least frequently used keys | Caches with clear usage patterns |
| **volatile-lru** | Remove LRU keys with TTL set | Mixed workloads |
| **volatile-lfu** | Remove LFU keys with TTL set | Cache with some permanent data |
| **allkeys-random** | Remove random keys | When access patterns are unpredictable |
| **volatile-random** | Remove random keys with TTL | Simple cache cleanup |
| **volatile-ttl** | Remove keys with shortest TTL | Time-sensitive cache data |

### Practical Policy Examples

```redis
# Cache server - evict any key when memory full
CONFIG SET maxmemory-policy allkeys-lru
CONFIG SET maxmemory 2gb

# Session store - only evict temporary data
CONFIG SET maxmemory-policy volatile-lru
CONFIG SET maxmemory 1gb

# Database - never lose data automatically
CONFIG SET maxmemory-policy noeviction
CONFIG SET maxmemory 8gb
```

---

## Redis Persistence Options

Redis offers two main persistence methods to survive server restarts:

### 1. RDB (Redis Database Backup)

**What is RDB?** Think of RDB like taking a photo of all your data at one moment in time. It creates a single file that contains everything.

**RDB** creates **point-in-time snapshots** of your data.

```redis
# Manual snapshot - tell Redis to take a photo of all data right now
127.0.0.1:6379> BGSAVE
Background saving started

# Check when we last took a photo
127.0.0.1:6379> LASTSAVE
(integer) 1705751400
# This number is Unix timestamp = Mon Jan 20 2025 15:30:00 GMT

# Force synchronous save (blocks Redis - be careful!)
127.0.0.1:6379> SAVE
OK
# Warning: This freezes Redis until save is complete
```

**Understanding BGSAVE vs SAVE:**

- **BGSAVE** = Background save (Redis keeps working while saving)
- **SAVE** = Foreground save (Redis stops accepting commands until done)

#### RDB Configuration

```redis
# Save snapshot if at least 1 key changed in 900 seconds (15 minutes)
127.0.0.1:6379> CONFIG SET save "900 1"
OK

# Multiple save conditions (Redis checks ALL of these)
127.0.0.1:6379> CONFIG SET save "900 1 300 10 60 10000"
OK
# Save if: 1 change in 15 minutes OR 10 changes in 5 minutes OR 10000 changes in 1 minute

# Check current save settings
127.0.0.1:6379> CONFIG GET save
1) "save"
2) "900 1 300 10 60 10000"

# Disable RDB completely
127.0.0.1:6379> CONFIG SET save ""
OK
```

**What these numbers mean:**

- `900 1` = If 1 key changes, save after 900 seconds (15 minutes)
- `300 10` = If 10 keys change, save after 300 seconds (5 minutes)  
- `60 10000` = If 10,000 keys change, save after 60 seconds (1 minute)

#### RDB Pros and Cons

| Pros | Cons |
|---------|---------|
| **Compact** single file (easy to backup) | **Data loss** between snapshots (could lose 15 min of data) |
| **Fast recovery** on restart (just load one file) | **CPU intensive** during saves (temporary slowdown) |
| **Good for backups** (copy one file) | **Fork can fail** on low memory (needs 2x memory briefly) |
| **Minimal performance impact** (saves in background) | **Not real-time** persistence (gap between saves) |

### 2. AOF (Append Only File)

**What is AOF?** Think of AOF like keeping a diary. Every time you do something (write data), Redis writes it down in a log file. If Redis crashes, it can replay the diary to get back all your data.

**AOF** logs **every write command** for complete durability.

```redis
# Enable AOF (start keeping the diary)
127.0.0.1:6379> CONFIG SET appendonly yes
OK

# Check if AOF is enabled
127.0.0.1:6379> CONFIG GET appendonly
1) "appendonly"
2) "yes"

# AOF sync policies (how often to write diary to disk)
127.0.0.1:6379> CONFIG SET appendfsync always    # Write every command (slowest, safest)
127.0.0.1:6379> CONFIG SET appendfsync everysec  # Write every second (balanced)
127.0.0.1:6379> CONFIG SET appendfsync no        # Let OS decide when to write (fastest, risky)

# Manual AOF rewrite (clean up and compress the diary)
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
```

**Understanding AOF sync options:**

- **always** = Every command written to disk immediately (very safe, but slower)
- **everysec** = Write to disk once per second (good balance - at most 1 second of data loss)
- **no** = Operating system decides when to write (faster but could lose more data if crash)

#### AOF Configuration

```redis
# Auto-rewrite AOF when it grows 100% bigger and is at least 64MB
127.0.0.1:6379> CONFIG SET auto-aof-rewrite-percentage 100
OK
127.0.0.1:6379> CONFIG SET auto-aof-rewrite-min-size 67108864
OK

# Check AOF status and get detailed information
127.0.0.1:6379> INFO persistence
# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1705751400
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
aof_enabled:1
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_current_size:1024
aof_base_size:1024
```

**What auto-rewrite means:**
AOF files can get very large because they record every command. Rewriting compresses the file by replacing multiple commands with the final result. Example:
```
Original AOF: SET counter 1, INCR counter, INCR counter, INCR counter
After rewrite: SET counter 4
```

#### AOF Pros and Cons

| Pros | Cons |
|---------|---------|
| **Minimal data loss** (at most 1 second with everysec) | **Larger files** than RDB (stores every command) |
| **Readable format** (actual Redis commands you can read) | **Slower recovery** time (must replay all commands) |
| **Incremental backups** possible (append-only file) | **More disk I/O** (constantly writing to file) |
| **Corruption resilient** (can fix partial corruption) | **Can grow very large** without rewriting |

---

## Persistence Strategies

### Strategy 1: RDB Only (Snapshots)
**Use when:** You can afford to lose recent data, need fast restarts

**Simple explanation:** Like taking photos of your data every few minutes. If Redis crashes, you lose data since the last photo.

```redis
# Configure RDB snapshots
127.0.0.1:6379> CONFIG SET save "900 1 300 10 60 10000"
OK
127.0.0.1:6379> CONFIG SET appendonly no
OK
127.0.0.1:6379> CONFIG SET dbfilename "dump.rdb"
OK
127.0.0.1:6379> CONFIG SET dir "/var/lib/redis"
OK

# Verify settings
127.0.0.1:6379> CONFIG GET save
1) "save"
2) "900 1 300 10 60 10000"
```

**Example scenario:** Web page cache, where losing 15 minutes of cache data is acceptable because you can regenerate it.

### Strategy 2: AOF Only (Maximum Durability)
**Use when:** You can't afford to lose any data

**Simple explanation:** Like keeping a detailed diary of everything you do. Takes more space and time, but you won't lose anything important.

```redis
# Configure AOF
127.0.0.1:6379> CONFIG SET appendonly yes
OK
127.0.0.1:6379> CONFIG SET appendfsync everysec
OK
127.0.0.1:6379> CONFIG SET save ""  # Disable RDB
OK

# Verify AOF is working
127.0.0.1:6379> INFO persistence | grep aof_enabled
aof_enabled:1
```

**Example scenario:** User session storage, financial transactions where losing data means losing money or angry users.

### Strategy 3: Both RDB and AOF (Hybrid)
**Use when:** You want durability AND fast recovery

**Simple explanation:** Take photos AND keep a diary. Best of both worlds but uses more disk space.

```redis
# Enable both persistence methods
127.0.0.1:6379> CONFIG SET appendonly yes
OK
127.0.0.1:6379> CONFIG SET appendfsync everysec
OK
127.0.0.1:6379> CONFIG SET save "900 1 300 10 60 10000"
OK

# RDB-AOF hybrid mode (Redis 4.0+) - makes recovery faster
127.0.0.1:6379> CONFIG SET aof-use-rdb-preamble yes
OK

# Check both are enabled
127.0.0.1:6379> INFO persistence | grep -E "(rdb_last_save_time|aof_enabled)"
rdb_last_save_time:1705751400
aof_enabled:1
```

**What hybrid mode does:** When Redis restarts, it loads the RDB snapshot first (fast), then replays any AOF commands since the snapshot (complete recovery).

**Example scenario:** Production database where uptime and data safety are both critical - can't afford downtime OR data loss.

### Strategy 4: No Persistence (Cache Only)
**Use when:** All data can be regenerated

**Simple explanation:** Don't save anything to disk. Fastest performance but all data disappears when Redis restarts.

```redis
# Disable all persistence
127.0.0.1:6379> CONFIG SET save ""
OK
127.0.0.1:6379> CONFIG SET appendonly no
OK

# Verify persistence is disabled
127.0.0.1:6379> CONFIG GET save
1) "save"
2) ""
127.0.0.1:6379> CONFIG GET appendonly
1) "appendonly"
2) "no"
```

**Example scenario:** Temporary cache, computed results that can be recalculated, or session data where users can just log in again.

---

## Memory Optimization Techniques

### 1. Choose Efficient Data Types

**Why this matters:** Different data types use different amounts of memory. Choosing the right one can save a lot of space.

```redis
# Less efficient: Multiple string keys (uses more memory)
127.0.0.1:6379> SET user:1001:name "John"
OK
127.0.0.1:6379> SET user:1001:email "john@example.com"
OK
127.0.0.1:6379> SET user:1001:age "30"
OK

# Check memory usage of each key
127.0.0.1:6379> MEMORY USAGE user:1001:name
(integer) 64
127.0.0.1:6379> MEMORY USAGE user:1001:email
(integer) 80
127.0.0.1:6379> MEMORY USAGE user:1001:age
(integer) 56
# Total: 64 + 80 + 56 = 200 bytes

# More efficient: Single hash (uses less memory)
127.0.0.1:6379> DEL user:1001:name user:1001:email user:1001:age
(integer) 3
127.0.0.1:6379> HSET user:1001 name "John" email "john@example.com" age "30"
(integer) 3

# Check memory usage of hash
127.0.0.1:6379> MEMORY USAGE user:1001
(integer) 144
# Total: 144 bytes (saved 56 bytes = 28% less memory!)
```

**Why hashes are more efficient:** Redis has special optimizations for small hashes that pack data more tightly.

### 2. Use Appropriate Value Sizes

**Understanding memory overhead:** Every key and value has some "overhead" - extra bytes Redis needs for bookkeeping.

```redis
# Let's see how memory usage changes with value size
127.0.0.1:6379> SET small_string "hi"
OK
127.0.0.1:6379> MEMORY USAGE small_string
(integer) 56
# 2 characters + 54 bytes overhead

127.0.0.1:6379> SET medium_string "This is a medium length string with some content"
OK
127.0.0.1:6379> MEMORY USAGE medium_string
(integer) 104
# 49 characters + 55 bytes overhead

127.0.0.1:6379> SET large_string "This is a very long string that contains a lot of text and demonstrates how memory usage scales with content size. The overhead becomes less significant as the content grows larger."
OK
127.0.0.1:6379> MEMORY USAGE large_string
(integer) 240
# 186 characters + 54 bytes overhead
```

**Key insight:** For small values, overhead is a big percentage. For large values, overhead becomes less important.

### 3. Enable Compression

**What is compression?** Redis can squeeze small data structures to use less memory, like compressing a zip file.

```redis
# Configure hash compression (small hashes stored efficiently)
127.0.0.1:6379> CONFIG SET hash-max-ziplist-entries 512
OK
127.0.0.1:6379> CONFIG SET hash-max-ziplist-value 64
OK

# Configure list compression
127.0.0.1:6379> CONFIG SET list-max-ziplist-size -2     # 8KB nodes
OK
127.0.0.1:6379> CONFIG SET list-compress-depth 1       # Compress all but first/last
OK

# Test compression with a small hash
127.0.0.1:6379> HSET compressed:test field1 "value1" field2 "value2" field3 "value3"
(integer) 3
127.0.0.1:6379> DEBUG OBJECT compressed:test
Value at:0x7fb8a1c0c1e0 refcount:1 encoding:ziplist serializedlength:45 lru:677 lru_seconds_idle:5
# "encoding:ziplist" means Redis is using compression!
```

**What these settings mean:**

- `hash-max-ziplist-entries 512` = Use compression for hashes with up to 512 fields
- `hash-max-ziplist-value 64` = Use compression when each value is under 64 bytes
- `list-compress-depth 1` = Compress all list nodes except the first and last (for fast access to ends)

### 4. Set Appropriate TTL

**Why TTL helps:** Automatic cleanup prevents memory from filling up with old data.

```redis
# Clean up temporary data automatically
127.0.0.1:6379> SETEX cache:temp 300 "temporary_data"      # 5 minutes
OK
127.0.0.1:6379> SETEX session:user123 7200 "session"      # 2 hours
OK
127.0.0.1:6379> SETEX rate_limit:user123 60 "5"           # 1 minute
OK

# Check how much time is left
127.0.0.1:6379> TTL cache:temp
(integer) 287
# 287 seconds left before automatic deletion

# See all keys with expiration
127.0.0.1:6379> SCAN 0 MATCH "*" | xargs -I {} sh -c 'echo "Key: {}, TTL: $(redis-cli TTL {})"'
```

**Memory benefit:** Data automatically disappears when no longer needed, preventing memory leaks.

### 5. Monitor Key Distribution

**Why monitor?** Find which keys use the most memory so you can optimize them.

```bash
# Analyze key patterns for optimization
redis-cli --bigkeys

# Sample output:
[00.00%] Biggest string  found so far 'cache:large_page' with 1048576 bytes
[50.00%] Biggest list    found so far 'queue:tasks' with 10000 items  
[75.00%] Biggest hash    found so far 'user:profile:1001' with 50 fields
[100.00%] Biggest set     found so far 'tags:popular' with 5000 members
[100.00%] Biggest zset    found so far 'leaderboard:global' with 1000 members

-------- summary -------

Sampled 1000 keys in the keyspace!
Total key length in bytes is 15000 (avg len 15.00)

Biggest string: 'cache:large_page' (1048576 bytes)
Biggest list: 'queue:tasks' (10000 items)  
Biggest hash: 'user:profile:1001' (50 fields)
Biggest set: 'tags:popular' (5000 members)
Biggest zset: 'leaderboard:global' (1000 members)

5 strings with 2097152 total bytes (41.94% of keys, avg size 419430.40)
2 lists with 10500 total items (0.20% of keys, avg size 5250.00)
1 hashs with 50 total fields (0.10% of keys, avg size 50.00)
1 sets with 5000 total members (0.10% of keys, avg size 5000.00)  
1 zsets with 1000 total members (0.10% of keys, avg size 1000.00)
```

**What this tells you:**

- `cache:large_page` is using 1MB - maybe too big for a cache entry
- `queue:tasks` has 10,000 items - might need cleanup
- Most memory (41.94%) is used by string keys
- You have 1000 total keys using 15KB for key names (average 15 bytes per key name)

---

## Monitoring Memory and Persistence

### Memory Monitoring Commands

**Why monitor memory?** To catch problems before Redis runs out of space or slows down.

```redis
# Memory overview - get the big picture
127.0.0.1:6379> INFO memory
# Memory
used_memory:1048576
used_memory_human:1.00M
used_memory_rss:2097152
used_memory_rss_human:2.00M
used_memory_peak:1200000
used_memory_peak_human:1.14M
used_memory_peak_perc:87.38%
mem_fragmentation_ratio:2.00
mem_fragmentation_bytes:1048576
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_allocated:1000000
allocator_active:1500000
allocator_resident:2000000

# Memory usage by sample - deeper analysis
127.0.0.1:6379> MEMORY STATS
 1) "peak.allocated"
 2) (integer) 1200000
 3) "total.allocated"  
 4) (integer) 1048576
 5) "startup.allocated"
 6) (integer) 524288
 7) "replication.backlog"
 8) (integer) 0
 9) "clients.slaves"
10) (integer) 0
11) "clients.normal"
12) (integer) 16384
13) "aof.buffer"
14) (integer) 0
15) "db.0"
16) 1) "overhead.hashtable.main"
    2) (integer) 72
    3) "overhead.hashtable.expires"
    4) (integer) 0

# Doctor report (Redis 4.0+) - get advice from Redis
127.0.0.1:6379> MEMORY DOCTOR
"Hi Sam, I can see you have some peak memory usage issues. You can reduce memory usage by setting a limit with CONFIG SET maxmemory <bytes>."

# Track memory usage over time - quick check
127.0.0.1:6379> INFO memory | grep used_memory_human
used_memory_human:1.00M
```

**Understanding the important numbers:**

- `used_memory_human:1.00M` = Redis is using 1MB of memory for your data
- `used_memory_rss_human:2.00M` = Operating system sees Redis using 2MB (includes fragmentation)
- `mem_fragmentation_ratio:2.00` = Memory is fragmented (2x more OS memory than data)
- `maxmemory:0` = No memory limit set (could use all system memory!)

### Persistence Monitoring

**Why monitor persistence?** To make sure your data is actually being saved and backups are working.

```redis
# Check RDB and AOF status - see if saves are working
127.0.0.1:6379> INFO persistence
# Persistence
loading:0
rdb_changes_since_last_save:15
rdb_bgsave_in_progress:0
rdb_last_save_time:1705751400
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:2
rdb_current_bgsave_time_sec:-1
rdb_saves:5
rdb_last_cow_size:1048576
aof_enabled:1
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:3
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok
aof_last_cow_size:524288
aof_current_size:2048
aof_base_size:1024
aof_pending_rewrite:0
aof_buffer_length:0
aof_rewrite_buffer_length:0
aof_pending_bio_fsync:0
aof_delayed_fsync:0
```

**Key things to watch:**

- `rdb_last_bgsave_status:ok` = Last RDB save worked fine
- `rdb_changes_since_last_save:15` = 15 changes since last save (might need a new snapshot soon)
- `aof_last_write_status:ok` = AOF is writing successfully
- `aof_current_size:2048` vs `aof_base_size:1024` = AOF doubled in size, might need rewrite

### Setting Up Alerts

**What to monitor in production:** These numbers tell you when something is wrong.

```bash
# Memory usage > 80% (getting close to limit)
used_memory / maxmemory > 0.8

# High fragmentation (wasting memory)  
mem_fragmentation_ratio > 1.5

# Failed saves (data not being backed up!)
rdb_last_bgsave_status != "ok"

# AOF growing too fast (disk space problem)
aof_current_size > aof_base_size * 3

# Too many changes without save (potential data loss)
rdb_changes_since_last_save > 10000
```

**How to check these with commands:**
```redis
# Check memory percentage (if you set maxmemory)
127.0.0.1:6379> INFO memory | grep -E "(used_memory:|maxmemory:)"
used_memory:1048576
maxmemory:2097152
# Calculate: 1048576 / 2097152 = 0.5 = 50% memory used

# Check fragmentation
127.0.0.1:6379> INFO memory | grep mem_fragmentation_ratio
mem_fragmentation_ratio:1.25
# 1.25 is good (between 1.0-1.5)

# Check save status
127.0.0.1:6379> INFO persistence | grep rdb_last_bgsave_status
rdb_last_bgsave_status:ok
# "ok" means saves are working
```

---

## Common Memory Issues and Solutions

### Issue 1: Memory Fragmentation

**What is fragmentation?** Think of it like having a messy closet - you have space, but it's all broken up into small pieces that are hard to use.

**Problem:** `mem_fragmentation_ratio > 1.5`

```redis
# Check fragmentation - see how messy your memory is
127.0.0.1:6379> INFO memory | grep fragmentation_ratio
mem_fragmentation_ratio:2.34
# 2.34 means Redis is using 2.34x more OS memory than needed - very fragmented!

# Check what this means in actual bytes
127.0.0.1:6379> INFO memory | grep -E "(used_memory:|used_memory_rss:)"
used_memory:1048576      # 1MB of actual data
used_memory_rss:2455552  # 2.3MB used by OS (wasted: 1.3MB)
```

**Solutions:**
```redis
# Solution 1: Restart Redis (temporary fix - like cleaning your closet)
# This requires downtime but immediately fixes fragmentation

# Solution 2: Use memory defragmentation (Redis 4.0+)
127.0.0.1:6379> CONFIG SET activedefrag yes
OK
127.0.0.1:6379> CONFIG GET activedefrag
1) "activedefrag"
2) "yes"
# Redis will slowly reorganize memory in the background

# Solution 3: Adjust memory allocator settings
127.0.0.1:6379> CONFIG SET jemalloc-bg-thread yes
OK
# Helps the memory allocator work more efficiently
```

### Issue 2: Memory Leaks

**What is a memory leak?** Data that should be deleted but isn't, like leaving old food in your fridge.

**Problem:** Memory keeps growing without new data

```redis
# Find large keys that might be eating memory
redis-cli --bigkeys
# Look for surprisingly large keys in the output

# Check for keys without TTL that should expire
redis-cli --scan --pattern "temp:*" | head -10
temp:user_session_abc123
temp:cache_result_xyz789
temp:download_token_456

# Check if these temporary keys have expiration
127.0.0.1:6379> TTL temp:user_session_abc123  
(integer) -1  # -1 means no expiration set - this is a leak!

127.0.0.1:6379> TTL temp:cache_result_xyz789
(integer) -1  # Another leak!

# Fix by adding expiration to existing keys
127.0.0.1:6379> EXPIRE temp:user_session_abc123 3600  # 1 hour
(integer) 1
127.0.0.1:6379> EXPIRE temp:cache_result_xyz789 1800  # 30 minutes  
(integer) 1
```

**Find memory leaks with specific patterns:**
```bash
# Find all keys by memory usage (if available)
redis-cli --memkeys

# Count keys by pattern to find unexpected growth
redis-cli --scan --pattern "user:*" | wc -l
redis-cli --scan --pattern "cache:*" | wc -l  
redis-cli --scan --pattern "session:*" | wc -l
```


### Issue 3: Persistence Failures

**What happens when saves fail?** Your data isn't backed up - like forgetting to save your homework.

**Problem:** RDB/AOF saves failing

```redis
# Check disk space - most common cause of save failures
127.0.0.1:6379> INFO server | grep used_memory
used_memory:1048576
# Compare this with available disk space using system commands:
# df -h /var/lib/redis (Linux)
# dir C:\redis-data (Windows)

# Check permissions - make sure Redis can write files
127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/var/lib/redis"
# Make sure this directory exists and Redis can write to it

# Test if you can save manually
127.0.0.1:6379> BGSAVE
Background saving started
# Wait a moment, then check:
127.0.0.1:6379> LASTSAVE
(integer) 1705751500  # New timestamp means save worked

# Check for save errors in the log
127.0.0.1:6379> INFO persistence | grep bgsave_status
rdb_last_bgsave_status:ok  # Should be "ok", not "err"

# Fix AOF corruption if needed
redis-check-aof --fix appendonly.aof
# Run this command from terminal when Redis is stopped
```

**Common persistence problems and fixes:**

1. **No disk space** = Delete old files or add more disk space
2. **Permission denied** = Fix file/directory permissions  
3. **Memory too low for fork** = Add more RAM or reduce Redis memory usage
4. **Corrupted AOF file** = Use `redis-check-aof --fix` to repair

---

## Practical Exercises

### Exercise 1: Memory Analysis

```redis
# Setup test data
SET test:string "Hello Redis"
HSET test:hash field1 "value1" field2 "value2"
LPUSH test:list "item1" "item2" "item3"
SADD test:set "member1" "member2" "member3"

# Analyze memory usage
MEMORY USAGE test:string
MEMORY USAGE test:hash  
MEMORY USAGE test:list
MEMORY USAGE test:set

# Compare with INFO memory before and after
```

### Exercise 2: Persistence Setup

```redis
# Configure RDB snapshots
CONFIG SET save "300 10"  # Save if 10 changes in 5 minutes
CONFIG SET dbfilename "exercise.rdb"

# Make some changes
SET counter 1
INCR counter
HSET user:test name "Test User"

# Force save and check
BGSAVE
LASTSAVE

# Configure AOF
CONFIG SET appendonly yes
CONFIG SET appendfsync everysec
```

### Exercise 3: Memory Optimization

```redis
# Create inefficient structure
SET user:1:name "John"
SET user:1:email "john@example.com"
SET user:1:age "30"
SET user:1:city "New York"

# Measure memory
MEMORY USAGE user:1:name
MEMORY USAGE user:1:email
MEMORY USAGE user:1:age
MEMORY USAGE user:1:city

# Convert to efficient hash
DEL user:1:name user:1:email user:1:age user:1:city
HSET user:1 name "John" email "john@example.com" age "30" city "New York"

# Compare memory usage
MEMORY USAGE user:1
```

---

## Memory and Persistence Checklist

For production deployments:

**Memory Management:**

- Set appropriate `maxmemory` limit
- Configure suitable eviction policy
- Monitor memory fragmentation
- Use efficient data structures
- Set TTL on temporary data

**Persistence:**

- Choose appropriate persistence strategy
- Configure automatic saves/rewrites
- Monitor persistence operations
- Test recovery procedures
- Set up backup strategies

**Monitoring:**

- Track memory usage trends
- Alert on high memory usage
- Monitor save/rewrite durations
- Check disk space for persistence files
- Log persistence failures

---

## What's Next?

Now let's learn about configuring Redis for different use cases and environments.

The next section will cover:

- Essential Redis configuration options
- Performance tuning parameters
- Security settings
- Environment-specific configurations
