# Project Database Design

> **Version**: v3.0
> **Last Updated**: 2026-01-04
> **Database**: PostgreSQL 16
> **ORM**: TypeORM 0.3.x

---

## Entity Relationship Diagram (Core)

```
┌──────────────────┐                     ┌──────────────────┐
│      User        │◄──────1:N──────────│   Application    │
│──────────────────│                     │──────────────────│
│ id (PK)          │                     │ id (PK)          │
│ email (UK)       │                     │ fullName         │
│ name             │                     │ email            │
│ password         │                     │ phoneNumber      │
│ role (enum)      │                     │ koreanLanguageLevel│
│ activeStatus     │                     │ status (enum)    │
│ paymentStatus    │                     │ freeClassTutor FK│
└────────┬─────────┘                     └──────────────────┘
         │
         │ 1:N (teacher/student)
         ├─────────────────────────────────────────┐
         │                                         │
         ▼                                         ▼
┌──────────────────┐                     ┌──────────────────┐
│      Class       │◄─────1:N──────────│  ClassStudents   │
│──────────────────│                     │──────────────────│
│ id (PK)          │                     │ id (PK)          │
│ classNumber (UK) │──────1:N──────────▶│ student FK       │
│ teacher FK       │                     │ class FK         │
│ classType FK     │                     │ enrollmentDate   │
│ latestBook FK    │                     └──────────────────┘
│ status (enum)    │
│ startDate        │                     ┌──────────────────┐
│ endDate          │◄─────1:N──────────│  ClassSchedule   │
│ zoomId           │                     │──────────────────│
│ zoomPasscode     │                     │ id (PK)          │
└────────┬─────────┘                     │ class FK         │
         │                               │ date             │
         │                               │ status (enum)    │
         │ 1:N                           └──────────────────┘
         ▼
┌──────────────────┐                     ┌──────────────────┐
│      Forum       │─────1:N───────────▶│  ForumComment    │
│──────────────────│                     │──────────────────│
│ id (PK)          │                     │ id (PK)          │
│ title            │                     │ content          │
│ description      │                     │ forum FK         │
│ type (enum)      │                     │ writer FK        │
│ writer FK        │                     │ replyTo FK       │
│ class FK         │                     └──────────────────┘
└──────────────────┘

┌──────────────────┐                     ┌──────────────────┐
│   StudentPayment │                     │  TeacherPayment  │
│──────────────────│                     │──────────────────│
│ id (PK)          │                     │ id (PK)          │
│ student FK       │                     │ teacher FK       │
│ class FK         │                     │ class FK         │
│ amount           │                     │ amount           │
│ paymentStatus    │                     │ paymentType      │
│ paymentType      │                     │ paymentDate      │
└──────────────────┘                     └──────────────────┘

Legend: PK = Primary Key, FK = Foreign Key, UK = Unique Key
```

---

## Entities Overview

| Entity | Table Name | Description | Module |
|--------|------------|-------------|--------|
| **Core** ||||
| User | `user` | Users (Admin, Tutor, Student) | user |
| Class | `class` | Course/class units | class |
| ClassType | `class_type` | Class type classification | class-type |
| ClassStudents | `class_students` | Student enrollment junction | class-students |
| ClassSchedule | `class_schedule` | Individual class sessions | class-schedule |
| ClassWeekSchedule | `class_week_schedule` | Weekly schedule patterns | class |
| ClassReview | `class_review` | Student reviews of classes | class-review |
| **Learning** ||||
| TextBook | `text_book` | Textbook management | text-book |
| StudentAttendance | `student_attendance` | Attendance tracking | student-attendance |
| FreeClass | `free_class` | Free class offerings | class |
| ScheduleSyllabus | `schedule_syllabus` | Syllabus per schedule | class-schedule |
| **Applications & Enrollment** ||||
| Application | `application` | Free class applications | application |
| Enrollment | `enrollment` | Class enrollment records | enrollment |
| AvailableTime | `available_time` | Tutor availability slots | available-time |
| Calendly | `calendly` | Calendly meeting data | calendly |
| **Payments** ||||
| StudentPayment | `student_payment` | Student payment records | student-payment |
| TeacherPayment | `teacher_payment` | Teacher payment records | teacher-payment |
| PaypalInformation | `paypal_information` | PayPal account info | paypal |
| Promo | `promo` | Promotional codes | promo |
| StudentPromo | `student_promo` | Promo usage by students | student-promo |
| **Communication** ||||
| Forum | `forum` | Discussion boards | forum |
| ForumComment | `forum_comment` | Forum replies | forum-comment |
| Inquiry | `inquiry` | User inquiries | inquiry |
| InquiryReply | `inquiry_reply` | Inquiry responses | inquiry-reply |
| Notification | `notification` | Notification messages | notification |
| UserNotification | `user_notification` | User-notification junction | notification |
| **Content** ||||
| Blog | `blog` | Blog posts | blog |
| BlogCategory | `blog_category` | Blog categorization | blog-category |
| Testimonial | `testimonial` | User testimonials | testimonial |
| FAQ | `faq` | Frequently asked questions | faq |
| Media | `media` | Media content (Jeju) | jeju/media |
| MediaCategory | `media_category` | Media categorization | jeju/media-category |
| **Admin & Publishing** ||||
| PublishTutor | `publish_tutor` | Published tutor profiles | publish-tutor |
| PublishClassReview | `publish_class_review` | Published class reviews | publish-class-review |
| ContactUs | `contact_us` | Contact form submissions | contact-us |
| Newsletter | `newsletter` | Newsletter subscriptions | newsletter |
| **Bulk Operations** ||||
| BulkEmail | `bulk_email` | Bulk email campaigns | bulk-email |
| BulkNotification | `bulk_notification` | Bulk notifications | bulk-notification |
| **Supporting** ||||
| Document | `document` | User documents | user |
| OTP | `otp` | One-time passwords | otp |

---

## Relationships

### Core Relationships

| Parent | Child | Type | FK Column | Description |
|--------|-------|------|-----------|-------------|
| User | Class | 1:N | `teacher_id` | Teacher's classes |
| User | ClassStudents | 1:N | `student_id` | Student enrollments |
| User | Document | 1:N | `user_id` | User's documents |
| Class | ClassStudents | 1:N | `class_id` | Class enrollments |
| Class | ClassSchedule | 1:N | `class_id` | Class sessions |
| Class | ClassReview | 1:N | `class_id` | Class reviews |
| Class | Forum | 1:N | `class_id` | Class forums |
| ClassType | Class | 1:N | `class_type_id` | Classes of type |
| TextBook | Class | M:N | `class-books-relation` | Books used in class |

### Communication Relationships

| Parent | Child | Type | FK Column | Description |
|--------|-------|------|-----------|-------------|
| User | Forum | 1:N | `writer_id` | User's forum posts |
| User | Inquiry | 1:N | `writer_id` | User's inquiries |
| Forum | ForumComment | 1:N | `forum_id` | Forum comments |
| ForumComment | ForumComment | 1:N | `reply_to_id` | Nested replies |
| Inquiry | InquiryReply | 1:N | `inquiry_id` | Inquiry replies |

### Payment Relationships

| Parent | Child | Type | FK Column | Description |
|--------|-------|------|-----------|-------------|
| User | StudentPayment | 1:N | `student_id` | Student's payments |
| User | TeacherPayment | 1:N | `teacher_id` | Teacher's payments |
| Class | StudentPayment | 1:N | `class_id` | Class payments |
| User | StudentPromo | 1:N | `student_id` | Promo usage |

### Application Relationships

| Parent | Child | Type | FK Column | Description |
|--------|-------|------|-----------|-------------|
| User | Application | 1:N | `free_class_tutor_id` | Assigned tutor |
| User | AvailableTime | M:N | `user-available-times` | Tutor availability |

---

## Enums

| Enum | Values | Used In | File |
|------|--------|---------|------|
| RolesEnum | `SUPER_ADMIN (1)`, `TUTOR (2)`, `STUDENT (3)` | User.role | role.enums.ts |
| ActiveStatusEnum | `ACTIVE (1)`, `IN_ACTIVE (2)`, `BLOCK (3)` | User.activeStatus | active-status.enum.ts |
| ClassActiveStatusEnum | `ACTIVE (1)`, `INACTIVE (2)` | Class.status | class-active-status.enum.ts |
| PaymentStatusEnum | `CANCEL (1)`, `PENDING (2)`, `PAID (3)`, `PARTIAL (4)`, `INVOICE_ISSUED (5)`, `NO_SHOW (6)`, `ABANDONED (7)`, `RESCHEDULED (8)` | User.paymentStatus, StudentPayment | payment-status.enum.ts |
| AttendanceStatusEnum | `PRESENT (1)`, `ABSENT (2)`, `LATE (3)` | StudentAttendance.status | attendance-status.enum.ts |
| ApplicationStatusEnum | `NEW (1)`, `EMAIL_SENT (2)`, `CONVERTED_TO_STUDENT (3)`, `DROPPED (4)`, `ZOOM_MEETING_SCHEDULED (5)` | Application.status | application-status.enum.ts |
| KoreanLanguageLevelEnum | `BEGINNER_LOW (1)`, `BEGINNER_HIGH (2)`, `INTERMEDIATE_LOW (3)`, `INTERMEDIATE_HIGH (4)`, `ADVANCE_LOW (5)`, `ADVANCE_HIGH (6)` | Application.koreanLanguageLevel | korean-language-level.enum.ts |
| ForumTypeEnum | `CLASS (1)`, `SCHOOL (2)` | Forum.type | forum-type.enum.ts |
| PostTypeEnum | `ALL (1)`, `SPECIFIC (2)` | Forum.postType | post-type.enum.ts |
| PaymentTypeEnum | Various | StudentPayment.paymentType | payment-type.enum.ts |
| TutorPaymentTypeEnum | `DAILY`, `WEEKLY`, `MONTHLY` | TeacherPayment.paymentType | tutor-payment-type.enum.ts |
| WeekDayEnum | `MONDAY`-`SUNDAY` | ClassWeekSchedule | week-day.enum.ts |
| TimePeriodEnum | `MORNING`, `AFTERNOON`, `EVENING` | AvailableTime | time-period.enum.ts |
| CampusTypeEnum | `ONLINE`, `OFFLINE` | Enrollment | campus-type.enum.ts |
| PromoStatusEnum | Various | Promo.status | promo-status.enum.ts |
| ClassDurationEnum | Various | Class duration options | class-duration.enum.ts |
| BulkEmailStatusEnum | `PENDING`, `SENT`, `FAILED` | BulkEmail.status | bulk-email-status.enum.ts |
| BulkNotificationStatusEnum | `PENDING`, `SENT`, `FAILED` | BulkNotification.status | bulk-notification-status.enum.ts |
| CalendlyMeetingStatusEnum | `SCHEDULED`, `CANCELLED`, `COMPLETED` | Calendly.status | calendly-meeting-status.enum.ts |
| MediaTypeEnum | `VIDEO`, `DOCUMENT`, `IMAGE` | Media.type | media-type.enum.ts |

---

## Entity Specifications

### User

**Table**: `user`
**File**: `backend/src/modules/user/user.entity.ts`

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | INT | No | auto | Primary key |
| email | VARCHAR | No | - | Unique email address |
| name | VARCHAR | Yes | - | Display name |
| phone | VARCHAR | Yes | - | Phone number |
| password | VARCHAR | No | - | Bcrypt hashed (excluded from queries) |
| refresh_token | VARCHAR | Yes | - | JWT refresh token (excluded) |
| email_verification_token | VARCHAR | Yes | - | Email verification token |
| is_email_verified | BOOLEAN | No | false | Email verified status |
| role | ENUM | No | STUDENT | RolesEnum |
| image | VARCHAR | Yes | - | Profile image URL |
| cover_image | VARCHAR | Yes | - | Cover image URL |
| date_of_birth | DATE | Yes | - | Birth date |
| date_of_joining | DATE | Yes | - | Join date |
| subscription_end_date | DATE | Yes | - | Subscription expiry |
| nationality | INT | Yes | - | Nationality code |
| country_of_residency | VARCHAR | Yes | - | Country of residence |
| spoken_language | VARCHAR | Yes | - | Languages spoken |
| comments | TEXT | Yes | - | Admin comments |
| active_status | ENUM | No | ACTIVE | ActiveStatusEnum |
| payment_status | ENUM | No | NO_SHOW | PaymentStatusEnum |
| account_holder_name | VARCHAR | Yes | - | Bank account holder |
| account_number | VARCHAR | Yes | - | Bank account number |
| bank_name | VARCHAR | Yes | - | Bank name |
| branch_name | VARCHAR | Yes | - | Branch name |
| bank_address | VARCHAR | Yes | - | Bank address |
| create_date_time | TIMESTAMPTZ | No | now() | Creation timestamp |
| last_changed_date_time | TIMESTAMPTZ | No | now() | Last update |
| deleted_date_time | TIMESTAMPTZ | Yes | - | Soft delete |

**Relations**:
- `teacherClass`: OneToMany -> Class (as teacher)
- `studentClasss`: OneToMany -> ClassStudents (as student)
- `documents`: OneToMany -> Document
- `availableTimes`: ManyToMany -> AvailableTime
- `studentReviews`: OneToMany -> ClassReview
- `writerForums`: OneToMany -> Forum
- `writerInquiries`: OneToMany -> Inquiry
- `studentPayments`: OneToMany -> StudentPayment
- `teacherPayments`: OneToMany -> TeacherPayment

---

### Class

**Table**: `class`
**File**: `backend/src/modules/class/entities/class.entity.ts`

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | INT | No | auto | Primary key (BaseEntity) |
| class_number | VARCHAR | No | - | Unique class identifier |
| start_date | DATE | No | - | Class start date |
| end_date | DATE | Yes | - | Class end date |
| status | ENUM | No | ACTIVE | ClassActiveStatusEnum |
| teacher_id | INT | Yes | - | FK to User |
| class_type_id | INT | Yes | - | FK to ClassType |
| latest_textbook_id | INT | Yes | - | FK to TextBook |
| note | TEXT | No | - | Class notes |
| class_link | VARCHAR | Yes | - | Online class link |
| zoom_id | VARCHAR | Yes | - | Zoom meeting ID |
| zoom_passcode | VARCHAR | Yes | - | Zoom passcode |
| + BaseEntity fields ||||

**Relations**:
- `teacher`: ManyToOne -> User
- `classType`: ManyToOne -> ClassType
- `books`: ManyToMany -> TextBook
- `latestBook`: ManyToOne -> TextBook
- `classSchedules`: OneToMany -> ClassSchedule
- `classStudents`: OneToMany -> ClassStudents
- `classReviews`: OneToMany -> ClassReview
- `classForums`: OneToMany -> Forum
- `weekSchedules`: OneToMany -> ClassWeekSchedule

---

### Application

**Table**: `application`
**File**: `backend/src/modules/application/application.entity.ts`

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | INT | No | auto | Primary key (BaseEntity) |
| full_name | VARCHAR | No | - | Applicant name |
| email | VARCHAR | No | - | Applicant email |
| phone_number | VARCHAR | No | - | Phone number |
| nationality | INT | No | - | Nationality code |
| country_of_residency | INT | Yes | - | Residency country code |
| korean_language_level | ENUM | No | - | KoreanLanguageLevelEnum |
| status | ENUM | No | NEW | ApplicationStatusEnum |
| comment | TEXT | Yes | - | Admin comments |
| schedule_date | DATE | Yes | - | Scheduled meeting date |
| schedule_time | TIME | Yes | - | Scheduled meeting time |
| zoom_link | VARCHAR | Yes | - | Zoom meeting link |
| + BaseEntity fields ||||

**Relations**:
- `freeClassTutor`: ManyToOne -> User
- `availableTimes`: ManyToMany -> AvailableTime

---

### Forum

**Table**: `forum`
**File**: `backend/src/modules/forum/forum.entity.ts`

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | INT | No | auto | Primary key (BaseEntity) |
| title | VARCHAR | No | - | Forum post title |
| description | TEXT | No | - | Post content |
| file_urls | TEXT[] | No | {} | Attached file URLs |
| type | ENUM | Yes | SCHOOL | ForumTypeEnum |
| post_type | ENUM | No | ALL | PostTypeEnum |
| total_view | INT | No | 0 | View count |
| writer_id | INT | Yes | - | FK to User |
| class_id | INT | Yes | - | FK to Class |
| + BaseEntity fields ||||

**Relations**:
- `writer`: ManyToOne -> User
- `class`: ManyToOne -> Class
- `comments`: OneToMany -> ForumComment
- `specificUsers`: ManyToMany -> User

---

## Base Entity Fields

All entities (except User) extend `BaseEntity`:

```typescript
// backend/src/database/entities/base.ts
export abstract class BaseEntity {
    @PrimaryGeneratedColumn()
    id: number;

    @CreateDateColumn({ type: 'timestamptz' })
    createDateTime: Date;

    @Column({ nullable: true })
    createdBy: number;

    @UpdateDateColumn({ type: 'timestamptz' })
    lastChangedDateTime: Date;

    @Column({ nullable: true })
    lastChangedBy: number;

    @DeleteDateColumn({ type: 'timestamptz' })
    deletedDateTime: Date;

    @Column({ nullable: true })
    deletedBy: number;
}
```

---

## Common Query Patterns

### Find Classes with Relations

```typescript
const classes = await classRepository.find({
    relations: ['teacher', 'classType', 'classStudents', 'books'],
    where: { status: ClassActiveStatusEnum.ACTIVE },
});
```

### Find User with Enrollments

```typescript
const student = await userRepository.findOne({
    where: { id: userId },
    relations: ['studentClasss', 'studentClasss.class', 'studentPayments'],
});
```

### Find Forums by Class

```typescript
const forums = await forumRepository.find({
    relations: ['writer', 'comments', 'comments.writer'],
    where: { class: { id: classId }, type: ForumTypeEnum.CLASS },
    order: { createDateTime: 'DESC' },
});
```

### Soft Delete

```typescript
// Soft delete (sets deletedDateTime)
await repository.softDelete(id);

// Find including soft-deleted
await repository.find({ withDeleted: true });

// Restore soft-deleted
await repository.restore(id);
```

### Pagination

```typescript
const [items, total] = await repository.findAndCount({
    skip: (page - 1) * limit,
    take: limit,
    order: { createDateTime: 'DESC' },
});
```

---

## Database Configuration

**File**: `backend/src/config/db.config.ts`

```typescript
{
    type: 'postgres',
    host: process.env.POSTGRES_HOST,
    port: parseInt(process.env.POSTGRES_PORT),
    username: process.env.POSTGRES_USER,
    password: process.env.POSTGRES_PASSWORD,
    database: process.env.POSTGRES_DATABASE,
    entities: ['**/*.entity{.ts,.js}'],
    migrations: ['src/database/migrations/*{.ts,.js}'],
    namingStrategy: new SnakeNamingStrategy(),
}
```

**Naming Convention**: Snake case (e.g., `create_date_time`, `teacher_id`)

---

## Migrations

**Location**: `backend/src/database/migrations/`

**Commands**:
```bash
# Generate migration from entity changes
npm run migration:generate -- src/database/migrations/MigrationName

# Run pending migrations
npm run migration:run

# Revert last migration
npm run migration:revert
```

---

## Seeders

**Location**: `backend/src/database/seeders/`

**Command**:
```bash
npm run seed:run
```

**Default Seed Data**:
- Admin user
- Available time slots
- Initial configuration data

---

## Related Documentation

- [PROJECT_API.md](PROJECT_API.md) - API endpoints
- [PROJECT_KNOWLEDGE.md](PROJECT_KNOWLEDGE.md) - Project overview
- [BEST_PRACTICES.md](BEST_PRACTICES.md) - Database patterns
