# System Design Case Studies

Real-world system design examples to practice for interviews. Each case study includes requirements, architecture diagrams, technology choices, and scaling strategies.

---

## Table of Contents

### Traditional Systems
1. [Twitter (Social Media Feed)](#twitter-social-media-feed)
2. [Netflix (Video Streaming)](#netflix-video-streaming)
3. [Uber (Ride Sharing)](#uber-ride-sharing)
4. [WhatsApp (Messaging)](#whatsapp-messaging)
5. [URL Shortener (TinyURL)](#url-shortener-tinyurl)

### Gen AI Systems
6. [ChatGPT (LLM Chat Application)](#chatgpt-llm-chat-application)
7. [Midjourney (AI Image Generation)](#midjourney-ai-image-generation)
8. [GitHub Copilot (AI Code Assistant)](#github-copilot-ai-code-assistant)
9. [Personalized Recommendation System (ML)](#personalized-recommendation-system-ml)
10. [RAG-based Q&A System](#rag-based-qa-system)

---

## Twitter (Social Media Feed)

### Requirements

**Functional:**
- Users can post tweets (280 characters)
- Users can follow other users
- Users see timeline of tweets from people they follow
- Users can like and retweet
- Search tweets

**Non-Functional:**
- 500M daily active users
- 200M tweets per day
- Timeline should load in < 1 second
- High availability (99.99%)
- Eventually consistent (okay to see tweets with slight delay)

### Capacity Estimation

**Storage:**
- 200M tweets/day × 280 chars × 2 bytes = 112 GB/day
- With metadata: ~200 GB/day
- For 5 years: 200 GB × 365 × 5 = 365 TB

**Traffic:**
- Read-heavy: 100:1 read-to-write ratio
- 200M tweets/day = 2,300 writes/sec
- Timeline reads: 230,000 reads/sec

### High-Level Architecture

```
┌─────────┐
│ Client  │
└────┬────┘
     │
┌────▼────────────────────────────┐
│     Load Balancer (Nginx)       │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│      API Gateway                │
└─┬──────┬──────┬─────────┬───────┘
  │      │      │         │
  │      │      │         │
┌─▼──┐ ┌─▼──┐ ┌─▼──┐   ┌─▼──────┐
│User│ │Tweet│ │Time│   │ Search │
│Svc │ │ Svc │ │line│   │  Svc   │
└─┬──┘ └─┬──┘ └─┬──┘   └─┬──────┘
  │      │      │         │
┌─▼──────▼──────▼─────────▼──────┐
│         Message Queue           │
│          (Kafka)                │
└─┬──────┬──────┬─────────┬──────┘
  │      │      │         │
┌─▼──┐ ┌─▼──┐ ┌─▼──┐   ┌─▼──────┐
│User│ │Tweet│ │Time│   │ Search │
│ DB │ │ DB  │ │line│   │ Index  │
│    │ │     │ │Cache│  │(Elastic)│
└────┘ └─────┘ └────┘   └────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Load Balancer** | Nginx | High performance, handles millions of connections |
| **API Gateway** | Kong | Rate limiting, authentication, routing |
| **User Service** | Node.js | Fast I/O for user operations |
| **Tweet Service** | Java/Spring | Robust for write-heavy operations |
| **User Database** | PostgreSQL (sharded) | Relational data, ACID for user info |
| **Tweet Database** | Cassandra | Write-optimized, handles massive scale |
| **Timeline Cache** | Redis | Sub-millisecond reads for hot timelines |
| **Message Queue** | Kafka | High-throughput event streaming |
| **Search** | Elasticsearch | Full-text search on tweets |
| **Object Storage** | S3 | Store images, videos |
| **CDN** | CloudFront | Serve static assets globally |

### Key Design Decisions

#### 1. Timeline Generation: Fan-out on Write vs Read

**Fan-out on Write (Chosen for most users):**
- When user tweets, push to all followers' timelines immediately
- Timeline read is just a cache lookup (fast!)
- Trade-off: Expensive for celebrities (millions of followers)

**Fan-out on Read (For celebrities):**
- Don't pre-compute timelines
- Generate timeline on-demand by merging tweets
- Trade-off: Slower reads, but saves write amplification

**Hybrid Approach:**
- Regular users: Fan-out on write
- Celebrities (>1M followers): Fan-out on read
- Best of both worlds

#### 2. Database Sharding Strategy

**Tweet Database (Cassandra):**
- Shard by `tweet_id` (hash-based)
- Evenly distributes writes
- Each shard handles subset of tweets

**User Database (PostgreSQL):**
- Shard by `user_id` (hash-based)
- Co-locate user data with their tweets
- Use consistent hashing for rebalancing

#### 3. Caching Strategy

**Timeline Cache (Redis):**
```
Key: user_id:timeline
Value: [tweet_id_1, tweet_id_2, ..., tweet_id_100]
TTL: 1 hour
```

**Tweet Cache:**
```
Key: tweet_id
Value: {tweet_data}
TTL: 24 hours
```

**Cache Invalidation:**
- New tweet → Update follower timelines (fan-out on write)
- Delete tweet → Remove from all caches
- Use cache-aside pattern

### Scaling Strategies

**Horizontal Scaling:**
- Stateless API servers (auto-scale based on traffic)
- Database sharding (add more shards as data grows)
- Redis cluster (partition timeline caches)

**Read Optimization:**
- CDN for static assets
- Redis cache for hot timelines
- Read replicas for user database

**Write Optimization:**
- Kafka for async processing
- Batch writes to Cassandra
- Write-through cache for new tweets

### Handling Edge Cases

**Celebrity Problem:**
- Don't fan-out tweets from users with >1M followers
- Generate timeline on-demand (fan-out on read)
- Cache celebrity tweets separately

**Thundering Herd:**
- When celebrity tweets, millions request timeline
- Use request coalescing (deduplicate identical requests)
- Stagger cache expiration times

**Hot Partitions:**
- Some shards get more traffic (trending topics)
- Use consistent hashing with virtual nodes
- Monitor and rebalance dynamically

---

## Netflix (Video Streaming)

### Requirements

**Functional:**
- Users can browse and search content
- Stream videos in multiple qualities (480p, 720p, 1080p, 4K)
- Resume playback from where they left off
- Personalized recommendations
- Download for offline viewing

**Non-Functional:**
- 200M subscribers globally
- 1B hours watched per week
- 99.99% availability
- Low latency (< 2 sec to start playback)
- Adaptive bitrate streaming

### Capacity Estimation

**Storage:**
- 10,000 titles
- Average movie: 2 hours, 5 GB (1080p)
- Multiple formats: 480p, 720p, 1080p, 4K
- Total: 10,000 × 5 GB × 4 formats = 200 TB

**Bandwidth:**
- 1B hours/week = 6M hours/hour = 1,666 concurrent streams/sec
- Average bitrate: 5 Mbps
- Total: 1,666 × 5 Mbps = 8.3 Tbps

### High-Level Architecture

```
┌─────────┐
│ Client  │
└────┬────┘
     │
┌────▼────────────────────────────┐
│     CDN (Open Connect)          │ ← 95% of traffic served here
└────┬────────────────────────────┘
     │ (CDN miss)
┌────▼────────────────────────────┐
│     API Gateway                 │
└─┬──────┬──────┬─────────┬───────┘
  │      │      │         │
┌─▼──┐ ┌─▼──┐ ┌─▼──┐   ┌─▼──────┐
│User│ │Video│ │Rec │   │Playback│
│Svc │ │ Svc │ │Svc │   │  Svc   │
└─┬──┘ └─┬──┘ └─┬──┘   └─┬──────┘
  │      │      │         │
┌─▼──┐ ┌─▼──┐ ┌─▼──┐   ┌─▼──────┐
│User│ │Video│ │ML  │   │Playback│
│ DB │ │Meta │ │Model│  │  State │
└────┘ └─────┘ └────┘   └────────┘
         │
    ┌────▼────┐
    │   S3    │ ← Master video files
    └─────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **CDN** | Open Connect (Netflix's own) | 95% traffic served from edge, low latency |
| **API Gateway** | Zuul (Netflix OSS) | Dynamic routing, resilience |
| **Microservices** | Java/Spring Boot | Scalable, maintainable |
| **User Database** | MySQL (sharded) | User profiles, subscriptions |
| **Video Metadata** | Cassandra | Fast reads, distributed |
| **Recommendations** | Apache Spark + ML | Personalized content |
| **Playback State** | DynamoDB | Low-latency reads/writes |
| **Video Storage** | S3 | Durable, scalable object storage |
| **Encoding** | FFmpeg | Transcode to multiple formats |
| **Message Queue** | Kafka | Event streaming for analytics |

### Key Design Decisions

#### 1. Content Delivery: CDN Strategy

**Open Connect (Netflix's CDN):**
- Deploy servers in ISP data centers
- Pre-populate popular content during off-peak hours
- Serve 95% of traffic from edge (low latency)

**How it Works:**
1. User requests video
2. CDN checks local cache
3. If hit: Stream from edge server (< 50ms latency)
4. If miss: Fetch from S3, cache, then stream

**Benefits:**
- Reduced bandwidth costs (ISPs don't charge for local traffic)
- Lower latency (content closer to users)
- Better user experience

#### 2. Adaptive Bitrate Streaming

**Problem:** Network conditions vary (WiFi → 4G → 3G)

**Solution:** Dynamic Adaptive Streaming over HTTP (DASH)
- Video split into small chunks (2-10 seconds)
- Each chunk available in multiple bitrates
- Client measures bandwidth and requests appropriate quality

**Example:**
```
Chunk 1: 1080p (good WiFi)
Chunk 2: 720p (WiFi degraded)
Chunk 3: 480p (switched to 4G)
Chunk 4: 720p (4G improved)
```

**Benefits:**
- Smooth playback (no buffering)
- Optimal quality for current network
- Better user experience

#### 3. Video Encoding Pipeline

**Workflow:**
1. Upload master file (4K, uncompressed)
2. Transcode to multiple formats:
   - 480p, 720p, 1080p, 4K
   - Different codecs (H.264, H.265, VP9)
   - Different audio tracks (languages)
3. Generate thumbnails (every 10 seconds)
4. Upload to S3
5. Distribute to CDN edge servers

**Optimization:**
- Parallel encoding (multiple workers)
- Prioritize popular content
- Use GPU for faster encoding

#### 4. Recommendation System

**Data Collection:**
- What you watch
- When you pause/rewind
- What you search
- What you rate

**ML Pipeline:**
1. Collect user interactions (Kafka)
2. Process with Spark
3. Train models (collaborative filtering, deep learning)
4. Generate personalized recommendations
5. A/B test different algorithms

**Personalization:**
- Different thumbnails for different users
- Personalized row ordering
- "Because you watched X" recommendations

### Scaling Strategies

**Global Distribution:**
- Deploy Open Connect servers in 1000+ locations
- Geo-routing (direct users to nearest edge)
- Regional failover

**Database Scaling:**
- Shard user database by `user_id`
- Cassandra for video metadata (distributed by default)
- DynamoDB for playback state (auto-scaling)

**Encoding at Scale:**
- Distributed encoding cluster (1000+ workers)
- Queue-based job distribution
- Priority queue (new releases first)

### Handling Edge Cases

**Peak Traffic (New Release):**
- Pre-populate CDN with new content
- Gradual rollout (region by region)
- Rate limiting on API

**CDN Failure:**
- Fallback to origin (S3)
- Degrade quality (serve lower bitrate)
- Redirect to alternate CDN

**Network Instability:**
- Adaptive bitrate (downgrade quality)
- Aggressive buffering (download ahead)
- Resume from last position

---

## Uber (Ride Sharing)

### Requirements

**Functional:**
- Riders request rides
- Drivers accept ride requests
- Real-time location tracking
- ETA calculation
- Fare calculation
- Payment processing
- Ratings and reviews

**Non-Functional:**
- 100M active users
- 15M trips per day
- Real-time (< 1 sec location updates)
- High availability (99.99%)
- Accurate ETA (within 2 minutes)

### Capacity Estimation

**Location Updates:**
- 1M concurrent drivers
- Update every 4 seconds
- 1M / 4 = 250,000 updates/sec

**Storage:**
- 15M trips/day × 365 days = 5.5B trips/year
- Each trip: ~5 KB (route, fare, etc.)
- Total: 5.5B × 5 KB = 27.5 TB/year

### High-Level Architecture

```
┌──────────┐          ┌──────────┐
│  Rider   │          │  Driver  │
│   App    │          │   App    │
└────┬─────┘          └────┬─────┘
     │                     │
     │  WebSocket          │  WebSocket
     │                     │
┌────▼─────────────────────▼─────┐
│     Load Balancer              │
└────┬───────────────────┬───────┘
     │                   │
┌────▼────┐         ┌────▼────────┐
│ Rider   │         │   Driver    │
│ Service │         │   Service   │
└────┬────┘         └────┬────────┘
     │                   │
     │              ┌────▼────────┐
     │              │  Location   │
     │              │   Service   │
     │              └────┬────────┘
     │                   │
┌────▼───────────────────▼─────┐
│      Matching Service        │
└────┬─────────────────────────┘
     │
┌────▼────┐  ┌─────────┐  ┌────────┐
│  Trip   │  │   ETA   │  │  Fare  │
│ Service │  │ Service │  │Service │
└────┬────┘  └────┬────┘  └────┬───┘
     │            │            │
┌────▼────────────▼────────────▼───┐
│         PostgreSQL (sharded)     │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│  Redis (driver locations)        │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│  Kafka (event streaming)         │
└──────────────────────────────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Real-time Communication** | WebSocket | Bi-directional, low-latency |
| **Location Service** | Node.js + Redis | Fast I/O, in-memory geospatial queries |
| **Matching Service** | Go | High performance, concurrent |
| **Trip Database** | PostgreSQL (sharded) | ACID for financial data |
| **Location Cache** | Redis (Geospatial) | Fast proximity searches |
| **Maps/Routing** | Google Maps API | Accurate ETA, routing |
| **Message Queue** | Kafka | Event streaming, analytics |
| **Payment** | Stripe API | PCI-compliant |

### Key Design Decisions

#### 1. Real-Time Location Tracking

**Challenge:** Track 1M drivers updating location every 4 seconds

**Solution: Redis Geospatial**
```redis
# Store driver location
GEOADD drivers:locations <longitude> <latitude> <driver_id>

# Find nearby drivers (within 5km)
GEORADIUS drivers:locations <rider_lon> <rider_lat> 5 km WITHDIST
```

**Benefits:**
- O(log N) proximity search
- Sub-millisecond queries
- Built-in distance calculation

**Optimization:**
- Partition by city (separate Redis instances)
- Update only when driver moves significantly (> 50 meters)
- Batch updates every 4 seconds

#### 2. Driver-Rider Matching Algorithm

**Factors:**
- Distance (closest driver)
- Driver rating
- Acceptance rate
- Estimated time to pickup

**Algorithm:**
1. Rider requests ride
2. Find drivers within 5km radius
3. Score each driver:
   ```
   score = (distance_weight × distance) + 
           (rating_weight × rating) + 
           (acceptance_weight × acceptance_rate)
   ```
4. Send request to top 3 drivers (in parallel)
5. First to accept gets the ride

**Optimization:**
- Pre-compute driver scores
- Cache nearby drivers
- Use machine learning for better matching

#### 3. ETA Calculation

**Approach:**
1. Get current traffic data (Google Maps API)
2. Calculate route (A* algorithm)
3. Estimate time based on:
   - Distance
   - Current traffic
   - Historical data (same route, same time of day)
   - Driver behavior (speed patterns)

**Caching:**
- Cache popular routes
- Update every 5 minutes
- Invalidate on traffic incidents

#### 4. Surge Pricing

**When:** Demand > Supply

**Algorithm:**
```python
surge_multiplier = min(
    max(1.0, demand / supply),
    3.0  # Cap at 3x
)

fare = base_fare × surge_multiplier
```

**Implementation:**
- Calculate per geographic area (grid-based)
- Update every minute
- Notify users of surge pricing

### Scaling Strategies

**Geographic Partitioning:**
- Partition by city/region
- Each region has its own:
  - Location service
  - Redis instance
  - Matching service
- Reduces cross-region traffic

**Database Sharding:**
- Shard trips by `trip_id` (hash-based)
- Shard users by `user_id`
- Co-locate related data

**WebSocket Scaling:**
- Stateful connections (sticky sessions)
- Use Redis pub/sub for cross-server communication
- Horizontal scaling with connection pooling

### Handling Edge Cases

**No Nearby Drivers:**
- Expand search radius (5km → 10km → 15km)
- Notify user of longer wait time
- Suggest alternative pickup location

**Driver Cancellation:**
- Immediately re-match with next available driver
- Penalize driver (lower acceptance rate)
- Compensate rider (discount on next ride)

**Network Issues:**
- Cache last known location
- Use dead reckoning (estimate position)
- Reconnect automatically

**Payment Failure:**
- Retry payment (exponential backoff)
- Allow ride to complete (collect later)
- Flag account for review

---

## WhatsApp (Messaging)

### Requirements

**Functional:**
- Send/receive text messages
- Group chats (up to 256 members)
- Media sharing (images, videos, documents)
- End-to-end encryption
- Online/offline status
- Read receipts
- Message delivery confirmation

**Non-Functional:**
- 2B users globally
- 100B messages per day
- Real-time delivery (< 1 sec)
- 99.99% availability
- End-to-end encryption (security)

### Capacity Estimation

**Messages:**
- 100B messages/day
- Average message: 100 bytes
- Total: 100B × 100 bytes = 10 TB/day
- For 5 years: 10 TB × 365 × 5 = 18 PB

**Media:**
- 10% of messages have media
- Average media: 1 MB
- Total: 10B × 1 MB = 10 PB/day

### High-Level Architecture

```
┌─────────┐
│ Client  │
└────┬────┘
     │ WebSocket
┌────▼────────────────────────────┐
│     Load Balancer               │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│   Chat Service (WebSocket)      │
└─┬──────────────────────────┬────┘
  │                          │
┌─▼──────┐              ┌────▼────┐
│ Online │              │ Message │
│ Status │              │ Queue   │
│ (Redis)│              │ (Kafka) │
└────────┘              └────┬────┘
                             │
                        ┌────▼────┐
                        │ Message │
                        │ Service │
                        └────┬────┘
                             │
                   ┌─────────┴─────────┐
                   │                   │
              ┌────▼────┐         ┌────▼────┐
              │ Message │         │  Media  │
              │   DB    │         │Storage  │
              │(Cassandra)│       │  (S3)   │
              └─────────┘         └─────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Real-time** | WebSocket (Erlang) | Handles millions of concurrent connections |
| **Message Queue** | Kafka | High-throughput message delivery |
| **Message Database** | Cassandra | Write-optimized, distributed |
| **Online Status** | Redis | Fast in-memory lookups |
| **Media Storage** | S3 | Scalable object storage |
| **Encryption** | Signal Protocol | End-to-end encryption |
| **Push Notifications** | FCM/APNs | Offline message delivery |

### Key Design Decisions

#### 1. Message Delivery Flow

**Online Recipient:**
```
Sender → WebSocket → Chat Service → Kafka → 
Recipient's Chat Service → WebSocket → Recipient
```

**Offline Recipient:**
```
Sender → WebSocket → Chat Service → Kafka → 
Message DB → Push Notification → Recipient
```

**Delivery Guarantees:**
- Single checkmark: Sent to server
- Double checkmark: Delivered to recipient
- Blue checkmarks: Read by recipient

#### 2. End-to-End Encryption

**Signal Protocol:**
1. Each user has public/private key pair
2. Sender encrypts message with recipient's public key
3. Only recipient can decrypt with their private key
4. Server cannot read messages

**Key Exchange:**
- Use Diffie-Hellman for initial key exchange
- Rotate keys periodically
- Perfect forward secrecy

#### 3. Group Chat Architecture

**Challenges:**
- 256 members
- Each message sent to 256 recipients
- Encryption for each recipient

**Solution:**
- Sender encrypts message once with group key
- Server fans out to all members
- Each member decrypts with shared group key

**Optimization:**
- Use sender keys (Signal Protocol)
- Lazy member addition (don't block on all members)

#### 4. Media Handling

**Upload Flow:**
1. Client uploads media to S3
2. Receives URL
3. Sends message with URL (encrypted)
4. Recipient downloads from S3

**Optimization:**
- Compress images (reduce size by 70%)
- Generate thumbnails
- CDN for popular media
- Expire media after 30 days (optional)

### Scaling Strategies

**WebSocket Scaling:**
- Each server handles 1M connections
- Use consistent hashing to route users
- Redis pub/sub for cross-server messaging

**Database Sharding:**
- Shard by `user_id` (hash-based)
- Co-locate user's messages on same shard
- Partition by time (recent messages hot, old messages cold)

**Geographic Distribution:**
- Deploy chat servers in multiple regions
- Route users to nearest region
- Replicate messages across regions

### Handling Edge Cases

**Message Ordering:**
- Use timestamp + sequence number
- Client-side ordering
- Handle out-of-order delivery

**Duplicate Messages:**
- Use message ID (UUID)
- Client deduplicates based on ID
- Idempotent message processing

**Network Partitions:**
- Queue messages locally
- Retry with exponential backoff
- Sync when connection restored

---

## URL Shortener (TinyURL)

### Requirements

**Functional:**
- Shorten long URLs to short codes (7 characters)
- Redirect short URL to original URL
- Custom aliases (optional)
- Analytics (click count, geography)
- Expiration (optional)

**Non-Functional:**
- 100M URLs shortened per month
- 10B redirects per month (100:1 read-to-write)
- Low latency (< 100ms redirect)
- High availability (99.9%)

### Capacity Estimation

**Storage:**
- 100M URLs/month × 12 months × 5 years = 6B URLs
- Each URL: ~500 bytes (original + short + metadata)
- Total: 6B × 500 bytes = 3 TB

**URL Length:**
- 7 characters, base62 (a-z, A-Z, 0-9)
- 62^7 = 3.5 trillion possible URLs
- Enough for 6B URLs

### High-Level Architecture

```
┌─────────┐
│ Client  │
└────┬────┘
     │
┌────▼────────────────────────────┐
│     Load Balancer               │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│      API Service                │
│  (Create, Redirect, Analytics)  │
└─┬──────────────────────────┬────┘
  │                          │
┌─▼──────┐              ┌────▼────┐
│ Cache  │              │   DB    │
│(Redis) │              │(Postgres)│
└────────┘              └─────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **API Service** | Node.js | Fast I/O, simple logic |
| **Database** | PostgreSQL | Relational, ACID |
| **Cache** | Redis | Fast lookups for hot URLs |
| **Analytics** | ClickHouse | Fast analytics queries |

### Key Design Decisions

#### 1. Short Code Generation

**Approach 1: Hash (MD5)**
```python
import hashlib

def shorten(url):
    hash = hashlib.md5(url.encode()).hexdigest()
    return hash[:7]  # Take first 7 characters
```

**Problem:** Collisions (two URLs → same hash)

**Approach 2: Counter (Chosen)**
```python
counter = 1000000  # Start from 1M

def shorten(url):
    global counter
    short_code = base62_encode(counter)
    counter += 1
    return short_code

def base62_encode(num):
    chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    result = ""
    while num > 0:
        result = chars[num % 62] + result
        num //= 62
    return result.zfill(7)  # Pad to 7 characters
```

**Benefits:**
- No collisions
- Sequential, predictable
- Easy to implement

**Distributed Counter:**
- Use database sequence
- Or pre-allocate ranges to each server

#### 2. Database Schema

```sql
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(7) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    click_count INT DEFAULT 0,
    INDEX idx_short_code (short_code)
);
```

#### 3. Caching Strategy

**Cache-Aside Pattern:**
```python
def redirect(short_code):
    # Check cache first
    url = redis.get(short_code)
    if url:
        return url
    
    # Cache miss, query database
    url = db.query("SELECT original_url FROM urls WHERE short_code = ?", short_code)
    
    # Store in cache
    redis.setex(short_code, 3600, url)  # TTL: 1 hour
    
    return url
```

**Cache Eviction:**
- LRU (Least Recently Used)
- TTL: 1 hour for hot URLs

#### 4. Analytics

**Click Tracking:**
```python
def redirect(short_code):
    url = get_url(short_code)
    
    # Async analytics (don't block redirect)
    kafka.send("clicks", {
        "short_code": short_code,
        "timestamp": now(),
        "ip": request.ip,
        "user_agent": request.user_agent
    })
    
    return redirect(url)
```

**Analytics Service:**
- Consume from Kafka
- Aggregate clicks (per hour, per day)
- Store in ClickHouse for fast queries

### Scaling Strategies

**Read Scaling:**
- Cache hot URLs in Redis
- Read replicas for database
- CDN for redirect service

**Write Scaling:**
- Shard database by `short_code` (hash-based)
- Pre-allocate ID ranges to avoid contention

**Global Distribution:**
- Deploy in multiple regions
- Geo-routing for low latency
- Replicate database across regions

---

# Gen AI Systems

---

## ChatGPT (LLM Chat Application)

### Requirements

**Functional:**
- Users can have conversations with AI
- Context-aware responses (remember conversation history)
- Support multiple conversation threads
- Stream responses token-by-token
- Support different models (GPT-3.5, GPT-4)
- Code execution capability
- Image understanding (multimodal)

**Non-Functional:**
- 100M weekly active users
- 10M concurrent users
- Response latency < 2 seconds (first token)
- 99.9% availability
- Handle 1B+ tokens per day
- Cost optimization (inference is expensive)

### Capacity Estimation

**Compute:**
- 10M concurrent users
- Average conversation: 20 turns
- Average tokens per turn: 500 (input) + 1000 (output)
- Total: 10M × 20 × 1500 = 300B tokens/day

**Cost:**
- GPT-4: $0.03 per 1K input tokens, $0.06 per 1K output tokens
- Daily cost: ~$15M (need aggressive optimization!)

**Storage:**
- Conversation history: 100M users × 10 conversations × 50KB = 50 TB

### High-Level Architecture

```
┌─────────┐
│ Client  │
└────┬────┘
     │ Server-Sent Events (SSE)
┌────▼────────────────────────────┐
│     Load Balancer + CDN         │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│      API Gateway                │
│  (Rate Limiting, Auth)          │
└─┬──────────────────────────┬────┘
  │                          │
┌─▼──────┐              ┌────▼────────┐
│ Chat   │              │   Model     │
│Service │              │  Router     │
└─┬──────┘              └────┬────────┘
  │                          │
┌─▼──────┐              ┌────▼────────┐
│Context │              │   Model     │
│Manager │              │  Inference  │
│(Redis) │              │   Cluster   │
└────────┘              └────┬────────┘
                             │
                   ┌─────────┴─────────┐
                   │                   │
              ┌────▼────┐         ┌────▼────┐
              │  GPU    │         │  GPU    │
              │ Server  │         │ Server  │
              │  Pool   │         │  Pool   │
              └─────────┘         └─────────┘

┌──────────────────────────────────┐
│  Vector DB (Conversation History)│
│        (Pinecone/Weaviate)       │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│  PostgreSQL (User, Metadata)     │
└──────────────────────────────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **API Gateway** | Kong | Rate limiting, authentication, routing |
| **Model Serving** | vLLM / TensorRT-LLM | Optimized inference, continuous batching |
| **GPU Infrastructure** | NVIDIA A100/H100 | High throughput, large memory |
| **Context Cache** | Redis | Fast conversation history lookup |
| **Vector Database** | Pinecone | Semantic search, conversation retrieval |
| **Message Queue** | Kafka | Async processing, logging |
| **Monitoring** | Prometheus + Grafana | GPU utilization, latency tracking |
| **Object Storage** | S3 | Store conversation logs |

### Key Design Decisions

#### 1. Model Serving Infrastructure

**Challenge:** GPT-4 is huge (1.7T parameters), expensive to run

**Solution: Multi-tier Model Architecture**
```
Fast queries → GPT-3.5-turbo (cheaper, faster)
Complex queries → GPT-4 (expensive, smarter)
Routing decision → Lightweight classifier
```

**Inference Optimization:**
- **Continuous Batching**: Batch multiple requests together
- **KV Cache**: Cache key-value pairs from previous tokens
- **Quantization**: Use INT8/FP16 instead of FP32 (2-4x faster)
- **Speculative Decoding**: Generate multiple tokens in parallel

**Example with vLLM:**
```python
from vllm import LLM, SamplingParams

llm = LLM(model="gpt-4", tensor_parallel_size=8)
sampling_params = SamplingParams(temperature=0.7, max_tokens=1000)

# Continuous batching handles multiple requests efficiently
outputs = llm.generate(prompts, sampling_params)
```

#### 2. Streaming Responses

**Why:** Better UX (users see tokens as they're generated)

**Implementation: Server-Sent Events (SSE)**
```python
@app.route('/chat/stream')
def stream_chat():
    def generate():
        for token in model.generate_stream(prompt):
            yield f"data: {json.dumps({'token': token})}\n\n"
    
    return Response(generate(), mimetype='text/event-stream')
```

**Benefits:**
- Users see response immediately (< 1 sec to first token)
- Can cancel long responses
- Better perceived performance

#### 3. Context Management

**Challenge:** Conversations can be long (100+ turns)

**Token Limit:**
- GPT-4: 128K tokens max
- Average conversation: 10K tokens
- Need to manage context window

**Strategy:**
```python
def manage_context(conversation_history, max_tokens=8000):
    # 1. Always keep system prompt
    context = [system_prompt]
    
    # 2. Keep last N messages
    recent_messages = conversation_history[-10:]
    
    # 3. Summarize older messages
    if len(conversation_history) > 10:
        summary = summarize(conversation_history[:-10])
        context.append({"role": "system", "content": f"Previous context: {summary}"})
    
    context.extend(recent_messages)
    return context
```

**Optimization:**
- Cache conversation embeddings (vector DB)
- Retrieve relevant past messages (RAG pattern)
- Compress old context with summarization

#### 4. Cost Optimization

**Strategies:**

**a) Prompt Caching**
```python
# Cache common system prompts
cache_key = hash(system_prompt)
if cache_key in redis:
    cached_kv = redis.get(cache_key)
    # Reuse KV cache (saves 50% compute)
```

**b) Model Routing**
```python
def route_to_model(query):
    complexity_score = classifier.predict(query)
    
    if complexity_score < 0.3:
        return "gpt-3.5-turbo"  # $0.002/1K tokens
    else:
        return "gpt-4"  # $0.03/1K tokens
```

**c) Response Caching**
```python
# Cache identical queries
query_hash = hash(normalize(query))
if query_hash in cache:
    return cache[query_hash]  # Free!
```

**d) Batch Processing**
- Group similar requests
- Process in single GPU batch
- 10x throughput improvement

#### 5. Safety & Moderation

**Content Filtering:**
```python
def moderate_content(text):
    # 1. Check for harmful content
    moderation_result = openai.Moderation.create(input=text)
    
    if moderation_result.flagged:
        return "I can't help with that request."
    
    # 2. PII detection
    if detect_pii(text):
        text = redact_pii(text)
    
    return text
```

**Rate Limiting:**
- Free tier: 3 requests/minute
- Plus tier: 50 requests/minute
- Enterprise: Custom limits

### Scaling Strategies

**Horizontal Scaling:**
- Stateless API servers (auto-scale)
- GPU server pool (add more GPUs)
- Load balance across model replicas

**GPU Optimization:**
- Multi-GPU inference (tensor parallelism)
- Pipeline parallelism (split model across GPUs)
- Dynamic batching (group requests)

**Geographic Distribution:**
- Deploy inference clusters in multiple regions
- Route to nearest cluster (low latency)
- Replicate models globally

### Handling Edge Cases

**Long Conversations:**
- Summarize old messages
- Use sliding window (keep recent context)
- Offer "new conversation" option

**GPU Failures:**
- Health checks on GPU servers
- Automatic failover to healthy GPUs
- Queue requests during recovery

**Rate Limit Exceeded:**
- Return 429 with Retry-After header
- Offer upgrade to paid tier
- Queue requests (process when capacity available)

**Toxic Content:**
- Pre-filter with moderation API
- Post-filter generated responses
- Log and review flagged content

---

## Midjourney (AI Image Generation)

### Requirements

**Functional:**
- Generate images from text prompts
- Multiple aspect ratios (square, portrait, landscape)
- Style variations (realistic, anime, artistic)
- Upscale images (4x resolution)
- Variations of existing images
- Blend multiple images

**Non-Functional:**
- 15M users
- 2M images generated per day
- Generation time < 60 seconds
- High-quality outputs (1024x1024+)
- 99.5% availability

### Capacity Estimation

**Compute:**
- 2M images/day
- Average generation: 50 steps × 4 images = 200 inference steps
- Total: 2M × 200 = 400M inference steps/day

**Storage:**
- 2M images/day × 5 MB/image = 10 TB/day
- For 1 year: 10 TB × 365 = 3.65 PB

**GPU Requirements:**
- 1 A100 GPU: ~100 images/hour
- Need: 2M images/day ÷ 24 hours ÷ 100 = ~830 GPUs

### High-Level Architecture

```
┌─────────┐
│ Discord │
│  Bot    │
└────┬────┘
     │
┌────▼────────────────────────────┐
│     API Gateway                 │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│   Job Queue (RabbitMQ)          │
│   Priority: Fast/Relax          │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│   Job Scheduler                 │
│   (GPU Orchestration)           │
└─┬──────────────────────────┬────┘
  │                          │
┌─▼──────┐              ┌────▼────────┐
│  GPU   │              │    GPU      │
│Worker 1│              │  Worker N   │
│(A100)  │              │  (A100)     │
└─┬──────┘              └────┬────────┘
  │                          │
  └──────────┬───────────────┘
             │
        ┌────▼────┐
        │   S3    │ ← Generated images
        └─────────┘

┌──────────────────────────────────┐
│  CDN (CloudFront)                │ ← Serve images
└──────────────────────────────────┘

┌──────────────────────────────────┐
│  PostgreSQL (Jobs, Users)        │
└──────────────────────────────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Model** | Stable Diffusion XL / Custom | High-quality generation |
| **GPU** | NVIDIA A100 (40GB/80GB) | Fast inference, large VRAM |
| **Job Queue** | RabbitMQ | Priority queues, reliable |
| **Orchestration** | Kubernetes + GPU Operator | Auto-scaling, GPU management |
| **Storage** | S3 | Scalable image storage |
| **CDN** | CloudFront | Fast image delivery |
| **Monitoring** | Prometheus + Grafana | GPU metrics, queue depth |

### Key Design Decisions

#### 1. Image Generation Pipeline

**Diffusion Model Workflow:**
```python
def generate_image(prompt, steps=50, guidance_scale=7.5):
    # 1. Encode text prompt
    text_embeddings = clip_model.encode(prompt)
    
    # 2. Start with random noise
    latent = torch.randn((1, 4, 64, 64))
    
    # 3. Iterative denoising (50 steps)
    for t in scheduler.timesteps:
        # Predict noise
        noise_pred = unet(latent, t, text_embeddings)
        
        # Remove predicted noise
        latent = scheduler.step(noise_pred, t, latent)
    
    # 4. Decode latent to image
    image = vae.decode(latent)
    
    return image
```

**Optimization:**
- **Half Precision (FP16)**: 2x faster, same quality
- **Flash Attention**: 3x faster attention mechanism
- **Compiled Models**: TorchScript/TensorRT (20% faster)
- **Batch Generation**: Generate 4 images at once

#### 2. Priority Queue System

**Tiers:**
- **Fast Mode**: Pay per image, immediate processing
- **Relax Mode**: Subscription, queued processing

**Queue Management:**
```python
class JobQueue:
    def __init__(self):
        self.fast_queue = PriorityQueue()  # High priority
        self.relax_queue = Queue()         # Low priority
    
    def get_next_job(self):
        # Always prioritize fast queue
        if not self.fast_queue.empty():
            return self.fast_queue.get()
        
        # Fall back to relax queue
        if not self.relax_queue.empty():
            return self.relax_queue.get()
        
        return None
```

**Benefits:**
- Monetization (fast mode premium)
- Better GPU utilization (relax fills idle time)
- Fair queuing

#### 3. GPU Orchestration

**Challenge:** Efficiently manage 1000+ GPUs

**Solution: Kubernetes + GPU Operator**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: image-generator
spec:
  containers:
  - name: generator
    image: midjourney/generator:latest
    resources:
      limits:
        nvidia.com/gpu: 1  # Request 1 GPU
```

**Auto-scaling:**
```python
# Scale based on queue depth
if queue_depth > 1000:
    scale_up_gpus(target=queue_depth / 100)
elif queue_depth < 100:
    scale_down_gpus(min_replicas=50)
```

**GPU Sharing:**
- Multi-Instance GPU (MIG): Split A100 into 7 instances
- Serve multiple users per GPU
- Better utilization (80%+ vs 50%)

#### 4. Image Upscaling

**Challenge:** Upscale 1024x1024 → 4096x4096

**Approach:**
1. **Latent Upscaling**: Upscale in latent space (faster)
2. **Real-ESRGAN**: Super-resolution model
3. **Tiled Processing**: Process in chunks (avoid OOM)

```python
def upscale_image(image, scale=4):
    # Split into tiles
    tiles = split_into_tiles(image, tile_size=512)
    
    # Upscale each tile
    upscaled_tiles = []
    for tile in tiles:
        upscaled = esrgan_model(tile, scale=scale)
        upscaled_tiles.append(upscaled)
    
    # Merge tiles
    return merge_tiles(upscaled_tiles)
```

#### 5. Cost Optimization

**Strategies:**

**a) Model Distillation**
- Train smaller, faster model
- 50% faster, 90% quality
- Use for "quick preview"

**b) Caching**
```python
# Cache similar prompts
prompt_embedding = embed(prompt)
similar = vector_db.search(prompt_embedding, k=1, threshold=0.95)

if similar:
    return similar[0].image  # Reuse!
```

**c) Spot Instances**
- Use AWS Spot GPUs (70% cheaper)
- Handle interruptions gracefully
- Checkpoint and resume jobs

### Scaling Strategies

**Horizontal Scaling:**
- Add more GPU workers
- Distribute across regions
- Load balance job queue

**Optimization:**
- Batch processing (4 images at once)
- Model compilation (TensorRT)
- Mixed precision (FP16)

**Storage Scaling:**
- S3 for long-term storage
- CDN for delivery
- Expire old images (30 days)

### Handling Edge Cases

**NSFW Content:**
- Pre-filter prompts (keyword blocking)
- Post-filter images (NSFW classifier)
- Blur and flag inappropriate content

**GPU Failures:**
- Health checks every 30 seconds
- Automatic job re-queuing
- Failover to healthy GPUs

**Queue Overflow:**
- Reject new jobs (return 503)
- Notify users of wait time
- Offer fast mode upgrade

---

## GitHub Copilot (AI Code Assistant)

### Requirements

**Functional:**
- Real-time code suggestions as user types
- Multi-language support (Python, JavaScript, Go, etc.)
- Context-aware (understand surrounding code)
- Function/class completion
- Comment-to-code generation
- Multi-line suggestions

**Non-Functional:**
- 1M+ active developers
- Latency < 300ms (for suggestions)
- 99.9% availability
- Support 50+ programming languages
- Privacy (don't leak code)

### Capacity Estimation

**Requests:**
- 1M developers
- Average: 100 suggestions per hour
- Total: 1M × 100 = 100M suggestions/hour = 28K requests/sec

**Model Inference:**
- Average context: 2K tokens
- Average completion: 100 tokens
- Total: 28K × 2.1K tokens = 58M tokens/sec

### High-Level Architecture

```
┌─────────────┐
│  VS Code    │
│  Extension  │
└──────┬──────┘
       │ HTTPS
┌──────▼──────────────────────────┐
│     API Gateway                 │
│  (Auth, Rate Limiting)          │
└──────┬──────────────────────────┘
       │
┌──────▼──────────────────────────┐
│   Suggestion Service            │
│  (Context Extraction)           │
└──┬─────────────────────────┬────┘
   │                         │
┌──▼──────┐           ┌──────▼─────┐
│ Cache   │           │   Model    │
│(Redis)  │           │  Inference │
└─────────┘           └──────┬─────┘
                             │
                   ┌─────────┴─────────┐
                   │                   │
              ┌────▼────┐         ┌────▼────┐
              │  GPU    │         │  GPU    │
              │ Server  │         │ Server  │
              └─────────┘         └─────────┘

┌──────────────────────────────────┐
│  Telemetry (Kafka)               │
│  (Accepted/Rejected suggestions) │
└──────────────────────────────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Model** | Codex (GPT-3.5 fine-tuned) | Code-optimized LLM |
| **Inference** | vLLM / TensorRT-LLM | Low latency, high throughput |
| **Cache** | Redis | Fast suggestion caching |
| **API** | FastAPI | Async, high performance |
| **Monitoring** | Datadog | Latency tracking, acceptance rate |

### Key Design Decisions

#### 1. Context Extraction

**Challenge:** What context to send to model?

**Strategy:**
```python
def extract_context(cursor_position, file_content):
    # 1. Get surrounding code (2K tokens)
    before_cursor = file_content[:cursor_position][-1000:]
    after_cursor = file_content[cursor_position:][:1000:]
    
    # 2. Include imports and function signatures
    imports = extract_imports(file_content)
    current_function = get_current_function(cursor_position)
    
    # 3. Include related files (if available)
    related_files = get_related_files(current_file)
    
    context = {
        "before": before_cursor,
        "after": after_cursor,
        "imports": imports,
        "function": current_function,
        "related": related_files[:500]  # Limit tokens
    }
    
    return context
```

**Optimization:**
- Only send relevant context (not entire file)
- Prioritize recent code
- Include type hints and docstrings

#### 2. Low-Latency Inference

**Target:** < 300ms end-to-end

**Breakdown:**
- Network: 50ms
- Context extraction: 20ms
- Model inference: 200ms
- Post-processing: 30ms

**Optimization Techniques:**

**a) Speculative Decoding**
```python
# Generate multiple tokens in parallel
# 2-3x faster for multi-token completions
tokens = model.generate_speculative(context, num_tokens=50)
```

**b) KV Cache Reuse**
```python
# Cache key-value pairs from context
# Only compute new tokens
cache_key = hash(context)
if cache_key in kv_cache:
    cached_kv = kv_cache[cache_key]
    # 50% faster inference
```

**c) Batching**
```python
# Batch multiple requests
# 5x throughput improvement
batch = collect_requests(timeout=10ms)
results = model.generate_batch(batch)
```

#### 3. Suggestion Ranking

**Challenge:** Model generates multiple suggestions

**Ranking Factors:**
1. **Likelihood**: Model confidence score
2. **Relevance**: Matches context
3. **Popularity**: Common patterns in training data
4. **User History**: Personalized based on past acceptances

```python
def rank_suggestions(suggestions, context, user_id):
    scores = []
    for suggestion in suggestions:
        score = (
            0.4 * suggestion.likelihood +
            0.3 * relevance_score(suggestion, context) +
            0.2 * popularity_score(suggestion) +
            0.1 * personalization_score(suggestion, user_id)
        )
        scores.append(score)
    
    # Return top suggestion
    return suggestions[argmax(scores)]
```

#### 4. Privacy & Security

**Challenges:**
- Don't train on user's private code
- Don't leak code between users
- Comply with licensing

**Solutions:**

**a) Telemetry Opt-in**
```python
# Only collect if user opts in
if user.telemetry_enabled:
    log_suggestion(context, suggestion, accepted=True)
```

**b) Code Filtering**
- Remove secrets (API keys, passwords)
- Redact PII
- Filter copyrighted code

**c) Differential Privacy**
- Add noise to training data
- Prevent memorization of specific code

#### 5. Multi-Language Support

**Challenge:** Support 50+ languages

**Approach:**
1. **Unified Model**: Single model for all languages
2. **Language Detection**: Auto-detect from file extension
3. **Language-Specific Prompts**: Customize for each language

```python
def get_language_prompt(language):
    prompts = {
        "python": "# Python code:\n",
        "javascript": "// JavaScript code:\n",
        "go": "// Go code:\n",
    }
    return prompts.get(language, "")
```

### Scaling Strategies

**Horizontal Scaling:**
- Stateless API servers
- GPU pool (add more GPUs)
- Geographic distribution

**Caching:**
- Cache common completions
- 30% cache hit rate
- Redis cluster for high throughput

**Model Optimization:**
- Quantization (INT8)
- Distillation (smaller model)
- Pruning (remove unused parameters)

### Handling Edge Cases

**Slow Network:**
- Show "loading" indicator
- Timeout after 500ms
- Cache last suggestion

**Model Hallucination:**
- Syntax validation
- Type checking
- Reject invalid code

**Rate Limiting:**
- 100 suggestions/minute (free)
- Unlimited (paid)
- Queue overflow → reject

---

## Personalized Recommendation System (ML)

### Requirements

**Functional:**
- Recommend products/content to users
- Personalized based on user history
- Real-time recommendations
- Trending items
- Similar items
- Collaborative filtering

**Non-Functional:**
- 100M users
- 1M items (products/videos/articles)
- Recommendation latency < 100ms
- Update recommendations daily
- High accuracy (CTR > 5%)

### Capacity Estimation

**Requests:**
- 100M users
- Average: 10 page views/day
- Total: 1B recommendations/day = 11.5K requests/sec

**Model Training:**
- User-item interactions: 10B events/month
- Feature matrix: 100M × 1M = 100 trillion entries (sparse)
- Training time: 24 hours (daily refresh)

### High-Level Architecture

```
┌─────────┐
│  User   │
└────┬────┘
     │
┌────▼────────────────────────────┐
│     API Gateway                 │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│  Recommendation Service         │
└─┬──────────────────────────┬────┘
  │                          │
┌─▼──────┐              ┌────▼────────┐
│ Cache  │              │   Feature   │
│(Redis) │              │   Store     │
└────────┘              └────┬────────┘
                             │
                   ┌─────────┴─────────┐
                   │                   │
              ┌────▼────┐         ┌────▼────┐
              │Candidate│         │ Ranking │
              │Generator│         │  Model  │
              └────┬────┘         └────┬────┘
                   │                   │
                   └─────────┬─────────┘
                             │
                        ┌────▼────┐
                        │ Results │
                        └─────────┘

┌──────────────────────────────────┐
│  Offline Training Pipeline       │
│  (Spark + TensorFlow)            │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│  Event Stream (Kafka)            │
│  (Clicks, Views, Purchases)      │
└──────────────────────────────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Candidate Generation** | ANN (Approximate Nearest Neighbors) | Fast similarity search |
| **Ranking Model** | TensorFlow / PyTorch | Deep learning for ranking |
| **Feature Store** | Feast | Centralized feature management |
| **Vector Database** | Faiss / Milvus | Fast vector similarity |
| **Training** | Apache Spark | Distributed training |
| **Serving** | TensorFlow Serving | Low-latency inference |
| **Cache** | Redis | Fast recommendation lookup |

### Key Design Decisions

#### 1. Two-Stage Recommendation

**Stage 1: Candidate Generation (Fast)**
- Goal: Narrow down from 1M items to 1000 candidates
- Method: Collaborative filtering, content-based

**Stage 2: Ranking (Accurate)**
- Goal: Rank top 1000 candidates
- Method: Deep learning model with rich features

```python
def recommend(user_id, k=10):
    # Stage 1: Generate candidates (fast)
    candidates = candidate_generator.get_candidates(user_id, k=1000)
    
    # Stage 2: Rank candidates (accurate)
    features = feature_store.get_features(user_id, candidates)
    scores = ranking_model.predict(features)
    
    # Return top K
    top_k = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)[:k]
    return [item for item, score in top_k]
```

#### 2. Collaborative Filtering

**Matrix Factorization:**
```python
# User-Item matrix (sparse)
# Users: 100M, Items: 1M
# Factorize into: User embeddings × Item embeddings

user_embedding = model.get_user_embedding(user_id)  # 128-dim
item_embeddings = model.get_all_item_embeddings()   # 1M × 128-dim

# Compute similarity (dot product)
scores = user_embedding @ item_embeddings.T

# Top K items
top_k = argsort(scores)[-10:]
```

**Optimization:**
- Use Approximate Nearest Neighbors (ANN)
- Faiss for fast similarity search
- 100ms for 1M items

#### 3. Feature Engineering

**User Features:**
- Demographics (age, gender, location)
- Behavior (clicks, purchases, time spent)
- Preferences (categories, brands)
- Recency (last activity)

**Item Features:**
- Category, brand, price
- Popularity (views, sales)
- Quality (ratings, reviews)
- Freshness (publish date)

**Contextual Features:**
- Time of day, day of week
- Device (mobile, desktop)
- Location

```python
def extract_features(user_id, item_id, context):
    features = {
        # User features
        "user_age": user_db.get_age(user_id),
        "user_gender": user_db.get_gender(user_id),
        "user_lifetime_purchases": user_db.get_purchase_count(user_id),
        
        # Item features
        "item_category": item_db.get_category(item_id),
        "item_price": item_db.get_price(item_id),
        "item_popularity": item_db.get_view_count(item_id),
        
        # Contextual features
        "hour_of_day": context.hour,
        "day_of_week": context.day,
        "device_type": context.device,
        
        # Interaction features
        "user_viewed_category": user_id in viewed_category(item_category),
        "user_purchased_brand": user_id in purchased_brand(item_brand),
    }
    
    return features
```

#### 4. Ranking Model

**Deep Neural Network:**
```python
class RankingModel(nn.Module):
    def __init__(self, num_features):
        super().__init__()
        self.fc1 = nn.Linear(num_features, 512)
        self.fc2 = nn.Linear(512, 256)
        self.fc3 = nn.Linear(256, 128)
        self.fc4 = nn.Linear(128, 1)  # Score
        
    def forward(self, features):
        x = F.relu(self.fc1(features))
        x = F.relu(self.fc2(x))
        x = F.relu(self.fc3(x))
        score = self.fc4(x)
        return score
```

**Training:**
```python
# Objective: Predict click probability
# Loss: Binary cross-entropy

for batch in dataloader:
    features, labels = batch  # labels: 1 (clicked), 0 (not clicked)
    
    scores = model(features)
    loss = F.binary_cross_entropy_with_logits(scores, labels)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

#### 5. Cold Start Problem

**New Users (No History):**
- Show trending items
- Ask for preferences (onboarding)
- Use demographic-based recommendations

**New Items (No Interactions):**
- Content-based filtering (similar to popular items)
- Boost in recommendations (exploration)
- A/B test different positions

### Scaling Strategies

**Offline Training:**
- Distributed training with Spark
- Daily model refresh
- Incremental updates (don't retrain from scratch)

**Online Serving:**
- Pre-compute recommendations (batch)
- Cache in Redis (TTL: 1 hour)
- Real-time updates for hot items

**Feature Store:**
- Centralized feature management
- Online features (real-time)
- Offline features (batch)

### Handling Edge Cases

**Filter Bubble:**
- Add diversity (recommend different categories)
- Exploration vs exploitation (10% random)
- Temporal diversity (don't repeat recent items)

**Popularity Bias:**
- Down-weight popular items
- Boost long-tail items
- Personalized popularity

**Seasonality:**
- Time-based features
- Trending items
- Holiday-specific recommendations

---

## RAG-based Q&A System

### Requirements

**Functional:**
- Answer questions based on knowledge base
- Cite sources
- Handle follow-up questions
- Support multiple document types (PDF, HTML, Markdown)
- Real-time document updates

**Non-Functional:**
- 1M documents in knowledge base
- 100K queries per day
- Response latency < 3 seconds
- High accuracy (>90% relevant answers)
- Scalable to 10M+ documents

### Capacity Estimation

**Storage:**
- 1M documents × 10 pages × 500 words = 5B words
- Average: 5 tokens/word = 25B tokens
- Embeddings: 25B × 1536 dims × 4 bytes = 150 TB

**Queries:**
- 100K queries/day = 1.15 queries/sec (average)
- Peak: 10x = 11.5 queries/sec

### High-Level Architecture

```
┌─────────┐
│  User   │
└────┬────┘
     │
┌────▼────────────────────────────┐
│     API Gateway                 │
└────┬────────────────────────────┘
     │
┌────▼────────────────────────────┐
│      Query Service              │
└─┬──────────────────────────┬────┘
  │                          │
┌─▼──────┐              ┌────▼────────┐
│ Query  │              │  Retrieval  │
│Rewriter│              │   Service   │
└────────┘              └────┬────────┘
                             │
                   ┌─────────▼─────────┐
                   │                   │
              ┌────▼────┐         ┌────▼────┐
              │ Vector  │         │Document │
              │   DB    │         │  Store  │
              │(Pinecone)│        │  (S3)   │
              └────┬────┘         └────┬────┘
                   │                   │
                   └─────────┬─────────┘
                             │
                        ┌────▼────┐
                        │   LLM   │
                        │(GPT-4)  │
                        └────┬────┘
                             │
                        ┌────▼────┐
                        │ Answer  │
                        └─────────┘

┌──────────────────────────────────┐
│  Ingestion Pipeline              │
│  (Document Processing)           │
└──────────────────────────────────┘
```

### Technology Choices

| Component | Technology | Why |
|-----------|------------|-----|
| **Vector Database** | Pinecone / Weaviate | Fast semantic search |
| **Embeddings** | OpenAI text-embedding-3 | High-quality embeddings |
| **LLM** | GPT-4 | Accurate answer generation |
| **Document Store** | S3 + Elasticsearch | Scalable storage + search |
| **Chunking** | LangChain | Smart document splitting |
| **Cache** | Redis | Cache frequent queries |

### Key Design Decisions

#### 1. Document Ingestion Pipeline

**Workflow:**
```python
def ingest_document(document_path):
    # 1. Load document
    doc = load_document(document_path)  # PDF, HTML, etc.
    
    # 2. Extract text
    text = extract_text(doc)
    
    # 3. Chunk into smaller pieces
    chunks = chunk_text(text, chunk_size=512, overlap=50)
    
    # 4. Generate embeddings
    embeddings = embedding_model.encode(chunks)
    
    # 5. Store in vector database
    for chunk, embedding in zip(chunks, embeddings):
        vector_db.upsert({
            "id": generate_id(),
            "text": chunk,
            "embedding": embedding,
            "metadata": {
                "source": document_path,
                "page": chunk.page_number
            }
        })
```

**Chunking Strategy:**
- Fixed size: 512 tokens
- Overlap: 50 tokens (preserve context)
- Respect boundaries (paragraphs, sentences)

#### 2. Semantic Search (Retrieval)

**Query Flow:**
```python
def retrieve_relevant_chunks(query, k=5):
    # 1. Embed query
    query_embedding = embedding_model.encode(query)
    
    # 2. Search vector database
    results = vector_db.query(
        vector=query_embedding,
        top_k=k,
        include_metadata=True
    )
    
    # 3. Return relevant chunks
    return [
        {
            "text": result.text,
            "source": result.metadata.source,
            "score": result.score
        }
        for result in results
    ]
```

**Optimization:**
- Approximate Nearest Neighbors (ANN)
- HNSW index (Hierarchical Navigable Small World)
- Sub-100ms search for 1M+ vectors

#### 3. Answer Generation (RAG)

**Prompt Engineering:**
```python
def generate_answer(query, relevant_chunks):
    # Build context from retrieved chunks
    context = "\n\n".join([
        f"Source: {chunk['source']}\n{chunk['text']}"
        for chunk in relevant_chunks
    ])
    
    # Create prompt
    prompt = f"""
    Answer the question based on the context below. 
    If the answer is not in the context, say "I don't know."
    
    Context:
    {context}
    
    Question: {query}
    
    Answer:
    """
    
    # Generate answer
    answer = llm.generate(prompt, max_tokens=500)
    
    # Add citations
    citations = [chunk['source'] for chunk in relevant_chunks]
    
    return {
        "answer": answer,
        "citations": citations
    }
```

**Benefits:**
- Grounded in knowledge base (no hallucination)
- Cites sources (verifiable)
- Up-to-date (reflects latest documents)

#### 4. Query Rewriting

**Challenge:** User queries are often vague

**Solution: Rewrite for better retrieval**
```python
def rewrite_query(query, conversation_history):
    prompt = f"""
    Rewrite the user's query to be more specific and searchable.
    
    Conversation history:
    {conversation_history}
    
    User query: {query}
    
    Rewritten query:
    """
    
    rewritten = llm.generate(prompt, max_tokens=100)
    return rewritten
```

**Example:**
- Original: "What about pricing?"
- Rewritten: "What are the pricing plans for the Enterprise tier?"

#### 5. Hybrid Search with Cross-Encoder Reranking

**Three-Stage Retrieval Pipeline:**

```
Stage 1: Hybrid Search (Fast, Broad)
  ↓
Stage 2: Cross-Encoder Reranking (Accurate, Expensive)
  ↓
Stage 3: Final Selection
```

**Stage 1: Hybrid Search**

Combine semantic (vector) and keyword (BM25) search for comprehensive retrieval.

```python
def hybrid_search(query, k=100):
    """
    Retrieve top candidates using both semantic and keyword search.
    Returns more candidates (k=100) for reranking.
    """
    # 1. Semantic search (vector similarity)
    query_embedding = embedding_model.encode(query)
    semantic_results = vector_db.query(
        vector=query_embedding,
        top_k=k,
        include_metadata=True
    )
    
    # 2. Keyword search (BM25)
    keyword_results = elasticsearch.search(
        query=query,
        size=k,
        fields=["text", "title"],
        analyzer="standard"
    )
    
    # 3. Reciprocal Rank Fusion (RRF)
    # Combines rankings from multiple sources
    combined_scores = reciprocal_rank_fusion(
        semantic_results,
        keyword_results,
        k=60  # RRF parameter
    )
    
    return combined_scores

def reciprocal_rank_fusion(semantic_results, keyword_results, k=60):
    """
    RRF formula: score = Σ(1 / (k + rank_i))
    where rank_i is the rank in each retrieval method
    """
    scores = defaultdict(float)
    
    # Add semantic scores
    for rank, result in enumerate(semantic_results, start=1):
        scores[result.id] += 1 / (k + rank)
    
    # Add keyword scores
    for rank, result in enumerate(keyword_results, start=1):
        scores[result.id] += 1 / (k + rank)
    
    # Sort by combined score
    ranked = sorted(scores.items(), key=lambda x: x[1], reverse=True)
    
    return ranked
```

**Why Hybrid Search?**

| Search Type | Strengths | Weaknesses |
|-------------|-----------|------------|
| **Semantic (Vector)** | Handles synonyms, paraphrasing, conceptual similarity | Misses exact matches, technical terms |
| **Keyword (BM25)** | Exact matches, technical terms, acronyms | Misses semantic similarity |
| **Hybrid (RRF)** | Best of both worlds | Slightly more complex |

**Stage 2: Cross-Encoder Reranking**

Cross-encoders provide more accurate relevance scoring but are computationally expensive.

```python
def rerank_with_cross_encoder(query, candidates, top_k=10):
    """
    Use cross-encoder to rerank top candidates.
    Cross-encoder jointly encodes query + document for accurate scoring.
    """
    # Prepare pairs for cross-encoder
    pairs = [(query, candidate.text) for candidate in candidates]
    
    # Score with cross-encoder (e.g., ms-marco-MiniLM-L-12-v2)
    scores = cross_encoder_model.predict(pairs)
    
    # Combine with original scores (optional)
    final_scores = []
    for i, candidate in enumerate(candidates):
        # Weighted combination
        final_score = (
            0.7 * scores[i] +           # Cross-encoder score (primary)
            0.3 * candidate.rrf_score   # RRF score (secondary)
        )
        final_scores.append((candidate, final_score))
    
    # Sort and return top K
    ranked = sorted(final_scores, key=lambda x: x[1], reverse=True)
    return [candidate for candidate, score in ranked[:top_k]]
```

**Complete Retrieval Flow:**

```python
def retrieve_and_rerank(query, final_k=5):
    """
    Complete enterprise-grade retrieval pipeline.
    """
    # Stage 1: Hybrid search (retrieve 100 candidates)
    candidates = hybrid_search(query, k=100)
    
    # Stage 2: Cross-encoder reranking (rerank to top 20)
    reranked_candidates = rerank_with_cross_encoder(
        query, 
        candidates, 
        top_k=20
    )
    
    # Stage 3: Diversity filtering (ensure diverse sources)
    diverse_results = apply_diversity_filter(
        reranked_candidates,
        max_per_source=2  # Max 2 chunks from same document
    )
    
    # Return final top K
    return diverse_results[:final_k]

def apply_diversity_filter(results, max_per_source=2):
    """
    Ensure diversity in results (avoid all chunks from same doc).
    """
    source_counts = defaultdict(int)
    filtered = []
    
    for result in results:
        source = result.metadata.source
        if source_counts[source] < max_per_source:
            filtered.append(result)
            source_counts[source] += 1
    
    return filtered
```

**Performance Comparison:**

| Method | Latency | Accuracy (NDCG@10) | Use Case |
|--------|---------|-------------------|----------|
| **Semantic Only** | 50ms | 0.65 | Fast, general queries |
| **Keyword Only** | 30ms | 0.60 | Exact match queries |
| **Hybrid (RRF)** | 80ms | 0.75 | Balanced performance |
| **Hybrid + Cross-Encoder** | 200ms | 0.85 | High accuracy needed |

#### 6. Evaluation Metrics (Enterprise-Level)

**Offline Evaluation (Pre-Production)**

Evaluate system performance using labeled test datasets.

**A) Retrieval Metrics**

```python
def evaluate_retrieval(test_queries, ground_truth):
    """
    Evaluate retrieval quality using standard IR metrics.
    """
    metrics = {
        'mrr': [],      # Mean Reciprocal Rank
        'ndcg': [],     # Normalized Discounted Cumulative Gain
        'precision': [], # Precision@K
        'recall': []    # Recall@K
    }
    
    for query, relevant_docs in test_queries:
        # Retrieve documents
        retrieved = retrieve_and_rerank(query, final_k=10)
        
        # Calculate metrics
        metrics['mrr'].append(mean_reciprocal_rank(retrieved, relevant_docs))
        metrics['ndcg'].append(ndcg_at_k(retrieved, relevant_docs, k=10))
        metrics['precision'].append(precision_at_k(retrieved, relevant_docs, k=5))
        metrics['recall'].append(recall_at_k(retrieved, relevant_docs, k=10))
    
    # Aggregate
    return {
        'MRR': np.mean(metrics['mrr']),
        'NDCG@10': np.mean(metrics['ndcg']),
        'Precision@5': np.mean(metrics['precision']),
        'Recall@10': np.mean(metrics['recall'])
    }

def mean_reciprocal_rank(retrieved, relevant):
    """
    MRR = 1 / rank of first relevant document
    """
    for rank, doc in enumerate(retrieved, start=1):
        if doc.id in relevant:
            return 1.0 / rank
    return 0.0

def ndcg_at_k(retrieved, relevant, k=10):
    """
    NDCG = DCG / IDCG
    Measures ranking quality with position-based discounting.
    """
    # DCG (Discounted Cumulative Gain)
    dcg = sum([
        (1 if doc.id in relevant else 0) / np.log2(rank + 1)
        for rank, doc in enumerate(retrieved[:k], start=1)
    ])
    
    # IDCG (Ideal DCG)
    ideal_ranks = min(len(relevant), k)
    idcg = sum([1.0 / np.log2(rank + 1) for rank in range(1, ideal_ranks + 1)])
    
    return dcg / idcg if idcg > 0 else 0.0

def precision_at_k(retrieved, relevant, k=5):
    """
    Precision@K = (# relevant in top K) / K
    """
    relevant_retrieved = sum([1 for doc in retrieved[:k] if doc.id in relevant])
    return relevant_retrieved / k

def recall_at_k(retrieved, relevant, k=10):
    """
    Recall@K = (# relevant in top K) / (total # relevant)
    """
    relevant_retrieved = sum([1 for doc in retrieved[:k] if doc.id in relevant])
    return relevant_retrieved / len(relevant) if len(relevant) > 0 else 0.0
```

**B) Answer Quality Metrics**

```python
def evaluate_answer_quality(test_queries, ground_truth_answers):
    """
    Evaluate generated answer quality.
    """
    metrics = {
        'exact_match': [],
        'f1_score': [],
        'bleu': [],
        'rouge_l': [],
        'bertscore': []
    }
    
    for query, ground_truth in test_queries:
        # Generate answer
        retrieved_chunks = retrieve_and_rerank(query, final_k=5)
        generated_answer = generate_answer(query, retrieved_chunks)
        
        # Calculate metrics
        metrics['exact_match'].append(
            exact_match(generated_answer, ground_truth)
        )
        metrics['f1_score'].append(
            f1_score(generated_answer, ground_truth)
        )
        metrics['bleu'].append(
            bleu_score(generated_answer, ground_truth)
        )
        metrics['rouge_l'].append(
            rouge_l_score(generated_answer, ground_truth)
        )
        metrics['bertscore'].append(
            bert_score(generated_answer, ground_truth)
        )
    
    return {
        'Exact Match': np.mean(metrics['exact_match']),
        'F1 Score': np.mean(metrics['f1_score']),
        'BLEU': np.mean(metrics['bleu']),
        'ROUGE-L': np.mean(metrics['rouge_l']),
        'BERTScore': np.mean(metrics['bertscore'])
    }
```

**C) Faithfulness & Hallucination Detection**

```python
def evaluate_faithfulness(test_queries):
    """
    Measure if answers are grounded in retrieved context.
    """
    metrics = {
        'faithfulness': [],
        'answer_relevance': [],
        'context_relevance': []
    }
    
    for query in test_queries:
        retrieved_chunks = retrieve_and_rerank(query, final_k=5)
        answer = generate_answer(query, retrieved_chunks)
        
        # Faithfulness: Is answer supported by context?
        metrics['faithfulness'].append(
            check_faithfulness(answer, retrieved_chunks)
        )
        
        # Answer Relevance: Does answer address the query?
        metrics['answer_relevance'].append(
            check_answer_relevance(query, answer)
        )
        
        # Context Relevance: Are retrieved chunks relevant?
        metrics['context_relevance'].append(
            check_context_relevance(query, retrieved_chunks)
        )
    
    return {
        'Faithfulness': np.mean(metrics['faithfulness']),
        'Answer Relevance': np.mean(metrics['answer_relevance']),
        'Context Relevance': np.mean(metrics['context_relevance'])
    }

def check_faithfulness(answer, context_chunks):
    """
    Use NLI model to verify answer is entailed by context.
    """
    context = " ".join([chunk.text for chunk in context_chunks])
    
    # Natural Language Inference
    result = nli_model.predict(premise=context, hypothesis=answer)
    
    # Return 1 if entailed, 0 if contradicted, 0.5 if neutral
    return 1.0 if result == "entailment" else 0.0
```

**Online Evaluation (Production)**

Monitor real-time performance with user interactions.

**A) User Engagement Metrics**

```python
class OnlineMetricsTracker:
    """
    Track online metrics in production.
    """
    def __init__(self):
        self.metrics_db = MetricsDatabase()
    
    def track_query(self, query_id, query, results, answer):
        """
        Log query and track user interactions.
        """
        self.metrics_db.log_query({
            'query_id': query_id,
            'query': query,
            'timestamp': datetime.now(),
            'num_results': len(results),
            'answer_length': len(answer),
            'latency_ms': self.measure_latency()
        })
    
    def track_user_feedback(self, query_id, feedback):
        """
        Track explicit user feedback.
        """
        self.metrics_db.log_feedback({
            'query_id': query_id,
            'thumbs_up': feedback.thumbs_up,
            'thumbs_down': feedback.thumbs_down,
            'comment': feedback.comment,
            'timestamp': datetime.now()
        })
    
    def track_click(self, query_id, clicked_result_rank):
        """
        Track which results users click.
        """
        self.metrics_db.log_click({
            'query_id': query_id,
            'clicked_rank': clicked_result_rank,
            'timestamp': datetime.now()
        })
    
    def calculate_daily_metrics(self):
        """
        Calculate aggregated daily metrics.
        """
        return {
            # Engagement
            'total_queries': self.count_queries(),
            'unique_users': self.count_unique_users(),
            'avg_queries_per_user': self.avg_queries_per_user(),
            
            # Performance
            'avg_latency_ms': self.avg_latency(),
            'p95_latency_ms': self.p95_latency(),
            'p99_latency_ms': self.p99_latency(),
            
            # Quality
            'answer_acceptance_rate': self.acceptance_rate(),
            'thumbs_up_rate': self.thumbs_up_rate(),
            'ctr': self.click_through_rate(),
            'mrr': self.online_mrr(),
            
            # Errors
            'error_rate': self.error_rate(),
            'timeout_rate': self.timeout_rate(),
            'no_answer_rate': self.no_answer_rate()
        }
```

**B) Key Online Metrics**

| Metric | Formula | Target | Description |
|--------|---------|--------|-------------|
| **CTR** | Clicks / Impressions | > 30% | Click-through rate on results |
| **Answer Acceptance** | Accepted / Total | > 70% | Users accept answer (thumbs up) |
| **MRR (Online)** | Avg(1 / rank of clicked) | > 0.8 | Mean reciprocal rank of clicks |
| **Session Success** | Sessions with answer / Total | > 85% | User found answer in session |
| **Avg Latency** | Avg response time | < 2s | Average end-to-end latency |
| **P95 Latency** | 95th percentile | < 3s | Tail latency |
| **Error Rate** | Errors / Total | < 1% | System errors |
| **No Answer Rate** | "I don't know" / Total | < 10% | System couldn't answer |

**C) A/B Testing Framework**

```python
class ABTestFramework:
    """
    A/B test different retrieval/generation strategies.
    """
    def __init__(self):
        self.experiments = {}
    
    def create_experiment(self, name, variants):
        """
        Create new A/B test.
        
        Example:
        variants = {
            'control': {'retrieval': 'semantic_only'},
            'treatment': {'retrieval': 'hybrid_with_reranking'}
        }
        """
        self.experiments[name] = {
            'variants': variants,
            'start_date': datetime.now(),
            'metrics': defaultdict(list)
        }
    
    def assign_variant(self, user_id, experiment_name):
        """
        Assign user to variant (50/50 split).
        """
        hash_value = hash(f"{user_id}_{experiment_name}")
        return 'treatment' if hash_value % 2 == 0 else 'control'
    
    def track_metric(self, experiment_name, variant, metric_name, value):
        """
        Track metric for variant.
        """
        self.experiments[experiment_name]['metrics'][variant].append({
            'metric': metric_name,
            'value': value,
            'timestamp': datetime.now()
        })
    
    def analyze_results(self, experiment_name):
        """
        Statistical analysis of A/B test results.
        """
        experiment = self.experiments[experiment_name]
        
        # Calculate metrics for each variant
        results = {}
        for variant in experiment['variants']:
            metrics = experiment['metrics'][variant]
            results[variant] = {
                'ctr': np.mean([m['value'] for m in metrics if m['metric'] == 'ctr']),
                'acceptance_rate': np.mean([m['value'] for m in metrics if m['metric'] == 'acceptance']),
                'latency': np.mean([m['value'] for m in metrics if m['metric'] == 'latency'])
            }
        
        # Statistical significance test (t-test)
        control_ctr = [m['value'] for m in experiment['metrics']['control'] if m['metric'] == 'ctr']
        treatment_ctr = [m['value'] for m in experiment['metrics']['treatment'] if m['metric'] == 'ctr']
        
        t_stat, p_value = stats.ttest_ind(control_ctr, treatment_ctr)
        
        return {
            'results': results,
            'p_value': p_value,
            'significant': p_value < 0.05,
            'winner': 'treatment' if results['treatment']['ctr'] > results['control']['ctr'] else 'control'
        }
```

**D) Monitoring Dashboard**

```python
# Prometheus metrics for monitoring
from prometheus_client import Counter, Histogram, Gauge

# Counters
queries_total = Counter('rag_queries_total', 'Total queries')
answers_accepted = Counter('rag_answers_accepted', 'Answers accepted by users')
errors_total = Counter('rag_errors_total', 'Total errors', ['error_type'])

# Histograms
query_latency = Histogram('rag_query_latency_seconds', 'Query latency')
retrieval_latency = Histogram('rag_retrieval_latency_seconds', 'Retrieval latency')
generation_latency = Histogram('rag_generation_latency_seconds', 'Generation latency')

# Gauges
active_users = Gauge('rag_active_users', 'Active users')
cache_hit_rate = Gauge('rag_cache_hit_rate', 'Cache hit rate')

# Usage
@query_latency.time()
def process_query(query):
    queries_total.inc()
    
    try:
        with retrieval_latency.time():
            chunks = retrieve_and_rerank(query)
        
        with generation_latency.time():
            answer = generate_answer(query, chunks)
        
        return answer
    except Exception as e:
        errors_total.labels(error_type=type(e).__name__).inc()
        raise
```

**Enterprise Evaluation Summary:**

| Phase | Metrics | Frequency | Purpose |
|-------|---------|-----------|---------|
| **Offline** | MRR, NDCG, Precision, Recall, F1, BLEU | Weekly | Validate improvements before deployment |
| **Online** | CTR, Acceptance Rate, Latency, Error Rate | Real-time | Monitor production performance |
| **A/B Testing** | Comparative metrics | Per experiment | Test new features safely |
| **User Feedback** | Thumbs up/down, Comments | Continuous | Qualitative insights |



### Scaling Strategies

**Vector Database:**
- Shard by document collection
- Replicate for high availability
- Use approximate search (trade accuracy for speed)

**LLM Inference:**
- Cache frequent queries
- Batch similar queries
- Use smaller model for simple questions

**Document Updates:**
- Incremental updates (don't re-index everything)
- Background indexing (don't block queries)
- Version control (track changes)

### Handling Edge Cases

**No Relevant Documents:**
```python
if max(scores) < 0.7:  # Low confidence
    return "I don't have enough information to answer that question."
```

**Contradictory Information:**
```python
# Present multiple perspectives
answer = """
Based on the documents:
- Source A says: {answer_a}
- Source B says: {answer_b}
"""
```

**Long Documents:**
- Hierarchical chunking (summaries + details)
- Multi-hop retrieval (retrieve → read → retrieve again)

---

## Summary

---

Each case study demonstrates:
- **Requirements gathering** (functional + non-functional)
- **Capacity estimation** (storage, bandwidth, QPS)
- **Technology choices** (with justification)
- **Key design decisions** (trade-offs explained)
- **Scaling strategies** (how to handle growth)
- **Edge cases** (real-world challenges)

### Interview Tips

1. **Start with requirements** - Ask clarifying questions
2. **Estimate capacity** - Show you can do back-of-envelope calculations
3. **Draw diagrams** - Visual architecture helps interviewer follow
4. **Explain trade-offs** - Every choice has pros/cons
5. **Think about scale** - Start simple, then explain how to scale
6. **Handle edge cases** - Show you think about real-world problems

### Common Patterns Across Systems

- **Caching** (Redis) - Almost every system
- **Load Balancing** - Distribute traffic
- **Database Sharding** - Scale writes
- **Message Queues** (Kafka) - Async processing
- **CDN** - Global content delivery
- **Microservices** - Independent scaling

Good luck with your interviews! 🚀
