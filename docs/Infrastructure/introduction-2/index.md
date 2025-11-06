---
title: Introduction (Cloud)
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
## 🌐 System Architecture Overview

The Actualize infrastructure is designed for **resilience, scalability, and intelligent orchestration** — combining Cloudflare’s edge security with AWS’s elastic compute and MongoDB Atlas’s globally distributed database.\
Each layer is optimized for **high availability**, **fault tolerance**, and **low latency**, ensuring the system scales seamlessly from small workloads to global user bases.

***

### 1. Cloudflare Edge Network + Web Application Firewall (WAF)

All inbound traffic enters through **Cloudflare’s global edge**, spanning **310+ data centers** worldwide.

* **Security & Filtering:** Layer 3–7 DDoS mitigation capable of absorbing **100 Tbps+** attacks.
* **Web Application Firewall:** Adaptive protection from OWASP Top 10 vulnerabilities.
* **Edge Caching & Routing:** Reduces latency by up to **80%** through CDN caching and intelligent routing.

***

### 2. AWS Load Balancer Layer

The **AWS Application Load Balancer (ALB)** sits behind Cloudflare, distributing requests across multiple stateless backend servers.

* **Intelligent Routing:** Supports both HTTP and WebSocket connections.
* **Auto Health Checks:** Continuously probes endpoints, rerouting within seconds of failure.
* **TLS Termination:** Centralized SSL termination simplifies key management and offloads compute.
* **Horizontal Elasticity:** Handles **tens of thousands of concurrent connections**, seamlessly scaling via AWS Auto Scaling Groups.

***

### 3. Application Layer: Distributed Ubuntu 24.04 Servers

Core API and middleware services run on **Ubuntu 24.04 LTS** across **Amazon EC2 Auto Scaling Groups**, tuned for ultra-low-latency execution.

* **Core Stack:** PHP 8.3 APIs with embedded proxy modules (Search, ReadMe Connector, Socket Bridge).
* **Stateless Design:** Ensures quick failover and horizontal scalability.
* **Auto-Scaling Window:** 3 → 30 nodes dynamically adjusted by CPU, queue depth, or request latency.
* **Performance:** Each node sustains **2–5k req/s**, supporting **>100k req/s cluster-wide** bursts.

***

### 4. Proxy & Long-Running Microservices

To handle intensive or asynchronous workloads, the system routes jobs from the API layer to dedicated **proxy-based microservices**.\
These run in separate containers or long-running EC2 instances and communicate via internal message queues (RabbitMQ or Redis Streams).

#### Key Proxies & Services

* **Search Proxy:** Optimized for fast full-text and filtered queries, backed by indexed Mongo collections or ElasticSearch.
* **ReadMe Connector:** Handles outbound API documentation syncing, analytics, and schema introspection.
* **Socket Proxy:** Manages WebSocket connections and real-time event distribution across nodes.
* **Job Queue System:** Dedicated microservice for deferred or scheduled tasks (data syncs, report generation, etc.).
* **Notification Service:** Queues and dispatches multi-channel notifications (email, SMS, push, in-app), with rate limiting and batching.

#### Scalability & Fault Isolation

* Each service can autoscale independently (1 → 10+ instances) based on queue length or CPU utilization.
* If a microservice becomes unhealthy, tasks automatically re-queue and retry within **5–10 seconds**, ensuring delivery consistency.
* Average message throughput per service: **10–50k jobs/minute**, with durable delivery guarantees.

***

### 5. Distributed Storage: Amazon S3

* **Purpose:** Hosts static assets, images, and long-term backups.
* **Durability:** 99.999999999% (11 nines).
* **Replication:** Multi-region replication for data locality and disaster recovery.
* **Lifecycle Management:** Automatic archival to Glacier for cold storage optimization.

***

### 6. Data Layer: MongoDB Atlas

The primary database cluster is hosted on **MongoDB Atlas**, offering elasticity and geographic redundancy.

* **Multi-Region Replica Sets:** Ensures high read/write availability with automatic failover in \<10 seconds.
* **Autoscaling Tiers:** Scales vertically (M10 → M100) and horizontally via sharding.
* **Global Reads:** Queries automatically route to nearest regional nodes for \<50 ms response times.
* **Backup & Recovery:** Continuous backups with **4-hour recovery point objectives**.

***

### 7. Observability, Monitoring, and Reliability

* **Metrics Stack:** Cloudflare Analytics + AWS CloudWatch + MongoDB Atlas Monitoring.
* **Error Tracking:** Sentry or Elastic APM captures request-level telemetry.
* **Alerting:** Latency, error rates, and queue depths trigger automated Slack or PagerDuty alerts.
* **Uptime Target:** 99.99% across all layers.
* **Mean Time to Recovery (MTTR):** \<5 minutes via automatic instance recycling or blue-green deployment.

***

### 📈 Summary of Strengths

| Layer                          | Function                         | Resilience     | Scalability     | Notes                    |
| ------------------------------ | -------------------------------- | -------------- | --------------- | ------------------------ |
| **Cloudflare Edge + WAF**      | Global ingress, security         | 310 PoPs       | Global          | 100 Tbps DDoS protection |
| **AWS ALB**                    | Routing, SSL termination         | Multi-AZ       | Tens of k req/s | Auto health checks       |
| **App Servers (Ubuntu 24.04)** | API / middleware                 | 3 AZs          | 3–30 nodes      | Stateless, \<50 ms       |
| **Proxy Microservices**        | Background jobs, search, sockets | Multi-instance | 10–50k jobs/min | Fault-isolated, async    |
| **S3 Storage**                 | Assets & backups                 | 11 nines       | Unlimited       | Cross-region replication |
| **MongoDB Atlas**              | Main database                    | Replica sets   | Auto-tier       | Global latency \<50 ms   |

***

### 🚀 System Characteristics

* **Elastic:** Dynamically scales compute and data based on real-time demand.
* **Resilient:** Self-healing components and distributed microservices minimize single points of failure.
* **Performant:** Globally cached, edge-delivered assets with sub-100 ms API response targets.
* **Observable:** Unified telemetry for proactive detection and recovery.
