### Step 11: System Design Tradeoffs

In the field of System Design, **there is no such thing as a perfect architecture**. Senior engineers do not look for the "best" technology; they look for the correct set of compromises. Every architectural pattern you select is a deliberate balance of engineering tradeoffs—gaining one structural benefit while willingly sacrificing another.

---

#### 1. Push vs. Pull Architecture

This trade-off governs the direction and timing of how data objects traverse the network from a source producer to a destination consumer (such as a backend service updating a user interface).

- **Push Architecture:** The server maintains persistent network pipes and actively broadcasts data updates down to the client tier the exact millisecond an event occurs.
  - _Core Protocols:_ WebSockets, Server-Sent Events (SSE).
  - _The Compromise:_ Real-time, sub-millisecond updates, but forces the backend to manage millions of concurrent, open, memory-heavy socket connections.
- **Pull Architecture:** The client tier acts as the initiator, periodically polling the server at structured intervals (e.g., every 10 seconds) asking: _"Is there any new data payload available?"_
  - _Core Protocols:_ HTTP Short/Long Polling.
  - _The Compromise:_ Extremely simple to build and completely stateless, but introduces artificial latency and wastes massive server bandwidth processing empty requests when nothing has changed.

---

#### 2. Consistency vs. Availability (The CAP Theorem)

As mapped out in Step 3, when a physical network partition event (**P**) inevitably occurs within a distributed cluster, you must select one of two design paths:

- **Consistency (CP):** The cluster locks down partitioned nodes to guarantee every operational node returns the exact same, most recent write state. If a node cannot verify it has the freshest data, it returns an explicit error.
- **Availability (AP):** Every active node returns a valid response immediately without delay, even if it means serving slightly stale data to the user.
  - _The Dilemma:_ You must choose between forcing a banking application to throw an error screen during a minor network partition to preserve financial audit trail accuracy (**Consistency**), or allowing a social media feed to load instantly even if it temporarily displays a slightly outdated like or comment count (**Availability**).

---

#### 3. SQL vs. NoSQL Databases

Determining how data schemas are structured and committed to disk maps directly to how easily your application can scale horizontally under load.

- **SQL (Relational):** Data is organized into strict, tabular schemas and supports highly complex multi-table `JOIN` statements. It scales natively via **Vertical Scaling** (provisioning a larger, more expensive server box).
- **NoSQL (Non-Relational):** Data is written using dynamic, fluid structures (JSON files, Key-Value schemas). It scales horizontally by clustering hundreds of cheaper commodity servers together.
  - _The Decision Matrix:_ Select **SQL** if absolute data integrity, ACID transactional rules, and complex multi-row relationships are critical requirements (e.g., double-entry bookkeeping ledgers). Select **NoSQL** if your system requires ultra-high write throughput, massive horizontal elasticity, and manages unstructured datasets (e.g., live chat histories, telemetry tracking logs, or real-time gaming sessions).

---

#### 4. Memory vs. Latency

To make a high-traffic web application respond instantaneously (ultra-low latency), system designs rely on shifting data closer to the compute tier by pre-calculating and storing records inside volatile, high-cost system RAM.

- _The Compromise:_ Optimizing heavily for ultra-low **Latency** forces you to consume massive quantities of expensive, distributed **Memory** caching layers, which exponentially drives up cloud infrastructure costs. Conversely, if you optimize to save budget by using minimal RAM memory, your compute tier must continuously read blocks from persistent, slow disk drives, drastically increasing your application latency metrics.

---

#### 5. Throughput vs. Latency

Understanding the difference between execution volume and execution speed is fundamental to tuning infrastructure:

- **Throughput:** The absolute volume of operations or tasks a system can process within a fixed time window (e.g., handling `50,000` API requests per minute).
- **Latency:** The round-trip time it takes to complete one single, individual request from start to finish (e.g., responding in `20` milliseconds).
  - _The Compromise (Batching):_ To maximize system **Throughput**, systems gather incoming operations into memory arrays (**Batching**) and process them all together in a single bulk database command. While batching handles far more total data volume overall, it actively degrades individual **Latency**; the absolute first item inserted into the batch queue must stall and wait for the remaining batch buffer to fill completely before it can finish processing.

---

#### 6. Accuracy vs. Latency

In massive, big-data architectures (such as calculating real-time globally trending hashtags across millions of concurrent streaming tweets), counting every single raw row mathematically exact in real time is computationally impossible without grinding the system to a halt.

- _The Compromise (Probabilistic Data Structures):_ To keep system **Latency** exceptionally low, engineers trade away absolute mathematical **Accuracy** by utilizing advanced probabilistic data structures, such as **HyperLogLog** (for cardinality estimation) or **Bloom Filters** (for set membership checking).
- These specialized algorithms estimate a highly accurate approximation (e.g., `99.2%` correct) within microseconds using negligible memory, bypassing the massive, high-latency database overhead of scanning billions of raw data logs.
