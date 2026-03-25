# 📱 Design: Instagram Reels / YouTube Shorts

---


### ✅ Functional Requirements
- Users can upload short videos (<90s)  
- Users can scroll infinitely (feed)  
- Videos autoplay instantly  
- Feed is personalized  
- Users can like, comment, share  
- System tracks engagement (watch time, likes, etc.)  
- Trending videos should surface  

### ⚙️ Non-Functional Requirements
- Low latency (especially playback start)  
- High availability  
- Massive scale (hundreds of millions of users)  
- High write throughput (uploads + engagement)  
- Strong consistency for ownership (uploads)  
- Eventual consistency acceptable for metrics  

---

## 2. 📊 Capacity Estimation

> “Let me do a quick back-of-the-envelope estimation.”

- **250M DAU**
- **30 videos/user/day → 7.5B views/day**
- **Avg video size = 7MB**  
  → ~52 PB/day bandwidth  

- **Uploads = 25M/day**  
  → ~1PB storage/day  

### 🔥 Peak Traffic
- ~400K video starts/sec  

### ✅ Insight
> “This is clearly a read-heavy, CDN-dominated system, and optimizing video delivery latency is critical.”

---

## 3. 🏗️ High-Level Architecture

> “I’ll divide the system into 3 major parts:”

- Upload pipeline  
- Feed generation + ranking  
- Video delivery + engagement tracking 

---

## 4. 🎥 Upload Flow

> “Let’s first design the upload pipeline.”

### Step 1: Upload Initialization
- Client hits upload API  
- Generate globally unique video ID (Snowflake)  

👉 **Why?**  
> “We need unique IDs across distributed systems.”

---

### Step 2: Metadata Write
Stored in **Distributed SQL DB**

**Fields:**
- `video_id`, `user_id`  
- `status (init → processing → ready)`  
- `duration`, `visibility`  

👉 **Why SQL?**  
> “We need strong consistency for ownership and correctness.”

---

### Step 3: Upload to Object Storage
- Client uploads directly to S3-like storage  
- File is chunked + replicated  

👉  
> “We bypass backend to reduce load and improve scalability.”

---

### Step 4: Async Processing (Kafka)
- Upload event → Kafka  

**Workers consume:**
- Transcoding (FFmpeg)  
- Generate multiple resolutions (240p–720p)  
- Segment into 1–2 sec chunks  

---

### Step 5: Multi-Region Replication
- Same region → synchronous  
- Cross-region → asynchronous  

👉 **Trade-off:**  
> “We prioritize durability over availability for uploads.”

---

## 5. 📱 Feed Generation

> “Now the most critical part — personalized feed.”

### Step 1: Candidate Generation

We combine 3 sources:
- Follow graph (Graph DB)  
- ML recommendations (vector similarity)  
- Trending videos (Redis)  

👉 Output:  
- ~1000–1500 candidates  

---

### Step 2: Ranking

#### User Features
- Watch time  
- Like ratio  
- Completion rate  

#### Video Features
- Engagement velocity  
- Global watch time  
- Category  

#### Context Features
- Time of day  
- Network speed  
- Device type  

---

### Step 3: ML Model

Predict:
- P(3-sec watch)  
- P(full watch)  
- P(like)  

👉 Final:
- Compute score  
- Return top 20 videos  

### ✅ Key Insight
> “This is a fanout-on-read system, which scales better than fanout-on-write for large follower graphs.”

---

## 6. ⚡ Video Delivery (CDN Strategy)

> “Video delivery is handled entirely via CDN.”

### Key Points
- Videos stored as segments (HLS/DASH)  
- CDN caches popular content at edge  

### 🔥 Advanced FAANG Points
- Signed URLs for secure access  
- Cache pre-warming for trending videos  
- Geo-routing to nearest edge  
- Adaptive bitrate streaming  

👉 **Insight:**  
> “This ensures low startup latency and smooth playback globally.”

---

## 7. 🔄 Playback Optimization

- While watching video 1 → prefetch video 2 segments  

👉 Result:  
> “Instant playback — no buffering during scroll”

---

## 8. ❤️ Engagement System (Likes / Views / Comments)

### ❌ Naive Approach
- Update DB per like → ❌ will crash  

---

### ✅ Correct Design

#### Step 1: Store Relationship
- `(user_id, video_id)`  

👉 Prevents:
- Duplicate likes  
- Enables unlike  

---

#### Step 2: Event Streaming
- Send event → Kafka  

#### Step 3: Aggregation Service
- Batch events (1–5 sec)  
- Maintain in-memory counters  

#### Step 4: Write to DB (bulk)

#### Step 5: Cache in Redis
- Feed reads from Redis  

### ✅ Key Insight
> “We accept eventual consistency for engagement counters to achieve scalability.”

---

## 9. 🗄️ Database Choices

| Component | DB |
|----------|----|
| Metadata | Distributed SQL |
| Likes    | KV Store |
| Stats    | NoSQL |
| Cache    | Redis |
| Graph    | Graph DB |

---

## 10. ⚠️ Bottlenecks & Solutions

### 🔥 1. Viral Content Hotspot
**Problem:**  
- Millions of likes/sec  

**Solution:**
- Sharded counters  
- Kafka buffering  
- Batch aggregation  

---

### 🔥 2. Storage Explosion
**Problem:**  
- 1PB/day  

**Solution:**
- Compression  
- Cold storage (Glacier)  
- Retention policies  

---

### 🔥 3. ML Latency
**Problem:**  
- Ranking must be fast  

**Solution:**
- Precompute features  
- Cache embeddings  
- Online inference <50ms  

---

### 🔥 4. CDN Cache Miss
**Solution:**
- Pre-warm trending videos  
- Smart TTL  

---

### 🔥 5. Failure Handling
- Kafka retries  
- Idempotent consumers  
- Dead letter queues  

---

## 11. 🎯 Final Wrap-Up

> “To summarize:”

- We separate upload and feed pipelines  
- Use event-driven architecture with Kafka  
- Optimize reads using CDN and Redis  
- Use ML-based ranking for personalization  
- Accept eventual consistency for scalability  

### ✅ Final Insight
> “This design ensures low latency, high scalability, and a personalized user experience.” 
