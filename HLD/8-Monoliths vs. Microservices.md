Step 8: Monoliths vs Microservices and Step 9: Monitoring & Logging from your notes.

Step 8: Monoliths vs Microservices
This step focuses on how applications are structurally organized. A Monolith bundles all software components (auth, payments, products, notifications) into a single, massive codebase and deployment unit. Microservices break that large application down into a collection of small, independent, specialized services that communicate over a network (usually via HTTP REST or message queues).

1. Why Microservices?
   Companies migrate to microservices to solve the scaling bottlenecks of a monolithic codebase:

Independent Scalability: If your video streaming feature gets a massive traffic spike but your billing page is quiet, you can scale up just the video service rather than replicating the entire massive monolith.

Independent Deployments: Teams can update and push code for their specific service without waiting for other teams or risking a crash across the entire application.

Technology Flexibility: Different services can use different technology stacks (e.g., a Python service for Machine Learning and a Node.js service for real-time chat).

2. Concept of 'Single Point of Failure' (SPOF)
   A Single Point of Failure is any part of a system that, if it fails, will stop the entire system from working.

In a monolithic architecture, the deployment itself is often a SPOF. If a single line of memory-leaking code crashes the billing module, the entire website or app goes offline.

Microservices aim to eliminate SPOFs by isolating domains. If the billing service crashes, users can still browse products, add items to their carts, and read reviews.

3. Avoiding Cascading Failures
   In a microservices ecosystem, services rely on one another. If Service A calls Service B, and Service B slows down or crashes, Service A can run out of threads waiting for a response, causing Service A to crash too. This domino effect is called a cascading failure.

To avoid this, engineers use specific design patterns:

Circuit Breaker Pattern: If a downstream service fails repeatedly, the "circuit breaker" trips open. Future requests immediately fail-fast with a fallback response rather than hanging and draining system resources.

Bulkheads: Isolating resources (like thread pools) so that a failure in one isolated module cannot spill over and starve resources in another.

4. Containerization (Docker)
   Because microservices break a system into dozens of moving parts, managing different dependencies, language versions, and environments becomes a nightmare. Containerization solves this.

Docker: Package up a microservice's code, runtime, system tools, and libraries into a single lightweight container.

Benefits: It guarantees consistency. If the container runs correctly on a developer's laptop, it will run exactly the same way in production, regardless of the underlying OS.

5. Migrating to Microservices
   You rarely build a microservices system entirely from scratch on day one; you usually migrate an existing monolith over time.

Strangler Fig Pattern: The most common migration strategy. Instead of a high-risk "rewrite everything from scratch," you slowly carve off small features from the monolith one by one and replace them with microservices. Eventually, the monolith shrinks down to nothing, "strangled" by the new microservices architecture.

Phase 1: Monolith Only Phase 2: Carving Off Phase 3: Fully Strangled
┌─────────────────────┐ ┌─────────────────────┐ ┌─────────────────────┐
│ │ │ MONOLITH │ │ ~~OLD MONOLITH~~ │
│ MONOLITH │ │ ┌─────────────────┐ │ │ (Completely Gone) │
│ │ │ │ ~~Auth Domain~~ │ │ └─────────────────────┘
│ (All Domains Inside)│ │ └────────┬────────┘ │ ┌─────────────────────┐
└─────────────────────┘ └──────────┼──────────┘ │ NEW MICROSERVICES │
▼ │ ┌──────┐ ┌──────┐ │
┌─────────────────────┐ │ │ Auth │ │Billing│ │
│ New Auth Service │ │ └──────┘ └──────┘ │
└─────────────────────┘ └─────────────────────┘

- **The Execution:** You construct a routing intermediary (like an API Gateway) in front of the application. One by one, you cut a specific domain out of the legacy monolith, rewrite it as an isolated microservice, and update the gateway to route traffic to the new service.
- Over time, the monolith is progressively chipped away and shrunk down until it is entirely replaced—or "strangled"—by the new microservices architecture.

---

### Step 9: Monitoring & Logging

Operating a highly distributed system means you cannot easily peek inside a single server file log to find an error. Real-time observability across the entire system infrastructure is crucial.

#### 1. Centralized Logging vs. Metrics

- **Centralized Logging (The "What"):** Individual microservices continuously emit timestamped text logs of system events. A specialized log collection pipeline (such as the ELK Stack: Elasticsearch, Logstash, Kibana) gathers these distributed text streams from every running container and indexes them into a single searchable dashboard.
  - _Purpose:_ Pinpointing the exact root cause of an application error or trace execution logs for a specific transaction ID.
- **Metrics (The "How Much"):** Quantitative numerical data captured over specific time intervals monitoring infrastructure vitals (such as CPU consumption %, available RAM bytes, request-per-second volume, and API latency rates). These are usually stored in specialized Time-Series Databases (e.g., Prometheus) and visualized using dashboards like Grafana.
  - _Purpose:_ Real-time system monitoring, load evaluation, and alerting rules.

---

#### 2. Anomaly Detection & Alerting

A system shouldn't wait for a user to report a bug before developers realize something is broken.

- **Automated Alerting:** Setting up automated guardrails that actively scan metrics. If a critical metric passes a threshold (e.g., your API error rates cross `2%` or disk space reaches `90%`), the monitoring system instantly triggers high-priority alerts to on-call engineers via platforms like PagerDuty or Slack.
- **Anomaly Detection:** Advanced monitoring setups establish a rolling baseline of standard system behaviors using historical data. If a metric acts highly unusual—such as a sudden, sharp drop in checkout transaction volume that doesn't trigger standard hardware errors—the anomaly detection controller immediately flags it as a potential silent outage.
