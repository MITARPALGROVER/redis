# Lua Scripting

Lua scripting in Redis is like writing a recipe that Redis follows exactly. Instead of sending multiple commands one by one, you write a small program (script) that Redis runs all at once. It's faster and guarantees all steps happen together.

## What is Lua Scripting?

Think of Lua scripts like:

- **Recipe:** Follow all steps in order without interruption
- **Batch Job:** Do multiple tasks together instead of one by one
- **Magic Spell:** Write once, run many times with different inputs
- **Assembly Line:** All steps happen in sequence without breaks

Lua is a simple programming language that Redis understands.

## Why Use Lua Scripts?

### Problem Without Scripts

```redis
# Multiple commands (slow and can be interrupted)
127.0.0.1:6379> GET counter
"5"
127.0.0.1:6379> SET counter 6
OK
127.0.0.1:6379> INCR visits
(integer) 1
```

**Problems:**

- Multiple network calls (slow)
- Other clients can interfere between commands
- No guarantee all commands succeed together

### Solution With Lua Scripts

```redis
# One script does everything (fast and atomic)
127.0.0.1:6379> EVAL "local counter = redis.call('GET', 'counter') or 0; redis.call('SET', 'counter', counter + 1); return redis.call('INCR', 'visits')" 0
(integer) 1
```

**Benefits:**

- Single network call (fast)
- All commands run together (atomic)
- No interference from other clients

## Basic Lua Script Structure

### Simple Script Example

```lua
-- This is a Lua comment
local value = redis.call('GET', 'mykey')
return value
```

**Explanation:**

- `--` starts a comment (like `#` in other languages)
- `local` creates a variable (like `let` in JavaScript)
- `redis.call()` runs Redis commands from inside the script
- `return` sends the result back to the client

### Running Scripts with EVAL

```redis
127.0.0.1:6379> EVAL "return 'Hello from Lua!'" 0
"Hello from Lua!"
```

**What this means:**

- `EVAL` runs a Lua script
- `"return 'Hello from Lua!'"` is the script code
- `0` means no Redis keys are used (will explain this later)

## Basic Examples

### Example 1: Get and Set in One Operation

```redis
# Script that gets current value and sets a new one
127.0.0.1:6379> EVAL "local old = redis.call('GET', KEYS[1]); redis.call('SET', KEYS[1], ARGV[1]); return old" 1 mykey "new_value"
"old_value"
```

**Explanation:**

- `KEYS[1]` = first key parameter (`mykey`)
- `ARGV[1]` = first argument parameter (`"new_value"`)
- `1` = number of keys (just `mykey`)
- Script gets old value, sets new value, returns old value

### Example 2: Conditional Increment

```redis
# Only increment if value is less than 10
127.0.0.1:6379> EVAL "local val = tonumber(redis.call('GET', KEYS[1]) or 0); if val < 10 then return redis.call('INCR', KEYS[1]) else return val end" 1 counter
(integer) 1
```

**Explanation:**

- `tonumber()` converts text to number
- `or 0` provides default value if key doesn't exist
- `if val < 10 then` checks condition
- `else return val` returns current value if too high

## Python Examples with Detailed Explanations

```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def atomic_increment_with_limit(key, limit):
    """Increment a counter but only if it's below the limit"""
    
    script = """
    local current = tonumber(redis.call('GET', KEYS[1]) or 0)
    if current < tonumber(ARGV[1]) then
        return redis.call('INCR', KEYS[1])
    else
        return current
    end
    """
    
    result = r.eval(script, 1, key, limit)
    return result

def transfer_points_safely(from_user, to_user, points):
    """Transfer points between users only if sender has enough"""
    
    script = """
    local from_key = KEYS[1]
    local to_key = KEYS[2]
    local transfer_amount = tonumber(ARGV[1])
    
    -- Get current balances
    local from_balance = tonumber(redis.call('GET', from_key) or 0)
    local to_balance = tonumber(redis.call('GET', to_key) or 0)
    
    -- Check if sender has enough points
    if from_balance >= transfer_amount then
        -- Do the transfer
        redis.call('SET', from_key, from_balance - transfer_amount)
        redis.call('SET', to_key, to_balance + transfer_amount)
        return {from_balance - transfer_amount, to_balance + transfer_amount}
    else
        -- Not enough points
        return {-1, -1}
    end
    """
    
    result = r.eval(script, 2, f"points:{from_user}", f"points:{to_user}", points)
    return result

def get_and_update_stats(user_id):
    """Get user stats and update last_seen in one operation"""
    
    script = """
    local user_key = KEYS[1]
    local stats_key = KEYS[2]
    local current_time = ARGV[1]
    
    -- Get current stats
    local stats = redis.call('HGETALL', stats_key)
    
    -- Update last seen
    redis.call('HSET', stats_key, 'last_seen', current_time)
    redis.call('INCR', user_key .. ':visits')
    
    -- Return stats as a table
    return stats
    """
    
    import time
    current_time = str(int(time.time()))
    
    result = r.eval(script, 2, f"user:{user_id}", f"stats:{user_id}", current_time)
    return result

# Example usage
def demo_lua_scripts():
    """Demo various Lua script examples"""
    
    print("=== TESTING CONDITIONAL INCREMENT ===")
    
    # Set initial counter
    r.set("counter", 8)
    
    # Try to increment (should work - 8 < 10)
    result1 = atomic_increment_with_limit("counter", 10)
    print(f"Increment attempt 1: {result1}")
    
    # Try again (should work - 9 < 10)
    result2 = atomic_increment_with_limit("counter", 10)
    print(f"Increment attempt 2: {result2}")
    
    # Try again (should fail - 10 is not < 10)
    result3 = atomic_increment_with_limit("counter", 10)
    print(f"Increment attempt 3: {result3}")
    
    print("\n=== TESTING POINT TRANSFER ===")
    
    # Set up user balances
    r.set("points:alice", 100)
    r.set("points:bob", 50)
    
    print(f"Alice starts with: {r.get('points:alice').decode('utf-8')} points")
    print(f"Bob starts with: {r.get('points:bob').decode('utf-8')} points")
    
    # Valid transfer
    result = transfer_points_safely("alice", "bob", 30)
    print(f"Transfer 30 points: Alice={result[0]}, Bob={result[1]}")
    
    # Invalid transfer (not enough points)
    result = transfer_points_safely("alice", "bob", 100)
    if result[0] == -1:
        print("Transfer failed: Not enough points")
    
    print("\n=== TESTING STATS UPDATE ===")
    
    # Set up user stats
    r.hset("stats:user123", mapping={
        "name": "Alice",
        "level": "5",
        "score": "1500"
    })
    
    stats = get_and_update_stats("user123")
    print("User stats:")
    for i in range(0, len(stats), 2):
        key = stats[i].decode('utf-8')
        value = stats[i+1].decode('utf-8')
        print(f"  {key}: {value}")

demo_lua_scripts()
```

**Sample Output:**
```
=== TESTING CONDITIONAL INCREMENT ===
Increment attempt 1: 9
Increment attempt 2: 10
Increment attempt 3: 10

=== TESTING POINT TRANSFER ===
Alice starts with: 100 points
Bob starts with: 50 points
Transfer 30 points: Alice=70, Bob=80
Transfer failed: Not enough points

=== TESTING STATS UPDATE ===
User stats:
  name: Alice
  level: 5
  score: 1500
  last_seen: 1693939200
```

## Code Explanation

Let me explain the important parts of the Lua scripts:

### Script Structure

```python
script = """
local current = tonumber(redis.call('GET', KEYS[1]) or 0)
if current < tonumber(ARGV[1]) then
    return redis.call('INCR', KEYS[1])
else
    return current
end
"""
```

**What each part does:**

1. `local current = ...` creates a variable to store the current value
2. `redis.call('GET', KEYS[1])` gets the value from Redis
3. `or 0` provides a default if the key doesn't exist
4. `tonumber()` converts text to a number
5. `if ... then ... else ... end` is Lua's if-statement
6. `return` sends the result back to Python

### KEYS vs ARGV

```python
r.eval(script, 2, f"points:{from_user}", f"points:{to_user}", points)
```

**Parameters explained:**

- `script` = the Lua code to run
- `2` = number of keys (2 keys: from_user and to_user)
- `f"points:{from_user}"` = KEYS[1] in the script
- `f"points:{to_user}"` = KEYS[2] in the script
- `points` = ARGV[1] in the script

**Why separate KEYS and ARGV?**

- KEYS = Redis keys that the script will read/write
- ARGV = Arguments/parameters for the script
- Redis uses this for optimization and clustering

### Error Handling in Scripts

```lua
-- Check if sender has enough points
if from_balance >= transfer_amount then
    -- Do the transfer
    redis.call('SET', from_key, from_balance - transfer_amount)
    redis.call('SET', to_key, to_balance + transfer_amount)
    return {from_balance - transfer_amount, to_balance + transfer_amount}
else
    -- Not enough points
    return {-1, -1}
end
```

**What this does:**

- Checks if the transfer is valid before doing it
- If valid: performs the transfer and returns new balances
- If invalid: returns error codes (-1, -1)
- All operations are atomic (happen together or not at all)

## Script Caching with EVALSHA

For scripts you use often, Redis can cache them:

```python
def use_cached_script():
    """Example of using script caching"""
    
    script = "return redis.call('INCR', KEYS[1])"
    
    # First time: upload and run script
    script_sha = r.script_load(script)
    print(f"Script cached with SHA: {script_sha}")
    
    # Later: run cached script (faster)
    result = r.evalsha(script_sha, 1, "mycounter")
    print(f"Counter value: {result}")
    
    return result

# Demo caching
cached_result = use_cached_script()
```

**Sample Output:**
```
Script cached with SHA: 7413dc2440db1fea7c0a0bde6cf6ccc02c02f05e
Counter value: 1
```

### Benefits of Caching

**EVAL (not cached):**

- Sends full script code every time
- Redis parses script every time
- Slower for repeated use

**EVALSHA (cached):**

- Sends only script hash (shorter)
- Redis reuses parsed script
- Faster for repeated use

## Common Use Cases

### 1. Rate Limiting with Lua

```python
def lua_rate_limit(user_id, limit, window):
    """Rate limiting using Lua script"""
    
    script = """
    local key = KEYS[1]
    local limit = tonumber(ARGV[1])
    local window = tonumber(ARGV[2])
    local current_time = tonumber(ARGV[3])
    
    -- Remove expired entries
    redis.call('ZREMRANGEBYSCORE', key, 0, current_time - window)
    
    -- Count current requests
    local current_count = redis.call('ZCARD', key)
    
    if current_count < limit then
        -- Add this request
        redis.call('ZADD', key, current_time, current_time)
        redis.call('EXPIRE', key, window)
        return {1, limit - current_count - 1}
    else
        return {0, 0}
    end
    """
    
    import time
    current_time = time.time()
    
    result = r.eval(script, 1, f"rate:{user_id}", limit, window, current_time)
    return result  # [allowed (1/0), remaining_requests]
```

### 2. Leaderboard Updates

```python
def update_leaderboard(player_id, score_change):
    """Update player score and leaderboard atomically"""
    
    script = """
    local player_key = KEYS[1]
    local leaderboard_key = KEYS[2]
    local score_change = tonumber(ARGV[1])
    
    -- Get current score
    local current_score = tonumber(redis.call('GET', player_key) or 0)
    local new_score = current_score + score_change
    
    -- Update player score
    redis.call('SET', player_key, new_score)
    
    -- Update leaderboard
    redis.call('ZADD', leaderboard_key, new_score, KEYS[1])
    
    -- Get player's new rank
    local rank = redis.call('ZREVRANK', leaderboard_key, KEYS[1])
    
    return {new_score, rank + 1}
    """
    
    result = r.eval(script, 2, f"score:{player_id}", "leaderboard", score_change)
    return result  # [new_score, rank]
```

## Important Lua Rules

### 1. Scripts Must Be Deterministic
```lua
-- BAD: Uses random numbers
return math.random(100)

-- GOOD: Use input parameters
return tonumber(ARGV[1])
```

### 2. No Infinite Loops
```lua
-- BAD: Can run forever
while true do
    -- something
end

-- GOOD: Limited iterations
for i = 1, 10 do
    -- something
end
```

### 3. Use Local Variables
```lua
-- BAD: Global variable
value = redis.call('GET', 'key')

-- GOOD: Local variable
local value = redis.call('GET', 'key')
```

## Summary

Lua scripting lets you run multiple Redis commands atomically:

1. **Write scripts** in Lua programming language
2. **Run with EVAL** for one-time use
3. **Cache with EVALSHA** for repeated use
4. **Pass KEYS and ARGV** for parameters
5. **Everything runs atomically** (all or nothing)

**What you learned:**

- Basic Lua syntax and structure
- How to run scripts with EVAL
- Passing keys and arguments to scripts
- Real-world examples (transfers, rate limiting, leaderboards)
- Script caching for better performance
- Important rules and limitations

**Perfect for:** Operations that need multiple Redis commands to run together without interruption (transfers, complex updates, calculations).
