# Hash Operations

Redis hashes are like mini-databases or objects - perfect for storing structured data with multiple fields. Think of them as user profiles, product details, or configuration objects where you need to store related information together.

## What Are Redis Hashes?

Think of Redis hashes as:

- A **user profile** with name, age, email, city
- A **product record** with title, price, description, category
- A **configuration object** with multiple settings
- A **database row** with different columns

Unlike storing JSON as a string, hashes let you update individual fields efficiently!

---

## Basic Hash Operations

### Setting and Getting Hash Fields

```redis
# Set individual fields in a hash
127.0.0.1:6379> HSET user:1001 name "Alice"
(integer) 1
# 1 = new field created

127.0.0.1:6379> HSET user:1001 age "28"
(integer) 1

127.0.0.1:6379> HSET user:1001 email "alice@example.com"
(integer) 1

# Set multiple fields at once (more efficient)
127.0.0.1:6379> HSET user:1002 name "Bob" age "32" city "New York" status "active"
(integer) 4
# 4 = four new fields created

# Get individual field
127.0.0.1:6379> HGET user:1001 name
"Alice"

127.0.0.1:6379> HGET user:1001 age
"28"

# Get multiple fields at once
127.0.0.1:6379> HMGET user:1002 name age city
1) "Bob"
2) "32"
3) "New York"

# Get field that doesn't exist
127.0.0.1:6379> HGET user:1001 phone
(nil)
```

### Viewing Complete Hashes

```redis
# Get all fields and values
127.0.0.1:6379> HGETALL user:1001
1) "name"
2) "Alice"
3) "age"
4) "28"
5) "email"
6) "alice@example.com"
# Format: field1, value1, field2, value2, ...

# Get only field names
127.0.0.1:6379> HKEYS user:1002
1) "name"
2) "age"
3) "city"
4) "status"

# Get only values
127.0.0.1:6379> HVALS user:1002
1) "Bob"
2) "32"
3) "New York"
4) "active"

# Count number of fields
127.0.0.1:6379> HLEN user:1001
(integer) 3

127.0.0.1:6379> HLEN user:1002
(integer) 4
```

### Checking and Managing Fields

```redis
# Check if field exists
127.0.0.1:6379> HEXISTS user:1001 name
(integer) 1
# 1 = field exists

127.0.0.1:6379> HEXISTS user:1001 phone
(integer) 0
# 0 = field doesn't exist

# Delete specific fields
127.0.0.1:6379> HDEL user:1002 status
(integer) 1
# 1 = field was deleted

127.0.0.1:6379> HDEL user:1002 nonexistent
(integer) 0
# 0 = field didn't exist

# Delete multiple fields
127.0.0.1:6379> HSET user:1003 name "Charlie" age "25" temp1 "delete_me" temp2 "delete_me_too"
(integer) 4

127.0.0.1:6379> HDEL user:1003 temp1 temp2
(integer) 2
# Deleted 2 fields

127.0.0.1:6379> HKEYS user:1003
1) "name"
2) "age"
```

---

## Conditional Hash Operations

### Set Only If Field Doesn't Exist

```redis
# Set field only if it doesn't already exist
127.0.0.1:6379> HSETNX user:1001 phone "555-1234"
(integer) 1
# 1 = field was set (didn't exist before)

127.0.0.1:6379> HSETNX user:1001 phone "555-9999"
(integer) 0
# 0 = field not set (already exists)

127.0.0.1:6379> HGET user:1001 phone
"555-1234"
# Original value unchanged

# Useful for default values
127.0.0.1:6379> HSETNX user:1001 status "active"
(integer) 1
# Set default status

127.0.0.1:6379> HSETNX user:1001 status "inactive"
(integer) 0
# Won't change existing status
```

---

## Working with Numbers in Hashes

### Incrementing Numeric Fields

```redis
# Set initial numeric values
127.0.0.1:6379> HSET stats:user123 login_count "10"
(integer) 1

127.0.0.1:6379> HSET stats:user123 points "1500"
(integer) 1

# Increment by 1
127.0.0.1:6379> HINCRBY stats:user123 login_count 1
(integer) 11

# Increment by specific amount
127.0.0.1:6379> HINCRBY stats:user123 points 50
(integer) 1550

# Increment field that doesn't exist (starts at 0)
127.0.0.1:6379> HINCRBY stats:user123 post_count 1
(integer) 1

127.0.0.1:6379> HGETALL stats:user123
1) "login_count"
2) "11"
3) "points"
4) "1550"
5) "post_count"
6) "1"

# Work with decimal numbers
127.0.0.1:6379> HSET product:123 price "19.99"
(integer) 1

127.0.0.1:6379> HINCRBYFLOAT product:123 price 5.50
"25.49"

127.0.0.1:6379> HINCRBYFLOAT product:123 price -2.00
"23.49"

# Negative increments (decrement)
127.0.0.1:6379> HINCRBY stats:user123 points -100
(integer) 1450
```

---

## Real-World Hash Use Cases

### 1. User Profiles

```redis
# Complete user profile
127.0.0.1:6379> HSET user:alice \
  name "Alice Johnson" \
  email "alice@example.com" \
  age "28" \
  city "San Francisco" \
  country "USA" \
  joined "2024-01-15" \
  status "active" \
  verified "true"
(integer) 8

# Update specific fields
127.0.0.1:6379> HSET user:alice age "29" city "Los Angeles"
(integer) 0
# 0 = updated existing fields

# Get profile summary
127.0.0.1:6379> HMGET user:alice name email city status
1) "Alice Johnson"
2) "alice@example.com"
3) "Los Angeles"
4) "active"

# Update verification status
127.0.0.1:6379> HSET user:alice verified "true" verification_date "2024-02-01"
(integer) 1
# 1 = created new field (verification_date)

# Check if user is verified
127.0.0.1:6379> HGET user:alice verified
"true"

# Get full profile
127.0.0.1:6379> HGETALL user:alice
 1) "name"
 2) "Alice Johnson"
 3) "email"
 4) "alice@example.com"
 5) "age"
 6) "29"
 7) "city"
 8) "Los Angeles"
 9) "country"
10) "USA"
11) "joined"
12) "2024-01-15"
13) "status"
14) "active"
15) "verified"
16) "true"
17) "verification_date"
18) "2024-02-01"
```

### 2. Product Catalog

```redis
# Product information
127.0.0.1:6379> HSET product:laptop001 \
  name "MacBook Pro 14-inch" \
  brand "Apple" \
  price "1999.00" \
  category "Electronics" \
  subcategory "Laptops" \
  stock "25" \
  weight "3.5" \
  color "Space Gray" \
  warranty "1 year"
(integer) 9

# Update stock when item is sold
127.0.0.1:6379> HINCRBY product:laptop001 stock -1
(integer) 24

# Apply discount
127.0.0.1:6379> HINCRBYFLOAT product:laptop001 price -200.00
"1799"

# Get product display info
127.0.0.1:6379> HMGET product:laptop001 name brand price stock
1) "MacBook Pro 14-inch"
2) "Apple"
3) "1799"
4) "24"

# Check if product is in stock
127.0.0.1:6379> HGET product:laptop001 stock
"24"

# Add product reviews summary
127.0.0.1:6379> HSET product:laptop001 \
  review_count "145" \
  avg_rating "4.7" \
  last_review "2024-01-20"
(integer) 3

# Update review stats
127.0.0.1:6379> HINCRBY product:laptop001 review_count 1
(integer) 146

127.0.0.1:6379> HSET product:laptop001 avg_rating "4.8" last_review "2024-01-21"
(integer) 0
```

### 3. Session Storage

```redis
# User session data
127.0.0.1:6379> HSET session:abc123 \
  user_id "1001" \
  username "alice" \
  role "admin" \
  login_time "2024-01-21T10:30:00Z" \
  last_activity "2024-01-21T11:45:00Z" \
  ip_address "192.168.1.100" \
  user_agent "Mozilla/5.0..."
(integer) 7

# Update last activity
127.0.0.1:6379> HSET session:abc123 last_activity "2024-01-21T12:00:00Z"
(integer) 0

# Check session validity
127.0.0.1:6379> HEXISTS session:abc123 user_id
(integer) 1

# Get session user info
127.0.0.1:6379> HMGET session:abc123 user_id username role
1) "1001"
2) "alice"
3) "admin"

# Add session expiration (hash itself expires)
127.0.0.1:6379> EXPIRE session:abc123 3600
(integer) 1
# Session expires in 1 hour

# Extend session
127.0.0.1:6379> EXPIRE session:abc123 7200
(integer) 1
# Now expires in 2 hours
```

### 4. Application Configuration

```redis
# Application settings
127.0.0.1:6379> HSET config:app \
  debug_mode "false" \
  max_users "1000" \
  session_timeout "3600" \
  upload_limit "10485760" \
  default_theme "light" \
  maintenance_mode "false" \
  api_rate_limit "100"
(integer) 7

# Get specific config values
127.0.0.1:6379> HMGET config:app debug_mode max_users session_timeout
1) "false"
2) "1000"
3) "3600"

# Update configuration
127.0.0.1:6379> HSET config:app debug_mode "true" max_users "2000"
(integer) 0

# Check if maintenance mode is on
127.0.0.1:6379> HGET config:app maintenance_mode
"false"

# Enable maintenance mode
127.0.0.1:6379> HSET config:app maintenance_mode "true"
(integer) 0

# Get all configuration
127.0.0.1:6379> HGETALL config:app
 1) "debug_mode"
 2) "true"
 3) "max_users"
 4) "2000"
 5) "session_timeout"
 6) "3600"
 7) "upload_limit"
 8) "10485760"
 9) "default_theme"
10) "light"
11) "maintenance_mode"
12) "true"
13) "api_rate_limit"
14) "100"
```

### 5. Analytics and Counters

```redis
# Website analytics for today
127.0.0.1:6379> HSET analytics:2024-01-21 \
  page_views "0" \
  unique_visitors "0" \
  new_users "0" \
  bounce_rate "0.0" \
  avg_time_on_site "0.0"
(integer) 5

# Track page view
127.0.0.1:6379> HINCRBY analytics:2024-01-21 page_views 1
(integer) 1

# Track unique visitor
127.0.0.1:6379> HINCRBY analytics:2024-01-21 unique_visitors 1
(integer) 1

# Update averages
127.0.0.1:6379> HINCRBYFLOAT analytics:2024-01-21 avg_time_on_site 45.5
"45.5"

# Get daily summary
127.0.0.1:6379> HGETALL analytics:2024-01-21
 1) "page_views"
 2) "1"
 3) "unique_visitors"
 4) "1"
 5) "new_users"
 6) "0"
 7) "bounce_rate"
 8) "0.0"
 9) "avg_time_on_site"
10) "45.5"

# User-specific analytics
127.0.0.1:6379> HSET user_stats:alice \
  posts_created "5" \
  comments_made "23" \
  likes_received "47" \
  profile_views "156"
(integer) 4

# User creates a post
127.0.0.1:6379> HINCRBY user_stats:alice posts_created 1
(integer) 6

127.0.0.1:6379> HINCRBY user_stats:alice likes_received 3
(integer) 50
```

---

## Advanced Hash Patterns

### 1. Hash as Database Row

```redis
# Order record
127.0.0.1:6379> HSET order:12345 \
  customer_id "1001" \
  status "pending" \
  total "156.78" \
  items "3" \
  created "2024-01-21T10:00:00Z" \
  updated "2024-01-21T10:00:00Z" \
  shipping_address "123 Main St, City, State"
(integer) 7

# Update order status
127.0.0.1:6379> HSET order:12345 \
  status "processing" \
  updated "2024-01-21T11:30:00Z"
(integer) 0

# Add tracking info when shipped
127.0.0.1:6379> HSET order:12345 \
  status "shipped" \
  tracking_number "1Z999AA1234567890" \
  updated "2024-01-21T14:00:00Z"
(integer) 1
# 1 = tracking_number is new field

# Get order summary
127.0.0.1:6379> HMGET order:12345 customer_id status total tracking_number
1) "1001"
2) "shipped"
3) "156.78"
4) "1Z999AA1234567890"
```

### 2. Nested Data Structure Simulation

```redis
# User preferences (simulating nested objects)
127.0.0.1:6379> HSET user:alice:prefs \
  theme "dark" \
  language "en" \
  timezone "America/New_York" \
  notifications_email "true" \
  notifications_push "false" \
  privacy_profile_public "false" \
  privacy_show_online "true"
(integer) 7

# Update notification preferences
127.0.0.1:6379> HSET user:alice:prefs \
  notifications_email "false" \
  notifications_sms "true"
(integer) 1
# 1 = notifications_sms is new

# Get all preferences
127.0.0.1:6379> HGETALL user:alice:prefs

# Get specific preference category (using pattern)
127.0.0.1:6379> HMGET user:alice:prefs \
  notifications_email \
  notifications_push \
  notifications_sms
1) "false"
2) "false"
3) "true"
```

### 3. Hash Expiration Patterns

```redis
# Temporary cache with expiration
127.0.0.1:6379> HSET cache:weather:london \
  temperature "22" \
  humidity "65" \
  condition "sunny" \
  wind_speed "5" \
  updated "2024-01-21T12:00:00Z"
(integer) 5

# Cache expires in 10 minutes
127.0.0.1:6379> EXPIRE cache:weather:london 600
(integer) 1

# Check if cache is still valid
127.0.0.1:6379> TTL cache:weather:london
(integer) 567
# 567 seconds remaining

# Get cached weather
127.0.0.1:6379> HMGET cache:weather:london temperature condition
1) "22"
2) "sunny"
```

---

## Performance Tips

### 1. Efficient Field Operations

```redis
# Fast: Get specific fields you need
HMGET user:1001 name email status

# Slower: Get everything when you only need few fields
HGETALL user:1001

# Fast: Set multiple fields at once
HSET user:1001 name "Alice" age "28" city "NYC"

# Slower: Multiple separate operations
HSET user:1001 name "Alice"
HSET user:1001 age "28"
HSET user:1001 city "NYC"
```

### 2. Memory Optimization

```redis
# Good: Use short field names for large datasets
HSET u:1001 n "Alice" a "28" e "alice@ex.com"

# Wasteful: Long field names with many hashes
HSET user:1001 full_name "Alice" age_in_years "28" email_address "alice@ex.com"

# Good: Store only necessary data
HSET product:123 name "iPhone" price "999" stock "50"

# Wasteful: Store redundant or derived data
HSET product:123 name "iPhone" price "999" price_with_tax "1099" stock "50"
```

### 3. Field Naming Conventions

```redis
# Consistent naming
HSET user:1001 first_name "Alice" last_name "Johnson" email "alice@example.com"

# Grouped field names
HSET user:1001 \
  addr_street "123 Main St" \
  addr_city "NYC" \
  addr_zip "10001" \
  pref_theme "dark" \
  pref_lang "en"

# Use consistent patterns across your application
```

---

## Practice Exercises

### Exercise 1: E-commerce Product

```redis
# Create a product with all details
HSET product:shoe001 \
  name "Running Shoes" \
  brand "Nike" \
  price "129.99" \
  sizes "7,8,9,10,11" \
  colors "black,white,red" \
  stock_total "150" \
  category "footwear"

# Customer buys 2 pairs
HINCRBY product:shoe001 stock_total -2

# Apply 10% discount
HINCRBYFLOAT product:shoe001 price -13.00

# Check current status
HMGET product:shoe001 name price stock_total
```

### Exercise 2: User Activity Tracking

```redis
# Initialize user activity
HSET activity:user123 \
  posts "0" \
  comments "0" \
  likes_given "0" \
  likes_received "0" \
  last_login "2024-01-21"

# User activity throughout the day
HINCRBY activity:user123 posts 1
HINCRBY activity:user123 comments 3
HINCRBY activity:user123 likes_given 5
HINCRBY activity:user123 likes_received 8
HSET activity:user123 last_login "2024-01-21T15:30:00Z"

# Get activity summary
HGETALL activity:user123
```

### Exercise 3: Game Player Stats

```redis
# Player profile
HSET player:gamer123 \
  username "ProGamer" \
  level "45" \
  experience "134567" \
  coins "2500" \
  lives "3" \
  high_score "987654"

# Player completes a level
HINCRBY player:gamer123 experience 1500
HINCRBY player:gamer123 coins 100
HSET player:gamer123 level "46"

# Player loses a life
HINCRBY player:gamer123 lives -1

# Check if new high score
HGET player:gamer123 high_score
# In app: compare with current score and update if higher
```

### Exercise 4: Article Metadata

```redis
# Article information
HSET article:123 \
  title "Redis Hash Guide" \
  author "TechWriter" \
  category "Technology" \
  published "2024-01-21" \
  views "0" \
  likes "0" \
  comments "0" \
  status "published"

# Track article engagement
HINCRBY article:123 views 1
HINCRBY article:123 likes 1

# Someone comments
HINCRBY article:123 comments 1

# Get article stats
HMGET article:123 title views likes comments
```

---

## Hash Commands Quick Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `HSET key field value` | Set field(s) | `HSET user name "Alice"` |
| `HGET key field` | Get field value | `HGET user name` |
| `HMGET key field1 field2` | Get multiple fields | `HMGET user name age` |
| `HGETALL key` | Get all fields and values | `HGETALL user` |
| `HKEYS key` | Get all field names | `HKEYS user` |
| `HVALS key` | Get all values | `HVALS user` |
| `HEXISTS key field` | Check if field exists | `HEXISTS user email` |
| `HLEN key` | Get number of fields | `HLEN user` |
| `HDEL key field` | Delete field(s) | `HDEL user temp_field` |
| `HSETNX key field value` | Set if field doesn't exist | `HSETNX user phone "555-1234"` |
| `HINCRBY key field increment` | Increment numeric field | `HINCRBY stats count 1` |
| `HINCRBYFLOAT key field increment` | Increment float field | `HINCRBYFLOAT product price 5.99` |

---

## What's Next?

Ready to learn about **Sorted Sets** - ranked collections perfect for leaderboards, rankings, and time-series data!
