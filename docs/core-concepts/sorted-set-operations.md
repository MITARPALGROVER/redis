# Sorted Set Operations

Redis sorted sets are collections where each member has a score. Members are automatically sorted by their scores, making them perfect for leaderboards, rankings, time-series data, and priority queues.

## What Are Redis Sorted Sets?

Think of sorted sets as:

- A **leaderboard** where players are ranked by score
- A **priority queue** where tasks have importance levels
- A **timeline** where events are ordered by timestamp
- A **ranking system** where items are ordered by rating

Unlike regular sets, every member has a score, and Redis keeps them sorted automatically.

---

## Basic Sorted Set Operations

### Adding Members with Scores

```redis
# Add members with scores
127.0.0.1:6379> ZADD leaderboard 1500 "alice"
(integer) 1
# score=1500, member="alice"

127.0.0.1:6379> ZADD leaderboard 1200 "bob" 1800 "charlie"
(integer) 2
# Added 2 members

# Add member that already exists (updates score)
127.0.0.1:6379> ZADD leaderboard 1600 "alice"
(integer) 0
# 0 = member existed, score updated from 1500 to 1600

# View sorted set (lowest to highest score)
127.0.0.1:6379> ZRANGE leaderboard 0 -1
1) "bob"
2) "alice"
3) "charlie"
# Sorted by score: 1200, 1600, 1800

# View with scores
127.0.0.1:6379> ZRANGE leaderboard 0 -1 WITHSCORES
1) "bob"
2) "1200"
3) "alice"
4) "1600"
5) "charlie"
6) "1800"
```

### Getting Members and Scores

```redis
# Get members in order (lowest to highest score)
127.0.0.1:6379> ZRANGE leaderboard 0 -1
1) "bob"
2) "alice"
3) "charlie"

# Get members in reverse order (highest to lowest score)
127.0.0.1:6379> ZREVRANGE leaderboard 0 -1
1) "charlie"
2) "alice"
3) "bob"

# Get top 2 players
127.0.0.1:6379> ZREVRANGE leaderboard 0 1
1) "charlie"
2) "alice"

# Get specific member's score
127.0.0.1:6379> ZSCORE leaderboard "alice"
"1600"

# Get member's rank (position from lowest score)
127.0.0.1:6379> ZRANK leaderboard "alice"
(integer) 1
# Position 1 (0-based indexing)

# Get member's rank (position from highest score)
127.0.0.1:6379> ZREVRANK leaderboard "alice"
(integer) 1
# Position 1 from the top

# Count total members
127.0.0.1:6379> ZCARD leaderboard
(integer) 3
```

### Removing Members

```redis
# Remove specific members
127.0.0.1:6379> ZREM leaderboard "bob"
(integer) 1
# 1 = member removed

127.0.0.1:6379> ZRANGE leaderboard 0 -1
1) "alice"
2) "charlie"

# Remove members by rank (position)
127.0.0.1:6379> ZADD numbers 1 "one" 2 "two" 3 "three" 4 "four" 5 "five"
(integer) 5

127.0.0.1:6379> ZREMRANGEBYRANK numbers 0 1
(integer) 2
# Remove first 2 members (ranks 0 and 1)

127.0.0.1:6379> ZRANGE numbers 0 -1
1) "three"
2) "four"
3) "five"

# Remove members by score range
127.0.0.1:6379> ZADD scores 10 "low" 50 "medium" 90 "high" 95 "very_high"
(integer) 4

127.0.0.1:6379> ZREMRANGEBYSCORE scores 40 60
(integer) 1
# Remove members with scores between 40 and 60

127.0.0.1:6379> ZRANGE scores 0 -1 WITHSCORES
1) "low"
2) "10"
3) "high"
4) "90"
5) "very_high"
6) "95"
```

---

## Score-Based Queries

### Finding Members by Score Range

```redis
# Add test data
127.0.0.1:6379> ZADD prices 19.99 "book" 299.99 "tablet" 999.99 "laptop" 49.99 "headphones"
(integer) 4

# Get items in price range $20-$300
127.0.0.1:6379> ZRANGEBYSCORE prices 20 300
1) "headphones"
2) "tablet"

# Get items with scores included
127.0.0.1:6379> ZRANGEBYSCORE prices 20 300 WITHSCORES
1) "headphones"
2) "49.99"
3) "tablet"
4) "299.99"

# Get items above $100 (reverse order)
127.0.0.1:6379> ZREVRANGEBYSCORE prices +inf 100
1) "laptop"
2) "tablet"

# Count items in price range
127.0.0.1:6379> ZCOUNT prices 20 300
(integer) 2

# Get items with limit
127.0.0.1:6379> ZRANGEBYSCORE prices 0 +inf LIMIT 0 2
1) "book"
2) "headphones"
# First 2 items (offset 0, count 2)

127.0.0.1:6379> ZRANGEBYSCORE prices 0 +inf LIMIT 2 2
1) "tablet"
2) "laptop"
# Next 2 items (offset 2, count 2)
```

### Score Ranges with Exclusive Bounds

```redis
# Exclusive ranges using ( )
127.0.0.1:6379> ZRANGEBYSCORE prices (20 (300
1) "headphones"
# Greater than 20 AND less than 300 (excludes 20 and 300)

# Mixed inclusive/exclusive
127.0.0.1:6379> ZRANGEBYSCORE prices 20 (300
1) "headphones"
# Greater than or equal to 20 AND less than 300

# Using infinity
127.0.0.1:6379> ZRANGEBYSCORE prices 100 +inf
1) "tablet"
2) "laptop"
# All items with score >= 100

127.0.0.1:6379> ZRANGEBYSCORE prices -inf 50
1) "book"
2) "headphones"
# All items with score <= 50
```

---

## Incrementing Scores

```redis
# Increment member's score
127.0.0.1:6379> ZADD game_scores 100 "player1" 150 "player2"
(integer) 2

127.0.0.1:6379> ZINCRBY game_scores 25 "player1"
"125"
# player1's score increased from 100 to 125

# Increment by negative amount (decrease)
127.0.0.1:6379> ZINCRBY game_scores -10 "player2"
"140"
# player2's score decreased from 150 to 140

# Increment non-existent member (starts at 0)
127.0.0.1:6379> ZINCRBY game_scores 75 "player3"
"75"
# player3 added with score 75

127.0.0.1:6379> ZRANGE game_scores 0 -1 WITHSCORES
1) "player3"
2) "75"
3) "player1"
4) "125"
5) "player2"
6) "140"
```

---

## Real-World Sorted Set Use Cases

### 1. Game Leaderboard

```redis
# Initialize player scores
127.0.0.1:6379> ZADD game:leaderboard 1500 "alice" 1200 "bob" 1800 "charlie" 900 "diana"
(integer) 4

# Player completes level and gains points
127.0.0.1:6379> ZINCRBY game:leaderboard 300 "alice"
"1800"

127.0.0.1:6379> ZINCRBY game:leaderboard 150 "bob"
"1350"

# Get top 3 players
127.0.0.1:6379> ZREVRANGE game:leaderboard 0 2 WITHSCORES
1) "alice"
2) "1800"
3) "charlie"
4) "1800"
5) "bob"
6) "1350"

# Get player's current rank
127.0.0.1:6379> ZREVRANK game:leaderboard "bob"
(integer) 2
# Bob is in 3rd place (0-based)

# Get players around a specific rank
127.0.0.1:6379> ZREVRANGE game:leaderboard 1 3
1) "charlie"
2) "bob"
3) "diana"

# Find players with score above 1000
127.0.0.1:6379> ZRANGEBYSCORE game:leaderboard 1000 +inf
1) "bob"
2) "alice"
3) "charlie"

# Get total number of players
127.0.0.1:6379> ZCARD game:leaderboard
(integer) 4
```

### 2. Priority Task Queue

```redis
# Add tasks with priority scores (higher = more urgent)
127.0.0.1:6379> ZADD task:queue 1 "send_email" 5 "backup_database" 3 "update_cache" 10 "fix_critical_bug"
(integer) 4

# Get highest priority task
127.0.0.1:6379> ZREVRANGE task:queue 0 0
1) "fix_critical_bug"

# Process task (remove it)
127.0.0.1:6379> ZREM task:queue "fix_critical_bug"
(integer) 1

# Add urgent task
127.0.0.1:6379> ZADD task:queue 8 "security_patch"
(integer) 1

# Get next 3 tasks to process
127.0.0.1:6379> ZREVRANGE task:queue 0 2 WITHSCORES
1) "security_patch"
2) "8"
3) "backup_database"
4) "5"
5) "update_cache"
6) "3"

# Increase task priority
127.0.0.1:6379> ZINCRBY task:queue 2 "send_email"
"3"

# Get all tasks by priority
127.0.0.1:6379> ZREVRANGE task:queue 0 -1 WITHSCORES
1) "security_patch"
2) "8"
3) "backup_database"
4) "5"
5) "update_cache"
6) "3"
7) "send_email"
8) "3"
```

### 3. Time-Series Data (Using Timestamps)

```redis
# Store events with timestamps as scores
127.0.0.1:6379> ZADD user:activity 1705751400 "login" 1705751500 "view_profile" 1705751600 "post_message"
(integer) 3

# Add more recent activity
127.0.0.1:6379> ZADD user:activity 1705751700 "like_post" 1705751800 "logout"
(integer) 2

# Get all activity in chronological order
127.0.0.1:6379> ZRANGE user:activity 0 -1 WITHSCORES
1) "login"
2) "1705751400"
3) "view_profile"
4) "1705751500"
5) "post_message"
6) "1705751600"
7) "like_post"
8) "1705751700"
9) "logout"
10) "1705751800"

# Get recent activity (last 3)
127.0.0.1:6379> ZREVRANGE user:activity 0 2
1) "logout"
2) "like_post"
3) "post_message"

# Get activity in time range (last 200 seconds)
127.0.0.1:6379> ZRANGEBYSCORE user:activity 1705751600 1705751800
1) "post_message"
2) "like_post"
3) "logout"

# Remove old activity (keep only recent)
127.0.0.1:6379> ZREMRANGEBYSCORE user:activity -inf 1705751500
(integer) 2
# Remove activity older than timestamp 1705751500

127.0.0.1:6379> ZRANGE user:activity 0 -1
1) "post_message"
2) "like_post"
3) "logout"
```

### 4. Product Ratings and Reviews

```redis
# Products with average ratings
127.0.0.1:6379> ZADD products:by_rating 4.2 "laptop" 4.8 "headphones" 3.9 "tablet" 4.5 "phone"
(integer) 4

# Get top-rated products
127.0.0.1:6379> ZREVRANGE products:by_rating 0 2 WITHSCORES
1) "headphones"
2) "4.8"
3) "phone"
4) "4.5"
5) "laptop"
6) "4.2"

# Get products with rating above 4.0
127.0.0.1:6379> ZRANGEBYSCORE products:by_rating 4.0 +inf
1) "laptop"
2) "phone"
3) "headphones"

# Update product rating (new reviews came in)
127.0.0.1:6379> ZADD products:by_rating 4.3 "laptop"
(integer) 0
# Updated existing score

# Get product's current rating
127.0.0.1:6379> ZSCORE products:by_rating "laptop"
"4.3"

# Find product's rank in ratings
127.0.0.1:6379> ZREVRANK products:by_rating "laptop"
(integer) 2
# 3rd best rated product
```

### 5. Recent Items with Timestamps

```redis
# Recent viewed articles (timestamp as score)
127.0.0.1:6379> ZADD recent:articles 1705751400 "article:123" 1705751500 "article:456"
(integer) 2

# User views an article again (update timestamp)
127.0.0.1:6379> ZADD recent:articles 1705751800 "article:123"
(integer) 0
# Moved to most recent position

# Add new article
127.0.0.1:6379> ZADD recent:articles 1705751900 "article:789"
(integer) 1

# Get recent articles (newest first)
127.0.0.1:6379> ZREVRANGE recent:articles 0 -1
1) "article:789"
2) "article:123"
3) "article:456"

# Keep only 10 most recent articles
127.0.0.1:6379> ZREMRANGEBYRANK recent:articles 0 -11
(integer) 0
# Remove all but last 10 (would remove items if we had more than 10)

# Alternative: Remove items older than 1 hour
127.0.0.1:6379> ZREMRANGEBYSCORE recent:articles -inf 1705747800
(integer) 0
# Remove items with timestamp older than 1 hour ago
```

---

## Advanced Sorted Set Operations

### 1. Set Operations with Sorted Sets

```redis
# Create two sorted sets
127.0.0.1:6379> ZADD skills:alice 8 "python" 6 "javascript" 9 "sql" 7 "docker"
(integer) 4

127.0.0.1:6379> ZADD skills:bob 7 "python" 8 "java" 9 "sql" 5 "kubernetes"
(integer) 4

# Union with sum of scores
127.0.0.1:6379> ZUNIONSTORE skills:combined 2 skills:alice skills:bob
(integer) 6

127.0.0.1:6379> ZRANGE skills:combined 0 -1 WITHSCORES
 1) "kubernetes"
 2) "5"
 3) "javascript"
 4) "6"
 5) "docker"
 6) "7"
 7) "java"
 8) "8"
 9) "python"
10) "15"
11) "sql"
12) "18"

# Intersection (common skills)
127.0.0.1:6379> ZINTERSTORE skills:common 2 skills:alice skills:bob
(integer) 2

127.0.0.1:6379> ZRANGE skills:common 0 -1 WITHSCORES
1) "python"
2) "15"
3) "sql"
4) "18"

# Union with different weights
127.0.0.1:6379> ZUNIONSTORE skills:weighted 2 skills:alice skills:bob WEIGHTS 1 2
(integer) 6
# Bob's scores are multiplied by 2, Alice's by 1
```

### 2. Lexicographical Ordering

```redis
# When scores are the same, Redis sorts lexicographically
127.0.0.1:6379> ZADD names 0 "alice" 0 "bob" 0 "charlie" 0 "diana"
(integer) 4

127.0.0.1:6379> ZRANGE names 0 -1
1) "alice"
2) "bob"
3) "charlie"
4) "diana"

# Lexicographical range queries
127.0.0.1:6379> ZRANGEBYLEX names [b [d
1) "bob"
2) "charlie"
3) "diana"

# Get names starting with 'c'
127.0.0.1:6379> ZRANGEBYLEX names [c [d
1) "charlie"
2) "diana"
```

---

## Performance Patterns

### 1. Efficient Range Operations

```redis
# Get top N items efficiently
ZREVRANGE leaderboard 0 9
# Top 10 players

# Get items around specific rank
ZREVRANGE leaderboard 45 55
# Players ranked 46-56

# Use LIMIT for pagination
ZRANGEBYSCORE products 0 +inf LIMIT 0 20
# First 20 products

ZRANGEBYSCORE products 0 +inf LIMIT 20 20
# Next 20 products (page 2)
```

### 2. Memory Optimization

```redis
# Use integers as scores when possible
ZADD timestamps 1705751400 "event1"
# Better than using floats

# Use shorter member names for large sets
ZADD rankings 1500 "u123" 1600 "u456"
# Instead of "user:123", "user:456"

# Remove old data regularly
ZREMRANGEBYRANK activity 0 -1001
# Keep only 1000 most recent items
```

### 3. Batch Operations

```redis
# Add multiple members efficiently
ZADD scores 100 "player1" 150 "player2" 200 "player3"
# Better than multiple ZADD commands

# Use pipelines for bulk operations in application code
```

---

## Common Sorted Set Patterns

### 1. Sliding Window Pattern

```python
# Python pseudocode for sliding window
def add_to_window(key, member, score, window_size):
    redis.zadd(key, {member: score})
    redis.zremrangebyrank(key, 0, -(window_size + 1))
    
# Keep only last 100 items
add_to_window("recent_activity", "action", timestamp, 100)
```

### 2. Time-Based Cleanup

```python
# Remove old entries
def cleanup_old_data(key, max_age_seconds):
    cutoff = time.time() - max_age_seconds
    redis.zremrangebyscore(key, "-inf", cutoff)

# Remove entries older than 1 hour
cleanup_old_data("user_activity", 3600)
```

### 3. Top-K Pattern

```python
# Maintain top K items
def update_top_k(key, member, score, k):
    redis.zadd(key, {member: score})
    # Keep only top K
    redis.zremrangebyrank(key, 0, -(k + 1))
    
# Keep top 10 scores
update_top_k("high_scores", "player", 1500, 10)
```

---

## Practice Exercises

### Exercise 1: News Article Popularity

```redis
# Articles with view counts
ZADD articles:popular 150 "article:tech-news" 89 "article:sports" 234 "article:politics"

# Article gets more views
ZINCRBY articles:popular 25 "article:sports"

# Get top 3 most popular
ZREVRANGE articles:popular 0 2 WITHSCORES

# Get articles with 100+ views
ZRANGEBYSCORE articles:popular 100 +inf
```

### Exercise 2: Event Timeline

```redis
# Events with timestamps
ZADD timeline:user123 1705751400 "user_registered" 1705751500 "first_login" 1705751600 "profile_updated"

# Add new event
ZADD timeline:user123 1705751700 "password_changed"

# Get chronological order
ZRANGE timeline:user123 0 -1

# Get events from last hour
ZRANGEBYSCORE timeline:user123 1705748000 +inf
```

### Exercise 3: Student Grades

```redis
# Student scores
ZADD class:grades 85 "alice" 92 "bob" 78 "charlie" 96 "diana"

# Update grade
ZADD class:grades 88 "alice"

# Get class ranking
ZREVRANGE class:grades 0 -1 WITHSCORES

# Find students with A grades (90+)
ZRANGEBYSCORE class:grades 90 +inf

# Get alice's rank
ZREVRANK class:grades "alice"
```

### Exercise 4: Website Analytics

```redis
# Page visits with timestamps
ZADD analytics:page_views 1705751400 "homepage" 1705751450 "about" 1705751500 "contact"

# New page visit
ZADD analytics:page_views 1705751550 "products"

# Get recent page views
ZREVRANGE analytics:page_views 0 -1

# Remove views older than 1 day
ZREMRANGEBYSCORE analytics:page_views -inf 1705665000
```

---

## Sorted Set Commands Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `ZADD key score member` | Add member with score | `ZADD leaderboard 1500 "alice"` |
| `ZRANGE key start stop` | Get members by rank | `ZRANGE scores 0 -1` |
| `ZREVRANGE key start stop` | Get members by rank (desc) | `ZREVRANGE scores 0 2` |
| `ZSCORE key member` | Get member's score | `ZSCORE leaderboard "alice"` |
| `ZRANK key member` | Get member's rank (asc) | `ZRANK scores "bob"` |
| `ZREVRANK key member` | Get member's rank (desc) | `ZREVRANK scores "bob"` |
| `ZCARD key` | Get number of members | `ZCARD leaderboard` |
| `ZREM key member` | Remove member | `ZREM scores "alice"` |
| `ZINCRBY key increment member` | Increment score | `ZINCRBY scores 10 "bob"` |
| `ZRANGEBYSCORE key min max` | Get members by score range | `ZRANGEBYSCORE prices 10 100` |
| `ZCOUNT key min max` | Count members in score range | `ZCOUNT scores 50 100` |
| `ZREMRANGEBYRANK key start stop` | Remove by rank range | `ZREMRANGEBYRANK old 0 10` |
| `ZREMRANGEBYSCORE key min max` | Remove by score range | `ZREMRANGEBYSCORE logs 0 1000` |

---
## What's Next

Ready to learn about **Expiration and TTL** - managing data lifecycle and automatic cleanup!
