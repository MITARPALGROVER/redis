# List Operations

Redis lists are like ordered lines of people - you can add items to the front, back, or middle, and everyone stays in their position. Perfect for queues, timelines, activity feeds, and any data where order matters.

## What Are Redis Lists?

Think of Redis lists as:

- A **queue** at a coffee shop (first in, first out)
- A **stack** of plates (last in, first out) 
- A **timeline** of events (newest first)
- A **to-do list** where order matters

Lists can hold up to **4 billion items**, and adding/removing from the ends is super fast!

---

## Basic List Operations

### Adding Items to Lists

```redis
# Add items to the front (left) of a list
127.0.0.1:6379> LPUSH todo "buy milk"
(integer) 1
# Returns the new length of the list

127.0.0.1:6379> LPUSH todo "walk dog" "call mom"
(integer) 3
# Added 2 more items to the front

# Add items to the back (right) of a list
127.0.0.1:6379> RPUSH todo "read book"
(integer) 4

# See what's in the list
127.0.0.1:6379> LRANGE todo 0 -1
1) "call mom"
2) "walk dog" 
3) "buy milk"
4) "read book"
# Notice the order: newest LPUSH items are at the front
```

**Think of it like this:**

- `LPUSH` = Add to **L**eft (front of line)
- `RPUSH` = Add to **R**ight (back of line)

### Getting Items from Lists

```redis
# Get all items (0 = start, -1 = end)
127.0.0.1:6379> LRANGE todo 0 -1
1) "call mom"
2) "walk dog"
3) "buy milk" 
4) "read book"

# Get first 2 items
127.0.0.1:6379> LRANGE todo 0 1
1) "call mom"
2) "walk dog"

# Get last 2 items  
127.0.0.1:6379> LRANGE todo -2 -1
1) "buy milk"
2) "read book"

# Get just the middle items
127.0.0.1:6379> LRANGE todo 1 2
1) "walk dog"
2) "buy milk"

# Get specific item by position
127.0.0.1:6379> LINDEX todo 0
"call mom"
# First item (position 0)

127.0.0.1:6379> LINDEX todo -1
"read book"
# Last item (position -1)

# Check how many items in list
127.0.0.1:6379> LLEN todo
(integer) 4
```

### Removing Items from Lists

```redis
# Remove and return first item (left/front)
127.0.0.1:6379> LPOP todo
"call mom"

127.0.0.1:6379> LRANGE todo 0 -1
1) "walk dog"
2) "buy milk"
3) "read book"

# Remove and return last item (right/back)
127.0.0.1:6379> RPOP todo
"read book"

127.0.0.1:6379> LRANGE todo 0 -1
1) "walk dog"
2) "buy milk"

# Remove specific values (not by position)
127.0.0.1:6379> LPUSH numbers 1 2 1 3 1 4
(integer) 6

127.0.0.1:6379> LRANGE numbers 0 -1
1) "4"
2) "1"
3) "3"
4) "1"
5) "2"
6) "1"

# Remove first 2 occurrences of "1"
127.0.0.1:6379> LREM numbers 2 "1"
(integer) 2

127.0.0.1:6379> LRANGE numbers 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
# Removed the first 2 "1"s we encountered
```

**LREM parameters explained:**

- `LREM key count value`
- `count > 0`: Remove first N occurrences
- `count < 0`: Remove last N occurrences  
- `count = 0`: Remove all occurrences

---

## Modifying Lists

### Changing Items in Lists

```redis
# Set item at specific position
127.0.0.1:6379> LPUSH fruits "apple" "banana" "orange"
(integer) 3

127.0.0.1:6379> LRANGE fruits 0 -1
1) "orange"
2) "banana"
3) "apple"

# Change the second item (position 1)
127.0.0.1:6379> LSET fruits 1 "grape"
OK

127.0.0.1:6379> LRANGE fruits 0 -1
1) "orange"
2) "grape"
3) "apple"

# Insert item before or after existing item
127.0.0.1:6379> LINSERT fruits BEFORE "grape" "kiwi"
(integer) 4

127.0.0.1:6379> LINSERT fruits AFTER "apple" "mango"
(integer) 5

127.0.0.1:6379> LRANGE fruits 0 -1
1) "orange"
2) "kiwi"
3) "grape" 
4) "apple"
5) "mango"
```

### Trimming Lists (Keep Only Part)

```redis
# Keep only items from position 1 to 3
127.0.0.1:6379> LTRIM fruits 1 3
OK

127.0.0.1:6379> LRANGE fruits 0 -1
1) "kiwi"
2) "grape"
3) "apple"
# "orange" and "mango" were removed

# Common pattern: Keep only last 10 items
127.0.0.1:6379> LPUSH log "event1" "event2" "event3" "event4" "event5"
(integer) 5

127.0.0.1:6379> LTRIM log 0 2
OK
# Keep only first 3 items

127.0.0.1:6379> LRANGE log 0 -1
1) "event5"
2) "event4" 
3) "event3"
# This is how you maintain a "recent items" list
```

---

## Advanced List Operations

### Blocking Operations (Wait for Items)

```redis
# Wait for item to be added to list (blocks until item arrives)
127.0.0.1:6379> BLPOP queue:jobs 30
# Waits up to 30 seconds for an item in "queue:jobs"

# In another terminal/connection, add an item:
127.0.0.1:6379> LPUSH queue:jobs "process_payment"
(integer) 1

# The BLPOP immediately returns:
1) "queue:jobs"
2) "process_payment"

# Wait for items from multiple lists
127.0.0.1:6379> BLPOP queue:urgent queue:normal queue:low 60
# Checks lists in order, waits up to 60 seconds
```

**When to use blocking operations:**

- Job queues (workers waiting for tasks)
- Real-time messaging
- Producer-consumer patterns

### Moving Items Between Lists

```redis
# Move item from end of one list to start of another
127.0.0.1:6379> RPUSH source "item1" "item2" "item3"
(integer) 3

127.0.0.1:6379> LPUSH destination "existing"
(integer) 1

127.0.0.1:6379> RPOPLPUSH source destination
"item3"

127.0.0.1:6379> LRANGE source 0 -1
1) "item1" 
2) "item2"

127.0.0.1:6379> LRANGE destination 0 -1
1) "item3"
2) "existing"
# "item3" moved from end of source to front of destination

# Blocking version (waits for items)
127.0.0.1:6379> BRPOPLPUSH source destination 30
# Waits up to 30 seconds for item to move
```

---

## Real-World List Use Cases

### 1. Activity Feed / Timeline

```redis
# Add new activities (newest first)
127.0.0.1:6379> LPUSH feed:user123 "liked a post"
(integer) 1

127.0.0.1:6379> LPUSH feed:user123 "commented on photo"
(integer) 2

127.0.0.1:6379> LPUSH feed:user123 "shared an article"
(integer) 3

# Get recent activities (last 10)
127.0.0.1:6379> LRANGE feed:user123 0 9
1) "shared an article"
2) "commented on photo"
3) "liked a post"

# Keep only recent 100 activities (memory management)
127.0.0.1:6379> LTRIM feed:user123 0 99
OK
```

### 2. Task Queue System

```redis
# Add urgent task to front of queue
127.0.0.1:6379> LPUSH tasks:urgent "process_refund:order123"
(integer) 1

# Add normal task to back of queue
127.0.0.1:6379> RPUSH tasks:normal "send_email:welcome"
(integer) 1

# Worker picks up urgent tasks first
127.0.0.1:6379> LPOP tasks:urgent
"process_refund:order123"

# If no urgent tasks, pick up normal tasks
127.0.0.1:6379> LPOP tasks:urgent
(nil)

127.0.0.1:6379> LPOP tasks:normal
"send_email:welcome"

# Worker waits for new tasks (blocking)
127.0.0.1:6379> BLPOP tasks:urgent tasks:normal 60
# Waits 60 seconds for tasks, checking urgent first
```

### 3. Shopping Cart

```redis
# Add items to cart (order might matter for checkout)
127.0.0.1:6379> RPUSH cart:user456 "item:laptop:1299"
(integer) 1

127.0.0.1:6379> RPUSH cart:user456 "item:mouse:29"
(integer) 2

127.0.0.1:6379> RPUSH cart:user456 "item:keyboard:89"
(integer) 3

# View cart contents
127.0.0.1:6379> LRANGE cart:user456 0 -1
1) "item:laptop:1299"
2) "item:mouse:29"
3) "item:keyboard:89"

# Remove specific item from cart
127.0.0.1:6379> LREM cart:user456 1 "item:mouse:29"
(integer) 1

127.0.0.1:6379> LRANGE cart:user456 0 -1
1) "item:laptop:1299"
2) "item:keyboard:89"

# Clear entire cart
127.0.0.1:6379> DEL cart:user456
(integer) 1
```

### 4. Recent Searches / History

```redis
# Add search to history (newest first)
127.0.0.1:6379> LPUSH search:user789 "redis tutorial"
(integer) 1

127.0.0.1:6379> LPUSH search:user789 "python lists"
(integer) 2

127.0.0.1:6379> LPUSH search:user789 "web development"
(integer) 3

# Show recent searches
127.0.0.1:6379> LRANGE search:user789 0 4
1) "web development"
2) "python lists"
3) "redis tutorial"

# Keep only last 20 searches
127.0.0.1:6379> LTRIM search:user789 0 19
OK

# Remove duplicate if user searches same thing again
# (You'd need to LREM old occurrence before LPUSH new)
127.0.0.1:6379> LREM search:user789 1 "redis tutorial"
(integer) 1

127.0.0.1:6379> LPUSH search:user789 "redis tutorial"
(integer) 3
# Now "redis tutorial" is at the top again
```

### 5. Undo/Redo System

```redis
# Save document states for undo
127.0.0.1:6379> LPUSH doc:states:user123 '{"text":"Hello World","cursor":11}'
(integer) 1

127.0.0.1:6379> LPUSH doc:states:user123 '{"text":"Hello World!","cursor":12}'
(integer) 2

127.0.0.1:6379> LPUSH doc:states:user123 '{"text":"Hello Redis World!","cursor":18}'
(integer) 3

# Undo: get previous state
127.0.0.1:6379> LINDEX doc:states:user123 1
"{\"text\":\"Hello World!\",\"cursor\":12}"

# Keep only last 50 states (memory limit)
127.0.0.1:6379> LTRIM doc:states:user123 0 49
OK
```

---

## List Performance Patterns

### 1. Efficient Queue Operations

```redis
# Fast: Add to one end, remove from other
LPUSH queue "task1"  # Add to left
RPOP queue           # Remove from right

# Also fast: Add to right, remove from left  
RPUSH queue "task2"  # Add to right
LPOP queue           # Remove from left

# Slower: Operations in the middle
LINDEX queue 1000    # Getting middle items
LSET queue 1000 "new" # Setting middle items
```

### 2. Memory-Efficient History

```redis
# Good: Maintain fixed-size history
LPUSH history "new_item"
LTRIM history 0 999  # Keep only 1000 items

# Bad: Unlimited growth
LPUSH history "new_item"
# List grows forever, uses more memory
```

### 3. Batch Processing

```redis
# Process multiple items at once
RPOP queue  # Get one item
RPOP queue  # Get another
RPOP queue  # Get third
# Process batch of 3 items

# Even better: Use LRANGE to get multiple, then LTRIM
LRANGE queue -10 -1  # Get last 10 items
LTRIM queue 0 -11    # Remove those 10 items
```

---

## Common List Patterns

### 1. Latest Items Feed

```python
# Python pseudocode
def add_to_feed(user_id, activity):
    redis.lpush(f"feed:{user_id}", activity)
    redis.ltrim(f"feed:{user_id}", 0, 99)  # Keep latest 100

def get_feed(user_id, page=0, per_page=10):
    start = page * per_page
    end = start + per_page - 1
    return redis.lrange(f"feed:{user_id}", start, end)
```

### 2. Job Queue with Priority

```python
def add_job(job_data, priority="normal"):
    if priority == "urgent":
        redis.lpush("jobs:urgent", job_data)
    else:
        redis.rpush("jobs:normal", job_data)

def get_next_job():
    # Check urgent first, then normal
    job = redis.lpop("jobs:urgent") 
    if not job:
        job = redis.lpop("jobs:normal")
    return job
```

### 3. Recent Items with Duplicates Removed

```python
def add_recent_item(user_id, item):
    key = f"recent:{user_id}"
    # Remove item if it exists
    redis.lrem(key, 1, item)
    # Add to front
    redis.lpush(key, item)
    # Keep only 20 recent
    redis.ltrim(key, 0, 19)
```

---

## Practice Exercises

### Exercise 1: Chat Messages

```redis
# Store chat messages for a room
LPUSH chat:room1 "Alice: Hello everyone!"
LPUSH chat:room1 "Bob: Hi Alice!"
LPUSH chat:room1 "Charlie: Good morning!"

# Get recent 10 messages (newest first)
LRANGE chat:room1 0 9

# Keep only last 100 messages
LTRIM chat:room1 0 99
```

### Exercise 2: Breadcrumb Navigation

```redis
# User navigates through pages
RPUSH breadcrumb:user123 "Home"
RPUSH breadcrumb:user123 "Products" 
RPUSH breadcrumb:user123 "Electronics"
RPUSH breadcrumb:user123 "Laptops"

# Show breadcrumb trail
LRANGE breadcrumb:user123 0 -1

# User goes back one page
RPOP breadcrumb:user123

# Show updated breadcrumb
LRANGE breadcrumb:user123 0 -1
```

### Exercise 3: Notification Queue

```redis
# Add notifications (newest first)
LPUSH notifications:user456 "You have a new message"
LPUSH notifications:user456 "Your order has shipped"

# User reads notifications (oldest first)
RPOP notifications:user456
RPOP notifications:user456

# Check remaining notifications
LLEN notifications:user456
```

### Exercise 4: Work Queue with Workers

```redis
# Add work to queue
RPUSH work:queue "resize_image:photo1.jpg"
RPUSH work:queue "send_email:welcome@user.com"
RPUSH work:queue "backup_database:daily"

# Worker waits for work (blocks until available)
BLPOP work:queue 30

# Another worker can pick up next job
BLPOP work:queue 30
```

---

## List Commands Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `LPUSH key value` | Add to front | `LPUSH todo "urgent task"` |
| `RPUSH key value` | Add to back | `RPUSH log "new entry"` |
| `LPOP key` | Remove from front | `LPOP queue` |
| `RPOP key` | Remove from back | `RPOP stack` |
| `LRANGE key start stop` | Get range | `LRANGE items 0 -1` |
| `LLEN key` | Get length | `LLEN shopping_cart` |
| `LINDEX key index` | Get by position | `LINDEX list 0` |
| `LSET key index value` | Set by position | `LSET list 1 "new"` |
| `LREM key count value` | Remove by value | `LREM list 2 "item"` |
| `LTRIM key start stop` | Keep only range | `LTRIM recent 0 99` |
| `LINSERT key BEFORE/AFTER pivot value` | Insert relative | `LINSERT list BEFORE "item" "new"` |
| `BLPOP key timeout` | Blocking remove front | `BLPOP queue 30` |
| `BRPOP key timeout` | Blocking remove back | `BRPOP stack 30` |
| `RPOPLPUSH source dest` | Move between lists | `RPOPLPUSH todo done` |

---

## What's Next?

Ready to learn about **Sets** - collections of unique items perfect for tags, followers, and membership tracking!
