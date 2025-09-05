# Redis Glossary

Essential Redis terms and concepts explained in simple language.

## A

**ACL (Access Control List)**
Security feature in Redis 6+ that allows creating users with specific permissions for commands and keys.

**AOF (Append Only File)**
A persistence method that logs every write operation to a file, allowing complete data reconstruction.

**Atomic Operation**
An operation that completes entirely or not at all - no partial execution possible.

## B

**Background Save**
A non-blocking save operation (BGSAVE) that creates a snapshot while Redis continues serving requests.

**Binary Safe**
Ability to store any kind of data (including binary data like images) as Redis treats all data as byte sequences.

**Blocking Operation**
A command that waits for a condition to be met before returning (like BLPOP waiting for list items).

## C

**Cache**
Temporary storage for frequently accessed data to improve application performance.

**Cluster**
Multiple Redis instances working together to provide horizontal scaling and high availability.

**Command**
An instruction sent to Redis to perform an operation (like SET, GET, LPUSH).

**Connection Pool**
A set of reusable database connections shared among multiple application requests.

## D

**Database**
Redis supports 16 numbered databases (0-15) that can hold different data sets.

**Data Structure**
The way data is organized in Redis (strings, lists, sets, hashes, sorted sets).

**DEL**
Command to delete one or more keys from Redis.

## E

**Eviction Policy**
Rules that determine which keys to remove when Redis reaches memory limits.

**EVAL**
Command to execute Lua scripts on the Redis server.

**Expiration**
Automatic removal of keys after a specified time period.

**EXISTS**
Command to check if a key exists in Redis.

## F

**Failover**
Automatic switching to a backup server when the primary server fails.

**FLUSHALL**
Dangerous command that removes all data from all databases.

**FLUSHDB**
Command that removes all data from the current database.

## G

**GET**
Command to retrieve the value of a string key.

**Geospatial**
Redis data type for storing and querying geographical coordinates.

## H

**Hash**
Redis data structure similar to a dictionary or object, storing field-value pairs.

**High Availability**
System design that ensures continuous operation even when components fail.

**HGET**
Command to get a specific field value from a hash.

**HSET**
Command to set a field value in a hash.

## I

**INCR**
Command to increment a numeric string value by 1.

**Index**
In Redis context, usually refers to the position of an item in a list or sorted set.

**In-Memory Database**
Database that stores data in RAM for ultra-fast access.

## J

**JSON**
Data format; RedisJSON module adds native JSON support to Redis.

## K

**Key**
Unique identifier for data stored in Redis (like "user:123" or "session:abc").

**Key Space**
The collection of all keys in a Redis database.

**KEYS**
Command to find keys matching a pattern (avoid in production).

## L

**List**
Redis data structure that stores ordered sequences of strings.

**LPUSH**
Command to add elements to the left (beginning) of a list.

**LPOP**
Command to remove and return an element from the left of a list.

**Lua**
Scripting language supported by Redis for server-side operations.

## M

**Master**
Primary Redis server in a replication setup that handles writes.

**Memory Policy**
Configuration that determines how Redis manages memory usage.

**MONITOR**
Command to see all commands processed by Redis in real-time.

**MULTI**
Command to start a Redis transaction.

## N

**Namespace**
Using prefixes in key names to organize data (like "user:" or "cache:").

**Node**
A single Redis server instance, especially in cluster context.

## O

**Operation**
Any action performed on Redis data (read, write, delete, etc.).

**Out of Memory (OOM)**
Error when Redis reaches its memory limit and cannot store more data.

## P

**Persistence**
Saving Redis data to disk so it survives server restarts.

**Pipeline**
Technique to send multiple commands to Redis at once for better performance.

**Pub/Sub**
Publish/Subscribe messaging pattern for real-time communication.

**PUBLISH**
Command to send a message to a channel.

## Q

**Query**
A request for data from Redis (though Redis uses commands rather than SQL).

**Queue**
Using Redis lists to implement message queues for task processing.

## R

**RDB (Redis Database)**
Binary file format for Redis data snapshots.

**Replica**
Secondary Redis server that maintains a copy of master's data.

**Replication**
Process of copying data from master to replica servers.

**RPUSH**
Command to add elements to the right (end) of a list.

## S

**SADD**
Command to add members to a set.

**SCAN**
Command to iterate through keys safely (better than KEYS).

**Sentinel**
Redis component that provides high availability and monitoring.

**SET**
Command to store a string value with a key.

**Set**
Redis data structure that stores unique, unordered strings.

**Shard**
Portion of data in a distributed Redis cluster.

**Slave**
Old term for replica server (now called replica).

**Sorted Set**
Redis data structure that stores unique strings with associated scores.

## T

**TTL (Time To Live)**
Remaining time before a key expires and is automatically deleted.

**Transaction**
Group of commands executed atomically using MULTI/EXEC.

**TYPE**
Command to determine what data structure a key contains.

## U

**Union**
Set operation that combines elements from multiple sets.

**UNLINK**
Non-blocking version of DEL command for large keys.

## V

**Value**
The data stored with a key in Redis.

**Volatile**
Keys with expiration times (as opposed to persistent keys).

## W

**WATCH**
Command to monitor keys for changes during transactions.

**Write**
Operation that modifies data in Redis (SET, LPUSH, SADD, etc.).

## X

**XADD**
Command to add entries to a Redis stream.

## Z

**ZADD**
Command to add members with scores to a sorted set.

**ZRANGE**
Command to get members from a sorted set by rank.

**ZSCORE**
Command to get the score of a member in a sorted set.

## Common Acronyms  
**API** - Application Programming Interface  
**CLI** - Command Line Interface  
**CPU** - Central Processing Unit  
**DNS** - Domain Name System  
**HTTP** - Hypertext Transfer Protocol  
**IP** - Internet Protocol  
**JSON** - JavaScript Object Notation  
**RAM** - Random Access Memory  
**SQL** - Structured Query Language  
**TCP** - Transmission Control Protocol  
**TLS** - Transport Layer Security  
**URL** - Uniform Resource Locator  
**UTF** - Unicode Transformation Format  

## Redis-Specific Terms

**Redis**
REmote DIctionary Server - the original name explaining Redis as a networked data structure server.

**redis-cli**
The command-line interface tool for interacting with Redis.

**redis-server**
The main Redis server program.

**redis.conf**
The configuration file for Redis server.

**Keyspace**
The space containing all keys in a Redis database.

**Keyspace Events**
Notifications about changes to Redis data.

**Memory Fragmentation**
Inefficient memory usage due to how Redis allocates and frees memory.

**Hot Key**
A key that receives significantly more requests than others.

**Cold Key**
A key that is rarely accessed.

## Data Type Specific Terms

**String Operations**

- APPEND, INCR, DECR, STRLEN

**List Operations**

- LPUSH, RPUSH, LPOP, RPOP, LRANGE

**Set Operations**

- SADD, SREM, SINTER, SUNION, SDIFF

**Hash Operations**

- HSET, HGET, HGETALL, HDEL

**Sorted Set Operations**

- ZADD, ZRANGE, ZRANK, ZSCORE

## Performance Terms

**Latency**
Time delay between sending a command and receiving a response.

**Throughput**
Number of operations Redis can handle per second.

**Benchmark**
Performance test to measure Redis capabilities.

**Bottleneck**
The component that limits overall system performance.

## Operational Terms

**Backup**
Copy of Redis data for disaster recovery.

**Migration**
Moving data from one Redis instance to another.

**Scaling**
Increasing Redis capacity (vertical = more resources, horizontal = more servers).

**Load Balancing**
Distributing requests across multiple Redis instances.

**Monitoring**
Tracking Redis performance and health metrics.

**Alerting**
Automatic notifications when Redis issues occur.

This glossary covers the essential Redis terminology you'll encounter while learning and using Redis. Refer back to it whenever you come across unfamiliar terms!
