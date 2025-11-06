---
title: Full Systems Archetecture
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
# Actualize Earth – Backend & Network Architecture (v1)

*Last updated: Oct 13, 2025 (us‑east‑1)*

## 0. Intent

A concise, implementation‑ready description of the **as‑is** infrastructure for the mobile app backend, plus **surgical upgrades** that improve reliability, scalability, security, and ops cost without over‑engineering.

***

## 1. High‑Level Overview (as‑is)

* **Edge**: Cloudflare for DNS, WAF, CDN.
* **TLS**: Client → Cloudflare (TLS) → **ALB (TLS termination)**.
* **Region/VPC**: Single VPC in **us‑east‑1**.
* **Compute**: EC2 Auto Scaling Group using AMIs on **t3.medium**; stateless app tier.
* **Services** (on EC2):

  * `api` → PHP/FPM REST endpoints.
  * `api2` → Node endpoints for async‑heavy operations; exposed at **api2.actualize.earth** (reverse‑proxied).
  * `ws` → Socket.IO server; exposed at **ws.actualize.earth** (reverse‑proxied).
* **Workers (private)**: `cron.js`, `jobs.js`, `notifier.js` under pm2 on private instances (no public ingress).
* **Data**: MongoDB Atlas (details TBD); **S3** for static assets.

### Diagram A — Request Flow (Edge → Services)

```
Client ─▶ Cloudflare (DNS/WAF/CDN)
        └── TLS ─▶ AWS ALB (443)
                    ├─ Host=api.actualize.earth  → TG: api-php (PHP/FPM)
                    ├─ Host=api2.actualize.earth → TG: api2-node (Node)
                    └─ Host=ws.actualize.earth   → TG: ws-socketio (WS upgrade)
```

### Diagram B — VPC Topology (conceptual)

```
VPC (us-east-1)
├── Public Subnets (AZ-a/b)
│   └─ ALB (HTTPS 443)
└── Private Subnets (AZ-a/b)
    ├─ ASG: App Nodes (t3.medium; api/api2/ws)
    ├─ Worker Nodes (pm2: cron/jobs/notifier)
    ├─ (Future) ElastiCache Redis (pub/sub; sessions)
    └─ NAT GW (egress to Atlas/S3/3rd-party)
```

### Diagram C — Realtime + Jobs (near‑term target)

```
Socket.IO Nodes ── adapter ──▶ Redis (pub/sub) ──▶ cross-node rooms & presence
Workers (jobs/notifier/cron) ─▶ MongoDB Atlas ─▶ Email/SMS/Push + S3
```

***

## 2. Networking & Security

* **Cloudflare**

  * DNS, WAF rules for `/api/*` JSON; basic rate limits on auth/abuse‑prone routes.
  * Caching static and "safe" semi‑static responses (configurable per path).
* **ALB**

  * HTTPS listener (443), host‑based routing to three target groups: `api`, `api2`, `ws`.
  * WebSocket upgrades enabled for `ws` target group.
* **TLS**

  * End‑to‑end TLS; termination at ALB. Prefer ACM certs on ALB; Cloudflare set to **Full (strict)**.
* **Security Groups**

  * ALB: 0.0.0.0/0:443 inbound → App TGs.
  * App/Worker nodes: inbound from ALB SG; egress to Atlas, S3, SES/SNS/APNs/FCM.
* **IAM & Secrets**

  * App/Workers use IAM roles; secrets from AWS Secrets Manager / SSM Parameter Store.

***

## 3. Compute & Services

* **EC2/ASG**

  * AMI‑based immutable instances; stateless services (sessions not stored locally).
  * `t3.medium` today; monitor CPU credits & network PPS.
* **Service separation**

  * `api` (PHP/FPM) for core REST.
  * `api2` (Node) for async‑heavy endpoints.
  * `ws` (Socket.IO) for realtime.
* **Reverse Proxy**

  * Lighttpd/nginx on instance; host‑based route to local processes/ports.

***

## 4. Background Workers

* **cron.js**: time‑based tasks (pm2).
* **jobs.js**: queue processor. Mongo collection for scheduled/active/complete; currently polling.
* **notifier.js**: email/push/SMS dispatch; Mongo‑polled queue.
* **Access**: workers in private subnets; no public ingress.

### Reliability controls to add (minimal effort)

1. **Distributed lock** via Mongo `findOneAndUpdate` for cron ticks (prevent duplicates during scale/restarts).
2. **TTL indexes** on completed/failed jobs to cap growth; `attempts`, `nextRunAt` for backoff.
3. Prefer **Mongo Change Streams** over polling when feasible (less I/O, lower latency variance).
4. (Later) Optional migration of queues to **SQS** + DLQ.

***

## 5. Realtime (Socket.IO)

* **Today**: single‑ASG behind ALB; no external adapter (risk for fan‑out across nodes).
* **Add**: **Redis adapter** for presence/rooms/broadcasts across nodes.
* **Health**: `/socketz` endpoint (returns connection counts, room metrics, adapter status).

***

## 6. Observability

* **Health**: `/healthz` (api), `/readyz` (api2), `/socketz` (ws).
* **Metrics (min set)**: request rate, p50/p95 latency, WS connections, queue depth/lag, job failure rate.
* **Logs**: JSON logs -> CloudWatch; redact PII; correlate with request IDs.
* **Alerts**:

  * p95 latency > budget for 5 min,
  * queue lag > N minutes,
  * WS conn drop > X% within 2 min,
  * 5xx rate spike.

***

## 7. CI/CD & AMI Pipeline (proposed minimal flow)

* **Build**: GitHub Actions → tests → Packer AMI build (includes PHP/FPM/Node + systemd/pm2).
* **Deploy**: Update Launch Template → ASG rolling update (minHealthy 90%, maxSurge 1).
* **Config/Secrets**: pulled at boot from SSM/Secrets Manager.
* **Smoke**: synthetic probe via Cloudflare → ALB → `/healthz` and representative API call.

***

## 8. Cost & Scaling Notes

* **t3.medium**: watch CPU credits; consider **split ASGs** (api/api2 vs ws) if WS spikes.
* **Redis (small ElastiCache)**: \~$15–30/mo; major reduction in WS operational complexity.
* **Mongo polling → Change Streams**: reduces Atlas read load & jitter; lowers cost at scale.
* **Cloudflare**: cache headers for static/semi‑static to reduce ALB+EC2 egress and compute.

***

## 9. Action Checklist (90‑minute sprintable)

1. **TLS strictness**: Set Cloudflare → Origin to **Full (strict)**; ensure ACM cert on ALB.
2. **Redis for WS**: provision tiny ElastiCache; add Socket.IO adapter; add `/socketz` health.
3. **Cron safety**: add Mongo lock doc; implement extend/release; add alert if lock contention.
4. **Queue hygiene**: TTL indexes; `attempts`+backoff; dead‑letter collection.
5. **Health & Alerts**: wire `/healthz`/`/readyz`/`/socketz`; create CloudWatch alarms for p95, 5xx, queue lag.
6. **CI/CD skeleton**: GitHub Actions → Packer AMI → ASG rolling; post‑deploy smoke test.

***

## 10. Appendices

### A) Hostnames & Target Groups

* `api.actualize.earth`  → TG: `api-php`
* `api2.actualize.earth` → TG: `api2-node`
* `ws.actualize.earth`   → TG: `ws-socketio`

### B) Suggested Health Endpoints

* `GET /healthz` → \{ status: "ok", build\_sha, uptime }
* `GET /readyz`  → checks DB/connectivity; returns 200 only when ready
* `GET /socketz` → \{ connections, rooms, adapter: "redis|none" }

### C) Minimal Code Sketches

**Socket.IO Redis adapter**

```js
import { createClient } from "redis";
import { createAdapter } from "@socket.io/redis-adapter";
import { Server } from "socket.io";

const io = new Server(httpServer, { cors: { origin: "*" } });
const pub = createClient({ url: process.env.REDIS_URL });
const sub = pub.duplicate();
await pub.connect(); await sub.connect();
io.adapter(createAdapter(pub, sub));
```

**Mongo distributed lock (cron safety)**

```js
const now = new Date();
const lock = await db.collection('locks').findOneAndUpdate(
  { _id: 'cron:daily', $or: [ { expiresAt: { $lt: now } }, { expiresAt: { $exists: false } } ] },
  { $set: { holder: process.env.INSTANCE_ID, expiresAt: new Date(Date.now() + 5*60*1000) } },
  { upsert: true, returnDocument: 'after' }
);
if (lock.value.holder !== process.env.INSTANCE_ID) return; // another node holds the lock
```

**ALB listener rules (concept)**

```
:443 HTTPS
  Host == api.actualize.earth  → TG api-php
  Host == api2.actualize.earth → TG api2-node
  Host == ws.actualize.earth   → TG ws-socketio (WS upgrade)
```

***

## 11. Roadmap (optional, when ready)

* Define Mongo Atlas topology (tier, multi‑AZ, PITR backups; connection pooling & retryable writes).
* Add synthetic monitoring from multiple geos (Cloudflare + Route 53 health checks optional).
* Document DR drill (Atlas regional failover; ASG warm pool; S3 restoration runbook).
* Explore portable/edge node path once core cloud is stable.

***

**End of v1**
