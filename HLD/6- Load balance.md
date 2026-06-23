### Step 6: Load Balancers

A **Load Balancer** functions as a highly available network traffic controller positioned upstream from your application tier. Its primary mandate is to intercept incoming client requests and distribute them dynamically across healthy backend server nodes to maximize system throughput, optimize response latency, and eliminate resource bottlenecks.

---

#### 1. Load Balancing Algorithms (Stateless & Stateful)

The routing choices made by a load balancer are governed by programmatic algorithms. These methodologies are divided into two fundamental operational patterns:

##### A. Stateless Algorithms

Stateless routing strategies make distribution choices dynamically based entirely on immediate server availability or deterministic mathematical sequences, requiring no retention of persistent client session history.

- **Round Robin:** Cycles through the pool of available backend servers in a sequential, repeating chain (Server A ──► Server B ──► Server C ──► Server A). This assumes all target servers possess identical performance metrics.
- **Weighted Round Robin:** Formulates a capacity assignment where individual host machines are given a specific "weight" integer based on their CPU, RAM, or core capacity. High-spec servers take a larger proportion of sequential requests.
- **Least Connections:** Monitors active connections on every node and routes the incoming packet to the machine experiencing the lightest concurrent volume. This is ideal for execution profiles with highly variable request lifecycles.

##### B. Stateful Algorithms

Stateful routing mechanisms evaluate and track historical client data attributes, anchoring a specific user session securely to a designated backend node.

- **IP Hash / Sticky Sessions:** Computes a mathematical hash from the client's source IP address or session cookie ID. This persistent hash output uniquely points the user to a dedicated application server.
  - _Use Case:_ Crucial for legacy backend applications that cache session context (such as temporary in-memory shopping carts) locally inside server RAM rather than an external stateless database.

---

#### 2. Consistent Hashing

In highly elastic distributed systems, traditional modular hashing algorithms introduce massive scaling vulnerabilities. Consider standard hashing:

$$Server\_Index = Hash(Key) \pmod n$$

Where $n$ represents the total count of operational servers. If the system scales up by adding a new node, or scales down because a machine crashes, $n$ changes. This instantly shifts the mathematical modulo remainder for nearly every single key in the ecosystem. Caches are invalidated instantly, resulting in a catastrophic **Cache Stampede** that can crash down core databases.

**Consistent Hashing** resolves this architectural crisis through a circular token layout:

- **The Hash Ring:** Both the unique physical identifiers of the backend servers and the incoming data keys are hashed using the exact same cryptographic function mapping across a contiguous mathematical circle (a 360-degree range from $0$ to $2^{32}-1$).
- **The Routing Law:** To locate the target destination for a specific data key, you map the key's hash point onto the circle, then scan clockwise along the perimeter until you intercept the very first active server node.
- **The Scaling Impact:** When a database or caching server is added or dropped from the topology, only a minute portion of keys—mathematically averaging $\frac{1}{n}$—require redistribution or remapping. The remaining database nodes face zero state corruption or cache eviction overhead.

---

#### 3. Proxy vs. Reverse Proxy

A proxy is a proxying network intermediary node positioned explicitly between a source client network interface and a target destination web server. Its classification changes completely depending on which entity it protects:

##### A. Forward Proxy (The Client Shield)

A Forward Proxy sits directly in front of a private client machine pool, intercepting outbound requests destined for the wider public internet.

- _Architectural Directives:_ Masks internal client IP identities, enforces corporate access whitelist/blacklist firewalls, and filters out malicious external traffic.

##### B. Reverse Proxy (The Server Shield)

A Reverse Proxy sits in front of your internal server farm, intercepting inbound public web traffic and carefully routing it downstream across an isolated internal subnetwork.

- _Architectural Directives:_ **This is the structural framework of a Load Balancer.** It prevents direct exposure of backend infrastructure to the wild internet, manages SSL/TLS certificate handshakes (**SSL Termination**), strips away heavy encryption overhead from app servers, and handles static payload gzip compression.

---

#### 4. Rate Limiting

To prevent system starvation, denial-of-service degradation, or crash faults, architectures deploy **Rate Limiting** filters to restrict the total volume of requests an API endpoint or IP address can submit within a rolling window.

- _Defensive Value:_ Eradicates Distributed Denial of Service (DDoS) exploitation, blocks automated web-scraping bots, and minimizes brute-force authentication cracking attempts.

##### Core Algorithmic Frameworks

- **Token Bucket:** A logic bucket maintains a finite token capacity limit. Every incoming HTTP request consumes exactly one token to pass through. Tokens regenerate at a fixed, periodic time interval. If the bucket runs dry of tokens, incoming requests face immediate standard `429 Too Many Requests` execution errors.
  - _Operational Characteristic:_ Permits controlled, temporary bursts of traffic if the bucket has been resting at full token capacity.
- **Leaky Bucket:** Incoming client requests enter a linear data queue structured like a bucket with a small hole in the base. Traffic leaks out of the bottom to be processed at a strict, continuous, uniform rate. If a massive traffic spike occurs and floods the buffer, the queue overflows, and any excess requests are instantly dropped.
  - _Operational Characteristic:_ Enforces a highly predictable, perfectly smooth output traffic rate to protect fragile legacy backend components.
