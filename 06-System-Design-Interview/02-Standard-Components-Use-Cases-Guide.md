# Standard Components & Use Cases - Complete Guide

> **Critical**: For each component, answer two questions: (1) What problem does it solve? (2) What trade-offs does it bring?

---

## Table of Contents

1. [Database](#1-database)
2. [Cache](#2-cache)
3. [Message Queue](#3-message-queue)
4. [Blob Storage](#4-blob-storage)
5. [CDN (Content Delivery Network)](#5-cdn-content-delivery-network)
6. [Search Index](#6-search-index)
7. [Component Selection Framework](#component-selection-framework)

---

## 1. Database

### Problem it Solves

- **Persistent storage** of structured data
- **Query and retrieve** data efficiently
- **Maintain data integrity** through constraints
- **Handle concurrent** reads and writes

### Types and When to Use Each

#### Relational Database (PostgreSQL, MySQL)

**What problem does it solve?**

- Structured data with relationships
- ACID transactions
- Complex queries with JOINs
- Data integrity constraints

**Trade-offs:**

- ✅ Strong consistency guarantees
- ✅ Mature ecosystem and tooling
- ✅ Complex query support
- ✅ Referential integrity
- ❌ Vertical scaling limits
- ❌ Schema rigidity
- ❌ Horizontal scaling complexity
- ❌ Performance degrades at massive scale

**Real-World Examples:**

```
Use Case: E-commerce Order System
Problem: Need ACID transactions for order processing
Solution: PostgreSQL
Reason:
- Order creation involves multiple tables (orders, order_items, inventory)
- All updates must succeed or rollback atomically
- Strong consistency for inventory counts
- Complex queries for order history, analytics
```

---

#### Document Database (MongoDB, DynamoDB)

**What problem does it solve?**

- Flexible schema
- Hierarchical data structures
- Fast reads for independent documents
- Horizontal scalability

**Trade-offs:**

- ✅ Schema flexibility
- ✅ Easy horizontal scaling
- ✅ Fast document retrieval
- ✅ Handles semi-structured data
- ❌ Limited JOIN support
- ❌ Potential data duplication
- ❌ Eventual consistency (many deployments)
- ❌ Complex transactions difficult

**Real-World Examples:**

```
Use Case: Content Management System (CMS)
Problem: Blog posts have varying structures (text, images, videos, metadata)
Solution: MongoDB
Reason:
- Each post is independent
- Schema varies by content type
- No complex relationships
- Need to scale reads horizontally
- Fast retrieval of complete posts
```

---

#### Time-Series Database (InfluxDB, TimescaleDB)

**What problem does it solve?**

- Optimized for time-stamped data
- High write throughput
- Efficient compression
- Time-based queries and aggregations

**Trade-offs:**

- ✅ Extremely fast writes
- ✅ Efficient storage compression
- ✅ Time-based query optimization
- ❌ Limited to time-series use cases
- ❌ No general-purpose queries
- ❌ Specialized learning curve

**Real-World Examples:**

```
Use Case: IoT Sensor Data Collection
Problem: Millions of sensor readings per second
Solution: InfluxDB
Reason:
- Every data point has timestamp
- Append-only writes (no updates)
- Queries are time-range based
- Need downsampling and aggregation
- Massive write throughput required
```

---

#### Graph Database (Neo4j, Amazon Neptune)

**What problem does it solve?**

- Complex relationships between entities
- Path finding and traversal
- Network analysis
- Relationship-heavy queries

**Trade-offs:**

- ✅ Relationship queries are fast
- ✅ Intuitive for connected data
- ✅ Path traversal optimization
- ❌ Niche use case
- ❌ Scaling challenges
- ❌ Specialized knowledge required

**Real-World Examples:**

```
Use Case: Social Network Friend Recommendations
Problem: Find friends-of-friends, common connections
Solution: Neo4j
Reason:
- Relationship traversal is core operation
- "Friend-of-friend" queries are natural
- Path-based recommendations
- Network effects matter
```

---

### Database Selection Decision Tree

```
START
  ↓
Need relationships and ACID? ──YES→ Relational DB (PostgreSQL)
  ↓ NO
Primarily time-series data? ──YES→ Time-Series DB (InfluxDB)
  ↓ NO
Graph/network relationships? ──YES→ Graph DB (Neo4j)
  ↓ NO
Flexible schema needed? ──YES→ Document DB (MongoDB)
  ↓ NO
Simple key-value lookups? ──YES→ Key-Value Store (Redis/DynamoDB)
```

---

## 2. Cache

### Problem it Solves

- **Reduce latency** by keeping frequently accessed data in memory
- **Reduce database load** by serving repeated requests from cache
- **Improve throughput** by handling more requests
- **Cost savings** by reducing database queries

### Types and When to Use Each

#### In-Memory Cache (Redis, Memcached)

**What problem does it solve?**

- Sub-millisecond data access
- Reduce database load
- Session storage
- Rate limiting

**Trade-offs:**

- ✅ Extremely fast (microsecond latency)
- ✅ Reduces backend load significantly
- ✅ Flexible data structures (Redis)
- ✅ Pub/sub support (Redis)
- ❌ Volatile (data lost on restart)
- ❌ Limited by memory
- ❌ Cache invalidation complexity
- ❌ Additional infrastructure cost

**Real-World Examples:**

```
Use Case: User Session Management
Problem: Every request needs user session data
Solution: Redis cache
Reason:
- Session checked on every request (high read volume)
- Sub-millisecond latency needed
- TTL support for session expiration
- Can't afford database hit per request
- Key: session_token, Value: user_id, permissions, metadata

Performance Impact:
- Without cache: 50ms per request (database query)
- With cache: 1ms per request (Redis lookup)
- 50x improvement in response time
```

---

#### Application-Level Cache (In-Process)

**What problem does it solve?**

- Zero network latency
- Reduce even cache server load
- Perfect for static or rarely changing data

**Trade-offs:**

- ✅ Fastest possible (no network hop)
- ✅ No external dependency
- ✅ Free (uses app memory)
- ❌ Not shared across instances
- ❌ Invalidation difficult
- ❌ Limited by app memory
- ❌ Cold start on deployment

**Real-World Examples:**

```
Use Case: Configuration Data
Problem: App config loaded on every request
Solution: In-memory cache (HashMap/Dictionary)
Reason:
- Config changes rarely (hours/days)
- Same across all requests
- Don't need shared cache
- Eliminate all network calls

Implementation:
private static final Map<String, String> configCache = new ConcurrentHashMap<>();

// Load on startup, refresh every hour
```

---

### Caching Patterns

#### 1. Cache-Aside (Lazy Loading)

**How it works:**

```python
def get_user(user_id):
    # Try cache first
    user = cache.get(f"user:{user_id}")

    if user is None:  # Cache miss
        user = database.query(f"SELECT * FROM users WHERE id = {user_id}")
        cache.set(f"user:{user_id}", user, ttl=3600)  # 1 hour

    return user
```

**When to use:**

- ✅ Read-heavy workloads
- ✅ Unpredictable access patterns
- ✅ Cache failures acceptable

---

#### 2. Write-Through

**How it works:**

```python
def update_user(user_id, data):
    # Write to cache and database together
    database.update(user_id, data)
    cache.set(f"user:{user_id}", data)
```

**When to use:**

- ✅ Strong consistency needed
- ✅ Reads vastly exceed writes
- ✅ Can afford write latency

---

#### 3. Write-Behind (Write-Back)

**How it works:**

```python
def update_user(user_id, data):
    # Write to cache immediately
    cache.set(f"user:{user_id}", data)

    # Asynchronously write to database
    queue.enqueue("db_write", user_id, data)
```

**When to use:**

- ✅ Write-heavy workloads
- ✅ Can tolerate potential data loss
- ✅ Need fast write responses

---

### Cache Placement Strategies

**1. In Front of Database (Most Common)**

```
Client → App Server → Cache → Database
                         ↓
                    Cache Hit: Return
                    Cache Miss: Query DB
```

**2. Client-Side Cache (Browser)**

```
Browser Cache → CDN → App Server
```

**3. Multiple Cache Layers**

```
Browser → CDN → API Gateway Cache → App Cache → Redis → Database
```

---

### Common Cache Keys Design

| Use Case          | Key Pattern                    | Value                  | TTL              |
| ----------------- | ------------------------------ | ---------------------- | ---------------- |
| **User Profile**  | `user:{user_id}`               | User object JSON       | 1 hour           |
| **Session**       | `session:{token}`              | User session data      | Session duration |
| **Rate Limiting** | `ratelimit:{user_id}:{minute}` | Request count          | 1 minute         |
| **Feed**          | `feed:{user_id}`               | Array of post IDs      | 5 minutes        |
| **Product**       | `product:{product_id}`         | Product details        | 1 day            |
| **Popular Posts** | `trending:posts`               | Sorted set of post IDs | 10 minutes       |

---

## 3. Message Queue

### Problem it Solves

- **Asynchronous processing** - decouple sender and receiver
- **Load smoothing** - handle traffic spikes
- **Reliability** - retry failed operations
- **Scalability** - process tasks in parallel

### Types and When to Use Each

#### Traditional Queue (RabbitMQ, AWS SQS)

**What problem does it solve?**

- Point-to-point messaging
- Task distribution
- Background job processing
- Decoupling services

**Trade-offs:**

- ✅ Guaranteed delivery
- ✅ Message persistence
- ✅ Multiple consumers
- ✅ Dead letter queues
- ❌ Messages consumed once (not replayable)
- ❌ Complex routing can be tricky
- ❌ Ordering guarantees limited

**Real-World Examples:**

```
Use Case: Email Sending Service
Problem: Sending emails synchronously blocks request
Solution: RabbitMQ

Flow:
1. User registers → Create account in DB
2. Push email task to queue → Return success immediately
3. Background worker consumes from queue → Sends email
4. If email fails → Retry with exponential backoff

Benefits:
- User doesn't wait for email sending (slow SMTP)
- Can scale email workers independently
- Failures don't affect user experience
- Can handle email spikes (Black Friday campaigns)

Without Queue:
- User waits 2-3 seconds for email to send
- Email failure fails the entire registration
- Email spikes overload web servers

With Queue:
- User response in 100ms
- Email failures retried automatically
- Email workers scale independently
```

---

#### Event Stream (Kafka, AWS Kinesis)

**What problem does it solve?**

- Event sourcing
- Multiple consumers read same events
- Replay capability
- High throughput streaming

**Trade-offs:**

- ✅ Messages are replayable
- ✅ Multiple independent consumers
- ✅ Massive throughput
- ✅ Event log persistence
- ❌ More complex setup
- ❌ Overkill for simple queuing
- ❌ At-least-once delivery (need idempotency)

**Real-World Examples:**

```
Use Case: User Activity Tracking
Problem: Multiple systems need user events (analytics, recommendations, notifications)
Solution: Kafka

Event: User watched video
  ↓
Kafka Topic: "user-activity"
  ├→ Analytics Service (calculate watch time)
  ├→ Recommendation Service (update preferences)
  └→ Notification Service (suggest similar videos)

Benefits:
- One event, multiple consumers
- Services don't know about each other
- Can add new consumers without changing producers
- Can replay events to rebuild state

Example Event:
{
  "event_type": "video_watched",
  "user_id": 123,
  "video_id": 456,
  "timestamp": "2026-01-03T10:30:00Z",
  "watch_duration": 180
}
```

---

### Queue vs Stream Decision

| Aspect                  | Queue (RabbitMQ, SQS)   | Stream (Kafka, Kinesis)      |
| ----------------------- | ----------------------- | ---------------------------- |
| **Message Consumption** | Once (deleted after)    | Multiple times (replay)      |
| **Use Case**            | Background tasks        | Event sourcing, analytics    |
| **Ordering**            | Limited                 | Strong within partition      |
| **Throughput**          | Moderate                | Very high                    |
| **Complexity**          | Simple                  | Complex                      |
| **Consumers**           | Competing (shared work) | Independent (all get events) |

---

### Message Queue Patterns

#### 1. Task Queue Pattern

```
[Web Server] → Queue → [Worker 1]
                    → [Worker 2]
                    → [Worker 3]

Use: Image processing, PDF generation, data exports
```

#### 2. Pub/Sub Pattern

```
[Publisher] → Topic → [Subscriber 1: Email]
                  → [Subscriber 2: SMS]
                  → [Subscriber 3: Push]

Use: Notifications, event broadcasting
```

#### 3. Priority Queue Pattern

```
[Critical Tasks] → Priority 1 Queue → Workers
[Normal Tasks]   → Priority 2 Queue → Workers
[Low Priority]   → Priority 3 Queue → Workers

Use: SLA-based processing
```

---

## 4. Blob Storage

### Problem it Solves

- **Store large files** (images, videos, documents)
- **Cost-effective storage** at scale
- **Direct access** via URLs
- **CDN integration** for global distribution

### Types and When to Use Each

#### Object Storage (AWS S3, Google Cloud Storage, Azure Blob)

**What problem does it solve?**

- Virtually unlimited storage
- Highly durable (99.999999999% durability)
- HTTP-based access
- Scalable and cost-effective

**Trade-offs:**

- ✅ Unlimited scalability
- ✅ Extremely durable
- ✅ Cost-effective (pennies per GB)
- ✅ Built-in versioning, lifecycle
- ❌ Eventual consistency (in some operations)
- ❌ No file system semantics
- ❌ API-based access (not POSIX)
- ❌ Retrieval latency (compared to local disk)

**Real-World Examples:**

```
Use Case: Photo Sharing Application
Problem: Store millions of user-uploaded photos
Solution: AWS S3

Architecture:
1. User uploads photo → App Server
2. App generates unique filename: {user_id}/{timestamp}_{uuid}.jpg
3. Upload to S3: s3://my-app-photos/users/123/2026-01-03_abc123.jpg
4. Store metadata in database:
   - photo_id, user_id, s3_key, upload_date, size
5. Return S3 URL to client (or CDN URL)

Benefits:
- Pay only for storage used (~$0.023/GB/month)
- Don't manage disk space on app servers
- Built-in redundancy (11 9's durability)
- Can integrate with CDN for fast global access
- Automatic scaling

Cost Example:
- 1 million photos, average 2MB each = 2TB storage
- S3 Standard: ~$46/month
- Local storage: Need to manage disks, backups, scaling
```

---

#### File System vs Blob Storage

| Aspect         | File System (EBS, Disk)      | Blob Storage (S3)     |
| -------------- | ---------------------------- | --------------------- |
| **Capacity**   | Limited by disk              | Virtually unlimited   |
| **Durability** | Single disk failure risk     | 99.999999999%         |
| **Cost**       | High ($/GB)                  | Low (pennies/GB)      |
| **Access**     | POSIX file operations        | HTTP API              |
| **Latency**    | Very low                     | Moderate              |
| **Use Case**   | Application files, databases | User uploads, backups |

---

### Blob Storage Patterns

#### 1. Direct Upload (Small Files)

```
Client → App Server → S3
  ↓
Upload file in request body

Good for: Files < 5MB
Bad for: Large files (slow, timeout risk)
```

---

#### 2. Pre-Signed URL (Recommended)

```
1. Client requests upload URL → App Server
2. App generates pre-signed S3 URL (valid 15 min)
3. Client uploads directly to S3
4. Client notifies App Server (optional)

Benefits:
- App server not in data path
- Faster uploads
- Reduced server bandwidth
- Better user experience
```

**Example:**

```python
# Server-side
def get_upload_url(file_name, file_type):
    s3_key = f"uploads/{user_id}/{uuid.uuid4()}_{file_name}"

    url = s3_client.generate_presigned_url(
        'put_object',
        Params={
            'Bucket': 'my-bucket',
            'Key': s3_key,
            'ContentType': file_type
        },
        ExpiresIn=900  # 15 minutes
    )

    return {"upload_url": url, "s3_key": s3_key}

# Client-side
# 1. GET /api/upload-url?filename=photo.jpg
# 2. Receive pre-signed URL
# 3. PUT to pre-signed URL with file
# 4. POST /api/photos with s3_key
```

---

#### 3. Multipart Upload (Large Files)

```
For files > 100MB:
1. Split file into parts (5MB-5GB each)
2. Upload parts in parallel
3. Complete multipart upload

Benefits:
- Resume capability
- Parallel uploads (faster)
- Network resilience
```

---

### Storage Classes and Lifecycle

**S3 Storage Classes:**
| Class | Use Case | Cost | Retrieval Time |
|-------|----------|------|---------------|
| **Standard** | Frequently accessed | $0.023/GB | Milliseconds |
| **Infrequent Access** | Accessed monthly | $0.0125/GB | Milliseconds |
| **Glacier** | Archives | $0.004/GB | Minutes-hours |
| **Glacier Deep Archive** | Long-term backup | $0.00099/GB | 12 hours |

**Lifecycle Policy Example:**

```
Photos:
- 0-30 days: S3 Standard (frequently viewed)
- 31-90 days: S3 IA (occasional viewing)
- 91-365 days: Glacier (rare access)
- 365+ days: Glacier Deep Archive (compliance)

Automatically reduces costs over time!
```

---

## 5. CDN (Content Delivery Network)

### Problem it Solves

- **Reduce latency** by serving content from locations close to users
- **Reduce origin server load** by caching at edge
- **Improve availability** through geographic distribution
- **Handle traffic spikes** with massive edge capacity

### How CDN Works

```
User in Tokyo requests image
  ↓
DNS routes to Tokyo CDN edge
  ↓
Edge checks cache
  ├─ HIT: Return image (5ms latency)
  └─ MISS: Fetch from origin → Cache → Return (200ms first time, 5ms subsequent)

Without CDN:
- Every request goes to US origin server
- 200ms latency for Tokyo user
- Origin server handles all traffic

With CDN:
- First request: 200ms (cache miss)
- Subsequent requests: 5ms (cache hit)
- 95% of traffic served from edge (origin sees 5% of traffic)
```

---

### What to Cache on CDN

#### Static Assets (100% Cache Hit Rate)

```
✅ Images (JPEG, PNG, WebP)
✅ Videos
✅ CSS, JavaScript files
✅ Fonts
✅ Documents (PDFs)
✅ App bundles

Cache-Control: public, max-age=31536000 (1 year)
```

---

#### Semi-Dynamic Content (High Cache Hit Rate)

```
✅ Product pages (update every 5 minutes)
✅ News articles (update every minute)
✅ API responses (personalized but cacheable)

Cache-Control: public, max-age=300, s-maxage=300
```

---

#### Never Cache (0% Cache Hit Rate)

```
❌ User-specific data (dashboard, cart)
❌ Real-time data (stock prices, chat)
❌ POST/PUT/DELETE requests

Cache-Control: private, no-cache
```

---

### CDN Configuration Examples

#### CloudFront for E-Commerce

```yaml
Origin: www.mystore.com

Behaviors:
  /static/*:
    cache: 1 year
    compress: true

  /api/*:
    cache: no-cache
    forward-headers: Authorization

  /products/*:
    cache: 5 minutes
    compress: true
    query-strings: all

  /user/*:
    cache: no-cache
```

---

### Cache Invalidation Strategies

#### 1. Versioned URLs (Recommended)

```
Old: /static/styles.css
New: /static/styles.v2.css  or  /static/styles.css?v=2

Benefits:
- No invalidation needed
- Instant updates
- No cache consistency issues
```

---

#### 2. Cache Purge (Manual)

```
Deploy new version → Purge CDN cache → Wait for propagation

Drawbacks:
- Propagation delay (minutes)
- Costs money (on some CDNs)
- Risk of stale content
```

---

#### 3. Short TTL (Simple)

```
Cache-Control: max-age=300  # 5 minutes

Trade-off:
- Always reasonably fresh
- More origin requests
- Lower cache hit rate
```

---

### Real-World CDN Impact

**Example: Media Streaming App**

**Before CDN:**

```
Users: 1M daily active
Avg video size: 100MB
Traffic: 100TB/day
Origin bandwidth cost: $10,000/month (at $0.10/GB)
Latency: 500ms (global average)
```

**After CDN:**

```
Cache hit rate: 95%
Origin traffic: 5TB/day (95% served from edge)
Origin bandwidth cost: $500/month
CDN cost: $2,000/month
Total cost: $2,500/month (75% savings!)
Latency: 50ms (global average, 10x improvement)
```

---

## 6. Search Index

### Problem it Solves

- **Full-text search** across large datasets
- **Fast lookups** on non-indexed fields
- **Fuzzy matching** and autocomplete
- **Aggregations** and analytics

### When NOT to Use Primary Database for Search

**Problems with SQL LIKE queries:**

```sql
-- This is SLOW on large tables
SELECT * FROM products WHERE name LIKE '%phone%' OR description LIKE '%phone%';

Issues:
- Full table scan (no index can help)
- Gets slower linearly with data size
- Doesn't handle typos, relevance ranking
- Poor user experience
```

---

### Search Engines (Elasticsearch, Solr, Algolia)

**What problem does it solve?**

- Inverted index for fast full-text search
- Relevance scoring
- Typo tolerance
- Autocomplete / suggestions
- Faceted search (filters)

**Trade-offs:**

- ✅ Extremely fast search (milliseconds)
- ✅ Rich query capabilities
- ✅ Relevance ranking
- ✅ Scalable to billions of documents
- ❌ Eventually consistent (not source of truth)
- ❌ Complex to operate
- ❌ Additional infrastructure
- ❌ Costs (compute + storage)

---

### Real-World Examples

#### Example 1: E-Commerce Product Search

**Problem:**

- 10M products in PostgreSQL
- Users search: "wireless headphones bluetooth"
- Need results in < 100ms
- Need typo tolerance ("hedphones" → "headphones")
- Need filters (price, brand, rating)

**Solution: Elasticsearch**

**Architecture:**

```
PostgreSQL (Source of Truth)
  ↓ (sync via Kafka/Logstash)
Elasticsearch (Search Index)
  ↓ (search queries)
API returns results
```

**Data Flow:**

```
1. Product created/updated → PostgreSQL
2. Change captured (CDC or app) → Kafka
3. Logstash consumes → Indexes to Elasticsearch
4. User searches → Query Elasticsearch (not PostgreSQL)
5. Results returned with IDs → App enriches from PostgreSQL if needed
```

**Index Schema:**

```json
{
  "product_id": 12345,
  "name": "Sony WH-1000XM4 Wireless Headphones",
  "description": "Industry-leading noise cancellation...",
  "brand": "Sony",
  "category": "Electronics > Audio > Headphones",
  "price": 349.99,
  "rating": 4.7,
  "tags": ["wireless", "bluetooth", "noise-cancelling"],
  "created_at": "2026-01-01"
}
```

**Search Query:**

```json
{
  "query": {
    "multi_match": {
      "query": "wireless headphones bluetooth",
      "fields": ["name^3", "description", "tags^2"],
      "fuzziness": "AUTO"
    }
  },
  "filter": {
    "range": {"price": {"lte": 500}},
    "term": {"brand": "Sony"}
  },
  "aggs": {
    "brands": {"terms": {"field": "brand"}},
    "price_ranges": {"range": {"field": "price", "ranges": [...]}}
  }
}
```

**Results:**

- Search time: 20ms
- Facets returned: Brands, Price ranges
- Typo tolerant: "hedphones" finds "headphones"
- Relevance ranked: Name matches score higher

---

#### Example 2: Autocomplete

**Problem:**

- User types "iph" → Suggest "iphone 15", "iphone 14 pro", "iphone accessories"
- Must be instant (< 50ms)
- Must handle billions of searches

**Solution: Elasticsearch Completion Suggester**

**Index:**

```json
{
  "suggest": {
    "input": ["iphone 15", "iphone 15 pro", "iphone 15 pro max"],
    "weight": 1000 // based on popularity
  }
}
```

**Query:**

```json
{
  "suggest": {
    "product-suggest": {
      "prefix": "iph",
      "completion": { "field": "suggest" }
    }
  }
}
```

**Response (10ms):**

```json
["iPhone 15", "iPhone 15 Pro Max", "iPhone 14", "iPhone Accessories"]
```

---

### Search Index Sync Strategies

#### 1. Synchronous (Not Recommended)

```
User creates product
  → Save to PostgreSQL
  → Index to Elasticsearch
  → Return success

Problem: Slow, Elasticsearch failure fails entire request
```

---

#### 2. Asynchronous via Queue (Better)

```
User creates product
  → Save to PostgreSQL
  → Push to queue
  → Return success

Worker:
  → Consume from queue
  → Index to Elasticsearch

Problem: Eventual consistency, lag
```

---

#### 3. Change Data Capture (Best)

```
PostgreSQL Write-Ahead Log (WAL)
  → Debezium captures changes
  → Publishes to Kafka
  → Logstash consumes
  → Indexes to Elasticsearch

Benefits:
- Zero application code changes
- Guaranteed consistency (eventual)
- Can rebuild index from log
```

---

## Component Selection Framework

### Ask These Questions:

#### 1. What is the access pattern?

- **Read-heavy** → Add cache
- **Write-heavy** → Queue + async processing
- **Search-heavy** → Search index (Elasticsearch)

#### 2. What is the consistency requirement?

- **Strong consistency** → Relational DB, synchronous
- **Eventual consistency acceptable** → NoSQL, async, cache

#### 3. What is the latency requirement?

- **< 10ms** → Cache (Redis)
- **< 100ms** → Database with proper indexing
- **< 1s** → Acceptable for most APIs

#### 4. What is the scale?

- **< 10K RPS** → Single database, cache
- **10K-100K RPS** → Read replicas, cache, sharding
- **> 100K RPS** → Distributed systems, CDN, queue

#### 5. Is data structured or unstructured?

- **Structured with relationships** → Relational DB
- **Flexible schema** → Document DB
- **Binary files** → Blob storage

#### 6. Do multiple consumers need the same data?

- **No** → Queue (RabbitMQ, SQS)
- **Yes** → Stream (Kafka)

---

## Interview Decision Framework

When asked "How would you store X?":

```
Step 1: Clarify the data
  - Structured or unstructured?
  - Relationships?
  - Size?

Step 2: Clarify access patterns
  - Reads vs writes ratio?
  - Query patterns?
  - Search needed?

Step 3: Clarify requirements
  - Consistency?
  - Latency?
  - Scale?

Step 4: Choose components

Example:
"For user profiles: PostgreSQL (structured, relationships)
 For profile images: S3 (large files, blob storage)
 For profile cache: Redis (frequently accessed)
 For user search: Elasticsearch (full-text search)"
```

---

## Component Combinations (Common Patterns)

### Pattern 1: Web Application

```
User Request
  → CDN (static assets)
  → Load Balancer
  → App Server
  → Redis Cache
  → PostgreSQL
  → S3 (user uploads)
```

### Pattern 2: Event-Driven Microservices

```
Service A
  → Kafka (events)
  → Service B (consumer)
  → PostgreSQL (own database)
  → Elasticsearch (search index)
```

### Pattern 3: High-Throughput Analytics

```
Data Sources
  → Kafka (ingestion)
  → Stream Processor (Flink)
  → Time-Series DB (InfluxDB)
  → S3 (archive)
  → Elasticsearch (ad-hoc queries)
```

---

## Final Checklist

Before your interview, can you explain:

- [ ] When to use PostgreSQL vs MongoDB with specific reasoning?
- [ ] Three caching strategies with trade-offs?
- [ ] When to use a queue vs a stream?
- [ ] Why blob storage instead of database for files?
- [ ] How a CDN reduces both latency AND cost?
- [ ] When a search index is necessary vs database queries?

If yes to all, you're ready to choose components confidently! 🎯
