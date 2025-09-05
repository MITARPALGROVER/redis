# Session Storage

Session storage is like keeping track of who is using your website and what they are doing. Think of it like giving each visitor a name tag when they enter your store.

## What is a Session?

When someone visits your website, you give them a special ID (like a ticket number). This ID helps you remember:
- Who they are
- What they put in their shopping cart
- If they are logged in
- What page they were on

## Simple Session Example

### Step 1: Create a Session When User Logs In

```redis
# User logs in, create session
127.0.0.1:6379> SET session:abc123 "user_id:456"
OK

# Session expires in 30 minutes if not used
127.0.0.1:6379> EXPIRE session:abc123 1800
(integer) 1
```

### Step 2: Check Session on Each Page

```redis
# Check if session exists
127.0.0.1:6379> GET session:abc123
"user_id:456"

# If session exists, user is logged in
# If session is nil, user needs to log in again
```

### Step 3: Update Session Activity

```redis
# User does something, refresh session time
127.0.0.1:6379> EXPIRE session:abc123 1800
(integer) 1
```

## Store More Session Data

You can store more information about the user:

```redis
# Store multiple pieces of information
127.0.0.1:6379> HSET session:abc123 user_id 456
(integer) 1
127.0.0.1:6379> HSET session:abc123 username "john"
(integer) 1
127.0.0.1:6379> HSET session:abc123 role "admin"
(integer) 1
127.0.0.1:6379> HSET session:abc123 login_time "2025-01-01 10:00:00"
(integer) 1

# Set expiration for the whole session
127.0.0.1:6379> EXPIRE session:abc123 1800
(integer) 1

# Get session data
127.0.0.1:6379> HGETALL session:abc123
1) "user_id"
2) "456"
3) "username" 
4) "john"
5) "role"
6) "admin"
7) "login_time"
8) "2025-01-01 10:00:00"
```

## Simple Python Session Example

```python
import redis
import uuid
import time

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

def create_session(user_id, username):
    # Create unique session ID
    session_id = str(uuid.uuid4())
    
    # Store session data
    session_key = f"session:{session_id}"
    r.hset(session_key, mapping={
        "user_id": user_id,
        "username": username,
        "created_at": time.time()
    })
    
    # Session expires in 30 minutes
    r.expire(session_key, 1800)
    
    return session_id

def get_session(session_id):
    # Get session data
    session_key = f"session:{session_id}"
    session_data = r.hgetall(session_key)
    
    if not session_data:
        return None  # Session doesn't exist or expired
    
    # Convert bytes to strings
    session = {}
    for key, value in session_data.items():
        session[key.decode('utf-8')] = value.decode('utf-8')
    
    # Refresh session (extend expiration)
    r.expire(session_key, 1800)
    
    return session

def delete_session(session_id):
    # User logs out, delete session
    session_key = f"session:{session_id}"
    r.delete(session_key)

# Example usage
# User logs in
session_id = create_session("123", "alice")
print(f"Created session: {session_id}")

# Check session later
session_data = get_session(session_id)
print(f"Session data: {session_data}")

# User logs out
delete_session(session_id)
```

## Shopping Cart Session

You can store shopping cart items in the session:

```redis
# Add items to cart
127.0.0.1:6379> HSET session:abc123 cart "item1,item2,item3"
(integer) 1

# Add user preferences
127.0.0.1:6379> HSET session:abc123 language "english"
(integer) 1
127.0.0.1:6379> HSET session:abc123 theme "dark"
(integer) 1

# Get cart contents
127.0.0.1:6379> HGET session:abc123 cart
"item1,item2,item3"
```

## Session Security

### Make Session IDs Hard to Guess

```python
import secrets

def create_secure_session(user_id):
    # Create random, hard-to-guess session ID
    session_id = secrets.token_urlsafe(32)
    
    session_key = f"session:{session_id}"
    r.hset(session_key, mapping={
        "user_id": user_id,
        "created_at": time.time(),
        "ip_address": "192.168.1.1"  # Store user's IP
    })
    
    r.expire(session_key, 1800)
    return session_id
```

### Check Session IP Address

```python
def validate_session(session_id, current_ip):
    session_key = f"session:{session_id}"
    session_data = r.hgetall(session_key)
    
    if not session_data:
        return None
    
    # Check if IP address matches
    stored_ip = session_data.get(b'ip_address', b'').decode('utf-8')
    if stored_ip != current_ip:
        # IP changed, delete session for security
        r.delete(session_key)
        return None
    
    return session_data
```

## Multiple Device Sessions

Let one user log in from multiple devices:

```redis
# User logs in from phone
127.0.0.1:6379> HSET user:456:sessions phone "session:abc123"
(integer) 1

# User logs in from computer
127.0.0.1:6379> HSET user:456:sessions computer "session:def456"
(integer) 1

# See all user's active sessions
127.0.0.1:6379> HGETALL user:456:sessions
1) "phone"
2) "session:abc123"
3) "computer"
4) "session:def456"
```

## Clean Up Old Sessions

Remove sessions that are too old:

```python
def cleanup_old_sessions():
    # Find all session keys
    session_keys = r.keys("session:*")
    
    current_time = time.time()
    cleaned = 0
    
    for key in session_keys:
        # Check if session is older than 24 hours
        session_data = r.hgetall(key)
        if session_data:
            created_at = float(session_data.get(b'created_at', 0))
            if (current_time - created_at) > 86400:  # 24 hours
                r.delete(key)
                cleaned += 1
    
    print(f"Cleaned up {cleaned} old sessions")

# Run cleanup
cleanup_old_sessions()
```

## Simple Session Management

```python
class SimpleSession:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.timeout = 1800  # 30 minutes
    
    def login(self, user_id, username):
        """Create session when user logs in"""
        session_id = str(uuid.uuid4())
        session_key = f"session:{session_id}"
        
        self.redis.hset(session_key, mapping={
            "user_id": user_id,
            "username": username
        })
        self.redis.expire(session_key, self.timeout)
        
        return session_id
    
    def get_user(self, session_id):
        """Get user info from session"""
        session_key = f"session:{session_id}"
        user_data = self.redis.hgetall(session_key)
        
        if user_data:
            # Refresh session
            self.redis.expire(session_key, self.timeout)
            return {
                "user_id": user_data[b'user_id'].decode('utf-8'),
                "username": user_data[b'username'].decode('utf-8')
            }
        return None
    
    def logout(self, session_id):
        """Delete session when user logs out"""
        session_key = f"session:{session_id}"
        self.redis.delete(session_key)

# Usage
sessions = SimpleSession(r)

# User logs in
session_id = sessions.login("123", "alice")

# Check if user is logged in
user = sessions.get_user(session_id)
if user:
    print(f"User {user['username']} is logged in")

# User logs out
sessions.logout(session_id)
```

## Summary

Session storage with Redis is like giving each visitor a temporary name tag that remembers who they are and what they're doing on your website.

Key points:

- Each user gets a unique session ID
- Store user information with that ID
- Set expiration time so old sessions disappear
- Refresh session time when user is active
- Delete session when user logs out

This keeps your website secure and remembers users without asking them to log in repeatedly!
