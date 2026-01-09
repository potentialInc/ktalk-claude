# Project Knowledge

> K Talk - Korean Language Learning Platform
> Last updated: 2026-01-04

## Product Overview

K Talk is a comprehensive Korean language learning platform supporting both online and offline education:

| Platform | Description | Frontend |
|----------|-------------|----------|
| **K Talk Live** | Online Korean language learning with live classes, scheduling, and payments | frontend-live |
| **K Talk Language Center (LC)** | Offline campus-based Korean language programs | frontend-lc |

### Purpose

- Unified student management system for both online and offline learning
- Connect students with Korean language tutors
- Manage classes, schedules, payments, and student progress
- Handle applications, enrollments, and community engagement

---

## Business Domain

### User Roles

| Role | ID | Capabilities |
|------|----|--------------|
| **SUPER_ADMIN** | 1 | Full system access, manage users/classes/payments/settings |
| **TUTOR** | 2 | Teach classes, track attendance, manage schedule, view payments |
| **STUDENT** | 3 | Attend classes, make payments, participate in forums |

### Korean Language Levels

```
BEGINNER_LOW → BEGINNER_HIGH → INTERMEDIATE_LOW → INTERMEDIATE_HIGH → ADVANCE_LOW → ADVANCE_HIGH
```

### Student Journey

```
Application → Free Class Consultation → Enrollment → Payment → Class Attendance → Review/Testimonial
```

1. **Application**: Student applies for free class (online via Calendly or form)
2. **Free Class**: One-on-one session with tutor to assess level
3. **Enrollment**: Student registers for a program
4. **Payment**: PayPal subscription or one-time payment with optional promo codes
5. **Class Attendance**: Regular class sessions with attendance tracking
6. **Review**: Student provides feedback and testimonials

### Class Lifecycle

- **Class**: Unique course with classNumber, teacher, students, schedule
- **ClassType**: Category defining capacity and payment structure
- **ClassSchedule**: Individual sessions (UPCOMING → DONE/CANCEL)
- **Attendance**: PRESENT / ABSENT / LATE per session

### Payment System

- **PayPal Integration**: Subscription plans with auto-renewal
- **Promo Codes**: Discount percentages applied to payments
- **Payment Statuses**: PENDING → PAID / PARTIAL / CANCEL / NO_SHOW
- **Teacher Payments**: Compensation tracking (DEBIT/CREDIT)

---

## Tech Stack

### Backend

| Technology | Version | Purpose |
|------------|---------|---------|
| NestJS | 10.x | Framework |
| TypeORM | 0.3.x | Database ORM |
| PostgreSQL | 16 | Database |
| Passport JWT | - | Authentication |
| class-validator | - | DTO validation |
| Winston | - | Logging |
| AWS S3 | - | File storage |
| PayPal SDK | - | Payment processing |
| Bull/Redis | - | Job queues (emails, notifications) |
| Nodemailer | - | Email service |

### Frontend LC (Language Centre)

| Technology | Version | Purpose |
|------------|---------|---------|
| React | 19 | UI library |
| React Router | 7.5 | Routing (with SSR) |
| Redux Toolkit | - | State management |
| Tailwind CSS | 4 | Styling |
| React Hook Form | - | Form handling |
| Axios | - | HTTP client |

### Frontend Live (K Talk Live App)

| Technology | Version | Purpose |
|------------|---------|---------|
| Remix | 2.15 | Full-stack framework |
| React | 18 | UI library |
| Redux Toolkit | - | State management |
| Tailwind CSS | 3.4 | Styling |
| FullCalendar | - | Calendar UI |
| Chart.js | - | Analytics charts |
| TinyMCE | - | Rich text editor |
| PayPal React | - | Payment integration |
| Framer Motion | - | Animations |

---

## Key Entities

### User Management (5 entities)
| Entity | Purpose |
|--------|---------|
| User | Core user with role, profile, bank info, payment status |
| Document | User-uploaded files (documents, certificates) |
| PaypalInformation | PayPal account and subscription details |
| PublishTutor | Published tutor profiles for display |
| AvailableTime | Weekly availability slots (dayOfWeek + timeSlot) |

### Classes & Learning (7 entities)
| Entity | Purpose |
|--------|---------|
| Class | Course with classNumber, teacher, dates, zoomLink |
| ClassType | Category with capacity and allowance type |
| ClassStudents | Student enrollment (junction table) |
| ClassSchedule | Individual sessions with duration and status |
| ClassWeekSchedule | Weekly schedule patterns |
| TextBook | Course materials (title, chapter, fileUrl) |
| FreeClass | One-off promotional classes |

### Payments (4 entities)
| Entity | Purpose |
|--------|---------|
| StudentPayment | Payment records with amounts, dates, status |
| TeacherPayment | Tutor compensation tracking |
| StudentPromo | Promo code usage per student |
| Promo | Discount codes with percentages |

### Community & Content (7 entities)
| Entity | Purpose |
|--------|---------|
| Forum | Class/school discussions with file attachments |
| ForumComment | Threaded comments (self-referencing replies) |
| Blog | Articles with slug, tags, metaDescription |
| BlogCategory | Blog categorization |
| Inquiry | Support tickets from users |
| InquiryReply | Admin responses to inquiries |
| Testimonial | Published student success stories |

### Applications & Scheduling (3 entities)
| Entity | Purpose |
|--------|---------|
| Application | Free class applications with status tracking |
| Calendly | Calendly meeting records and integration |
| Enrollment | Campus enrollment records |

### Communications (4 entities)
| Entity | Purpose |
|--------|---------|
| Notification | System notifications |
| UserNotification | User-notification mapping |
| BulkEmail | Batch email campaigns with tracking |
| BulkNotification | Batch in-app notifications |

**Total: 42 entities** (all inherit BaseEntity with audit fields)

---

## Backend Modules

### Core
- `auth` - JWT authentication, registration, password reset
- `user` - User management and profiles

### Class Management
- `class` - Class CRUD and management
- `class-type` - Class categorization
- `class-students` - Enrollment management
- `class-schedule` - Session scheduling
- `class-review` - Student feedback
- `text-book` - Course materials

### Payments
- `student-payment` - Student billing
- `teacher-payment` - Tutor compensation
- `promo` - Discount codes
- `student-promo` - Promo usage
- `paypal` - PayPal integration

### Community
- `forum` / `forum-comment` - Discussions
- `blog` / `blog-category` - Articles
- `inquiry` / `inquiry-reply` - Support tickets
- `testimonial` - Success stories
- `contact-us` - Contact form

### Admin
- `dashboard` - Analytics
- `newsletter` - Email subscriptions
- `bulk-email` - Mass emails
- `bulk-notification` - Mass notifications
- `application` - Free class applications
- `calendly` - Calendar integration
- `enrollment` - Campus enrollments

---

## Frontend Applications

### Frontend-LC (Language Center Marketing Site)

**Purpose**: Public marketing website for offline Korean language programs

**Key Routes**:
| Route | Page |
|-------|------|
| `/` | Homepage |
| `/campus` | Campus listings |
| `/campus/:id` | Campus details |
| `/about` | About page |
| `/programs` | Program information |
| `/activities` | Activities |
| `/accommodation` | Housing options |
| `/contact` | Contact form |
| `/enroll` | Enrollment form |

**Features**: No authentication, marketing content, testimonials, campus showcase

---

### Frontend-Live (K Talk Live App)

**Purpose**: Full e-learning platform with role-based dashboards

**Public Routes**:
| Route | Page |
|-------|------|
| `/` | Homepage |
| `/blog` | Blog articles |
| `/pricing` | Pricing plans |
| `/tutor` | Tutor profiles |
| `/contact-us` | Contact form |
| `/community` | Public forum |

**Auth Routes**:
| Route | Page |
|-------|------|
| `/login` | Login |
| `/registration` | Sign up |
| `/apply` | Free class application |
| `/forgot-password` | Password recovery |

**Dashboard Routes (Protected)**:

| Role | Routes |
|------|--------|
| **All Users** | `/dashboard`, `/dashboard/profile`, `/dashboard/my-schedule`, `/dashboard/my-class`, `/dashboard/forum` |
| **Student** | `/dashboard/payment` |
| **Teacher** | `/dashboard/tuition-fee-management` |
| **Admin** | `/dashboard/student-management`, `/dashboard/tutor-management`, `/dashboard/class-management`, `/dashboard/payment-management`, `/dashboard/applicant-management`, `/dashboard/inquiry-management`, `/dashboard/site-settings/*` |

---

## Services

| Service | Port | Profile | Description |
|---------|------|---------|-------------|
| postgres | 5432 | default | PostgreSQL database |
| postgres-test | 5433 | test | Test database |
| backend | 5001 | dev | NestJS API server |
| frontend-lc | 3000 | dev | Language Centre site |
| frontend-live | 3001 | dev | K Talk Live app |

---

## Development Setup

### Prerequisites
- Docker & Docker Compose
- Node.js 20+
- Make (optional)

### Quick Start

```bash
# Install all dependencies
make install

# Start database
make docker-db

# Run migrations and seed
make migrate && make seed

# Start services (in separate terminals)
cd backend && npm run start:dev          # Port 5001
cd frontend-lc && npm run dev            # Port 3000
cd frontend-live && npm run dev -- --port 3001
```

### Test Accounts (Seeded)

| Role | Email | Password |
|------|-------|----------|
| **SUPER_ADMIN** | admin@gmail.com | Admin123123@ |
| **TUTOR** | teacher@gmail.com | Teacher123123@ |
| **STUDENT** | student@gmail.com | Student123123@ |

> Note: These accounts are created by the seed script (`make seed`)

### Environment Variables

**Backend (.env)**
```
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=postgres
DATABASE_PASSWORD=postgres
DATABASE_NAME=ktalk_db
AUTH_JWT_SECRET=your-secret-key
PORT=5001
```

**Frontend (.env)**
```
VITE_API_BASE_URL=http://localhost:5001
VITE_GOOGLE_RECAPTCHA_SITE_KEY=...
VITE_PAYPAL_CLIENT_ID=...  # frontend-live only
```

---

## Key Enums Reference

| Enum | Values |
|------|--------|
| RolesEnum | SUPER_ADMIN (1), TUTOR (2), STUDENT (3) |
| ActiveStatusEnum | ACTIVE, IN_ACTIVE, BLOCK |
| PaymentStatusEnum | PENDING, PAID, PARTIAL, CANCEL, NO_SHOW, ABANDONED, RESCHEDULED |
| ApplicationStatusEnum | NEW, EMAIL_SENT, CONVERTED_TO_STUDENT, DROPPED, ZOOM_MEETING_SCHEDULED |
| KoreanLanguageLevelEnum | BEGINNER_LOW/HIGH, INTERMEDIATE_LOW/HIGH, ADVANCE_LOW/HIGH |
| ClassStatusEnum | UPCOMING, DONE, CANCEL |
| AttendanceStatusEnum | PRESENT, ABSENT, LATE |
| WeekDayEnum | MONDAY - SUNDAY |
| BulkEmailStatusEnum | QUEUED, SENDING, COMPLETED |

---

## Project Structure

```
ktalk/
├── backend/              # NestJS backend API
│   ├── src/
│   │   ├── modules/      # 37+ feature modules
│   │   ├── database/     # Entities, migrations, seeders
│   │   ├── shared/       # Enums, guards, decorators, interfaces
│   │   └── config/       # Configuration
│   └── test/
│
├── frontend-lc/          # Language Centre marketing site
│   ├── app/
│   │   ├── components/   # Pages, atoms, common, UI
│   │   ├── services/     # HTTP services
│   │   └── routes/       # Route definitions
│   └── public/
│
├── frontend-live/        # K Talk Live main app
│   ├── app/
│   │   ├── routes/       # Remix routes (public, auth, dashboard)
│   │   ├── components/   # Role-based components
│   │   ├── services/     # 40+ HTTP services
│   │   └── redux/        # State management
│   └── public/
│
├── docker-compose.yml
├── Makefile
└── .claude/docs/         # Documentation
```

---

## Related Documentation

- [PROJECT_API.md](PROJECT_API.md) - API endpoints reference
- [PROJECT_DATABASE.md](PROJECT_DATABASE.md) - Database schema details
- [BEST_PRACTICES.md](BEST_PRACTICES.md) - Coding standards
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) - Common issues and solutions
