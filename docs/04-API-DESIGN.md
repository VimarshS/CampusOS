# CampusOS — API Design

## Conventions

- Base URL: `/api/v1`
- Auth: `Authorization: Bearer <accessToken>` header
- Response envelope (always):
```json
// success
{ "success": true, "data": { ... }, "meta": { "page": 1, "limit": 20, "total": 100 } }
// error
{ "success": false, "error": { "code": "VALIDATION_ERROR", "message": "Email is required", "details": [...] } }
```
- Pagination: `?page=1&limit=20` query params on all list endpoints.
- Filtering: `?status=open&category=hostel` style query params, module-specific.
- Versioning: prefix now (`/v1`) so we can introduce `/v2` without breaking clients later.

## Auth Module — `/api/v1/auth`
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/register` | Public | Register (student self-signup or admin-created faculty) |
| POST | `/login` | Public | Returns access + refresh token |
| POST | `/refresh` | Public (refresh token) | Issue new access token |
| POST | `/logout` | Authenticated | Invalidate refresh token |
| POST | `/forgot-password` | Public | Send reset email |
| POST | `/reset-password` | Public (reset token) | Set new password |
| GET | `/me` | Authenticated | Current user profile |

## Users — `/api/v1/users`
| Method | Route | Access |
|---|---|---|
| GET | `/` | Admin | List/search users |
| GET | `/:id` | Self, Admin |
| PATCH | `/:id` | Self, Admin |
| DELETE | `/:id` | Admin (soft delete) |

## Academic Structure — `/api/v1/academics`
```
/departments        GET(all), POST(admin)
/departments/:id     GET, PATCH(admin), DELETE(admin)
/courses             GET, POST(admin)
/batches              GET, POST(admin)
/sections              GET, POST(admin)
/subjects              GET, POST(admin), PATCH(admin)
/timetable/:sectionId  GET
/timetable             POST(admin) — create slot
```

## Attendance — `/api/v1/attendance`
| Method | Route | Access | Description |
|---|---|---|---|
| POST | `/mark` | Faculty | Mark attendance for a session |
| GET | `/section/:sectionId/subject/:subjectId` | Faculty, Admin | Session-wise records |
| GET | `/student/:studentId` | Student(self), Faculty, Admin | Student's attendance summary + % |
| PATCH | `/:recordId` | Faculty | Correct an existing record |

## Assignments — `/api/v1/assignments`
```
GET    /                         list (filtered by subject/section)
POST   /                Faculty  create
GET    /:id
PATCH  /:id             Faculty  update/close
DELETE /:id             Faculty
POST   /:id/submit      Student  upload submission
GET    /:id/submissions Faculty  list submissions
PATCH  /submissions/:id Faculty  grade + feedback
```

## Marks & Results — `/api/v1/results`
```
POST   /                Faculty  add marks entry
GET    /student/:id     Student(self), Faculty, Admin
GET    /subject/:id     Faculty, Admin
PATCH  /:id             Faculty  correct entry
GET    /student/:id/transcript   generate PDF transcript
```

## Library — `/api/v1/library`
```
GET    /books
POST   /books                     LibraryStaff
POST   /issue                     LibraryStaff
POST   /return/:issueId           LibraryStaff
GET    /student/:id/issues        Student(self), LibraryStaff
```

## Hostel — `/api/v1/hostel`
```
GET    /hostels
POST   /apply                     Student
GET    /applications              Warden, Admin
PATCH  /applications/:id          Warden — approve/reject
GET    /rooms/:hostelId           Warden
POST   /rooms/:id/assign          Warden
```

## Mess — `/api/v1/mess`
```
GET    /menu/:hostelId
POST   /menu                      Warden
POST   /preference                Student — opt in/out for a meal
GET    /preference/:studentId     Student(self)
POST   /feedback                  Student
GET    /feedback/:hostelId        Warden, Admin
```

## Placement — `/api/v1/placement`
```
GET    /drives
POST   /drives                    PlacementOfficer
GET    /drives/:id
POST   /drives/:id/apply          Student — with resume upload
GET    /drives/:id/applications   PlacementOfficer
PATCH  /applications/:id          PlacementOfficer — update status
```

## Complaints — `/api/v1/complaints`
```
GET    /                          role-scoped (own vs all)
POST   /                          Authenticated
GET    /:id
PATCH  /:id                       Admin/assigned staff — update status/resolution
```

## Events — `/api/v1/events`
```
GET    /
POST   /                          Admin, Faculty
GET    /:id
POST   /:id/register              Student
GET    /:id/registrations         Organizer
```

## Communication — `/api/v1/communication`
```
GET    /announcements
POST   /announcements             Admin, Faculty
GET    /conversations             Authenticated
POST   /conversations             Authenticated — start DM/group
GET    /conversations/:id/messages
POST   /conversations/:id/messages
GET    /notifications             Authenticated
PATCH  /notifications/:id/read
```

## Analytics — `/api/v1/analytics`
```
GET    /overview                  Admin — cross-module KPIs
GET    /attendance-trends         Admin
GET    /placement-stats           Admin
GET    /complaint-volume          Admin
```

## AI — `/api/v1/ai`
```
POST   /study-planner              Student
POST   /doubt-assistant            Student — chat message in/out
POST   /resume-review              Student — analyze uploaded resume
GET    /placement-readiness/:id    Student(self), PlacementOfficer
GET    /attendance-risk            Admin, Faculty — batch/section-wide
GET    /performance-analysis/:id   Student(self), Faculty
GET    /feedback-analysis          Admin — mess/event sentiment summary
GET    /mess-demand/:hostelId      Warden — predicted counts
```

## WebSocket Events (Socket.io namespace `/ws`)

| Event | Direction | Payload |
|---|---|---|
| `notification:new` | server→client | `{ notification }` |
| `message:new` | server→client | `{ conversationId, message }` |
| `complaint:statusChanged` | server→client | `{ complaintId, status }` |
| `attendance:updated` | server→client | `{ sectionId, subjectId }` |
| `typing` | client↔server | `{ conversationId, userId }` |

Auth for sockets: JWT passed in `socket.handshake.auth.token`, verified in connection middleware before joining any room.

---
**Status:** Draft v1 — exact route lists will grow slightly as we implement each module, but this is the contract we build against.
