# Redis Cheatsheet

A comprehensive guide to Redis - an in-memory data structure store used as a database, cache, and message broker.

## Table of Contents

- [Installation](#installation)
- [Basic Commands](#basic-commands)
- [Data Types](#data-types)
- [Persistence](#persistence)
- [Pub/Sub](#pubsub)
- [Transactions](#transactions)
- [Lua Scripting](#lua-scripting)
- [Cluster & Replication](#cluster--replication)
- [Performance](#performance)
- [Common Patterns](#common-patterns)

## Installation

### Install Redis

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install redis-server

# macOS
brew install redis

# Docker
docker run --name redis -p 6379:6379 -d redis

# Start Redis server
redis-server

# Start with config file
redis-server /path/to/redis.conf

# Start Redis CLI
redis-cli

# Connect to remote Redis
redis-cli -h hostname -p 6379 -a password

# Connect with authentication
redis-cli -a yourpassword

# Ping Redis
redis-cli ping
# PONG
```

### Configuration

```bash
# redis.conf important settings

# Bind address
bind 127.0.0.1

# Port
port 6379

# Password
requirepass yourpassword

# Max memory
maxmemory 256mb

# Eviction policy
maxmemory-policy allkeys-lru

# Persistence
save 900 1      # Save if 1 key changed in 900 seconds
save 300 10     # Save if 10 keys changed in 300 seconds
save 60 10000   # Save if 10000 keys changed in 60 seconds

# Append only file
appendonly yes
appendfilename "appendonly.aof"

# Log level
loglevel notice

# Log file
logfile "/var/log/redis/redis-server.log"
```

## Basic Commands

### Server Commands

```bash
# Server info
INFO
INFO server
INFO stats
INFO memory

# Select database (0-15)
SELECT 0

# Get all keys
KEYS *
KEYS user:*
KEYS *name*

# Flush database
FLUSHDB        # Current database
FLUSHALL       # All databases

# Save data
SAVE           # Blocking save
BGSAVE         # Background save

# Get last save time
LASTSAVE

# Client list
CLIENT LIST

# Monitor all commands
MONITOR

# Slow log
SLOWLOG GET 10

# Configuration
CONFIG GET maxmemory
CONFIG SET maxmemory 512mb

# Shutdown
SHUTDOWN
```

### Key Commands

```bash
# Set key
SET key value
SET name "John"
SET user:1 "John Doe"

# Set with expiration (seconds)
SET session:123 "data" EX 3600
SETEX session:123 3600 "data"

# Set if not exists
SET lock:resource "value" NX
SETNX lock:resource "value"

# Get key
GET key
GET name

# Check if key exists
EXISTS key
EXISTS name

# Delete key
DEL key
DEL name age email

# Delete asynchronously
UNLINK key

# Rename key
RENAME oldkey newkey

# Get key type
TYPE key

# Set expiration (seconds)
EXPIRE key 60

# Set expiration (milliseconds)
PEXPIRE key 60000

# Set expiration at timestamp
EXPIREAT key 1640000000

# Get TTL (time to live)
TTL key        # Returns seconds
PTTL key       # Returns milliseconds

# Remove expiration
PERSIST key

# Get all matching keys
SCAN 0 MATCH user:* COUNT 100

# Random key
RANDOMKEY

# Dump and restore
DUMP key
RESTORE key 0 <serialized-value>
```

## Data Types

### Strings

```bash
# Set string
SET name "John"

# Get string
GET name

# Multiple set
MSET key1 "value1" key2 "value2"

# Multiple get
MGET key1 key2 key3

# Append
APPEND key "value"
APPEND name " Doe"

# Get substring
GETRANGE key start end
GETRANGE name 0 4

# Set substring
SETRANGE key offset value

# String length
STRLEN key

# Increment
INCR counter
INCRBY counter 5
INCRBYFLOAT price 0.1

# Decrement
DECR counter
DECRBY counter 5

# Set and return old value
GETSET key newvalue
```

### Lists

```bash
# Push to list (left)
LPUSH mylist "value1"
LPUSH mylist "value2" "value3"

# Push to list (right)
RPUSH mylist "value4"

# Push if list exists
LPUSHX mylist "value"
RPUSHX mylist "value"

# Pop from list
LPOP mylist
RPOP mylist

# Blocking pop (wait for element)
BLPOP mylist 10    # Wait 10 seconds
BRPOP mylist 10

# Get range
LRANGE mylist 0 -1    # All elements
LRANGE mylist 0 9     # First 10 elements

# Get by index
LINDEX mylist 0

# Set by index
LSET mylist 0 "newvalue"

# Insert before/after
LINSERT mylist BEFORE "value2" "newvalue"
LINSERT mylist AFTER "value2" "newvalue"

# List length
LLEN mylist

# Remove elements
LREM mylist 2 "value"   # Remove 2 occurrences

# Trim list
LTRIM mylist 0 99       # Keep first 100 elements

# Move element between lists
RPOPLPUSH source dest
BRPOPLPUSH source dest 10
```

### Sets

```bash
# Add to set
SADD myset "member1"
SADD myset "member2" "member3"

# Get all members
SMEMBERS myset

# Check membership
SISMEMBER myset "member1"

# Remove from set
SREM myset "member1"

# Pop random member
SPOP myset
SPOP myset 3    # Pop 3 members

# Get random member (without removing)
SRANDMEMBER myset
SRANDMEMBER myset 3

# Set cardinality (size)
SCARD myset

# Move member between sets
SMOVE source dest "member"

# Set operations
SUNION set1 set2        # Union
SINTER set1 set2        # Intersection
SDIFF set1 set2         # Difference

# Store result of set operation
SUNIONSTORE dest set1 set2
SINTERSTORE dest set1 set2
SDIFFSTORE dest set1 set2

# Scan set
SSCAN myset 0 MATCH pattern* COUNT 100
```

### Sorted Sets (ZSets)

```bash
# Add to sorted set (score, member)
ZADD leaderboard 100 "player1"
ZADD leaderboard 200 "player2" 150 "player3"

# Get range by index
ZRANGE leaderboard 0 -1          # Ascending
ZRANGE leaderboard 0 -1 WITHSCORES
ZREVRANGE leaderboard 0 -1       # Descending

# Get range by score
ZRANGEBYSCORE leaderboard 100 200
ZRANGEBYSCORE leaderboard -inf +inf
ZRANGEBYSCORE leaderboard 100 200 WITHSCORES LIMIT 0 10

# Get member score
ZSCORE leaderboard "player1"

# Get member rank
ZRANK leaderboard "player1"      # Ascending
ZREVRANK leaderboard "player1"   # Descending

# Increment score
ZINCRBY leaderboard 10 "player1"

# Cardinality
ZCARD leaderboard

# Count in score range
ZCOUNT leaderboard 100 200

# Remove members
ZREM leaderboard "player1"

# Remove by rank
ZREMRANGEBYRANK leaderboard 0 2

# Remove by score
ZREMRANGEBYSCORE leaderboard 0 100

# Set operations
ZUNIONSTORE dest 2 set1 set2
ZINTERSTORE dest 2 set1 set2
```

### Hashes

```bash
# Set hash field
HSET user:1 name "John"
HSET user:1 age 30 email "john@example.com"

# Get hash field
HGET user:1 name

# Get all fields and values
HGETALL user:1

# Get multiple fields
HMGET user:1 name age

# Set multiple fields
HMSET user:1 name "John" age 30

# Set if field doesn't exist
HSETNX user:1 email "john@example.com"

# Check if field exists
HEXISTS user:1 name

# Delete fields
HDEL user:1 age

# Get all field names
HKEYS user:1

# Get all values
HVALS user:1

# Get number of fields
HLEN user:1

# Increment field
HINCRBY user:1 age 1
HINCRBYFLOAT user:1 balance 10.5

# Scan hash
HSCAN user:1 0 MATCH name* COUNT 100
```

### Bitmaps

```bash
# Set bit
SETBIT key offset value
SETBIT online_users 123 1

# Get bit
GETBIT key offset
GETBIT online_users 123

# Count set bits
BITCOUNT key
BITCOUNT key start end

# Bit operations
BITOP AND destkey key1 key2
BITOP OR destkey key1 key2
BITOP XOR destkey key1 key2
BITOP NOT destkey key

# Find first bit
BITPOS key 1        # First set bit
BITPOS key 0        # First unset bit
```

### HyperLogLog

```bash
# Add elements
PFADD unique_visitors user1 user2 user3

# Count unique elements
PFCOUNT unique_visitors

# Merge HyperLogLogs
PFMERGE dest source1 source2
```

### Streams

```bash
# Add to stream
XADD mystream * field1 value1 field2 value2
XADD mystream 1640000000000-0 name "John" age 30

# Read from stream
XREAD COUNT 10 STREAMS mystream 0
XREAD BLOCK 5000 STREAMS mystream $    # Block for new entries

# Get stream length
XLEN mystream

# Get range
XRANGE mystream - +
XRANGE mystream 1640000000000 1640000000001

# Consumer groups
XGROUP CREATE mystream mygroup $
XREADGROUP GROUP mygroup consumer1 COUNT 10 STREAMS mystream >

# Acknowledge message
XACK mystream mygroup message-id

# Get pending messages
XPENDING mystream mygroup
```

## Persistence

### RDB (Redis Database)

```bash
# Configuration
save 900 1
save 300 10
save 60 10000

# Manual save
SAVE        # Blocking
BGSAVE      # Background

# Disable RDB
save ""

# RDB file location
dir /var/lib/redis
dbfilename dump.rdb

# Compression
rdbcompression yes

# Checksum
rdbchecksum yes
```

### AOF (Append Only File)

```bash
# Enable AOF
appendonly yes
appendfilename "appendonly.aof"

# Fsync policy
appendfsync always      # Slow, safest
appendfsync everysec    # Good balance (default)
appendfsync no          # Fast, less safe

# Rewrite AOF
BGREWRITEAOF

# Auto rewrite
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# Load truncated AOF
aof-load-truncated yes
```

## Pub/Sub

### Publish/Subscribe

```bash
# Subscribe to channels
SUBSCRIBE channel1 channel2

# Subscribe to pattern
PSUBSCRIBE news:*

# Publish message
PUBLISH channel1 "message"

# Unsubscribe
UNSUBSCRIBE channel1
PUNSUBSCRIBE news:*

# List channels
PUBSUB CHANNELS
PUBSUB CHANNELS news:*

# Number of subscribers
PUBSUB NUMSUB channel1

# Number of pattern subscriptions
PUBSUB NUMPAT
```

### Example (Python)

```python
import redis

# Publisher
r = redis.Redis(host='localhost', port=6379)
r.publish('channel1', 'Hello, World!')

# Subscriber
r = redis.Redis(host='localhost', port=6379)
pubsub = r.pubsub()
pubsub.subscribe('channel1')

for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"Received: {message['data']}")
```

## Transactions

### MULTI/EXEC

```bash
# Start transaction
MULTI

# Queue commands
SET key1 "value1"
SET key2 "value2"
INCR counter

# Execute transaction
EXEC

# Discard transaction
DISCARD
```

### WATCH

```bash
# Watch keys for changes
WATCH key1 key2

# Start transaction
MULTI
SET key1 "newvalue"
EXEC

# If key1 or key2 changed, EXEC returns nil
```

### Example (Optimistic Locking)

```python
import redis

r = redis.Redis()

# Watch balance
r.watch('balance')

current_balance = int(r.get('balance'))

if current_balance >= 100:
    pipe = r.pipeline()
    pipe.multi()
    pipe.decrby('balance', 100)
    pipe.execute()
else:
    r.unwatch()
```

## Lua Scripting

### Basic Script

```bash
# Execute Lua script
EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 mykey "myvalue"

# Get and increment atomically
EVAL "local value = redis.call('GET', KEYS[1]); redis.call('INCR', KEYS[1]); return value" 1 counter

# Script with multiple commands
EVAL "
  local current = redis.call('GET', KEYS[1])
  if current == ARGV[1] then
    redis.call('SET', KEYS[1], ARGV[2])
    return 1
  else
    return 0
  end
" 1 mykey "oldvalue" "newvalue"

# Load script
SCRIPT LOAD "return redis.call('SET', KEYS[1], ARGV[1])"
# Returns SHA1 hash

# Execute loaded script
EVALSHA <sha1> 1 mykey "myvalue"

# Check if script exists
SCRIPT EXISTS <sha1>

# Flush scripts
SCRIPT FLUSH

# Kill running script
SCRIPT KILL
```

### Example: Rate Limiting

```lua
-- Rate limiter: max 10 requests per minute
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])

local current = redis.call('INCR', key)

if current == 1 then
    redis.call('EXPIRE', key, window)
end

if current > limit then
    return 0  -- Rate limit exceeded
else
    return 1  -- Allow request
end
```

```bash
# Use rate limiter
EVAL "..." 1 "ratelimit:user:123" 10 60
```

## Cluster & Replication

### Replication

```bash
# On slave/replica
REPLICAOF hostname 6379
REPLICAOF 127.0.0.1 6379

# Stop replication
REPLICAOF NO ONE

# Read-only replica
replica-read-only yes

# Replication info
INFO replication

# Configuration
# Master
bind 0.0.0.0
requirepass masterpassword

# Replica
replicaof masterhost 6379
masterauth masterpassword
```

### Sentinel (High Availability)

```bash
# sentinel.conf
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel auth-pass mymaster password
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 10000

# Start Sentinel
redis-sentinel /path/to/sentinel.conf

# Sentinel commands
SENTINEL masters
SENTINEL master mymaster
SENTINEL slaves mymaster
SENTINEL sentinels mymaster
SENTINEL get-master-addr-by-name mymaster
SENTINEL failover mymaster
```

### Cluster

```bash
# Create cluster (Redis 5.0+)
redis-cli --cluster create \
  127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1

# Check cluster
redis-cli --cluster check 127.0.0.1:7000

# Cluster info
CLUSTER INFO
CLUSTER NODES

# Add node
redis-cli --cluster add-node new-node-ip:port existing-node-ip:port

# Remove node
redis-cli --cluster del-node node-ip:port node-id

# Reshard
redis-cli --cluster reshard node-ip:port

# Configuration
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
```

## Performance

### Benchmarking

```bash
# Basic benchmark
redis-benchmark

# Specific test
redis-benchmark -t set,get -n 100000 -q

# With pipeline
redis-benchmark -t set,get -n 100000 -P 16 -q

# Specific key size
redis-benchmark -t set -d 100 -n 100000

# Custom test
redis-benchmark -t get -n 100000 -q
```

### Memory Optimization

```bash
# Memory usage
MEMORY USAGE key

# Memory stats
MEMORY STATS

# Memory doctor
MEMORY DOCTOR

# Eviction policies
maxmemory-policy noeviction       # Return errors
maxmemory-policy allkeys-lru      # Remove least recently used
maxmemory-policy volatile-lru     # Remove LRU with TTL
maxmemory-policy allkeys-random   # Remove random keys
maxmemory-policy volatile-random  # Remove random keys with TTL
maxmemory-policy volatile-ttl     # Remove keys with shortest TTL
maxmemory-policy allkeys-lfu      # Least frequently used
maxmemory-policy volatile-lfu     # LFU with TTL

# Optimize memory
# Use hashes for small objects
# Use integers for keys when possible
# Compress large values
# Use appropriate data structures
```

### Pipeline

```python
# Python example
import redis

r = redis.Redis()

# Without pipeline (slow)
for i in range(10000):
    r.set(f'key{i}', f'value{i}')

# With pipeline (fast)
pipe = r.pipeline()
for i in range(10000):
    pipe.set(f'key{i}', f'value{i}')
pipe.execute()
```

## Common Patterns

### Caching

```python
import redis
import json

r = redis.Redis()

def get_user(user_id):
    # Try cache first
    cache_key = f'user:{user_id}'
    cached = r.get(cache_key)

    if cached:
        return json.loads(cached)

    # Cache miss, get from database
    user = fetch_user_from_db(user_id)

    # Store in cache (expires in 1 hour)
    r.setex(cache_key, 3600, json.dumps(user))

    return user
```

### Session Storage

```python
import redis
import uuid

r = redis.Redis()

def create_session(user_id):
    session_id = str(uuid.uuid4())
    session_key = f'session:{session_id}'

    session_data = {
        'user_id': user_id,
        'created_at': time.time()
    }

    # Session expires in 24 hours
    r.setex(session_key, 86400, json.dumps(session_data))

    return session_id

def get_session(session_id):
    session_key = f'session:{session_id}'
    data = r.get(session_key)

    if data:
        # Refresh expiration
        r.expire(session_key, 86400)
        return json.loads(data)

    return None
```

### Distributed Lock

```python
import redis
import time
import uuid

r = redis.Redis()

def acquire_lock(lock_name, timeout=10):
    identifier = str(uuid.uuid4())
    lock_key = f'lock:{lock_name}'

    # Try to acquire lock
    if r.set(lock_key, identifier, ex=timeout, nx=True):
        return identifier

    return None

def release_lock(lock_name, identifier):
    lock_key = f'lock:{lock_name}'

    # Lua script for atomic check and delete
    script = """
    if redis.call('get', KEYS[1]) == ARGV[1] then
        return redis.call('del', KEYS[1])
    else
        return 0
    end
    """

    return r.eval(script, 1, lock_key, identifier)

# Usage
lock_id = acquire_lock('resource')
if lock_id:
    try:
        # Do work
        pass
    finally:
        release_lock('resource', lock_id)
```

### Leaderboard

```python
import redis

r = redis.Redis()

def add_score(player_id, score):
    r.zadd('leaderboard', {player_id: score})

def get_rank(player_id):
    # 0-indexed, so add 1
    rank = r.zrevrank('leaderboard', player_id)
    return rank + 1 if rank is not None else None

def get_top(n=10):
    return r.zrevrange('leaderboard', 0, n-1, withscores=True)

def get_score(player_id):
    return r.zscore('leaderboard', player_id)
```

### Rate Limiting

```python
import redis
import time

r = redis.Redis()

def is_allowed(user_id, max_requests=10, window=60):
    key = f'ratelimit:{user_id}'

    # Using Lua script
    script = """
    local current = redis.call('INCR', KEYS[1])
    if current == 1 then
        redis.call('EXPIRE', KEYS[1], ARGV[1])
    end
    return current
    """

    current = r.eval(script, 1, key, window)

    return current <= max_requests
```

## Additional Resources

- [Redis Documentation](https://redis.io/documentation)
- [Redis Commands Reference](https://redis.io/commands)
- [Redis Best Practices](https://redis.io/topics/best-practices)
- [Redis University](https://university.redis.com/)

---

*Last updated: 2025-11-16*
