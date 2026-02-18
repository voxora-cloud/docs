## Voxora Engineering & Contribution Guidelines

This document explains the architectural rules, scaling principles, and safe modification patterns for Voxora.
It is intended for both human contributors and AI coding assistants working on the repository.

---

## 🚨 Core Architectural Rule

**Voxora must remain horizontally scalable and stateless.**

No feature should assume:

* single server deployment
* local filesystem storage
* in-memory session persistence
* direct socket-to-process coupling

All persistent or shared state MUST use external services.

---

## 🧠 System Overview

Voxora is a real-time AI communication infrastructure designed for:

* self-hosted deployments
* independent organization installations
* websocket-heavy workloads
* async AI processing
* horizontal container scaling

Primary components:

* API Server (HTTP + WebSocket handling)
* Worker Service (queue processing / AI tasks)
* Redis (pub/sub + queue backend)
* MongoDB (persistent storage)
* Reverse Proxy / Load Balancer

---

## ⚠️ Critical Engineering Constraints

### 1. Never store shared runtime state in memory

DO NOT implement:

* in-memory chat sessions
* in-memory rate limits
* in-memory user presence tracking

Always use Redis.

---

### 2. WebSocket handlers must remain non-blocking

Never call external AI providers directly inside:

* socket connection handlers
* message event handlers
* HTTP request lifecycle for chat

Instead:

```
enqueue job → worker processes → result returned async
```

Blocking calls will freeze event loops and collapse connection scalability.

---

### 3. All services must support external endpoints

Hardcoding localhost is forbidden.

Always use environment variables:

```
MONGO_URL
REDIS_URL
AI_SERVICE_URL
```

This ensures Voxora works in:

* single-server installs
* split-server enterprise deployments
* Kubernetes environments

---

### 4. Containers must remain independently runnable

Each service must:

* start independently
* not rely on local container ordering assumptions
* tolerate delayed database availability
* retry external connections

---

## 🧩 Scaling Model

Voxora scales by:

* increasing API container count
* sharing socket state via Redis adapter
* distributing AI work through queue workers

The system must behave identically whether:

```
1 API container
or
20 API containers
```

Any feature that breaks this rule is invalid.

---

## 🛑 Forbidden Implementation Patterns

Contributors MUST NOT introduce:

* synchronous AI calls in request path
* file-based message storage
* process-local websocket routing tables
* singleton global state for user sessions
* blocking CPU-heavy work in API server

These patterns destroy production scalability.

---

## ✅ Approved Patterns

Use:

* Redis for ephemeral shared state
* MongoDB for persistence
* queue workers for heavy processing
* async message flows for AI responses
* stateless API design

---

## 🧪 Testing Requirements for Infrastructure Changes

If modifying:

* WebSocket logic
* Redis usage
* worker queues
* connection handling

You must ensure:

* multiple API containers can run simultaneously
* messages propagate correctly across containers
* worker queues process reliably under retry

---

## 🧭 Deployment Philosophy

Voxora officially supports:

* single-server Docker Compose installations

Voxora is designed to be compatible with:

* multi-server deployments
* managed cloud databases
* Kubernetes environments

The project does NOT enforce any specific infrastructure topology.

---

## 🧱 Contributor Mindset

When implementing a feature, always ask:

> “Would this still work if Voxora was running across multiple machines?”

If the answer is unclear, the implementation is unsafe.

---

## 🤖 Guidance for AI Coding Assistants

When modifying this repository:

* prefer stateless implementations
* avoid adding new runtime dependencies unless necessary
* do not introduce synchronous external calls in API handlers
* never assume only one instance of a service exists
* preserve environment-based configuration patterns

If unsure, choose the design that supports horizontal scaling.

---

## 🎯 Project Goal

Voxora aims to be:

* easy to install for small teams
* safe to scale for enterprises
* production-ready by default
* infrastructure-neutral

All contributions should move the system toward these goals.
