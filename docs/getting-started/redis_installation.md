# Installing Redis

Ready to get Redis running on your system? Choose your platform below:

!!! tip "Recommended for Beginners"
    If you're completely new to Redis, I recommend starting with **Docker** - it's the most consistent across all systems and easiest to manage.

=== "Docker"

    ## Docker Installation (Recommended)

    ### Why Docker?
    - **Works on any system** (Windows, macOS, Linux)
    - **No complex setup** - one command to get started
    - **Easy to remove** if you want to uninstall later
    - **Matches production environments**

    ### Prerequisites
    - Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) for your system

    ### Installation Steps

    1. **Pull the Redis image**:
       ```bash
       docker pull redis:latest
       ```

    2. **Run Redis**:
    (dont worry, will explain these commands further)
       ```bash
       docker run --name my-redis -p 6379:6379 -d redis:latest
       ```

    3. **Connect to Redis**:
       ```bash
       docker exec -it my-redis redis-cli
       ```

    4. **Test it works**:
       ```redis
       127.0.0.1:6379> ping
       PONG
       127.0.0.1:6379> set hello "world"
       OK
       127.0.0.1:6379> get hello
       "world"
       ```

    !!! success "Success!"
        If you see `PONG` when you type `ping`, Redis is working perfectly!

    ### Docker Useful Commands

    ```bash
    # Start Redis (if stopped)
    docker start my-redis

    # Stop Redis
    docker stop my-redis

    # Check if Redis is running
    docker ps

    # View Redis logs
    docker logs my-redis

    # Remove Redis (if you want to uninstall)
    docker rm -f my-redis
    ```

=== "Windows"

    ## Windows Installation

    ### Method A: Using Windows Subsystem for Linux (WSL) - Recommended

    1. **Install WSL2** (if not already installed):
       ```powershell
       wsl --install
       ```

    2. **Open WSL terminal** and follow the Linux Ubuntu instructions in the Linux tab.

    ### Method B: Using Redis for Windows

    !!! warning "Note"
        Microsoft maintains a Windows port, but it's not officially supported by Redis.

    1. **Download** from [Redis for Windows releases](https://github.com/tporadowski/redis/releases)

    2. **Extract** the ZIP file to `C:\Redis`

    3. **Add to PATH**:
       - Open System Properties → Advanced → Environment Variables
       - Add `C:\Redis` to your PATH

    4. **Start Redis**:
       ```cmd
       redis-server
       ```

    5. **Connect** (in a new command prompt):
       ```cmd
       redis-cli
       ```

=== "macOS"

    ## macOS Installation

    ### Method A: Using Homebrew (Recommended)

    1. **Install Homebrew** (if not installed):
       ```bash
       /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
       ```

    2. **Install Redis**:
       ```bash
       brew install redis
       ```

    3. **Start Redis**:
       ```bash
       brew services start redis
       ```

    4. **Connect to Redis**:
       ```bash
       redis-cli
       ```

    ### Method B: Manual Installation

    1. **Install Xcode command line tools**:
       ```bash
       xcode-select --install
       ```

    2. **Download and compile Redis**:
       ```bash
       wget https://download.redis.io/redis-stable.tar.gz
       tar xzf redis-stable.tar.gz
       cd redis-stable
       make
       ```

    3. **Install Redis**:
       ```bash
       sudo make install
       ```

    ### macOS Management Commands

    ```bash
    # Start Redis as a service
    brew services start redis

    # Stop Redis service
    brew services stop redis

    # Restart Redis
    brew services restart redis
    ```

=== "Linux"

    ## Linux Installation

    ### Ubuntu/Debian

    1. **Update package list**:
       ```bash
       sudo apt update
       ```

    2. **Install Redis**:
       ```bash
       sudo apt install redis-server
       ```

    3. **Start Redis service**:
       ```bash
       sudo systemctl start redis-server
       ```

    4. **Enable auto-start**:
       ```bash
       sudo systemctl enable redis-server
       ```

    5. **Test connection**:
       ```bash
       redis-cli ping
       ```

    ### CentOS/RHEL/Fedora

    1. **Install EPEL repository** (CentOS/RHEL only):
       ```bash
       sudo yum install epel-release
       ```

    2. **Install Redis**:
       ```bash
       # CentOS/RHEL
       sudo yum install redis
       
       # Fedora
       sudo dnf install redis
       ```

    3. **Start and enable Redis**:
       ```bash
       sudo systemctl start redis
       sudo systemctl enable redis
       ```

    ### Linux Management Commands

    ```bash
    # Check Redis status
    sudo systemctl status redis

    # Start Redis
    sudo systemctl start redis

    # Stop Redis
    sudo systemctl stop redis

    # Restart Redis
    sudo systemctl restart redis

    # View Redis logs
    sudo journalctl -u redis
    ```

---

## Verification Steps

After installation, let's verify everything works correctly:

### 1. Check Redis is Running

```bash
redis-cli ping
```
**OR**

```bash title="For Docker"
docker exec -it my-redis redis-cli ping 
#everytime for docker this command will be used
```

**Expected output**: `PONG`

### 2. Basic Functionality Test

```redis
# Connect to Redis
redis-cli 
#for docker use: docker exec -it my-redis redis-cli

# Set a value
127.0.0.1:6379> SET test "Hello Redis"
#output: OK

# Get the value
127.0.0.1:6379> GET test
#output: "Hello Redis"

# Check Redis info
127.0.0.1:6379> INFO server
```

### 3. Performance Test

Let's test Redis with some basic commands to see how it handles data:

```redis
# Create a counter and set it to 0
127.0.0.1:6379> SET counter 0
OK

# Increase the counter by 1 (increment)
127.0.0.1:6379> INCR counter
(integer) 1

# Increase the counter by 1 again
127.0.0.1:6379> INCR counter
(integer) 2
```

**What these commands do:**

- `SET counter 0`: Creates a key called "counter" and sets its value to 0
- `INCR counter`: Automatically increases the counter value by 1 and returns the new value
- Redis responds with `(integer) 1` and `(integer) 2` showing the updated counter values

**Why this is a good test:** 

- This shows Redis can store data (`SET`) and perform mathematical operations (`INCR`) quickly 
- Perfect for things like counting website visits or tracking scores in games.

!!! success "Installation Complete!"
    If all the above commands work, congratulations! Redis is successfully installed and running.

---

## Basic Configuration

### Default Settings
- **Port**: 6379 - This is the network port Redis listens on (like a door number for your Redis server)
- **Host**: 127.0.0.1 (localhost) - This means Redis only accepts connections from your own computer
- **Configuration file**: Where Redis stores its settings
  - Linux: `/etc/redis/redis.conf`
  - macOS: `/usr/local/etc/redis.conf`
  - Docker: Inside the container

### Important First Steps

#### 1. Set a Password (Security)
**Why:** By default, Redis has no password - anyone who can connect can access your data!

```redis
# Set a password for your Redis server
CONFIG SET requirepass your-password-here

# After setting a password, you must authenticate to use Redis
AUTH your-password-here
```

**What happens:**

- `CONFIG SET requirepass`: Tells Redis to require a password for all future connections
- `AUTH`: Proves you know the password so you can run commands
- **Important:** Replace "your-password-here" with a strong password like "RedisGuide@2447"

**Real-world use:** Essential for production servers to prevent unauthorized access to your data.

#### 2. Check Memory Usage
**Why:** Redis stores everything in memory, so you need to monitor how much it's using.

```redis
INFO memory
```

**What this shows:**

- How much RAM Redis is currently using
- Maximum memory limit (if set)
- Memory efficiency statistics

**Example output you might see:**
```
used_memory_human:1.2M    # Redis is using 1.2 megabytes
maxmemory_human:0B        # No memory limit set
```

**Real-world use:** Monitor this to ensure Redis doesn't use all your server's memory.

#### 3. See All Configuration
**Why:** Sometimes you need to check Redis settings or troubleshoot issues.

```redis
CONFIG GET "*"
```

**What this does:**

- Shows **every** Redis configuration setting
- Displays current values for all options
- Helps you understand how Redis is configured

**Example of what you'll see:**
```
1) "timeout"
2) "0"
3) "port"
4) "6379"
5) "requirepass"
6) "your-password-here"
```

**Real-world use:** Debugging connection issues, verifying security settings, or checking performance configurations.

!!! tip "Pro Tips"
    - **Security:** Always set a password in production environments
    - **Memory:** Keep an eye on memory usage - Redis can fill up your RAM quickly with large datasets

---

## Troubleshooting Common Issues

### Redis Won't Start

#### Problem 1: `redis-server` command not found
**What this means:** Your computer doesn't know where Redis is installed.

**Solution:** Check if Redis is in your PATH or use the full path

**How to fix:**
```bash
# Try finding Redis manually
which redis-server    # On macOS/Linux
where redis-server     # On Windows

# If found, use the full path
/usr/local/bin/redis-server   # Example full path
```

#### Problem 2: Port 6379 already in use
**What this means:** Something else is already using Redis's default port (like another Redis instance or different program).

**Solutions:**
```bash
# Option 1: Find what's using the port and stop it
sudo lsof -i :6379                    # Shows what's using port 6379
kill -9 [process-id]                  # Stop the process (replace [process-id] with actual number)

# Option 2: Start Redis on a different port
redis-server --port 6380              # Use port 6380 instead
```

**What these commands do:**

- `lsof -i :6379`: Lists all processes using port 6379
- `kill -9`: Forcefully stops a process
- `--port 6380`: Tells Redis to use port 6380 instead of the default 6379

### Connection Issues

#### Problem: Can't connect with `redis-cli`
**What this means:** Redis CLI can't talk to the Redis server (server might be stopped or crashed).

**Step-by-step diagnosis:**

**Step 1:** Check if Redis is actually running
```bash
# See all Redis processes
ps aux | grep redis

# What you should see:
# redis-server *:6379    (this means Redis is running on port 6379)
# If you see nothing, Redis isn't running!
```

**Step 2:** Check the service status (Linux only)
```bash
# Check if Redis service is active
sudo systemctl status redis

# Possible outputs:
# Active (running) = Redis is working fine
# Inactive (dead) = Redis has stopped
# Failed = Something went wrong
```

**Step 3:** Try to restart Redis
```bash
# Linux
sudo systemctl restart redis

# macOS
brew services restart redis

# Docker
docker restart my-redis
```

### Permission Issues (Linux)

#### Problem: Permission denied errors
**What this means:** Redis doesn't have the right permissions to read/write its files.

**Common error messages:**

- "Permission denied"
- "Can't save DB"
- "Can't open log file"

**Solution:** Fix Redis directory permissions
```bash
# Give Redis user ownership of its data directory
sudo chown redis:redis /var/lib/redis

# Set proper permissions (755 = read/write for owner, read for others)
sudo chmod 755 /var/lib/redis
```

**What these commands do:**

- `chown redis:redis`: Makes the 'redis' user the owner of the directory
- `chmod 755`: Sets permissions so Redis can read and write files

### Docker Issues

#### Problem: Docker container won't start
**What this means:** Something is preventing your Redis Docker container from running.

**Step-by-step troubleshooting:**

**Step 1:** Check the error logs
```bash
# See what went wrong
docker logs my-redis

# Common error messages:
# "port already in use" = Something else is using port 6379
# "no space left" = Your computer is out of disk space
# "container already exists" = A container with this name already exists
```

**Step 2:** Remove and recreate the container
```bash
# Stop and completely remove the old container
docker rm -f my-redis

# Create a fresh new container
docker run --name my-redis -p 6379:6379 -d redis:latest
```

**What these commands do:**

- `docker rm -f`: Forcefully removes the container (even if it's running)
- `docker run`: Creates a brand new container with the same settings

**Step 3:** Check if port is available
```bash
# See what's using port 6379
netstat -tulpn | grep :6379

# If something else is using it, either:
# 1. Stop that process, OR
# 2. Use a different port: docker run --name my-redis -p 6380:6379 -d redis:latest
```

!!! tip "Quick Debug Tips"
    - **Always check logs first** - they usually tell you exactly what's wrong
    - **Try restarting** - fixes 80% of issues
    - **Check ports** - many problems are caused by port conflicts
    - **For Docker**: When in doubt, remove and recreate the container

---

## What's Next?

Now that Redis is installed and running, let's learn how to use it!

In the next section, you'll:

- Take your first steps with Redis
- Learn basic server management
- Run your first Redis commands
- Understand how to start and stop Redis safely