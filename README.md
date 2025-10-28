

## 🔍 1. ANALYSIS OF CURRENT ERP DOCUMENTATION

### ✅ Strengths
- Clear user roles (Demo, Coordinator, HR, Teacher, Parent, Admin)
- Well-defined core workflows (registration → demo → payment → time table → class → attendance → billing)
- Strong emphasis on **dispute resolution**, **feedback**, and **exam tracking**
- Department-based operational isolation (AA, BB, etc.)
- GST-compliant accounting logic (inclusive pricing)

### ❌ Key Pain Points & Gaps

| Area | Pain Point | Risk |
|------|-----------|------|
| **Data Silos** | Demo team uses **Google Sheets**; ERP only used post-registration | Lost leads, duplicate entries, no audit trail |
| **Manual Scheduling** | Coordinators confirm teacher availability via **calls/WhatsApp** before ERP entry | Double-booking, scheduling delays, no real-time sync |
| **GMeet Ambiguity** | Conflicting specs: “static link” vs “daily shuffled links” | UX confusion, security risk if links leak |
| **Attendance Disputes** | Dispute resolution is **manual** (coordinator calls parent) | Delayed resolution, inconsistent outcomes |
| **Payment Handling** | Temporary credits added manually → **no approval workflow** | Financial reconciliation errors |
| **Department Scalability** | No clear schema for **unlimited departments** or **staff assignment** | Operational bottlenecks as business grows |
| **Notification Fragmentation** | Mix of in-app banners + WhatsApp → **no unified engine** | Missed alerts, inconsistent messaging |
| **No Real-Time Monitoring** | Admin “class not started” alert is **reactive**, not proactive | Poor class compliance, revenue leakage |

---

## 🌟 2. BEST PRACTICES FOR EDUCATIONAL ERP SYSTEMS

| Practice | Application to Claps Learn |
|--------|----------------------------|
| **Single Source of Truth** | Replace Google Sheets with **ERP-native lead management** |
| **Real-Time Availability Engine** | Teacher availability → auto-blocked slots → no WhatsApp confirmation needed |
| **Event-Driven Architecture** | Attendance marked → auto-bill → auto-notify → auto-update dashboard |
| **Role-Based Data Isolation** | Department-scoped data access (teachers, students, coordinators) |
| **Audit-First Design** | Every edit (attendance, payment, time table) → immutable log |
| **Mobile-First Parent Experience** | In-app actions (cancel, dispute, recharge) reduce WhatsApp dependency |
| **Compliance by Design** | GST split, export formats, international payment flags built into core |

---

## 🧩 3. REQUIREMENTS FOR IMPROVED LOGICAL FLOW

### Core Principles:
1. **Automate what’s manual** (scheduling, notifications, balance updates)
2. **Verify before commit** (e.g., teacher availability check → auto-reserve)
3. **Real-time sync across roles** (teacher marks attendance → parent sees instantly)
4. **Department = tenant** (data, staff, WhatsApp number fully isolated)

### Revised Workflow Flow:
```
Lead → Demo Scheduling (in ERP) → Registration → Payment → 
Time Table (auto-check availability) → Class (GMeet) → 
Attendance (auto-bill) → Dispute (system-tracked) → 
Exam/Feedback → Monthly Statement
```

---

## 🏗️ 4. SYSTEM DESIGN

### A. System Architecture Diagram (Text Representation)

```
┌──────────────────┐     ┌──────────────────────┐
│   Frontend       │     │   Backend Services   │
│  (Web + Mobile)  │<--->│                      │
└─────────┬────────┘     └──────────┬───────────┘
          │                         │
          ▼                         ▼
┌──────────────────┐     ┌──────────────────────┐
│  User Roles:     │     │  Core Microservices: │
│  - Parent        │     │  - Auth & RBAC       │
│  - Teacher       │     │  - User Management   │
│  - Coordinator   │     │  - Department Mgmt   │
│  - HR            │     │  - Time Table Engine │
│  - Admin         │     │  - GMeet Integration │
│  - Accountant    │     │  - Attendance & Dispute │
│                  │     │  - Exam & Feedback   │
│                  │     │  - Accounts & GST    │
│                  │     │  - Notification Hub  │
└──────────────────┘     └──────────┬───────────┘
                                    │
                                    ▼
                          ┌──────────────────────┐
                          │   Database Layer     │
                          │  - PostgreSQL        │
                          │  - Tables:           │
                          │    • Users           │
                          │    • Departments     │
                          │    • Teachers        │
                          │    • Students        │
                          │    • TimeTable       │
                          │    • Attendance      │
                          │    • Disputes        │
                          │    • Accounts        │
                          │    • Exams           │
                          │    • Notifications   │
                          └──────────────────────┘
```

### B. Improved Data Flow Between Modules

| Trigger | Action | Affected Modules |
|--------|--------|------------------|
| Teacher marks attendance | → Deduct balance → Notify parent → Update teacher salary → Log dispute option | Accounts, Notification, Dispute, Dashboard |
| Coordinator creates time table | → Block teacher slot → Calculate projected hours → Notify teacher & parent | TimeTable, Teacher, Student, Notification |
| Parent raises dispute | → Freeze attendance → Alert coordinator → Pause billing | Attendance, Dispute, Accounts, Dashboard |
| Payment added | → Update balance (₹ + hrs) → Enable class access | Accounts, Student, Class Engine |

### C. Core Entities & Relationships

```plaintext
Department (1) —< User (Coordinator, Teacher, Student)
Department (1) —< WhatsAppHelpDesk

User (Teacher) —< AvailabilitySlot
User (Teacher) —< TimeTable —> User (Student)
User (Student) —< Attendance —< Dispute
User (Student) —< Exam
User (Student) —< AccountLedger

TimeTable —< ClassSession (date-specific)
Attendance —< ClassSession
```

### D. User Role Hierarchy & Permissions

| Role | Department Scope | Can View | Can Edit |
|------|------------------|--------|--------|
| **Admin** | All | All data | All data |
| **HR** | All Teachers | Teacher profiles, Availability | Teacher data, Availability |
| **Coordinator** | Assigned Dept | Students, Teachers (dept), TT, Disputes | Student fields, TT, Disputes, Payments |
| **Teacher** | Self + Assigned Students | Own TT, Attendance, Salary, Materials | Availability, Attendance, Exams |
| **Parent** | Own Student | TT, Attendance, Exams, Balance | Disputes, Class Cancellation (2h prior) |
| **Accountant** | All | AccountLedger, Salary | Manual ledger entries |

---

## 📦 5. MODULE DESIGN

### A. Integrated User Management System
- **Single sign-on** with DOB-based auth
- **Department assignment** at user creation
- **Auto-generated codes**: StudentCode, TeacherCode
- **Profile completeness** tracking (e.g., “39/39 fields filled”)

### B. Unified Dashboard (Role-Based Views)
- **Admin**: Daily cross-check tab (low balance, first class, feedback due, class monitoring)
- **Coordinator**: Dept-wise student list, dispute queue, time table builder
- **Teacher**: “My Class Today”, Attendance Log, Salary Preview
- **Parent**: Balance (₹ + hrs), Upcoming Class, Exam Results, Dispute Button

### C. Streamlined Workflow Processes

| Process | Improvement |
|--------|------------|
| **Demo Scheduling** | ERP-native calendar (no Google Sheets) → auto-assign to department |
| **Time Table Creation** | Real-time teacher availability grid → auto-reserve on save |
| **Attendance** | One-click marking → auto-bill → dispute option auto-enabled |
| **Dispute Resolution** | In-system form → coordinator action → audit trail → balance recalc |
| **Monthly Statements** | Auto-generate → “Send All” → WhatsApp + in-app |

### D. Real-Time Data Synchronization Plan
- **WebSockets** for live class monitoring (teacher joined? student joined?)
- **Database triggers** for balance updates (attendance → accounts)
- **Cron jobs** for daily alerts (low balance, feedback due)
- **API-first design** for future mobile app or WhatsApp Bot integration

---

## ✅ 1. DATABASE SCHEMA (PostgreSQL DDL)

```sql
-- Core Enums
CREATE TYPE user_role AS ENUM ('admin', 'hr', 'coordinator', 'teacher', 'parent', 'accountant');
CREATE TYPE department_status AS ENUM ('active', 'inactive');
CREATE TYPE student_status AS ENUM ('demo', 'ongoing', 'paused', 'stopped');
CREATE TYPE dispute_status AS ENUM ('pending', 'resolved', 'rejected');
CREATE TYPE payment_origin AS ENUM ('domestic', 'international');

-- Departments
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    code VARCHAR(10) UNIQUE NOT NULL,  -- e.g., 'AA', 'BB'
    name VARCHAR(100),
    whatsapp_number VARCHAR(20) NOT NULL,
    status department_status DEFAULT 'active'
);

-- Users (Unified)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    user_code VARCHAR(20) UNIQUE,  -- StudentCode / TeacherCode
    email VARCHAR(255) UNIQUE,
    password VARCHAR(10) NOT NULL,  -- DD-MM-YYYY (DOB)
    role user_role NOT NULL,
    department_id INT REFERENCES departments(id),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    is_active BOOLEAN DEFAULT true
);

-- Teachers (extends users)
CREATE TABLE teachers (
    user_id INT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    full_name VARCHAR(100) NOT NULL,
    gender VARCHAR(20),
    dob DATE NOT NULL,
    qualification TEXT,
    experience_years INT,
    subject VARCHAR(100),
    preferred_class VARCHAR(50),
    preferred_syllabus VARCHAR(50),
    medium VARCHAR(30),
    mobile VARCHAR(15),
    whatsapp VARCHAR(15),
    pin_code VARCHAR(10),
    source VARCHAR(50),
    internet_connectivity VARCHAR(30),
    gadgets TEXT[],  -- ['Laptop', 'Tablet', ...]
    interview_video_url TEXT,
    points_earned INT,
    performance_rating VARCHAR(20),
    hourly_salary NUMERIC(10,2),  -- pre-GST
    total_hours_taken NUMERIC(6,2) DEFAULT 0,
    conversion_ratio NUMERIC(5,2) DEFAULT 0,
    complaints_count INT DEFAULT 0,
    last_minute_cancellations INT DEFAULT 0,
    status VARCHAR(20) DEFAULT 'pending'
);

-- Students (extends users)
CREATE TABLE students (
    user_id INT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    full_name VARCHAR(100) NOT NULL,
    gender VARCHAR(10),
    dob DATE NOT NULL,
    school_name VARCHAR(150),
    class VARCHAR(20),
    syllabus VARCHAR(50),
    medium VARCHAR(30),
    parent_whatsapp VARCHAR(15) NOT NULL,
    student_contact VARCHAR(15),
    father_name VARCHAR(100),
    father_occupation VARCHAR(100),
    father_contact VARCHAR(15),
    mother_name VARCHAR(100),
    mother_occupation VARCHAR(100),
    mother_contact VARCHAR(15),
    address TEXT,
    city VARCHAR(100),
    country VARCHAR(100),
    pin_code VARCHAR(10),
    network_type VARCHAR(30),
    gmeet_link TEXT,
    first_class_date DATE,
    status student_status DEFAULT 'demo',
    feedback_submitted BOOLEAN DEFAULT false,
    tt_created BOOLEAN DEFAULT false
);

-- Teacher Availability (Grid: Day × Time Slot)
CREATE TABLE availability_slots (
    id SERIAL PRIMARY KEY,
    teacher_id INT REFERENCES teachers(user_id),
    day_of_week INT CHECK (day_of_week BETWEEN 1 AND 7),  -- Mon=1
    time_slot VARCHAR(20) NOT NULL,  -- e.g., 'FN', 'AN', '18:00-19:00'
    is_available BOOLEAN DEFAULT true,
    UNIQUE(teacher_id, day_of_week, time_slot)
);

-- Time Table
CREATE TABLE timetable (
    id SERIAL PRIMARY KEY,
    student_id INT REFERENCES students(user_id),
    teacher_id INT REFERENCES teachers(user_id),
    day_of_week INT,          -- for recurring
    specific_date DATE,       -- for one-time
    start_time TIME,
    duration_hours NUMERIC(3,1) NOT NULL,
    subject VARCHAR(100),
    fee_per_hour NUMERIC(10,2) NOT NULL,  -- inclusive of GST
    salary_per_hour NUMERIC(10,2) NOT NULL, -- pre-GST
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    CHECK (day_of_week IS NOT NULL OR specific_date IS NOT NULL)
);

-- Attendance
CREATE TABLE attendance (
    id SERIAL PRIMARY KEY,
    timetable_id INT REFERENCES timetable(id),
    class_date DATE NOT NULL,
    duration_hours NUMERIC(3,1) NOT NULL,
    topic TEXT,
    homework TEXT,
    marked_at TIMESTAMPTZ DEFAULT NOW(),
    is_disputed BOOLEAN DEFAULT false
);

-- Disputes
CREATE TABLE disputes (
    id SERIAL PRIMARY KEY,
    attendance_id INT REFERENCES attendance(id) ON DELETE CASCADE,
    student_id INT REFERENCES students(user_id),
    reason TEXT NOT NULL,
    status dispute_status DEFAULT 'pending',
    resolved_by INT REFERENCES users(id),  -- coordinator
    resolution_notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    resolved_at TIMESTAMPTZ
);

-- Accounts (Manual Ledger)
CREATE TABLE account_ledger (
    id SERIAL PRIMARY KEY,
    student_id INT REFERENCES students(user_id),
    date DATE NOT NULL,
    invoice_no VARCHAR(50),
    particulars TEXT,
    credit NUMERIC(12,2) DEFAULT 0,
    debit NUMERIC(12,2) DEFAULT 0,
    narration TEXT,
    payment_origin payment_origin DEFAULT 'domestic',
    added_by INT REFERENCES users(id),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Exams
CREATE TABLE exams (
    id SERIAL PRIMARY KEY,
    student_id INT REFERENCES students(user_id),
    teacher_id INT REFERENCES teachers(user_id),
    subject VARCHAR(100),
    chapter TEXT,
    exam_date DATE,
    mode VARCHAR(30),  -- 'viva', 'pdf', 'written'
    total_marks INT,
    obtained_marks INT,
    remarks TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Feedback
CREATE TABLE feedback (
    student_id INT PRIMARY KEY REFERENCES students(user_id),
    rating INT CHECK (rating BETWEEN 1 AND 5),
    comments TEXT,
    suggestions TEXT,
    submitted_at TIMESTAMPTZ DEFAULT NOW()
);
```

> ✅ **Key Features**:  
> - Department-scoped data via `department_id`  
> - Dual fee/salary tracking per timetable entry  
> - Dispute linked to attendance with audit trail  
> - Manual ledger with GST-ready structure  

---

## ✅ 2. API CONTRACT (Core Modules – REST)

### **Auth**
- `POST /auth/login`  
  → `{ user_code, password }` → JWT + role + department

---

### **Teachers**
- `GET /teachers?department_id=1&subject=Maths&syllabus=CBSE`  
  → Paginated list with filters
- `PUT /teachers/{id}/availability`  
  → `{ day_of_week, time_slot, is_available }`

---

### **Students**
- `POST /students` → Create (parent form)
- `PATCH /students/{id}` → Coordinator fills GMeet, status, etc.

---

### **Time Table**
- `POST /timetable`  
  → `{ student_id, teacher_id, day_of_week, start_time, duration, fee_per_hour, salary_per_hour }`
- `GET /timetable/student/{id}` → Time zone-aware
- `GET /timetable/teacher/{id}/today`

---

### **Attendance**
- `POST /attendance`  
  → `{ timetable_id, class_date, duration, topic, homework }`
- `GET /attendance/student/{id}`

---

### **Disputes**
- `POST /disputes`  
  → `{ attendance_id, reason }` (by parent)
- `PATCH /disputes/{id}`  
  → `{ status: 'resolved', resolution_notes }` (by coordinator)

---

### **Accounts**
- `POST /accounts/credit`  
  → `{ student_id, amount, invoice_no, payment_origin, narration }`
- `GET /accounts/student/{id}/balance`  
  → `{ balance_inr: 500, balance_hours: 2, fee_rate: 250 }`

---

### **Notifications**
- `GET /notifications` → Role-based alerts
- `POST /notifications/whatsapp/send-monthly` → Admin-only

---

### **Dashboard**
- `GET /dashboard/admin/daily` → Low balance, first class, feedback due
- `GET /dashboard/analytics/monthly?department=AA`

---

## ✅ 3. UI WIREFRAMES (Text-Based)

### **A. Time Table Creation (Coordinator View)**

```
[ HEADER: Create Time Table – Student: Arjun (Code: STU101) ]

STUDENT INFO
Name: Arjun | Class: 10 | Syllabus: CBSE | Medium: English
Current Fee: ₹250/hr (applies to all subjects)

TEACHER SELECTION
[Search: Subject=Maths, Syllabus=CBSE] → Dropdown: 
✅ Priya T (AA) – 4.8★, Available: Mon/Wed/Fri 5-6 PM
✅ Rajesh T (AA) – 4.5★, Available: Tue/Thu 6-7 PM

SELECT SCHEDULE TYPE:
(●) Recurring Days     ( ) One-Time Date

IF RECURRING:
Days: [Mon] [Wed] [Fri] 
Time: [5:00 PM - 6:00 PM] 
Duration: [1.0] hrs
Fee: ₹250/hr  [Apply to all subjects]

[ PREVIEW BUTTON ] → Shows 4-week projection: 12 hrs = ₹3000

[ CREATE TIME TABLE ] → Auto-blocks teacher slots
```

---

### **B. Attendance Dispute (Parent View)**

```
[ ATTENDANCE RECORD ]
Date: 25 Oct 2025 | Duration: 1.0 hr | Topic: Quadratic Equations
Status: ✅ Confirmed

[ RAISE DISPUTE ] BUTTON

DISPUTE FORM:
Reason: 
( ) Class not conducted
( ) Duration incorrect (actual: ___ hrs)
( ) Other: [__________________]

Comments: [Teacher didn’t join GMeet]

[ SUBMIT DISPUTE ]

→ Shows: "Dispute raised. Coordinator will contact you within 24 hrs."
→ Attendance status: ⚠️ Disputed (frozen)
```

---

### **C. Admin Daily Cross-Check Dashboard**

```
[ DASHBOARD – Admin | Department: AA ]

🚨 URGENT ACTIONS (Today: 28 Oct 2025)
──────────────────────────────────────
• LOW BALANCE (5 students): 
  - STU101 (₹200 = 0.8 hrs) → [Recharge Now]
  - STU105 (₹0) → [Paused]

• FIRST CLASS NOT STARTED (3):
  - STU202 → Demo on 25 Oct, no class yet → [Assign TT]

• FEEDBACK PENDING (2-week rule):
  - STU110 → First class: 14 Oct → [Send Reminder]

• TODAY’S CLASS MONITORING:
  [LIVE] 10:00 AM – Maths (Priya T ↔ Arjun) → ✅ Both joined
  [DELAY] 11:00 AM – Physics (Rajesh T) → ❌ Student not joined

[ EXPORT DAILY REPORT ]
```

---

Below are the **four deliverables** you requested — all **fully aligned** with your `user.xlsx` requirements and our prior clarifications.

---

## ✅ 1. ENTITY-RELATIONSHIP DIAGRAM (ERD) – TEXT REPRESENTATION

```
┌──────────────┐       ┌──────────────┐
│  departments │       │     users    │
├──────────────┤       ├──────────────┤
│ id (PK)      │<──────│ department_id│
│ code         │       │ id (PK)      │
│ whatsapp_num │       │ user_code    │
│ status       │       │ email        │
└──────────────┘       │ password     │
                       │ role         │
                       └──────┬───────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────────────┐    ┌────────────────┐    ┌────────────────┐
│   teachers    │    │   students     │    │  coordinators  │
├───────────────┤    ├────────────────┤    ├────────────────┤
│ user_id (PK)  │    │ user_id (PK)   │    │ user_id (PK)   │
│ full_name     │    │ full_name      │    │ (inherits user)│
│ ...           │    │ gmeet_link     │    └────────────────┘
│ hourly_salary │    │ status         │
└───────┬───────┘    │ feedback_sub...│
        │            └───────┬────────┘
        │                    │
        │                    │
┌───────────────┐    ┌───────▼────────┐
│availability_  │    │   timetable    │
│   slots       │    ├────────────────┤
├───────────────┤    │ id (PK)        │
│ teacher_id    │    │ student_id     │
│ day_of_week   │    │ teacher_id     │
│ time_slot     │    │ day/specific_dt│
│ is_available  │    │ start_time     │
└───────┬───────┘    │ duration       │
        │            │ fee_per_hour   │
        └────────────►│ salary_per_hr  │
                      └───────┬────────┘
                              │
                      ┌───────▼────────┐
                      │   attendance   │
                      ├────────────────┤
                      │ id (PK)        │
                      │ timetable_id   │
                      │ class_date     │
                      │ duration       │
                      │ is_disputed    │
                      └───────┬────────┘
                              │
                      ┌───────▼────────┐    ┌────────────────┐
                      │    disputes    │    │     exams      │
                      ├────────────────┤    ├────────────────┤
                      │ attendance_id  │    │ student_id     │
                      │ student_id     │    │ teacher_id     │
                      │ status         │    │ subject        │
                      └────────────────┘    │ exam_date      │
                                            └────────────────┘

                      ┌────────────────┐    ┌────────────────┐
                      │ account_ledger │    │    feedback    │
                      ├────────────────┤    ├────────────────┤
                      │ student_id     │    │ student_id (PK)│
                      │ credit/debit   │    │ rating         │
                      │ payment_origin │    │ comments       │
                      └────────────────┘    └────────────────┘
```

> 🔑 **Key Relationships**:  
> - **1 Department → Many Users** (teachers, students, coordinators)  
> - **1 Teacher → Many TimeTable entries**  
> - **1 TimeTable → Many Attendance records**  
> - **1 Attendance → 0 or 1 Dispute**  
> - **All financials tied to Student via `account_ledger`**

---

## ✅ 2. ROLE-BASED PERMISSION MATRIX

| Module / Action | Admin | HR | Coordinator (Dept) | Teacher | Parent/Student | Accountant |
|-----------------|-------|----|---------------------|---------|----------------|------------|
| **View All Departments** | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Create/Edit Department** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Teacher Registration (39 fields)** | ✅ | ✅ | ❌ | Self-fill only | ❌ | ❌ |
| **Edit Teacher Availability** | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| **View Teachers (Dept-scoped)** | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Student Registration (22 fields)** | ✅ | ❌ | ✅ | ❌ | Self-fill only | ❌ |
| **Assign GMeet Link** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| **Create/Edit Time Table** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| **View Own Time Table** | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ |
| **Mark Attendance** | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| **Raise Dispute** | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Resolve Dispute** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |
| **Log Exam** | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ |
| **View Exam Results** | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ |
| **Add Manual Credit/Debit** | ✅ | ❌ | ✅* | ❌ | ❌ | ✅ |
| **Export Salary Sheet** | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| **View Dashboard (Dept-scoped)** | ✅ | ✅ | ✅ | Limited | Limited | ✅ |
| **Send WhatsApp Notifications** | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ |

> *✅ = Full access | ✅* = Temporary credit only (if Accountant unavailable)

---

## ✅ 3. UI WIREFRAMES FOR CRITICAL USER JOURNEYS

### **A. Teacher Availability Grid (HR/Teacher View)**

```
[ HEADER: My Availability – Priya T (Maths, CBSE) ]

        MON    TUE    WED    THU    FRI    SAT    SUN
FN      [✅]   [ ]    [✅]   [ ]    [✅]   [ ]    [ ]
AN      [✅]   [ ]    [✅]   [ ]    [✅]   [ ]    [ ]

5:00-6:00  [ ]    [✅]   [ ]    [✅]   [ ]    [✅]   [ ]
6:00-7:00  [ ]    [✅]   [ ]    [✅]   [ ]    [✅]   [ ]
7:00-8:00  [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
8:00-9:00  [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]
9:00-10:00 [ ]    [ ]    [ ]    [ ]    [ ]    [ ]    [ ]

✅ = Available  |  ⚪ = Blocked  |  🟥 = Booked

[ SAVE AVAILABILITY ]
```

---

### **B. Time Table Creation (Coordinator – Dept AA)**

```
[ CREATE TIME TABLE – Student: Arjun (STU101) ]

STUDENT: Arjun | Class: 10 | Syllabus: CBSE | Medium: English
FEE: ₹250/hr (applies to all subjects)

SELECT TEACHER:
Subject: [Maths ▼]  Syllabus: [CBSE ▼]
→ Priya T (AA) ★4.8 – Available: Mon/Wed/Fri 5-6 PM

SCHEDULE TYPE:
(●) Recurring Days    ( ) One-Time Date

DAYS: [Mon] [Wed] [Fri]  
TIME: [5:00 PM] – Duration: [1.0] hrs

FEES OVERRIDE (per subject):
Maths: ₹250/hr  |  Sanskrit: ₹300/hr ← [Edit]

[ PREVIEW 4-WEEK PLAN ]
→ 12 hrs = ₹3,000

[ CREATE & BLOCK SLOTS ]
```

---

### **C. Attendance Dispute (Parent View)**

```
[ ATTENDANCE RECORD – 25 Oct 2025 ]
Duration: 1.0 hr | Topic: Quadratic Equations | Status: ✅ Confirmed

[ RAISE DISPUTE ]

Reason:
( ) Class not conducted
( ) Duration incorrect → Actual: [___] hrs
( ) Teacher didn’t join GMeet
( ) Other: [__________________]

Comments: [Teacher was absent]

[ SUBMIT DISPUTE ]

→ Status: ⚠️ Disputed (frozen)
→ Coordinator will contact you within 24 hrs.
```

---

### **D. Admin Daily Cross-Check Dashboard**

```
[ DASHBOARD – Admin | Today: 28 Oct 2025 ]

🚨 URGENT ACTIONS
──────────────────────────────────────
• LOW BALANCE (5): STU101 (₹200), STU105 (₹0) → [Recharge]
• FIRST CLASS NOT STARTED (3): STU202 → [Assign TT]
• FEEDBACK DUE (2-week rule): STU110 → [Send Reminder]

👁️ LIVE CLASS MONITORING
──────────────────────────────────────
10:00 AM – Maths (Priya T ↔ Arjun) → ✅ Both joined
11:00 AM – Physics (Rajesh T) → ❌ Student not joined

[ EXPORT DAILY REPORT ]
```

---

## ✅ 4. PRODUCTION-READY DATABASE SCHEMA (WITH DEPARTMENT ISOLATION)

> ✅ **Already provided in full DDL above** (see Section 1 of previous response).  
> 🔁 **Key reinforcement**:  
> - Every user (teacher, student, coordinator) has `department_id`  
> - All queries **must filter by `department_id`** for non-Admin roles  
> - `departments` table supports unlimited departments with WhatsApp number  

---

