# Redis CLI Basics

The Redis Command Line Interface (CLI) is your primary tool for interacting with Redis. Let's master it and become efficient Redis users!

## Understanding the Redis CLI

The `redis-cli` is a simple but powerful command-line tool that allows you to:

- Send commands to Redis server
- Execute scripts and batch operations
- Monitor Redis activity
- Import/export data
- Debug Redis issues

## Starting Redis CLI

### Basic Connection

```bash
# Connect to local Redis (default: localhost:6379)
redis-cli

# Connect to specific host and port
redis-cli -h redis.example.com -p 6379

# Connect with authentication
redis-cli -a your-password

# Connect to specific database
redis-cli -n 2
```

**What do these commands mean?**

#### 1. Basic Connection: `redis-cli`
This is the simplest way to connect to Redis:

- **What it does**: Connects to Redis running on your local computer
- **Default assumptions**: 
  - **Host**: `localhost` (127.0.0.1) - your own computer
  - **Port**: `6379` - Redis's standard port
  - **Database**: `0` - the default database
- **When to use**: Perfect for local development and testing

#### 2. Specific Host and Port: `redis-cli -h redis.example.com -p 6380`
When Redis is running on a different computer or port:

- **`-h redis.example.com`**: Connect to a specific server
  - Example: `-h 192.168.1.100` (another computer on your network)
  - Example: `-h my-redis-server.com` (a remote server)
- **`-p 6380`**: Connect to a different port
  - **Why different ports?** Multiple Redis instances, custom configurations, or security
  - **Common ports**: 6379 (default), 6380, 6381, etc.

#### 3. With Password: `redis-cli -a your-password`
When Redis requires authentication:

- **`-a your-password`**: Provide the password to access Redis
- **When needed**: Production servers, shared Redis instances, security-enabled setups
- **Security note**: Password will be visible in command history (we'll show safer methods later)

#### 4. Specific Database: `redis-cli -n 2`
Redis has 16 databases numbered 0-15:

- **`-n 2`**: Connect directly to database number 2
- **Why multiple databases?** Separate different applications or environments
- **Example use cases**:
  - Database 0: Production data
  - Database 1: Development data
  - Database 2: Testing data

### Connection Options Explained

| Option | What It Does | Detailed Explanation | Real Example |
|--------|-------------|---------------------|--------------|
| `-h hostname` | Specify Redis server location | **localhost**: Your computer<br/>**IP address**: Remote server<br/>**Domain name**: Production server | `redis-cli -h 192.168.1.50`<br/>(Connect to Redis on another computer) |
| `-p port` | Specify Redis server port | **6379**: Standard Redis port<br/>**Other ports**: Custom installations<br/>**Why change?** Multiple Redis instances | `redis-cli -p 6380`<br/>(Connect to Redis on port 6380) |
| `-a password` | Provide authentication password | **Security**: Protects Redis from unauthorized access<br/>**Production**: Always use passwords<br/>**Development**: Usually no password needed | `redis-cli -a "my_secret_password"`<br/>(Connect with authentication) |
| `-n database` | Select specific database (0-15) | **0**: Default database<br/>**1-15**: Additional databases<br/>**Use case**: Separate environments | `redis-cli -n 3`<br/>(Connect to database 3) |
| `--raw` | Show output without quotes | **Normal**: `"hello world"`<br/>**Raw**: `hello world`<br/>**Use case**: Cleaner output for scripts | `redis-cli --raw GET message`<br/>(Show value without quotes) |
| `--csv` | Format output as CSV | **Normal**: Multiple lines<br/>**CSV**: Comma-separated values<br/>**Use case**: Export data to spreadsheets | `redis-cli --csv LRANGE mylist 0 -1`<br/>(List as CSV format) |

**Real-World Connection Examples:**

```bash
# Local development (most common)
redis-cli

# Connect to Redis in Docker container
redis-cli -h localhost -p 6379

# Connect to production server with password
redis-cli -h prod-redis.mycompany.com -p 6379 -a "production_password"

# Connect to development database
redis-cli -n 1

# Connect to testing environment
redis-cli -h test-server.com -p 6380 -a "test_pass" -n 2

# Get data in clean format for scripts
redis-cli --raw GET user_name

# Export list data to CSV
redis-cli --csv LRANGE shopping_cart 0 -1
```

---


## Getting Help

### Built-in Help System

```redis
# Get help for any command
127.0.0.1:6379> HELP SET
#output
SET key value [EX seconds|PX milliseconds|KEEPTTL] [NX|XX]
summary: Set the string value of a key
since: 1.0.0
group: string

# List all commands
127.0.0.1:6379> HELP

# Get help by category
127.0.0.1:6379> HELP @string
127.0.0.1:6379> HELP @list
127.0.0.1:6379> HELP @hash
```

### Command Categories

| Category | Description | Example Commands |
|----------|-------------|------------------|
| `@string` | String operations | SET, GET, INCR |
| `@list` | List operations | LPUSH, RPOP, LLEN |
| `@set` | Set operations | SADD, SREM, SISMEMBER |
| `@hash` | Hash operations | HSET, HGET, HDEL |
| `@sorted_set` | Sorted set operations | ZADD, ZRANGE, ZREM |
| `@generic` | Generic key operations | DEL, EXISTS, EXPIRE |

---

## Useful CLI Commands

### Information and Debugging

```redis
# Check connection
127.0.0.1:6379> PING
PONG

# Get server information
127.0.0.1:6379> INFO
127.0.0.1:6379> INFO memory
127.0.0.1:6379> INFO stats

# List all keys (use carefully in production!)
127.0.0.1:6379> KEYS *

# Count total keys
127.0.0.1:6379> DBSIZE
(integer) 42

# Get random key
127.0.0.1:6379> RANDOMKEY
"some_key"
```

### Database Operations

```redis
# Switch to different database (0-15)
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> 

# Clear current database
127.0.0.1:6379[1]> FLUSHDB
OK

# Clear all databases
127.0.0.1:6379[1]> FLUSHALL
OK
```

**What do these database commands do?**

#### 1. SELECT - Switch Between Databases
```redis
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]>
```

**What it does:**

- Redis has 16 separate databases numbered 0-15
- `SELECT 1` switches you to database number 1
- Notice how the prompt changes to `127.0.0.1:6379[1]>` - the `[1]` shows you're now in database 1

**Think of it like this:** Imagine Redis as an apartment building with 16 separate apartments (databases). `SELECT` is like choosing which apartment to enter.

**Common use cases:**

- **Database 0**: Production data (default)
- **Database 1**: Development data  
- **Database 2**: Testing data
- **Database 3**: Cache data

**Practical example:**
```redis
# Start in default database 0
127.0.0.1:6379> SET user:prod "production user"
OK

# Switch to database 1 for development
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> SET user:dev "development user"
OK

# The data is completely separate!
127.0.0.1:6379[1]> GET user:prod
(nil)  # Not found because it's in database 0

# Switch back to see production data
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> GET user:prod
"production user"
```

#### 2. FLUSHDB - Clear Current Database
```redis
127.0.0.1:6379[1]> FLUSHDB
OK
```

**What it does:**

- **Deletes ALL data** in the currently selected database
- Only affects the database you're currently in
- Other databases remain untouched

!!! warning "WARNING"
    This permanently deletes data!

**When to use:**

- **Development**: Clear test data
- **Testing**: Reset between test runs
- **NEVER in production**: Unless you really know what you're doing

**Example scenario:**
```redis
# You're in database 1 with test data
127.0.0.1:6379[1]> KEYS *
1) "test:user:1"
2) "test:user:2" 
3) "test:session:abc"

# Clear only database 1
127.0.0.1:6379[1]> FLUSHDB
OK

# Database 1 is now empty
127.0.0.1:6379[1]> KEYS *
(empty list or set)

# But database 0 still has data
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> KEYS *
1) "user:prod"
# Production data is safe!
```

#### 3. FLUSHALL - Clear ALL Databases
```redis
127.0.0.1:6379[1]> FLUSHALL
OK
```

**What it does:**

- **Deletes ALL data** from ALL 16 databases
- Nuclear option - everything goes away
- No undo button!

!!! warning "EXTREME WARNING"
    This is like formatting your hard drive!

**When to use:**

- **Fresh start**: Completely reset Redis
- **Development only**: When you want to start completely clean
- **ABSOLUTELY NEVER in production**

**Visual comparison:**
```redis
# Before FLUSHALL - multiple databases with data
Database 0: user:prod, session:abc
Database 1: test:user:1, test:user:2  
Database 2: cache:item:123

# After FLUSHALL
Database 0: (empty)
Database 1: (empty)
Database 2: (empty)
# Everything is gone!
```
---

### Best Practices for Database Operations

#### Safe Database Management
```redis
# 1. Always check which database you're in
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> # Notice the [1] in prompt

# 2. Check what data exists before flushing
127.0.0.1:6379[1]> DBSIZE
(integer) 150

# 3. Use FLUSHDB only in development/testing
127.0.0.1:6379[1]> FLUSHDB  # Only if you're sure!
```

#### Development Workflow Example
```redis
# Separate your environments using different databases
# Database 0: Production (don't touch!)
# Database 1: Development 
# Database 2: Testing

# Switch to development database
127.0.0.1:6379> SELECT 1
OK

# Work with development data
127.0.0.1:6379[1]> SET dev:feature "new feature"
OK

# Clear when starting fresh development cycle
127.0.0.1:6379[1]> FLUSHDB
OK
```

#### Production Safety Rules
```redis
# ❌ NEVER do this in production:
FLUSHALL  # Deletes everything!
FLUSHDB   # Deletes current database!

# ✅ Instead, delete specific keys:
DEL specific:key:name
DEL user:session:expired

# ✅ Or use expiration for automatic cleanup:
EXPIRE temp:data 3600  # Expires in 1 hour
```

---

### Key Inspection

```redis
# Check key type
127.0.0.1:6379> TYPE mykey
string

# Check if key exists
127.0.0.1:6379> EXISTS mykey
(integer) 1

# Get time-to-live
127.0.0.1:6379> TTL mykey
(integer) -1

# Get memory usage of a key
127.0.0.1:6379> MEMORY USAGE mykey
(integer) 64
```

**What do these key inspection commands do?**

#### 1. TYPE - What Kind of Data Is This?
```redis
127.0.0.1:6379> TYPE mykey
string
```

**What it tells you:**
Redis can store different types of data. `TYPE` tells you what kind of data structure a key contains.

**Possible return values:**

- **`string`**: Simple text or numbers (most common)
- **`list`**: Ordered collection of items
- **`set`**: Unordered collection of unique items
- **`hash`**: Key-value pairs (like a mini-database)
- **`zset`**: Sorted set with scores
- **`stream`**: Time-series data
- **`none`**: Key doesn't exist

**Why this matters:**
Different data types use different commands!

**Practical example:**
```redis
# Create different types of data
127.0.0.1:6379> SET username "john_doe"
OK
127.0.0.1:6379> LPUSH shopping_list "milk" "bread"
(integer) 2
127.0.0.1:6379> HSET user:1001 name "John" age "25"
(integer) 2

# Check their types
127.0.0.1:6379> TYPE username
string

127.0.0.1:6379> TYPE shopping_list
list

127.0.0.1:6379> TYPE user:1001
hash

# Now you know which commands to use!
# For string: GET username
# For list: LRANGE shopping_list 0 -1
# For hash: HGET user:1001 name
```

#### 2. EXISTS - Is This Key Actually There?
```redis
127.0.0.1:6379> EXISTS mykey
(integer) 1
```

**What it tells you:**

- **`(integer) 1`**: Key exists
- **`(integer) 0`**: Key doesn't exist

**Why this is useful:**

- **Before operations**: Check if data exists before trying to read it
- **Debugging**: "Why can't I find my data?"
- **Conditional logic**: Only do something if key exists

**Practical examples:**
```redis
# Check if user profile exists before trying to get it
127.0.0.1:6379> EXISTS user:1001
(integer) 1  # Exists! Safe to get data

127.0.0.1:6379> GET user:1001
"John's profile data"

# Check for non-existent key
127.0.0.1:6379> EXISTS user:9999
(integer) 0  # Doesn't exist

127.0.0.1:6379> GET user:9999
(nil)  # As expected, nothing there

# Check multiple keys at once
127.0.0.1:6379> EXISTS user:1001 user:1002 user:1003
(integer) 2  # 2 out of 3 keys exist
```

**Real-world scenario:**
```redis
# Before expensive operation, check if cache exists
127.0.0.1:6379> EXISTS cache:expensive_calculation
(integer) 0  # Not cached yet

# Calculate and cache the result
127.0.0.1:6379> SET cache:expensive_calculation "result_data"
OK

# Next time, check again
127.0.0.1:6379> EXISTS cache:expensive_calculation
(integer) 1  # Now it's cached!
```

#### 3. TTL - How Long Until This Disappears?
```redis
127.0.0.1:6379> TTL mykey
(integer) -1
```

**What TTL means:** "Time To Live" - how many seconds until the key expires and gets deleted automatically.

**Return values explained:**

- **Positive number (e.g., `300`)**: Key expires in 300 seconds (5 minutes)
- **`-1`**: Key exists but will never expire
- **`-2`**: Key doesn't exist

**Why this matters:**

- **Cache management**: Know when cached data will expire
- **Session tracking**: See when user sessions expire
- **Debugging**: "Why did my data disappear?"

**Practical examples:**
```redis
# Set data with expiration
127.0.0.1:6379> SETEX session:user123 3600 "session_data"
OK

# Check how much time is left
127.0.0.1:6379> TTL session:user123
(integer) 3543  # 3543 seconds left (about 59 minutes)

# Wait a bit and check again
127.0.0.1:6379> TTL session:user123
(integer) 3521  # Counting down...

# Check permanent data
127.0.0.1:6379> SET permanent_data "never expires"
OK
127.0.0.1:6379> TTL permanent_data
(integer) -1  # Will never expire

# Check non-existent key
127.0.0.1:6379> TTL nonexistent_key
(integer) -2  # Doesn't exist
```

**Monitoring expiration example:**
```redis
# Monitor cache expiration
127.0.0.1:6379> TTL cache:product:123
(integer) 300  # 5 minutes left

# If getting close to expiration, might want to refresh
127.0.0.1:6379> TTL cache:product:123
(integer) 30   # Only 30 seconds left - time to refresh!
```

#### 4. MEMORY USAGE - How Much Space Does This Take?
```redis
127.0.0.1:6379> MEMORY USAGE mykey
(integer) 64
```

**What it tells you:**
The number of bytes this key uses in Redis memory.

**Why this is useful:**

- **Memory optimization**: Find which keys use the most memory
- **Cost analysis**: Understand memory costs of your data
- **Debugging**: "Why is Redis using so much memory?"
- **Capacity planning**: Plan for future growth

**Understanding the numbers:**

- **Small strings**: Usually 50-100 bytes
- **Large text**: Thousands of bytes
- **Lists/Sets**: Depends on number of items
- **Hashes**: Depends on number of fields

**Practical examples:**
```redis
# Check memory usage of different data types

# Small string
127.0.0.1:6379> SET small_text "hello"
OK
127.0.0.1:6379> MEMORY USAGE small_text
(integer) 51  # About 51 bytes

# Larger string
127.0.0.1:6379> SET large_text "This is a much longer string with lots of text content that takes more memory"
OK
127.0.0.1:6379> MEMORY USAGE large_text
(integer) 126  # About 126 bytes

# List with multiple items
127.0.0.1:6379> LPUSH shopping_list "item1" "item2" "item3"
(integer) 3
127.0.0.1:6379> MEMORY USAGE shopping_list
(integer) 145  # More memory for the list structure

# Hash with multiple fields
127.0.0.1:6379> HSET user:profile name "John" email "john@example.com" age "30"
(integer) 3
127.0.0.1:6379> MEMORY USAGE user:profile
(integer) 167  # Even more for the hash structure
```

---

### Key Inspection Workflow

**Complete key analysis example:**
```redis
# 1. Check if key exists
127.0.0.1:6379> EXISTS user:1001
(integer) 1  # Exists!

# 2. What type of data is it?
127.0.0.1:6379> TYPE user:1001
hash  # It's a hash

# 3. When does it expire?
127.0.0.1:6379> TTL user:1001
(integer) -1  # Never expires

# 4. How much memory does it use?
127.0.0.1:6379> MEMORY USAGE user:1001
(integer) 234  # Uses 234 bytes

# 5. Now get the data using the right command for hash type
127.0.0.1:6379> HGETALL user:1001
1) "name"
2) "John"
3) "email" 
4) "john@example.com"
```
---

## CLI Shortcuts and Navigation

### Command History

```bash
# Use arrow keys to navigate command history (all OS)
↑ # Previous command
↓ # Next command

# Search command history
Ctrl + R # Start reverse search (Windows/Linux)
Cmd + R  # Start reverse search (macOS alternative)
```

**Practical example:**
```redis
# You typed this complex command earlier:
127.0.0.1:6379> CONFIG SET maxmemory-policy "allkeys-lru"

# Press ↑ to bring it back instead of retyping
# Modify and press Enter to execute
```

### Line Editing Shortcuts

#### Universal Shortcuts (Work on All Operating Systems)

| Shortcut | Action | Explanation |
|----------|--------|-------------|
| ++ctrl+"A"++ | Move to beginning of line | Jump to start - faster than pressing ← many times |
| ++ctrl+"E"++ | Move to end of line | Jump to end - faster than pressing → many times |
| ++ctrl+"K"++ | Delete from cursor to end | Clear everything after cursor position |
| ++ctrl+"U"++ | Delete entire line | Clear the whole command line |
| ++ctrl+"W"++ | Delete previous word | Remove the word before cursor |
| ++tab++ | Auto-complete commands | Let Redis suggest commands and keys |

#### Windows-Specific Shortcuts

| Shortcut | Action | Explanation |
|----------|--------|-------------|
| ++ctrl+"C"++ | Exit/Cancel | Stop current command or exit Redis CLI |
| ++ctrl+"D"++ | Exit CLI | Alternative way to exit Redis CLI |
| ++ctrl+"L"++ | Clear screen | Clean up the terminal (same as `clear` command) |
| ++alt+"Backspace"++ | Delete previous word | Alternative to Ctrl+W |

#### macOS-Specific Shortcuts

| Shortcut | Action | Explanation |
|----------|--------|-------------|
| ++cmd+"A"++ | Move to beginning of line | macOS alternative to Ctrl+A |
| ++cmd+"E"++ | Move to end of line | macOS alternative to Ctrl+E |
| ++cmd+"K"++ | Delete from cursor to end | macOS alternative to Ctrl+K |
| ++cmd+"U"++ | Delete entire line | macOS alternative to Ctrl+U |
| ++opt+"Backspace"++ | Delete previous word | Delete word backward on Mac |
| ++opt+"Left"++ | Move word backward | Jump backward by whole words |
| ++opt+"Right"++ | Move word forward | Jump forward by whole words |
| ++cmd+"C"++ | Exit/Cancel | Stop current command or exit Redis CLI |
| ++cmd+"D"++ | Exit CLI | Alternative way to exit Redis CLI |
| ++cmd+"L"++ | Clear screen | Clean up the terminal |

#### Linux-Specific Shortcuts

| Shortcut | Action | Explanation |
|----------|--------|-------------|
| ++ctrl+"C"++ | Exit/Cancel | Stop current command or exit Redis CLI |
| ++ctrl+"D"++ | Exit CLI | Alternative way to exit Redis CLI |
| ++ctrl+"L"++ | Clear screen | Clean up the terminal |
| ++alt+"Backspace"++ | Delete previous word | Alternative to Ctrl+W |
| ++alt+"Left"++ | Move word backward | Jump backward by whole words |
| ++alt+"Right"++ | Move word forward | Jump forward by whole words |


### Tab Completion

Redis CLI supports intelligent auto-completion across all operating systems:

```redis
127.0.0.1:6379> SE<Tab>
SELECT  SET     SETBIT  SETEX   SETNX   SETRANGE

127.0.0.1:6379> SET my<Tab>
# If you have keys starting with "my", they'll be suggested
```

### Exiting Redis CLI

#### All Operating Systems
```redis
# Standard exit command
127.0.0.1:6379> EXIT

# Or type quit
127.0.0.1:6379> QUIT
```

#### Keyboard Shortcuts by OS

**Windows/Linux:**
```bash
Ctrl + C  # Force exit (works anywhere)
Ctrl + D  # Graceful exit (at empty prompt)
```

**macOS:**
```bash
Cmd + C   # Force exit (works anywhere)  
Cmd + D   # Graceful exit (at empty prompt)
Ctrl + C  # Also works (Unix-style)
Ctrl + D  # Also works (Unix-style)
```

---

## Advanced CLI Features

### Batch Operations

Execute multiple commands at once:

```bash
# From command line (one-liner)
redis-cli SET key1 "value1" SET key2 "value2" GET key1

# Pipe commands
echo -e "SET batch1 hello\nSET batch2 world\nGET batch1" | redis-cli

# From file
redis-cli < commands.txt
```

**What do batch operations do?**

Instead of running Redis commands one by one, you can run many commands together. This is faster and more efficient!

#### 1. Command Line One-Liner
```bash
redis-cli SET key1 "value1" SET key2 "value2" GET key1
```

**What this does:**

- Runs three commands in sequence: SET, SET, GET
- All commands execute on the Redis server in one connection
- Much faster than running three separate `redis-cli` commands

**Output example:**
```
OK
OK
"value1"
```

#### 2. Pipe Commands (For Multiple Lines)
```bash
echo -e "SET batch1 hello\nSET batch2 world\nGET batch1" | redis-cli
```

**Breaking this down:**

- **`echo -e`**: Prints text with escape sequences
- **`\n`**: New line character (creates separate commands)
- **`|`**: Pipe - sends the output to redis-cli
- **`redis-cli`**: Receives and executes all the commands

**What happens:**
```
Command 1: SET batch1 hello
Command 2: SET batch2 world  
Command 3: GET batch1
```

**Output:**
```
OK
OK
"hello"
```

**Practical example:**
```bash
# Set up user data in one go
echo -e "SET user:1 John\nSET user:2 Jane\nSET user:3 Bob\nGET user:1" | redis-cli
```

#### 3. Execute Commands From File
```bash
redis-cli < commands.txt
```

**What this does:**

- Reads Redis commands from a text file
- Executes all commands in the file
- Perfect for complex setups or backups

**Example `commands.txt` file:**
```
SET user:1001 "John Doe"
SET user:1002 "Jane Smith"  
HSET profile:1001 name "John" age "30"
HSET profile:1002 name "Jane" age "25"
EXPIRE user:1001 3600
EXPIRE user:1002 3600
```

**How to use:**
```bash
# Create the file first (on Windows)
echo SET user:1001 "John Doe" > commands.txt
echo SET user:1002 "Jane Smith" >> commands.txt

# Execute all commands
redis-cli < commands.txt
```
---

### Output Formatting

```bash
# Raw output (no quotes)
redis-cli --raw GET mykey

# CSV format  
redis-cli --csv LRANGE mylist 0 -1

# JSON-like output
redis-cli --json GET user:profile
```

**Why format output differently?**

By default, Redis CLI shows quotes around strings and formats data for human reading. But sometimes you want cleaner output for scripts or data export.

#### 1. Raw Output (--raw)
```bash
redis-cli --raw GET mykey
```

**Normal vs Raw output:**
```bash
# Normal output (with quotes)
redis-cli GET username
"john_doe"

# Raw output (no quotes)
redis-cli --raw GET username  
john_doe
```


#### 2. CSV Format (--csv)
```bash
redis-cli --csv LRANGE mylist 0 -1
```

**What this does:**

- Formats Redis list/set data as comma-separated values
- Perfect for importing into spreadsheets
- Makes data analysis easier

**Example:**
```bash
# Add items to a list
redis-cli LPUSH shopping_list "milk" "bread" "eggs" "cheese"

# Get as normal output
redis-cli LRANGE shopping_list 0 -1
1) "cheese"
2) "eggs" 
3) "bread"
4) "milk"

# Get as CSV
redis-cli --csv LRANGE shopping_list 0 -1
"cheese","eggs","bread","milk"
```


**Save to file example:**
```bash
# Export shopping lists to CSV file
redis-cli --csv LRANGE shopping_list 0 -1 > shopping_list.csv
```

#### 3. JSON Output (--json)
```bash
redis-cli --json GET user:profile
```

**What this does:**

- Formats output as JSON (if the data is JSON)
- Makes it easier to process with JSON tools
- Better for API integrations

**Example:**
```bash
# Store JSON data
redis-cli SET user:1001 '{"name":"John","age":30,"city":"NYC"}'

# Get as normal output
redis-cli GET user:1001
"{\"name\":\"John\",\"age\":30,\"city\":\"NYC\"}"

# Get as formatted JSON
redis-cli --json GET user:1001
{"name":"John","age":30,"city":"NYC"}
```

---

### Pattern Scanning

```bash
# Scan for keys with pattern (safer than KEYS *)
redis-cli --scan --pattern "user:*"

# Count keys matching pattern
redis-cli --scan --pattern "session:*" | wc -l
```

**Why use SCAN instead of KEYS?**

`KEYS *` can be dangerous in production because it blocks Redis while searching through ALL keys. `SCAN` is much safer!

#### 1. Pattern Scanning (--scan --pattern)
```bash
redis-cli --scan --pattern "user:*"
```

**What this does:**

- Searches for keys matching a pattern
- **Safe for production**: Doesn't block Redis
- **Progressive**: Returns results in chunks

**Pattern examples:**
```bash
# Find all user keys
redis-cli --scan --pattern "user:*"
# Finds: user:1001, user:1002, user:profile:john

# Find all session keys  
redis-cli --scan --pattern "session:*"
# Finds: session:abc123, session:def456

# Find all cache keys
redis-cli --scan --pattern "cache:*"
# Finds: cache:product:123, cache:user:456

# Find keys ending with specific pattern
redis-cli --scan --pattern "*:temp"
# Finds: data:temp, user:temp, cache:temp
```

**Comparison with KEYS:**
```bash
# ❌ Dangerous in production - blocks Redis
redis-cli KEYS "user:*"

# ✅ Safe - doesn't block Redis
redis-cli --scan --pattern "user:*"
```

#### 2. Count Matching Keys
```bash
redis-cli --scan --pattern "session:*" | wc -l
```

**Breaking this down:**

- **`redis-cli --scan --pattern "session:*"`**: Find all session keys
- **`|`**: Pipe the results to the next command
- **`wc -l`**: Count the number of lines (each line = one key)

**Examples:**
```bash
# Count how many users you have
redis-cli --scan --pattern "user:*" | wc -l
# Output: 150

# Count active sessions
redis-cli --scan --pattern "session:*" | wc -l  
# Output: 23

# Count cached items
redis-cli --scan --pattern "cache:*" | wc -l
# Output: 891
```

**For Windows users:**
```cmd
# Windows equivalent (using find instead of wc)
redis-cli --scan --pattern "user:*" | find /c /v ""
```

#### 3. Advanced Pattern Operations

**Find and process keys:**
```bash
# Find all user sessions and check their TTL
redis-cli --scan --pattern "session:*" | while read key; do
    echo "$key: $(redis-cli TTL "$key") seconds left"
done
```

**Find and delete expired keys:**
```bash
# Find all temporary keys older than 1 hour
redis-cli --scan --pattern "temp:*" | while read key; do
    ttl=$(redis-cli TTL "$key")
    if [ "$ttl" -eq -2 ]; then
        echo "Deleting expired key: $key"
        redis-cli DEL "$key"
    fi
done
```

**Memory analysis:**
```bash
# Find your biggest keys
redis-cli --scan --pattern "*" | head -10 | while read key; do
    memory=$(redis-cli MEMORY USAGE "$key")
    echo "$key: $memory bytes"
done
```

### Batch Operations Best Practices

#### Safe File Operations
```bash
# 1. Always test commands first
echo "SET test:key 'test_value'" | redis-cli

# 2. Use a specific database for testing
echo "SELECT 1" > test_commands.txt
echo "SET test:data 'safe_testing'" >> test_commands.txt
redis-cli < test_commands.txt

# 3. Create backups before big operations
redis-cli BGSAVE  # Creates backup
```

#### Windows-Specific Examples
```cmd
# Windows batch file example (save as setup.bat)
echo SET config:app_name "MyApp" > redis_setup.txt
echo SET config:version "1.0" >> redis_setup.txt  
echo HSET settings theme "dark" >> redis_setup.txt
echo HSET settings notifications "enabled" >> redis_setup.txt

# Execute the batch
redis-cli < redis_setup.txt
```

#### Error Handling
```bash
# Check if commands succeeded
redis-cli SET test:key "value" && echo "Success!" || echo "Failed!"

# From file with error checking
if redis-cli < important_commands.txt; then
    echo "All commands executed successfully"
else  
    echo "Some commands failed - check Redis logs"
fi
```

---

## Configuration and Monitoring

### Runtime Configuration

```redis
# View configuration
127.0.0.1:6379> CONFIG GET "*max*"
127.0.0.1:6379> CONFIG GET "save"

# Change configuration
127.0.0.1:6379> CONFIG SET maxmemory 100mb
OK

# Save configuration to file
127.0.0.1:6379> CONFIG REWRITE
OK
```

**What do configuration commands do?**

Redis configuration can be viewed and changed without restarting the server. This is super useful for tuning performance and managing settings.

#### 1. CONFIG GET - View Current Settings
```redis
127.0.0.1:6379> CONFIG GET "*max*"
```

**Sample Output:**
```
 1) "maxmemory"
 2) "0"
 3) "maxmemory-policy"
 4) "noeviction"
 5) "maxmemory-samples"
 6) "5"
 7) "maxclients"
 8) "10000"
```

**What this shows:**

- **maxmemory**: `0` means no memory limit (Redis can use all available RAM)
- **maxmemory-policy**: `noeviction` means Redis won't delete data when memory is full
- **maxmemory-samples**: `5` controls how Redis samples keys for eviction
- **maxclients**: `10000` maximum number of client connections allowed

**More CONFIG GET examples:**
```redis
# Get save settings (how often Redis saves to disk)
127.0.0.1:6379> CONFIG GET "save"
1) "save"
2) "3600 1 300 100 60 10000"
```

**Save setting explained:**

- **3600 1**: Save if at least 1 key changed in 3600 seconds (1 hour)
- **300 100**: Save if at least 100 keys changed in 300 seconds (5 minutes)  
- **60 10000**: Save if at least 10000 keys changed in 60 seconds (1 minute)

```redis
# Get database settings
127.0.0.1:6379> CONFIG GET "databases"
1) "databases"
2) "16"

# Get timeout settings
127.0.0.1:6379> CONFIG GET "timeout"
1) "timeout"
2) "0"
```

#### 2. CONFIG SET - Change Settings Live
```redis
127.0.0.1:6379> CONFIG SET maxmemory 100mb
OK
```

**What this does:**

- Sets Redis memory limit to 100 megabytes
- Takes effect immediately (no restart needed)
- Redis will start managing memory according to the policy

**Verification:**
```redis
127.0.0.1:6379> CONFIG GET "maxmemory"
1) "maxmemory"
2) "104857600"
```
*Note: 104857600 bytes = 100MB*

**More CONFIG SET examples:**
```redis
# Set password for security
127.0.0.1:6379> CONFIG SET requirepass "my_secure_password"
OK

# Set timeout for idle connections (300 seconds = 5 minutes)
127.0.0.1:6379> CONFIG SET timeout 300
OK

# Change memory eviction policy
127.0.0.1:6379> CONFIG SET maxmemory-policy "allkeys-lru"
OK
```

**Memory policies explained:**

- **noeviction**: Never delete keys (default) - Redis will error when full
- **allkeys-lru**: Delete least recently used keys
- **allkeys-lfu**: Delete least frequently used keys
- **volatile-lru**: Delete LRU keys that have expiration set
- **volatile-lfu**: Delete LFU keys that have expiration set

#### 3. CONFIG REWRITE - Save Changes Permanently
```redis
127.0.0.1:6379> CONFIG REWRITE
OK
```

**What this does:**

- Writes current configuration to the Redis config file
- Makes temporary changes permanent
- Config survives Redis restarts

**Example workflow:**
```redis
# 1. Test new settings
127.0.0.1:6379> CONFIG SET maxmemory 512mb
OK
127.0.0.1:6379> CONFIG SET maxmemory-policy "allkeys-lru"
OK

# 2. Test your application - make sure it works

# 3. If everything works, save permanently
127.0.0.1:6379> CONFIG REWRITE
OK
```

---

### Real-time Monitoring

```bash
# Monitor all commands (separate terminal)
redis-cli MONITOR

# Statistics mode
redis-cli --stat

# Latency monitoring
redis-cli --latency

# Latency history
redis-cli --latency-history
```

**What do monitoring commands do?**

These commands help you understand what's happening in Redis in real-time - perfect for debugging and performance optimization.

#### 1. MONITOR - See Every Command Live
```bash
redis-cli MONITOR
```

**Sample Output:**
```
OK
1694012345.123456 [0 127.0.0.1:54321] "SET" "user:1001" "John Doe"
1694012346.789012 [0 127.0.0.1:54321] "GET" "user:1001" 
1694012347.456789 [0 127.0.0.1:54322] "INCR" "page_views"
1694012348.234567 [0 127.0.0.1:54321] "EXPIRE" "session:abc123" "3600"
1694012349.876543 [1 127.0.0.1:54323] "LPUSH" "notifications" "New message"
```

**What each line means:**

- **1694012345.123456**: Timestamp (when command was executed)
- **[0 127.0.0.1:54321]**: Database number and client IP:port
- **"SET" "user:1001" "John Doe"**: The actual Redis command and arguments

!!! warning 
    MONITOR can slow down Redis in high-traffic environments!

#### 2. Statistics Mode (--stat)
```bash
redis-cli --stat
```

**Sample Output:**
```
------- data ------ --------------------- load -------------------- - child -
keys       mem      clients blocked requests            connections          
2          1.00M    1       0       158 (+1)            7            
2          1.00M    1       0       159 (+1)            7            
2          1.00M    2       0       162 (+3)            8            
3          1.05M    2       0       165 (+3)            8            
3          1.05M    1       0       167 (+2)            8            
```

**Column explanations:**

- **keys**: Number of keys in current database (2, then 3)
- **mem**: Memory usage (1.00M = 1 megabyte, then 1.05M)
- **clients**: Connected clients (1, then 2, then back to 1)
- **blocked**: Clients waiting for operations (usually 0)
- **requests**: Total commands processed (158, 159, 162, etc.)
- **(+1), (+3)**: Commands processed since last update
- **connections**: Total connections made since Redis started

#### 3. Latency Monitoring (--latency)
```bash
redis-cli --latency
```

**Sample Output:**
```
min: 0, max: 3, avg: 0.12 (427 samples)
min: 0, max: 3, avg: 0.11 (428 samples)  
min: 0, max: 4, avg: 0.13 (429 samples)
min: 0, max: 4, avg: 0.12 (430 samples)
```

**What this shows:**

- **min**: Fastest command response time (milliseconds)
- **max**: Slowest command response time  
- **avg**: Average response time
- **samples**: Number of commands measured

**Good latency indicators:**

- **min: 0**: Very fast responses
- **avg < 1**: Excellent performance (sub-millisecond)
- **max < 10**: Acceptable peak times

**Warning signs:**

- **avg > 5**: Might have performance issues
- **max > 100**: Some operations are very slow
- **Growing averages**: Performance degrading over time

#### 4. Latency History (--latency-history)
```bash
redis-cli --latency-history
```

**Sample Output:**
```
min: 0, max: 2, avg: 0.09 (1321 samples) -- 15.01 seconds range
min: 0, max: 3, avg: 0.11 (1337 samples) -- 15.02 seconds range  
min: 0, max: 1, avg: 0.08 (1355 samples) -- 15.01 seconds range
min: 0, max: 4, avg: 0.12 (1389 samples) -- 15.03 seconds range
```

**What's different from --latency:**

- **Historical tracking**: Shows latency over time periods
- **15.01 seconds range**: Each line covers about 15 seconds
- **Trend analysis**: See if performance gets better or worse over time

**Use cases:**

- **Performance monitoring**: Track Redis performance over time
- **Issue detection**: Spot when performance problems started  
- **Capacity planning**: See if you need more Redis resources

---

## Data Import/Export

### Exporting Data

```bash
# Export all data as Redis commands
redis-cli --rdb dump.rdb

# Export specific keys
redis-cli --scan --pattern "user:*" | xargs redis-cli DUMP

# Export as JSON
redis-cli --json GET user:1001 > user_data.json
```

**What do data export commands do?**

Sometimes you need to save your Redis data for backup, migration, or analysis. Redis CLI provides several ways to export data in different formats.

#### 1. RDB Export (--rdb)
```bash
redis-cli --rdb dump.rdb
```

**What this does:**

- Creates a binary snapshot of ALL Redis data
- Saves it to a file called `dump.rdb`
- This is Redis's native backup format
- Contains everything: all databases, keys, values, expiration times

**Sample process:**
```bash
# Run the export command
redis-cli --rdb backup_2025_09_05.rdb

# You'll see output like:
SYNC sent to master, writing 1024 bytes to 'backup_2025_09_05.rdb'
Transfer finished with success.
```

#### 2. Selective Key Export (DUMP with patterns)
```bash
redis-cli --scan --pattern "user:*" | xargs redis-cli DUMP
```

**Breaking this down:**

- **`--scan --pattern "user:*"`**: Find all keys starting with "user:"
- **`|`**: Pipe the results to the next command
- **`xargs redis-cli DUMP`**: Run DUMP command on each key

**Sample output:**
```bash
# If you have keys: user:1001, user:1002, user:1003
user:1001
"\x00\x08John Doe\t\x00\x8b\x9e\x9b\x8a\x14\x00\x00\x00"

user:1002  
"\x00\x0aJane Smith\t\x00\x7c\x5d\x8f\x2a\x19\x00\x00\x00"

user:1003
"\x00\x07Bob Lee\t\x00\x4e\x8c\x7a\x1b\x11\x00\x00\x00"
```

**What the output means:**

- Each line shows a key and its binary data
- The weird characters are Redis's internal format
- Includes expiration info and data type information

**Practical use case:**
```bash
# Export all user data for analysis
redis-cli --scan --pattern "user:*" | xargs redis-cli DUMP > user_backup.txt

# Export all session data
redis-cli --scan --pattern "session:*" | xargs redis-cli DUMP > sessions.txt

# Export cache data
redis-cli --scan --pattern "cache:*" | xargs redis-cli DUMP > cache_data.txt
```

#### 3. JSON Export (--json)
```bash
redis-cli --json GET user:1001 > user_data.json
```

**What this does:**

- Gets data from a specific key
- Formats it as clean JSON (if the data is JSON)
- Saves to a file for easy reading/processing

**Example workflow:**
```bash
# First, let's store some JSON data
redis-cli SET user:1001 '{"name":"John","age":30,"email":"john@example.com"}'

# Now export it as formatted JSON
redis-cli --json GET user:1001 > user_1001.json
```

**Contents of user_1001.json:**
```json
{"name":"John","age":30,"email":"john@example.com"}
```

**Multiple JSON exports:**
```bash
# Export multiple user profiles
redis-cli --json GET user:1001 > user_1001.json
redis-cli --json GET user:1002 > user_1002.json
redis-cli --json GET user:1003 > user_1003.json

# Or export a list as JSON
redis-cli --json LRANGE shopping_list 0 -1 > shopping_list.json
```

---

### Importing Data

```bash
# Import from RDB file
redis-cli --rdb < dump.rdb

# Import commands from file
redis-cli < backup_commands.txt

# Bulk insert from CSV
redis-cli --csv --bulk-insert data.csv
```

**What do data import commands do?**

When you need to restore data, migrate from another system, or load test data, these import methods help you get data back into Redis.

#### 1. RDB Import (--rdb)
```bash
redis-cli --rdb < dump.rdb
```

!!! note "Important Note"
    This command actually doesn't work as shown! The correct way to restore RDB files is different:

    **Correct RDB restore process:**
    ```bash
    # 1. Stop Redis server
    redis-cli SHUTDOWN

    # 2. Copy your RDB file to Redis data directory
    # (Location varies by installation)
    cp backup_2025_09_05.rdb /var/lib/redis/dump.rdb

    # 3. Start Redis server
    redis-server

    # Redis automatically loads the RDB file on startup
    ```

**Alternative method using Redis commands:**
```bash
# Use DEBUG RELOAD (careful - this is a debug command!)
redis-cli DEBUG RELOAD
```

#### 2. Command File Import
```bash
redis-cli < backup_commands.txt
```

**What this does:**

- Reads Redis commands from a text file
- Executes each command line by line
- Perfect for restoring data exported as commands

**Example backup_commands.txt:**
```title="backup_commands.txt"
SET user:1001 "John Doe"
SET user:1002 "Jane Smith"
HSET profile:1001 name "John" age "30" email "john@example.com"
HSET profile:1002 name "Jane" age "25" email "jane@example.com"
EXPIRE user:1001 3600
EXPIRE user:1002 3600
LPUSH notifications "Welcome John!"
LPUSH notifications "Welcome Jane!"
```

**Import process:**
```bash
# Execute all commands in the file
redis-cli < backup_commands.txt

# Output shows each command result:
OK
OK
(integer) 3
(integer) 3
(integer) 1
(integer) 1
(integer) 1
(integer) 2
```


#### 3. CSV Bulk Insert
```bash
redis-cli --csv --bulk-insert data.csv
```
!!! note
    This specific command syntax may not work in all Redis versions. Here's the correct approach:

    **Creating CSV data:**
    ```csv
    user:1001,"John Doe"
    user:1002,"Jane Smith" 
    user:1003,"Bob Johnson"
    product:123,"Red Shirt"
    product:124,"Blue Pants"
    ```

    **Converting CSV to Redis commands:**
    ```bash
    # Convert CSV to Redis commands (Linux/Mac)
    awk -F',' '{print "SET " $1 " " $2}' data.csv > redis_commands.txt

    # Then import the commands
    redis-cli < redis_commands.txt
    ```

    **Windows equivalent:**
    ```cmd
    # Create a batch file to convert CSV
    for /f "tokens=1,2 delims=," %%a in (data.csv) do echo SET %%a %%b >> redis_commands.txt

    # Then import
    redis-cli < redis_commands.txt
    ```

---

### Import/Export Best Practices

#### Safety First
```bash
# 1. Always test imports on a separate database first
redis-cli SELECT 15  # Use database 15 for testing
redis-cli < test_import.txt

# 2. Verify import worked
redis-cli SELECT 15
redis-cli KEYS "*"
redis-cli DBSIZE

# 3. If good, import to target database
redis-cli SELECT 0
redis-cli < production_import.txt
```

#### Large Data Handling
```bash
# For large datasets, use pipelining
# Instead of importing one command at a time, batch them:

# Split large files
split -l 1000 large_backup.txt batch_

# Import in batches
for file in batch_*; do
    echo "Importing $file..."
    redis-cli < "$file"
    echo "Batch complete"
done
```

#### Error Handling
```bash
# Check for import errors
redis-cli < backup.txt 2> import_errors.log

# Verify data integrity after import
echo "Expected keys: $(wc -l < backup.txt)"
echo "Actual keys: $(redis-cli DBSIZE)"
```

---

## CLI Best Practices

### Security Best Practices

!!! warning "Password Security"
    ```bash
    # ❌ Bad - password visible in history
    redis-cli -a mypassword
    
    # ✅ Good - use environment variable
    export REDISCLI_AUTH=mypassword
    redis-cli
    
    # ✅ Better - use auth after connecting
    redis-cli
    127.0.0.1:6379> AUTH mypassword
    ```

### Performance Tips

```redis
# ❌ Avoid in production - can block server
KEYS *

# ✅ Use SCAN instead
SCAN 0 MATCH pattern COUNT 100

# ❌ Don't use DEBUG commands in production
DEBUG SEGFAULT

# ✅ Use INFO for debugging
INFO memory
```

### Production Guidelines

!!! danger "Production Safety"
    - Never use `FLUSHALL` or `FLUSHDB` in production
    - Use `SCAN` instead of `KEYS` for key discovery
    - Be careful with `MONITOR` - it can impact performance
    - Always test commands in development first

---



## Practice Exercises

### Exercise 1: CLI Navigation
1. Connect to Redis
2. Use tab completion to find all commands starting with "S"
3. Get help for the `SET` command
4. Check your command history with arrow keys

### Exercise 2: Monitoring Practice
1. Open two terminals
2. Start `MONITOR` in terminal 1
3. Execute various commands in terminal 2
4. Observe the real-time output

### Exercise 3: Batch Operations
1. Create a file with 5 Redis commands
2. Execute them using `redis-cli < filename`
3. Verify the results


