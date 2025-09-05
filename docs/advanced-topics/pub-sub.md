# Pub/Sub (Publish/Subscribe)

Pub/Sub is like having a radio station. Some people broadcast messages (publishers) and others listen to those messages (subscribers). When someone broadcasts, everyone who is listening gets the message instantly.

## What is Pub/Sub?

Think of Pub/Sub like:

- **Radio Station:** DJ broadcasts music, people tune in to listen
- **News Alert:** Breaking news gets sent to everyone who subscribed
- **School Announcements:** Principal speaks, all classes hear it
- **Group Chat:** One person sends message, everyone in group sees it

In Redis, this happens through "channels" - like different radio stations for different topics.

## Basic Commands

### PUBLISH - Broadcasting a Message

```redis
# Send a message to the "news" channel
127.0.0.1:6379> PUBLISH news "Breaking: New Redis tutorial released!"
(integer) 3
```

**What this means:**

- `PUBLISH` = broadcast a message
- `news` = the channel name (like a radio station)
- The message = what you want to say
- `(integer) 3` = 3 people received this message

### SUBSCRIBE - Listening to Messages

```redis
# Listen to the "news" channel
127.0.0.1:6379> SUBSCRIBE news
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news" 
3) (integer) 1
```

**What this means:**

- `SUBSCRIBE` = start listening
- `news` = which channel to listen to
- The response confirms you're now subscribed
- `(integer) 1` = you're the 1st subscriber to this channel

### What Subscribers See

When someone publishes a message, subscribers see:
```redis
1) "message"
2) "news"
3) "Breaking: New Redis tutorial released!"
```

**Explanation:**

- Line 1: `"message"` = this is a new message (not a subscription confirmation)
- Line 2: `"news"` = which channel it came from
- Line 3: The actual message content

## Simple Python Example

```python
import redis
import time
import threading

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def publisher():
    """This function sends messages"""
    print("📻 Publisher starting...")
    time.sleep(2)  # Wait a bit for subscribers to connect
    
    # Send some news updates
    r.publish("news", "Weather: Sunny today!")
    print("✅ Sent: Weather update")
    
    time.sleep(1)
    r.publish("news", "Sports: Local team wins!")
    print("✅ Sent: Sports update")
    
    time.sleep(1)
    r.publish("news", "Traffic: Road closure on Main St")
    print("✅ Sent: Traffic update")

def subscriber(name):
    """This function listens for messages"""
    print(f"👂 {name} is listening for news...")
    
    # Create a subscriber
    pubsub = r.pubsub()
    pubsub.subscribe("news")
    
    # Listen for messages
    for message in pubsub.listen():
        if message['type'] == 'message':
            news = message['data'].decode('utf-8')
            print(f"📰 {name} heard: {news}")

# Demo: Run publisher and subscriber together
def demo():
    """Run a simple pub/sub demo"""
    
    # Start subscriber in background
    subscriber_thread = threading.Thread(target=subscriber, args=("Alice",))
    subscriber_thread.daemon = True
    subscriber_thread.start()
    
    # Start another subscriber
    subscriber_thread2 = threading.Thread(target=subscriber, args=("Bob",))
    subscriber_thread2.daemon = True
    subscriber_thread2.start()
    
    # Give subscribers time to connect
    time.sleep(1)
    
    # Now publish messages
    publisher()
    
    # Wait to see all messages
    time.sleep(2)

# Run the demo
demo()
```

## Code Explanation

Let me explain each part of the Python code:

### Creating a Publisher

```python
def publisher():
    r.publish("news", "Weather: Sunny today!")
```

**What happens:**

1. `r.publish()` sends a message to Redis
2. `"news"` is the channel name
3. `"Weather: Sunny today!"` is the message
4. Redis instantly sends this to all subscribers

### Creating a Subscriber

```python
pubsub = r.pubsub()
pubsub.subscribe("news")
```

**What happens:**

1. `r.pubsub()` creates a subscriber object
2. `subscribe("news")` tells Redis "I want to listen to the news channel"
3. Now this subscriber will get all messages sent to "news"

### Listening for Messages

```python
for message in pubsub.listen():
    if message['type'] == 'message':
        news = message['data'].decode('utf-8')
```

**What happens:**

1. `pubsub.listen()` keeps checking for new messages
2. `message['type'] == 'message'` filters out subscription confirmations
3. `message['data']` contains the actual message
4. `.decode('utf-8')` converts it to readable text

## Multiple Channels Example

You can have different channels for different topics:

```python
def multi_channel_demo():
    """Demo with multiple channels"""
    
    # Publisher sends to different channels
    r.publish("sports", "Football game starts at 8pm")
    r.publish("weather", "Temperature: 25°C")
    r.publish("traffic", "Heavy traffic on Highway 1")
    
    print("✅ Sent messages to sports, weather, and traffic channels")

def specific_subscriber(channels, name):
    """Subscribe to specific channels only"""
    pubsub = r.pubsub()
    pubsub.subscribe(*channels)  # Subscribe to multiple channels
    
    print(f"👂 {name} listening to: {', '.join(channels)}")
    
    for message in pubsub.listen():
        if message['type'] == 'message':
            channel = message['channel'].decode('utf-8')
            content = message['data'].decode('utf-8')
            print(f"📻 {name} from {channel}: {content}")

# Example usage
def channel_demo():
    # Sports fan only wants sports news
    sports_fan = threading.Thread(
        target=specific_subscriber, 
        args=(["sports"], "Sports Fan")
    )
    sports_fan.daemon = True
    sports_fan.start()
    
    # Weather watcher wants weather and traffic
    weather_watcher = threading.Thread(
        target=specific_subscriber,
        args=(["weather", "traffic"], "Weather Watcher")
    )
    weather_watcher.daemon = True
    weather_watcher.start()
    
    time.sleep(1)
    multi_channel_demo()
    time.sleep(2)

channel_demo()
```

## Pattern Matching

You can subscribe to channels using patterns:

```redis
# Subscribe to all channels starting with "game"
127.0.0.1:6379> PSUBSCRIBE game*
Reading messages...

# This will receive messages from:
# - game1, game2, game3
# - game:level1, game:level2
# - gameroom, gameplay
```

Python example:

```python
def pattern_subscriber():
    """Subscribe to channels using patterns"""
    pubsub = r.pubsub()
    pubsub.psubscribe("game*")  # Listen to all channels starting with "game"
    
    print("👂 Listening to all game channels...")
    
    for message in pubsub.listen():
        if message['type'] == 'pmessage':  # Pattern message
            pattern = message['pattern'].decode('utf-8')
            channel = message['channel'].decode('utf-8')
            content = message['data'].decode('utf-8')
            print(f"🎮 Pattern {pattern} matched {channel}: {content}")

# Test pattern matching
def test_patterns():
    pattern_thread = threading.Thread(target=pattern_subscriber)
    pattern_thread.daemon = True
    pattern_thread.start()
    
    time.sleep(1)
    
    # Send to different game channels
    r.publish("game1", "Player joined!")
    r.publish("game2", "Level completed!")
    r.publish("gameroom", "New room created!")
    r.publish("sports", "This won't be received")  # Doesn't match pattern
    
    time.sleep(2)

test_patterns()
```

## When to Use Pub/Sub

**Good for:**

- **Real-time chat** (messages appear instantly)
- **Live notifications** (alerts to users)
- **Game events** (player actions, score updates)
- **System monitoring** (error alerts, status updates)

**Not good for:**

- **Storing data** (messages disappear if no one is listening)
- **Critical messages** (if subscriber is offline, they miss the message)
- **Message history** (you can't see old messages)

!!! note "Important Notes"
    ### Messages Don't Wait
    ```python
    # If you publish before anyone subscribes, message is lost
    r.publish("news", "This message disappears!")  # Nobody listening = lost

    # Subscribe first, then publish
    pubsub = r.pubsub()
    pubsub.subscribe("news")  # Now listening
    r.publish("news", "This message is received!")  # Gets delivered
    ```

    ### Subscribers Must Stay Connected
    ```python
    # If subscriber disconnects, they miss messages
    pubsub = r.pubsub()
    pubsub.subscribe("news")

    # If program crashes or disconnects here...
    # ...all messages sent during downtime are lost

    # When reconnecting, you only get NEW messages
    ```

## Summary

Pub/Sub is like a radio system for instant messaging:

1. **PUBLISH** sends messages to a channel
2. **SUBSCRIBE** listens to a channel
3. **Messages are instant** - no delay
4. **Messages don't wait** - if no one is listening, they disappear
5. **Multiple channels** allow organized topics
6. **Patterns** let you listen to multiple similar channels

**What you learned:**

- Basic pub/sub commands (`PUBLISH`, `SUBSCRIBE`)
- Python code for publishers and subscribers
- Multiple channels for different topics
- Pattern matching for channel groups
- When to use pub/sub vs other solutions

**Perfect for:** Real-time features where instant delivery matters more than message persistence.
