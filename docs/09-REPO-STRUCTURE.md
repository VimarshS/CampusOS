# CampusOS — GitHub Repository Structure

## Repo Strategy

**Single monorepo** named `campusos` (matches folder structure in `05-FOLDER-STRUCTURE.md`). One repo is correct here — frontend/backend are tightly coupled in development velocity, and a solo/small team gets no benefit from repo-splitting overhead (separate CI, separate versioning, cross-repo PRs) at this scale.

## Branching Strategy

We use **trunk-based development with short-lived feature branches** — simple, and appropriate for solo/small-team pace (GitFlow's multiple long-lived branches would be overkill).

```
main                → always deployable, protected branch
  └─ feature/phase1-auth
  └─ feature/phase2-attendance
  └─ fix/attendance-duplicate-marking
```

- Branch naming: `feature/phase<N>-<short-desc>`, `fix/<short-desc>`, `chore/<short-desc>`
- Every phase = one or more feature branches → PR into `main` → squash merge
- `main` is protected: requires CI (lint+test+build) to pass before merge

## Commit Message Convention

**Conventional Commits**, since it pairs naturally with generating changelogs and keeps devlog entries consistent:
```
<type>(<scope>): <short description>

feat(auth): implement JWT login and refresh token flow
fix(attendance): prevent duplicate attendance marking per session
docs(devlog): update phase 1 completion log
chore(docker): add redis service to docker-compose
refactor(complaints): extract status-transition logic into service layer
test(assignments): add integration tests for submission grading
```
Types used: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `ci`

At the end of every phase, I will give you the exact commit message(s) to use — this is part of the Phase completion ritual per your Development Rules.

## Repository Root Files

```
campusos/
├── .github/workflows/{ci.yml, deploy.yml}
├── .gitignore
├── .env.example              # root-level, documents all required env vars across apps
├── docker-compose.yml
├── docker-compose.prod.yml
├── README.md                 # project overview, setup instructions, architecture summary
├── devlog.md                 # running development journal, updated every phase
├── CONTRIBUTING.md            # (optional, useful if you ever open-source it)
├── LICENSE
└── package.json               # workspace root
```

## GitHub Actions Overview (built in Phase 10, referenced now for repo shape)

- `ci.yml`: triggered on PR → lint, typecheck, unit tests, build both apps
- `deploy.yml`: triggered on push to `main` → build Docker images → push to registry (or directly SSH+build on EC2) → restart containers via `docker-compose.prod.yml`

## Issue/Project Tracking (optional but recommended)

GitHub Projects board with columns `Backlog / This Phase / In Progress / Done`, one card per module/task from `07-DEVELOPMENT-PHASES.md`. Not required for us to proceed, but useful if you want visible progress tracking.

---
**Status:** Draft v1 — repo will be initialized in Phase 0.
