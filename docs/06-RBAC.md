# CampusOS — Role-Based Access Control (RBAC) Design

## Roles

| Role | Description |
|---|---|
| `student` | Enrolled student |
| `faculty` | Teaching staff |
| `admin` | Institution administrator (super-user) |
| `warden` | Hostel warden |
| `placement_officer` | Manages placement drives |
| `library_staff` | Manages library operations |

A user has exactly **one** primary `role` field on `users`. If a person genuinely holds two roles in real life (e.g., faculty who is also a warden), we create two separate accounts in v1 rather than multi-role-per-user — this keeps permission logic linear. (Multi-role support is a documented future enhancement, not v1 scope.)

## Enforcement Layers

1. **Backend middleware (source of truth):** `requireRole(['admin','warden'])` on every protected route. This is non-negotiable and never bypassed.
2. **Ownership checks:** Some routes need role + ownership, e.g. a student can `GET /results/student/:id` only if `:id === req.user.studentProfileId` OR role is faculty/admin. Implemented as a small `requireSelfOrRole()` helper, not ad-hoc per controller.
3. **Frontend route guards:** Next.js `middleware.ts` reads the JWT role claim and redirects unauthorized navigation — this is UX convenience, NOT security (never trust it alone).
4. **UI conditional rendering:** Sidebar/menu items hidden per role — also UX only.

## Permission Matrix (module × role)

`R` = Read, `W` = Write/Create, `U` = Update, `D` = Delete/Manage, `-` = No access, `Own` = only own records

| Module | Student | Faculty | Admin | Warden | Placement Officer | Library Staff |
|---|---|---|---|---|---|---|
| Auth/Profile | Own RWU | Own RWU | All RWUD | Own RWU | Own RWU | Own RWU |
| Academic Structure | R | R | RWUD | R | R | R |
| Attendance | Own R | RWU (own classes) | RWUD | R | - | - |
| Assignments | Own R + submit | RWUD (own subjects) | R | - | - | - |
| Marks/Results | Own R | RWU (own subjects) | RWUD | - | R | - |
| Library | Own R | R | RWUD | - | - | RWUD |
| Hostel | Own RW (apply) | - | RWUD | RWUD (own hostel) | - | - |
| Mess | Own RW (preference/feedback) | - | R | RWUD (own hostel) | - | - |
| Placement | Own RW (apply) | - | R | - | RWUD | - |
| Complaints | Own RW | Own RW + assigned RU | RWUD | Assigned RU | - | - |
| Events | R + register | RW (own) | RWUD | R | R | R |
| Communication | Own R/W | Own R/W | RWUD (announcements) | Own R/W | Own R/W | Own R/W |
| Analytics | - | Limited (own classes) | Full | Limited (own hostel) | Limited (placement) | - |
| AI Features | Own use | Own use (analysis tools) | Full view | Mess-demand only | Readiness-score only | - |

## Implementation Pattern

```ts
// middlewares/rbac.middleware.ts
export const requireRole = (roles: Role[]) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    return next(new ApiError(403, 'FORBIDDEN', 'You do not have access to this resource'));
  }
  next();
};

export const requireSelfOrRole = (paramKey: string, roles: Role[]) => (req, res, next) => {
  const isSelf = req.params[paramKey] === req.user.profileId;
  const hasRole = roles.includes(req.user.role);
  if (!isSelf && !hasRole) {
    return next(new ApiError(403, 'FORBIDDEN', 'Access denied'));
  }
  next();
};
```

Route usage example:
```ts
router.get('/student/:studentId', verifyJWT, requireSelfOrRole('studentId', ['faculty','admin']), getAttendance);
```

## JWT Claims

Access token payload kept minimal (never store sensitive data in JWT — it's decodable, not encrypted):
```json
{ "sub": "<userId>", "role": "student", "profileId": "<studentProfileId>", "iat": ..., "exp": ... }
```

---
**Status:** Draft v1 — this matrix is the contract every route's middleware must match in Phase 1.
