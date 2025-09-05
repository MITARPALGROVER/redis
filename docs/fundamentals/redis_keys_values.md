# Keys and Values

Understanding how to work with keys and values effectively is crucial for building scalable Redis applications. Let's master the art of Redis key management!

## What are Redis Keys?

In Redis, **keys** are unique identifiers that point to **values**. Think of keys as addresses and values as the data stored at those addresses.

```redis
# Key: "username"    →    Value: "alice_cooper"
# Key: "user:1001"   →    Value: {"name": "Alice", "age": 25}
# Key: "counter:views" →   Value: 1500
```

### Key Characteristics

- **Unique**: Each key can exist only once in a database
- **Case-sensitive**: `User` and `user` are different keys
- **Binary safe**: Can contain any byte sequence (including null bytes)
- **Maximum size**: 512MB per key (practically much smaller)
- **Expire automatically**: Can have TTL (Time To Live)

---

## Key Naming Conventions

Good key naming is essential for maintainable Redis applications.

### 1. Use Descriptive Names

```redis
# Bad - unclear what this represents
SET u1 "john"
SET d "2024-01-15"

# Good - clear and descriptive
SET user:1001:name "john"
SET last_backup_date "2024-01-15"
```

### 2. Use Hierarchical Structure

Use colons (`:`) to create namespaces and hierarchies:

```redis
# User-related keys
user:1001:profile          # User profile data
user:1001:settings         # User preferences
user:1001:sessions         # Active sessions

# Product-related keys  
product:electronics:456    # Electronics product #456
product:books:789         # Books product #789

# Application features
cache:homepage            # Homepage cache
queue:email_tasks        # Email task queue
stats:daily:2024-01-15   # Daily statistics
```

### 3. Common Naming Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| `type:id` | `user:1001` | Single entity |
| `type:id:attribute` | `user:1001:email` | Entity attribute |
| `type:category:id` | `product:books:456` | Categorized items |
| `app:feature:data` | `cache:user:profile` | Application features |
| `date:type:id` | `2024-01:stats:views` | Time-based data |

### 4. Naming Best Practices

!!! tip "Key Naming Guidelines"
    - **Use lowercase** for consistency
    - **Use underscores or hyphens** for multi-word keys
    - **Keep keys reasonably short** but descriptive
    - **Use consistent patterns** across your application
    - **Avoid special characters** that might cause issues

```redis
# Good examples
user:profile:1001
session:active:abc123
cache:product_list
stats:daily:page_views
queue:email-notifications

# Avoid these patterns
User Profile 1001          # Spaces and mixed case
session/active/abc123      # Inconsistent separators
cache@product#list         # Special characters
x                         # Too vague
very_long_key_name_that_describes_everything_in_detail  # Too long
```

---

## Working with Keys

### Basic Key Operations

```redis
# Check if key exists
127.0.0.1:6379> EXISTS user:1001
(integer) 1

# Get key type
127.0.0.1:6379> TYPE user:1001:profile
hash

# Rename key
127.0.0.1:6379> RENAME old_key new_key
OK

# Copy key (Redis 6.2+)
127.0.0.1:6379> COPY source_key dest_key
(integer) 1

# Delete keys
127.0.0.1:6379> DEL user:1001:temp
(integer) 1

# Delete multiple keys
127.0.0.1:6379> DEL key1 key2 key3
(integer) 3
```

### Finding Keys

```redis
# List all keys (use carefully in production!)
127.0.0.1:6379> KEYS *

# Find keys with pattern
127.0.0.1:6379> KEYS user:*
1) "user:1001:profile"
2) "user:1001:settings"
3) "user:1002:profile"

# More specific patterns
127.0.0.1:6379> KEYS user:100?:profile  # ? matches single character.,
1) "user:1001:profile"
2) "user:1002:profile"

127.0.0.1:6379> KEYS user:[12]*         # [12] matches 1 or 2
1) "user:1001:profile"
2) "user:1001:settings"
3) "user:2001:profile"
```

!!! warning "Production Warning"
    `KEYS *` can be slow on large databases. Use `SCAN` instead for production:
    ```redis
    SCAN 0 MATCH user:* COUNT 100
    ```

### Safe Key Discovery with SCAN

**Why SCAN is better than KEYS:**

- **Non-blocking**: SCAN doesn't freeze Redis while searching
- **Iterative**: Returns results in chunks, not all at once
- **Production-safe**: Won't impact performance on large datasets
- **Memory-friendly**: Processes keys in batches

**How SCAN works:**

SCAN uses a **cursor-based approach** to iterate through keys:

```redis
# Start scanning from cursor 0
127.0.0.1:6379> SCAN 0 MATCH user:* COUNT 10
1) "5"           # Next cursor to continue from
2) 1) "user:1001:profile"
   2) "user:1001:settings"
   3) "user:1002:profile"

# Continue scanning with returned cursor (5)
127.0.0.1:6379> SCAN 5 MATCH user:* COUNT 10
1) "12"          # Next cursor
2) 1) "user:1003:profile"
   2) "user:1004:settings"

# Continue until cursor returns "0" (scan complete)
127.0.0.1:6379> SCAN 12 MATCH user:* COUNT 10
1) "0"           # Cursor 0 means scan is complete
2) 1) "user:1005:profile"
   2) "user:1006:settings"
```

**SCAN Parameters Explained:**

- **Cursor**: Position in the keyspace (start with 0, use returned value to continue)
- **MATCH**: Pattern to filter keys (optional)
- **COUNT**: Hint for how many keys to return per iteration (not exact)

**Important SCAN characteristics:**

```redis
# COUNT is a hint, not a guarantee
SCAN 0 MATCH user:* COUNT 100
# Might return 50, 100, or 150 keys depending on Redis internal structure

# SCAN can return duplicates across iterations
# Always deduplicate results in your application

# SCAN guarantees to visit all keys that exist for the full iteration
# Keys added during scan might or might not be returned
```

**SCAN vs KEYS comparison:**

| Feature | KEYS | SCAN |
|---------|------|------|
| **Blocking** |  Blocks Redis |  Non-blocking |
| **Memory usage** |  High |  Low |
| **Production safe** |  No |  Yes |
| **Complete results** |  All at once |  All eventually |
| **Order** |  Unordered |  Unordered |
| **Duplicates** |  No duplicates |  Possible duplicates |

**Real-world SCAN examples:**

```redis
# Find all session keys for cleanup
SCAN 0 MATCH "session:*" COUNT 500

# Find cache keys to analyze memory usage
SCAN 0 MATCH "cache:*" COUNT 100

# Find expired temp keys
SCAN 0 MATCH "temp:*" COUNT 200

# Find all user profile keys
SCAN 0 MATCH "user:*:profile" COUNT 300
```



---

## Key Expiration and TTL

Redis can automatically delete keys after a specified time - perfect for caches, sessions, and temporary data.

### Setting Expiration

```redis
# Set key with expiration (seconds)
127.0.0.1:6379> SETEX session:abc123 3600 "user_data"
OK

# Set key with expiration (milliseconds)  
127.0.0.1:6379> PSETEX temp:token 30000 "quick_access"
OK

# Add expiration to existing key
127.0.0.1:6379> EXPIRE user:cache 300
(integer) 1

# Set expiration at specific timestamp
127.0.0.1:6379> EXPIREAT user:cache 1757069124
(integer) 1
```

**What does timestamp `1757069124` mean?**

This is a **Unix timestamp** (also called epoch time) - the number of seconds since January 1, 1970, 00:00:00 UTC.

**Converting the timestamp:**
```
1757069124 = Fri Sep 05 2025 10:45:24 GMT+0000
```

```redis title="Current time in Redis"
# Current timestamp (example)
127.0.0.1:6379> TIME
1) "1757069246"  # Current Unix timestamp
2) "670472"      # Microseconds  
```

### Checking and Managing TTL

```redis
# Check time-to-live (seconds)
127.0.0.1:6379> TTL session:abc123
(integer) 3456

# Check time-to-live (milliseconds)
127.0.0.1:6379> PTTL session:abc123
(integer) 3456789

# Remove expiration (make key persistent)
127.0.0.1:6379> PERSIST session:abc123
(integer) 1
```

### TTL Return Values

| TTL Value | Meaning |
|-----------|---------|
| **Positive number** | Seconds until expiration |
| **-1** | Key exists but no expiration set |
| **-2** | Key does not exist |

### Practical Expiration Examples

```redis
# User session (1 hour)
SETEX session:user123 3600 "logged_in"

# Cache entry (10 minutes)
SETEX cache:user:profile:456 600 '{"name":"John","email":"john@example.com"}'

# Rate limiting (1 minute window)
SETEX rate_limit:api:user123 60 "5"

# Temporary verification code (5 minutes)
SETEX verify:email:token123 300 "user456"

# Daily statistics (expires at midnight)
# Calculate seconds until next midnight
EXPIREAT stats:daily:2024-01-15 1704153600
```

---

## Understanding Values

### Value Types and Limits

| Value Type | Max Size | Best For |
|------------|----------|----------|
| **String** | 512MB | Text, JSON, counters |
| **List** | 4 billion elements | Queues, timelines |
| **Set** | 4 billion members | Tags, unique items |
| **Hash** | 4 billion fields | Objects, records |
| **Sorted Set** | 4 billion members | Rankings, scores |

### Value Size Considerations

Ways to find length of different data types:

```redis
# Check value memory usage
127.0.0.1:6379> MEMORY USAGE user:1001:profile
(integer) 64

# String length
127.0.0.1:6379> STRLEN user:name
(integer) 12

# Collection sizes
127.0.0.1:6379> LLEN user:activity      # List length
127.0.0.1:6379> SCARD user:interests    # Set cardinality
127.0.0.1:6379> HLEN user:profile       # Hash length
127.0.0.1:6379> ZCARD leaderboard       # Sorted set cardinality
```

---

## Key Design Patterns

**Why do we need design patterns for Redis keys?**

Think of key design patterns like organizing your house - you could throw everything randomly in rooms, or you could organize logically (kitchen items in kitchen, bedroom items in bedroom). Redis key patterns help you:

- **Find data quickly** - Know exactly where to look
- **Scale efficiently** - Handle millions of keys without confusion
- **Maintain easily** - Other developers understand your structure
- **Avoid conflicts** - Different features don't interfere with each other

### 1. Entity-Attribute Pattern

**The Problem:** You have complex objects (like users) with multiple pieces of information.

**Bad Approach:**
```redis
# Putting everything in one big JSON string
SET user_1001 '{"profile":{"name":"John","email":"john@example.com"},"settings":{"theme":"dark","notifications":"on"},"activity":["login","view_profile"],"friends":["user:1002","user:1003"]}'
```

**Problems with bad approach:**

- Must fetch ALL data even if you only need the name
- Can't efficiently update just one piece (must rewrite entire JSON)
- Can't use Redis data types optimally
- Memory inefficient for large objects

**Good Approach - Entity-Attribute Pattern:**
```redis
# Split entity into logical attributes with appropriate data types
user:1001:profile    → Hash: {name: "John", email: "john@example.com"}
user:1001:settings   → Hash: {theme: "dark", notifications: "on"}
user:1001:activity   → List: ["login", "view_profile", "update_settings"]
user:1001:friends    → Set: ["user:1002", "user:1003", "user:1004"]
```

**Why this is better:**
```redis
# Get only what you need
HGET user:1001:profile name          # Just the name
HGETALL user:1001:settings          # All settings

# Update efficiently 
HSET user:1001:profile email "new@email.com"  # Update only email
LPUSH user:1001:activity "logout"             # Add activity without touching profile

# Use Redis features
SISMEMBER user:1001:friends "user:1002"       # Fast friend check
LLEN user:1001:activity                       # Count activities
```

**Real-world example:**
```redis
# E-commerce user
user:1001:profile     → {name: "Alice", email: "alice@shop.com", phone: "+1234567890"}
user:1001:cart        → ["product:123", "product:456", "product:789"]
user:1001:wishlist    → {"product:111", "product:222", "product:333"}
user:1001:orders      → {order_count: 15, last_order: "2024-01-15", total_spent: "1299.99"}
user:1001:preferences → {currency: "USD", language: "en", newsletter: "true"}
```

### 2. Time-based Patterns

**The Problem:** Your application generates data over time and you need to organize it by periods.

**Bad Approach:**
```redis
# All data mixed together
SET stats_data '{"2024-01-15":{"page_views":15234,"users":3456},"2024-01-16":{"page_views":16789,"users":3891}}'
```

**Problems:**

- Must parse entire JSON to get one day's data
- Gets huge as time progresses
- Can't set different expiration for different days
- Hard to analyze trends

**Good Approach - Time-based Pattern:**
```redis
# Organize by time periods
# Daily metrics
stats:2024-01-15:page_views     → String: "15234"
stats:2024-01-15:unique_users   → String: "3456"
stats:2024-01-16:page_views     → String: "16789"
stats:2024-01-16:unique_users   → String: "3891"

# Hourly data for detailed analysis
logs:2024-01-15:14:errors      → List: ["404 error on /api/users", "500 error in database"]
logs:2024-01-15:15:errors      → List: ["timeout in payment service"]
cache:2024-01-15:14:trending   → Sorted Set: {topic1: 150, topic2: 89, topic3: 234}

# User activity by date
user:1001:activity:2024-01-15  → List: ["login", "view_profile", "update_settings"]
user:1001:activity:2024-01-16  → List: ["login", "purchase", "logout"]
```

**Why this is better:**
```redis
# Get specific time period data
GET stats:2024-01-15:page_views              # Just today's views
LRANGE user:1001:activity:2024-01-15 0 -1   # Today's user activity

# Set different expiration policies
EXPIRE logs:2024-01-15:14:errors 604800     # Keep logs for 1 week
EXPIRE stats:2024-01-15:page_views 2592000  # Keep stats for 30 days

# Easy time-based cleanup
DEL stats:2024-01-01:*                      # Delete old January data

# Analyze trends across time
for date in 2024-01-{01..31}; do
    redis-cli GET "stats:$date:page_views"
done
```

**Real-world example:**
```redis
# Web analytics
analytics:2024-01-15:page_views → "15234"
analytics:2024-01-15:unique_visitors → "3456"
analytics:2024-01-15:bounce_rate → "45.6"

# Error tracking
errors:2024-01-15:api:count → "23"
errors:2024-01-15:database:count → "5"
errors:2024-01-15:critical → List: ["payment_failed", "auth_timeout"]

# User engagement by hour
engagement:2024-01-15:09:active_users → "1234"
engagement:2024-01-15:10:active_users → "2345"
engagement:2024-01-15:11:active_users → "3456"
```

### 3. Categorization Patterns

**The Problem:** You have different types of content that need different treatment.

**Bad Approach:**
```redis
# Everything mixed together
SET all_products '{"electronics":{"featured":["laptop","phone"]},"books":{"bestsellers":["book1","book2"]},"clothing":{"sale":["shirt","pants"]}}'
```

**Problems:**

- Can't efficiently query one category
- Different categories have different needs (sorting, filtering)
- Hard to add new categories

**Good Approach - Categorization Pattern:**
```redis
# Separate by category and use appropriate data structures
# Product categories
products:electronics:featured   → Sorted Set: {laptop: 95, smartphone: 89, tablet: 76}
products:books:bestsellers     → Sorted Set: {book1: 150, book2: 120, book3: 98}
products:clothing:sale         → Set: {"shirt_123", "pants_456", "jacket_789"}

# Content types
content:articles:published     → Set: {"article_001", "article_002", "article_003"}
content:videos:trending       → Sorted Set: {video1: 1500, video2: 1200, video3: 950}
content:images:recent         → List: ["img_001", "img_002", "img_003"]  # Chronological order
```

**Why this is better:**
```redis
# Category-specific operations
ZREVRANGE products:electronics:featured 0 2 WITHSCORES  # Top 3 electronics
SCARD products:clothing:sale                            # Count of sale items
SISMEMBER content:articles:published "article_001"     # Check if published

# Category-specific expiration
EXPIRE content:images:recent 86400                      # Recent images expire daily
# articles:published never expires (permanent content)

# Easy category management
SADD products:clothing:sale "new_jacket_999"           # Add to sale
ZREM products:electronics:featured "old_laptop"        # Remove from featured
```

**Real-world example:**
```redis
# News website
news:sports:trending     → Sorted Set: {article1: 250, article2: 180, article3: 120}
news:politics:breaking   → List: ["breaking1", "breaking2"]  # Chronological
news:tech:featured      → Set: {"tech1", "tech2", "tech3"}

# Social media
posts:images:recent     → List: ["img1", "img2", "img3"]
posts:videos:popular    → Sorted Set: {vid1: 1500, vid2: 1200}
posts:text:trending     → Sorted Set: {post1: 89, post2: 76}
```

### 4. Cache Patterns

**The Problem:** You need to cache different types of data with different strategies.

**Bad Approach:**
```redis
# Generic cache without structure
SET cache_1 "some homepage html"
SET cache_2 "some user list"
SET cache_3 "some search results"
```

**Problems:**

- Can't manage different cache types differently
- No way to clear specific cache categories
- Hard to set appropriate expiration times

**Good Approach - Cache Pattern:**
```redis
# Structured cache with clear purposes
# Page cache (HTML content)
cache:page:/homepage           → String: "<html>...</html>"
cache:page:/products/123       → String: "<html>product page...</html>"

# API response cache (JSON data)
cache:api:users:list           → String: '[{"id":1,"name":"John"}...]'
cache:api:products:search:laptop → String: '{"total":45,"products":[...]}'

# Query result cache (processed data)
cache:query:popular_posts      → List: ["post_123", "post_456", "post_789"]
cache:query:user_count         → String: "15234"
```

**Why this is better:**
```redis
# Different expiration strategies
SETEX cache:page:/homepage 300 "<html>..."          # Page cache: 5 minutes
SETEX cache:api:users:list 600 '[{"id":1}...]'      # API cache: 10 minutes  
SETEX cache:query:popular_posts 3600 '["post1"]'    # Query cache: 1 hour

# Selective cache clearing
DEL cache:page:*           # Clear all page cache
DEL cache:api:users:*      # Clear user-related API cache
DEL cache:query:*          # Clear all query cache

# Cache monitoring
KEYS cache:page:*          # See all cached pages
KEYS cache:api:*           # See all cached API responses
```

**Real-world example:**
```redis
# E-commerce site
cache:page:homepage → "<html>homepage content</html>"
cache:page:category:electronics → "<html>electronics page</html>"
cache:api:product:123:details → '{"name":"Laptop","price":999}'
cache:search:query:laptop:page:1 → '{"total":45,"products":[...]}'
cache:user:1001:recommendations → '["product:789","product:456"]'

# Different TTL for different cache types
SETEX cache:page:homepage 600 "..."           # 10 minutes (changes often)
SETEX cache:api:product:123:details 3600 "..." # 1 hour (stable data)
SETEX cache:search:query:laptop:page:1 1800 "..." # 30 minutes (search results)
```

### Pattern Comparison

| Pattern | Best For | Key Structure | Data Types Used |
|---------|----------|---------------|-----------------|
| **Entity-Attribute** | Complex objects | `entity:id:attribute` | Hash, List, Set |
| **Time-based** | Time-series data | `type:date:metric` | String, List, Sorted Set |
| **Categorization** | Different content types | `type:category:data` | Set, Sorted Set, List |
| **Cache** | Temporary data | `cache:type:identifier` | String (mostly) |

### Choosing the Right Pattern

**Use Entity-Attribute when:**

- You have complex objects (users, products, orders)
- Different parts are accessed separately
- You need efficient partial updates

**Use Time-based when:**

- You generate data over time (logs, metrics, events)
- You need time-based analysis
- You want automatic cleanup of old data

**Use Categorization when:**

- You have different types of content
- Different types need different operations
- You want category-specific management

**Use Cache when:**

- You're storing temporary data
- You need different expiration strategies
- You want selective cache clearing

---

## Key Performance Optimization

### 1. Memory-Efficient Keys

**Why key length matters:** Redis stores every key name in memory. With millions of keys, longer names consume significantly more RAM.

```redis
# Use shorter keys when possible
# Less efficient (45 bytes for key name)
SET user:profile:personal_information:full_name "John Smith"

# More efficient (11 bytes for key name)
SET u:p:1001:name "John Smith"

# But balance with readability for maintenance
# Good compromise (16 bytes for key name)
SET user:1001:name "John Smith"
```

**Memory savings example:** With 1 million user keys, the difference between `user:profile:personal_information:full_name` (45 bytes) and `user:1001:name` (16 bytes) saves about 29MB of RAM just for key names!

### 2. Key Distribution

**Why distribution matters:** Redis stores keys in hash slots. If all keys start with the same prefix, they might end up in the same hash slot, creating a "hot spot" that gets overloaded.

```redis
# Distribute keys across keyspace to avoid hot spots
# All keys start with same prefix (potential hotspot)
SET user_1001_profile "data"
SET user_1002_profile "data" 
SET user_1003_profile "data"

# Better distribution
SET profile:user:1001 "data"
SET settings:user:1001 "data"
SET activity:user:1001 "data"
```

### 3. Key Expiration Strategies

**Why stagger expiration:** If many keys expire at exactly the same time, Redis might lag while deleting them all at once.

```redis
# Stagger expiration times to avoid mass expiration
BASE_TTL=3600
JITTER=$((RANDOM % 300))  # 0-300 seconds
FINAL_TTL=$((BASE_TTL + JITTER))

redis-cli SETEX "cache:user:$USER_ID" "$FINAL_TTL" "$DATA"
```

**What this does:** Instead of all cache keys expiring at exactly 1 hour, they expire between 1 hour and 1 hour 5 minutes, spreading the load.

---

## Practical Exercises

### Exercise 1: User Management System

Design keys for a user management system:

```redis
# User profile
HSET user:1001:profile name "Alice Cooper" email "alice@example.com"
HSET user:1001:profile created "2024-01-15" last_login "2024-01-20"

# User permissions
SADD user:1001:roles "admin" "moderator"
SADD user:1001:permissions "read_users" "write_posts" "delete_comments"

# User activity with expiration
LPUSH user:1001:activity:2024-01-20 "login" "view_dashboard" "edit_profile"
EXPIRE user:1001:activity:2024-01-20 2592000  # 30 days

# User session
SETEX session:abc123def456 7200 '{"user_id":1001,"ip":"192.168.1.100"}'
```

### Exercise 2: E-commerce Cache System

```redis
# Product cache with different TTLs
SETEX cache:product:123:details 3600 '{"name":"Laptop","price":999}'
SETEX cache:product:123:reviews 1800 '[{"rating":5,"comment":"Great!"}]'
SETEX cache:product:123:inventory 300 '{"stock":15,"reserved":3}'

# Category listings
SETEX cache:category:electronics:page:1 600 '["product:123","product:456"]'
SETEX cache:search:laptop:page:1 900 '{"total":45,"products":[...]}'

# User-specific cache
SETEX cache:user:1001:recommendations 7200 '["product:789","product:101"]'
SETEX cache:user:1001:cart 3600 '{"items":3,"total":125.99}'
```

### Exercise 3: Real-time Application

```redis
# Online user tracking
SADD online:users "user:1001" "user:1002" "user:1003"
SETEX user:1001:last_seen 300 "2024-01-20T15:30:00Z"

# Real-time notifications
LPUSH notifications:user:1001 '{"type":"message","from":"user:1002"}'
EXPIRE notifications:user:1001 86400  # 24 hours

# Live activity feed
ZADD feed:global 1705751400 "user:1001:posted:article:123"
ZADD feed:global 1705751450 "user:1002:liked:article:123"

# Rate limiting
SETEX rate:api:user:1001:minute 60 "5"    # 5 requests this minute
SETEX rate:api:user:1001:hour 3600 "100"  # 100 requests this hour
```

---

## Key Analysis and Debugging

### Finding Memory Usage

**Why monitor memory:** Understanding which keys consume the most memory helps optimize your Redis instance.

```redis
# Total memory info - shows overall Redis memory statistics
127.0.0.1:6379> INFO memory
# Returns detailed memory information
used_memory:1048576              # Memory used by Redis (1MB)
used_memory_human:1.00M          # Human readable format
used_memory_rss:2097152          # Memory from OS perspective
used_memory_peak:2097152         # Peak memory usage
maxmemory:0                      # Max memory limit (0 = unlimited)
...

# Memory usage by key - shows exact bytes used by a specific key
127.0.0.1:6379> MEMORY USAGE user:1001:profile
(integer) 96
# This key uses 96 bytes of memory (includes overhead)

# Sample keys to analyze patterns - shows internal Redis object details
127.0.0.1:6379> DEBUG OBJECT user:1001:profile
Value at:0x7f8b8c0a5c80 refcount:1 encoding:zipmap serializedlength:45
# refcount: How many references point to this object
# encoding: Internal data structure used (zipmap, hashtable, etc.)
# serializedlength: Size when saved to disk
```

### Key Distribution Analysis

**Why analyze distribution:** Understanding your key patterns helps identify naming issues and optimize storage.

```bash
# Analyze key patterns with redis-cli
# This command finds common key prefixes and counts them
redis-cli --scan --pattern "user:*" | head -1000 | \
awk -F: '{print $1":"$2}' | sort | uniq -c | sort -nr

# Sample output:
#    450 user:1001
#    380 user:1002  
#    275 user:1003
#     89 user:cache
#     12 user:temp
```

**Breaking down this complex command:**

1. **`redis-cli --scan --pattern "user:*"`** - Safely scan for all keys starting with "user:"
2. **`head -1000`** - Take only first 1000 results (prevent overwhelming output)
3. **`awk -F: '{print $1":"$2}'`** - Split by colon (:) and print first two parts
   - `user:1001:profile` becomes `user:1001`
   - `user:1001:settings` becomes `user:1001`
4. **`sort`** - Sort the patterns alphabetically
5. **`uniq -c`** - Count duplicate patterns
6. **`sort -nr`** - Sort by count (highest first)

**What this tells you:** User 1001 has 450 keys, user 1002 has 380 keys, etc. This helps identify users with too many keys.

### Finding Hot Keys

**Why find hot keys:** Heavily accessed keys can become bottlenecks and impact performance.

```redis
# Monitor frequently accessed keys - shows real-time command activity
127.0.0.1:6379> MONITOR | grep -E "(GET|SET|HGET)"
# Sample output (live stream):
1704067200.123456 [0 127.0.0.1:54321] "GET" "user:1001:profile"
1704067200.234567 [0 127.0.0.1:54322] "HGET" "user:1001:settings" "theme"
1704067200.345678 [0 127.0.0.1:54323] "SET" "cache:homepage" "<html>..."
1704067200.456789 [0 127.0.0.1:54324] "GET" "user:1001:profile"

# Use Redis slow log for analysis - tracks commands that take too long
127.0.0.1:6379> CONFIG SET slowlog-log-slower-than 10000
OK

127.0.0.1:6379> SLOWLOG GET 10
1) 1) (integer) 2          # Slow log entry ID
   2) (integer) 1704067200 # Unix timestamp when command was executed
   3) (integer) 15000      # Execution time in microseconds (15ms)
   4) 1) "KEYS"            # The slow command
      2) "user:*"          # Command arguments
   5) "127.0.0.1:54321"    # Client IP and port

2) 1) (integer) 1
   2) (integer) 1704067190
   3) (integer) 12000      # 12ms execution time
   4) 1) "HGETALL"
      2) "large_hash_key"
   5) "127.0.0.1:54322"
```

**SLOWLOG detailed explanation:**

- **Entry ID (2):** Unique identifier for this slow log entry
- **Timestamp (1704067200):** When the slow command was executed (Unix timestamp)
- **Execution time (15000):** How long the command took in **microseconds** (15,000 μs = 15ms)
- **Command and args:** The actual slow command that was executed
- **Client info:** Which client IP and port sent the command

**Understanding the timing:**

- 10,000 microseconds = 10 milliseconds
- Commands taking longer than this threshold are logged
- Typical fast commands take < 1ms
- Commands taking > 10ms might need optimization

**Common slow operations you'll see:**
```redis
# These operations commonly appear in SLOWLOG:
"KEYS" "*"              # Scanning all keys (avoid in production)
"HGETALL" "huge_hash"   # Getting large hash with many fields  
"LRANGE" "long_list" "0" "-1"  # Getting entire long list
"SMEMBERS" "large_set"  # Getting all members of large set
```

**How to use this information:**

1. **High frequency in MONITOR** = Hot key (accessed often)
2. **Appears in SLOWLOG** = Slow operation (takes too long)
3. **Both** = Hot key with slow operations (biggest problem!)

**Practical monitoring script:**
```bash
# Count most accessed keys in real-time
redis-cli monitor | grep -E "(GET|SET|HGET)" | \
awk '{print $4}' | sort | uniq -c | sort -nr | head -10

# This will show the top 10 most accessed keys
```

---

## Common Pitfalls and Solutions

### 1. Key Naming Inconsistencies

**Why this is a problem:** Mixed naming makes it hard to find keys and write maintainable code.

```redis
# Problem: Inconsistent naming
user:1001           # Some keys use colons
user_1002           # Others use underscores  
User-1003           # Mixed case and hyphens

# Solution: Establish and follow conventions
user:1001:profile
user:1002:profile  
user:1003:profile
```

### 2. Key Explosion

**Why this is a problem:** Too many similar keys waste memory and make Redis slower.

```redis
# Problem: Too many similar keys
user:1001:last_activity:2024-01-15:14:30:00
user:1001:last_activity:2024-01-15:14:31:00
user:1001:last_activity:2024-01-15:14:32:00

# Solution: Aggregate or use different data structures
user:1001:last_activity  # Single timestamp
# Or use a list/sorted set for multiple activities
LPUSH user:1001:activities "2024-01-15:14:30:00"
```

### 3. Missing Expiration

**Why this is a problem:** Cache keys without TTL grow forever and fill up memory.

```redis
# Problem: Cache keys without TTL
SET cache:user:profile "data"  # Never expires!

# Solution: Always set appropriate TTL
SETEX cache:user:profile 3600 "data"  # 1 hour expiration
```

### 4. Inefficient Key Patterns

**Why this is a problem:** Long keys use more memory and reduce Redis performance.

```redis
# Problem: Keys that don't compress well
SET user_profile_information_for_id_1001 "data"
SET user_profile_information_for_id_1002 "data"

# Solution: Shorter, consistent patterns
SET user:1001:profile "data"
SET user:1002:profile "data"
```

---

## Key Management Checklist

Before deploying to production, ensure:

- **Consistent naming convention** across all keys
- **Appropriate expiration** for cache and temporary data
- **Reasonable key length** (not too long, not too cryptic)
- **Namespace separation** between different features
- **No sensitive data** in key names
- **Documented key patterns** for team reference
- **Key cleanup strategy** for old/unused keys
- **Monitoring setup** for key metrics

---

## What's Next?

Now that you understand keys and values, let's dive into:

- How Redis manages memory
- Persistence options (RDB vs AOF)
- Memory optimization techniques
- Redis configuration essentials

