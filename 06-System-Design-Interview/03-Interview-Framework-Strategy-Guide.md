# Interview Framework & Strategy - Complete Guide

> **Critical**: Use this strict framework in EVERY interview. This structure keeps you organized under time pressure and ensures you hit all evaluation criteria.

---

## Table of Contents

1. [The 4-Step Interview Framework](#the-4-step-interview-framework)
2. [Step 1: Clarify Requirements (5-7 minutes)](#step-1-clarify-requirements-5-7-minutes)
3. [Step 2: Identify Core Entities and APIs (5-8 minutes)](#step-2-identify-core-entities-and-apis-5-8-minutes)
4. [Step 3: Draw Simple First Design (10-12 minutes)](#step-3-draw-simple-first-design-10-12-minutes)
5. [Step 4: Deep Dive (15-20 minutes)](#step-4-deep-dive-15-20-minutes)
6. [Communication Best Practices](#communication-best-practices)
7. [Time Management](#time-management)
8. [Common Mistakes to Avoid](#common-mistakes-to-avoid)

---

## The 4-Step Interview Framework

**Total Time: 45-50 minutes**

```
┌─────────────────────────────────────────────────────────┐
│ Step 1: Clarify Requirements (5-7 min)                  │
│   - Functional: What the system must do                 │
│   - Non-functional: Scale, latency, consistency, etc.   │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│ Step 2: Core Entities & APIs (5-8 min)                  │
│   - List 3-5 main entities                              │
│   - Define 5-8 key endpoints                            │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│ Step 3: Simple First Design (10-12 min)                 │
│   - One web tier, one service, one database, maybe cache│
│   - Walk through request flow                           │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│ Step 4: Deep Dive on 2 Critical Pieces (15-20 min)      │
│   - Pick 2 components that matter most                  │
│   - Go deep: data model, algorithms, trade-offs         │
└─────────────────────────────────────────────────────────┘
```

---

## Step 1: Clarify Requirements (5-7 minutes)

### Why This Step Matters

- Shows you understand that ambiguous problems need scoping
- Prevents building the wrong system
- Demonstrates product thinking
- Sets constraints for design decisions

### A. Functional Requirements

**What you ask:**

```
"Let me clarify what the system needs to do. Can you help me understand:"

1. Core features (must-have)
2. Nice-to-have features
3. Out of scope features
```

#### Example: "Design Twitter"

**Questions to Ask:**

```
✅ Good Questions:
- "Can users post tweets? What's the character limit?"
- "Do we need a timeline/feed feature?"
- "Should users be able to follow other users?"
- "Do we need direct messaging, or just public tweets?"
- "Are we supporting media (images/videos) in tweets?"
- "Do we need search functionality?"
- "Should we support hashtags and trending topics?"

❌ Bad Questions:
- "What technology should I use?" (You decide)
- "Do you want me to design everything?" (Too vague)
```

**Clarified Scope:**

```
In Scope:
✅ Users can post tweets (280 chars)
✅ Users can follow others
✅ Users see home timeline (tweets from people they follow)
✅ Users can like and retweet

Out of Scope (for this interview):
❌ Direct messaging
❌ Search
❌ Trending topics
❌ Notifications
```

---

### B. Non-Functional Requirements

**Always ask about SCALE:**

```
"Let me understand the scale and performance requirements:"

1. How many users?
   - Total registered users
   - Daily Active Users (DAU)
   - Monthly Active Users (MAU)

2. How many requests/transactions?
   - Reads per second
   - Writes per second
   - Peak vs average

3. What data volume?
   - Data generated per day
   - Storage growth rate

4. What latency requirements?
   - p50, p95, p99 latency targets

5. What consistency requirements?
   - Strong vs eventual consistency
   - Acceptable staleness

6. What availability requirements?
   - 99.9% vs 99.99% vs 99.999%
```

---

#### Example: "Design Twitter" - Scale Clarification

**Your Questions & Answers:**

```
You: "How many users are we targeting?"
Interviewer: "Let's say 200M DAU"

You: "What's the ratio of reads to writes?"
Interviewer: "Users read way more than they post. Let's say 100:1"

You: "What's the average number of people a user follows?"
Interviewer: "Let's say 200"

You: "What latency are we targeting for the timeline?"
Interviewer: "Under 200ms for p99"

You: "For timeline consistency, is it okay if a tweet appears with a few seconds delay?"
Interviewer: "Yes, eventual consistency is fine"
```

**Now Calculate:**

```
Given:
- 200M DAU
- Avg user posts 2 tweets/day
- Read:Write ratio = 100:1

Writes:
- 200M users × 2 tweets/day = 400M tweets/day
- 400M tweets/day ÷ 86,400 sec ≈ 4,630 tweets/sec
- Peak (3x average) ≈ 14K tweets/sec

Reads:
- 4,630 × 100 = 463K reads/sec
- Peak ≈ 1.4M reads/sec

Storage:
- 400M tweets/day × 280 chars × 2 bytes ≈ 224GB/day
- Plus metadata (user_id, timestamp, etc.) ≈ 300GB/day
- Per year: ~110TB
```

**Write it down on the whiteboard!** Shows you can translate vague requirements into concrete numbers.

---

### Template for Requirements Gathering

**Fill this template in first 5-7 minutes:**

```
┌──────────────────────────────────────────────────┐
│ FUNCTIONAL REQUIREMENTS                           │
├──────────────────────────────────────────────────┤
│ ✅ In Scope:                                     │
│   - [Feature 1]                                  │
│   - [Feature 2]                                  │
│   - [Feature 3]                                  │
│                                                   │
│ ❌ Out of Scope:                                 │
│   - [Feature A]                                  │
│   - [Feature B]                                  │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│ NON-FUNCTIONAL REQUIREMENTS                       │
├──────────────────────────────────────────────────┤
│ Scale:                                           │
│   - XXX DAU                                      │
│   - XXX reads/sec, XXX writes/sec                │
│   - XXX TB storage per year                      │
│                                                   │
│ Latency:                                         │
│   - < XXX ms (p99)                               │
│                                                   │
│ Consistency:                                     │
│   - [Strong / Eventual]                          │
│                                                   │
│ Availability:                                    │
│   - XX.XX% uptime                                │
└──────────────────────────────────────────────────┘
```

---

## Step 2: Identify Core Entities and APIs (5-8 minutes)

### Why This Step Matters

- Defines data model foundation
- Shows you think about API contract first
- Prevents over-complicating the design
- Gives structure to discussion

---

### A. Identify 3-5 Core Entities

**Entity = Main nouns in your system**

#### Example: Twitter

```
Core Entities:
1. User
   - user_id (PK)
   - username
   - email
   - created_at
   - follower_count
   - following_count

2. Tweet
   - tweet_id (PK)
   - user_id (FK)
   - content (280 chars)
   - created_at
   - like_count
   - retweet_count

3. Follow
   - follower_id (FK to User)
   - followee_id (FK to User)
   - created_at
   - (Composite PK: follower_id + followee_id)

4. Like
   - user_id (FK)
   - tweet_id (FK)
   - created_at
   - (Composite PK: user_id + tweet_id)

5. Timeline (Materialized/Cached)
   - user_id
   - tweet_ids (ordered list)
   - last_updated
```

**Keep it simple!** Don't model everything. Mention, "We can extend this with other entities like Retweets, Mentions, etc."

---

### B. Define 5-8 Key APIs

**API = How clients interact with system**

**Format:**

```
HTTP Method + Endpoint + Brief Description
```

#### Example: Twitter APIs

```
Core APIs:

1. POST /api/v1/tweets
   - Create a new tweet
   - Request: {user_id, content, media_urls}
   - Response: {tweet_id, created_at, ...}

2. GET /api/v1/timeline/{user_id}
   - Get home timeline for user
   - Query params: ?page=1&limit=20
   - Response: {tweets: [...]}

3. POST /api/v1/follow
   - Follow a user
   - Request: {follower_id, followee_id}
   - Response: {success: true}

4. DELETE /api/v1/follow
   - Unfollow a user
   - Request: {follower_id, followee_id}
   - Response: {success: true}

5. POST /api/v1/tweets/{tweet_id}/like
   - Like a tweet
   - Request: {user_id}
   - Response: {success: true, new_like_count}

6. GET /api/v1/users/{user_id}
   - Get user profile
   - Response: {user_id, username, follower_count, ...}

7. GET /api/v1/tweets/{tweet_id}
   - Get a specific tweet
   - Response: {tweet details}

8. DELETE /api/v1/tweets/{tweet_id}
   - Delete a tweet
   - Request: {user_id} (for auth)
   - Response: {success: true}
```

**Pro Tip:** Mention REST conventions. Shows you know best practices.

---

### Template for Entities & APIs

```
┌──────────────────────────────────────────────────┐
│ CORE ENTITIES (3-5)                              │
├──────────────────────────────────────────────────┤
│ 1. [Entity Name]                                 │
│    - field_1                                     │
│    - field_2                                     │
│    - field_3                                     │
│                                                   │
│ 2. [Entity Name]                                 │
│    - field_1                                     │
│    - field_2                                     │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│ KEY APIs (5-8)                                   │
├──────────────────────────────────────────────────┤
│ 1. METHOD /endpoint - What it does              │
│ 2. METHOD /endpoint - What it does              │
│ 3. METHOD /endpoint - What it does              │
│ ...                                              │
└──────────────────────────────────────────────────┘
```

---

## Step 3: Draw Simple First Design (10-12 minutes)

### Why This Step Matters

- Shows you start simple and iterate
- Demonstrates understanding of basic architecture
- Provides foundation for deep dive
- Tests if you can articulate request flow

---

### A. Standard Components in First Design

**Basic Building Blocks:**

```
┌─────────────┐
│   Client    │
│ (Web/Mobile)│
└──────┬──────┘
       │
       ↓
┌──────────────┐
│     CDN      │ (Static assets only)
└──────┬───────┘
       │
       ↓
┌──────────────┐
│Load Balancer │
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ App Servers  │ (Stateless, horizontally scalable)
│ (Service)    │
└───┬─────┬────┘
    │     │
    ↓     ↓
┌───────┐ ┌────────┐
│ Cache │ │Database│
│(Redis)│ │ (SQL)  │
└───────┘ └────────┘
```

---

### B. Walk Through Request Flow

**Pick 2-3 critical flows and walk through them.**

#### Example: Twitter - "Post a Tweet"

```
Flow:
1. User creates tweet in mobile app
   ↓
2. POST /api/v1/tweets to Load Balancer
   ↓
3. Load Balancer routes to available App Server
   ↓
4. App Server validates tweet (auth, length, content)
   ↓
5. Generate tweet_id (UUID or Snowflake ID)
   ↓
6. Insert into Database:
   INSERT INTO tweets (tweet_id, user_id, content, created_at)
   VALUES (...)
   ↓
7. Invalidate cache (if caching user's tweets)
   ↓
8. [Optional] Push to message queue for fanout
   ↓
9. Return tweet_id to client

Response time: ~100ms
```

---

#### Example: Twitter - "Get Timeline"

```
Flow:
1. User opens app, requests timeline
   ↓
2. GET /api/v1/timeline/{user_id} to Load Balancer
   ↓
3. Load Balancer routes to App Server
   ↓
4. App Server checks Cache (Redis)
   Key: timeline:{user_id}
   ↓
5a. Cache HIT:
    - Return cached timeline (10ms)

5b. Cache MISS:
    - Query Database for tweets from followed users
    - Query:
      SELECT t.* FROM tweets t
      JOIN follows f ON t.user_id = f.followee_id
      WHERE f.follower_id = {user_id}
      ORDER BY t.created_at DESC
      LIMIT 20
    - Cache result with TTL (e.g., 60 seconds)
    - Return to client (150ms)

Cached response: ~10ms
Uncached response: ~150ms
```

---

### C. Database Schema (High Level)

**Draw basic tables:**

```sql
-- Users Table
users
├─ user_id (PK, BIGINT)
├─ username (VARCHAR, UNIQUE)
├─ email (VARCHAR, UNIQUE)
├─ password_hash (VARCHAR)
├─ created_at (TIMESTAMP)
└─ [Index on username, email]

-- Tweets Table
tweets
├─ tweet_id (PK, BIGINT)
├─ user_id (FK → users.user_id)
├─ content (VARCHAR(280))
├─ created_at (TIMESTAMP)
├─ like_count (INT, default 0)
└─ [Index on user_id, created_at]

-- Follows Table
follows
├─ follower_id (FK → users.user_id)
├─ followee_id (FK → users.user_id)
├─ created_at (TIMESTAMP)
└─ [Composite PK: (follower_id, followee_id)]
   [Index on followee_id for reverse lookup]

-- Likes Table
likes
├─ user_id (FK → users.user_id)
├─ tweet_id (FK → tweets.tweet_id)
├─ created_at (TIMESTAMP)
└─ [Composite PK: (user_id, tweet_id)]
   [Index on tweet_id]
```

**Mention indexes!** Shows you think about query performance.

---

### D. Identify Bottlenecks (Before Scaling)

**Ask yourself:**

```
1. What happens if traffic increases 10x?
2. What's the slowest operation?
3. What's the single point of failure?
```

#### Example: Twitter Bottlenecks

```
Bottleneck 1: Timeline Query
- Joining tweets and follows for large follower count is slow
- Solution: Pre-compute timelines (fanout on write)

Bottleneck 2: Database is single point of failure
- Solution: Add read replicas for timeline queries

Bottleneck 3: Celebrity tweets fanout
- A user with 10M followers causes 10M writes
- Solution: Hybrid approach (fanout for normal users, pull for celebrities)

Bottleneck 4: Write-heavy during peak times
- Solution: Message queue to smooth writes
```

**This sets up your deep dive!**

---

## Step 4: Deep Dive (15-20 minutes)

### Why This Step Matters

- **This is where you differentiate yourself**
- Shows depth of knowledge, not just breadth
- Tests problem-solving under constraints
- Demonstrates trade-off thinking

---

### A. How to Choose What to Deep Dive

**Pick 2 areas that:**

1. Are critical to the system working
2. Have interesting scaling/complexity challenges
3. Show your expertise

**For each deep dive:**

- Explain the problem clearly
- Present 2-3 solution approaches
- Compare trade-offs
- Make a decision and justify it

---

### B. Deep Dive Examples by System

#### Twitter: Timeline Generation (Fanout)

**Problem:**

```
When user_A posts a tweet, whose timelines need updating?
- All of user_A's followers
- If user_A has 10M followers, that's 10M timeline updates per tweet
```

**Approach 1: Fanout on Write (Push Model)**

**How it works:**

```
When user posts tweet:
1. Identify all followers (query follows table)
2. For each follower:
   - Add tweet_id to their timeline cache
   - Key: timeline:{follower_id}
   - Value: [tweet_id_1, tweet_id_2, ..., tweet_id_N]
3. Timeline read is instant (just read from cache)
```

**Pros:**

- ✅ Fast reads (timeline already precomputed)
- ✅ Good for users who read frequently
- ✅ Predictable read latency

**Cons:**

- ❌ Slow writes for users with many followers
- ❌ Wastes resources if follower never reads timeline
- ❌ Celebrity problem (10M fanout writes)

**When to use:**

- Normal users (< 10K followers)
- Read-heavy systems
- Read latency is critical

---

**Approach 2: Fanout on Read (Pull Model)**

**How it works:**

```
When user requests timeline:
1. Fetch list of people they follow
2. Query tweets from each followee
3. Merge and sort by timestamp
4. Return top N
```

**Pros:**

- ✅ Fast writes (no fanout)
- ✅ No wasted work
- ✅ Works for celebrities

**Cons:**

- ❌ Slow reads (complex query)
- ❌ Database load spikes on timeline request
- ❌ Difficult to scale for users following many people

**When to use:**

- Users with massive follower counts (celebrities)
- Write latency is critical
- Read frequency is low

---

**Approach 3: Hybrid (Best)**

**How it works:**

```
Normal users (< 10K followers):
  - Fanout on write (push)

Celebrities (> 10K followers):
  - Fanout on read (pull)

Timeline generation:
  - Merge pre-computed timeline + celebrity tweets on-the-fly
```

**Example:**

```
User follows: 200 normal users + 5 celebrities

Timeline request:
1. Fetch precomputed timeline from cache (fanout-on-write for 200 users)
2. Query latest tweets from 5 celebrities (fanout-on-read)
3. Merge the two lists, sort by timestamp
4. Cache result for 60 seconds
5. Return to client

Result:
- Reads: ~20ms (mostly cached)
- Writes: Efficient (no celebrity fanout)
- Best of both worlds
```

**Trade-offs:**

- ✅ Balanced read/write performance
- ✅ Handles scale gracefully
- ❌ More complex implementation
- ❌ Need logic to determine celebrity threshold

**Decision:**

```
"I'd use a hybrid approach with a threshold of 10K followers.
This gives us fast reads for 99% of users while avoiding
expensive fanouts for celebrities. We can adjust the threshold
based on monitoring."
```

---

#### Twitter: Sharding Strategy

**Problem:**

```
Single database can't handle:
- 14K tweets/sec writes
- 1.4M timeline reads/sec
- 110TB/year storage growth
```

**Approach 1: Shard by Tweet ID**

**How it works:**

```
hash(tweet_id) % num_shards = shard_id

Tweet 12345 → Shard 2
Tweet 67890 → Shard 4
```

**Pros:**

- ✅ Even distribution
- ✅ Simple to implement

**Cons:**

- ❌ Timeline query needs to query ALL shards (scatter-gather)
- ❌ Slow for user-specific queries

**Verdict:** ❌ Not good for timeline-heavy system

---

**Approach 2: Shard by User ID**

**How it works:**

```
hash(user_id) % num_shards = shard_id

All of user_123's tweets → Shard 2
All of user_456's tweets → Shard 5
```

**Pros:**

- ✅ User's tweets are co-located
- ✅ Fast user-specific queries

**Cons:**

- ❌ Timeline query still needs to query multiple shards (for followed users)
- ❌ Hot users (celebrities) create hot shards

**Verdict:** ✅ Better, but timeline query still problematic

---

**Approach 3: Separate Tweet Storage and Timeline Storage**

**How it works:**

```
Tweet Storage (Shard by Tweet ID):
- Stores tweet content
- Used for: Creating tweet, viewing single tweet

Timeline Storage (Shard by User ID):
- Stores precomputed timelines (fanout result)
- Used for: Fetching timeline

When user posts tweet:
1. Store tweet in Tweet Storage (tweet shard)
2. Fanout: Update followers' timelines in Timeline Storage (user shards)

When user requests timeline:
1. Query Timeline Storage (single shard, user's shard)
2. Get tweet_ids
3. Fetch tweet details from Tweet Storage (multiple shards, cached)
```

**Pros:**

- ✅ Timeline read is single shard (fast)
- ✅ Tweets evenly distributed
- ✅ Separates write and read paths

**Cons:**

- ❌ More complex
- ❌ Two storage systems to maintain

**Decision:**

```
"I'd use separate storage for tweets and timelines:
- Tweet table sharded by tweet_id for even distribution
- Timeline cache sharded by user_id for fast reads
- This optimizes both read and write paths
- Adds complexity but critical for scale"
```

---

### C. Other Common Deep Dive Topics

#### Rate Limiting

```
Problem: Prevent abuse (spam, DDoS)
Solution: Token bucket algorithm with Redis
- Key: user_id or IP
- Increment on each request
- Reset after time window
- Distributed rate limiter with sliding window
```

---

#### Hot Key Handling (Cache)

```
Problem: Celebrity profile cached on one Redis shard (hot key)
Solution:
1. Replicate hot keys across multiple cache servers
2. Client-side randomization (10 copies of same key)
3. Local cache (app-level) for extremely hot data
```

---

#### Data Consistency (Booking Systems)

```
Problem: Two users book the last seat simultaneously
Solution:
1. Optimistic locking (version number)
2. Pessimistic locking (database lock)
3. Distributed lock (Redis)
Trade-offs discussed
```

---

#### Feed Ranking (Social Media)

```
Problem: Show most relevant posts, not just chronological
Solution:
1. Score each post (recency, engagement, user affinity)
2. Machine learning model for personalization
3. Precompute scores asynchronously
4. Re-rank cached timeline
```

---

## Communication Best Practices

### 1. Think Out Loud

```
❌ Bad: [Silent for 2 minutes] "I think we should use MongoDB"

✅ Good: "Let me think through storage options... We have structured
user data with relationships, so relational makes sense. But
posts have flexible schema. I'm weighing PostgreSQL for users
and MongoDB for posts... Actually, let's keep it simple and use
PostgreSQL for both since we need transactions for payments."
```

---

### 2. Make Trade-Offs Explicit

```
❌ Bad: "We'll use eventual consistency"

✅ Good: "For the timeline, I'm choosing eventual consistency over
strong consistency. Trade-off: users might see a tweet with
a few seconds delay, but we gain better availability and
lower latency. For this use case, that's acceptable."
```

---

### 3. Use Numbers

```
❌ Bad: "This will be slow"

✅ Good: "A scatter-gather query across 16 shards, each taking 50ms,
with network overhead of 10ms, gives us ~250ms latency.
That exceeds our 200ms p99 target, so we need to optimize."
```

---

### 4. Ask Clarifying Questions Mid-Design

```
"Before I decide on the caching strategy, can I clarify:
Would users expect to see their own tweet immediately in
their timeline, or is a few seconds delay acceptable?"

Shows you care about requirements, not just building stuff.
```

---

### 5. Acknowledge Interviewer's Hints

```
Interviewer: "What if a celebrity posts a tweet?"

❌ Bad: [Ignore and continue]

✅ Good: "Good point! Let me think about the celebrity use case...
If we naively fanout to 10M followers, that's 10M writes.
Let me explore a hybrid approach..."
```

---

## Time Management

**Strict Timing (45-min interview):**

| Phase                 | Time      | What to Accomplish                              |
| --------------------- | --------- | ----------------------------------------------- |
| **Requirements**      | 0-7 min   | Functional + non-functional, write down numbers |
| **Entities & APIs**   | 7-15 min  | 3-5 entities, 5-8 APIs                          |
| **High-Level Design** | 15-27 min | Draw boxes, explain 2-3 flows                   |
| **Deep Dive**         | 27-45 min | Go deep on 2 topics, discuss trade-offs         |
| **Wrap-Up**           | 45+ min   | Interviewer Q&A, clarifications                 |

**Red Flag:** Spending 20 minutes on requirements → Not enough time for deep dive

**Green Flag:** Requirements done in 5 minutes → More time for depth

---

### Time-Saving Tips

1. **Don't draw perfect boxes** - Rough sketches are fine
2. **Don't explain every single component** - Focus on critical path
3. **Don't code** - Unless explicitly asked (rare)
4. **Don't get stuck** - If unsure, state assumptions and move on
5. **Watch for interviewer cues** - If they're nodding, move faster. If confused, slow down.

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Jumping to Solution Without Requirements

```
Interviewer: "Design Instagram"
You: "We'll use Cassandra for storage..."

Problem: You don't know requirements yet!
```

**✅ Fix:**

```
You: "Let me first understand what we're building.
     Should users be able to upload photos, follow others,
     see a feed? What scale are we targeting?"
```

---

### ❌ Mistake 2: Obsessing Over Irrelevant Details

```
You: "For user authentication, we'll use OAuth 2.0 with PKCE flow,
     and JWT tokens with RS256 signing..."

Problem: Unless asked specifically, auth isn't the focus!
```

**✅ Fix:**

```
You: "For auth, we'll use standard OAuth with JWT tokens.
     [Don't elaborate unless asked]"
```

---

### ❌ Mistake 3: No Numbers

```
You: "We'll need to scale the database"

Problem: How do you know? What scale?
```

**✅ Fix:**

```
You: "With 200M DAU and 1.4M reads/sec at peak, a single
     PostgreSQL instance can handle ~10K QPS, so we'd need
     read replicas or sharding."
```

---

### ❌ Mistake 4: Treating Everything as Netflix Scale

```
You: "We'll shard the database across 1000 nodes and use Kafka..."

Problem: Maybe 1 server is enough for the requirements!
```

**✅ Fix:**

```
You: "With 10K DAU, we can start with a single PostgreSQL instance
     and a Redis cache. As we scale past 100K DAU, we'd add
     read replicas and consider sharding."
```

---

### ❌ Mistake 5: Glossing Over Bottlenecks

```
You: "And this is the design!" [Doesn't mention problems]

Problem: Interviewer wants to see you identify issues!
```

**✅ Fix:**

```
You: "This design works for moderate scale, but we have some
     bottlenecks: [1] Timeline query is expensive, [2] Database
     is single point of failure. Let me address these..."
```

---

### ❌ Mistake 6: Ignoring Trade-Offs

```
You: "We'll use microservices"

Problem: Why? What's the downside?
```

**✅ Fix:**

```
You: "I'm starting with a monolith because:
     - Simpler deployment and debugging
     - Faster initial development
     - Can extract services later based on bottlenecks
     Trade-off: Harder to scale teams, but for our current
     scale, the simplicity benefit outweighs that."
```

---

## Final Interview Checklist

**Before you start:**

- [ ] Do I have a pen/whiteboard ready?
- [ ] Am I ready to write down numbers?
- [ ] Do I know the 4-step framework?

**During requirements (5-7 min):**

- [ ] Asked about functional requirements?
- [ ] Asked about scale (DAU, RPS, storage)?
- [ ] Asked about latency and consistency?
- [ ] Wrote down numbers on board?

**During entities & APIs (5-8 min):**

- [ ] Defined 3-5 core entities?
- [ ] Defined 5-8 key APIs?
- [ ] Mentioned database schema?

**During high-level design (10-12 min):**

- [ ] Drew simple box diagram?
- [ ] Walked through 2-3 request flows?
- [ ] Explained what each component does?
- [ ] Identified bottlenecks?

**During deep dive (15-20 min):**

- [ ] Picked 2 critical areas?
- [ ] Presented multiple approaches?
- [ ] Discussed trade-offs explicitly?
- [ ] Made and justified decisions?
- [ ] Used numbers/calculations?

**Communication throughout:**

- [ ] Thought out loud?
- [ ] Asked clarifying questions?
- [ ] Acknowledged interviewer hints?
- [ ] Managed time effectively?

**If yes to all, you're executing the framework correctly!** 🎯

---

## Practice Drill

**Use this for every mock interview:**

1. Set timer for 45 minutes
2. Use the 4-step framework
3. Record yourself or have a friend observe
4. Check the checklist above
5. Identify where you spent too much/little time
6. Adjust for next iteration

**After 10 practice rounds using this framework, it becomes automatic.** 🚀
