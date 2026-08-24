# CampusOS — AI-Powered Smart College ERP & Campus Management System

CampusOS is a production-grade, full-stack ERP for colleges covering academics, hostel/mess, placements, complaints, events, real-time communication, analytics, and AI-driven insights (study planning, doubt-solving, resume review, attendance-risk and performance prediction, feedback analysis, mess-demand forecasting).

> **Status:** Architecture & planning phase complete. Implementation not yet started. See `devlog.md` for progress.

## Tech Stack

**Frontend:** Next.js 16, TypeScript, Tailwind CSS, Shadcn UI, Zustand, TanStack Query
**Backend:** Node.js, Express.js, Socket.io
**Database:** MongoDB Atlas, Redis
**Auth:** JWT (access + refresh)
**Storage:** Cloudinary (images), AWS S3 (documents)
**DevOps:** Docker, Docker Compose, GitHub Actions, AWS EC2, Nginx, Prometheus, Grafana

## Architecture

Modular monolith backend (service-extractable later), Next.js App Router frontend with role-scoped route groups, MongoDB as primary store, Redis for cache/queues/real-time pub-sub. Full rationale in [`docs/01-ARCHITECTURE.md`](docs/01-ARCHITECTURE.md).

## Documentation

| Doc | Contents |
|---|---|
| [docs/01-ARCHITECTURE.md](docs/01-ARCHITECTURE.md) | System architecture, deployment topology, ADRs |
| [docs/02-MODULES.md](docs/02-MODULES.md) | Module breakdown, dependencies, build order |
| [docs/03-DATABASE-DESIGN.md](docs/03-DATABASE-DESIGN.md) | MongoDB collections, Redis keys, ER summary |
| [docs/04-API-DESIGN.md](docs/04-API-DESIGN.md) | REST API contract, Socket.io events |
| [docs/05-FOLDER-STRUCTURE.md](docs/05-FOLDER-STRUCTURE.md) | Monorepo folder layout |
| [docs/06-RBAC.md](docs/06-RBAC.md) | Roles, permission matrix, enforcement pattern |
| [docs/07-DEVELOPMENT-PHASES.md](docs/07-DEVELOPMENT-PHASES.md) | Phase-by-phase build plan (0-11) |
| [docs/08-TIMELINE.md](docs/08-TIMELINE.md) | Milestone timeline & portfolio checkpoints |
| [docs/09-REPO-STRUCTURE.md](docs/09-REPO-STRUCTURE.md) | Repo/branching/commit strategy |
| [devlog.md](devlog.md) | Running development journal |

## Core Modules

Student / Faculty / Admin Portals, Attendance, Assignments, Marks & Results, Library, Hostel, Mess Management, Placement Portal, Complaint Management, Event Management, Communication System, Analytics Dashboard, and 8 AI features (Study Planner, Doubt Assistant, Resume Review, Placement Readiness Score, Attendance Risk Prediction, Academic Performance Analysis, Feedback Analysis, Mess Demand Prediction).

## Getting Started

_Setup instructions will be added at the end of Phase 0 (Project Bootstrap) once the monorepo is scaffolded._

## License

TBD.
