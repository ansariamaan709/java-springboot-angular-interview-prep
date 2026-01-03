# Practice Problems & Patterns - Complete Guide

> **Critical**: Do three passes for each problem: (1) Watch/read solution, (2) Do it yourself on timer, (3) Teach it out loud. This is how you internalize patterns.

---

## Table of Contents

1. [The 10 Essential Patterns](#the-10-essential-patterns)
2. [Pattern 1: Content Feed](#pattern-1-content-feed-social-media-feed)
3. [Pattern 2: File Storage and Sharing](#pattern-2-file-storage-and-sharing-google-drive)
4. [Pattern 3: Messaging or Chat](#pattern-3-messaging-or-chat-whatsapp)
5. [Pattern 4: Rate Limiter or API Gateway](#pattern-4-rate-limiter-or-api-gateway)
6. [Pattern 5: Search and Autocomplete](#pattern-5-search-and-autocomplete-google-search)
7. [Pattern 6: Analytics or Click Tracking](#pattern-6-analytics-or-click-tracking-google-analytics)
8. [Pattern 7: Ticket Booking or Reservation](#pattern-7-ticket-booking-or-reservation-ticketmaster)
9. [Pattern 8: Notification Service](#pattern-8-notification-service-push-sms-email)
10. [Pattern 9: URL Shortener](#pattern-9-url-shortener-bitly)
11. [Pattern 10: Video Streaming](#pattern-10-video-streaming-youtube-netflix)
12. [How to Practice Effectively](#how-to-practice-effectively)

---

## The 10 Essential Patterns

These 10 patterns cover **90% of system design interviews**:

| Pattern             | Example Systems              | Key Concepts                               |
| ------------------- | ---------------------------- | ------------------------------------------ |
| **Content Feed**    | Twitter, Instagram, Facebook | Fanout, timeline generation, ranking       |
| **File Storage**    | Google Drive, Dropbox        | Blob storage, chunking, sync               |
| **Messaging**       | WhatsApp, Slack, Discord     | WebSockets, message queues, delivery       |
| **Rate Limiter**    | API Gateway, DDoS protection | Token bucket, sliding window, distributed  |
| **Search**          | Google, Elasticsearch        | Inverted index, autocomplete, ranking      |
| **Analytics**       | Google Analytics, Mixpanel   | Stream processing, aggregation, real-time  |
| **Booking**         | Ticketmaster, Airbnb         | Consistency, locking, inventory            |
| **Notifications**   | Push, SMS, Email             | Delivery guarantee, queues, templates      |
| **URL Shortener**   | bit.ly, TinyURL              | Base62 encoding, key generation, analytics |
| **Video Streaming** | YouTube, Netflix             | CDN, encoding, adaptive bitrate            |

**Master these 10, and you can handle variations easily.**

---

## Pattern 1: Content Feed (Social Media Feed)

### Example: Design Twitter / Instagram Feed

### What Makes This Pattern Unique

- **Core challenge:** Generating personalized feed for millions of users efficiently
- **Key concepts:** Fanout on write vs read, timeline generation, ranking
- **Variants:** Twitter, Instagram, Facebook, LinkedIn feed

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ Users can post content (text/images)
- ✅ Users can follow others
- ✅ Users see feed of posts from people they follow
- ✅ Users can like and comment

**Non-Functional:**

- 200M DAU
- Avg 2 posts/day per user
- Read:Write = 100:1
- Feed load < 200ms (p99)
- Eventual consistency OK

**Calculations:**

```
Writes: 200M × 2 = 400M posts/day = 4,630 posts/sec (peak: 14K/sec)
Reads: 4,630 × 100 = 463K reads/sec (peak: 1.4M/sec)
Storage: 400M posts × 1KB = 400GB/day = 146TB/year
```

---

### Step 2: Core Entities & APIs (5 min)

**Entities:**

```
1. User (user_id, username, follower_count, following_count)
2. Post (post_id, user_id, content, media_urls, created_at)
3. Follow (follower_id, followee_id, created_at)
4. Like (user_id, post_id, created_at)
5. Timeline (user_id, post_ids[], last_updated)
```

**APIs:**

```
1. POST /api/posts - Create post
2. GET /api/feed/{user_id} - Get user's feed
3. POST /api/follow - Follow a user
4. POST /api/posts/{post_id}/like - Like a post
5. GET /api/users/{user_id} - Get user profile
```

---

### Step 3: High-Level Design (10 min)

```
┌─────────┐
│ Client  │
└────┬────┘
     │
     ↓
┌────────────┐
│    CDN     │ (Images/videos)
└────┬───────┘
     │
     ↓
┌────────────┐
│Load Balancer│
└────┬───────┘
     │
     ↓
┌────────────────┐
│  App Servers   │
└──┬─────────┬───┘
   │         │
   ↓         ↓
┌─────┐  ┌──────────┐
│Redis│  │PostgreSQL│
│Cache│  │          │
└─────┘  └──────────┘
   ↑
   │ (Precomputed feeds)
   │
┌──────────┐
│ Timeline │
│  Service │
└──────────┘
```

**Request Flow: Post Creation**

```
1. User creates post
2. App server writes to database (posts table)
3. Push to message queue (Kafka) for fanout
4. Timeline service consumes from queue
5. For each follower:
   - Update timeline cache (Redis)
   - Key: feed:{follower_id}
   - Value: [post_id_1, post_id_2, ..., post_id_N]
```

**Request Flow: Feed Fetch**

```
1. User requests feed
2. App server checks Redis cache
3. Key: feed:{user_id}
4. If HIT: Return post IDs, fetch post details (batch query or cache)
5. If MISS: Fall back to database query (slow path)
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive 1: Fanout Strategy (Critical!)

**Problem:** When user posts, how do we update followers' feeds?

**Approach 1: Fanout on Write (Push)**

```
When user posts:
  → For each follower:
    → Add post_id to their timeline cache

Pros:
✅ Fast reads (timeline precomputed)
✅ Predictable read latency

Cons:
❌ Slow writes for celebrities (10M followers = 10M writes)
❌ Wasted work if follower never reads
```

**Approach 2: Fanout on Read (Pull)**

```
When user requests feed:
  → Query posts from all followed users
  → Merge and sort

Pros:
✅ Fast writes (no fanout)
✅ No wasted work

Cons:
❌ Slow reads (complex join query)
❌ Scales poorly with many followings
```

**Approach 3: Hybrid (Twitter's Approach)**

```
Normal users (< 10K followers):
  → Fanout on write

Celebrities (> 10K followers):
  → Fanout on read

Feed generation:
  → Merge precomputed timeline + celebrity tweets on-the-fly
  → Cache merged result for 60 seconds
```

**Decision:**

```
Use hybrid approach:
- 99% of users get instant feeds (fanout-on-write)
- Celebrities don't cause 10M fanout writes
- Feed read latency: < 50ms (mostly cached)
- Celebrity tweets fetched on-demand: +20ms
- Total: ~70ms well within 200ms target
```

---

#### Deep Dive 2: Feed Ranking

**Problem:** Show most relevant posts, not just chronological

**Approach:**

```
1. Fetch candidate posts from timeline cache (chronological)
2. Score each post:
   score = f(recency, engagement, user_affinity)

   - recency: exponential decay (posts from last hour score higher)
   - engagement: like_count + comment_count × 2
   - user_affinity: how often user interacts with author

3. Re-rank by score
4. Return top 20 posts
```

**Implementation:**

```
Async Pipeline:
  → Fetch posts (from cache): 10ms
  → Fetch engagement metrics (batch query): 20ms
  → Compute scores (in-memory): 5ms
  → Sort: 5ms
  → Total: 40ms
```

**Trade-off:**

```
- Adds 40ms latency
- But significantly improves user engagement
- Can A/B test: chronological vs ranked
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- Fanout on write vs read vs hybrid
- Timeline caching strategies
- Celebrity/hot user handling
- Feed ranking algorithms

**Related problems:**

- Facebook newsfeed
- LinkedIn feed
- Instagram feed
- Reddit home page

---

## Pattern 2: File Storage and Sharing (Google Drive)

### Example: Design Dropbox / Google Drive

### What Makes This Pattern Unique

- **Core challenge:** Storing large files, syncing across devices
- **Key concepts:** Chunking, deduplication, conflict resolution
- **Variants:** Dropbox, Google Drive, OneDrive

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ Upload/download files
- ✅ Sync files across devices
- ✅ Share files with other users
- ✅ Version history

**Non-Functional:**

- 100M users
- 10GB storage per user (avg)
- File size: up to 10GB
- Sync latency: < 1 minute

**Calculations:**

```
Total storage: 100M × 10GB = 1 Exabyte (with replication: 3 EB)
Upload traffic: Assume 10% active daily, 100MB/day → 1PB/day
```

---

### Step 2: Core Entities & APIs (5 min)

**Entities:**

```
1. User (user_id, email, storage_used, storage_limit)
2. File (file_id, user_id, filename, size, version, created_at)
3. Chunk (chunk_id, file_id, chunk_hash, s3_key, size)
4. Device (device_id, user_id, last_sync_time)
```

**APIs:**

```
1. POST /api/files/upload - Upload file
2. GET /api/files/{file_id}/download - Download file
3. GET /api/files/sync - Get file changes since last sync
4. POST /api/files/{file_id}/share - Share file
5. GET /api/files/{file_id}/versions - Get version history
```

---

### Step 3: High-Level Design (10 min)

```
┌─────────┐
│ Client  │
│ (Desktop/│
│  Mobile) │
└────┬────┘
     │
     ↓
┌────────────┐
│    API     │
│  Gateway   │
└─────┬──────┘
      │
      ├──→ ┌──────────────┐
      │    │ File Service │
      │    └──────┬───────┘
      │           │
      │           ↓
      │    ┌────────────┐
      │    │ PostgreSQL │ (Metadata)
      │    └────────────┘
      │
      ├──→ ┌──────────────┐
      │    │Sync Service  │
      │    └──────┬───────┘
      │           │
      │           ↓
      │    ┌────────────┐
      │    │ Message    │
      │    │ Queue      │
      │    └────────────┘
      │
      └──→ ┌──────────────┐
           │ S3 / Blob    │
           │ Storage      │
           └──────────────┘
```

**Upload Flow:**

```
1. Client splits file into chunks (4MB each)
2. Compute hash for each chunk (SHA-256)
3. Check metadata service: does chunk exist? (deduplication)
4. If new chunk:
   - Get pre-signed S3 URL
   - Upload directly to S3
5. Update metadata in PostgreSQL:
   - file_id, chunks mapping, version
6. Notify other devices via message queue
```

**Download Flow:**

```
1. Client requests file metadata
2. Get list of chunk_ids and S3 keys
3. Download chunks in parallel from S3
4. Reassemble file locally
5. Verify integrity (hash)
```

**Sync Flow:**

```
1. Device polls /api/files/sync?since={last_sync_timestamp}
2. Server returns changed files
3. Device downloads changed chunks
4. Conflict detection (if same file modified on multiple devices)
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive 1: Chunking and Deduplication

**Why Chunking?**

```
Benefits:
✅ Resume uploads (if connection drops, only re-upload failed chunk)
✅ Parallel uploads (faster)
✅ Deduplication (save storage)
✅ Delta sync (only changed chunks)
```

**Chunk Size Trade-off:**

```
Small chunks (1MB):
  ✅ Fine-grained deduplication
  ❌ More metadata overhead
  ❌ More S3 requests

Large chunks (10MB):
  ✅ Less metadata
  ✅ Fewer S3 requests
  ❌ Coarse deduplication

Sweet spot: 4MB chunks
```

**Deduplication:**

```
When uploading chunk:
1. Compute SHA-256 hash: abc123def456...
2. Check database: SELECT chunk_id FROM chunks WHERE chunk_hash = 'abc123...'
3. If exists:
   - Reference existing chunk (don't upload)
   - Save storage!
4. If not exists:
   - Upload to S3
   - Store metadata

Example:
User A uploads "video.mp4" (100MB)
User B uploads same "video.mp4"
  → Deduplication saves 100MB
  → Only store once, reference twice
```

---

#### Deep Dive 2: Conflict Resolution

**Problem:** User edits file on phone and laptop offline, then both sync

**Approach 1: Last Write Wins**

```
- Simplest
- Laptop sync at 10:00 AM → Overwrites
- Phone sync at 10:01 AM → Overwrites laptop's changes
- ❌ Data loss possible
```

**Approach 2: Version Control (Better)**

```
- Each edit creates new version
- Both versions saved
- User manually merges
- ✅ No data loss
- ❌ User burden
```

**Approach 3: Operational Transformation (Complex)**

```
- Merge edits automatically (like Google Docs)
- ✅ Seamless
- ❌ Extremely complex for arbitrary files
```

**Decision for Dropbox-like system:**

```
Use Version Control:
- When conflict detected:
  → Save both versions
  → file.txt (Original)
  → file (Laptop's conflicted copy).txt
  → file (Phone's conflicted copy).txt
- User chooses which to keep
- Simple, no data loss
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- File chunking strategies
- Deduplication via content hashing
- Pre-signed URLs for direct S3 upload
- Conflict resolution approaches
- Metadata vs blob storage separation

**Related problems:**

- Google Photos
- iCloud
- OneDrive
- File backup systems

---

## Pattern 3: Messaging or Chat (WhatsApp)

### Example: Design WhatsApp / Slack

### What Makes This Pattern Unique

- **Core challenge:** Real-time bidirectional communication
- **Key concepts:** WebSockets, message delivery guarantees, offline support
- **Variants:** WhatsApp, Slack, Discord, iMessage

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ 1-on-1 chat
- ✅ Group chat (up to 256 members)
- ✅ Message delivery status (sent, delivered, read)
- ✅ Offline message storage
- ✅ Message history

**Non-Functional:**

- 1B users, 500M DAU
- Avg 50 messages per user per day
- Message delivery < 1 second
- 99.99% availability

**Calculations:**

```
Messages/day: 500M × 50 = 25B messages/day
Messages/sec: 25B / 86,400 = 289K messages/sec (peak: 870K/sec)
Storage: 25B × 1KB = 25TB/day = 9PB/year
```

---

### Step 2: Core Entities & APIs (5 min)

**Entities:**

```
1. User (user_id, phone, username, last_seen)
2. Message (message_id, sender_id, receiver_id, content, timestamp, status)
3. Conversation (conversation_id, participants[], type, created_at)
4. Device (device_id, user_id, websocket_connection)
```

**APIs (WebSocket events):**

```
1. send_message - Send a message
2. message_received - Notify delivery
3. message_read - Mark as read
4. typing_indicator - User is typing
5. get_history - Fetch message history
```

---

### Step 3: High-Level Design (10 min)

```
┌──────────┐                         ┌──────────┐
│ Client A │                         │ Client B │
└────┬─────┘                         └────┬─────┘
     │                                    │
     │ WebSocket                          │ WebSocket
     ↓                                    ↓
┌─────────────────────────────────────────────┐
│        WebSocket Gateway (Stateful)          │
│   Maintains persistent connections           │
└────┬─────────────────────────────┬──────────┘
     │                              │
     ↓                              ↓
┌──────────┐                  ┌──────────┐
│  Chat    │                  │ Presence │
│ Service  │                  │ Service  │
└────┬─────┘                  └──────────┘
     │
     ↓
┌──────────┐      ┌──────────┐      ┌──────────┐
│PostgreSQL│      │  Redis   │      │  Kafka   │
│(History) │      │ (Temp)   │      │ (Queue)  │
└──────────┘      └──────────┘      └──────────┘
```

**Message Flow (1-on-1 chat):**

```
1. User A sends message via WebSocket
2. Gateway receives message
3. Chat Service:
   - Generate message_id (Snowflake)
   - Store in database (PostgreSQL)
   - Store in Redis (temporary, fast access)
   - Publish to Kafka (for delivery)
4. Delivery Service:
   - Consume from Kafka
   - Check if User B is online (Presence Service)
   - If online: Push via WebSocket to User B
   - If offline: Store for later delivery
5. User B receives message:
   - Send acknowledgment (delivered)
6. User A gets delivery confirmation
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive 1: Message Delivery Guarantees

**Problem:** Ensure message is delivered exactly once

**Challenges:**

```
1. Network failures (message sent but ack lost)
2. Server crashes (message in memory but not persisted)
3. Duplicate deliveries (retry sends duplicate)
```

**Solution: At-Least-Once + Idempotency**

**Sender Side:**

```
1. Client generates unique message_id (UUID)
2. Sends message with message_id
3. If no ack within 5 seconds → Retry
4. Keep retrying until ack received
```

**Receiver Side:**

```
1. Check if message_id already processed (Redis set)
2. If yes: Return ack (deduplicate)
3. If no:
   - Store message
   - Add message_id to processed set (TTL: 7 days)
   - Return ack
```

**Message Status:**

```
1. Sent (client sent, server received)
2. Delivered (server pushed to recipient)
3. Read (recipient opened message)

Implementation:
- Each status update is a separate ack
- Sent → Server stores in DB
- Delivered → Server pushed via WebSocket
- Read → Recipient sends read receipt
```

---

#### Deep Dive 2: Group Chat Fanout

**Problem:** User sends message to group of 256 members

**Naive Approach:**

```
For each member in group:
  → Send message via WebSocket

Issues:
❌ 256 sequential sends (slow)
❌ Blocks sender until all sent
```

**Better Approach: Async Fanout**

```
1. Sender sends message to Chat Service
2. Chat Service:
   - Stores message once
   - Publishes to Kafka topic: group_{group_id}
   - Returns ack to sender immediately
3. Fanout Service:
   - Consumes from Kafka
   - For each group member:
     → If online: Push via WebSocket
     → If offline: Increment unread counter
4. Parallel delivery to online members
```

**Optimization: Batching**

```
Instead of 256 individual messages:
→ Batch online members by WebSocket server
→ Server A has 100 members → 1 message
→ Server B has 50 members → 1 message
→ Server C has 106 members → 1 message
→ Total: 3 messages instead of 256
```

---

#### Deep Dive 3: Online Presence

**Problem:** Show "last seen" and "online" status

**Approach:**

```
Presence Service:
1. When user connects:
   - Set Redis: user:{user_id}:online = true (TTL: 60 sec)
   - Set Redis: user:{user_id}:last_seen = timestamp

2. Client sends heartbeat every 30 sec
   - Refreshes TTL

3. If no heartbeat for 60 sec:
   - Redis key expires
   - User marked offline

4. Last seen query:
   - Check Redis: user:{user_id}:online
   - If true → "Online"
   - If false → "Last seen at {user:{user_id}:last_seen}"
```

**Scalability:**

```
500M users online:
  → 500M heartbeats every 30 sec
  → ~16M heartbeats/sec
  → Solution: Redis cluster with sharding by user_id
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- WebSocket for bidirectional communication
- Message delivery guarantees (at-least-once)
- Idempotency for deduplication
- Fanout strategies for group messages
- Presence tracking with TTL

**Related problems:**

- Telegram
- Signal
- Discord
- Microsoft Teams

---

## Pattern 4: Rate Limiter or API Gateway

### Example: Design a Rate Limiter / API Gateway

### What Makes This Pattern Unique

- **Core challenge:** Prevent abuse while allowing legitimate traffic
- **Key concepts:** Token bucket, sliding window, distributed rate limiting
- **Variants:** DDoS protection, API throttling, login attempt limiting

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ Limit requests per user/IP
- ✅ Different limits for different endpoints
- ✅ Return clear error when limit exceeded

**Non-Functional:**

- Handle 1M RPS
- Low latency overhead (< 5ms)
- Accurate rate limiting (minimal false positives)
- Distributed (works across multiple servers)

---

### Step 2: Rate Limiting Algorithms

#### Algorithm 1: Fixed Window Counter

**How it works:**

```
Window: 0-60 seconds
User makes 100 requests between 0-60 sec → Block after 100
At 61 sec → Counter resets

Key: rate_limit:{user_id}:{minute}
Value: request_count
TTL: 60 seconds
```

**Pros:**

- ✅ Simple
- ✅ Memory efficient

**Cons:**

- ❌ Burst at window edges (100 at 11:59:59, 100 at 12:00:01 = 200 in 2 sec)

---

#### Algorithm 2: Sliding Window Log

**How it works:**

```
Store timestamp of each request
Key: rate_limit:{user_id}
Value: [timestamp1, timestamp2, timestamp3, ...]

On new request:
1. Remove timestamps older than 60 sec
2. Count remaining timestamps
3. If < 100 → Allow, add new timestamp
4. If >= 100 → Reject
```

**Pros:**

- ✅ Accurate (no burst problem)

**Cons:**

- ❌ Memory intensive (store every request)

---

#### Algorithm 3: Sliding Window Counter (Best)

**How it works:**

```
Hybrid of fixed window + sliding window
Estimate current count based on previous and current window

Current minute: 12:00:00 - 12:00:59
Previous minute: 11:59:00 - 11:59:59

At timestamp 12:00:30 (30 seconds into current minute):
  count = (prev_count × 0.5) + current_count

If result < 100 → Allow
If result >= 100 → Reject
```

**Pros:**

- ✅ Accurate (minimal burst)
- ✅ Memory efficient (only 2 counters)

**Cons:**

- ❌ Slight approximation

---

#### Algorithm 4: Token Bucket (Most Popular)

**How it works:**

```
Bucket holds tokens (e.g., 100 tokens)
Tokens refill at fixed rate (e.g., 10 tokens/sec)
Each request consumes 1 token

Redis implementation:
Key: rate_limit:{user_id}
Value: {tokens: 95, last_refill: timestamp}

On new request:
1. Calculate tokens to add: (now - last_refill) × refill_rate
2. Add tokens (up to max: 100)
3. If tokens >= 1:
   - Consume 1 token
   - Allow request
4. Else:
   - Reject request
```

**Pros:**

- ✅ Handles bursts (accumulate tokens)
- ✅ Smooth traffic
- ✅ Memory efficient

**Cons:**

- ❌ Slightly complex

**Decision: Use Token Bucket for flexibility**

---

### Step 3: High-Level Design (10 min)

```
┌─────────┐
│ Client  │
└────┬────┘
     │
     ↓
┌──────────────┐
│ Load Balancer│
└──────┬───────┘
       │
       ↓
┌──────────────────┐
│  API Gateway     │
│ (Rate Limiter)   │
└────┬─────────────┘
     │
     ├──→ ┌─────────┐
     │    │  Redis  │ (Rate limit state)
     │    └─────────┘
     │
     ↓
┌──────────────────┐
│ Backend Service  │
└──────────────────┘
```

**Request Flow:**

```
1. Client sends request
2. API Gateway intercepts
3. Extract identifier (user_id or IP address)
4. Check Redis for rate limit state
5. Apply token bucket algorithm
6. If allowed:
   - Update Redis (consume token)
   - Forward to backend
7. If rejected:
   - Return HTTP 429 (Too Many Requests)
   - Headers: X-RateLimit-Remaining, X-RateLimit-Reset
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive 1: Distributed Rate Limiting

**Problem:** Multiple API Gateway instances, how to share state?

**Approach 1: Centralized Redis**

```
All gateways check same Redis cluster

Pros:
✅ Accurate (single source of truth)

Cons:
❌ Redis becomes bottleneck
❌ Network latency to Redis
```

**Approach 2: Local Cache + Redis (Better)**

```
1. Gateway maintains local cache (in-memory)
2. Sync with Redis every 100ms
3. Check local cache first (fast)
4. Periodically sync with Redis (consistency)

Trade-off:
✅ Low latency (local cache)
❌ Slight inaccuracy (eventual consistency)
```

**Example:**

```
Local cache: user_123 has 50 tokens
Redis: user_123 has 48 tokens (another gateway consumed 2)

After 100ms sync:
Local cache updated to 48 tokens

Worst case: User sends 100 requests in 100ms across 2 gateways
→ Might allow 120 requests instead of 100
→ Acceptable for most use cases
```

---

#### Deep Dive 2: Rate Limit Rules Engine

**Problem:** Different limits for different users/endpoints

**Configuration:**

```yaml
rate_limits:
  - endpoint: /api/login
    limit: 5
    window: 60s
    identifier: ip_address

  - endpoint: /api/search
    limit: 100
    window: 60s
    identifier: user_id
    tier: free

  - endpoint: /api/search
    limit: 1000
    window: 60s
    identifier: user_id
    tier: premium

  - endpoint: /api/upload
    limit: 10
    window: 86400s # 1 day
    identifier: user_id
```

**Implementation:**

```python
def check_rate_limit(user, endpoint):
    rule = get_rule(endpoint, user.tier)
    key = f"rate_limit:{rule.identifier}:{endpoint}"

    current = redis.get(key)
    if current >= rule.limit:
        return False, f"Rate limit exceeded. Try again in {ttl} seconds"

    redis.incr(key)
    redis.expire(key, rule.window)
    return True, None
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- Token bucket algorithm
- Fixed window vs sliding window
- Distributed rate limiting with Redis
- Configuration-based rules engine

**Related problems:**

- API throttling
- DDoS protection
- Login attempt limiting
- Preventing spam

---

## Pattern 5: Search and Autocomplete (Google Search)

### Example: Design Google Search / Autocomplete

### What Makes This Pattern Unique

- **Core challenge:** Full-text search across billions of documents
- **Key concepts:** Inverted index, autocomplete, ranking
- **Variants:** Google Search, Elasticsearch, Product search

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ Search query returns ranked results
- ✅ Autocomplete suggestions as user types
- ✅ Handle typos (fuzzy matching)

**Non-Functional:**

- 10B documents indexed
- 100K search queries/sec
- Search results < 100ms
- Autocomplete results < 50ms

---

### Step 2: Core Components

#### A. Inverted Index

**Problem:** Find all documents containing "apple"

**Naive approach:**

```
Scan all 10B documents → Too slow (hours)
```

**Inverted Index approach:**

```
Index: term → list of document IDs

apple → [doc1, doc5, doc123, doc456, ...]
banana → [doc2, doc5, doc789, ...]
orange → [doc3, doc10, doc123, ...]

Search "apple":
→ Lookup index["apple"]
→ Get [doc1, doc5, doc123, doc456]
→ Fetch documents
→ Total time: milliseconds
```

---

#### B. Autocomplete (Trie Data Structure)

**Problem:** User types "app", suggest "apple", "application", "app store"

**Trie structure:**

```
        (root)
         /
        a
       /
      p
     /
    p
   / \
  l   s (app store)
 /
e (apple)
```

**Search:**

```
User types "app":
1. Traverse trie: a → p → p
2. Get all children (sub-branches)
3. Return top suggestions by frequency

Implementation:
- Each node stores: character + children + is_end + frequency
- Top suggestions precomputed at each node (for speed)
```

---

### Step 3: High-Level Design (10 min)

```
┌─────────┐
│ Client  │
└────┬────┘
     │
     ↓
┌──────────────┐
│Load Balancer │
└──────┬───────┘
       │
       ├──→ ┌───────────────┐
       │    │ Query Service │
       │    └───────┬───────┘
       │            │
       │            ↓
       │    ┌───────────────┐
       │    │ Elasticsearch │ (Inverted index)
       │    └───────────────┘
       │
       ├──→ ┌───────────────┐
       │    │  Autocomplete │
       │    │    Service    │
       │    └───────┬───────┘
       │            │
       │            ↓
       │    ┌───────────────┐
       │    │  Trie (Redis) │
       │    └───────────────┘
       │
       └──→ ┌───────────────┐
            │ Indexing      │
            │ Service       │
            └───────────────┘
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive 1: Ranking Algorithm

**Problem:** 1,000 documents contain "apple". Which to show first?

**Factors:**

```
1. Relevance (TF-IDF)
   - Term Frequency: How often "apple" appears in document
   - Inverse Document Frequency: How rare "apple" is across all docs

2. PageRank (Google's secret sauce)
   - Importance based on incoming links

3. User signals
   - Click-through rate (CTR)
   - Dwell time (how long user stayed on page)

4. Freshness
   - Newer content scored higher (for news)

5. Personalization
   - User's location, history, preferences
```

**Simplified Scoring:**

```python
def calculate_score(doc, query, user):
    score = 0

    # Term frequency
    score += count_occurrences(query, doc.content) * 10

    # Document popularity
    score += doc.click_count * 0.01

    # Recency (for news)
    age_days = (now - doc.created_at).days
    score += max(0, 100 - age_days)

    # Personalization
    if doc.category in user.interests:
        score += 50

    return score
```

---

#### Deep Dive 2: Handling Typos (Fuzzy Matching)

**Problem:** User searches "aple" (misspelled "apple")

**Approach 1: Edit Distance (Levenshtein)**

```
Compute distance between query and indexed terms
"aple" vs "apple" → distance = 1 (one insertion)
"aple" vs "orange" → distance = 6

Accept results with distance <= 2
```

**Approach 2: N-grams (Better for scale)**

```
Index terms as n-grams (character sequences)

"apple" → bigrams: ["ap", "pp", "pl", "le"]

"aple" → bigrams: ["ap", "pl", "le"]

Match 3 out of 4 bigrams → Likely match!
```

**Elasticsearch approach:**

```
POST /search
{
  "query": {
    "match": {
      "content": {
        "query": "aple",
        "fuzziness": "AUTO"  # Allows 1-2 char difference
      }
    }
  }
}
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- Inverted index for fast search
- Trie for autocomplete
- Ranking algorithms (TF-IDF, PageRank)
- Fuzzy matching (edit distance, n-grams)

**Related problems:**

- Product search (Amazon)
- Document search (Google Drive)
- Log search (Splunk)
- Autocomplete (any search bar)

---

## Pattern 6: Analytics or Click Tracking (Google Analytics)

### Example: Design Google Analytics

### What Makes This Pattern Unique

- **Core challenge:** Ingest massive event streams, aggregate in real-time
- **Key concepts:** Stream processing, time-series data, aggregation
- **Variants:** Google Analytics, Mixpanel, Amplitude

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ Track page views, clicks, events
- ✅ Real-time dashboard (traffic right now)
- ✅ Historical analytics (last 30 days)
- ✅ Aggregations (by country, device, page)

**Non-Functional:**

- 1M websites tracked
- 10B events/day
- Real-time dashboard update (< 1 minute delay)
- Query response < 2 seconds

**Calculations:**

```
Events/sec: 10B / 86,400 = 115K events/sec (peak: 350K/sec)
Storage: 10B × 1KB = 10TB/day = 3.6PB/year
```

---

### Step 2: High-Level Design (10 min)

```
┌─────────┐
│ Website │
│ (JS SDK)│
└────┬────┘
     │ POST /track
     ↓
┌──────────────┐
│ API Gateway  │
│ (Load Bal.)  │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│    Kafka     │ (Message queue)
└──────┬───────┘
       │
       ├──→ ┌─────────────────┐
       │    │ Stream Processor│ (Flink / Spark)
       │    │ (Real-time agg) │
       │    └────────┬────────┘
       │             │
       │             ↓
       │    ┌─────────────────┐
       │    │   Redis         │ (Real-time metrics)
       │    └─────────────────┘
       │
       └──→ ┌─────────────────┐
            │ Batch Processor │ (Hourly/Daily agg)
            └────────┬────────┘
                     │
                     ↓
            ┌─────────────────┐
            │ Data Warehouse  │ (BigQuery / Redshift)
            │ (Historical)    │
            └─────────────────┘
```

**Event Flow:**

```
1. User clicks button on website
2. JS SDK sends event to API Gateway:
   {
     "event_type": "button_click",
     "user_id": "123",
     "page": "/products",
     "timestamp": 1704276000,
     "metadata": {"button": "buy_now"}
   }
3. API Gateway writes to Kafka (fast ack)
4. Stream processor consumes from Kafka:
   - Real-time aggregation (last 5 minutes)
   - Write to Redis: clicks_per_minute:{timestamp} = count
5. Batch processor (hourly):
   - Aggregate by hour, day, week
   - Write to data warehouse
6. Dashboard queries:
   - Real-time: Redis (last 1 hour)
   - Historical: Data warehouse (last 30 days)
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive 1: Real-Time Aggregation (Stream Processing)

**Problem:** Count page views in last 5 minutes

**Approach: Tumbling Window**

```
Window: 5 minutes
11:00 - 11:05: 1,000 events
11:05 - 11:10: 1,500 events
11:10 - 11:15: 2,000 events

At 11:05: Emit count for window 11:00-11:05 → 1,000
At 11:10: Emit count for window 11:05-11:10 → 1,500
```

**Flink Code (Conceptual):**

```java
stream
  .keyBy(event -> event.page)
  .window(TumblingProcessingTimeWindows.of(Time.minutes(5)))
  .aggregate(new CountAggregator())
  .addSink(new RedisSink());  // Write to Redis
```

**Redis Schema:**

```
Key: pageviews:{page}:{timestamp}
Value: count

Example:
pageviews:/products:1704276000 = 1500
pageviews:/products:1704276300 = 2000
```

---

#### Deep Dive 2: Multi-Dimensional Aggregation

**Problem:** Query "Page views by country, device, and page"

**Naive Approach:**

```
Store each event individually
On query: Scan all events and aggregate

10B events → Too slow (minutes)
```

**Pre-Aggregation Approach:**

```
Pre-aggregate common dimensions

Dimensions:
- Page
- Country
- Device
- Hour

Aggregation tables:
1. pageviews_by_page_hour
2. pageviews_by_country_hour
3. pageviews_by_device_hour
4. pageviews_by_page_country_hour  (combination)

Trade-off:
✅ Fast queries (pre-computed)
❌ More storage (multiple aggregation tables)
❌ Limited query flexibility
```

**OLAP Cube (Best for Analytics):**

```
Use columnar database (BigQuery, Redshift, ClickHouse)
Store raw events in columnar format
Query engine aggregates on-the-fly (optimized)

Query:
SELECT page, country, COUNT(*)
FROM events
WHERE timestamp >= '2026-01-01'
GROUP BY page, country

Execution time: < 2 seconds (even on 10B rows)
```

**Decision:**

```
- Real-time (< 1 hour): Pre-aggregated in Redis
- Historical (> 1 hour): OLAP database (BigQuery)
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- Stream processing (Kafka + Flink)
- Tumbling/sliding windows
- Pre-aggregation vs on-the-fly aggregation
- OLAP databases for analytics
- Lambda architecture (batch + stream)

**Related problems:**

- Application monitoring (Datadog)
- Business intelligence (Tableau)
- Ad click tracking
- IoT sensor analytics

---

## Pattern 7: Ticket Booking or Reservation (Ticketmaster)

### Example: Design Ticketmaster / Airbnb Booking

### What Makes This Pattern Unique

- **Core challenge:** Prevent double-booking with high concurrency
- **Key concepts:** Distributed locking, optimistic locking, inventory management
- **Variants:** Ticketmaster, Airbnb, restaurant reservations, hotel booking

---

### Step 1: Requirements (5 min)

**Functional:**

- ✅ Users can view available seats
- ✅ Users can book seats (atomic)
- ✅ Booking held for 10 minutes (payment)
- ✅ No double-booking (consistency critical)

**Non-Functional:**

- 10K concurrent users booking for same event
- Booking must be atomic (strong consistency)
- Booking latency < 500ms

---

### Step 2: High-Level Design (10 min)

```
┌─────────┐
│  User   │
└────┬────┘
     │
     ↓
┌──────────────┐
│ Load Balancer│
└──────┬───────┘
       │
       ↓
┌──────────────────┐
│ Booking Service  │
└────┬─────────────┘
     │
     ├──→ ┌─────────┐
     │    │  Redis  │ (Distributed lock)
     │    └─────────┘
     │
     └──→ ┌──────────────┐
          │ PostgreSQL   │ (Inventory)
          └──────────────┘
```

---

### Step 4: Deep Dive (20 min)

#### Deep Dive: Prevent Double-Booking

**Problem:** 2 users book the last seat simultaneously

**Approach 1: Database Lock (Pessimistic Locking)**

```sql
BEGIN TRANSACTION;

-- Lock the seat row
SELECT * FROM seats
WHERE seat_id = 123 AND event_id = 456 AND status = 'available'
FOR UPDATE;  -- Pessimistic lock

-- If seat found, book it
UPDATE seats
SET status = 'reserved', user_id = 789
WHERE seat_id = 123;

COMMIT;
```

**Pros:**

- ✅ Guaranteed no double-booking
- ✅ Simple to implement

**Cons:**

- ❌ Blocks other transactions (low throughput)
- ❌ Deadlock risk

---

**Approach 2: Optimistic Locking (Version Number)**

```sql
-- Read seat with version
SELECT * FROM seats WHERE seat_id = 123;
-- Returns: {seat_id: 123, status: 'available', version: 5}

-- Attempt to book
UPDATE seats
SET status = 'reserved', user_id = 789, version = version + 1
WHERE seat_id = 123 AND version = 5;  -- Only succeeds if version unchanged

-- Check affected rows
IF rows_affected == 0:
  -- Someone else booked it (version changed)
  RETURN "Seat already booked"
ELSE:
  RETURN "Booking successful"
```

**Pros:**

- ✅ No blocking (high throughput)
- ✅ No deadlocks

**Cons:**

- ❌ Retry logic needed (if version conflict)

---

**Approach 3: Distributed Lock (Redis)**

```python
def book_seat(seat_id, user_id):
    lock_key = f"lock:seat:{seat_id}"

    # Acquire lock (try for 5 seconds)
    lock_acquired = redis.set(lock_key, user_id, nx=True, ex=10)

    if not lock_acquired:
        return "Seat is being booked by someone else"

    try:
        # Check availability
        seat = db.query("SELECT * FROM seats WHERE seat_id = ?", seat_id)

        if seat.status != 'available':
            return "Seat not available"

        # Book it
        db.execute("UPDATE seats SET status = 'reserved', user_id = ? WHERE seat_id = ?", user_id, seat_id)

        return "Booking successful"

    finally:
        # Release lock
        redis.delete(lock_key)
```

**Pros:**

- ✅ Works across distributed services
- ✅ Simple to reason about

**Cons:**

- ❌ Single point of failure (Redis)
- ❌ Lock can be lost (Redis crash)

---

**Decision: Optimistic Locking for Scalability**

**Why:**

```
- High throughput (10K concurrent bookings)
- No blocking
- Retry acceptable for booking flow
- Database ensures consistency (ACID)
```

---

### Key Takeaways for This Pattern

**Must-know concepts:**

- Pessimistic vs optimistic locking
- Version numbers for concurrency control
- Distributed locks (Redis)
- Inventory management
- Idempotency for retries

**Related problems:**

- Flight booking
- Hotel reservation
- Restaurant table booking
- Limited inventory sales (flash sales)

---

_(Continued in next message due to length...)_

---

## How to Practice Effectively

### Three-Pass Method (Per Problem)

#### Pass 1: Learn the Pattern (2 hours)

```
1. Read a good solution (blog, video, book)
2. Understand the WHY, not just the WHAT
3. Take notes on:
   - Key bottlenecks
   - Deep dive topics
   - Trade-offs discussed
   - Numbers/calculations
```

---

#### Pass 2: Do It Yourself (1 hour)

```
1. Set timer for 45 minutes
2. Whiteboard/paper (no looking at solution)
3. Follow the 4-step framework
4. After 45 min, compare with original solution
5. Identify gaps:
   - What did I miss?
   - Where did I get stuck?
   - What trade-offs did I not consider?
```

---

#### Pass 3: Teach It (30 minutes)

```
1. Explain design out loud to a friend (or camera)
2. Pretend you're the interviewer and candidate
3. If you can't explain clearly, you don't understand it
4. Focus on:
   - Clear communication
   - Justifying decisions
   - Trade-off articulation
```

---

### Practice Schedule (4 Weeks)

**Week 1: Patterns 1-3**

- Day 1-2: Content Feed (Pass 1+2)
- Day 3: Content Feed (Pass 3)
- Day 4-5: File Storage (Pass 1+2)
- Day 6: File Storage (Pass 3)
- Day 7: Messaging (Pass 1+2+3)

**Week 2: Patterns 4-6**

- Day 1-2: Rate Limiter (Pass 1+2+3)
- Day 3-4: Search (Pass 1+2+3)
- Day 5-6: Analytics (Pass 1+2+3)
- Day 7: Review Week 1+2

**Week 3: Patterns 7-10**

- Day 1-2: Booking System (Pass 1+2+3)
- Day 3-4: Notification Service (Pass 1+2+3)
- Day 5: URL Shortener (Pass 1+2+3)
- Day 6: Video Streaming (Pass 1+2+3)
- Day 7: Review all 10 patterns

**Week 4: Mock Interviews**

- Day 1-6: 2 mock interviews per day (random patterns)
- Day 7: Final review, identify weak patterns

---

### After 4 Weeks

**You will have:**

- ✅ Done 30 passes (10 patterns × 3 passes)
- ✅ 12+ mock interviews
- ✅ Internalized common patterns
- ✅ Confidence in the interview framework

**You are ready!** 🎯

---

## Quick Reference: Pattern Cheat Sheet

| Pattern             | Key Challenge        | Deep Dive Topics                        |
| ------------------- | -------------------- | --------------------------------------- |
| **Content Feed**    | Timeline generation  | Fanout (write/read/hybrid), Ranking     |
| **File Storage**    | Large file handling  | Chunking, Deduplication, Sync           |
| **Messaging**       | Real-time delivery   | WebSockets, Delivery guarantees, Fanout |
| **Rate Limiter**    | Prevent abuse        | Token bucket, Distributed limiting      |
| **Search**          | Fast retrieval       | Inverted index, Autocomplete, Ranking   |
| **Analytics**       | Event ingestion      | Stream processing, Aggregation          |
| **Booking**         | No double-booking    | Locking strategies, Consistency         |
| **Notifications**   | Reliable delivery    | Queues, Templates, Multi-channel        |
| **URL Shortener**   | Unique short URLs    | Base62 encoding, Key generation         |
| **Video Streaming** | Large media delivery | CDN, Encoding, Adaptive bitrate         |

---

**Master these 10 patterns, and you can handle 90% of system design interviews with confidence!** 🚀
