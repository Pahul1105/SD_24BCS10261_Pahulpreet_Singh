# Author Information
* **Name:** Pahulpreet Singh
* **UID:** 24BCS10261

---

# Scalable URL Shortener — System Design Documentation

This repository contains the comprehensive system design artifacts, functional specifications, architecture workflows, API contracts, and scale estimations for a distributed high-performance URL Shortener service.

---

## 1. Functional Requirements (`Functional requirements.png`)

The system fulfills core functional requirements (FR-1 through FR-6) through an end-to-end request pipeline backed by strict system guarantees.

### Pipeline Execution Stages
1. **Client / Consumer:** Sends `POST /v1/urls` (creation) or `GET /{shortCode}` (redirection).
2. **Ingress Layer (`[FR-1 & FR-6]`):** Validates target URL length (<2,000 characters) and enforces dynamic rate limits.
3. **Idempotency Layer (`[FR-5]`):** Checks `Idempotency-Key` headers to safely deduplicate client retry attempts.
4. **Allocation Layer (`[FR-2]`):** Generates unique Base62 encoded keys and resolves potential collisions.
5. **Resolution & Storage (`[FR-3 & FR-4]`):** Manages key lookups and returns HTTP `302 Found` redirections or appropriate `404`/`410` error codes.

### Governing System Guarantees
* **Guarantee 1: Uniqueness:** Each short code maps strictly to one target URL.
* **Guarantee 2: Read-After-Write:** Immediate `GET` requests successfully resolve right after a `201 Created` response.
* **Guarantee 3: Non-Predictability:** Short codes are opaque strings with no sequential enumeration leaks.

---

## 2. Non-Functional Requirements & SLAs (`Non-functional requirements.png`)

System quality attributes, high-availability targets, and operational SLAs designed for enterprise scale:

### System Quality Attributes
* **[NFR-1] High Availability & Fault Tolerance:** Targets **99.99% uptime** (maximum ~52 minutes of downtime per year) with multi-region active-passive fallback for read caches and graceful degradation during network partitions.
* **[NFR-2] Ultra-Low Latency:** Redirection reads target **p99 < 10ms** via distributed in-memory caching with a cache hit ratio goal > 95%. Creation writes target **p99 < 50ms**.
* **[NFR-3] Scalability & High Throughput:** Supports a 10:1 Read-to-Write traffic ratio with horizontal scaling for stateless API nodes and sharded databases.
* **[NFR-4] Security & Data Durability:** Enforces TLS 1.3 encryption in transit, AES-256 at rest, robust rate-limiting defense mechanisms against DDoS/crawlers, and persistent backups with a 10-year retention guarantee.

### Target Scale Metrics & Capacity SLAs
* **Peak Read Throughput:** 10,000 QPS (Redirection)
* **Peak Write Throughput:** 1,000 QPS (Creation)
* **Datastore Growth Rate:** ~3.6 TB / Year
* **Redirection Latency Target:** < 10 ms (p99)

---

## 3. High-Level Architecture & Request Flow (`High level design.png`)

The architecture decouples read paths from write paths using caching, asynchronous message streams, and pre-generated key stores.

### Component Breakdown
1. **Client Tier:** Web browsers, mobile apps, and programmatic REST clients.
2. **API Gateway & Load Balancer:** Handles TLS termination, rate limiting, token authentication, and regional traffic routing.
3. **URL App Service:** Stateless application nodes handling Base62 encoding and orchestration.
4. **Key Generation Service (KGS):** Pre-computes Base62 keys offline in in-memory buffers to eliminate database write locks and collision overhead during creation.
5. **Redis Cache Cluster:** In-memory lookups utilizing an LRU eviction policy to serve >95% of read traffic.
6. **Distributed NoSQL Database:** Sharded storage layer (e.g., Cassandra / DynamoDB) ensuring durable persistence.
7. **Async Analytics Pipeline:** Decoupled Kafka click event streams feeding real-time metric processors and OLAP data lakes.

---

## 4. API Design & Capacity Estimations (`Api and estimation.png`)

### RESTful API Contracts
* **Create Short URL**
  * `POST /api/v1/urls`
  * **Request Body:** `long_url`, `custom_alias`, `expires_at`
  * **Response (201 Created):** Returns `short_code`, `short_url`, and creation timestamps.
* **Redirect to Original URL**
  * `GET /{short_code}`
  * **Response (302 Found):** Redirects via `Location` header to the target `long_url` (Fallback: `404 Not Found` if expired).
* **Retrieve Click Analytics**
  * `GET /api/v1/urls/{short_code}/analytics`
  * **Response (200 OK):** Returns `total_clicks`, `unique_visitors`, and geographic breakdown.

### Back-of-the-Envelope Scale Estimations
* **Traffic Throughput:** ~200 writes/sec (Peak 400 RPS); ~19,290 reads/sec (Peak ~38,580 RPS at 100:1 read-write ratio).
* **5-Year Storage Capacity:** 30 billion total records requiring ~15 TB raw storage (~45 TB cluster storage with a 3x NoSQL replication factor).
* **Redis Cache Memory Sizing:** ~1.73 billion daily reads translating to ~346 million hot records (20%), requiring ~173 GB of RAM for high-hit redirection caching.
