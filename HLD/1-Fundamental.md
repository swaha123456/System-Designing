# 🌐 System Design Guide (2025–26)

## 🏗️ Chapter: High-Level Design (HLD)

Welcome to the fundamentals of High-Level Design. This section covers the macro-building blocks required to architect large-scale, highly distributed systems.

---

### Step 1: Fundamentals

#### 1. Serverless vs. Serverful

Every application needs a computer (a server) to run its code. The core difference lies in how much control—and operational management—you maintain over that infrastructure.

- **Serverful (Traditional):** You lease or provision specific server instances. You are entirely responsible for configuring the operating system, managing security patches, keeping it running 24/7, and paying for it regardless of user traffic.
  > 💡 **Analogy:** Renting a permanent apartment. You pay the full lease every single month whether you are sleeping there or away on a long vacation.
- **Serverless:** Physical servers are still used, but the cloud provider (e.g., AWS, GCP) abstracts them completely. Your code runs strictly on-demand, triggered by specific events (like an API hit). The infrastructure scales down to zero instantly when execution finishes. You only pay for the exact milliseconds your code runs.
  > 💡 **Analogy:** Staying in a hotel room or calling an Uber. You only pay for the exact duration of your trip or stay.

---

#### 2. Horizontal vs. Vertical Scaling

When your web application gains massive traction, a single baseline server will inevitably face resource exhaustion. You have two primary dimensions to scale your capacity:

- **Vertical Scaling (Scaling UP):** Enhancing your existing server by upgrading its hardware specifications—adding faster CPUs, more RAM, or higher-capacity SSDs.
  - _The Bottleneck:_ There is a strict physical threshold to how powerful a single computer can become. Furthermore, it introduces a **Single Point of Failure (SPOF)**; if that machine crashes, your entire application goes offline.
  - _Analogy:_ Upgrading a standard delivery truck to a massive monster truck.
- **Horizontal Scaling (Scaling OUT):** Adding multiple identical, smaller servers to a resource pool and distributing incoming network traffic dynamically across them using a Load Balancer.
  - _The Benefit:_ Scale becomes virtually limitless. If a single node experiences hardware failure, adjacent nodes seamlessly handle the remaining load, offering high availability.
  - _Analogy:_ Instead of buying one massive monster truck, deploying an entire fleet of 50 agile delivery vans.

---

#### 3. What are Threads?

To conceptualize concurrency, we must first define a **Process**. When you launch an application (like Google Chrome), the operating system initializes a Process, providing it with an isolated, secure boundary of memory allocation.

- A **Thread** represents a single, lightweight path of execution running _inside_ that parent process.
- Multi-threaded processes can spawn dozens of threads to handle independent execution tasks simultaneously while sharing the identical memory sandbox.
  > 👨‍🍳 **Analogy:** Think of the **Process** as an industrial restaurant kitchen. The **Threads** are the individual line chefs working inside that space. They all utilize the same physical counters, ingredients, and utensils (shared memory) to prepare separate dishes concurrently.

---

#### 4. What are Pages?

Computing architectures balance two primary memory components: System RAM (extremely fast, high cost, volatile, low capacity) and Solid State Drives/Hard Drives (slower, low cost, non-volatile, massive capacity).

- A **Page** is a fixed-length contiguous block of virtual memory (conventionally sized at 4 KB).
- To optimize RAM utilization, the operating system avoids loading a massive multi-gigabyte application executable into memory all at once. Instead, it segments the program binary into tiny **Pages**.
- The OS loads only the active pages required for immediate computation into RAM. If a running thread requests data residing on a page still sitting on the slow disk drive, it triggers a **Page Fault**, prompting the OS kernel to rapidly swap that page into active RAM memory.

---

#### 5. How Does the Internet Work?

When a client inputs `https://www.youtube.com` into a web browser address bar and hits execute, an intricate multi-stage lifecycle transpires within milliseconds:

[Browser] ──(1. DNS Lookup)──> [DNS Server (Finds IP Address)]
│
├──(2. TCP/IP Handshake)──> [Target Web Server]
│
├──(3. HTTP GET Request)──> [Server Processes Request]
│
└──(4. Data Packets Return)─ [HTML/CSS/JS Painted on Screen]

1.  **The Directory Lookup (DNS):** Machine nodes communicate natively using numerical coordinates called **IP Addresses** (e.g., `142.250.190.46`). The browser queries a **Domain Name System (DNS)** server, which translates human-readable domain text into a machine-routable IP address.
2.  **The Transport Handshake (TCP/IP):** The browser initiates a reliable, packet-verified network connection with the destination server via a 3-way handshake, guaranteeing data packet integrity.
3.  **The Application Protocol Request (HTTP):** The client transmits a structured HTTP request payload (e.g., `GET /index.html`) instructing the server on what digital asset it wishes to retrieve.
4.  **The Application Response:** The destination backend parses the header instructions, processes data via its application tier, and transmits back fragmented packets containing payload data (HTML, CSS, JavaScript, and asset buffers). The client-side browser reads these sequential data streams and renders the visual interface on screen.
