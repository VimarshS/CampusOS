# CampusOS — Module Breakdown

Each module below will become its own backend folder (`/modules/<name>`) and frontend route group. "Depends on" tells us build order later.

## Foundation Modules (build first — everything depends on these)

### 1. Auth & User Management
- Register/login (Student, Faculty, Admin, Warden, Placement Officer roles)
- JWT access + refresh token flow
- Password reset, email verification
- Profile management
- Depends on: nothing (foundation)

### 2. Core Academic Structure
- Departments, Courses, Batches/Sections, Subjects, Semesters, Timetable
- This is reference data every other academic module needs
- Depends on: Auth

## Academic Modules

### 3. Student Portal
- Dashboard aggregating: attendance %, upcoming assignments, results, notices
- Depends on: Auth, Core Academic Structure

### 4. Faculty Portal
- Dashboard: classes taught, pending grading, attendance to mark
- Depends on: Auth, Core Academic Structure

### 5. Admin Portal
- User management, department/course management, system-wide analytics, approvals
- Depends on: Auth

### 6. Attendance
- Faculty marks attendance per class/session
- Student views attendance %, subject-wise breakdown
- Depends on: Core Academic Structure

### 7. Assignments
- Faculty creates/publishes assignments with due dates + file attachments (S3)
- Student submits (file upload), faculty grades + feedback
- Depends on: Core Academic Structure, Auth

### 8. Marks & Results
- Internal marks, exam marks, grade computation, result publishing, transcripts (PDF export)
- Depends on: Core Academic Structure

## Campus Life Modules

### 9. Library
- Book catalog, issue/return, fines, reservation queue
- Depends on: Auth

### 10. Hostel
- Room allocation, hostel applications, warden approvals, occupancy tracking
- Depends on: Auth

### 11. Mess Management
- Weekly menu, meal preferences/opt-out, feedback, (later) AI demand prediction
- Depends on: Hostel (mess tied to hostel residents, but day-scholars may opt in too)

### 12. Placement Portal
- Company drives, job postings, student applications, resume upload (S3), placement status tracking
- Depends on: Auth, Student Portal

### 13. Complaint Management
- Raise complaint (hostel/mess/academic/infra), category routing, status tracking, admin resolution workflow
- Depends on: Auth

### 14. Event Management
- Event creation, RSVP/registration, event calendar, banner images (Cloudinary)
- Depends on: Auth

### 15. Communication System
- Announcements/notices (role-targeted), direct messaging, real-time notifications (Socket.io)
- Depends on: Auth, Socket.io infra

### 16. Analytics Dashboard
- Cross-module aggregated charts for Admin (attendance trends, placement stats, complaint volume, hostel occupancy)
- Depends on: most other modules (built near the end)

## AI Modules (built after core modules are stable)

### 17. AI Study Planner
- Generates personalized study schedules from timetable + upcoming assignments/exams
- Depends on: Core Academic Structure, Assignments, Marks

### 18. AI Doubt Assistant
- Chat-based Q&A scoped to subject/syllabus context
- Depends on: Core Academic Structure

### 19. AI Resume Review
- Analyzes uploaded resume (S3), gives structured feedback
- Depends on: Placement Portal

### 20. AI Placement Readiness Score
- Composite score from academics + resume + mock test/activity data
- Depends on: Marks, Placement Portal, AI Resume Review

### 21. AI Attendance Risk Prediction
- Flags students trending toward attendance shortage
- Depends on: Attendance

### 22. AI Academic Performance Analysis
- Trend analysis + weak-subject detection from Marks history
- Depends on: Marks

### 23. AI Feedback Analysis
- Sentiment/theme extraction from mess/event/course feedback text
- Depends on: Mess Management, Event Management (feedback fields)

### 24. AI Mess Demand Prediction
- Predicts next-day meal counts from historical opt-in/opt-out + attendance
- Depends on: Mess Management

## Build Order Summary

```
Phase 1: Auth + Core Academic Structure   (foundation)
Phase 2: Student/Faculty/Admin Portals (skeleton) + Attendance
Phase 3: Assignments + Marks & Results
Phase 4: Library + Complaint Management
Phase 5: Hostel + Mess Management
Phase 6: Placement Portal
Phase 7: Event Management + Communication System (+ Socket.io real-time)
Phase 8: Analytics Dashboard
Phase 9: AI Features (all 8, one by one)
Phase 10: DevOps hardening, monitoring, deployment polish
```

This order is also our phase roadmap — detailed in `07-DEVELOPMENT-PHASES.md`.
