# Leaderboards

A leaderboard is like a scoreboard that shows who has the highest scores in a game. Think of it like the high score list in an arcade game or the top students list in your class.

## What is a Leaderboard?

Imagine you have a game where people collect points. You want to show:

- Who has the most points
- What rank each player is
- The top 10 players

Redis makes this super easy with something called "sorted sets" - it's like a magic list that always keeps scores in order!

## Simple Example

Let's say we have a simple game where people collect coins:

```redis
# Add players and their coin counts
127.0.0.1:6379> ZADD game:coins 100 "alice"
(integer) 1
127.0.0.1:6379> ZADD game:coins 250 "bob"
(integer) 1
127.0.0.1:6379> ZADD game:coins 150 "charlie"
(integer) 1

# See who has the most coins (highest first)
127.0.0.1:6379> ZREVRANGE game:coins 0 2 WITHSCORES
1) "bob"
2) "250"
3) "charlie"
4) "150"
5) "alice"
6) "100"
```

Bob is winning with 250 coins!

## Find Someone's Rank

```redis
# What place is alice in?
127.0.0.1:6379> ZREVRANK game:coins "alice"
(integer) 2

# Get alice's score
127.0.0.1:6379> ZSCORE game:coins "alice"
"100"
```

Alice is in 3rd place (rank 2, since counting starts at 0) with 100 coins.

## Simple Python Code

```python
import redis

# Connect to Redis
r = redis.Redis()

def add_player_score(player_name, score):
    """Add a player's score to the leaderboard"""
    r.zadd("game:leaderboard", {player_name: score})
    print(f"Added {player_name} with {score} points")

def get_top_players():
    """Show the top 5 players"""
    top_players = r.zrevrange("game:leaderboard", 0, 4, withscores=True)
    
    print("=== TOP 5 PLAYERS ===")
    for i, (player, score) in enumerate(top_players):
        player_name = player.decode('utf-8')
        print(f"#{i+1}: {player_name} - {int(score)} points")

def find_player_rank(player_name):
    """Find where a player ranks"""
    rank = r.zrevrank("game:leaderboard", player_name)
    score = r.zscore("game:leaderboard", player_name)
    
    if rank is not None and score is not None:
        print(f"{player_name} is rank #{rank+1} with {int(score)} points")
    else:
        print(f"{player_name} is not on the leaderboard yet")

# Test it out
add_player_score("alice", 500)
add_player_score("bob", 800)
add_player_score("charlie", 300)
add_player_score("diana", 1000)

get_top_players()
find_player_rank("alice")
```

**Sample Output:**
```
Added alice with 500 points
Added bob with 800 points
Added charlie with 300 points
Added diana with 1000 points
=== TOP 5 PLAYERS ===
#1: diana - 1000 points
#2: bob - 800 points
#3: alice - 500 points
#4: charlie - 300 points
alice is rank #3 with 500 points
```

## Adding More Points

When someone gets more points, you can update their score:

```redis
# Bob gets 200 more points (now has 1000 total)
127.0.0.1:6379> ZADD game:coins 1000 "bob"
(integer) 0

# Or add points to their existing score
127.0.0.1:6379> ZINCRBY game:coins 100 "alice"
"200"
```

Alice now has 200 coins total (100 + 100 more).

Python example:

```python
def give_points(player_name, points):
    """Give someone more points"""
    new_score = r.zincrby("game:leaderboard", points, player_name)
    print(f"{player_name} got {points} more points! Now has {int(new_score)} total")

# Alice completes a level and gets 100 points
give_points("alice", 100)
```

**Sample Output:**
```
alice got 100 more points! Now has 600 total
```

## Intermediate: Weekly Leaderboards

Once you understand the basics, you might want leaderboards that reset every week. It's like having a "Player of the Week" contest:

```python
import datetime

def add_weekly_score(player_name, score):
    """Add score to this week's leaderboard"""
    # Get current week (like "2025-Week36")
    now = datetime.datetime.now()
    week = now.strftime("%Y-Week%U")
    
    # Add to this week's leaderboard
    weekly_key = f"leaderboard:{week}"
    r.zadd(weekly_key, {player_name: score})
    
    # Keep weekly leaderboards for 4 weeks
    r.expire(weekly_key, 86400 * 28)
    
    print(f"Added {player_name} to week {week} with {score} points")

def get_this_weeks_leaders():
    """Show this week's top players"""
    now = datetime.datetime.now()
    week = now.strftime("%Y-Week%U")
    weekly_key = f"leaderboard:{week}"
    
    top_players = r.zrevrange(weekly_key, 0, 4, withscores=True)
    
    print(f"=== TOP PLAYERS FOR {week} ===")
    for i, (player, score) in enumerate(top_players):
        player_name = player.decode('utf-8')
        print(f"#{i+1}: {player_name} - {int(score)} points")

# Test weekly leaderboard
add_weekly_score("alice", 300)
add_weekly_score("bob", 500)
get_this_weeks_leaders()
```

**Sample Output:**
```
Added alice to week 2025-Week36 with 300 points
Added bob to week 2025-Week36 with 500 points
=== TOP PLAYERS FOR 2025-Week36 ===
#1: bob - 500 points
#2: alice - 300 points
```

### Code Explanation

**Weekly Keys:**
```python
week = now.strftime("%Y-Week%U")  # Creates "2025-Week36"
weekly_key = f"leaderboard:{week}"  # Creates "leaderboard:2025-Week36"
```

- Each week gets its own leaderboard
- All players in the same week go to the same board

**Automatic Cleanup:**
```python
r.expire(weekly_key, 86400 * 28)  # Expires in 28 days
```

- Old weekly leaderboards automatically disappear
- Saves memory and keeps only recent weeks

**Why This is Useful:**

- New players can win weekly even if they can't beat all-time champions
- Players come back each week to compete
- Fresh competition every week keeps it exciting

## Summary

Leaderboards are like automatic scoreboards that always show who's winning:

1. Add players and their scores
2. Redis automatically sorts them from highest to lowest
3. You can see top players anytime
4. You can find anyone's rank and score
5. You can update scores easily

**What you learned:**

- Basic leaderboard with `ZADD`, `ZREVRANGE`, `ZREVRANK`
- How to add and update player scores
- Weekly leaderboards that reset automatically
- Simple Python code for managing leaderboards

Perfect for games, competitions, or any app where you want to show rankings!
