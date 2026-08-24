# CampusOS — Development Phases

Each phase is **independently testable**: it ends with something you can run, click through, and verify, plus a devlog + README update + commit message. We do not move to the next phase until you confirm the current one works for you.

## Phase 0 — Project Bootstrap
- Monorepo scaffold (npm workspaces), root configs (eslint/prettier/tsconfig)
- Docker Compose for local dev: Mongo (local, later swapped to Atlas), Redis, API, Web
- Health-check route (`GET /api/v1/health`) + basic Next.js landing page
- **Test:** `docker compose up` → both services respond

## Phase 1 — Auth & Core Academic Structure
- User model + register/login/refresh/logout, JWT + RBAC middleware
- Departments, Courses, Batches, Sections, Subjects CRUD (admin-only)
- Frontend: login/register pages, protected route middleware, role-based redirect
- **Test:** Register as each role, log in, confirm role-based redirect + 403 on wrong-role routes

## Phase 2 — Portals Skeleton + Attendance
- Student/Faculty/Admin dashboard shells (empty widgets wired to real auth)
- Attendance marking (faculty) + attendance viewing (student) + % calculation
- **Test:** Faculty marks a session, student sees updated attendance %

## Phase 3 — Assignments + Marks & Results
- Assignment CRUD, file upload to S3, submission + grading flow
- Marks entry, result publishing, PDF transcript generation
- **Test:** Full loop — faculty posts assignment → student submits → faculty grades → student sees grade

## Phase 4 — Library + Complaint Management
- Book catalog, issue/return, fines
- Complaint raise/track/resolve workflow with role-based status transitions
- **Test:** Issue/return a book with fine calc; raise + resolve a complaint end-to-end

## Phase 5 — Hostel + Mess Management
- Hostel application → warden approval → room assignment
- Mess menu, daily opt-in/out, feedback capture
- **Test:** Student applies for hostel, warden approves + assigns room; student sets meal preference

## Phase 6 — Placement Portal
- Drive creation, eligibility filtering, student application + resume upload
- Application status pipeline for placement officer
- **Test:** Officer creates drive, eligible student applies, officer updates status through to "selected"

## Phase 7 — Events + Communication System (Real-time)
- Event CRUD + registration
- Announcements, direct messaging, Socket.io live notifications
- **Test:** Two browser sessions — message sent in one appears live in the other; announcement triggers notification

## Phase 8 — Analytics Dashboard
- Aggregation endpoints (attendance trends, placement stats, complaint volume, hostel occupancy)
- Recharts-based admin dashboard
- **Test:** Dashboard reflects real data changes from prior phases

## Phase 9 — AI Features (sub-phased, one at a time)
- 9a. AI service layer + queue infra (BullMQ, prompt versioning, result caching)
- 9b. AI Attendance Risk Prediction (uses Phase 2 data — good first AI feature, clear signal)
- 9c. AI Academic Performance Analysis (uses Phase 3 data)
- 9d. AI Study Planner
- 9e. AI Doubt Assistant (chat)
- 9f. AI Resume Review + AI Placement Readiness Score
- 9g. AI Feedback Analysis (mess/event feedback sentiment)
- 9h. AI Mess Demand Prediction
- **Test (per sub-phase):** Each AI feature independently produces sane output on seeded test data

## Phase 10 — DevOps Hardening & Production Deployment
- Prometheus metrics + Grafana dashboards
- GitHub Actions CI (lint/test/build) + CD (deploy to EC2)
- Nginx TLS, production docker-compose, backup strategy for Mongo Atlas
- **Test:** Push to `main` → auto-deploys → live HTTPS URL serves the app; Grafana shows live metrics

## Phase 11 — Polish & Buffer
- Bug bash, accessibility pass, loading/error states audit, empty states, mobile responsiveness
- **Test:** Full manual QA pass against a checklist we'll write at that point

---
**Rule:** We do not start Phase N+1 until Phase N is confirmed working by you and devlog/README are updated.
