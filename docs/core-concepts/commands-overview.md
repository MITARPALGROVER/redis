# Commands Overview

Redis has over 200 commands, but don't worry! Most of them follow simple patterns that are easy to understand. Let's explore Redis commands in a logical, beginner-friendly way.

## Understanding Redis Commands

### What are Redis Commands?

Think of Redis commands like instructions you give to a very smart assistant. You tell Redis what you want to do, and it does it for you.

```redis
# Basic command structure: COMMAND key [arguments]
127.0.0.1:6379> SET username "alice"
OK
# SET = command, username = key, "alice" = value

127.0.0.1:6379> GET username
"alice"
# GET = command, username = key
```

### Command Naming Patterns

Redis commands follow logical patterns that make them easy to remember:

| Pattern | Examples | What They Do |
|---------|----------|--------------|
| **Action** | `GET`, `SET`, `DEL` | Basic operations |
| **Action + Type** | `HGET`, `LGET`, `SGET` | Operations on specific data types |
| **Action + All** | `GETALL`, `DELALL` | Operations on entire structures |
| **Action + Range** | `LRANGE`, `ZRANGE` | Get portions of data |

---

## Command Categories

### 1. Key Operations (Work with Any Data Type)

These commands work with keys regardless of what type of data they store.

```redis
# Check if key exists
127.0.0.1:6379> EXISTS username
(integer) 1
# 1 = exists, 0 = doesn't exist

127.0.0.1:6379> EXISTS nonexistent
(integer) 0

# Get the type of data stored in a key
127.0.0.1:6379> TYPE username
string

127.0.0.1:6379> TYPE user:profile
hash

# Delete keys
127.0.0.1:6379> DEL username
(integer) 1
# 1 = deleted, 0 = key didn't exist

# Delete multiple keys at once
127.0.0.1:6379> DEL key1 key2 key3
(integer) 2
# Returns number of keys actually deleted

# Rename a key
127.0.0.1:6379> SET oldname "value"
OK
127.0.0.1:6379> RENAME oldname newname
OK
127.0.0.1:6379> GET newname
"value"
127.0.0.1:6379> GET oldname
(nil)
# oldname no longer exists

# Copy a key (Redis 6.2+)
127.0.0.1:6379> SET original "data"
OK
127.0.0.1:6379> COPY original backup
(integer) 1
127.0.0.1:6379> GET backup
"data"
# Now both original and backup exist
```

**Key Operations Summary:**

- `EXISTS key` - Check if key exists
- `TYPE key` - Get data type of key
- `DEL key [key ...]` - Delete one or more keys
- `RENAME oldkey newkey` - Rename a key
- `COPY source dest` - Copy a key

### 2. String Operations

Strings are the simplest data type in Redis. They can store text, numbers, or even binary data.

```redis
# Basic string operations
127.0.0.1:6379> SET message "Hello Redis"
OK

127.0.0.1:6379> GET message
"Hello Redis"

# Set multiple strings at once
127.0.0.1:6379> MSET name "Alice" age "25" city "New York"
OK

# Get multiple strings at once
127.0.0.1:6379> MGET name age city
1) "Alice"
2) "25"
3) "New York"

# Set only if key doesn't exist
127.0.0.1:6379> SETNX username "bob"
(integer) 0
# 0 = not set because username already exists

127.0.0.1:6379> SETNX newuser "bob"
(integer) 1
# 1 = successfully set

# Set with expiration
127.0.0.1:6379> SETEX session "3600" "user_data"
OK
# Key expires in 3600 seconds (1 hour)

# Append to existing string
127.0.0.1:6379> SET greeting "Hello"
OK
127.0.0.1:6379> APPEND greeting " World"
(integer) 11
# Returns new length (11 characters)
127.0.0.1:6379> GET greeting
"Hello World"

# Get string length
127.0.0.1:6379> STRLEN greeting
(integer) 11

# Work with numbers
127.0.0.1:6379> SET counter "10"
OK
127.0.0.1:6379> INCR counter
(integer) 11
127.0.0.1:6379> INCRBY counter 5
(integer) 16
127.0.0.1:6379> DECR counter
(integer) 15
127.0.0.1:6379> DECRBY counter 3
(integer) 12
```

**String Operations Summary:**

- `SET key value` - Set string value
- `GET key` - Get string value
- `MSET key1 value1 key2 value2` - Set multiple strings
- `MGET key1 key2` - Get multiple strings
- `SETNX key value` - Set only if key doesn't exist
- `SETEX key seconds value` - Set with expiration
- `APPEND key value` - Append to string
- `STRLEN key` - Get string length
- `INCR/DECR key` - Increment/decrement numbers
- `INCRBY/DECRBY key amount` - Increment/decrement by amount

### 3. List Operations

Lists are ordered collections of strings. Think of them like a line of people - you can add to the front, back, or middle.

```redis
# Add items to list
127.0.0.1:6379> LPUSH fruits "apple"
(integer) 1
# LPUSH = add to Left (front) of list

127.0.0.1:6379> LPUSH fruits "banana" "cherry"
(integer) 3
# Multiple items added to front

127.0.0.1:6379> RPUSH fruits "date"
(integer) 4
# RPUSH = add to Right (end) of list

# View the list
127.0.0.1:6379> LRANGE fruits 0 -1
1) "cherry"
2) "banana"
3) "apple"
4) "date"
# 0 -1 means from start to end

# Get specific positions
127.0.0.1:6379> LRANGE fruits 0 1
1) "cherry"
2) "banana"
# First two items

127.0.0.1:6379> LRANGE fruits -2 -1
1) "apple"
2) "date"
# Last two items

# Remove items from list
127.0.0.1:6379> LPOP fruits
"cherry"
# Remove and return first item

127.0.0.1:6379> RPOP fruits
"date"
# Remove and return last item

127.0.0.1:6379> LRANGE fruits 0 -1
1) "banana"
2) "apple"

# Get list length
127.0.0.1:6379> LLEN fruits
(integer) 2

# Get item at specific position
127.0.0.1:6379> LINDEX fruits 0
"banana"
# First item (index 0)

127.0.0.1:6379> LINDEX fruits -1
"apple"
# Last item (index -1)

# Set item at specific position
127.0.0.1:6379> LSET fruits 0 "blueberry"
OK
127.0.0.1:6379> LRANGE fruits 0 -1
1) "blueberry"
2) "apple"

# Remove specific values
127.0.0.1:6379> LPUSH numbers 1 2 1 3 1
(integer) 5
127.0.0.1:6379> LREM numbers 2 1
(integer) 2
# Remove first 2 occurrences of value "1"
127.0.0.1:6379> LRANGE numbers 0 -1
1) "3"
2) "1"
3) "2"
```

**List Operations Summary:**

- `LPUSH key value [value ...]` - Add to front of list
- `RPUSH key value [value ...]` - Add to end of list
- `LPOP key` - Remove and return first item
- `RPOP key` - Remove and return last item
- `LRANGE key start stop` - Get range of items
- `LLEN key` - Get list length
- `LINDEX key index` - Get item at position
- `LSET key index value` - Set item at position
- `LREM key count value` - Remove occurrences of value

### 4. Set Operations

Sets are collections of unique strings. Like a bag of unique items - no duplicates allowed.

```redis
# Add items to set
127.0.0.1:6379> SADD colors "red"
(integer) 1
# 1 = item added

127.0.0.1:6379> SADD colors "blue" "green" "red"
(integer) 2
# Only blue and green added, red already exists

# View all items in set
127.0.0.1:6379> SMEMBERS colors
1) "blue"
2) "green"
3) "red"
# Order may vary (sets are unordered)

# Check if item exists in set
127.0.0.1:6379> SISMEMBER colors "red"
(integer) 1
# 1 = exists

127.0.0.1:6379> SISMEMBER colors "yellow"
(integer) 0
# 0 = doesn't exist

# Get set size
127.0.0.1:6379> SCARD colors
(integer) 3
# SCARD = Set CARDinality (count)

# Remove items from set
127.0.0.1:6379> SREM colors "blue"
(integer) 1
# 1 = item removed

127.0.0.1:6379> SMEMBERS colors
1) "green"
2) "red"

# Get random item(s)
127.0.0.1:6379> SRANDMEMBER colors
"red"
# Random item (doesn't remove it)

127.0.0.1:6379> SRANDMEMBER colors 2
1) "green"
2) "red"
# Multiple random items

# Remove and return random item
127.0.0.1:6379> SPOP colors
"green"
127.0.0.1:6379> SMEMBERS colors
1) "red"

# Set operations (combining sets)
127.0.0.1:6379> SADD fruits "apple" "banana"
(integer) 2
127.0.0.1:6379> SADD vegetables "carrot" "broccoli"
(integer) 2

# Union (combine sets)
127.0.0.1:6379> SUNION fruits vegetables
1) "apple"
2) "banana"
3) "carrot"
4) "broccoli"

# Find common items (intersection)
127.0.0.1:6379> SADD healthy_snacks "apple" "carrot"
(integer) 2
127.0.0.1:6379> SINTER fruits healthy_snacks
1) "apple"
# Only "apple" exists in both sets

# Find differences
127.0.0.1:6379> SDIFF fruits healthy_snacks
1) "banana"
# Items in fruits but not in healthy_snacks
```

**Set Operations Summary:**

- `SADD key member [member ...]` - Add items to set
- `SMEMBERS key` - Get all set members
- `SISMEMBER key member` - Check if item exists
- `SCARD key` - Get set size
- `SREM key member [member ...]` - Remove items from set
- `SRANDMEMBER key [count]` - Get random member(s)
- `SPOP key` - Remove and return random member
- `SUNION key1 key2` - Combine sets
- `SINTER key1 key2` - Find common items
- `SDIFF key1 key2` - Find differences

### 5. Hash Operations

Hashes are like objects or dictionaries - collections of field-value pairs. Perfect for storing structured data.

```redis
# Set hash fields
127.0.0.1:6379> HSET user:1001 name "Alice"
(integer) 1
# 1 = new field created

127.0.0.1:6379> HSET user:1001 age "25" city "New York"
(integer) 2
# 2 = two new fields created

# Get hash field
127.0.0.1:6379> HGET user:1001 name
"Alice"

# Get multiple fields
127.0.0.1:6379> HMGET user:1001 name age
1) "Alice"
2) "25"

# Get all fields and values
127.0.0.1:6379> HGETALL user:1001
1) "name"
2) "Alice"
3) "age"
4) "25"
5) "city"
6) "New York"
# Format: field1, value1, field2, value2, ...

# Get only field names
127.0.0.1:6379> HKEYS user:1001
1) "name"
2) "age"
3) "city"

# Get only values
127.0.0.1:6379> HVALS user:1001
1) "Alice"
2) "25"
3) "New York"

# Check if field exists
127.0.0.1:6379> HEXISTS user:1001 name
(integer) 1
# 1 = field exists

127.0.0.1:6379> HEXISTS user:1001 email
(integer) 0
# 0 = field doesn't exist

# Get number of fields
127.0.0.1:6379> HLEN user:1001
(integer) 3

# Delete fields
127.0.0.1:6379> HDEL user:1001 city
(integer) 1
# 1 = field deleted

127.0.0.1:6379> HGETALL user:1001
1) "name"
2) "Alice"
3) "age"
4) "25"

# Set field only if it doesn't exist
127.0.0.1:6379> HSETNX user:1001 email "alice@example.com"
(integer) 1
# 1 = field set

127.0.0.1:6379> HSETNX user:1001 email "different@example.com"
(integer) 0
# 0 = field not set because it already exists

# Increment numeric fields
127.0.0.1:6379> HSET user:1001 score "100"
(integer) 1
127.0.0.1:6379> HINCRBY user:1001 score 50
(integer) 150
127.0.0.1:6379> HGET user:1001 score
"150"

# Increment by decimal amount
127.0.0.1:6379> HSET user:1001 rating "4.5"
(integer) 1
127.0.0.1:6379> HINCRBYFLOAT user:1001 rating 0.3
"4.8"
```

**Hash Operations Summary:**

- `HSET key field value [field value ...]` - Set hash fields
- `HGET key field` - Get field value
- `HMGET key field1 field2` - Get multiple fields
- `HGETALL key` - Get all fields and values
- `HKEYS key` - Get all field names
- `HVALS key` - Get all values
- `HEXISTS key field` - Check if field exists
- `HLEN key` - Get number of fields
- `HDEL key field [field ...]` - Delete fields
- `HSETNX key field value` - Set field if it doesn't exist
- `HINCRBY key field increment` - Increment numeric field
- `HINCRBYFLOAT key field increment` - Increment by decimal

### 6. Sorted Set Operations

Sorted sets are like sets but with scores. Items are automatically ordered by their scores. Perfect for rankings and leaderboards.

```redis
# Add items with scores
127.0.0.1:6379> ZADD leaderboard 100 "alice"
(integer) 1
# score=100, member="alice"

127.0.0.1:6379> ZADD leaderboard 95 "bob" 110 "charlie"
(integer) 2
# Multiple items added

# View sorted set (lowest to highest score)
127.0.0.1:6379> ZRANGE leaderboard 0 -1
1) "bob"
2) "alice"  
3) "charlie"
# Sorted by score: 95, 100, 110

# View with scores
127.0.0.1:6379> ZRANGE leaderboard 0 -1 WITHSCORES
1) "bob"
2) "95"
3) "alice"
4) "100"
5) "charlie"
6) "110"

# View in reverse order (highest to lowest)
127.0.0.1:6379> ZREVRANGE leaderboard 0 -1 WITHSCORES
1) "charlie"
2) "110"
3) "alice"
4) "100"
5) "bob"
6) "95"

# Get score of specific member
127.0.0.1:6379> ZSCORE leaderboard "alice"
"100"

# Get rank (position) of member
127.0.0.1:6379> ZRANK leaderboard "alice"
(integer) 1
# Position 1 (0-based, from lowest score)

127.0.0.1:6379> ZREVRANK leaderboard "alice"
(integer) 1
# Position 1 (0-based, from highest score)

# Get number of members
127.0.0.1:6379> ZCARD leaderboard
(integer) 3

# Count members in score range
127.0.0.1:6379> ZCOUNT leaderboard 95 105
(integer) 2
# Members with scores between 95 and 105

# Get members by score range
127.0.0.1:6379> ZRANGEBYSCORE leaderboard 95 105
1) "bob"
2) "alice"

# Get members by score range with scores
127.0.0.1:6379> ZRANGEBYSCORE leaderboard 95 105 WITHSCORES
1) "bob"
2) "95"
3) "alice"
4) "100"

# Remove members
127.0.0.1:6379> ZREM leaderboard "bob"
(integer) 1
# 1 = member removed

127.0.0.1:6379> ZRANGE leaderboard 0 -1
1) "alice"
2) "charlie"

# Increment member score
127.0.0.1:6379> ZINCRBY leaderboard 25 "alice"
"125"
# alice's score increased from 100 to 125

127.0.0.1:6379> ZRANGE leaderboard 0 -1 WITHSCORES
1) "charlie"
2) "110"
3) "alice"
4) "125"
# alice moved to top position

# Remove by rank
127.0.0.1:6379> ZREMRANGEBYRANK leaderboard 0 0
(integer) 1
# Remove member at position 0 (charlie)

# Remove by score range
127.0.0.1:6379> ZADD numbers 1 "one" 2 "two" 3 "three" 4 "four"
(integer) 4
127.0.0.1:6379> ZREMRANGEBYSCORE numbers 2 3
(integer) 2
# Remove members with scores 2-3 ("two" and "three")
127.0.0.1:6379> ZRANGE numbers 0 -1
1) "one"
2) "four"
```

**Sorted Set Operations Summary:**

- `ZADD key score member [score member ...]` - Add members with scores
- `ZRANGE key start stop [WITHSCORES]` - Get members by rank (low to high)
- `ZREVRANGE key start stop [WITHSCORES]` - Get members by rank (high to low)
- `ZSCORE key member` - Get member's score
- `ZRANK key member` - Get member's rank (low to high)
- `ZREVRANK key member` - Get member's rank (high to low)
- `ZCARD key` - Get number of members
- `ZCOUNT key min max` - Count members in score range
- `ZRANGEBYSCORE key min max [WITHSCORES]` - Get members by score range
- `ZREM key member [member ...]` - Remove members
- `ZINCRBY key increment member` - Increment member's score
- `ZREMRANGEBYRANK key start stop` - Remove by rank range
- `ZREMRANGEBYSCORE key min max` - Remove by score range

---

## Expiration and TTL Commands

These commands work with any data type to set automatic expiration.

```redis
# Set expiration on existing key
127.0.0.1:6379> SET session_data "user123"
OK
127.0.0.1:6379> EXPIRE session_data 3600
(integer) 1
# Expires in 3600 seconds (1 hour)

# Set expiration in milliseconds
127.0.0.1:6379> PEXPIRE session_data 3600000
(integer) 1
# Expires in 3600000 milliseconds (1 hour)

# Set expiration at specific timestamp
127.0.0.1:6379> EXPIREAT session_data 1756647600
(integer) 1
# Expires at Unix timestamp 1756647600

# Check time-to-live
127.0.0.1:6379> TTL session_data
(integer) 3456
# 3456 seconds remaining

# Check time-to-live in milliseconds
127.0.0.1:6379> PTTL session_data
(integer) 3456789
# 3456789 milliseconds remaining

# Remove expiration (make permanent)
127.0.0.1:6379> PERSIST session_data
(integer) 1
127.0.0.1:6379> TTL session_data
(integer) -1
# -1 means no expiration set
```

**TTL Return Values:**

- **Positive number** = Seconds until expiration
- **-1** = Key exists but no expiration set
- **-2** = Key does not exist

---

## Database and Server Commands

### Database Operations

Redis has 16 databases (0-15) by default. You can switch between them.

```redis
# Check current database
127.0.0.1:6379> SELECT 0
OK
# Now using database 0 (default)

# Switch to different database
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> 
# Notice [1] in prompt - now in database 1

# Set data in database 1
127.0.0.1:6379[1]> SET test "database1"
OK

# Switch back to database 0
127.0.0.1:6379[1]> SELECT 0
OK
127.0.0.1:6379> GET test
(nil)
# Data doesn't exist in database 0

# Move key between databases
127.0.0.1:6379> SET mykey "value"
OK
127.0.0.1:6379> MOVE mykey 2
(integer) 1
# Move mykey to database 2

127.0.0.1:6379> GET mykey
(nil)
# No longer in database 0

127.0.0.1:6379> SELECT 2
OK
127.0.0.1:6379[2]> GET mykey
"value"
# Now in database 2

# Clear current database
127.0.0.1:6379[2]> FLUSHDB
OK
# All keys in current database deleted

# Clear all databases
127.0.0.1:6379> FLUSHALL
OK
# All keys in all databases deleted
```

### Server Information

```redis
# Test connection
127.0.0.1:6379> PING
PONG

# Get server information
127.0.0.1:6379> INFO
# Returns lots of information about Redis server

# Get specific info section
127.0.0.1:6379> INFO memory
# Memory
used_memory:1048576
used_memory_human:1.00M
...

# Get current time
127.0.0.1:6379> TIME
1) "1705751400"
2) "123456"
# Unix timestamp and microseconds

# Get number of keys in current database
127.0.0.1:6379> DBSIZE
(integer) 42

# Save data to disk
127.0.0.1:6379> SAVE
OK
# Blocks Redis until save completes

127.0.0.1:6379> BGSAVE
Background saving started
# Saves in background

# Get last save time
127.0.0.1:6379> LASTSAVE
(integer) 1705751400
```

---

## Command Patterns and Best Practices

### 1. Atomic Operations

Many Redis commands are atomic, meaning they complete entirely or not at all.

```redis
# Atomic increment
127.0.0.1:6379> SET counter 10
OK
127.0.0.1:6379> INCR counter
(integer) 11
# Even under high load, this is safe

# Atomic list operations
127.0.0.1:6379> LPUSH queue "item1"
(integer) 1
127.0.0.1:6379> RPOP queue
"item1"
# Perfect for producer-consumer patterns
```

### 2. Bulk Operations

Use bulk operations when possible for better performance.

```redis
# Instead of multiple SET commands
127.0.0.1:6379> SET key1 "value1"
127.0.0.1:6379> SET key2 "value2"
127.0.0.1:6379> SET key3 "value3"

# Use MSET for better performance
127.0.0.1:6379> MSET key1 "value1" key2 "value2" key3 "value3"
OK

# Similarly for GET operations
127.0.0.1:6379> MGET key1 key2 key3
1) "value1"
2) "value2"
3) "value3"
```

### 3. Safe Key Checking

Always check if operations succeeded when needed.

```redis
# Check if SET with NX worked
127.0.0.1:6379> SETNX lock:user123 "processing"
(integer) 1
# 1 = lock acquired successfully

127.0.0.1:6379> SETNX lock:user123 "processing"
(integer) 0
# 0 = lock already exists, operation failed

# Check if key was actually deleted
127.0.0.1:6379> DEL nonexistent_key
(integer) 0
# 0 = key didn't exist

127.0.0.1:6379> DEL existing_key
(integer) 1
# 1 = key was deleted
```

### 4. Working with Large Data

Be careful with operations that return all data.

```redis
# Dangerous on large sets
127.0.0.1:6379> SMEMBERS large_set # SMEMBERS: To get all data in sets
# Could return millions of items!

# Better approach - check size first
127.0.0.1:6379> SCARD large_set # SCARD: to check length of set
(integer) 1000000
# Now you know it's large

# Use SCAN for large keyspaces
127.0.0.1:6379> SCAN 0 MATCH user:* COUNT 100
# Safe iteration through keys
```

---

## Command Practice Exercises

### Exercise 1: User Profile Management

```redis
# Create user profile using hash
HSET user:alice name "Alice Cooper" age "28" email "alice@example.com"

# Update specific fields
HSET user:alice city "San Francisco" last_login "2024-01-15"

# Get user info
HGETALL user:alice
HGET user:alice name

# Check if field exists
HEXISTS user:alice phone

# Increment login count
HINCRBY user:alice login_count 1
```

### Exercise 2: Shopping Cart

```redis
# Add items to cart (use list for order preservation)
LPUSH cart:user123 "item1" "item2" "item3"

# View cart
LRANGE cart:user123 0 -1

# Remove item from cart
LREM cart:user123 1 "item2"

# Get cart size
LLEN cart:user123

# Clear cart
DEL cart:user123
```

### Exercise 3: Leaderboard System

```redis
# Add players with scores
ZADD game_scores 1500 "player1" 1200 "player2" 1800 "player3"

# Get top 3 players
ZREVRANGE game_scores 0 2 WITHSCORES

# Update player score
ZINCRBY game_scores 100 "player2"

# Get player rank
ZREVRANK game_scores "player2"

# Get players in score range
ZRANGEBYSCORE game_scores 1400 1600 WITHSCORES
```

### Exercise 4: Session Management

```redis
# Create session with expiration
SETEX session:abc123 3600 "user_data"

# Check session validity
EXISTS session:abc123

# Extend session
EXPIRE session:abc123 7200

# Check remaining time
TTL session:abc123

# Remove session
DEL session:abc123
```

---

## Quick Command Reference

### Most Common Commands

| Category | Commands | Usage |
|----------|----------|-------|
| **Strings** | `SET`, `GET`, `MSET`, `MGET` | Basic key-value operations |
| **Lists** | `LPUSH`, `RPUSH`, `LPOP`, `RPOP`, `LRANGE` | Ordered collections |
| **Sets** | `SADD`, `SMEMBERS`, `SREM` | Unique collections |
| **Hashes** | `HSET`, `HGET`, `HGETALL` | Structured objects |
| **Sorted Sets** | `ZADD`, `ZRANGE`, `ZREVRANGE` | Ranked collections |
| **Keys** | `EXISTS`, `DEL`, `TYPE`, `EXPIRE`, `TTL` | Key management |
| **Server** | `PING`, `INFO`, `DBSIZE` | Server operations |

### Command Syntax Patterns

```redis
# Basic pattern: COMMAND key [arguments]
GET mykey
SET mykey "value"
DEL mykey

# Type-specific pattern: [TYPE]COMMAND key [arguments]
HGET myhash field
LLEN mylist
SCARD myset

# Bulk pattern: M-prefix for multiple operations
MGET key1 key2 key3
MSET key1 "val1" key2 "val2"

# Range pattern: range in name
LRANGE mylist 0 -1
ZRANGE myzset 0 -1
```

---

## What's Next?

Now that you understand Redis commands overview, let's dive deep into each data type with detailed operations and real-world examples.

The next sections will cover:

- String operations in detail
- List operations and use cases
- Set operations and applications
- Hash operations for structured data
- Sorted set operations for rankings
