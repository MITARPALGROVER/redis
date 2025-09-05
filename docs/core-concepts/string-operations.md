# String Operations

Strings are the most basic and commonly used data type in Redis. Think of them as simple text notes or numbers that you can store and retrieve quickly.

## What Are Redis Strings?

Redis strings are like sticky notes - you can write anything on them:

- Text: `"Hello World"`
- Numbers: `"42"`
- JSON: `'{"name": "Alice"}'`
- Even images (as binary data)

The maximum size is 512MB, but usually you'll store much smaller values.

---

## Basic String Operations

### Setting and Getting Values

```redis
# Store a simple text value
127.0.0.1:6379> SET username "alice"
OK

# Get the value back
127.0.0.1:6379> GET username
"alice"

# Store a number (Redis treats it as text)
127.0.0.1:6379> SET age "25"
OK

127.0.0.1:6379> GET age
"25"

# Store JSON data
127.0.0.1:6379> SET user:profile '{"name":"Alice","age":25}'
OK

127.0.0.1:6379> GET user:profile
"{\"name\":\"Alice\",\"age\":25}"
```

### Working with Multiple Strings at Once

```redis
# Set multiple values in one command (faster than multiple SETs)
127.0.0.1:6379> MSET name "Bob" age "30" city "London"
OK

# Get multiple values in one command
127.0.0.1:6379> MGET name age city
1) "Bob"
2) "30"
3) "London"

# If a key doesn't exist, you get nil
127.0.0.1:6379> MGET name age country
1) "Bob"
2) "30"
3) (nil)
```

**Why use MSET/MGET?**

- Much faster than multiple separate commands
- Reduces network round trips
- Perfect for related data

---

## Conditional Setting

### Set Only If Key Doesn't Exist

```redis
# Try to set a username
127.0.0.1:6379> SETNX username "charlie"
(integer) 1
# 1 = success, key was set

# Try again with same key
127.0.0.1:6379> SETNX username "david"
(integer) 0
# 0 = failed, key already exists

127.0.0.1:6379> GET username
"charlie"
# Original value unchanged
```

**Real-world use case**: Creating unique user sessions or preventing duplicate registrations.

### Set Only If Key Exists

```redis
# Set a value only if key already exists
127.0.0.1:6379> SET status "online" XX
OK
# Only works if 'status' already exists

127.0.0.1:6379> SET newkey "value" XX
(nil)
# Failed because 'newkey' doesn't exist

# Set a value only if key doesn't exist
127.0.0.1:6379> SET newkey "value" NX
OK
# Only works if 'newkey' doesn't exist
```

---

## String Expiration

### Set with Automatic Expiration

```redis
# Set value that expires in 60 seconds
127.0.0.1:6379> SETEX session:abc123 60 "user_data"
OK

# Check how much time is left
127.0.0.1:6379> TTL session:abc123
(integer) 57
# 57 seconds remaining

# Set value that expires in 5000 milliseconds (5 seconds)
127.0.0.1:6379> PSETEX temp:token 5000 "quick_access"
OK

127.0.0.1:6379> PTTL temp:token
(integer) 4823
# 4823 milliseconds remaining

# Wait 5 seconds and check again
127.0.0.1:6379> GET temp:token
(nil)
# Key has expired and been deleted
```

**Common expiration times**:

- Session tokens: 1-24 hours
- Cache data: 5-60 minutes  
- Temporary codes: 30-300 seconds

---

## String Manipulation

### Adding Text to Existing Strings

```redis
# Start with a greeting
127.0.0.1:6379> SET message "Hello"
OK

# Add more text to the end
127.0.0.1:6379> APPEND message " World"
(integer) 11
# Returns the new length (11 characters)

127.0.0.1:6379> GET message
"Hello World"

# Keep adding text
127.0.0.1:6379> APPEND message "!"
(integer) 12

127.0.0.1:6379> GET message
"Hello World!"

# Append to non-existent key creates it
127.0.0.1:6379> APPEND newmessage "Starting fresh"
(integer) 14

127.0.0.1:6379> GET newmessage
"Starting fresh"
```

### Getting String Information

```redis
# Check string length
127.0.0.1:6379> STRLEN message
(integer) 12

# Get part of a string
127.0.0.1:6379> GETRANGE message 0 4
"Hello"
# Characters from position 0 to 4

127.0.0.1:6379> GETRANGE message 6 10
"World"
# Characters from position 6 to 10

127.0.0.1:6379> GETRANGE message -1 -1
"!"
# Last character (-1 means from end)

127.0.0.1:6379> GETRANGE message -6 -1
"World!"
# Last 6 characters
```

### Replacing Parts of Strings

```redis
# Replace part of a string
127.0.0.1:6379> SET text "Hello Redis"
OK

# Replace starting at position 6
127.0.0.1:6379> SETRANGE text 6 "World"
(integer) 11
# Returns new length

127.0.0.1:6379> GET text
"Hello World"

# Replace with longer text
127.0.0.1:6379> SETRANGE text 6 "Beautiful World"
(integer) 21

127.0.0.1:6379> GET text
"Hello Beautiful World"

# Replace at the end (extends string)
127.0.0.1:6379> SETRANGE text 21 " of Redis"
(integer) 30

127.0.0.1:6379> GET text
"Hello Beautiful World of Redis"
```

---

## Working with Numbers

Redis can treat strings as numbers and perform math operations on them.

### Basic Number Operations

```redis
# Set a number
127.0.0.1:6379> SET counter "10"
OK

# Increase by 1
127.0.0.1:6379> INCR counter
(integer) 11

127.0.0.1:6379> GET counter
"11"

# Decrease by 1
127.0.0.1:6379> DECR counter
(integer) 10

127.0.0.1:6379> GET counter
"10"

# Increase by specific amount
127.0.0.1:6379> INCRBY counter 5
(integer) 15

# Decrease by specific amount
127.0.0.1:6379> DECRBY counter 3
(integer) 12

# Work with decimal numbers
127.0.0.1:6379> SET price "19.99"
OK

127.0.0.1:6379> INCRBYFLOAT price 5.50
"25.49"

127.0.0.1:6379> INCRBYFLOAT price -2.99
"22.5"
```

### Number Operations on Non-Existent Keys

```redis
# Increment a key that doesn't exist (starts at 0)
127.0.0.1:6379> INCR page_views
(integer) 1

127.0.0.1:6379> INCR page_views
(integer) 2

127.0.0.1:6379> INCRBY downloads 100
(integer) 100
# Started at 0, now 100

# This is perfect for counters and statistics
```

### Error Handling with Numbers

```redis
# Try to increment non-numeric value
127.0.0.1:6379> SET name "alice"
OK

127.0.0.1:6379> INCR name
(error) ERR value is not an integer or out of range

# Redis is smart about what looks like a number
127.0.0.1:6379> SET score "  42  "
OK
# Extra spaces

127.0.0.1:6379> INCR score
(integer) 43
# Works fine, ignores spaces
```

---

## Advanced String Operations

### Atomic Get and Set

```redis
# Get old value and set new value in one operation
127.0.0.1:6379> SET status "offline"
OK

127.0.0.1:6379> GETSET status "online"
"offline"
# Returns the old value ("offline") and sets new value ("online")

127.0.0.1:6379> GET status
"online"

# Useful for toggling states or tracking changes
127.0.0.1:6379> GETSET last_action "login"
"online"
# Got previous status, set new action
```

### Bit Operations (Advanced)

```redis
# Set individual bits (for very advanced use cases)
127.0.0.1:6379> SETBIT flags 0 1
(integer) 0
# Set bit at position 0 to 1

127.0.0.1:6379> SETBIT flags 3 1
(integer) 0
# Set bit at position 3 to 1

127.0.0.1:6379> GETBIT flags 0
(integer) 1

127.0.0.1:6379> GETBIT flags 1
(integer) 0

# Count how many bits are set to 1
127.0.0.1:6379> BITCOUNT flags
(integer) 2
```

**When to use bit operations**: User permissions, feature flags, daily login tracking.

---

## Real-World String Use Cases

### 1. Session Management

```redis
# Create user session
127.0.0.1:6379> SETEX session:user123 3600 '{"user_id":123,"role":"admin"}'
OK
# Expires in 1 hour

# Check if session exists
127.0.0.1:6379> EXISTS session:user123
(integer) 1

# Extend session
127.0.0.1:6379> EXPIRE session:user123 7200
(integer) 1
# Now expires in 2 hours

# Get session data
127.0.0.1:6379> GET session:user123
"{\"user_id\":123,\"role\":\"admin\"}"
```

### 2. Caching API Responses

```redis
# Cache an API response for 5 minutes
127.0.0.1:6379> SETEX cache:weather:london 300 '{"temp":22,"humidity":65}'
OK

# Check cache before making API call
127.0.0.1:6379> GET cache:weather:london
"{\"temp\":22,\"humidity\":65}"
# Cache hit! No need to call API

# After 5 minutes
127.0.0.1:6379> GET cache:weather:london
(nil)
# Cache miss, need to fetch fresh data
```

### 3. Counters and Statistics

```redis
# Track website statistics
127.0.0.1:6379> INCR stats:page_views:today
(integer) 1

127.0.0.1:6379> INCR stats:downloads:total
(integer) 1

127.0.0.1:6379> INCRBY stats:bytes_served 1024
(integer) 1024

# Get current stats
127.0.0.1:6379> MGET stats:page_views:today stats:downloads:total stats:bytes_served
1) "1"
2) "1"
3) "1024"
```

### 4. Configuration Storage

```redis
# Store application settings
127.0.0.1:6379> MSET config:max_users "1000" config:debug_mode "true" config:api_key "abc123"
OK

# Read configuration
127.0.0.1:6379> MGET config:max_users config:debug_mode config:api_key
1) "1000"
2) "true"
3) "abc123"

# Update single config value
127.0.0.1:6379> SET config:debug_mode "false"
OK
```

### 5. Rate Limiting

```redis
# Simple rate limiting (10 requests per minute)
127.0.0.1:6379> SET rate_limit:user123:minute 1 EX 60 NX
OK
# First request this minute

127.0.0.1:6379> INCR rate_limit:user123:minute
(integer) 2
# Second request

# Check current count
127.0.0.1:6379> GET rate_limit:user123:minute
"2"

# Check remaining time
127.0.0.1:6379> TTL rate_limit:user123:minute
(integer) 45
# 45 seconds left in this minute
```

---

## Performance Tips

### 1. Use Bulk Operations

```redis
# Slow: Multiple round trips
SET user:1:name "Alice"
SET user:1:age "25"  
SET user:1:city "NYC"

# Fast: Single round trip
MSET user:1:name "Alice" user:1:age "25" user:1:city "NYC"
```

### 2. Choose Right Expiration Method

```redis
# Good: Set with expiration immediately
SETEX cache:key 300 "value"

# Less efficient: Set then expire
SET cache:key "value"
EXPIRE cache:key 300
```

### 3. Avoid Huge Strings

```redis
# Don't store huge values in strings
SET big_data "10MB of JSON data..."

# Better: Use hashes for structured data
HSET user:data name "Alice" age "25" bio "Long bio text..."
```

---

## Common String Patterns

### 1. Toggle Values

```redis
# Toggle online/offline status
127.0.0.1:6379> SET user:status "offline"
OK

127.0.0.1:6379> GETSET user:status "online"
"offline"
# Got old status, set new status

# Simple toggle function
def toggle_status(user_id):
    current = redis.get(f"user:{user_id}:status") 
    new_status = "offline" if current == "online" else "online"
    return redis.getset(f"user:{user_id}:status", new_status)
```

### 2. Unique ID Generation

```redis
# Generate unique order IDs
127.0.0.1:6379> INCR order_id_counter
(integer) 1

127.0.0.1:6379> INCR order_id_counter  
(integer) 2

# Use the number to create order IDs: ORDER-000001, ORDER-000002
```

### 3. Feature Flags

```redis
# Enable/disable features
127.0.0.1:6379> SET feature:new_ui "enabled"
OK

127.0.0.1:6379> SET feature:beta_chat "disabled" 
OK

# Check if feature is enabled
127.0.0.1:6379> GET feature:new_ui
"enabled"

def is_feature_enabled(feature_name):
    return redis.get(f"feature:{feature_name}") == "enabled"
```

---

## Practice Exercises

### Exercise 1: User Preferences

```redis
# Store user preferences as JSON
SETEX prefs:user123 86400 '{"theme":"dark","lang":"en","notifications":true}'

# Get and update preferences
GET prefs:user123

# Store individual preferences (alternative approach)
MSET pref:user123:theme "dark" pref:user123:lang "en" pref:user123:notifications "true"

# Get all preferences
MGET pref:user123:theme pref:user123:lang pref:user123:notifications
```

### Exercise 2: Simple Analytics

```redis
# Track daily metrics
INCR metrics:2024-01-15:visitors
INCRBY metrics:2024-01-15:page_views 5
INCRBYFLOAT metrics:2024-01-15:revenue 29.99

# Get daily summary
MGET metrics:2024-01-15:visitors metrics:2024-01-15:page_views metrics:2024-01-15:revenue
```

### Exercise 3: Temporary Data

```redis
# Store verification code for 5 minutes
SETEX verify:email:alice@example.com 300 "123456"

# Check code
GET verify:email:alice@example.com

# Check remaining time
TTL verify:email:alice@example.com
```

### Exercise 4: Log Messages

```redis
# Store recent log messages
SET log:latest "User login successful"
APPEND log:latest " - IP: 192.168.1.1"
APPEND log:latest " - Time: 2024-01-15 10:30:00"

# Get full log message
GET log:latest

# Get message length
STRLEN log:latest
```

---

## String Commands Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `SET key value` | Store string | `SET name "Alice"` |
| `GET key` | Get string | `GET name` |
| `MSET key1 val1 key2 val2` | Set multiple | `MSET name "Bob" age "30"` |
| `MGET key1 key2` | Get multiple | `MGET name age` |
| `SETNX key value` | Set if not exists | `SETNX lock "acquired"` |
| `SETEX key seconds value` | Set with expiration | `SETEX session 3600 "data"` |
| `APPEND key value` | Add to end | `APPEND log " new data"` |
| `STRLEN key` | Get length | `STRLEN message` |
| `INCR key` | Increment by 1 | `INCR counter` |
| `INCRBY key amount` | Increment by amount | `INCRBY score 10` |
| `DECR key` | Decrement by 1 | `DECR lives` |
| `GETSET key value` | Get old, set new | `GETSET status "online"` |
| `GETRANGE key start end` | Get substring | `GETRANGE text 0 5` |

---

## What's Next?

Ready to learn about **Lists** - ordered collections perfect for queues, timelines, and activity feeds!
