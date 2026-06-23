# System Design Essentials: Caching, Write Policies, Eviction, and CDNs

A comprehensive guide to understanding distributed caching mechanisms, data persistence policies, memory optimization strategies, and edge delivery systems for scalable infrastructure.

---

## 1. What is a Cache? (Redis vs. Memcached)

A **cache** is a high-speed data storage layer that stores a subset of data, typically transient, so that future requests for that data are served much faster than fetching it from its primary storage location (such as a relational database or spinning hard drives).

Caches trade memory capacity for speed: they utilize fast, expensive **RAM** instead of slower, more economical solid-state drives (SSDs) or hard disk drives (HDDs).

```
   [ Client / Application ]
         /           \
    (1) Read       (2) Cache Miss:
       /              Fetch from DB & Update Cache
      v                v
 [ Cache (RAM) ] --> [ Primary Database (SSD/HDD) ]
```

### Redis vs. Memcached

While both are popular open-source, in-memory distributed data stores, they serve different architecture design requirements:

| Feature              | Memcached                                                                              | Redis                                                                                                                                                                                                 |
| :------------------- | :------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Data Types**       | Simple Key-Value pairs (Strings only).                                                 | Rich data structures (Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLogs).                                                                                                              |
| **Persistence**      | Purely in-memory. If the instance crashes or restarts, all data is volatile and lost.  | Offers optional persistence. Can snapshot data to disk periodically (RDB) or append every modification operation to a log (AOF).                                                                      |
| **Architecture**     | **Multithreaded**: Scales linearly across multi-core CPU architectures out-of-the-box. | **Historically single-threaded**: Uses an event loop to handle requests sequentially, avoiding mutex lock overhead (modern versions utilize background threads for specific tasks like unlinking/IO). |
| **Replication & HA** | No native replication or high availability (handled at the client level via hashing).  | Native replication, clustering, and automatic failover via **Redis Sentinel** or Redis Cluster.                                                                                                       |
| **Primary Use Case** | Quick, straightforward caching for basic string data or pre-rendered HTML fragments.   | Advanced caching, message brokering (**Pub/Sub**, streams), priority queues, distributed locks, and state/session management.                                                                         |

---

## 2. Write Policies (Write-Through, Write-Back, & Write-Around)

When an application modifies data, it must reconcile updates across both the volatile cache layer and the persistent primary database. Write policies dictate how and when data is written to the system.

```
Write-Through:   App ──> [ Cache ] ──(Synchronous)──> [ Database ] ──> Success
Write-Back:      App ──> [ Cache ] ──> Success ──(Asynchronous / Batch)──> [ Database ]
Write-Around:    App ────────────────────────────────> [ Database ]
```

### A. Write-Through

The application writes data to the cache, and the cache **immediately and synchronously** writes it to the database. The system only confirms a successful write operation to the client _after_ both writes complete.

- **Pros:** High data consistency across layers. If the cache layer crashes, data integrity is perfectly maintained in the backing database.
- **Cons:** Higher write latency because every write operation is bounded by the speed of the slower database write.

### B. Write-Back (Write-Behind)

The application writes data directly to the cache, which immediately acknowledges success to the client. The cache then flushes the modified data to the primary database **asynchronously** (either via background workers or periodic batch runs).

- **Pros:** Extremely fast write performance and high throughput. Excellent choice for write-heavy applications (e.g., IoT sensor telemetry, real-time gaming leaderboards).
- **Cons:** High risk of data loss. If the cache tier experiences power loss, hardware failure, or crashes before syncing updates downstream, that data is permanently lost.

### C. Write-Around

The application writes data directly to the primary database, completely **bypassing** the cache. The cache is only updated subsequently when a client attempts to read that specific key, experiences a **cache miss**, and lazy-loads the data into the cache from the database.

- **Pros:** Prevents cache pollution. Highly efficient when written data is infrequently or never read again, preserving valuable RAM for high-demand keys.
- **Cons:** Temporal latency spike; the initial read request for recently written data always results in a cache miss, making the first lookup slow.

---

## 3. Cache Replacement Policies (Eviction Policies)

Because RAM is finite and expensive, caches must operate with an upper bound on memory capacity. When a cache reaches its threshold and a new item needs insertion, a **Replacement Policy** determines which existing element is evicted.

```
[ New Item In ] ──> [ Full Cache Layer ] ──> Eviction Policy Decisions ──> [ Expelled Item Out ]
```

### A. LRU (Least Recently Used)

Discards items that haven't been accessed for the longest duration of time. It relies on the principle of _temporal locality_: if an item hasn't been requested recently, it is statistically less likely to be requested in the immediate future.

### B. LFU (Least Frequently Used)

Tracks access metrics via an internal counter for each item. Keys with the lowest frequency score are selected for eviction first, regardless of when they were last accessed. This is highly effective for identifying and preserving consistently popular "hot keys".

### C. Segmented LRU (SLRU)

An advanced, multi-tiered variation of LRU designed to overcome the "one-hit wonder" problem (where a sudden burst of non-repeating data flushes out hot keys). It partitions available cache space into two specific segments:

1.  **Probationary Segment:** Where newly requested items are initially allocated.
2.  **Protected Segment:** Where items are promoted once they are requested a **second** time while residing in the probationary section.

Items are exclusively evicted from the tail of the _Probationary Segment_. This safeguards highly popular keys in the protected zone from being inadvertently purged by a sudden influx of single-use data bursts.

### Other Notable Policies:

- **FIFO (First In, First Out):** Evicts elements in the exact chronological order they arrived in the cache, ignoring access frequency or recency.
- **TTL (Time-To-Live):** Associating an absolute expiration timestamp with keys. Once the countdown timer hits zero, the key is automatically marked as expired and scheduled for deletion or lazy-purged during read operations.

---

## 4. Content Delivery Networks (CDNs)

A **Content Delivery Network (CDN)** is a geographically distributed infrastructure of caching servers—frequently referred to as **Edge Servers** or Point of Presence (PoP) locations—that cooperate to deliver internet assets with ultra-low latency.

Instead of routing all user requests back to a singular, centralized application server (the Origin Server), a CDN caches static resources (images, video files, stylesheet frameworks, client-side scripts) and certain dynamic API payloads close to where users reside globally.

```
[ User in Tokyo ] ──(Short Distance)──> [ CDN Tokyo Edge Server (Cache Hit) ]
                                                    | (Cache Miss Only)
                                                    v
                                        [ Origin Server in Virginia, US ]
```

### Core Architecture Workflow

If your main infrastructure stack is physically situated in Virginia, US, and a consumer in Tokyo requests a profile picture asset:

1. The request is georouted to the closest physical PoP edge server in Tokyo.
2. If the asset is present on the edge server (**Cache Hit**), it is delivered instantly over a short physical distance.
3. If it is missing (**Cache Miss**), the Edge server retrieves the item from the Virginia Origin server, returns it to the user, and caches it locally for subsequent regional requests.

### Key Operational Benefits

- **Drastic Latency Reduction:** Minimizes round-trip time (RTT) by decreasing the physical distance data must travel across global fiber-optic networks.
- **Origin Offloading:** Significantly mitigates bandwidth consumption, ingress/egress charges, and computational strain on your core database/application instances.
- **High Availability & Security:** Absorbs massive structural traffic spikes seamlessly, distributing loads and mitigating high-velocity Distributed Denial of Service (**DDoS**) attacks at the network edge.
