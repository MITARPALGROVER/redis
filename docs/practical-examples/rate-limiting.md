# Rate Limiting

Rate limiting is like having a rule that says "you can only eat 3 cookies per hour" to prevent someone from eating all the cookies at once.

## Why Do We Need This?

Sometimes people try to:

- Login with wrong password many times (maybe trying to hack)
- Send too many messages very fast (spam)
- Click buttons too quickly

Rate limiting stops this by counting how many times someone does something.

## Simple Cookie Counter Example

Imagine you're counting cookies someone eats:

```redis
# John eats his first cookie today
127.0.0.1:6379> SET cookies:john 1
OK

# John eats another cookie (add 1 more)
127.0.0.1:6379> INCR cookies:john
(integer) 2

# Check how many cookies John ate
127.0.0.1:6379> GET cookies:john
"2"
```

If John tries to eat a 4th cookie, you can say "No! You already had 3 cookies today!"

## Real Website Example - Login Attempts

Let's say someone tries to login to your website:

```redis
# First wrong password attempt
127.0.0.1:6379> SET attempts:alice 1 EX 300
OK

# Second wrong attempt
127.0.0.1:6379> INCR attempts:alice
(integer) 2

# Third wrong attempt
127.0.0.1:6379> INCR attempts:alice
(integer) 3
```

After 3 wrong attempts, you can say "Stop! Try again in 5 minutes."

The `EX 300` means the counter disappears after 5 minutes (300 seconds).

## Very Simple Python Code

```python
import redis

# Connect to Redis
r = redis.Redis()

def check_if_user_can_try_login(username):
    # Check how many times they tried
    attempts = r.get(f"login:{username}")
    
    if attempts is None:
        # First time trying - allow it
        r.set(f"login:{username}", 1, ex=300)  # Count for 5 minutes
        return "You can try to login"
    
    attempts = int(attempts)
    if attempts >= 3:
        return "Too many wrong passwords! Wait 5 minutes"
    
    # Still allowed - add 1 more attempt
    r.incr(f"login:{username}")
    return "You can try to login"

# Test it
message = check_if_user_can_try_login("alice")
print(message)
```

## Intermediate: Different Time Windows

Once you understand the basics, you might want different rules for different time periods. It's like having rules for eating cookies:

- Maximum 5 cookies per hour
- Maximum 20 cookies per day

### Multiple Time Limits Example

```python
import redis
import time

r = redis.Redis()

def check_multiple_limits(username):
    """Check both hourly and daily limits"""
    
    # Check hourly limit (3600 seconds = 1 hour)
    hour_key = f"login:{username}:hour"
    hour_attempts = r.get(hour_key)
    
    if hour_attempts is None:
        hour_attempts = 0
    else:
        hour_attempts = int(hour_attempts)
    
    # Check daily limit (86400 seconds = 1 day)
    day_key = f"login:{username}:day"
    day_attempts = r.get(day_key)
    
    if day_attempts is None:
        day_attempts = 0
    else:
        day_attempts = int(day_attempts)
    
    # Check if limits are exceeded
    if hour_attempts >= 5:
        return "Too many attempts this hour! Wait 1 hour"
    
    if day_attempts >= 15:
        return "Too many attempts today! Try tomorrow"
    
    # Both limits OK - count this attempt
    if hour_attempts == 0:
        r.set(hour_key, 1, ex=3600)  # Count for 1 hour
    else:
        r.incr(hour_key)
    
    if day_attempts == 0:
        r.set(day_key, 1, ex=86400)  # Count for 1 day
    else:
        r.incr(day_key)
    
    return "You can try to login"

# Test it
result = check_multiple_limits("bob")
print(result)
```

### Code Explanation

Let me explain what this intermediate code does:

**1. Two Different Counters:**
```python
hour_key = f"login:{username}:hour"  # Counts attempts in 1 hour
day_key = f"login:{username}:day"    # Counts attempts in 1 day
```

- We create two separate keys for the same user
- One tracks hourly attempts, one tracks daily attempts

**2. Getting Current Counts:**
```python
hour_attempts = r.get(hour_key)
if hour_attempts is None:
    hour_attempts = 0
else:
    hour_attempts = int(hour_attempts)
```

- We check how many attempts already happened
- If Redis returns `None`, it means no attempts yet (so we set it to 0)
- We convert to integer because Redis stores everything as text

**3. Checking Multiple Rules:**
```python
if hour_attempts >= 5:
    return "Too many attempts this hour! Wait 1 hour"

if day_attempts >= 15:
    return "Too many attempts today! Try tomorrow"
```

- We check BOTH limits
- If either limit is exceeded, we block the user
- Different messages tell the user which limit they hit

**4. Counting the Attempt:**
```python
if hour_attempts == 0:
    r.set(hour_key, 1, ex=3600)  # Start counting for 1 hour
else:
    r.incr(hour_key)  # Add 1 to existing count
```

- If it's the first attempt in this time window, we start a new counter
- If there's already a counter, we just add 1 to it
- `ex=3600` means the counter disappears after 1 hour (3600 seconds)

### Why This is Useful

This intermediate approach lets you have:

- **Strict short-term limits:** Stop rapid attacks (5 per hour)
- **Generous long-term limits:** Allow normal users (15 per day)

Real websites often use this pattern because:

- Hackers try many passwords quickly (caught by hourly limit)
- Normal users might forget their password a few times (allowed by daily limit)

## Summary

Rate limiting is just counting:

1. Someone does something (login, send message, etc.)
2. We count it in Redis 
3. If they do it too much, we say "Stop!"
4. After some time, the counter resets

This keeps bad people from breaking your website!