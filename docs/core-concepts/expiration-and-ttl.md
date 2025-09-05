# Expiration and TTL

Redis TTL (Time To Live) lets you set automatic expiration on keys. This is essential for sessions, caches, temporary data, and preventing memory bloat. Redis will automatically delete expired keys, making it perfect for time-sensitive data.

## What is TTL?

TTL stands for **Time To Live** - how long a key should exist before Redis automatically deletes it.

Think of TTL as:

- A **timer** on a parking meter - expires after time runs out
- An **expiration date** on food - gets removed when it goes bad
- A **temporary pass** - only valid for a specific time period
- **Self-cleaning memory** - old data disappears automatically

---

## Basic Expiration Operations

### Setting Expiration on Keys

```redis
# Set key with immediate expiration
127.0.0.1:6379> SETEX session:abc123 3600 "user_data"
OK
# Key expires in 3600 seconds (1 hour)

# Set key first, then add expiration
127.0.0.1:6379> SET cache:weather "sunny"
OK
127.0.0.1:6379> EXPIRE cache:weather 300
(integer) 1
# Key expires in 300 seconds (5 minutes)

# Set expiration in milliseconds
127.0.0.1:6379> SET temp:token "abc123"
OK
127.0.0.1:6379> PEXPIRE temp:token 5000
(integer) 1
# Key expires in 5000 milliseconds (5 seconds)

# Set expiration at specific Unix timestamp
127.0.0.1:6379> SET reminder "meeting at 3pm"
OK
127.0.0.1:6379> EXPIREAT reminder 1705751400
(integer) 1
# Key expires at Unix timestamp 1705751400

# Set expiration at specific timestamp in milliseconds
127.0.0.1:6379> SET alert "system maintenance"
OK
127.0.0.1:6379> PEXPIREAT alert 1705751400000
(integer) 1
# Key expires at timestamp 1705751400000 (milliseconds)
```

### Checking Time Remaining

```redis
# Check time-to-live in seconds
127.0.0.1:6379> TTL session:abc123
(integer) 3456
# 3456 seconds remaining

# Check time-to-live in milliseconds
127.0.0.1:6379> PTTL session:abc123
(integer) 3456789
# 3456789 milliseconds remaining

# Different TTL return values
127.0.0.1:6379> TTL nonexistent_key
(integer) -2
# -2 = key does not exist

127.0.0.1:6379> SET permanent "no expiration"
OK
127.0.0.1:6379> TTL permanent
(integer) -1
# -1 = key exists but has no expiration

127.0.0.1:6379> SETEX temporary 60 "expires soon"
OK
127.0.0.1:6379> TTL temporary
(integer) 57
# Positive number = seconds until expiration
```

### Removing and Extending Expiration

```redis
# Remove expiration (make key permanent)
127.0.0.1:6379> SET data "important"
OK
127.0.0.1:6379> EXPIRE data 300
(integer) 1
127.0.0.1:6379> TTL data
(integer) 298

127.0.0.1:6379> PERSIST data
(integer) 1
# Expiration removed

127.0.0.1:6379> TTL data
(integer) -1
# Now permanent (no expiration)

# Extend expiration time
127.0.0.1:6379> SETEX session 1800 "user123"
OK
# Set for 30 minutes

127.0.0.1:6379> EXPIRE session 3600
(integer) 1
# Extend to 1 hour

# Reset expiration to new value
127.0.0.1:6379> TTL session
(integer) 3598
127.0.0.1:6379> EXPIRE session 7200
(integer) 1
# Now expires in 2 hours
```

---

## Expiration with Different Data Types

### String Expiration

```redis
# Cache API response for 10 minutes
127.0.0.1:6379> SETEX cache:api:weather 600 '{"temp":22,"condition":"sunny"}'
OK

# Session token for 24 hours
127.0.0.1:6379> SETEX auth:token:xyz789 86400 "user_session_data"
OK

# Temporary verification code for 5 minutes
127.0.0.1:6379> SETEX verify:email:user123 300 "123456"
OK

# Check remaining time
127.0.0.1:6379> TTL cache:api:weather
(integer) 573
# 573 seconds left

127.0.0.1:6379> TTL verify:email:user123
(integer) 287
# 287 seconds left
```

### Hash Expiration

```redis
# User session data expires in 2 hours
127.0.0.1:6379> HSET session:user456 user_id "456" role "admin" login_time "2024-01-21T10:00:00Z"
(integer) 3

127.0.0.1:6379> EXPIRE session:user456 7200
(integer) 1

# Temporary user profile cache for 15 minutes
127.0.0.1:6379> HSET cache:profile:alice name "Alice" email "alice@example.com" status "active"
(integer) 3

127.0.0.1:6379> EXPIRE cache:profile:alice 900
(integer) 1

# Check expiration
127.0.0.1:6379> TTL session:user456
(integer) 7156
127.0.0.1:6379> TTL cache:profile:alice
(integer) 876
```

### List Expiration

```redis
# Recent activities expire after 1 hour
127.0.0.1:6379> LPUSH recent:activities:user789 "logged_in" "viewed_profile" "posted_message"
(integer) 3

127.0.0.1:6379> EXPIRE recent:activities:user789 3600
(integer) 1

# Temporary task queue for 30 minutes
127.0.0.1:6379> RPUSH temp:tasks "task1" "task2" "task3"
(integer) 3

127.0.0.1:6379> EXPIRE temp:tasks 1800
(integer) 1

127.0.0.1:6379> TTL recent:activities:user789
(integer) 3567
127.0.0.1:6379> TTL temp:tasks
(integer) 1789
```

### Set Expiration

```redis
# Online users set expires in 10 minutes
127.0.0.1:6379> SADD online:users "alice" "bob" "charlie"
(integer) 3

127.0.0.1:6379> EXPIRE online:users 600
(integer) 1

# Temporary permissions for 1 hour
127.0.0.1:6379> SADD temp:permissions:user123 "read" "write" "delete"
(integer) 3

127.0.0.1:6379> EXPIRE temp:permissions:user123 3600
(integer) 1

127.0.0.1:6379> TTL online:users
(integer) 578
127.0.0.1:6379> TTL temp:permissions:user123
(integer) 3587
```

### Sorted Set Expiration

```redis
# Leaderboard for daily contest expires at midnight
127.0.0.1:6379> ZADD daily:leaderboard 1500 "alice" 1200 "bob" 1800 "charlie"
(integer) 3

# Set to expire at specific time (midnight)
127.0.0.1:6379> EXPIREAT daily:leaderboard 1705806000
(integer) 1
# Expires at midnight (Unix timestamp)

# Recent scores expire in 6 hours
127.0.0.1:6379> ZADD recent:scores 95 "player1" 87 "player2" 92 "player3"
(integer) 3

127.0.0.1:6379> EXPIRE recent:scores 21600
(integer) 1

127.0.0.1:6379> TTL daily:leaderboard
(integer) 45231
127.0.0.1:6379> TTL recent:scores
(integer) 21567
```

---

## Real-World TTL Use Cases

### 1. Session Management

```redis
# Create user session with 30-minute timeout
127.0.0.1:6379> HSET session:abc123 \
  user_id "1001" \
  username "alice" \
  role "user" \
  created "2024-01-21T10:00:00Z"
(integer) 4

127.0.0.1:6379> EXPIRE session:abc123 1800
(integer) 1
# Session expires in 30 minutes

# User activity extends session
127.0.0.1:6379> HSET session:abc123 last_activity "2024-01-21T10:15:00Z"
(integer) 0

# Extend session timeout
127.0.0.1:6379> EXPIRE session:abc123 1800
(integer) 1
# Reset to 30 minutes from now

# Check session validity
127.0.0.1:6379> EXISTS session:abc123
(integer) 1
# Session still valid

127.0.0.1:6379> TTL session:abc123
(integer) 1734
# 1734 seconds remaining

# Session cleanup (automatic when TTL expires)
# After 30 minutes of inactivity, session is automatically deleted
```

### 2. API Rate Limiting

```redis
# Rate limit: 100 requests per hour per user
127.0.0.1:6379> SET rate_limit:user123:hour 1 EX 3600 NX
OK
# First request this hour

127.0.0.1:6379> INCR rate_limit:user123:hour
(integer) 2
# Second request

127.0.0.1:6379> GET rate_limit:user123:hour
"2"
# Current request count

127.0.0.1:6379> TTL rate_limit:user123:hour
(integer) 3567
# Time remaining in current hour

# Check if user can make more requests
def can_make_request(user_id, limit=100):
    key = f"rate_limit:{user_id}:hour"
    current = redis.get(key) or 0
    if int(current) >= limit:
        return False
    redis.incr(key)
    redis.expire(key, 3600)  # Reset expiration
    return True

# Rate limit: 10 requests per minute
127.0.0.1:6379> SET rate_limit:user123:minute 1 EX 60 NX
OK

127.0.0.1:6379> INCR rate_limit:user123:minute
(integer) 2

127.0.0.1:6379> TTL rate_limit:user123:minute
(integer) 47
```

### 3. Caching with Expiration

```redis
# Cache expensive database query for 5 minutes
127.0.0.1:6379> SET cache:user_profile:123 '{"name":"Alice","email":"alice@example.com"}' EX 300
OK

# Cache API response for 10 minutes
127.0.0.1:6379> SETEX cache:weather:london 600 '{"temp":18,"condition":"cloudy"}'
OK

# Cache product data for 1 hour
127.0.0.1:6379> HSET cache:product:456 \
  name "Laptop" \
  price "999.99" \
  stock "25" \
  updated "2024-01-21T10:00:00Z"
(integer) 4

127.0.0.1:6379> EXPIRE cache:product:456 3600
(integer) 1

# Check cache validity
127.0.0.1:6379> TTL cache:user_profile:123
(integer) 267
# Cache valid for 267 more seconds

127.0.0.1:6379> TTL cache:weather:london
(integer) 543
# Cache valid for 543 more seconds

# Cache miss handling
def get_cached_data(key):
    data = redis.get(key)
    if data is None:
        # Cache miss - fetch from database
        data = fetch_from_database()
        redis.setex(key, 300, data)  # Cache for 5 minutes
    return data
```

### 4. Temporary Data Storage

```redis
# Email verification code (expires in 10 minutes)
127.0.0.1:6379> SETEX verify:email:alice@example.com 600 "123456"
OK

# Password reset token (expires in 1 hour)
127.0.0.1:6379> SETEX reset:token:xyz789 3600 "user123"
OK

# Temporary file upload token (expires in 15 minutes)
127.0.0.1:6379> SETEX upload:token:abc456 900 '{"user_id":"123","file_type":"image"}'
OK

# Check validity
127.0.0.1:6379> EXISTS verify:email:alice@example.com
(integer) 1
# Verification code still valid

127.0.0.1:6379> TTL verify:email:alice@example.com
(integer) 456
# 456 seconds remaining

127.0.0.1:6379> TTL reset:token:xyz789
(integer) 3456
# 3456 seconds remaining

# Handle expiration
def verify_email_code(email, code):
    stored_code = redis.get(f"verify:email:{email}")
    if stored_code is None:
        return {"error": "Code expired or invalid"}
    if stored_code == code:
        redis.delete(f"verify:email:{email}")  # Remove used code
        return {"success": True}
    return {"error": "Invalid code"}
```

### 5. Real-Time Features

```redis
# User typing indicator (expires in 5 seconds)
127.0.0.1:6379> SETEX typing:chat:room123:alice 5 "1"
OK

# User online status (expires in 30 seconds, renewed by heartbeat)
127.0.0.1:6379> SETEX online:user456 30 "active"
OK

# Live game session (expires in 2 hours)
127.0.0.1:6379> HSET game:session:xyz789 \
  player1 "alice" \
  player2 "bob" \
  started "2024-01-21T10:00:00Z" \
  status "active"
(integer) 4

127.0.0.1:6379> EXPIRE game:session:xyz789 7200
(integer) 1

# Heartbeat to keep online status
def user_heartbeat(user_id):
    redis.setex(f"online:{user_id}", 30, "active")

# Check if user is typing
127.0.0.1:6379> EXISTS typing:chat:room123:alice
(integer) 1
# Alice is currently typing

# Wait 6 seconds
127.0.0.1:6379> EXISTS typing:chat:room123:alice
(integer) 0
# Typing indicator expired
```

---

## Advanced TTL Patterns

### 1. Sliding Window Expiration

```redis
# Keep last 100 user actions with 1-hour sliding window
127.0.0.1:6379> ZADD user:actions:alice 1705751400 "login"
(integer) 1

127.0.0.1:6379> ZADD user:actions:alice 1705751500 "view_profile"
(integer) 1

127.0.0.1:6379> ZADD user:actions:alice 1705751600 "post_message"
(integer) 1

# Remove actions older than 1 hour
127.0.0.1:6379> ZREMRANGEBYSCORE user:actions:alice -inf 1705748000
(integer) 0
# Remove actions older than 1 hour ago

# Set expiration on the entire sorted set
127.0.0.1:6379> EXPIRE user:actions:alice 3600
(integer) 1
```

### 2. Multi-Level Expiration

```redis
# Cache with different TTLs based on data type
def set_cache_with_ttl(key, data, data_type):
    ttl_map = {
        "user_profile": 1800,      # 30 minutes
        "product_data": 3600,      # 1 hour
        "static_content": 86400,   # 24 hours
        "api_response": 300        # 5 minutes
    }
    ttl = ttl_map.get(data_type, 600)  # Default 10 minutes
    redis.setex(key, ttl, data)

# Usage
set_cache_with_ttl("cache:user:123", user_data, "user_profile")
set_cache_with_ttl("cache:product:456", product_data, "product_data")
```

### 3. Conditional Expiration

```redis
# Set expiration only if key doesn't already have one
127.0.0.1:6379> SET temp:data "value"
OK

127.0.0.1:6379> TTL temp:data
(integer) -1
# No expiration set

# Set expiration if not already set
def set_expiration_if_none(key, ttl):
    if redis.ttl(key) == -1:  # No expiration
        redis.expire(key, ttl)
        return True
    return False

# Extend expiration only if key is about to expire
def extend_if_expiring_soon(key, threshold=300, new_ttl=3600):
    current_ttl = redis.ttl(key)
    if 0 < current_ttl < threshold:  # Expires within threshold
        redis.expire(key, new_ttl)
        return True
    return False
```

---

## TTL Best Practices

### 1. Choosing Appropriate TTL Values

```redis
# Short TTL for frequently changing data
SETEX cache:stock_price:AAPL 60 "150.25"
# Stock prices change frequently - 1 minute TTL

# Medium TTL for moderately stable data
SETEX cache:user_profile:123 1800 "user_data"
# User profiles change occasionally - 30 minute TTL

# Long TTL for stable data
SETEX cache:country_list 86400 "country_data"
# Country list rarely changes - 24 hour TTL

# Very short TTL for real-time features
SETEX typing:indicator:user456 3 "1"
# Typing indicators are momentary - 3 second TTL
```

### 2. Memory Management with TTL

```redis
# Prevent memory bloat with automatic cleanup
def store_temporary_data(key, data, max_age=3600):
    redis.setex(key, max_age, data)

# Clean up old analytics data daily
def setup_daily_cleanup():
    # Keep only last 30 days of daily stats
    for i in range(31, 365):
        date_key = f"stats:daily:{date_n_days_ago(i)}"
        redis.expire(date_key, 86400)  # Expire in 24 hours

# Batch set expiration on related keys
def expire_user_data(user_id, ttl=1800):
    keys = [
        f"cache:profile:{user_id}",
        f"cache:preferences:{user_id}",
        f"temp:actions:{user_id}"
    ]
    for key in keys:
        if redis.exists(key):
            redis.expire(key, ttl)
```

### 3. Monitoring and Alerting

```redis
# Check for keys about to expire
def check_expiring_sessions(threshold=300):
    # Find sessions expiring within 5 minutes
    expiring_sessions = []
    for key in redis.scan_iter(match="session:*"):
        ttl = redis.ttl(key)
        if 0 < ttl < threshold:
            expiring_sessions.append(key)
    return expiring_sessions

# Monitor TTL distribution
def get_ttl_stats():
    ttl_ranges = {
        "expired": 0,
        "no_expiry": 0,
        "short_term": 0,    # < 1 hour
        "medium_term": 0,   # 1-24 hours
        "long_term": 0      # > 24 hours
    }
    
    for key in redis.scan_iter():
        ttl = redis.ttl(key)
        if ttl == -2:
            ttl_ranges["expired"] += 1
        elif ttl == -1:
            ttl_ranges["no_expiry"] += 1
        elif ttl < 3600:
            ttl_ranges["short_term"] += 1
        elif ttl < 86400:
            ttl_ranges["medium_term"] += 1
        else:
            ttl_ranges["long_term"] += 1
    
    return ttl_ranges
```

---

## Common TTL Patterns

### 1. Session Extension Pattern

```python
def extend_session_on_activity(session_id, activity_ttl=1800):
    session_key = f"session:{session_id}"
    if redis.exists(session_key):
        # Update last activity
        redis.hset(session_key, "last_activity", time.time())
        # Extend session
        redis.expire(session_key, activity_ttl)
        return True
    return False
```

### 2. Cache-Aside with TTL

```python
def get_with_cache(key, fetch_function, ttl=300):
    # Try cache first
    cached = redis.get(key)
    if cached:
        return json.loads(cached)
    
    # Cache miss - fetch data
    data = fetch_function()
    
    # Store in cache with TTL
    redis.setex(key, ttl, json.dumps(data))
    return data
```

### 3. Rate Limiting with TTL

```python
def rate_limit_check(user_id, limit=100, window=3600):
    key = f"rate_limit:{user_id}"
    current = redis.get(key)
    
    if current is None:
        # First request in window
        redis.setex(key, window, 1)
        return True
    
    if int(current) >= limit:
        return False
    
    # Increment and maintain TTL
    redis.incr(key)
    return True
```

---

## Practice Exercises

### Exercise 1: User Session System

```redis
# Create session with 1 hour expiration
HSET session:user123 user_id "123" role "user" created "2024-01-21T10:00:00Z"
EXPIRE session:user123 3600

# Simulate user activity (extend session)
HSET session:user123 last_activity "2024-01-21T10:30:00Z"
EXPIRE session:user123 3600

# Check session status
EXISTS session:user123
TTL session:user123
```

### Exercise 2: API Response Caching

```redis
# Cache API response for 10 minutes
SETEX cache:api:weather:london 600 '{"temp":20,"humidity":65}'

# Check cache before API call
GET cache:api:weather:london

# Monitor cache expiration
TTL cache:api:weather:london
```

### Exercise 3: Temporary Verification System

```redis
# Email verification code (5 minutes)
SETEX verify:email:user456 300 "987654"

# Phone verification code (2 minutes)
SETEX verify:phone:user456 120 "123456"

# Check code validity
EXISTS verify:email:user456
TTL verify:email:user456

# Simulate verification
GET verify:email:user456
DEL verify:email:user456
```

### Exercise 4: Rate Limiting Implementation

```redis
# Set rate limit (100 requests per hour)
SET rate:user789:hour 1 EX 3600 NX

# Subsequent requests
INCR rate:user789:hour
GET rate:user789:hour

# Check remaining time in window
TTL rate:user789:hour

# Check if limit exceeded
GET rate:user789:hour
# If >= 100, reject request
```

---

## TTL Commands Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `EXPIRE key seconds` | Set expiration in seconds | `EXPIRE session 3600` |
| `PEXPIRE key milliseconds` | Set expiration in milliseconds | `PEXPIRE token 5000` |
| `EXPIREAT key timestamp` | Set expiration at Unix timestamp | `EXPIREAT data 1705751400` |
| `PEXPIREAT key timestamp` | Set expiration at timestamp (ms) | `PEXPIREAT data 1705751400000` |
| `TTL key` | Get time-to-live in seconds | `TTL session` |
| `PTTL key` | Get time-to-live in milliseconds | `PTTL token` |
| `PERSIST key` | Remove expiration | `PERSIST important_data` |
| `SETEX key seconds value` | Set string with expiration | `SETEX cache 300 "data"` |
| `PSETEX key milliseconds value` | Set string with expiration (ms) | `PSETEX temp 5000 "data"` |

---

## TTL Return Values

| TTL Value | Meaning |
|-----------|---------|
| Positive number | Seconds until expiration |
| -1 | Key exists but has no expiration |
| -2 | Key does not exist |

---

## What's Next

You've completed the **Core Concepts** section! You now understand all Redis data types and their operations. Ready to move on to **Practical Examples** where you'll build real applications using these concepts.
