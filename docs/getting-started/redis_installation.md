# Installing Redis

Ready to get Redis running on your system? This guide covers installation for **Windows**, **macOS**, and **Linux**, plus a beginner-friendly **Docker** option.

!!! tip "Recommended for Beginners"
    If you're completely new to Redis, I recommend starting with **Docker** - it's the most consistent across all systems and easiest to manage.

## Option 1: Docker Installation (Recommended)

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

---

## Option 2: Windows Installation

### Method A: Using Windows Subsystem for Linux (WSL) - Recommended

1. **Install WSL2** (if not already installed):
   ```powershell
   wsl --install
   ```

2. **Open WSL terminal** and follow the Linux Ubuntu instructions [below](#option-4-linux-installation).

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

---

## Option 3: macOS Installation

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

# Start Redis manually (stops when terminal closes)
redis-server
```

---

## Option 4: Linux Installation

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
**Expected output**: `PONG`

### 2. Basic Functionality Test

```redis
# Connect to Redis
redis-cli

# Set a value
127.0.0.1:6379> SET test "Hello Redis"
OK

# Get the value
127.0.0.1:6379> GET test
"Hello Redis"

# Check Redis info
127.0.0.1:6379> INFO server
```

### 3. Performance Test

```redis
# Run a simple benchmark
127.0.0.1:6379> DEBUG SEGFAULT
# Just kidding! Don't run that 😅

# Instead, try this:
127.0.0.1:6379> SET counter 0
127.0.0.1:6379> INCR counter
(integer) 1
127.0.0.1:6379> INCR counter
(integer) 2
```

!!! success "Installation Complete!"
    If all the above commands work, congratulations! Redis is successfully installed and running.

---

## Basic Configuration

### Default Settings
- **Port**: 6379
- **Host**: 127.0.0.1 (localhost)
- **Configuration file**: 
  - Linux: `/etc/redis/redis.conf`
  - macOS: `/usr/local/etc/redis.conf`
  - Docker: Inside the container

### Important First Steps

1. **Set a password** (recommended for security):
   ```redis
   CONFIG SET requirepass your-password-here
   AUTH your-password-here
   ```

2. **Check memory usage**:
   ```redis
   INFO memory
   ```

3. **See all configuration**:
   ```redis
   CONFIG GET "*"
   ```

---

## Troubleshooting Common Issues

### Redis Won't Start

**Problem**: `redis-server` command not found  
**Solution**: Check if Redis is in your PATH or use the full path

**Problem**: Port 6379 already in use  
**Solution**: 
```bash
# Find what's using the port
sudo lsof -i :6379

# Kill the process or use a different port
redis-server --port 6380
```

### Connection Issues

**Problem**: Can't connect with `redis-cli`  
**Solution**:
```bash
# Check if Redis is running
ps aux | grep redis

# Check if the service is active (Linux)
sudo systemctl status redis
```

### Permission Issues (Linux)

**Problem**: Permission denied errors  
**Solution**:
```bash
# Fix Redis directory permissions
sudo chown redis:redis /var/lib/redis
sudo chmod 755 /var/lib/redis
```

### Docker Issues

**Problem**: Docker container won't start  
**Solution**:
```bash
# Check Docker logs
docker logs my-redis

# Remove and recreate container
docker rm -f my-redis
docker run --name my-redis -p 6379:6379 -d redis:latest
```

---

## Installation Comparison

| Method | Difficulty | Best For | Pros | Cons |
|--------|------------|----------|------|------|
| **Docker** | Easy | Beginners, Development | Consistent, Easy cleanup | Requires Docker |
| **Package Manager** | Medium | Production, Long-term use | Native performance, Auto-updates | OS-specific |
| **Manual Compile** | Hard | Custom setups | Latest features, Full control | Complex setup |

## What's Next?

Now that Redis is installed and running, let's learn how to use it!

In the next section, you'll:
- Take your first steps with Redis
- Learn basic server management
- Run your first Redis commands
- Understand how to start and stop Redis safely