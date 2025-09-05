# Caching Basics

Caching means storing data temporarily so you can get it faster next time. Think of it like keeping snacks in your desk drawer instead of walking to the kitchen every time you're hungry.

## What is Caching?

When your website needs data, it usually asks the database (which is slow). With caching, we store that data in Redis (which is super fast) so next time we need it, we get it from Redis instead.

**Without Cache:**

1. User asks for data
2. App asks database (slow)
3. Database sends data back
4. App sends data to user

**With Cache:**

1. User asks for data
2. App checks Redis first (fast!)
3. If data is there, send it immediately
4. If not there, get from database and save to Redis

## Simple Caching Example

Let's say you have a website that shows user profiles. Every time someone visits a profile, your app has to ask the database for that user's information.

### Step 1: Check if Data Exists in Cache

```redis
# Check if user profile is already cached
127.0.0.1:6379> GET user:123
(nil)
```

`(nil)` means the data is not in cache yet.

### Step 2: Get Data from Database and Store in Cache

```redis
# After getting data from database, store it in Redis
127.0.0.1:6379> SET user:123 "Name: John, Age: 25, City: New York"
OK

# Set expiration time (data will disappear after 1 hour)
127.0.0.1:6379> EXPIRE user:123 3600
(integer) 1
```

### Step 3: Next Time, Get Data from Cache

```redis
# Now when someone visits the profile again
127.0.0.1:6379> GET user:123
"Name: John, Age: 25, City: New York"
```

This is much faster than asking the database!

## Simple Python Example

Here's how you would do this in Python:

```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def get_user_profile(user_id):
    # Step 1: Check cache first
    cached_data = r.get(f"user:{user_id}")
    
    if cached_data:
        print("Got data from cache (fast!)")
        return cached_data.decode('utf-8')
    
    # Step 2: If not in cache, get from database
    print("Getting data from database (slow)")
    user_data = get_from_database(user_id)  # Your database function
    
    # Step 3: Store in cache for next time
    r.set(f"user:{user_id}", user_data, ex=3600)  # expires in 1 hour
    
    return user_data

def get_from_database(user_id):
    # This is your slow database call
    return f"Name: User{user_id}, Age: 25, City: New York"

# Test it
profile = get_user_profile(123)
print(profile)
```

**Sample Output:**
```
🔍 Getting user profile from database... (slow)
Name: User123, Age: 25, City: New York

# Second call will be faster:
📦 Found user profile in cache! (fast)
Name: User123, Age: 25, City: New York
```

## Cache for Website Pages

You can also cache entire web pages:

```redis
# Store HTML page in cache
127.0.0.1:6379> SET page:homepage "<html><body>Welcome to our site!</body></html>"
OK

# Set to expire in 30 minutes
127.0.0.1:6379> EXPIRE page:homepage 1800
(integer) 1

# Get the page
127.0.0.1:6379> GET page:homepage
"<html><body>Welcome to our site!</body></html>"
```

## Cache Shopping Cart

For online shopping, you can cache what's in someone's cart:

```redis
# Store shopping cart items
127.0.0.1:6379> SET cart:user456 "item1,item2,item3"
OK

# Cart expires after 24 hours if not used
127.0.0.1:6379> EXPIRE cart:user456 86400
(integer) 1

# Get cart contents
127.0.0.1:6379> GET cart:user456
"item1,item2,item3"
```

## When to Clear Cache

Sometimes you need to remove old data from cache:

```redis
# Remove specific user data
127.0.0.1:6379> DEL user:123
(integer) 1

# Remove all pages that start with "page:"
127.0.0.1:6379> DEL page:homepage page:about page:contact
(integer) 3
```

## Cache Best Practices

### 1. Always Set Expiration Time
```redis
# Good - data expires in 1 hour
127.0.0.1:6379> SET user:123 "data" EX 3600

# Bad - data never expires (uses memory forever)
127.0.0.1:6379> SET user:123 "data"
```

### 2. Use Clear Naming
```redis
# Good - easy to understand
127.0.0.1:6379> SET user:profile:123 "user data"
127.0.0.1:6379> SET product:details:456 "product data"

# Bad - confusing names
127.0.0.1:6379> SET u123 "user data"
127.0.0.1:6379> SET p456 "product data"
```

### 3. Cache Frequently Used Data
Cache things that:

- Are requested often
- Take time to calculate
- Don't change frequently

Don't cache things that:

- Change every second
- Are requested rarely
- Are very large

## Summary

Caching with Redis is like having a super-fast notebook where you write down answers to questions you get asked a lot. Instead of looking up the answer in a big, slow book (database) every time, you just check your fast notebook (Redis) first!

The main steps are:

1. Check Redis for data
2. If not there, get from database
3. Store in Redis for next time
4. Set expiration time so old data gets removed

This makes your website much faster and happier users!
