# CampusOS — System Architecture

## 1. Vision

CampusOS is a multi-tenant-ready (single-tenant for v1), AI-augmented ERP for colleges. It replaces fragmented spreadsheets/notice-boards with one system covering academics, hostel/mess, placements, complaints, events, communication, and AI-driven insights.

## 2. Architectural Style

**Modular Monolith → Service-Ready.**

We are NOT starting with microservices. Reasons:
- A solo/small dev team building microservices from day 1 burns 80% of effort on plumbing (service discovery, distributed tracing, inter-service auth) instead of features.
- A well-structured **modular monolith** (strict module boundaries inside one Express app) can be split into services later *if* a module (e.g., AI, Notifications) needs independent scaling.
- MongoDB + Redis + Socket.io fit comfortably in a monolith at college scale (thousands, not millions, of users).

Each backend module (attendance, hostel, placements, ai, etc.) lives in its own folder with its own routes/controllers/services/models, and modules only talk to each other through **service-layer function calls**, never by reaching into another module's DB models directly. This is what makes future extraction into microservices possible without a rewrite.

## 3. High-Level Component Diagram

```
                         ┌────────────────────────┐
                         │        Clients          │
                         │  Next.js 16 (Web/PWA)    │
                         └───────────┬─────────────┘
                                     │ HTTPS (REST) + WSS (Socket.io)
                                     ▼
                         ┌────────────────────────┐
                         │   Nginx (Reverse Proxy)  │
                         │  TLS termination, LB      │
                         └───────────┬─────────────┘
                                     ▼
                 ┌───────────────────────────────────────┐
                 │           Express.js API Server          │
                 │  ┌─────────────────────────────────┐  │
                 │  │  Middleware: Auth(JWT), RBAC,      │  │
                 │  │  RateLimit, Validation, Logger     │  │
                 │  └─────────────────────────────────┘  │
                 │  ┌───────────┐ ┌───────────┐ ┌───────┐ │
                 │  │  Auth     │ │ Academic  │ │ Hostel│ │
                 │  │  Module   │ │ Modules   │ │ Mess  │ │
                 │  └───────────┘ └───────────┘ └───────┘ │
                 │  ┌───────────┐ ┌───────────┐ ┌───────┐ │
                 │  │ Placement │ │ Complaint │ │ Event │ │
                 │  └───────────┘ └───────────┘ └───────┘ │
                 │  ┌─────────────────────────────────┐  │
                 │  │   AI Service Layer (OpenAI/LLM)    │  │
                 │  └─────────────────────────────────┘  │
                 │  ┌─────────────────────────────────┐  │
                 │  │   Socket.io (real-time layer)      │  │
                 │  └─────────────────────────────────┘  │
                 └───────────┬─────────────┬─────────────┘
                             ▼             ▼
                 ┌───────────────┐  ┌───────────────┐
                 │  MongoDB Atlas │  │     Redis      │
                 │  (primary DB)  │  │ (cache/queue/  │
                 │                │  │  sessions/pubsub)│
                 └───────────────┘  └───────────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
        ┌────────────────┐     ┌────────────────┐
        │   Cloudinary     │     │     AWS S3      │
        │ (images/avatars) │     │ (docs/resumes/  │
        │                  │     │  large files)   │
        └────────────────┘     └────────────────┘
```

## 4. Deployment Architecture (AWS EC2)

```
Internet → Route53 (DNS) → EC2 (Nginx) ──┬─→ Docker: Next.js container (port 3000)
                                          ├─→ Docker: Express API container (port 5000)
                                          ├─→ Docker: Redis container (port 6379)
                                          ├─→ Docker: Prometheus (port 9090)
                                          └─→ Docker: Grafana (port 3001)
MongoDB Atlas → external managed cluster (not on EC2)
S3 / Cloudinary → external managed storage
GitHub Actions → CI/CD → builds images → SSH deploy to EC2 / pushes to registry
```

All services orchestrated via `docker-compose.yml` for local dev and `docker-compose.prod.yml` for production, fronted by Nginx which handles TLS (Let's Encrypt/Certbot) and reverse-proxies `/api/*` → Express, `/socket.io/*` → Socket.io, everything else → Next.js.

## 5. Key Architectural Decisions (ADRs)

| Decision | Choice | Why |
|---|---|---|
| Monolith vs Microservices | Modular monolith | Faster to build/debug at college scale; split later if needed |
| DB | MongoDB (not SQL) | Highly heterogeneous schemas (assignments, complaints, events) fit documents better than rigid joins; Atlas gives managed scaling |
| Cache/Queue | Redis | Session storage, rate-limiting, Socket.io adapter for multi-instance pub/sub, BullMQ job queues for AI tasks/notifications |
| Auth | JWT (access + refresh) | Stateless, scales horizontally, works cleanly with mobile clients later |
| Real-time | Socket.io | Needed for live notifications, chat, attendance updates, complaint status |
| File storage split | Cloudinary (images) + S3 (docs) | Cloudinary excels at image transforms/CDN for avatars/event banners; S3 is cheaper/better for PDFs, resumes, bulk documents |
| AI integration | Server-side AI service layer, never client-direct | Keeps API keys safe, allows prompt-engineering/caching/rate-limiting centrally |
| State (frontend) | Zustand (client/UI state) + TanStack Query (server state) | Avoids the classic Redux-for-everything anti-pattern; TanStack Query owns caching/invalidation of server data, Zustand owns UI-only state (sidebar, modals, theme) |

## 6. Cross-Cutting Concerns

- **Validation:** Zod schemas shared where possible between frontend and backend types.
- **Error handling:** Centralized Express error middleware → consistent `{ success, error: { code, message } }` shape.
- **Logging:** Winston (app logs) + Morgan (HTTP logs), shipped to stdout for Docker log drivers.
- **Rate limiting:** `express-rate-limit` backed by Redis store.
- **Security:** Helmet, CORS allow-list, bcrypt password hashing, JWT rotation, input sanitization (mongo-sanitize), file-type validation on uploads.
- **Observability:** `prom-client` exposes `/metrics`; Prometheus scrapes it; Grafana dashboards visualize request latency, error rate, queue depth.
- **CI/CD:** GitHub Actions runs lint + test + build on every PR; deploy workflow builds Docker images and deploys to EC2 on merge to `main`.

## 7. Multi-Role Design Philosophy

Every module is designed around **who can see/do what**, not just CRUD. Role definitions live in `06-RBAC.md`. This is enforced at the API layer via middleware (`requireRole(['admin','faculty'])`), never trusted from the frontend alone.

## 8. AI Architecture

AI features are NOT scattered `fetch()` calls to OpenAI from random routes. They live behind a single `ai/` module with:
- `ai.service.ts` — wraps provider calls (model choice, retries, timeouts)
- `prompts/` — versioned prompt templates per feature
- `ai.queue.ts` — BullMQ + Redis queue for slow/batch AI jobs (e.g., nightly attendance-risk scoring, mess-demand prediction) so they don't block request/response cycles
- Result caching in Redis/Mongo to avoid re-paying for identical AI calls

This is covered in depth when we reach the AI phase — we are not implementing AI logic yet.

---
**Status:** Draft v1 — pending your approval before we proceed to folder structure & DB design.
