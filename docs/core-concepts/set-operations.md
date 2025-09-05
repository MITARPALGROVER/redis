# Set Operations

Redis sets are like bags of unique items - no duplicates allowed! Perfect for tags, followers, permissions, and anything where you need to track unique values or find relationships between groups.

## What Are Redis Sets?

Think of Redis sets as:

- A **bag of unique marbles** (no two identical ones)
- **Tags on a blog post** (each tag appears only once)
- **People in a room** (each person can only be there once)
- **Skills on a resume** (you either have the skill or you don't)

Sets automatically handle uniqueness for you - try to add the same item twice, and Redis just ignores the duplicate!

---

## Basic Set Operations

### Adding Items to Sets

```redis
# Add items to a set
127.0.0.1:6379> SADD colors "red"
(integer) 1
# Returns 1 = new item added

127.0.0.1:6379> SADD colors "blue" "green"
(integer) 2
# Added 2 new items

# Try to add duplicate
127.0.0.1:6379> SADD colors "red"
(integer) 0
# Returns 0 = item already exists, nothing added

127.0.0.1:6379> SADD colors "yellow" "red" "purple"
(integer) 2
# Only "yellow" and "purple" added, "red" was ignored

# See what's in the set
127.0.0.1:6379> SMEMBERS colors
1) "blue"
2) "green"
3) "purple"
4) "red"
5) "yellow"
# Order may vary - sets are unordered!
```

### Checking Set Contents

```redis
# Check if item exists in set
127.0.0.1:6379> SISMEMBER colors "red"
(integer) 1
# 1 = yes, "red" is in the set

127.0.0.1:6379> SISMEMBER colors "orange"
(integer) 0
# 0 = no, "orange" is not in the set

# Get number of items in set
127.0.0.1:6379> SCARD colors
(integer) 5
# SCARD = Set CARDinality (size)

# Check multiple items at once (Redis 6.2+)
127.0.0.1:6379> SMISMEMBER colors "red" "orange" "blue"
1) (integer) 1
2) (integer) 0
3) (integer) 1
# red=yes, orange=no, blue=yes
```

### Removing Items from Sets

```redis
# Remove specific items
127.0.0.1:6379> SREM colors "green"
(integer) 1
# 1 = item was removed

127.0.0.1:6379> SREM colors "orange"
(integer) 0
# 0 = item wasn't in set

127.0.0.1:6379> SREM colors "red" "blue"
(integer) 2
# Removed 2 items

127.0.0.1:6379> SMEMBERS colors
1) "purple"
2) "yellow"

# Get random item(s) without removing
127.0.0.1:6379> SRANDMEMBER colors
"yellow"
# Returns random item (could be "purple" next time)

127.0.0.1:6379> SRANDMEMBER colors 2
1) "purple"
2) "yellow"
# Get 2 random items

# Remove and return random item
127.0.0.1:6379> SPOP colors
"purple"

127.0.0.1:6379> SMEMBERS colors
1) "yellow"
# "purple" is gone

# Remove multiple random items
127.0.0.1:6379> SADD numbers 1 2 3 4 5
(integer) 5

127.0.0.1:6379> SPOP numbers 3
1) "2"
2) "4" 
3) "1"
# Removed 3 random items

127.0.0.1:6379> SMEMBERS numbers
1) "3"
2) "5"
```

---

## Set Mathematics (The Cool Stuff!)

This is where sets become really powerful - you can combine them in mathematical ways.

### Union (Combine Sets)

```redis
# Create some sets
127.0.0.1:6379> SADD fruits "apple" "banana" "orange"
(integer) 3

127.0.0.1:6379> SADD vegetables "carrot" "broccoli" "spinach"
(integer) 3

127.0.0.1:6379> SADD healthy_foods "apple" "carrot" "salmon"
(integer) 3

# Union: Get all unique items from multiple sets
127.0.0.1:6379> SUNION fruits vegetables
1) "apple"
2) "banana"
3) "broccoli"
4) "carrot"
5) "orange"
6) "spinach"
# All items from both sets, no duplicates

# Union with 3 sets
127.0.0.1:6379> SUNION fruits vegetables healthy_foods
1) "apple"
2) "banana"
3) "broccoli"
4) "carrot"
5) "orange"
6) "salmon"
7) "spinach"

# Store union result in new set
127.0.0.1:6379> SUNIONSTORE all_foods fruits vegetables healthy_foods
(integer) 7

127.0.0.1:6379> SMEMBERS all_foods
1) "apple"
2) "banana"
3) "broccoli"
4) "carrot"
5) "orange"
6) "salmon"
7) "spinach"
```

### Intersection (Find Common Items)

```redis
# Find items that exist in ALL sets
127.0.0.1:6379> SINTER fruits healthy_foods
1) "apple"
# Only "apple" exists in both sets

127.0.0.1:6379> SINTER vegetables healthy_foods
1) "carrot"
# Only "carrot" exists in both sets

# Intersection with no common items
127.0.0.1:6379> SINTER fruits vegetables
(empty list or set)
# No items exist in both sets

# Store intersection result
127.0.0.1:6379> SINTERSTORE common_healthy fruits vegetables healthy_foods
(integer) 0
# No items exist in all 3 sets

# More practical example
127.0.0.1:6379> SADD alice_skills "python" "javascript" "sql" "docker"
(integer) 4

127.0.0.1:6379> SADD bob_skills "python" "java" "sql" "kubernetes"
(integer) 4

127.0.0.1:6379> SINTER alice_skills bob_skills
1) "python"
2) "sql"
# Skills both Alice and Bob have
```

### Difference (Find What's Unique)

```redis
# Find items in first set but NOT in second set
127.0.0.1:6379> SDIFF alice_skills bob_skills
1) "docker"
2) "javascript"
# Skills Alice has that Bob doesn't

127.0.0.1:6379> SDIFF bob_skills alice_skills
1) "java"
2) "kubernetes"
# Skills Bob has that Alice doesn't

# Store difference result
127.0.0.1:6379> SDIFFSTORE alice_unique alice_skills bob_skills
(integer) 2

127.0.0.1:6379> SMEMBERS alice_unique
1) "docker"
2) "javascript"

# Practical example: Find new followers
127.0.0.1:6379> SADD followers_yesterday "user1" "user2" "user3"
(integer) 3

127.0.0.1:6379> SADD followers_today "user1" "user2" "user3" "user4" "user5"
(integer) 5

127.0.0.1:6379> SDIFF followers_today followers_yesterday
1) "user4"
2) "user5"
# New followers today

127.0.0.1:6379> SDIFF followers_yesterday followers_today
(empty list or set)
# Nobody unfollowed (all yesterday's followers still there)
```

---

## Real-World Set Use Cases

### 1. User Tags and Categories

```redis
# Tag users with interests
127.0.0.1:6379> SADD tags:programming "user123" "user456" "user789"
(integer) 3

127.0.0.1:6379> SADD tags:music "user123" "user999" "user111"
(integer) 3

127.0.0.1:6379> SADD tags:sports "user456" "user999" "user222"
(integer) 3

# Find users interested in both programming AND music
127.0.0.1:6379> SINTER tags:programming tags:music
1) "user123"

# Find users interested in programming OR music (union)
127.0.0.1:6379> SUNION tags:programming tags:music
1) "user111"
2) "user123"
3) "user456"
4) "user789"
5) "user999"

# Find users interested in programming but NOT music
127.0.0.1:6379> SDIFF tags:programming tags:music
1) "user456"
2) "user789"

# Add user to tag
127.0.0.1:6379> SADD tags:programming "user333"
(integer) 1

# Remove user from tag
127.0.0.1:6379> SREM tags:sports "user456"
(integer) 1

# Check if user has specific tag
127.0.0.1:6379> SISMEMBER tags:programming "user123"
(integer) 1
```

### 2. Following/Followers System

```redis
# Alice follows some people
127.0.0.1:6379> SADD following:alice "bob" "charlie" "diana"
(integer) 3

# Bob follows some people
127.0.0.1:6379> SADD following:bob "alice" "charlie" "eve"
(integer) 3

# Find mutual follows (people both Alice and Bob follow)
127.0.0.1:6379> SINTER following:alice following:bob
1) "charlie"

# People Alice follows but Bob doesn't
127.0.0.1:6379> SDIFF following:alice following:bob
1) "bob"
2) "diana"

# Get suggested follows for Alice (people Bob follows but Alice doesn't)
127.0.0.1:6379> SDIFF following:bob following:alice
1) "eve"
# Alice might want to follow Eve

# Check if Alice follows Bob
127.0.0.1:6379> SISMEMBER following:alice "bob"
(integer) 1

# Alice unfollows Bob
127.0.0.1:6379> SREM following:alice "bob"
(integer) 1

# Count how many people Alice follows
127.0.0.1:6379> SCARD following:alice
(integer) 2
```

### 3. Product Features and Filtering

```redis
# Products with different features
127.0.0.1:6379> SADD features:waterproof "phone1" "watch1" "camera1"
(integer) 3

127.0.0.1:6379> SADD features:wireless "phone1" "headphones1" "speaker1"
(integer) 3

127.0.0.1:6379> SADD features:portable "phone1" "watch1" "headphones1"
(integer) 3

# Find products that are waterproof AND wireless
127.0.0.1:6379> SINTER features:waterproof features:wireless
1) "phone1"

# Find products that are waterproof AND wireless AND portable
127.0.0.1:6379> SINTER features:waterproof features:wireless features:portable
1) "phone1"

# Find all waterproof or wireless products
127.0.0.1:6379> SUNION features:waterproof features:wireless
1) "camera1"
2) "headphones1"
3) "phone1"
4) "speaker1"
5) "watch1"

# Find waterproof products that are NOT wireless
127.0.0.1:6379> SDIFF features:waterproof features:wireless
1) "camera1"
2) "watch1"
```

### 4. Access Control and Permissions

```redis
# Define permissions for different roles
127.0.0.1:6379> SADD permissions:admin "read" "write" "delete" "manage_users"
(integer) 4

127.0.0.1:6379> SADD permissions:editor "read" "write" "edit_content"
(integer) 3

127.0.0.1:6379> SADD permissions:viewer "read"
(integer) 1

# Assign roles to users
127.0.0.1:6379> SADD user:alice:roles "admin"
(integer) 1

127.0.0.1:6379> SADD user:bob:roles "editor" "viewer"
(integer) 2

# Check if user has specific permission
def has_permission(user, permission):
    roles = redis.smembers(f"user:{user}:roles")
    for role in roles:
        if redis.sismember(f"permissions:{role}", permission):
            return True
    return False

# Find common permissions between admin and editor
127.0.0.1:6379> SINTER permissions:admin permissions:editor
1) "read"
2) "write"

# Find permissions only admin has
127.0.0.1:6379> SDIFF permissions:admin permissions:editor
1) "delete"
2) "manage_users"
```

### 5. Online/Offline User Tracking

```redis
# Track online users
127.0.0.1:6379> SADD users:online "alice" "bob" "charlie"
(integer) 3

# User goes offline
127.0.0.1:6379> SREM users:online "bob"
(integer) 1

# User comes online
127.0.0.1:6379> SADD users:online "diana"
(integer) 1

# Check if user is online
127.0.0.1:6379> SISMEMBER users:online "alice"
(integer) 1

# Get all online users
127.0.0.1:6379> SMEMBERS users:online
1) "alice"
2) "charlie"
3) "diana"

# Count online users
127.0.0.1:6379> SCARD users:online
(integer) 3

# Track users in specific room
127.0.0.1:6379> SADD room:general:users "alice" "charlie"
(integer) 2

127.0.0.1:6379> SADD room:tech:users "alice" "diana"
(integer) 2

# Find users in both rooms
127.0.0.1:6379> SINTER room:general:users room:tech:users
1) "alice"

# Get random online user
127.0.0.1:6379> SRANDMEMBER users:online
"charlie"
```

---

## Advanced Set Patterns

### 1. Set Expiration for Temporary Memberships

```redis
# Create temporary VIP membership (expires in 1 hour)
127.0.0.1:6379> SADD vip:members "user123"
(integer) 1

127.0.0.1:6379> EXPIRE vip:members 3600
(integer) 1

# Check VIP status
127.0.0.1:6379> SISMEMBER vip:members "user123"
(integer) 1

# After 1 hour, the entire set expires
127.0.0.1:6379> EXISTS vip:members
(integer) 0
```

### 2. Set Size Monitoring

```redis
# Monitor if set gets too large
127.0.0.1:6379> SADD active_sessions "session1" "session2" "session3"
(integer) 3

127.0.0.1:6379> SCARD active_sessions
(integer) 3

# In application code:
def add_session_safely(session_id):
    if redis.scard("active_sessions") >= 1000:
        # Remove oldest or random session
        redis.spop("active_sessions")
    redis.sadd("active_sessions", session_id)
```

### 3. Set-Based Caching

```redis
# Cache search results as sets
127.0.0.1:6379> SADD search:python "result1" "result2" "result3"
(integer) 3

127.0.0.1:6379> EXPIRE search:python 300
(integer) 1
# Cache expires in 5 minutes

# Check if result is in cached search
127.0.0.1:6379> SISMEMBER search:python "result2"
(integer) 1
```

---

## Performance Tips

### 1. Set Operations Speed

```redis
# Fast operations (O(1))
SADD myset "item"     # Add item
SREM myset "item"     # Remove item  
SISMEMBER myset "item" # Check membership
SCARD myset           # Get size

# Slower operations - use carefully with large sets
SMEMBERS myset        # Get all members (O(N))
SRANDMEMBER myset 100 # Get many random (O(N))
```

### 2. Memory Optimization

```redis
# Good: Store user IDs as numbers
SADD followers:user123 1001 1002 1003

# Less efficient: Store as strings with prefixes
SADD followers:user123 "user:1001" "user:1002" "user:1003"

# Good: Use short, meaningful keys
SADD tags:prog "user123"     # "prog" for programming

# Wasteful: Long descriptive keys
SADD tags:programming_languages_enthusiasts "user123"
```

### 3. Bulk Operations

```redis
# Add multiple items at once
SADD interests "music" "sports" "coding" "travel"

# Multiple separate commands
SADD interests "music"
SADD interests "sports"
SADD interests "coding"
SADD interests "travel"
```

---

## Practice Exercises

### Exercise 1: Blog Post Tags

```redis
# Create posts with tags
SADD post:1:tags "redis" "database" "tutorial"
SADD post:2:tags "python" "programming" "tutorial"
SADD post:3:tags "redis" "python" "advanced"

# Find posts tagged with both "redis" and "python"
SINTER post:1:tags post:3:tags

# Find all unique tags
SUNION post:1:tags post:2:tags post:3:tags

# Find tags unique to post 1
SDIFF post:1:tags post:2:tags post:3:tags
```

### Exercise 2: Friend Recommendations

```redis
# Alice's friends
SADD friends:alice "bob" "charlie" "diana"

# Bob's friends  
SADD friends:bob "alice" "charlie" "eve" "frank"

# Find mutual friends
SINTER friends:alice friends:bob

# Find potential friends for Alice (Bob's friends who aren't Alice's friends, excluding Alice herself)
SDIFF friends:bob friends:alice
SREM temp:suggestions "alice"
```

### Exercise 3: Skill Matching

```redis
# Job requirements
SADD job:123:skills "python" "sql" "docker" "git"

# Candidate skills
SADD candidate:456:skills "python" "javascript" "sql" "git" "aws"

# Check if candidate has all required skills
SINTER job:123:skills candidate:456:skills
SCARD temp:match
# Compare with SCARD job:123:skills

# Find missing skills
SDIFF job:123:skills candidate:456:skills

# Find extra skills candidate has
SDIFF candidate:456:skills job:123:skills
```

### Exercise 4: Event Attendance

```redis
# Track who's attending events
SADD event:conf2024:attendees "alice" "bob" "charlie"
SADD event:workshop:attendees "bob" "charlie" "diana"

# Find people attending both events
SINTER event:conf2024:attendees event:workshop:attendees

# Find total unique attendees across all events
SUNION event:conf2024:attendees event:workshop:attendees

# Find people only attending the conference
SDIFF event:conf2024:attendees event:workshop:attendees
```

---

## Set Commands Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `SADD key member` | Add to set | `SADD tags "redis"` |
| `SREM key member` | Remove from set | `SREM tags "old"` |
| `SMEMBERS key` | Get all members | `SMEMBERS colors` |
| `SISMEMBER key member` | Check membership | `SISMEMBER tags "redis"` |
| `SCARD key` | Get set size | `SCARD followers` |
| `SRANDMEMBER key [count]` | Get random member(s) | `SRANDMEMBER users 3` |
| `SPOP key [count]` | Remove random member(s) | `SPOP queue 2` |
| `SUNION key1 key2` | Combine sets | `SUNION set1 set2` |
| `SINTER key1 key2` | Find common items | `SINTER skills1 skills2` |
| `SDIFF key1 key2` | Find differences | `SDIFF all new` |
| `SUNIONSTORE dest key1 key2` | Store union result | `SUNIONSTORE result set1 set2` |
| `SINTERSTORE dest key1 key2` | Store intersection result | `SINTERSTORE common set1 set2` |
| `SDIFFSTORE dest key1 key2` | Store difference result | `SDIFFSTORE unique set1 set2` |

---

## What's Next?

Ready to learn about **Hashes** - structured objects perfect for user profiles, product data, and configuration!
