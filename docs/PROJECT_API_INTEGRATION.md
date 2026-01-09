# Project API Integration

> Screen-to-API mapping for K Talk Frontends
> Last updated: 2026-01-04

This document maps frontend screens to their required backend API endpoints.

**API Reference:** See [PROJECT_API.md](PROJECT_API.md) for detailed endpoint documentation.

---

## Frontend Applications Overview

| App | Technology | Purpose | Port |
|-----|------------|---------|------|
| **Frontend-LC** | React Router 7 | Language Centre - Marketing & Enrollment | 3000 |
| **Frontend-Live** | Remix | Main Platform - Admin/Student/Teacher Dashboards | 3001 |

---

## Frontend-LC (Language Centre)

### Route Structure

```
/                     - Home page
/campus               - Campus information
/campus/:id           - Campus details
/accommodation        - Accommodation listing
/accommodation/:type/:id - Accommodation details
/programs             - Programs listing
/activities           - Activities
/about                - About page
/contact              - Contact page
/enroll               - Enrollment form
```

### Public Pages API Usage

| Page | APIs Used |
|------|-----------|
| Home | `GET /dashboard/landing-page/get-stats`, `POST /testimonial/get-all`, `POST /publish-tutor/get-all`, `POST /faq/get-all`, `POST /blog/get-all` |
| Programs | `POST /class-type/get-all` |
| Mentors | `POST /publish-tutor/get-all` |
| Blog | `POST /blog/get-all`, `GET /blog/:slug` |
| Contact | `POST /contact-us/create` |
| Enrollment | `POST /application/create`, `POST /available-time/get-all` |

---

## Frontend-Live (Main Platform)

### Route Structure

```
/                             - Landing page
/auth/login                   - Login
/auth/signup                  - Registration
/auth/forgot-password         - Password reset request
/auth/reset-password          - Password reset

/dashboard                    - Dashboard home
/dashboard/my-course          - Student courses
/dashboard/my-schedule        - Student schedule
/dashboard/forum              - Forums
/dashboard/payment            - Payments
/dashboard/profile            - User profile

# Admin Routes
/dashboard/student-management - Manage students
/dashboard/tutor-management   - Manage tutors
/dashboard/class-management   - Manage classes
/dashboard/textbook-management- Manage textbooks
/dashboard/applicant-management - Manage applications
/dashboard/payment-management - Payment tracking
/dashboard/forum-management   - Forum moderation
/dashboard/inquiry-management - Inquiry management
/dashboard/promotion          - Promotions
/dashboard/settings/*         - Blog, FAQ, Newsletter, etc.
```

---

## Auth Routes

### `/auth/login`
**Component:** `routes/auth/Login.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| User login | POST | `/auth/login` | Public |

---

### `/auth/signup`
**Component:** `routes/auth/UserSignUp.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Registration | POST | `/v2/auth/registration` | Public |

---

### `/auth/forgot-password`
**Component:** `routes/auth/ForgotPass.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Request reset | POST | `/auth/forgot-password` | Public |

---

## Dashboard - Student Routes

### `/dashboard` (Student)
**Component:** `routes/dashboard/index.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get stats | GET | `/dashboard/student/get-stats` | Required |
| Get notifications | POST | `/notification/get-all` | Required |

---

### `/dashboard/my-course`
**Component:** `routes/dashboard/my-course/MyCourse.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get enrolled classes | POST | `/class-students/get-all` | Required |
| Get class details | GET | `/class/:id` | Required |

---

### `/dashboard/forum`
**Component:** `routes/dashboard/forum/Forum.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get forums | POST | `/forum/get-all` | Required |
| Create forum | POST | `/forum/create` | Required |
| Get forum details | GET | `/forum/:id` | Required |

---

### `/dashboard/payment`
**Component:** `routes/dashboard/payment/Payment.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get payments | POST | `/student-payment/get-all` | Required |
| PayPal checkout | POST | `/paypal/create-order` | Required |

---

## Dashboard - Admin Routes

### `/dashboard` (Admin)
**Component:** `routes/dashboard/index.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get admin stats | GET | `/dashboard/admin/get-stats` | Admin |
| Get earnings | POST | `/dashboard/admin/get-earnings` | Admin |

---

### `/dashboard/student-management`
**Component:** `routes/dashboard/student-management/StudentManagement.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List students | POST | `/user/get-all` (role=3) | Admin |
| Create student | POST | `/user/create` | Admin |
| Get student | GET | `/user/:id` | Admin |
| Update student | PUT | `/user/:id` | Admin |
| Delete student | DELETE | `/user/:id` | Admin |

---

### `/dashboard/tutor-management`
**Component:** `routes/dashboard/tutor-management/TutorManagement.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List tutors | POST | `/user/get-all` (role=2) | Admin |
| Create tutor | POST | `/user/create` | Admin |
| Update tutor | PUT | `/user/:id` | Admin |
| Delete tutor | DELETE | `/user/:id` | Admin |
| Publish tutor | POST | `/publish-tutor/create` | Admin |

---

### `/dashboard/class-management`
**Component:** `routes/dashboard/class-management/AdminClassManagement.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List classes | POST | `/class/get-all` | Admin |
| Create class | POST | `/class/create` | Admin |
| Get class | GET | `/class/:id` | Admin |
| Update class | PUT | `/class/:id` | Admin |
| Delete class | DELETE | `/class/:id` | Admin |
| Get schedules | POST | `/class-schedule/get-all` | Admin |

---

### `/dashboard/applicant-management`
**Component:** `routes/dashboard/applicant-management/ApplicantManagement.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List applications | POST | `/application/get-all` | Admin |
| Get application | GET | `/application/:id` | Admin |
| Update application | PUT | `/application/:id` | Admin |
| Delete application | DELETE | `/application/:id` | Admin |

---

### `/dashboard/payment-management`
**Component:** `routes/dashboard/payment-management/PaymentManagement.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Student payments | POST | `/student-payment/get-all` | Admin |
| Teacher payments | POST | `/teacher-payment/get-all` | Admin |
| Create payment | POST | `/student-payment/create` | Admin |
| Update payment | PUT | `/student-payment/:id` | Admin |

---

### `/dashboard/settings/blog`
**Component:** `routes/dashboard/settings/blog/Blog.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List blogs | POST | `/blog/get-all` | Admin |
| Create blog | POST | `/blog/create` | Admin |
| Update blog | PUT | `/blog/:slug` | Admin |
| Delete blog | DELETE | `/blog/:slug` | Admin |
| List categories | POST | `/blog-category/get-all` | Admin |

---

### `/dashboard/settings/newsletter`
**Component:** `routes/dashboard/settings/newsletter/NewsletterMain.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List subscribers | POST | `/newsletter/get-all` | Admin |
| Send bulk email | POST | `/bulk-email/send/:id` | Admin |

---

### `/dashboard/settings/add-FAQ`
**Component:** `routes/dashboard/settings/add-FAQ/AddFAQ.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| List FAQs | POST | `/faq/get-all` | Admin |
| Create FAQ | POST | `/faq/create` | Admin |
| Update FAQ | PUT | `/faq/:id` | Admin |
| Delete FAQ | DELETE | `/faq/:id` | Admin |

---

## Dashboard - Tutor Routes

### `/dashboard` (Tutor)
**Component:** `routes/dashboard/index.tsx`

| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get stats | GET | `/dashboard/tutor/get-stats` | Required |
| Get earnings | POST | `/dashboard/tutor/get-earnings` | Required |

---

### `/dashboard/my-course` (Tutor)
| Action | Method | Endpoint | Auth |
|--------|--------|----------|------|
| Get assigned classes | POST | `/class/get-all` | Required |
| Update class | PUT | `/class/:id` | Required |
| Get attendance | POST | `/student-attendance/get-all` | Required |
| Mark attendance | POST | `/student-attendance/create` | Required |

---

## Shared APIs

### Authentication (All Protected Routes)

| Action | Method | Endpoint |
|--------|--------|----------|
| Check login | GET | `/auth/check-login` |
| Refresh token | GET | `/auth/refresh-access-token?refreshToken=xxx` |
| Logout | GET | `/auth/logout` |

### User Profile

| Action | Method | Endpoint |
|--------|--------|----------|
| Get profile | GET | `/user/:id` (id=0 for current) |
| Update profile | PUT | `/user/:id` |

### Notifications

| Action | Method | Endpoint |
|--------|--------|----------|
| Get notifications | POST | `/notification/get-all` |
| Mark as read | PUT | `/notification/:id` |

---

## HTTP Service Structure (Frontend-Live)

```
services/
├── httpService.ts                    # Axios instance with interceptors
└── httpService/
    ├── authHttpService.ts            # Auth API calls
    ├── adminDashboardHttpServices.ts # Admin dashboard APIs
    ├── studentDashboardHttpServices.ts # Student dashboard APIs
    ├── classManagementHttpService.ts # Class CRUD
    ├── forumHttpService.ts           # Forum APIs
    ├── inquiryHttpService.ts         # Inquiry APIs
    ├── paymentHttpService.ts         # Payment APIs
    ├── blogHttpService.ts            # Blog APIs
    ├── mediaHttpService.ts           # Media APIs
    ├── notificationHttpService.ts    # Notification APIs
    ├── applicantHttpService.ts       # Application APIs
    ├── testimonialHttpService.ts     # Testimonial APIs
    └── ...40+ service files
```

---

## Redux State Structure (Frontend-Live)

```
redux/features/
├── auth/
│   ├── userCheckSlice.ts           # Auth state
│   └── storageSlice.ts             # Persistent storage
└── dashboard/
    ├── adminDashboardSlice.ts      # Admin stats
    ├── studentDashboardSlice.ts    # Student stats
    ├── classManagementSlice.ts     # Classes
    ├── forumSlice.ts               # Forums
    ├── inquirySlice.ts             # Inquiries
    ├── blogSlice.ts                # Blogs
    ├── notificationSlice.ts        # Notifications
    └── ...35+ slices
```

---

## Implementation Pattern

### 1. HTTP Service (Async Thunk)

```typescript
// services/httpService/classHttpService.ts
import { createAsyncThunk } from '@reduxjs/toolkit';
import httpService from '../httpService';

export const getClassesHttpService = createAsyncThunk(
    'class/get-all',
    async (dto: GetAllClassDto, { rejectWithValue }) => {
        try {
            const { data } = await httpService.post('/class/get-all', dto);
            return data;
        } catch (error) {
            return rejectWithValue(error.response?.data);
        }
    }
);
```

### 2. Redux Slice

```typescript
// redux/features/dashboard/classManagementSlice.ts
const classSlice = createSlice({
    name: 'classManagement',
    initialState,
    reducers: {},
    extraReducers: (builder) => {
        builder
            .addCase(getClassesHttpService.pending, (state) => {
                state.loading = true;
            })
            .addCase(getClassesHttpService.fulfilled, (state, action) => {
                state.loading = false;
                state.classes = action.payload.result;
                state.total = action.payload.total;
            });
    },
});
```

### 3. Component Usage

```typescript
// In component
const dispatch = useAppDispatch();
const { classes, loading } = useAppSelector((state) => state.classManagement);

useEffect(() => {
    dispatch(getClassesHttpService({ skip: 0, take: 10 }));
}, [dispatch]);
```

---

**Related Documentation:**

- [PROJECT_API.md](PROJECT_API.md) - Backend API reference
- [PROJECT_DATABASE.md](PROJECT_DATABASE.md) - Database schema
- [BEST_PRACTICES.md](BEST_PRACTICES.md) - React patterns
