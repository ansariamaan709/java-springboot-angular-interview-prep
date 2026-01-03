# System Design Fundamentals - Complete Guide

> **Critical**: These fundamentals form the foundation of every system design interview. Master the top 6 concepts in each area and understand WHERE they show up in real systems.

---

## Table of Contents

1. [Storage Fundamentals](#storage-fundamentals)
2. [Scalability Fundamentals](#scalability-fundamentals)
3. [Networking Fundamentals](#networking-fundamentals)
4. [Performance and Reliability Fundamentals](#performance-and-reliability-fundamentals)

---

## Storage Fundamentals

### 1. Relational Databases (SQL)

**What it is:**

- Structured data storage with predefined schema
- Tables with rows and columns
- Relationships enforced through foreign keys
- Examples: PostgreSQL, MySQL, Oracle

**When to use:**

- ✅ Strong consistency requirements (banking, e-commerce transactions)
- ✅ Complex queries with JOINs
- ✅ ACID properties are critical
- ✅ Data has clear structure and relationships
- ✅ Need for referential integrity

**Trade-offs:**

- ❌ Harder to scale horizontally
- ❌ Schema changes can be expensive
- ❌ Performance degrades with massive scale
- ❌ Fixed schema can limit flexibility

**Real Interview Context:**

```
Interviewer: "Design a payment system"
You: "For payment transactions, I'd use a relational database because:
     - We need ACID guarantees (atomicity for debits/credits)
     - Strong consistency is non-negotiable
     - Complex queries for transaction history
     - Clear schema: users, accounts, transactions"
```

---

### 2. Document Databases (NoSQL)

**What it is:**

- Schema-less or flexible schema
- Stores data as JSON-like documents
- Examples: MongoDB, CouchDB, DynamoDB

**When to use:**

- ✅ Flexible or evolving schema
- ✅ Hierarchical data structures
- ✅ Need horizontal scalability
- ✅ Each document is independent
- ✅ Fast reads for specific documents

**Trade-offs:**

- ❌ Limited JOIN support
- ❌ Eventual consistency (in many cases)
- ❌ Data duplication common
- ❌ Complex transactions harder

**Real Interview Context:**

```
Interviewer: "Design a content management system"
You: "For storing blog posts and articles, I'd use MongoDB because:
     - Posts have varying structures (text, images, videos)
     - No complex relationships between posts
     - Schema flexibility for different content types
     - Easy horizontal scaling as content grows"
```

---

### 3. Key-Value Stores

**What it is:**

- Simplest NoSQL model
- Data stored as key-value pairs
- Extremely fast lookups by key
- Examples: Redis, Memcached, DynamoDB (can act as KV)

**When to use:**

- ✅ Caching layer
- ✅ Session storage
- ✅ Real-time leaderboards
- ✅ Rate limiting counters
- ✅ Simple data model with direct lookups

**Trade-offs:**

- ❌ No complex queries
- ❌ No relationships
- ❌ Limited data structures (in basic form)
- ❌ Not suitable for complex business logic

**Real Interview Context:**

```
Interviewer: "Design a rate limiter"
You: "I'd use Redis as a key-value store because:
     - Key: user_id or IP address
     - Value: request count and timestamp
     - Extremely fast lookups (sub-millisecond)
     - Built-in TTL for automatic cleanup
     - Atomic increment operations"
```

---

### 4. ACID vs BASE

#### ACID (Relational Databases)

**A - Atomicity:**

- All operations in a transaction succeed or all fail
- Example: Money transfer must debit AND credit, or neither

**C - Consistency:**

- Database moves from one valid state to another
- All constraints/rules are satisfied

**I - Isolation:**

- Concurrent transactions don't interfere
- Each transaction sees consistent snapshot

**D - Durability:**

- Once committed, data persists even if system crashes

**When you need ACID:**

- Financial transactions
- Inventory management
- Order processing
- Booking systems

#### BASE (NoSQL Systems)

**BA - Basically Available:**

- System guarantees availability
- Might not be the most recent data

**S - Soft State:**

- State may change over time without input
- Due to eventual consistency

**E - Eventual Consistency:**

- System will become consistent over time
- All replicas will eventually have same data

**When BASE is acceptable:**

- Social media feeds
- Product catalogs
- Analytics data
- Viewing history

---

### 5. Storage Decision Matrix

| Use Case                  | Storage Type                | Reasoning                              |
| ------------------------- | --------------------------- | -------------------------------------- |
| **Banking Transactions**  | Relational (PostgreSQL)     | ACID, strong consistency               |
| **User Profiles**         | Document (MongoDB)          | Flexible schema, independent documents |
| **Session Management**    | Key-Value (Redis)           | Fast access, TTL support               |
| **Product Catalog**       | Document or Relational      | Depends on relationships complexity    |
| **Order Management**      | Relational                  | Transactions, consistency              |
| **Social Media Posts**    | Document (MongoDB)          | Flexible content, high write volume    |
| **Cache Layer**           | Key-Value (Redis/Memcached) | Speed, simplicity                      |
| **Analytics Events**      | Column-Store (Cassandra)    | Time-series, write-heavy               |
| **File Metadata**         | Document or Relational      | Depends on query patterns              |
| **Real-time Leaderboard** | Key-Value (Redis)           | Sorted sets, atomic operations         |

---

## Scalability Fundamentals

### 1. Vertical Scaling (Scale Up)

**What it is:**

- Add more power to existing machine
- Increase CPU, RAM, disk

**When to use:**

- ✅ Initial stages
- ✅ Simple to implement
- ✅ No code changes needed
- ✅ Good for databases initially

**Limitations:**

- ❌ Hardware limits (can't scale infinitely)
- ❌ Single point of failure
- ❌ Expensive at high end
- ❌ Downtime during upgrades

**Real Interview Context:**

```
"We'd start with vertical scaling for the database to handle
initial growth, but plan for horizontal scaling as we hit limits."
```

---

### 2. Horizontal Scaling (Scale Out)

**What it is:**

- Add more machines
- Distribute load across multiple servers

**When to use:**

- ✅ Need unlimited scaling potential
- ✅ High availability requirements
- ✅ Geographical distribution
- ✅ Cost-effective at scale

**Challenges:**

- ❌ Complexity in code
- ❌ Data consistency issues
- ❌ State management
- ❌ Need load balancing

**Real Interview Context:**

```
"For the application servers, we'll use horizontal scaling:
 - Stateless services
 - Auto-scaling based on CPU/memory
 - Load balancer distributes traffic
 - Can add/remove instances dynamically"
```

---

### 3. Database Sharding

**What it is:**

- Partition data across multiple database instances
- Each shard holds a subset of data

**Sharding Strategies:**

#### A. Range-Based Sharding

```
User ID 1-1M    → Shard 1
User ID 1M-2M   → Shard 2
User ID 2M-3M   → Shard 3
```

**Pros:** Simple, range queries easy
**Cons:** Uneven distribution, hot shards

#### B. Hash-Based Sharding

```
hash(user_id) % num_shards = shard_number

user_123 → hash → 7 → Shard 2 (7 % 4 = 3... wait, off by one!)
```

**Pros:** Even distribution
**Cons:** Range queries difficult, resharding complex

#### C. Geographic Sharding

```
US users    → US Shard
EU users    → EU Shard
APAC users  → APAC Shard
```

**Pros:** Low latency, data residency compliance
**Cons:** Uneven growth, cross-region queries expensive

#### D. Entity-Based Sharding

```
All data for tenant_123 → Shard 1
All data for tenant_456 → Shard 2
```

**Pros:** Perfect for multi-tenant, transactions within tenant
**Cons:** Large tenants can overwhelm a shard

**Real Interview Context:**

```
Interviewer: "How would you shard a social media database?"
You: "I'd use hash-based sharding on user_id because:
     - Even distribution across shards
     - User's posts, likes, follows go to same shard
     - Downside: friend feed needs queries across shards
     - Mitigation: Cache friend feeds, aggregate asynchronously"
```

---

### 4. Database Replication

#### Master-Slave Replication

**Setup:**

```
Master (Write)
    ↓
    ├→ Slave 1 (Read)
    ├→ Slave 2 (Read)
    └→ Slave 3 (Read)
```

**When to use:**

- ✅ Read-heavy workloads (90% reads, 10% writes)
- ✅ Reporting/analytics queries
- ✅ Geographic distribution of readers

**Trade-offs:**

- ❌ Replication lag (eventual consistency)
- ❌ Single write bottleneck
- ❌ Master is single point of failure

#### Master-Master Replication

**Setup:**

```
Master 1 ←→ Master 2
(Both accept writes)
```

**When to use:**

- ✅ Write-heavy workloads
- ✅ High availability
- ✅ Multi-region writes

**Trade-offs:**

- ❌ Conflict resolution needed
- ❌ Complex to manage
- ❌ Potential data inconsistency

**Real Interview Context:**

```
Interviewer: "Design Instagram's database"
You: "For photos and metadata:
     - Master-slave replication with 3-5 read replicas
     - 95% of traffic is reads (browsing feed)
     - Writes only for uploads, likes, comments
     - Replication lag acceptable (few seconds delay is fine)
     - Read replicas can be geo-distributed"
```

---

### 5. Consistent Hashing

**Problem it solves:**

- Traditional hashing: `server = hash(key) % N`
- Adding/removing server rehashes EVERYTHING

**How consistent hashing works:**

```
Imagine a ring (0 to 2^32):
- Hash servers onto the ring
- Hash keys onto the ring
- Key goes to the next server clockwise

Benefits:
- Adding server: only 1/N keys rehash
- Removing server: only affected keys rehash
- Virtual nodes for even distribution
```

**When to use:**

- ✅ Distributed caching (Memcached, Redis cluster)
- ✅ Load balancing
- ✅ Distributed storage (Cassandra, DynamoDB)
- ✅ CDN routing

**Real Interview Context:**

```
Interviewer: "Design a distributed cache"
You: "I'd use consistent hashing because:
     - Cache servers can be added/removed without rehashing all keys
     - Virtual nodes ensure even distribution
     - Only 1/N keys need to move when scaling
     - Minimizes cache misses during topology changes"
```

---

## Networking Fundamentals

### 1. Request Flow (End-to-End)

**Complete Flow:**

```
1. User Browser
   ↓ DNS lookup
2. DNS Server → Returns IP
   ↓ HTTPS request
3. CDN (if static content) → Return or pass through
   ↓
4. Load Balancer (Layer 7 or Layer 4)
   ↓ Distributes to
5. Application Server
   ↓ Queries
6. Cache (Redis)
   ↓ If miss
7. Database
   ↓ Response
8. Application Server
   ↓ Format response
9. Load Balancer
   ↓
10. User Browser
```

**What you MUST explain in interviews:**

- Why each component exists
- What happens if it fails
- Where latency comes from
- How to optimize each step

---

### 2. REST vs gRPC vs WebSockets

#### REST (HTTP/1.1 or HTTP/2)

**Characteristics:**

- Request-response model
- Stateless
- Text-based (JSON/XML)
- Standard HTTP methods (GET, POST, PUT, DELETE)

**When to use:**

- ✅ Public APIs
- ✅ CRUD operations
- ✅ Request-response pattern
- ✅ Wide client compatibility
- ✅ Human-readable debugging

**Trade-offs:**

- ❌ Text overhead (larger payloads)
- ❌ No built-in streaming
- ❌ Slower than binary protocols

**Example:**

```http
GET /api/users/123
Response: {"id": 123, "name": "John", "email": "john@example.com"}
```

---

#### gRPC (HTTP/2)

**Characteristics:**

- Binary protocol (Protocol Buffers)
- Bidirectional streaming
- Strong typing
- Code generation

**When to use:**

- ✅ Microservices communication
- ✅ Performance-critical internal APIs
- ✅ Streaming data
- ✅ Polyglot environments (multiple languages)

**Trade-offs:**

- ❌ Not browser-friendly
- ❌ Harder to debug (binary)
- ❌ Steeper learning curve

**Example:**

```protobuf
service UserService {
  rpc GetUser(UserRequest) returns (UserResponse);
  rpc StreamUsers(Empty) returns (stream UserResponse);
}
```

---

#### WebSockets

**Characteristics:**

- Full-duplex persistent connection
- Real-time bidirectional communication
- Lower overhead than HTTP polling

**When to use:**

- ✅ Real-time chat
- ✅ Live notifications
- ✅ Collaborative editing
- ✅ Gaming
- ✅ Stock tickers

**Trade-offs:**

- ❌ Stateful (harder to scale)
- ❌ Connection management complexity
- ❌ Load balancing challenges

**Real Interview Context:**

```
Interviewer: "Design a chat application"
You: "I'd use WebSockets for real-time messaging because:
     - Need bidirectional, low-latency communication
     - Messages push instantly to connected users
     - Persistent connection avoids HTTP overhead

     But I'd use REST for:
     - Loading message history
     - User profile updates
     - Settings management"
```

---

### 3. Load Balancer - What it Actually Does

**Core Functions:**

#### A. Distributes Traffic

```
Algorithms:
1. Round Robin (simple rotation)
2. Least Connections (route to server with fewest active connections)
3. Weighted (based on server capacity)
4. IP Hash (same client → same server)
```

#### B. Health Checks

```
- Pings servers every N seconds
- Removes unhealthy servers from pool
- Adds them back when healthy
```

#### C. SSL Termination

```
- Handles HTTPS encryption/decryption
- Backend servers deal with plain HTTP
- Reduces CPU load on application servers
```

#### D. Layer 4 vs Layer 7

**Layer 4 (Transport Layer):**

- Looks at IP and port
- Faster (less inspection)
- No content-based routing
- TCP/UDP level

**Layer 7 (Application Layer):**

- Looks at HTTP headers, URLs, cookies
- Content-based routing
- `/api/*` → API servers
- `/images/*` → Image servers
- Slower but more flexible

**Real Interview Context:**

```
Interviewer: "How do you handle server failures?"
You: "Load balancer continuously health checks servers:
     - Every 5 seconds, ping each server
     - If 3 consecutive failures, mark as unhealthy
     - Route traffic only to healthy servers
     - When server recovers, gradually add back (warm-up)

     This gives us:
     - Automatic failover
     - Zero downtime deployments
     - Graceful handling of server crashes"
```

---

## Performance and Reliability Fundamentals

### 1. Latency vs Throughput

#### Latency

**Definition:** Time to complete ONE request
**Measured in:** Milliseconds (ms)

**What affects latency:**

- Network round trips
- Database query time
- Computation time
- Queue waiting time

**How to improve:**

- ✅ Caching
- ✅ CDN for static content
- ✅ Database indexing
- ✅ Reduce network hops
- ✅ Geographic distribution

**Example:**

```
API response time: 200ms
- 50ms network
- 100ms database query
- 30ms business logic
- 20ms serialization
```

---

#### Throughput

**Definition:** Requests processed per unit time
**Measured in:** Requests per second (RPS), transactions per second (TPS)

**What affects throughput:**

- Number of servers
- Server capacity
- Bottlenecks
- Concurrency

**How to improve:**

- ✅ Horizontal scaling
- ✅ Async processing
- ✅ Batch operations
- ✅ Remove bottlenecks
- ✅ Connection pooling

**Example:**

```
System handles 10,000 RPS
Each server: 1,000 RPS
Need 10 servers
```

**Real Interview Context:**

```
Interviewer: "System is slow. How do you improve?"
You: "First, identify if it's a latency or throughput problem:

     If LATENCY (individual requests slow):
     - Add caching layer
     - Optimize database queries
     - Use CDN for static assets

     If THROUGHPUT (system overloaded):
     - Add more servers
     - Use message queues to smooth spikes
     - Implement rate limiting"
```

---

### 2. Caching Strategies

#### Cache-Aside (Lazy Loading)

**How it works:**

```
1. App checks cache
2. If HIT: return data
3. If MISS:
   - Query database
   - Write to cache
   - Return data
```

**Pros:**

- ✅ Only cache what's needed
- ✅ Cache failure doesn't bring down system
- ✅ Flexible

**Cons:**

- ❌ First request is slow (cache miss)
- ❌ Stale data possible

**When to use:**

- Read-heavy workloads
- Unpredictable access patterns

---

#### Write-Through

**How it works:**

```
1. App writes to cache AND database simultaneously
2. Write succeeds only when both complete
```

**Pros:**

- ✅ Cache always up-to-date
- ✅ No stale data

**Cons:**

- ❌ Higher write latency
- ❌ Unnecessary caching of unused data

**When to use:**

- Strong consistency needed
- Reads vastly exceed writes

---

#### Write-Behind (Write-Back)

**How it works:**

```
1. App writes to cache
2. Cache asynchronously writes to database
3. Write returns immediately
```

**Pros:**

- ✅ Fast writes
- ✅ Batching possible
- ✅ Reduced database load

**Cons:**

- ❌ Risk of data loss if cache fails
- ❌ Complex to implement

**When to use:**

- Write-heavy workloads
- Can tolerate some data loss

---

#### Refresh-Ahead

**How it works:**

```
1. Automatically refresh cache before expiration
2. Based on access patterns
```

**Pros:**

- ✅ No cache misses for hot data
- ✅ Predictable latency

**Cons:**

- ❌ Wasted resources if data not accessed
- ❌ Complex logic

**When to use:**

- Predictable access patterns
- Low-latency requirements

---

### 3. Cache Eviction Policies

| Policy                          | How it Works                              | When to Use                      |
| ------------------------------- | ----------------------------------------- | -------------------------------- |
| **LRU** (Least Recently Used)   | Evicts item not accessed for longest time | General purpose, good default    |
| **LFU** (Least Frequently Used) | Evicts item accessed least often          | Long-term popularity matters     |
| **FIFO** (First In First Out)   | Evicts oldest item                        | Simple, predictable patterns     |
| **TTL** (Time To Live)          | Evicts after fixed time                   | Session data, temporary data     |
| **Random**                      | Evicts random item                        | Simplest, surprisingly effective |

---

### 4. Replication for Reliability

**Purpose:**

- Data redundancy
- High availability
- Disaster recovery
- Geographic distribution

**Synchronous Replication:**

```
Write to Primary
    ↓ (wait for confirmation)
Write to Replica
    ↓ (both succeed)
Acknowledge to client
```

**Pros:** Strong consistency, no data loss
**Cons:** Higher latency, availability risk

**Asynchronous Replication:**

```
Write to Primary
    ↓ (immediate ack)
Acknowledge to client
    ↓ (background)
Replicate to replicas
```

**Pros:** Low latency, high availability
**Cons:** Potential data loss, eventual consistency

---

### 5. Failover Strategies

#### Active-Passive (Hot Standby)

```
Primary (Active) → Handles all traffic
    ↓ (heartbeat)
Secondary (Passive) → Standby, ready to take over
```

**When primary fails:**

- Secondary promoted to primary
- DNS updated or load balancer switches

**Recovery Time:** Seconds to minutes

---

#### Active-Active (Hot-Hot)

```
Primary ←→ Secondary
(Both handle traffic)
```

**When one fails:**

- Other continues serving
- No promotion needed

**Recovery Time:** Instant

---

### 6. CAP Theorem

**You can have at most 2 of 3:**

#### C - Consistency

Every read receives the most recent write

#### A - Availability

Every request receives a response (success or failure)

#### P - Partition Tolerance

System continues despite network partitions

**Trade-offs:**

| System Type | Chooses                    | Example             | Use Case                           |
| ----------- | -------------------------- | ------------------- | ---------------------------------- |
| **CP**      | Consistency + Partition    | MongoDB, HBase      | Banking, inventory                 |
| **AP**      | Availability + Partition   | Cassandra, DynamoDB | Social media, analytics            |
| **CA**      | Consistency + Availability | Traditional RDBMS   | Single-node only (not distributed) |

**Real Interview Context:**

```
Interviewer: "Design a shopping cart"
You: "This is an AP system (Availability + Partition tolerance):
     - Users must always be able to add items (availability)
     - Seeing slightly stale cart is acceptable
     - Network partitions shouldn't block cart operations
     - Eventual consistency: carts sync when partition heals

     But for CHECKOUT, we need CP:
     - Inventory deduction requires strong consistency
     - Better to fail than oversell
     - Accept temporary unavailability if needed"
```

---

## Top 6 Concepts to Master (Summary)

### 1. **Storage**

- When to use SQL vs NoSQL vs Key-Value
- ACID vs BASE trade-offs

### 2. **Sharding**

- How to partition data
- Strategies and trade-offs

### 3. **Caching**

- Where to cache
- Eviction policies
- Cache invalidation

### 4. **Load Balancing**

- Algorithms
- Health checks
- Layer 4 vs Layer 7

### 5. **Replication**

- Sync vs async
- Master-slave vs master-master

### 6. **CAP Theorem**

- What to sacrifice when
- CP vs AP systems

---

## How to Study These Fundamentals

### Week 1: Storage

- Day 1-2: SQL vs NoSQL, when to pick what
- Day 3-4: ACID vs BASE with real examples
- Day 5: Practice explaining storage choices for 5 different systems

### Week 2: Scalability

- Day 1-2: Sharding strategies
- Day 3-4: Replication and consistent hashing
- Day 5: Practice scaling a database for 3 different scenarios

### Week 3: Networking

- Day 1-2: Request flow, load balancing
- Day 3-4: REST vs gRPC vs WebSockets
- Day 5: Practice explaining communication choices

### Week 4: Performance

- Day 1-2: Caching strategies
- Day 3-4: CAP theorem
- Day 5: Practice trade-off discussions

---

## Interview Red Flags to Avoid

❌ **"We'll use microservices for scalability"** (without explaining why)
✅ **"We'll start with a monolith, then extract services when we identify clear boundaries and scaling needs"**

❌ **"We'll shard the database"** (without explaining the strategy)
✅ **"We'll use hash-based sharding on user_id because... [specific reasoning]"**

❌ **"We'll add caching"** (vague)
✅ **"We'll use cache-aside with Redis, TTL of 1 hour, for user profile data because 90% of our reads are profile lookups"**

❌ **"We need to handle millions of users"** (without clarifying)
✅ **"Assuming 10M DAU, 50 requests per user per day, that's 500M requests daily or ~6K RPS on average"**

---

## Final Checklist Before Interview

Can you explain in plain language:

- [ ] Why you'd choose PostgreSQL over MongoDB for a specific use case?
- [ ] How consistent hashing minimizes cache invalidation?
- [ ] The difference between latency and throughput with examples?
- [ ] When you'd choose AP over CP in CAP theorem?
- [ ] How a load balancer detects and handles server failures?
- [ ] Three different sharding strategies with pros/cons?

If yes to all, your fundamentals are interview-ready. 🎯
