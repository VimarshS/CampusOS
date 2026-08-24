# CampusOS — Database Design (MongoDB Atlas)

## Design Principles

1. **Embed for tightly-coupled, bounded, rarely-independently-queried data** (e.g., address inside a user profile).
2. **Reference (ObjectId) for large, independently-growing, cross-module data** (e.g., a Student references Department, not embeds it).
3. Every collection has `createdAt`, `updatedAt` (Mongoose timestamps), and most have `isDeleted` (soft delete) instead of hard deletes for audit-friendliness.
4. Every collection has explicit indexes listed — defined in the Phase where that collection is built.

## Collections

### `users`
Central identity collection for ALL roles (Student, Faculty, Admin, Warden, PlacementOfficer, LibraryStaff).
```
{
  _id, name, email (unique, indexed), passwordHash, role: enum,
  phone, avatarUrl (Cloudinary), isActive, isEmailVerified,
  refreshTokenHash, lastLoginAt, createdAt, updatedAt
}
```

### `studentprofiles`
```
{ _id, userId (ref users, unique, indexed), rollNumber (unique, indexed),
  departmentId (ref departments), batchId (ref batches), sectionId (ref sections),
  semester, admissionYear, guardianInfo: { name, phone, email },
  hostelResident: bool, cgpa, backlog count }
```

### `facultyprofiles`
```
{ _id, userId (ref users, unique), employeeId (unique), departmentId,
  designation, subjectsTaught: [ref subjects], officeHours }
```

### `departments`
```
{ _id, name, code (unique), hodUserId (ref users) }
```

### `courses`
```
{ _id, name, code, departmentId (ref), durationSemesters, totalCredits }
```

### `batches`
```
{ _id, courseId (ref), year, name (e.g. "2024-2028") }
```

### `sections`
```
{ _id, batchId (ref), name (e.g. "A"), semester }
```

### `subjects`
```
{ _id, name, code (unique), departmentId (ref), semester, credits, facultyId (ref facultyprofiles) }
```

### `timetableslots`
```
{ _id, sectionId (ref), subjectId (ref), facultyId (ref), dayOfWeek, startTime, endTime, room }
```

### `attendancerecords`
One doc per class session per section-subject.
```
{ _id, subjectId, sectionId, facultyId, date (indexed),
  records: [ { studentId, status: enum['present','absent','late'] } ] }
```
Compound index: `{ sectionId: 1, subjectId: 1, date: 1 }` unique — prevents duplicate marking.

### `assignments`
```
{ _id, subjectId, facultyId, title, description, attachmentUrl (S3),
  dueDate (indexed), maxMarks, createdAt }
```

### `submissions`
```
{ _id, assignmentId (ref, indexed), studentId (ref, indexed), fileUrl (S3),
  submittedAt, status: enum['submitted','late','graded'],
  marksObtained, feedback, gradedBy, gradedAt }
```
Compound unique index: `{ assignmentId: 1, studentId: 1 }`

### `examresults`
```
{ _id, studentId (indexed), subjectId, examType: enum['internal','midterm','final'],
  semester, marksObtained, maxMarks, grade, publishedAt }
```

### `books`
```
{ _id, isbn (unique), title, author, category, totalCopies, availableCopies }
```

### `bookissues`
```
{ _id, bookId (ref), studentId (ref, indexed), issuedAt, dueDate, returnedAt,
  fineAmount, status: enum['issued','returned','overdue'] }
```

### `hostels`
```
{ _id, name, warden (ref users), totalRooms }
```

### `hostelrooms`
```
{ _id, hostelId (ref), roomNumber, capacity, occupants: [ref studentprofiles] }
```

### `hostelapplications`
```
{ _id, studentId (ref), hostelId, status: enum['pending','approved','rejected'], appliedAt, reviewedBy }
```

### `messmenus`
```
{ _id, hostelId (ref), dayOfWeek, meal: enum['breakfast','lunch','snacks','dinner'], items: [String] }
```

### `messpreferences`
Daily opt-in/out per student per meal — feeds AI demand prediction.
```
{ _id, studentId (indexed), date (indexed), meal, optedIn: bool }
```

### `messfeedback`
```
{ _id, studentId, date, meal, rating (1-5), comment }
```

### `companies`
```
{ _id, name, logoUrl (Cloudinary), website, sector }
```

### `placementdrives`
```
{ _id, companyId (ref), role, ctc, eligibilityCriteria: { minCgpa, allowedDepartments, maxBacklogs },
  driveDate, applicationDeadline, status: enum['upcoming','ongoing','closed'] }
```

### `placementapplications`
```
{ _id, driveId (ref, indexed), studentId (ref, indexed), resumeUrl (S3),
  status: enum['applied','shortlisted','interviewed','selected','rejected'], appliedAt }
```
Compound unique index: `{ driveId: 1, studentId: 1 }`

### `complaints`
```
{ _id, raisedBy (ref users, indexed), category: enum['academic','hostel','mess','infrastructure','other'],
  title, description, attachmentUrl, status: enum['open','in_progress','resolved','rejected'],
  assignedTo (ref users), resolutionNote, createdAt, resolvedAt }
```

### `events`
```
{ _id, title, description, bannerUrl (Cloudinary), venue, startAt, endAt,
  organizerId (ref users), category, registrationRequired: bool }
```

### `eventregistrations`
```
{ _id, eventId (ref, indexed), studentId (ref, indexed), registeredAt, attended: bool }
```
Compound unique index: `{ eventId: 1, studentId: 1 }`

### `announcements`
```
{ _id, title, body, targetRoles: [enum], targetDepartments: [ref], createdBy, createdAt, expiresAt }
```

### `messages` (direct/chat)
```
{ _id, conversationId (indexed), senderId, text, attachmentUrl, readBy: [userId], createdAt }
```

### `conversations`
```
{ _id, participantIds: [ref users], isGroup, lastMessageAt (indexed) }
```

### `notifications`
```
{ _id, userId (indexed), type, title, body, link, isRead (indexed), createdAt }
```

### `airesults` (generic AI output store — study plans, risk scores, readiness scores, resume feedback)
```
{ _id, userId (indexed), feature: enum['study_planner','doubt_qna','resume_review',
  'placement_readiness','attendance_risk','performance_analysis','feedback_analysis','mess_demand'],
  input: Mixed, output: Mixed, model, createdAt (indexed) }
```
Generic on purpose — avoids 8 near-identical collections; queried by `{ userId, feature }`.

## Entity Relationship Summary (textual)

```
users 1───1 studentprofiles ──* departments/batches/sections
users 1───1 facultyprofiles ──* subjects
subjects ──* timetableslots
sections + subjects ──* attendancerecords (embedded per-student records array)
subjects ──* assignments ──* submissions ──* studentprofiles
studentprofiles ──* examresults
studentprofiles ──* bookissues ──* books
hostels ──* hostelrooms ──* studentprofiles (occupants)
hostels ──* messmenus, messpreferences, messfeedback
companies ──* placementdrives ──* placementapplications ──* studentprofiles
users ──* complaints
users(organizer) ──* events ──* eventregistrations ──* studentprofiles
users ──* announcements, notifications, conversations/messages
users ──* airesults
```

## Redis Usage (non-relational, not "collections")

| Key pattern | Purpose | TTL |
|---|---|---|
| `session:refresh:<userId>` | Refresh token validity check | 7d |
| `ratelimit:<ip>:<route>` | Rate limiting counters | 1-15 min |
| `cache:attendance:<studentId>` | Cached computed attendance % | 10 min |
| `queue:ai-jobs` | BullMQ queue for async AI tasks | n/a |
| `pubsub:socket` | Socket.io Redis adapter channel (multi-instance) | n/a |

---
**Status:** Draft v1 — indexes will be finalized per-phase in actual Mongoose schemas, not all upfront.
