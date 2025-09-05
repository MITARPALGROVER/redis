# Redis Security

Redis security is like putting locks on your house. By default, Redis is like an open house - anyone can walk in. We need to add locks, passwords, and rules to keep our data safe.

## Why Redis Security Matters

Think of Redis like:

- **Bank Vault:** Needs multiple security layers
- **Office Building:** Needs keycards and access control
- **Safe Box:** Needs passwords and limited access
- **Private Diary:** Should only be readable by owner

**Without security:** Anyone can read, modify, or delete your data!

## Basic Security Setup

### 1. Enable Password Authentication

```bash
# redis.conf
requirepass mySecretPassword123

# Or set password via command
CONFIG SET requirepass "mySecretPassword123"
```

### 2. Disable Dangerous Commands

```bash
# redis.conf - Disable risky commands
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_HIDDEN_NAME_123"
rename-command EVAL ""
```

**What this does:**

- `requirepass` = Sets password required for all commands
- `rename-command FLUSHDB ""` = Completely disables FLUSHDB
- `rename-command CONFIG "SECRET"` = Renames CONFIG to SECRET

### 3. Network Security

```bash
# redis.conf - Limit network access
bind 127.0.0.1 192.168.1.100  # Only these IPs can connect
protected-mode yes              # Enable protected mode
port 6379                      # Change default port for security
```

## Python Security Examples

### Connecting with Password

```python
import redis

# Connect with password
secure_redis = redis.Redis(
    host='127.0.0.1',
    port=6379,
    password='mySecretPassword123',
    decode_responses=True
)

def test_authentication():
    """Test password authentication"""
    
    print("=== TESTING AUTHENTICATION ===")
    
    try:
        # This will work with correct password
        secure_redis.set("secure_data", "Top Secret Info")
        result = secure_redis.get("secure_data")
        print(f"✅ Authenticated successfully: {result}")
        
    except redis.AuthenticationError:
        print("❌ Wrong password!")
    except redis.ConnectionError:
        print("❌ Cannot connect to Redis")

def test_wrong_password():
    """Test what happens with wrong password"""
    
    print("\n=== TESTING WRONG PASSWORD ===")
    
    # Try with wrong password
    wrong_redis = redis.Redis(
        host='127.0.0.1',
        port=6379,
        password='wrongPassword',
        decode_responses=True
    )
    
    try:
        wrong_redis.get("test")
        print("⚠️  This shouldn't happen!")
    except redis.AuthenticationError:
        print("❌ Correctly rejected wrong password")

# Run authentication tests
test_authentication()
test_wrong_password()
```

**Sample Output:**
```
=== TESTING AUTHENTICATION ===
✅ Authenticated successfully: Top Secret Info

=== TESTING WRONG PASSWORD ===
❌ Correctly rejected wrong password
```

### User Management (Redis 6+)

```python
def setup_users():
    """Create different users with different permissions"""
    
    print("=== SETTING UP USERS ===")
    
    # Create read-only user
    secure_redis.acl_setuser(
        "reader",
        enabled=True,
        passwords=["reader123"],
        commands=["+get", "+mget", "+keys", "+exists"],
        keys=["public:*"]
    )
    print("✅ Created read-only user 'reader'")
    
    # Create write-only user for specific data
    secure_redis.acl_setuser(
        "writer", 
        enabled=True,
        passwords=["writer123"],
        commands=["+set", "+mset", "+del"],
        keys=["logs:*", "cache:*"]
    )
    print("✅ Created write-only user 'writer'")
    
    # List all users
    users = secure_redis.acl_list()
    print(f"📋 All users: {users}")

def test_user_permissions():
    """Test different user permissions"""
    
    print("\n=== TESTING USER PERMISSIONS ===")
    
    # Connect as read-only user
    reader = redis.Redis(
        host='127.0.0.1',
        port=6379,
        username='reader',
        password='reader123',
        decode_responses=True
    )
    
    # Connect as write user
    writer = redis.Redis(
        host='127.0.0.1',
        port=6379,
        username='writer', 
        password='writer123',
        decode_responses=True
    )
    
    # Set up some test data (as admin)
    secure_redis.set("public:info", "Everyone can read this")
    secure_redis.set("private:secret", "Only admin can read this")
    
    # Test reader permissions
    try:
        public_data = reader.get("public:info")
        print(f"📖 Reader can access public data: {public_data}")
    except redis.ResponseError as e:
        print(f"❌ Reader access denied: {e}")
    
    try:
        reader.get("private:secret")
        print("⚠️  Reader shouldn't access private data!")
    except redis.ResponseError:
        print("✅ Reader correctly denied private data")
    
    # Test writer permissions
    try:
        writer.set("cache:session", "user_session_data")
        print("✅ Writer can write to allowed keys")
    except redis.ResponseError as e:
        print(f"❌ Writer denied: {e}")

# Run user management (only works on Redis 6+)
try:
    setup_users()
    test_user_permissions()
except redis.ResponseError:
    print("ℹ️  User management requires Redis 6+ with ACL support")
```

**Sample Output:**
```
=== SETTING UP USERS ===
✅ Created read-only user 'reader'
✅ Created write-only user 'writer'
📋 All users: ['user default on +@all ~* &* >mySecretPassword123', 'user reader on +get +mget +keys +exists ~public:* >reader123', 'user writer on +set +mset +del ~logs:* ~cache:* >writer123']

=== TESTING USER PERMISSIONS ===
📖 Reader can access public data: Everyone can read this
✅ Reader correctly denied private data
✅ Writer can write to allowed keys
```

## SSL/TLS Encryption

### Enable TLS in Redis

```bash
# redis.conf - Enable TLS
port 0                    # Disable non-TLS port
tls-port 6380            # Enable TLS port
tls-cert-file redis.crt  # Certificate file
tls-key-file redis.key   # Private key file
tls-ca-cert-file ca.crt  # Certificate authority
```

### Connect with TLS in Python

```python
def connect_with_tls():
    """Connect to Redis using TLS encryption"""
    
    print("=== TLS CONNECTION ===")
    
    # Connect with TLS
    tls_redis = redis.Redis(
        host='127.0.0.1',
        port=6380,
        password='mySecretPassword123',
        ssl=True,
        ssl_cert_reqs='required',
        ssl_ca_certs='ca.crt',
        ssl_certfile='client.crt',
        ssl_keyfile='client.key',
        decode_responses=True
    )
    
    try:
        tls_redis.set("encrypted_data", "This travels encrypted")
        result = tls_redis.get("encrypted_data")
        print(f"🔒 TLS connection successful: {result}")
    except Exception as e:
        print(f"❌ TLS connection failed: {e}")

# Note: This requires proper TLS certificates
print("ℹ️  TLS example requires certificate setup")
```

## Security Best Practices

### 1. Input Validation

```python
def safe_user_lookup(user_id):
    """Safely lookup user data with validation"""
    
    # Validate input
    if not isinstance(user_id, str) or not user_id.isalnum():
        raise ValueError("Invalid user ID format")
    
    if len(user_id) > 50:
        raise ValueError("User ID too long")
    
    # Safe key construction
    safe_key = f"user:{user_id}"
    
    # Get data safely
    user_data = secure_redis.hgetall(safe_key)
    return user_data if user_data else None

def test_input_validation():
    """Test input validation"""
    
    print("=== INPUT VALIDATION ===")
    
    # Valid input
    try:
        user = safe_user_lookup("12345")
        print("✅ Valid input accepted")
    except ValueError as e:
        print(f"❌ Validation error: {e}")
    
    # Invalid input
    try:
        user = safe_user_lookup("user'; FLUSHALL; --")
        print("⚠️  Dangerous input should be rejected!")
    except ValueError:
        print("✅ Dangerous input correctly rejected")

test_input_validation()
```

**Sample Output:**
```
=== INPUT VALIDATION ===
✅ Valid input accepted
✅ Dangerous input correctly rejected
```

### 2. Rate Limiting for Security

```python
def security_rate_limit(client_ip, max_attempts=5, window_seconds=60):
    """Rate limit login attempts for security"""
    
    key = f"security:attempts:{client_ip}"
    current_attempts = secure_redis.get(key)
    
    if current_attempts and int(current_attempts) >= max_attempts:
        remaining_time = secure_redis.ttl(key)
        return False, f"Too many attempts. Try again in {remaining_time} seconds"
    
    # Increment attempts
    pipe = secure_redis.pipeline()
    pipe.incr(key)
    pipe.expire(key, window_seconds)
    pipe.execute()
    
    return True, "Attempt allowed"

def test_rate_limiting():
    """Test security rate limiting"""
    
    print("=== SECURITY RATE LIMITING ===")
    
    client_ip = "192.168.1.100"
    
    # Simulate multiple login attempts
    for attempt in range(1, 8):
        allowed, message = security_rate_limit(client_ip)
        
        if allowed:
            print(f"✅ Attempt {attempt}: {message}")
        else:
            print(f"❌ Attempt {attempt}: {message}")

test_rate_limiting()
```

**Sample Output:**
```
=== SECURITY RATE LIMITING ===
✅ Attempt 1: Attempt allowed
✅ Attempt 2: Attempt allowed
✅ Attempt 3: Attempt allowed
✅ Attempt 4: Attempt allowed
✅ Attempt 5: Attempt allowed
❌ Attempt 6: Too many attempts. Try again in 60 seconds
❌ Attempt 7: Too many attempts. Try again in 59 seconds
```

## Security Checklist

**Basic Security:**

- Set strong password with `requirepass`
- Disable dangerous commands
- Bind to specific IP addresses only
- Change default port
- Enable protected mode

**Advanced Security:**

- Use TLS encryption for network traffic
- Set up user accounts with specific permissions
- Validate all user inputs
- Implement rate limiting
- Monitor for suspicious activity

**Network Security:**

- Use firewall rules
- VPN for remote access
- Regular security updates

## Summary

Redis security protects your data from unauthorized access:

1. **Passwords** prevent unauthorized connections
2. **User accounts** limit what each user can do
3. **TLS encryption** protects data in transit
4. **Input validation** prevents attacks
5. **Rate limiting** stops brute force attempts

**What you learned:**

- Password authentication setup
- User permission management
- TLS encryption basics
- Input validation techniques
- Security best practices

**Perfect for:** Any Redis deployment handling sensitive data or exposed to networks.
