# CampusOS — Folder Structure

## Monorepo Layout

We use a monorepo with two apps + shared packages, managed via npm workspaces (no need for Turborepo/Nx complexity yet — can add later if build times demand it).

```
campusos/
├── apps/
│   ├── web/                      # Next.js 16 frontend
│   └── api/                      # Express backend
├── packages/
│   ├── shared-types/             # Shared TS types/interfaces (User, Role enums, API response shapes)
│   └── config/                   # Shared eslint/tsconfig/prettier configs
├── docs/                         # This documentation set
├── docker/
│   ├── nginx/
│   │   └── nginx.conf
│   ├── prometheus/
│   │   └── prometheus.yml
│   └── grafana/
│       └── dashboards/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── docker-compose.yml
├── docker-compose.prod.yml
├── devlog.md
├── README.md
└── package.json                  # workspace root
```

## `apps/api/` (Express Backend — Modular Monolith)

```
apps/api/
├── src/
│   ├── config/
│   │   ├── db.ts                 # Mongoose connection
│   │   ├── redis.ts              # Redis client
│   │   ├── cloudinary.ts
│   │   └── s3.ts
│   ├── middlewares/
│   │   ├── auth.middleware.ts    # verifyJWT
│   │   ├── rbac.middleware.ts    # requireRole([...])
│   │   ├── error.middleware.ts
│   │   ├── validate.middleware.ts
│   │   └── rateLimit.middleware.ts
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.routes.ts
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   └── auth.validation.ts
│   │   ├── users/
│   │   ├── academics/            # departments, courses, batches, sections, subjects, timetable
│   │   ├── attendance/
│   │   ├── assignments/
│   │   ├── results/
│   │   ├── library/
│   │   ├── hostel/
│   │   ├── mess/
│   │   ├── placement/
│   │   ├── complaints/
│   │   ├── events/
│   │   ├── communication/
│   │   ├── analytics/
│   │   └── ai/
│   │       ├── ai.routes.ts
│   │       ├── ai.service.ts
│   │       ├── ai.queue.ts
│   │       └── prompts/
│   ├── models/                   # Mongoose schemas (one file per collection)
│   │   ├── user.model.ts
│   │   ├── studentProfile.model.ts
│   │   └── ...
│   ├── sockets/
│   │   ├── index.ts               # Socket.io server init + auth middleware
│   │   └── handlers/
│   ├── utils/
│   │   ├── ApiError.ts
│   │   ├── ApiResponse.ts
│   │   ├── asyncHandler.ts
│   │   └── logger.ts
│   ├── jobs/                      # BullMQ workers (AI jobs, scheduled reports)
│   ├── app.ts                     # Express app setup (middleware, routes mount)
│   └── server.ts                  # HTTP server + Socket.io bootstrap
├── tests/
│   ├── unit/
│   └── integration/
├── Dockerfile
├── .env.example
├── tsconfig.json
└── package.json
```

Every module folder (e.g. `modules/attendance/`) follows the same internal shape: `*.routes.ts → *.controller.ts → *.service.ts → *.validation.ts`. Controllers never touch Mongoose models directly — only services do. This is the boundary that keeps modules swappable/extractable later.

## `apps/web/` (Next.js 16 Frontend)

```
apps/web/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (student)/
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── attendance/page.tsx
│   │   │   ├── assignments/page.tsx
│   │   │   ├── results/page.tsx
│   │   │   ├── hostel/page.tsx
│   │   │   ├── placement/page.tsx
│   │   │   └── ai/
│   │   │       ├── study-planner/page.tsx
│   │   │       └── doubt-assistant/page.tsx
│   │   ├── (faculty)/
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── attendance/page.tsx
│   │   │   └── grading/page.tsx
│   │   ├── (admin)/
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── users/page.tsx
│   │   │   └── analytics/page.tsx
│   │   ├── layout.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                   # shadcn generated primitives
│   │   ├── shared/                # Navbar, Sidebar, RoleGuard, etc.
│   │   └── modules/                # feature-specific components (AttendanceTable, ComplaintCard...)
│   ├── hooks/                     # custom hooks, e.g. useAuth, useSocket
│   ├── lib/
│   │   ├── api-client.ts          # axios instance w/ interceptors (refresh token)
│   │   ├── socket-client.ts
│   │   └── utils.ts
│   ├── stores/                    # Zustand stores (ui-store, auth-store)
│   ├── queries/                   # TanStack Query hooks per module (useAttendanceQuery, etc.)
│   ├── types/                     # re-exports from packages/shared-types
│   └── middleware.ts               # Next.js middleware for route protection
├── public/
├── Dockerfile
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

## Rationale

- **Route groups `(student)/(faculty)/(admin)`** in Next.js App Router let each role have its own layout (different sidebar/nav) without URL prefix pollution, while `middleware.ts` + a `RoleGuard` component enforce that a student can't navigate into `/admin/*` even by typing the URL.
- **`packages/shared-types`** prevents drift between backend response shapes and frontend expectations — both apps import from the same source of truth.
- Test folders exist from day one so we build the *habit* of testing per phase, not bolt it on at the end.

---
**Status:** Draft v1 — this is the structure Phase 1 will actually scaffold.
