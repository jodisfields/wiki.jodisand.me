# System Design Patterns & Principles

A comprehensive guide to system design patterns, principles, and best practices for building scalable, reliable, and maintainable systems.

## Table of Contents

- [Core Concepts](#core-concepts)
- [Scalability](#scalability)
- [Database Design](#database-design)
- [Caching Strategies](#caching-strategies)
- [Load Balancing](#load-balancing)
- [API Design](#api-design)
- [Microservices](#microservices)
- [Message Queues](#message-queues)
- [Consistency & CAP Theorem](#consistency--cap-theorem)
- [Design Patterns](#design-patterns)
- [Common System Designs](#common-system-designs)

## Core Concepts

### Performance Metrics

```
Latency: Time to complete a single operation
Throughput: Number of operations per unit time
Bandwidth: Maximum data transfer rate
Availability: Percentage of uptime (99.9% = 43.8 minutes downtime/month)

Availability Tiers:
- 99%     = 7.2 hours downtime/month
- 99.9%   = 43.8 minutes downtime/month
- 99.99%  = 4.38 minutes downtime/month
- 99.999% = 26.3 seconds downtime/month
```

### Latency Numbers

```
L1 cache reference:              0.5 ns
Branch mispredict:               5 ns
L2 cache reference:              7 ns
Mutex lock/unlock:              25 ns
Main memory reference:         100 ns
Compress 1KB with Snappy:    3,000 ns
Send 1KB over 1 Gbps:       10,000 ns
Read 4KB from SSD:         150,000 ns
Read 1MB sequentially:     250,000 ns
Round trip in datacenter:  500,000 ns
Read 1MB from SSD:       1,000,000 ns
Disk seek:              10,000,000 ns
Read 1MB from disk:     20,000,000 ns
Send packet US->EU:    150,000,000 ns
```

### System Design Principles

1. **KISS** - Keep It Simple, Stupid
2. **DRY** - Don't Repeat Yourself
3. **YAGNI** - You Aren't Gonna Need It
4. **Separation of Concerns**
5. **Single Responsibility Principle**
6. **Fail Fast**
7. **Idempotency**
8. **Immutability**
9. **Eventual Consistency**
10. **Defense in Depth**

## Scalability

### Vertical Scaling (Scale Up)

```
Pros:
- Simple to implement
- No application changes needed
- Maintains data consistency

Cons:
- Hardware limits
- Single point of failure
- Expensive
- Downtime during upgrade

Use Cases:
- Legacy applications
- Databases (up to a point)
- Applications with tight coupling
```

### Horizontal Scaling (Scale Out)

```
Pros:
- Nearly unlimited scaling
- Better fault tolerance
- Cost-effective (commodity hardware)
- No downtime

Cons:
- Complex architecture
- Data consistency challenges
- Network overhead
- State management

Use Cases:
- Web applications
- Stateless services
- Distributed systems
```

### Load Distribution Patterns

```yaml
Round Robin:
  - Simple rotation through servers
  - Equal distribution
  - No server state awareness

Least Connections:
  - Route to server with fewest active connections
  - Better for long-lived connections
  - Requires connection tracking

IP Hash:
  - Route based on client IP hash
  - Session persistence
  - Uneven distribution possible

Weighted:
  - Distribute based on server capacity
  - Handles heterogeneous servers
  - Dynamic weight adjustment

Least Response Time:
  - Route to fastest responding server
  - Optimal performance
  - Requires health checking
```

## Database Design

### SQL vs NoSQL

```yaml
SQL (Relational):
  Best For:
    - Complex queries
    - ACID transactions
    - Structured data
    - Relationships
  Examples:
    - PostgreSQL
    - MySQL
    - Oracle

NoSQL:
  Document Store (MongoDB, CouchDB):
    - Flexible schema
    - JSON-like documents
    - Good for hierarchical data

  Key-Value (Redis, DynamoDB):
    - Simple lookups
    - High performance
    - Caching

  Column-Family (Cassandra, HBase):
    - Write-heavy workloads
    - Time-series data
    - Wide rows

  Graph (Neo4j, Amazon Neptune):
    - Relationship-heavy data
    - Social networks
    - Recommendation engines
```

### Database Sharding

```yaml
Vertical Sharding (by feature):
  Users DB:
    - users table
    - profiles table
    - authentication table

  Products DB:
    - products table
    - inventory table
    - categories table

Horizontal Sharding (by data):
  User Shard 1:
    - user_id 1-1000000

  User Shard 2:
    - user_id 1000001-2000000

  User Shard 3:
    - user_id 2000001-3000000

Sharding Strategies:
  Hash-based:
    - shard = hash(user_id) % num_shards
    - Even distribution
    - Hard to add shards

  Range-based:
    - shard_1: 0-999999
    - shard_2: 1000000-1999999
    - Easy to add shards
    - Potential hotspots

  Directory-based:
    - Lookup table maps keys to shards
    - Flexible
    - Additional lookup overhead
```

### Replication Patterns

```yaml
Master-Slave (Primary-Replica):
  Setup:
    - One master handles writes
    - Multiple slaves handle reads
    - Async replication

  Pros:
    - Read scalability
    - Simple setup

  Cons:
    - Write bottleneck
    - Replication lag
    - Single point of failure

Master-Master (Multi-Master):
  Setup:
    - Multiple masters accept writes
    - Bidirectional replication

  Pros:
    - Write scalability
    - High availability

  Cons:
    - Conflict resolution
    - Complex setup
    - Eventual consistency

Peer-to-Peer:
  Setup:
    - All nodes are equal
    - Distributed consensus

  Pros:
    - No single point of failure
    - High availability

  Cons:
    - Complex
    - Network overhead
```

## Caching Strategies

### Cache Patterns

```yaml
Cache-Aside (Lazy Loading):
  Flow:
    1. Check cache
    2. If miss, read from DB
    3. Write to cache
    4. Return data

  Pros:
    - Only cache what's needed
    - Cache failures don't crash system

  Cons:
    - Initial request slow
    - Potential cache stampede

Write-Through:
  Flow:
    1. Write to cache
    2. Write to database
    3. Return success

  Pros:
    - Cache always consistent
    - Read performance

  Cons:
    - Write latency
    - Unused data cached

Write-Behind (Write-Back):
  Flow:
    1. Write to cache
    2. Return success
    3. Async write to DB

  Pros:
    - Fast writes
    - Batch DB writes

  Cons:
    - Data loss risk
    - Complex implementation

Refresh-Ahead:
  Flow:
    1. Predict needed data
    2. Pre-load into cache
    3. Serve from cache

  Pros:
    - Always fast reads
    - Reduced latency

  Cons:
    - Prediction complexity
    - Wasted cache on wrong predictions
```

### Cache Eviction Policies

```yaml
LRU (Least Recently Used):
  - Remove least recently accessed
  - Good for general use
  - O(1) with HashMap + LinkedList

LFU (Least Frequently Used):
  - Remove least frequently accessed
  - Good for varied access patterns
  - More complex tracking

FIFO (First In First Out):
  - Remove oldest entry
  - Simple implementation
  - May remove hot data

TTL (Time To Live):
  - Remove after expiration
  - Good for time-sensitive data
  - Prevents stale data
```

### Caching Levels

```yaml
Client-Side:
  - Browser cache
  - Mobile app cache
  - Fast access
  - Limited control

CDN (Content Delivery Network):
  - Geographic distribution
  - Static content
  - Reduced latency

Application Cache:
  - Redis, Memcached
  - Shared across instances
  - Flexible data structures

Database Cache:
  - Query cache
  - Buffer pool
  - Automatic management
```

## Load Balancing

### Layer 4 vs Layer 7

```yaml
Layer 4 (Transport):
  - TCP/UDP level
  - IP + Port routing
  - Fast performance
  - Simple routing
  - No content awareness

  Example: HAProxy, AWS NLB

Layer 7 (Application):
  - HTTP/HTTPS level
  - Content-based routing
  - URL path routing
  - Header inspection
  - SSL termination

  Example: Nginx, AWS ALB
```

### Load Balancer Algorithms

```yaml
Round Robin:
  def get_server(servers, counter):
    return servers[counter % len(servers)]

Weighted Round Robin:
  def get_server(servers_with_weights):
    total = sum(weight for _, weight in servers_with_weights)
    rand = random(0, total)
    for server, weight in servers_with_weights:
      if rand < weight:
        return server
      rand -= weight

Least Connections:
  def get_server(servers):
    return min(servers, key=lambda s: s.active_connections)

IP Hash:
  def get_server(servers, client_ip):
    hash_value = hash(client_ip)
    return servers[hash_value % len(servers)]
```

### Health Checks

```yaml
Active Health Checks:
  HTTP:
    - GET /health
    - Expect 200 OK
    - Interval: 10s
    - Timeout: 5s
    - Unhealthy threshold: 3
    - Healthy threshold: 2

  TCP:
    - Establish connection
    - Check port open
    - Fast but limited

Passive Health Checks:
  - Monitor actual traffic
  - Error rate threshold
  - Response time threshold
  - Circuit breaker pattern
```

## API Design

### REST API Best Practices

```yaml
HTTP Methods:
  GET:    Retrieve resource(s)
  POST:   Create resource
  PUT:    Update/Replace resource
  PATCH:  Partial update
  DELETE: Delete resource

Status Codes:
  200: OK
  201: Created
  204: No Content
  400: Bad Request
  401: Unauthorized
  403: Forbidden
  404: Not Found
  500: Internal Server Error
  503: Service Unavailable

URL Design:
  Good:
    GET    /users
    GET    /users/123
    POST   /users
    PUT    /users/123
    DELETE /users/123
    GET    /users/123/orders

  Bad:
    GET /getUsers
    GET /user/123/delete
    POST /createUser
```

### API Versioning

```yaml
URI Versioning:
  - /api/v1/users
  - /api/v2/users
  - Simple and clear
  - URL pollution

Header Versioning:
  - GET /api/users
  - Accept: application/vnd.company.v1+json
  - Clean URLs
  - Less visible

Query Parameter:
  - /api/users?version=1
  - Backward compatible
  - Easy to default
```

### Rate Limiting

```yaml
Algorithms:

Token Bucket:
  class TokenBucket:
    def __init__(self, capacity, refill_rate):
      self.capacity = capacity
      self.tokens = capacity
      self.refill_rate = refill_rate

    def allow_request(self):
      self.refill()
      if self.tokens >= 1:
        self.tokens -= 1
        return True
      return False

Leaky Bucket:
  - Fixed output rate
  - Queue incoming requests
  - Drop on overflow

Fixed Window:
  - Count requests per time window
  - Simple but allows bursts
  - Reset at window boundary

Sliding Window:
  - Weighted count across windows
  - Smooth rate limiting
  - More complex

Headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 87
  X-RateLimit-Reset: 1640000000
```

## Microservices

### Microservices Patterns

```yaml
Service Decomposition:
  By Business Capability:
    - Order Service
    - Payment Service
    - Inventory Service
    - User Service

  By Subdomain (DDD):
    - Bounded contexts
    - Domain models
    - Ubiquitous language

Communication:
  Synchronous:
    - REST API
    - gRPC
    - Direct calls
    - Tight coupling

  Asynchronous:
    - Message Queue
    - Event Bus
    - Pub/Sub
    - Loose coupling

Service Discovery:
  Client-Side:
    - Eureka, Consul
    - Client queries registry
    - Client-side load balancing

  Server-Side:
    - AWS ELB, Kubernetes Service
    - Load balancer queries registry
    - Transparent to client
```

### Saga Pattern (Distributed Transactions)

```yaml
Choreography:
  Order Service:
    → Creates order
    → Publishes OrderCreated event

  Payment Service:
    → Listens OrderCreated
    → Processes payment
    → Publishes PaymentCompleted

  Inventory Service:
    → Listens PaymentCompleted
    → Reserves items
    → Publishes ItemsReserved

Orchestration:
  Order Orchestrator:
    1. Call Order Service → Create order
    2. Call Payment Service → Process payment
    3. Call Inventory Service → Reserve items
    4. If any fails → Rollback all

  Compensation:
    - RefundPayment
    - ReleaseInventory
    - CancelOrder
```

### Circuit Breaker Pattern

```python
class CircuitBreaker:
    CLOSED = 'closed'      # Normal operation
    OPEN = 'open'          # Failing, reject requests
    HALF_OPEN = 'half_open'  # Testing recovery

    def __init__(self, failure_threshold=5, timeout=60):
        self.state = self.CLOSED
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.last_failure_time = None

    def call(self, func):
        if self.state == self.OPEN:
            if time.time() - self.last_failure_time >= self.timeout:
                self.state = self.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func()
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise e

    def on_success(self):
        self.failure_count = 0
        self.state = self.CLOSED

    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = self.OPEN
```

## Message Queues

### Messaging Patterns

```yaml
Point-to-Point (Queue):
  - Single consumer per message
  - Message deleted after consumption
  - Load balancing across consumers
  - Use: Task distribution

Publish-Subscribe (Topic):
  - Multiple consumers per message
  - Message retained
  - Broadcast to all subscribers
  - Use: Event notification

Message Ordering:
  FIFO:
    - First in, first out
    - Ordered processing
    - Single consumer

  Priority:
    - High priority first
    - SLA-based processing
    - Multiple queues

Dead Letter Queue:
  - Failed messages
  - Poison messages
  - Manual intervention
  - Debug & retry
```

### Popular Message Queues

```yaml
RabbitMQ:
  - AMQP protocol
  - Flexible routing
  - Complex topologies
  - Good for enterprise

Apache Kafka:
  - High throughput
  - Event streaming
  - Log aggregation
  - Good for big data

AWS SQS:
  - Fully managed
  - Highly scalable
  - Simple setup
  - Good for AWS ecosystem

Redis Pub/Sub:
  - In-memory
  - Very fast
  - Simple pub/sub
  - No persistence guarantee
```

## Consistency & CAP Theorem

### CAP Theorem

```yaml
Pick 2 of 3:

Consistency (C):
  - All nodes see same data
  - Reads return latest write
  - Strong consistency

Availability (A):
  - Every request gets response
  - No guarantee it's latest
  - System always operational

Partition Tolerance (P):
  - System works despite network splits
  - Required in distributed systems
  - Must choose between C and A

Examples:
  CP Systems:
    - MongoDB (with default settings)
    - HBase
    - Redis (in cluster mode)
    - Prioritize consistency

  AP Systems:
    - Cassandra
    - DynamoDB
    - CouchDB
    - Prioritize availability

  CA Systems:
    - Traditional RDBMS
    - Only in single node
    - Not partition tolerant
```

### Consistency Models

```yaml
Strong Consistency:
  - Immediate consistency
  - Linearizability
  - Slow but accurate
  - Use: Banking, inventory

Eventual Consistency:
  - Eventually consistent
  - Fast but stale reads possible
  - Use: Social media feeds, analytics

Causal Consistency:
  - Causally related operations ordered
  - Others may be out of order
  - Middle ground

Read-Your-Writes:
  - User sees own writes
  - Others may not
  - Good for user experience
```

## Design Patterns

### Common Architectural Patterns

```yaml
Layered Architecture:
  Presentation Layer (UI)
  ↓
  Business Logic Layer
  ↓
  Data Access Layer
  ↓
  Database

  Pros: Simple, testable
  Cons: Monolithic, scaling issues

Event-Driven Architecture:
  Producer → Event Bus → Consumer(s)

  Pros: Loose coupling, scalable
  Cons: Complex, debugging hard

CQRS (Command Query Responsibility Segregation):
  Commands (Write):
    → Write Model
    → Write Database

  Queries (Read):
    → Read Model
    → Read Database (optimized)

  Pros: Optimized reads/writes
  Cons: Complexity, eventual consistency

Serverless:
  API Gateway → Lambda Functions → Services

  Pros: Auto-scaling, pay-per-use
  Cons: Cold starts, vendor lock-in
```

### Database Patterns

```yaml
Database per Service:
  - Each microservice owns database
  - Data isolation
  - Independent scaling
  - Distributed transactions complex

Shared Database:
  - Multiple services share DB
  - Simple transactions
  - Tight coupling
  - Schema change coordination

Event Sourcing:
  - Store events, not state
  - Full audit trail
  - Replay events
  - Complex queries

Materialized View:
  - Pre-computed results
  - Fast queries
  - Stale data
  - Update overhead
```

## Common System Designs

### URL Shortener

```yaml
Requirements:
  - Shorten URLs
  - Redirect to original URL
  - Analytics (optional)
  - Custom aliases
  - Expiration

Components:
  API Server:
    - POST /shorten
    - GET /{short_code}

  Database:
    - url_mappings table
      - id: BIGINT
      - short_code: VARCHAR(7)
      - original_url: TEXT
      - created_at: TIMESTAMP
      - expires_at: TIMESTAMP

  Cache (Redis):
    - Key: short_code
    - Value: original_url
    - TTL: 24 hours

Algorithm:
  Base62 Encoding:
    charset = "0-9a-zA-Z"
    id = auto_increment
    short_code = base62_encode(id)

  MD5 Hash:
    hash = md5(url + timestamp)
    short_code = hash[:7]
    handle collisions

Scaling:
  - Database sharding by short_code
  - Cache layer (Redis)
  - CDN for redirects
  - Rate limiting
```

### Design Twitter

```yaml
Requirements:
  - Post tweets (280 chars)
  - Follow users
  - Timeline (home & user)
  - Like, retweet
  - Search

Components:
  Services:
    - User Service
    - Tweet Service
    - Timeline Service
    - Search Service
    - Notification Service

  Databases:
    Users:
      - user_id, username, email, etc.

    Tweets:
      - tweet_id, user_id, content, created_at
      - Sharded by tweet_id

    Followers:
      - follower_id, followee_id
      - Sharded by follower_id

    Timelines (Redis):
      - user:{id}:timeline → [tweet_ids]
      - Cached, pre-computed

  Feed Generation:
    Fanout on Write (Push):
      - Write to all followers' timelines
      - Fast reads
      - Slow writes for popular users

    Fanout on Read (Pull):
      - Read from followed users
      - Fast writes
      - Slow reads

    Hybrid:
      - Push for normal users
      - Pull for celebrities

Scaling:
  - Read replicas for user data
  - Sharded tweet storage
  - Redis for timelines
  - Elasticsearch for search
  - Message queue for notifications
```

### Design Chat System

```yaml
Requirements:
  - 1-on-1 chat
  - Group chat
  - Online status
  - Message history
  - Push notifications

Components:
  WebSocket Server:
    - Persistent connections
    - Real-time messaging
    - Presence detection

  Message Service:
    - Store messages
    - Delivery status
    - Read receipts

  Presence Service:
    - Online/offline status
    - Last seen
    - Typing indicators

Databases:
  Messages:
    - message_id
    - conversation_id
    - sender_id
    - content
    - timestamp
    - Cassandra (time-series)

  Conversations:
    - conversation_id
    - participants
    - type (1-on-1, group)
    - PostgreSQL

  Connection Store:
    - user_id → server_id mapping
    - Redis for fast lookup

Flow:
  Send Message:
    1. Client → WebSocket Server
    2. Server validates & stores
    3. Server finds recipient's server
    4. Forward to recipient
    5. Push notification if offline

Scaling:
  - Multiple WebSocket servers
  - Service discovery for routing
  - Message queue for async processing
  - Database sharding by conversation_id
```

## Additional Resources

- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [Designing Data-Intensive Applications](https://dataintensive.net/) by Martin Kleppmann
- [Web Architecture 101](https://engineering.videoblocks.com/web-architecture-101-a3224e126947)
- [High Scalability Blog](http://highscalability.com/)

---

*Last updated: 2025-11-16*
