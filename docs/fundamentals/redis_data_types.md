# Redis Data Types

Redis isn't just a simple key-value store - it's a **data structure server** that supports multiple data types. Understanding these data types is crucial to using Redis effectively.

## Why Multiple Data Types Matter

Think of Redis data types like different containers:

- **String** = Single item box
- **List** = Ordered stack of items  
- **Set** = Bag of unique items
- **Hash** = Filing cabinet with labeled folders
- **Sorted Set** = Ranked leaderboard

Each container is optimized for different operations, making your applications faster and your code simpler.

## Overview of Redis Data Types

| Data Type | What It Stores | Best For | Real-World Example |
|-----------|----------------|----------|-------------------|
| **String** | Text, numbers, binary data | Caching, counters, flags | User names, page views |
| **List** | Ordered collection | Queues, timelines, logs | Shopping cart, activity feed |
| **Set** | Unique unordered items | Tags, categories | User interests, unique visitors |
| **Hash** | Field-value pairs | Objects, records | User profiles, product details |
| **Sorted Set** | Ranked unique items | Leaderboards, ranges | High scores, trending topics |

---

## 1. Strings - The Foundation

**Strings** are the most basic Redis data type. Despite the name, they can store:

- Text strings
- Numbers  
- Binary data (images, files)
- JSON objects
- Any data up to 512MB

### Basic String Operations

```redis
# Store simple text
127.0.0.1:6379> SET username "alice_cooper"
OK

# Store numbers (treated as strings)
127.0.0.1:6379> SET age "28"
OK

# Store JSON data
127.0.0.1:6379> SET user:1001 '{"name":"Alice","email":"alice@example.com"}'
OK

# Retrieve values
127.0.0.1:6379> GET username
"alice_cooper"

127.0.0.1:6379> GET age
"28"
```

### String as Counters

```redis
# Initialize counter
127.0.0.1:6379> SET page_views 1000
OK

# Increment by 1
127.0.0.1:6379> INCR page_views
(integer) 1001

# Increment by specific amount
127.0.0.1:6379> INCRBY page_views 50
(integer) 1051

# Decrement
127.0.0.1:6379> DECR page_views
(integer) 1050

# Decrement by specific amount
127.0.0.1:6379> DECRBY page_views 10
(integer) 1040
```

### Advanced String Operations

```redis
# Get string length
127.0.0.1:6379> STRLEN username
(integer) 12

# Get substring
127.0.0.1:6379> GETRANGE username 0 4
"alice"

# Append to string
127.0.0.1:6379> APPEND username "_admin"
(integer) 18 #returns the length after appending

127.0.0.1:6379> GET username
"alice_cooper_admin"

# Set with expiration (atomic operation)
127.0.0.1:6379> SETEX temp_token 300 "abc123xyz"
OK
```

!!! example "Try This: User Session Example"
    ```redis
    # Store user session with 1-hour expiration
    SET session:user123 "logged_in" EX 3600
    
    # Check if user is logged in
    GET session:user123
    
    # Check remaining session time
    TTL session:user123
    ```

---

## 2. Lists - Ordered Collections

**Lists** are ordered collections of strings. Think of them as arrays that you can push and pop from both ends.

### Basic List Operations

```redis
# Add items to the left (beginning)
127.0.0.1:6379> LPUSH shopping_cart "apples"
(integer) 1

127.0.0.1:6379> LPUSH shopping_cart "bananas" "oranges"
(integer) 3

# Add items to the right (end)
127.0.0.1:6379> RPUSH shopping_cart "milk"
(integer) 4

# View the entire list
127.0.0.1:6379> LRANGE shopping_cart 0 -1 #here 0 means first and -1 means end
1) "oranges"
2) "bananas" 
3) "apples"
4) "milk"
```

### List as Queue (FIFO - First In, First Out)

```redis
# Add tasks to queue
127.0.0.1:6379> RPUSH task_queue "send_email"
127.0.0.1:6379> RPUSH task_queue "process_payment"
127.0.0.1:6379> RPUSH task_queue "update_inventory"

# Process tasks (remove from left)
127.0.0.1:6379> LPOP task_queue
"send_email"

127.0.0.1:6379> LPOP task_queue
"process_payment"
```

### List as Stack (LIFO - Last In, First Out)

```redis
# Add items to stack
127.0.0.1:6379> LPUSH history_stack "page1"
127.0.0.1:6379> LPUSH history_stack "page2"
127.0.0.1:6379> LPUSH history_stack "page3"

# Remove from stack (most recent first)
127.0.0.1:6379> LPOP history_stack
"page3"
```

### Advanced List Operations

```redis
# Get list length
127.0.0.1:6379> LLEN shopping_cart
(integer) 2

# Get specific element by index
127.0.0.1:6379> LINDEX shopping_cart 0
"bananas"

# Set element at specific index
127.0.0.1:6379> LSET shopping_cart 0 "green_bananas"
OK

# Remove elements
127.0.0.1:6379> LREM shopping_cart 1 "milk"
(integer) 1

# Keep only elements in range
127.0.0.1:6379> LTRIM shopping_cart 0 2
OK
```

!!! example "Try This: Activity Feed Example"
    ```redis
    # Add user activities (most recent first)
    LPUSH user:1001:activity "liked a photo"
    LPUSH user:1001:activity "posted a comment"
    LPUSH user:1001:activity "shared an article"
    
    # Get recent activities (last 5)
    LRANGE user:1001:activity 0 4
    
    # Keep only last 100 activities
    LTRIM user:1001:activity 0 99
    ```

---

## 3. Sets - Unique Collections

**Sets** are unordered collections of unique strings. Perfect for storing tags, categories, or any data where uniqueness matters.

### Basic Set Operations

```redis
# Add members to set
127.0.0.1:6379> SADD user_interests "programming"
(integer) 1

127.0.0.1:6379> SADD user_interests "music" "travel" "cooking"
(integer) 3

# Try to add duplicate (won't be added)
127.0.0.1:6379> SADD user_interests "music"
(integer) 0

# View all members
127.0.0.1:6379> SMEMBERS user_interests
1) "travel"
2) "programming"
3) "music"
4) "cooking"
```

### Set Operations

```redis
# Check if member exists
127.0.0.1:6379> SISMEMBER user_interests "programming"
(integer) 1

# Get set size
127.0.0.1:6379> SCARD user_interests
(integer) 4

# Remove member
127.0.0.1:6379> SREM user_interests "travel"
(integer) 1

# Get random member
127.0.0.1:6379> SRANDMEMBER user_interests
"music"

# Pop (remove and return) random member
127.0.0.1:6379> SPOP user_interests
"cooking"
```

### Set Mathematics

```redis
# Create two sets
127.0.0.1:6379> SADD skills:alice "python" "javascript" "sql"
127.0.0.1:6379> SADD skills:bob "java" "python" "docker"

# Find intersection (common skills)
127.0.0.1:6379> SINTER skills:alice skills:bob
1) "python"

# Find union (all skills)
127.0.0.1:6379> SUNION skills:alice skills:bob
1) "sql"
2) "java"
3) "docker"
4) "javascript"
5) "python"

# Find difference (Alice's unique skills)
127.0.0.1:6379> SDIFF skills:alice skills:bob
1) "sql"
2) "javascript"
```

!!! example "Try This: User Tagging System"
    ```redis
    # Tag users
    SADD tags:user:1001 "premium" "beta_tester" "early_adopter"
    SADD tags:user:1002 "premium" "power_user"
    
    # Find premium users
    SINTER tags:user:1001 tags:user:1002
    
    # Check if user has specific tag
    SISMEMBER tags:user:1001 "premium"
    ```

---

## 4. Hashes - Key-Value Within Keys

**Hashes** are perfect for storing objects or records. Think of them as a mini key-value store within a Redis key.

### Basic Hash Operations

```redis
# Set hash fields
127.0.0.1:6379> HSET user:1001 name "John Doe"
(integer) 1

127.0.0.1:6379> HSET user:1001 email "john@example.com" age "30"
(integer) 2

# Get single field
127.0.0.1:6379> HGET user:1001 name
"John Doe"

# Get multiple fields
127.0.0.1:6379> HMGET user:1001 name email
1) "John Doe"
2) "john@example.com"

# Get all fields and values
127.0.0.1:6379> HGETALL user:1001
1) "name"
2) "John Doe"
3) "email"
4) "john@example.com"
5) "age"
6) "30"
```

### Advanced Hash Operations

```redis
# Check if field exists
127.0.0.1:6379> HEXISTS user:1001 name
(integer) 1

# Get all field names
127.0.0.1:6379> HKEYS user:1001
1) "name"
2) "email"
3) "age"

# Get all values
127.0.0.1:6379> HVALS user:1001
1) "John Doe"
2) "john@example.com"
3) "30"

# Get number of fields
127.0.0.1:6379> HLEN user:1001
(integer) 3

# Delete field
127.0.0.1:6379> HDEL user:1001 age
(integer) 1
```

### Hash Counters

```redis
# Increment numeric field
127.0.0.1:6379> HINCRBY user:1001 login_count 1
(integer) 1

127.0.0.1:6379> HINCRBY user:1001 login_count 5
(integer) 6

# Increment by float
127.0.0.1:6379> HINCRBYFLOAT user:1001 account_balance 25.50
"25.5"
```

!!! example "Try This: Product Catalog"
    ```redis
    # Store product information
    HSET product:123 name "Wireless Headphones"
    HSET product:123 price "99.99" stock "50" category "electronics"
    
    # Update stock after purchase
    HINCRBY product:123 stock -1
    
    # Get product details
    HGETALL product:123
    ```

---

## 5. Sorted Sets - Ranked Collections

**Sorted Sets** combine the uniqueness of sets with the ability to rank items by score. Perfect for leaderboards, rankings, and time-based data.

### Basic Sorted Set Operations

```redis
# Add members with scores
127.0.0.1:6379> ZADD leaderboard 100 "alice"
(integer) 1

127.0.0.1:6379> ZADD leaderboard 85 "bob" 92 "charlie"
(integer) 2

# Add member with same name (updates score)
127.0.0.1:6379> ZADD leaderboard 95 "alice"
(integer) 0

# View by rank (lowest to highest)
127.0.0.1:6379> ZRANGE leaderboard 0 -1
1) "bob"
2) "charlie"
3) "alice"

# View by rank with scores
127.0.0.1:6379> ZRANGE leaderboard 0 -1 WITHSCORES
1) "bob"
2) "85"
3) "charlie"
4) "92"
5) "alice"
6) "95"
```

### Sorted Set Queries

```redis
# Get by rank (highest to lowest)
127.0.0.1:6379> ZREVRANGE leaderboard 0 -1 WITHSCORES
1) "alice"
2) "95"
3) "charlie"
4) "92"
5) "bob"
6) "85"

# Get top 2 players
127.0.0.1:6379> ZREVRANGE leaderboard 0 1
1) "alice"
2) "charlie"

# Get score for specific member
127.0.0.1:6379> ZSCORE leaderboard "alice"
"95"

# Get rank of member (0-based, lowest score = rank 0)
127.0.0.1:6379> ZRANK leaderboard "alice"
(integer) 2

# Get reverse rank (highest score = rank 0)
127.0.0.1:6379> ZREVRANK leaderboard "alice"
(integer) 0
```

### Score-based Queries

```redis
# Get members by score range
127.0.0.1:6379> ZRANGEBYSCORE leaderboard 90 100
1) "charlie"
2) "alice"

# Count members in score range
127.0.0.1:6379> ZCOUNT leaderboard 90 100
(integer) 2

# Remove member
127.0.0.1:6379> ZREM leaderboard "bob"
(integer) 1

# Get set size
127.0.0.1:6379> ZCARD leaderboard
(integer) 2
```

### Advanced Sorted Set Operations

```redis
# Increment score
127.0.0.1:6379> ZINCRBY leaderboard 10 "charlie"
"102"

# Remove by rank range (remove bottom 2)
127.0.0.1:6379> ZREMRANGEBYRANK leaderboard 0 1

# Remove by score range
127.0.0.1:6379> ZREMRANGEBYSCORE leaderboard 0 50
```

!!! example "Try This: Social Media Trending"
    ```redis
    # Track trending topics with engagement scores
    ZADD trending 1500 "artificial_intelligence"
    ZADD trending 2300 "climate_change"
    ZADD trending 1800 "space_exploration"
    
    # Get top trending topics
    ZREVRANGE trending 0 2 WITHSCORES
    
    # Increase engagement
    ZINCRBY trending 200 "artificial_intelligence"
    ```

---

## Choosing the Right Data Type

### Decision Matrix

| Use Case | Best Data Type | Why |
|----------|----------------|-----|
| **User session data** | String | Simple key-value with expiration |
| **Shopping cart items** | List | Ordered, can add/remove from both ends |
| **User tags/interests** | Set | Unique items, set operations |
| **User profile** | Hash | Multiple attributes for one entity |
| **Game leaderboard** | Sorted Set | Ranked by score |
| **Activity feed** | List | Chronological order matters |
| **Online users** | Set | Unique users, easy add/remove |
| **Product catalog** | Hash | Multiple properties per product |
| **Trending topics** | Sorted Set | Ranked by popularity score |

### Common Patterns

```redis
# Pattern 1: Namespaced keys
user:1001:profile          # Hash for user profile
user:1001:sessions         # Set for active sessions
user:1001:activity         # List for activity history

# Pattern 2: Time-based data
posts:2024:01:15           # Lists for daily posts
views:2024:01             # Sorted set for monthly views

# Pattern 3: Categorical data
tags:programming          # Set of users interested in programming
categories:electronics    # Sorted set of products by popularity
```

---

## Hands-on Practice

### Exercise 1: E-commerce Cart System

```redis
# User adds items to cart (List)
LPUSH cart:user123 "product:456"
LPUSH cart:user123 "product:789"

# Store product details (Hash)
HSET product:456 name "Laptop" price "999.99" stock "5"
HSET product:789 name "Mouse" price "25.99" stock "20"

# Track user's interests (Set)
SADD interests:user123 "electronics" "computers" "gaming"

# Update product popularity (Sorted Set)
ZINCRBY popular_products 1 "product:456"
```

### Exercise 2: Social Media Features

```redis
# User profile (Hash)
HSET profile:alice name "Alice" followers "1500" posts "342"

# Following relationship (Set)
SADD following:alice "bob" "charlie" "david"
SADD followers:alice "bob" "eve" "frank"

# Posts timeline (List)
LPUSH timeline:alice "Just had an amazing coffee!"
LPUSH timeline:alice "Working on a new project"

# Trending hashtags (Sorted Set)
ZADD trending_hashtags 150 "coffee" 89 "work" 234 "weekend"
```

---

## Data Types Mastery!

You now understand:

- **Strings** - For simple values, counters, and caching
- **Lists** - For ordered data, queues, and stacks  
- **Sets** - For unique collections and set operations
- **Hashes** - For structured data and objects
- **Sorted Sets** - For ranked data and leaderboards

## What's Next?

Now that you understand Redis data types, let's learn about:

- Key naming conventions and patterns
- How Redis manages keys and values
- Memory optimization techniques
- Key expiration strategies
