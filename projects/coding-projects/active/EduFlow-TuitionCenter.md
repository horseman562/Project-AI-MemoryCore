# EduFlow — Tuition Center Management System
*Coding Project - Created 2026-04-09*

## Description
MVP SaaS product for Malaysian tuition centers. First client: **Pusat Tusiyen Pintar Gemilang (PTPG)**. Manages students, classes, attendance, timetable, fees, and reports. Built as a server-driven SPA with Laravel + Inertia.js + Vue 3.

## Project Details
- **Type**: Coding Project
- **Status**: Active — Client branch in progress (client/ptpg)
- **Created**: 2026-04-09
- **Last Accessed**: 2026-04-12 (session 3)
- **Position**: #1
- **Codebase**: `D:\2025_project_portfolio\tuition-center`
- **Active Branch**: `client/ptpg` (MVP base on `main`)

## Technical Stack
- **Languages**: PHP 8.2, JavaScript (Vue 3)
- **Frameworks**: Laravel 12 (Backend), Vue 3 Composition API + Inertia.js v2 (Frontend)
- **Database**: PostgreSQL
- **Auth**: Laravel Breeze + Sanctum
- **Roles**: Spatie Laravel Permission v6 (future use)
- **CSS**: Tailwind CSS v3
- **Build**: Vite 7
- **Route helpers**: Ziggy v2

## Project Structure
```
tuition-center/
├── app/
│   ├── Http/Controllers/
│   │   ├── ClassController.php
│   │   ├── StudentController.php
│   │   ├── AttendanceController.php
│   │   └── ProfileController.php
│   ├── Models/
│   │   ├── TuitionClass.php      ← $table = 'classes', explicit FK names
│   │   ├── Student.php           ← SoftDeletes trait
│   │   └── Attendance.php        ← $table = 'attendance' (no auto-plural)
│   └── Providers/AppServiceProvider.php  ← Route::bind('class', ...)
├── resources/js/
│   ├── Pages/
│   │   ├── Dashboard.vue         ← 4 stat cards + quick actions + recent students
│   │   ├── Classes/Index.vue     ← inline add/edit/delete, student count
│   │   ├── Students/Index.vue    ← modal add/edit, class enrollment checkboxes
│   │   ├── Attendance/Index.vue  ← date picker + class table with status badges
│   │   └── Attendance/Take.vue   ← per-student status toggles, bulk set, save
│   └── Layouts/AuthenticatedLayout.vue   ← sticky navbar, flash banner
├── routes/web.php
└── docs/
    ├── README.md
    ├── architecture.md
    ├── database.md
    ├── controllers.md
    └── frontend.md
```

## Database Design (4 Tables)
- **classes**: id, name, timestamps
- **students**: id, name, age, parent_name, parent_phone, timestamps, deleted_at (soft delete)
- **class_student** (pivot): class_id FK, student_id FK — composite PK
- **attendance**: id, class_id FK, student_id FK, date, status (present/absent/late), timestamps — UNIQUE(class_id, student_id, date)

## Key Architecture Decisions
- **No REST API** — Inertia handles everything server-side, controllers return `Inertia::render()`
- **Soft deletes on students** — preserves attendance history when student leaves
- **No class scheduling in MVP** — no day/time columns; collect real feedback first
- **`updateOrCreate` for attendance** — re-submitting same date updates, never duplicates
- **`tuitionClass` not `class`** — "class" is reserved in PHP/Vue, renamed throughout

## Known Gotchas (Important!)
| Issue | Fix |
|-------|-----|
| `class` reserved in Vue as prop | Renamed to `tuitionClass` in controller + component |
| `attendance` table vs auto-plural `attendances` | Set `$table = 'attendance'` on model |
| TuitionClass FK auto-generates as `tuition_class_id` | Explicitly pass `'class_id'` in `belongsToMany()` |
| Route param `{class}` needs manual binding | `Route::bind('class', ...)` in AppServiceProvider |
| Form not updating after date change on Take.vue | Added `watch(() => props.attendance, ...)` and `watch(() => props.date, ...)` |

## Assessment Module — Confirmed Design (Ready to Build)

### Schema
```
assessments
├── title, type (awal_sesi/berkala/pertengahan_tahun/akhir_tahun/ujian_besar)
├── year (e.g. 2026), level_id, created_by

assessment_classes  (which classes take this assessment)
├── assessment_id, class_id
├── scheduled_date (nullable — admin OR teacher sets)
├── status (draft/completed)
├── scheduled_by (FK users)

assessment_scores  (one per student per subject)
├── assessment_id, class_id, student_id, subject_id
├── marks (out of 100), grade (auto-stored: A/B/C/D/E)
```

### Grading Scale
- A: 80-100, B: 60-79, C: 40-59, D: 20-39, E: 0-19

### Key Rules
- Max marks always 100
- Scores per subject = only subjects that student is enrolled in
- No retakes — one score per student per subject per assessment
- Teacher marks complete → visible to student/parent
- One assessment = one level (e.g. all Darjah 3 classes), each class has own date
- Admin creates + assigns classes, Admin OR Teacher sets scheduled_date
- Teacher enters scores + marks complete for their own class only

### Flow
1. Admin creates assessment → assigns classes (optionally sets dates)
2. Teacher sets date if not set → status: draft
3. Teacher enters scores per student per subject (only enrolled subjects)
4. Teacher marks complete → student/parent can see scores

---

## Progress Log

### 2026-04-12 (session 3)
- Built Assessment module end-to-end — fully complete
  - 3 migrations: `assessments`, `assessment_classes`, `assessment_scores`
  - 3 models: Assessment, AssessmentClass, AssessmentScore (with `computeGrade()` static helper)
  - AssessmentController: create, update (title/type/year), addClasses, destroy, scheduleClass, completeClass, reopenClass, take, saveScores
  - Assessments/Index.vue: cards grouped by year, edit modal, add classes modal, reopen (undo complete), subject tags on class rows
  - Assessments/Take.vue: score entry table per student (marks 0–100 + grade badge A–E), save + mark complete
  - Timetable integration: assessment events appear as all-day colored events; click to manage date + enter scores; choice modal when clicking empty day (Add Session vs Set Exam Date)
  - Student portal (Student/Dashboard.vue): tabs for Keputusan Peperiksaan + Kehadiran, results grouped by year
  - Parent dashboard updated: Keputusan Peperiksaan section per child
  - StudentController::portal() method added; student.dashboard route updated
  - Grading scale: A(80-100), B(60-79), C(40-59), D(20-39), E(0-19)
- Fixed 2 timetable bugs:
  - Month-view click defaulted both times to 00:00 → `after:start_time` validation failed → fixed default to 09:00/10:00 when allDay=true
  - Ziggy stale route issue: switched saveAssessDate + submitExamDate to Inertia `router.patch()` with hardcoded URL paths
- **NOT yet committed** — all changes since session 2 are uncommitted on `client/ptpg`

### 2026-04-12 (session 2)
- Fully designed + confirmed Assessment module (schema, flow, rules) — ready to build next session
- Discussed Report Card — depends on Assessment module + client format (still pending)
- Confirmed all 5 assessment types: Awal Sesi, Berkala, Pertengahan Tahun, Akhir Tahun, Ujian Besar
- All conducted by PTPG (mock exams) — NOT tracking school results
- Built Module Bank (file upload, admin+teacher only, filter by subject/level/type)
- Fixed Module Bank controller error (canDelete closure can't be Inertia prop)

### 2026-04-12 (session 1)
- Built Teachers module: CRUD + TeacherProfile (phone/bio) + password reset + nav link
- Built Parents module: full parent accounts + link children bidirectionally (from both Students and Parents pages) + parent portal dashboard (BM) showing attendance + invoices
- Timetable major improvements:
  - Class-color legend (solid colored pills above calendar, matching event colors)
  - `eventContent` custom render: compact single-line for month view, stacked name/time/teacher for week/day
  - Hover tooltip (dark floating card with class color band, shows all session details)
  - Font sizes bumped across whole calendar (older user friendly)
  - Slot height 5rem, `eventMinHeight: 56`, `slotDuration: 1hr`
  - Session status now editable from edit modal (draft/scheduled/completed/cancelled/makeup)
- Fixed attendance bug: `whereDate()` unreliable in PostgreSQL → changed to `->where('date', $dateString)` in AttendanceController + dashboard
- Fixed all existing sessions stuck in `draft` status (bulk updated to `scheduled` via tinker)
- Added inline enrollment date edit in Students modal (effective_from + first_month_override editable without withdraw/re-enroll)
- Committed all Module 1-8 + Teachers + Parents to `client/ptpg` branch (58ba21d)
- Discussed billing flow: generate → review drafts → issue all → mark paid
- Clarified: `per_session` / `per_enrollment` fee types exist in schema but NOT implemented in billing

### 2026-04-09
- Full MVP built from scratch: DB design → migrations → models → controllers → Vue pages → layout
- Fixed 5 major bugs (migration ordering, FK pluralization, attendance table naming, reserved keyword class, form watcher)
- Redesigned Attendance/Index to show date picker + status overview table (was confusing before)
- Built custom AuthenticatedLayout navbar (EduFlow branding, not default Breeze template)
- Landing page styled with modern hero/features design (based on Gemini prompt output)
- Created comprehensive docs in docs/ folder (architecture, database, controllers, frontend)
- Demo assessment: **MVP is demo-ready for cold calling** — needs seed data with Malaysian names to avoid empty dashboard

## Client: Pusat Tuisyen Pintar Gemilang (PTPG)
MoM: 8 April 2026. Branch: `client/ptpg`.
Full architecture doc: `docs/client-ptpg-architecture.md`
**Full DB design doc: `docs/ptpg-database-design.md`** ← complete 12-table schema with all columns
Client research doc: `docs/competitor-ptpg-analysis.md` (naming is misleading — it's client research)

**PTPG Details:**
- Location: Bandar Baru Bangi, Selangor (Bangi/Kajang market)
- Contact: 017-212 1035 / pintargemilang18@gmail.com
- Subjects: Academic (SR/SM), Religious (Quran, Fardhu Ain), STEAM (Robotik)
- Delivery modes: Physical, Online (WhatsApp/Google Meet), Home visit
- Pricing range: RM10/sesi – RM200/month
- Small group focus: max 15 students per class
- Key differentiator: Religious education + STEAM workshops

### Client Operational Model (Critical Context)
- Admin susuns timetable **months in advance** — no ad-hoc booking system
- Mid-month adjustments allowed (reschedule, substitute, cancellations)
- Personal class cancellation → make-up session based on teacher + student availability
- If no free slot this month → make-up pushed to next month

### New Schema (client/ptpg branch)
Two-layer scheduling model:
- `classes` = recurring template (what SHOULD happen)
- `class_sessions` = each actual occurrence (what ACTUALLY happened — can deviate)
- `attendance` now links to `session_id` + has `enrollment_type` (regular/drop_in)

**Enrollment — two paths:**
- `class_student` = regular enrollment, fee locked at join time
- `session_enrollments` = drop-in via **public form** (no login) — admin shares session link on social media, student/parent fills form, admin confirms payment → appears on attendance
  - `class_sessions.public_token` = safe URL token (not raw ID)
  - Captures name+phone directly (may not be existing student)
  - `payment_status`: pending → paid → appears on attendance
  - Private class naming: "[Subject] [Level] - Private - [Student Name]" (max_students=1)

**New tables:** `levels`, `subjects`, `fee_presets`, `class_sessions`, `session_enrollments`, `teacher_profiles`, `parent_student`, `invoices`, `invoice_items` — total 14 tables
**Modified:** `classes` (+ level_id, type, teacher_id, subject_id, schedule, fee columns), `students` (+ user_id, level_id), `attendance` (session_id + enrollment_type), `class_student` (+ fee_type, fee_amount, enrolled_at)

**Fee architecture — two billing models:**
- `fee_presets`: Level × Subject × Class type × Mode = default price (configured once by admin)
- **Grouping**: `classes.per_head_fee` → copied to `class_student.fee_amount` at enrollment → locked. Price changes don't affect existing students.
- **Personal (split billing)**: `classes.class_fee` is fixed total (e.g. RM180). Per-student invoice = `class_fee ÷ enrolled count` at billing time. Friends joining = cost shared. `class_student.fee_amount` is NULL for personal.
- `session_enrollments.fee_amount`: drop-in price per session
- Personal `max_students` default 5 (friends can share cost)
- No registration fee — removed
- `class_student.effective_from`: default first day of next month — new students start next month (aligns with advance timetable model). Admin can override for urgent mid-month joins.
- `class_student.first_month_override`: nullable — admin manually sets partial amount for mid-month. Null = full amount.
- `class_student.status` + `left_at`: active/inactive — stops invoice generation when student leaves a class. History preserved.
- `invoices`: one per student per month, status: draft/issued/paid/overdue. Paid = receipt, no separate table. Number format: INV-YYYYMM-XXXX.
- `invoice_items`: line items per invoice — one row per class charge. Handles all fee types.
- Invoice trigger: per_month = auto 1st of month. per_session = admin triggers after month ends. per_enrollment + drop_in = immediate.
- `class_sessions.status` includes `draft`: suggest+review flow — system generates drafts from template, admin confirms (→ scheduled) or clears all and restarts.

**class_sessions extras:** `topic` (nullable), `meeting_link` (online), `session_type` override, `makeup_for_id`
- 4 roles via Spatie: Admin, Teacher, Student, Parent

### Pending from Client (Blocked)
- Timetable format (including exam schedule)
- Attendance format
- Report Card format
- Invoice/receipt example + fee package details

### Build Order
**Phase 1 (ready to build):**
1. subjects table + model
2. Extend classes migration
3. class_sessions table + model
4. Modify attendance → session_id
5. Spatie roles (Admin, Teacher, Student, Parent)
6. teacher_profiles + parent_student + students.user_id

**Phase 2 (blocked on client info):** Timetable views, Fee/Invoice, Report Card

**Phase 3:** Notifications, Module Bank, Calendar

## Current Tasks
- [x] All 8 modules built + Teachers + Parents modules
- [x] Module Bank built (file upload, filter by subject/level/type, admin+teacher only)
- [x] Assessment module — fully built (see session 3 log)
- [x] All session 3 changes committed to client/ptpg (632f5ba exam module, 5a70454 module bank)
- [x] Report Card — built (barryvdh/laravel-dompdf, report_cards table, admin generate + student/parent download)
- [x] Navbar grouped into dropdowns: People / Academic / Admin
- [x] Substitute teacher tracking — substitute_teacher_id on class_sessions, timetable edit modal + visual badge
- [ ] **Reports module** — design confirmed session 4, ready to build (see spec below)
- [ ] Test billing, attendance, drop-in flows
- [ ] Teacher Dashboard — still a stub "Coming soon"

## Report Card — Confirmed Design (Session 4)

### Approach
- Auto-generate PDF using **barryvdh/laravel-dompdf**
- No admin selection of which assessments to include — system automatically pulls **all completed assessments** for that student in the selected year
- Admin only inputs: which student + which year (defaults to current year)
- Trigger: "Report Card" button per student row on the Students page

### PDF Layout
- **Header:** "Pusat Tuisyen Pintar Gemilang" (text only, no logo for now), "Laporan Prestasi Pelajar", student name, level, year
- **Score table:** rows = subjects, columns = completed assessments (e.g. Awal Sesi | Berkala | Pertengahan Tahun...)
  - Each cell: marks + grade
  - Empty cell if student has no score for that assessment
- **Attendance summary** at bottom: present / total sessions for the year
- **Footer:** generated date

### No remarks field — skip for now, add in v2 if PTPG requests

### DB Schema
- `report_cards` table: id, student_id FK, year (smallint), file_path (string), generated_by FK users, generated_at timestamp, unique(student_id, year)
- No additional tables needed — scores pulled from existing assessment_scores

### Access
- Admin: generate/regenerate from Students page
- Parent: download from parent portal ("Muat Turun Kad Laporan" or "Belum tersedia")
- Student: download from student portal

### Storage
- Private storage: `storage/app/private/report-cards/`
- Streamed securely — not a public URL
- Regenerate = overwrites file + updates record

### Files to create/modify
1. Migration — `report_cards` table
2. `ReportCard` model
3. `ReportCardController` — `generate()`, `download()`
4. `resources/views/report-card.blade.php` — Blade PDF template
5. Routes: `POST /students/{student}/report-card` (generate), `GET /students/{student}/report-card/{year}` (download)
6. `Students/Index.vue` — "Report Card" button per row
7. `Parent/Dashboard.vue` — download section per child
8. `Student/Dashboard.vue` — download section

### If no completed assessments for selected year
- Flash error: "No completed assessments found for this student in {year}"

## Reports Module — Confirmed Design (Session 4)

### 8 Reports
1. **Student Attendance** — per student, date range, session list + attendance %
2. **Teacher Substitute Log** — sessions with substitutes, original vs sub, date range
3. **Teacher Hours Summary** — total hours each teacher actually taught (original + sub sessions) in a period
4. **Monthly Revenue** — total collected per month, breakdown by class
5. **Outstanding / Tunggakan** — students with unpaid/overdue invoices + amount owed
6. **Daily Payment** — payments received on a specific day (paid_at date)
7. **Class Performance** — average marks per class per assessment, pass/fail rate
8. **Student Progress** — one student's marks trend across all assessments

### No new DB tables needed — all queries on existing data
### Payment method skipped — not stored yet, add later

### Architecture
- `ReportController` — one method per report, accepts filters via GET params
- All reports: read-only queries, no writes
- Single `/reports` page (Vue) with sidebar/tabs to switch between report types
- Each report: filter inputs + data table + print button
- Print: `window.print()` with CSS `@media print` (no PDF for reports, just browser print)
- Route: `GET /reports` (page) + `GET /reports/{type}` (data endpoint returning JSON)

### Files to create
- `ReportController.php`
- `Reports/Index.vue`
- Routes: `GET /reports` + data endpoints per report type

## Post-MVP / Future
- Seeder with realistic Malaysian data for demo

## Memory Patterns
- Inertia `useForm()` does NOT auto-sync with new props on page visit — must add explicit `watch(() => props.X, ...)` watchers if the page allows date/filter changes
- Laravel auto-pluralizes model names for table names — always set `$table` explicitly if the model name doesn't follow `table → model` convention
- `belongsToMany()` on a model named `TuitionClass` auto-generates FK as `tuition_class_id` — always explicitly pass the pivot FK names
- Route param `{class}` is reserved — use `Route::bind()` in AppServiceProvider to manually resolve
- PostgreSQL is strict about table/column names — errors are clear ("relation X does not exist")

---
*Coding Project Template v1.0*
