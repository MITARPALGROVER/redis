# Transactions

A transaction is like making a shopping list and buying everything at once. Instead of going to the store multiple times, you collect all your items and pay for them together. In Redis, transactions let you group multiple commands and run them all at once.

## What are Transactions?

Think of transactions like:

- **Shopping List:** Collect all items, then checkout together
- **Bank Transfer:** Move money from one account to another (both steps must work)
- **Game Move:** Update player score AND update leaderboard together
- **Recipe:** Mix all ingredients together, not one by one

The key idea: **All commands succeed together, or all fail together.**

## Why Use Transactions?

### Problem Without Transactions

Imagine updating a player's score and the leaderboard:

```redis
# Step 1: Update player score
127.0.0.1:6379> SET player:alice:score 1500
OK

# Step 2: Update leaderboard
127.0.0.1:6379> ZADD leaderboard 1500 alice
(integer) 1
```

**Problem:** What if something goes wrong between step 1 and step 2? The player's score gets updated but the leaderboard doesn't! Now the data is inconsistent.

### Solution With Transactions

```redis
# Start transaction
127.0.0.1:6379> MULTI
OK

# Add commands to the transaction
127.0.0.1:6379> SET player:alice:score 1500
QUEUED

127.0.0.1:6379> ZADD leaderboard 1500 alice
QUEUED

# Execute all commands together
127.0.0.1:6379> EXEC
1) OK
2) (integer) 1
```

**Result:** Both commands run together. If one fails, both fail. Data stays consistent!

## Basic Transaction Commands

### MULTI - Start a Transaction

```redis
127.0.0.1:6379> MULTI
OK
```

**What this means:**

- `MULTI` = "Start collecting commands"
- From now on, commands get queued (saved) instead of running immediately
- Like saying "I'm making a shopping list"

### EXEC - Execute the Transaction

```redis
127.0.0.1:6379> EXEC
1) OK
2) (integer) 1
3) "1500"
```

**What this means:**

- `EXEC` = "Run all the queued commands now"
- Returns results for each command in order
- Like paying for all items on your shopping list

### DISCARD - Cancel the Transaction

```redis
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET player:bob:score 2000
QUEUED
127.0.0.1:6379> DISCARD
OK
```

**What this means:**

- `DISCARD` = "Cancel everything, don't run any commands"
- All queued commands are thrown away
- Like putting your shopping cart back and leaving the store

## Simple Python Example

```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def update_player_score_safe(player_name, new_score):
    """Update player score and leaderboard together"""
    
    print(f"🎯 Updating {player_name}'s score to {new_score}...")
    
    # Start transaction
    pipe = r.pipeline()  # This creates a transaction
    
    # Add commands to transaction
    pipe.set(f"player:{player_name}:score", new_score)
    pipe.zadd("game:leaderboard", {player_name: new_score})
    pipe.get(f"player:{player_name}:score")  # Get the new score to confirm
    
    # Execute all commands together
    results = pipe.execute()
    
    print(f"✅ Transaction completed!")
    print(f"   - Set score: {results[0]}")
    print(f"   - Updated leaderboard: {results[1]}")
    print(f"   - Confirmed score: {results[2].decode('utf-8')}")
    
    return results

def transfer_points(from_player, to_player, points):
    """Transfer points from one player to another"""
    
    print(f"💸 Transferring {points} points from {from_player} to {to_player}...")
    
    # First check if from_player has enough points
    current_points = r.get(f"player:{from_player}:score")
    if current_points is None:
        print(f"❌ {from_player} doesn't exist!")
        return False
    
    current_points = int(current_points)
    if current_points < points:
        print(f"❌ {from_player} only has {current_points} points, needs {points}!")
        return False
    
    # Start transaction
    pipe = r.pipeline()
    
    # Calculate new scores
    new_from_score = current_points - points
    
    # Get to_player's current score
    to_current = r.get(f"player:{to_player}:score")
    new_to_score = int(to_current) + points if to_current else points
    
    # Add all commands to transaction
    pipe.set(f"player:{from_player}:score", new_from_score)
    pipe.set(f"player:{to_player}:score", new_to_score)
    pipe.zadd("game:leaderboard", {from_player: new_from_score})
    pipe.zadd("game:leaderboard", {to_player: new_to_score})
    
    # Execute transaction
    results = pipe.execute()
    
    print(f"✅ Transfer completed!")
    print(f"   - {from_player} now has {new_from_score} points")
    print(f"   - {to_player} now has {new_to_score} points")
    
    return True

# Example usage
def demo_transactions():
    """Demo showing transactions in action"""
    
    # Set up initial scores
    r.set("player:alice:score", 1000)
    r.set("player:bob:score", 500)
    r.zadd("game:leaderboard", {"alice": 1000, "bob": 500})
    
    print("=== INITIAL SCORES ===")
    print(f"Alice: {r.get('player:alice:score').decode('utf-8')} points")
    print(f"Bob: {r.get('player:bob:score').decode('utf-8')} points")
    
    print("\n=== UPDATING ALICE'S SCORE ===")
    update_player_score_safe("alice", 1500)
    
    print("\n=== TRANSFERRING POINTS ===")
    transfer_points("alice", "bob", 200)
    
    print("\n=== FINAL SCORES ===")
    print(f"Alice: {r.get('player:alice:score').decode('utf-8')} points")
    print(f"Bob: {r.get('player:bob:score').decode('utf-8')} points")

demo_transactions()
```

**Sample Output:**
```
=== INITIAL SCORES ===
Alice: 1000 points
Bob: 500 points

=== UPDATING ALICE'S SCORE ===
🎯 Updating alice's score to 1500...
✅ Transaction completed!
   - Set score: True
   - Updated leaderboard: 0
   - Confirmed score: 1500

=== TRANSFERRING POINTS ===
💸 Transferring 200 points from alice to bob...
✅ Transfer completed!
   - alice now has 1300 points
   - bob now has 700 points

=== FINAL SCORES ===
Alice: 1300 points
Bob: 700 points
```

## Code Explanation

Let me explain the important parts:

### Creating a Transaction (Pipeline)

```python
pipe = r.pipeline()
```

**What happens:**

- `pipeline()` creates a transaction object
- Commands added to `pipe` get queued instead of running immediately
- Like starting a shopping list

### Adding Commands to Transaction

```python
pipe.set(f"player:{player_name}:score", new_score)
pipe.zadd("game:leaderboard", {player_name: new_score})
```

**What happens:**

- Commands are added to the queue but not executed
- Each command returns `None` for now
- Like adding items to your shopping cart

### Executing the Transaction

```python
results = pipe.execute()
```

**What happens:**

- All queued commands run together
- Returns a list with results for each command
- Like paying for all items at checkout

### Why the Transfer Function is Safe

```python
# Without transaction - DANGEROUS!
r.set(f"player:{from_player}:score", new_from_score)  # ⚠️ What if this works...
r.set(f"player:{to_player}:score", new_to_score)     # ⚠️ ...but this fails?

# With transaction - SAFE!
pipe.set(f"player:{from_player}:score", new_from_score)
pipe.set(f"player:{to_player}:score", new_to_score)
results = pipe.execute()  # Both succeed or both fail
```

## Watching Keys (Advanced but Important)

Sometimes you need to check if a value changed before running your transaction:

```python
def safe_increment_if_less_than(key, max_value):
    """Increment a value only if it's less than max_value"""
    
    with r.pipeline() as pipe:
        while True:
            try:
                # Watch the key for changes
                pipe.watch(key)
                
                # Get current value
                current_value = pipe.get(key)
                current_value = int(current_value) if current_value else 0
                
                # Check if we can increment
                if current_value >= max_value:
                    pipe.unwatch()
                    return False, f"Value {current_value} is already at max {max_value}"
                
                # Start transaction
                pipe.multi()
                pipe.incr(key)
                
                # Execute
                result = pipe.execute()
                
                # If we get here, transaction succeeded
                return True, f"Incremented to {current_value + 1}"
                
            except redis.WatchError:
                # Someone else changed the key, try again
                print("🔄 Key was modified by someone else, retrying...")
                continue

# Example usage
def demo_watch():
    """Demo the WATCH functionality"""
    
    r.set("counter", 5)
    
    print("=== TESTING SAFE INCREMENT ===")
    print(f"Counter starts at: {r.get('counter').decode('utf-8')}")
    
    # This should work
    success, message = safe_increment_if_less_than("counter", 10)
    print(f"Increment attempt 1: {message}")
    print(f"Counter now: {r.get('counter').decode('utf-8')}")
    
    # Set counter to 9 and try again
    r.set("counter", 9)
    success, message = safe_increment_if_less_than("counter", 10)
    print(f"Increment attempt 2: {message}")
    print(f"Counter now: {r.get('counter').decode('utf-8')}")
    
    # This should fail (already at max)
    success, message = safe_increment_if_less_than("counter", 10)
    print(f"Increment attempt 3: {message}")

demo_watch()
```

**Sample Output:**
```
=== TESTING SAFE INCREMENT ===
Counter starts at: 5
Increment attempt 1: Incremented to 6
Counter now: 6
Increment attempt 2: Incremented to 10
Counter now: 10
Increment attempt 3: Value 10 is already at max 10
```

### WATCH Explanation

```python
pipe.watch(key)
```

**What this does:**

- "Watch" a key for changes
- If another client changes the key, the transaction will fail
- Like saying "only buy this if the price hasn't changed"

**Why it's useful:**

- Prevents race conditions (two people changing the same data)
- Ensures your transaction is based on current information
- Like checking the price before you pay

## Common Use Cases

### 1. Bank Transfer
```python
def bank_transfer(from_account, to_account, amount):
    pipe = r.pipeline()
    pipe.decrby(f"account:{from_account}", amount)  # Subtract from sender
    pipe.incrby(f"account:{to_account}", amount)    # Add to receiver
    pipe.execute()
```

### 2. Inventory Management
```python
def buy_item(user_id, item_id, price):
    pipe = r.pipeline()
    pipe.decrby(f"user:{user_id}:money", price)     # Pay money
    pipe.decr(f"item:{item_id}:stock")              # Reduce stock
    pipe.incr(f"user:{user_id}:items:{item_id}")    # Give item to user
    pipe.execute()
```

### 3. Game Score Update
```python
def complete_level(player_id, points_earned):
    pipe = r.pipeline()
    pipe.incrby(f"player:{player_id}:score", points_earned)
    pipe.incr(f"player:{player_id}:levels_completed")
    pipe.zadd("global:leaderboard", {player_id: points_earned})
    pipe.execute()
```

## Important Notes

### Transactions are Atomic

- **All commands succeed together** or **all fail together**
- No partial updates
- Data stays consistent

### Commands are Queued
```python
pipe.set("key1", "value1")  # Returns None (queued)
pipe.get("key1")            # Returns None (queued)
result = pipe.execute()     # Returns [True, "value1"]
```

### No Rollback

- Redis transactions don't have "rollback"
- If a command fails, other commands still run
- Plan your commands carefully

## Summary

Transactions group multiple commands to run together:

1. **MULTI** starts collecting commands
2. **Commands get queued** instead of running immediately  
3. **EXEC** runs all commands together
4. **DISCARD** cancels the transaction
5. **WATCH** prevents conflicts with other clients

**What you learned:**

- Basic transaction commands (`MULTI`, `EXEC`, `DISCARD`)
- Python pipeline for safe operations
- Point transfer and score updates
- WATCH for preventing race conditions
- Real-world use cases

**Perfect for:** Operations that must happen together (money transfers, inventory updates, game state changes).
