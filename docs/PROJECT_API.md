# Project API Reference

> K Talk - Language Learning Platform API
> Last updated: 2026-01-04

## Base URL

```
http://localhost:5001/api
```

## Authentication

JWT-based authentication with access and refresh tokens.

- **Cookie**: `NestjsStartKit=<accessToken>` (set automatically after login)
- **Query**: `refreshToken` for token refresh

Endpoints marked with **Public** do not require authentication.
Endpoints marked with **Admin Only** require SUPER_ADMIN role.

## Response Format

```json
{
    "success": true,
    "message": "Operation completed successfully",
    "result": { ... },
    "total": 100
}
```

### Login Response

```json
{
    "success": true,
    "token": "refresh-token-here",
    "message": "Login successful"
}
```

### Error Response

```json
{
    "success": false,
    "message": "Error description"
}
```

---

## Modules Overview

| Module | Description | Base Route |
|--------|-------------|------------|
| [Authentication](#authentication-1) | Login, register, password management | `/auth` |
| [User](#user) | User CRUD + documents | `/user` |
| [Class](#class) | Class management | `/class` |
| [Class Type](#class-type) | Class type categories | `/class-type` |
| [Class Schedule](#class-schedule) | Class session schedules | `/class-schedule` |
| [Class Students](#class-students) | Student enrollments | `/class-students` |
| [Class Review](#class-review) | Class reviews | `/class-review` |
| [Student Attendance](#student-attendance) | Attendance tracking | `/student-attendance` |
| [Forum](#forum) | Discussion boards | `/forum` |
| [Forum Comment](#forum-comment) | Forum replies | `/forum-comment` |
| [Inquiry](#inquiry) | User inquiries | `/inquiry` |
| [Inquiry Reply](#inquiry-reply) | Inquiry responses | `/inquiry-reply` |
| [Dashboard](#dashboard) | Analytics & stats | `/dashboard` |
| [Student Payment](#student-payment) | Student payments | `/student-payment` |
| [Teacher Payment](#teacher-payment) | Teacher payments | `/teacher-payment` |
| [Application](#application) | Free class applications | `/application` |
| [Enrollment](#enrollment) | Class enrollments | `/enrollment` |
| [Available Time](#available-time) | Tutor availability | `/available-time` |
| [Calendly](#calendly) | Calendly integration | `/calendly` |
| [Blog](#blog) | Blog posts | `/blog` |
| [Blog Category](#blog-category) | Blog categories | `/blog-category` |
| [Text Book](#text-book) | Textbook management | `/text-book` |
| [Testimonial](#testimonial) | User testimonials | `/testimonial` |
| [FAQ](#faq) | Frequently asked questions | `/faq` |
| [Promo](#promo) | Promotional codes | `/promo` |
| [Student Promo](#student-promo) | Promo usage | `/student-promo` |
| [Notification](#notification) | Notifications | `/notification` |
| [Newsletter](#newsletter) | Newsletter subscriptions | `/newsletter` |
| [Contact Us](#contact-us) | Contact form | `/contact-us` |
| [Publish Tutor](#publish-tutor) | Public tutor profiles | `/publish-tutor` |
| [Publish Class Review](#publish-class-review) | Published reviews | `/publish-class-review` |
| [Media](#media) | Media content (Jeju) | `/media` |
| [Media Category](#media-category) | Media categories | `/media-category` |
| [PayPal](#paypal) | PayPal integration | `/paypal` |
| [Bulk Email](#bulk-email) | Bulk email campaigns | `/bulk-email` |
| [Bulk Notification](#bulk-notification) | Bulk notifications | `/bulk-notification` |

**Total: ~100+ endpoints across 35 modules**

---

## Authentication

### POST /auth/login

**Public**

User login with email and password.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | Yes | User email |
| password | string | Yes | User password |

**Response**: Sets `NestjsStartKit` cookie with access token, returns refresh token

---

### POST /auth/registration

**Public** (v1)

Basic registration.

**Request Body**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | Yes | User email |
| password | string | Yes | Password |
| name | string | Yes | Full name |

---

### POST /v2/auth/registration

**Public** (v2)

Registration with email verification.

**Additional Fields**: Triggers email verification flow

---

### POST /v2/auth/resend-email

**Public**

Resend verification email.

| Field | Type | Required |
|-------|------|----------|
| email | string | Yes |

---

### GET /v2/auth/verify-email

**Public**

Verify email with token.

| Query | Type | Required |
|-------|------|----------|
| token | string | Yes |

---

### POST /auth/change-password

**Requires Auth**

Change current user's password.

| Field | Type | Required |
|-------|------|----------|
| currentPassword | string | Yes |
| newPassword | string | Yes |

---

### POST /auth/forgot-password

**Public**

Request password reset email.

| Field | Type | Required |
|-------|------|----------|
| email | string | Yes |

---

### POST /auth/reset-password

**Public**

Reset password with token.

| Field | Type | Required |
|-------|------|----------|
| token | string | Yes |
| password | string | Yes |

---

### GET /auth/check-login

**Requires Auth**

Get current user information.

---

### GET /auth/refresh-access-token

**Public**

Get new access token.

| Query | Type | Required |
|-------|------|----------|
| refreshToken | string | Yes |

---

### GET /auth/logout

**Requires Auth**

Logout and clear session.

---

## User

### POST /user/create

**Admin Only**

Create new user with documents.

**Content-Type**: `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | Yes | User email |
| password | string | Yes | Password |
| name | string | Yes | Full name |
| role | number | Yes | RolesEnum |
| phone | string | No | Phone number |
| activeStatus | number | No | ActiveStatusEnum |
| dateOfBirth | date | No | Birth date |
| dateOfJoining | date | Yes | Join date |
| nationality | number | No | Nationality code |
| countryOfResidency | string | No | Country |
| spokenLanguage | string | No | Languages |
| paymentStatus | number | No | PaymentStatusEnum |
| subscriptionEndDate | date | No | Subscription end |
| accountHolderName | string | No | Bank account name |
| accountNumber | string | No | Bank account |
| bankName | string | No | Bank name |
| branchName | string | No | Branch |
| bankAddress | string | No | Bank address |
| profile | file | No | Profile image |
| coverFile | file | No | Cover image |
| documents[n].title | string | No | Document title |
| documents[n].file | file | No | Document file |

---

### PUT /user/:id

**Requires Auth**

Update user by ID.

---

### POST /user/get-all

**Requires Auth**

Get all users with filters.

| Field | Type | Description |
|-------|------|-------------|
| skip | number | Pagination offset |
| take | number | Items per page |
| role | number | Filter by role |
| activeStatus | number | Filter by status |
| search | string | Search by name/email |
| sortType | enum | Sort type |

---

### GET /user/:id

**Requires Auth**

Get user by ID (0 = current user).

---

### DELETE /user/:id

**Admin Only**

Delete user.

---

### POST /user/document-upload

**Requires Auth**

Upload user document.

---

### PUT /user/document/:id

**Requires Auth**

Update user document.

---

### DELETE /user/document/:id

**Requires Auth**

Delete user document.

---

## Class

### POST /class/create

**Requires Auth**

Create new class.

| Field | Type | Required |
|-------|------|----------|
| classNumber | string | Yes |
| startDate | date | Yes |
| endDate | date | No |
| teacherId | number | No |
| classTypeId | number | No |
| note | string | Yes |
| classLink | string | No |
| zoomId | string | No |
| zoomPasscode | string | No |
| bookIds | number[] | No |
| weekSchedules | object[] | No |

---

### POST /class/get-all

**Requires Auth**

Get all classes with filters.

---

### GET /class/:id

**Public**

Get class by ID.

---

### PUT /class/:id

**Requires Auth**

Update class.

---

### DELETE /class/:id

**Requires Auth**

Delete class.

---

### DELETE /class/:classId/:studentId

**Requires Auth**

Remove student from class.

---

## Class Type

### POST /class-type/create

**Admin Only**

Create class type.

| Field | Type | Required |
|-------|------|----------|
| title | string | Yes |
| description | string | No |

---

### POST /class-type/get-all

**Requires Auth**

Get all class types.

---

### GET /class-type/:id

**Requires Auth**

Get class type by ID.

---

### PUT /class-type/:id

**Admin Only**

Update class type.

---

### DELETE /class-type/:id

**Admin Only**

Delete class type.

---

## Class Schedule

### POST /class-schedule/create

**Requires Auth**

Create class schedule.

---

### POST /class-schedule/get-all

**Requires Auth**

Get all schedules.

---

### GET /class-schedule/:id

**Requires Auth**

Get schedule by ID.

---

### PUT /class-schedule/:id

**Requires Auth**

Update schedule.

---

### DELETE /class-schedule/:id

**Requires Auth**

Delete schedule.

---

## Forum

### POST /forum/create

**Requires Auth**

Create forum post with attachments.

**Content-Type**: `multipart/form-data`

| Field | Type | Required |
|-------|------|----------|
| title | string | Yes |
| description | string | Yes |
| type | enum | No |
| postType | enum | No |
| classId | number | No |
| files | file[] | No |

---

### POST /forum/get-all

**Requires Auth**

Get all forums for current user.

---

### POST /forum/get-all-school-forum

**Public**

Get all school-level forums.

---

### GET /forum/:id

**Public**

Get forum by ID.

---

### GET /forum/increase-view/:id

**Requires Auth**

Increment forum view count.

---

### PUT /forum/:id

**Requires Auth**

Update forum.

---

### DELETE /forum/:id

**Requires Auth**

Delete forum.

---

## Forum Comment

### POST /forum-comment/create

**Requires Auth**

Create forum comment.

| Field | Type | Required |
|-------|------|----------|
| forumId | number | Yes |
| content | string | Yes |
| replyToId | number | No |

---

### POST /forum-comment/get-all

**Requires Auth**

Get all comments.

---

### PUT /forum-comment/:id

**Requires Auth**

Update comment.

---

### DELETE /forum-comment/:id

**Requires Auth**

Delete comment.

---

## Dashboard

### POST /dashboard/admin/get-earnings

**Requires Auth**

Get admin earnings overview.

---

### GET /dashboard/admin/get-stats

**Requires Auth**

Get admin dashboard stats.

---

### POST /dashboard/tutor/get-earnings

**Requires Auth**

Get tutor earnings.

---

### GET /dashboard/tutor/get-stats

**Requires Auth**

Get tutor dashboard stats.

---

### GET /dashboard/student/get-stats

**Requires Auth**

Get student dashboard stats.

---

### GET /dashboard/student/session-stats/:id/:classId

**Requires Auth**

Get student session stats for class.

---

### GET /dashboard/landing-page/get-stats

**Public**

Get landing page statistics.

---

## Application

### POST /application/create

**Public** (Rate limited: 3/min)

Submit free class application.

| Field | Type | Required |
|-------|------|----------|
| fullName | string | Yes |
| email | string | Yes |
| phoneNumber | string | Yes |
| nationality | number | Yes |
| countryOfResidency | number | No |
| koreanLanguageLevel | enum | Yes |
| availableTimeIds | number[] | No |

---

### POST /application/get-all

**Admin Only**

Get all applications.

---

### GET /application/:id

**Admin Only**

Get application by ID.

---

### PUT /application/:id

**Admin Only**

Update application (assign tutor, schedule meeting).

---

### DELETE /application/:id

**Admin Only**

Delete application.

---

## Blog

### POST /blog/create

**Admin Only**

Create blog post with image.

**Content-Type**: `multipart/form-data`

| Field | Type | Required |
|-------|------|----------|
| title | string | Yes |
| content | string | Yes |
| categoryId | number | No |
| metaDescription | string | No |
| image | file | No |

---

### POST /blog/get-all

**Public**

Get all blogs.

---

### GET /blog/:slug

**Public**

Get blog by slug.

---

### PUT /blog/:slug

**Admin Only**

Update blog.

---

### DELETE /blog/:slug

**Admin Only**

Delete blog.

---

### POST /blog/webhook

**Public**

Webhook for external blog creation.

---

## Student Payment

### POST /student-payment/create

**Requires Auth**

Create student payment record.

---

### POST /student-payment/get-all

**Requires Auth**

Get all student payments.

---

### GET /student-payment/:id

**Requires Auth**

Get payment by ID.

---

### PUT /student-payment/:id

**Requires Auth**

Update payment.

---

### DELETE /student-payment/:id

**Requires Auth**

Delete payment.

---

## Teacher Payment

### POST /teacher-payment/create

**Requires Auth**

Create teacher payment record.

---

### POST /teacher-payment/get-all

**Requires Auth**

Get all teacher payments.

---

### GET /teacher-payment/:id

**Requires Auth**

Get payment by ID.

---

### PUT /teacher-payment/:id

**Requires Auth**

Update payment.

---

### DELETE /teacher-payment/:id

**Requires Auth**

Delete payment.

---

## Notification

### POST /notification/create

**Requires Auth**

Create notification.

---

### POST /notification/get-all

**Requires Auth**

Get all notifications for user.

---

### PUT /notification/:id

**Requires Auth**

Mark notification as read.

---

### DELETE /notification/:id

**Requires Auth**

Delete notification.

---

## Newsletter

### POST /newsletter/subscribe

**Public**

Subscribe to newsletter.

| Field | Type | Required |
|-------|------|----------|
| email | string | Yes |

---

### POST /newsletter/get-all

**Admin Only**

Get all subscribers.

---

### DELETE /newsletter/:id

**Admin Only**

Remove subscriber.

---

## Contact Us

### POST /contact-us/create

**Public**

Submit contact form.

| Field | Type | Required |
|-------|------|----------|
| name | string | Yes |
| email | string | Yes |
| subject | string | Yes |
| message | string | Yes |

---

### POST /contact-us/get-all

**Admin Only**

Get all contact submissions.

---

### DELETE /contact-us/:id

**Admin Only**

Delete contact submission.

---

## Testimonial

### POST /testimonial/create

**Requires Auth**

Create testimonial.

---

### POST /testimonial/get-all

**Public**

Get all published testimonials.

---

### PUT /testimonial/:id

**Requires Auth**

Update testimonial.

---

### DELETE /testimonial/:id

**Requires Auth**

Delete testimonial.

---

## FAQ

### POST /faq/create

**Admin Only**

Create FAQ.

| Field | Type | Required |
|-------|------|----------|
| question | string | Yes |
| answer | string | Yes |
| order | number | No |

---

### POST /faq/get-all

**Public**

Get all FAQs.

---

### PUT /faq/:id

**Admin Only**

Update FAQ.

---

### DELETE /faq/:id

**Admin Only**

Delete FAQ.

---

## Publish Tutor

### POST /publish-tutor/create

**Admin Only**

Publish tutor profile.

---

### POST /publish-tutor/get-all

**Public**

Get all published tutors.

---

### GET /publish-tutor/:id

**Public**

Get published tutor by ID.

---

### PUT /publish-tutor/:id

**Admin Only**

Update published tutor.

---

### DELETE /publish-tutor/:id

**Admin Only**

Unpublish tutor.

---

## Media

### POST /media/create

**Admin Only**

Upload media content.

---

### POST /media/get-all

**Public**

Get all media.

---

### GET /media/:id

**Public**

Get media by ID.

---

### PUT /media/:id

**Admin Only**

Update media.

---

### DELETE /media/:id

**Admin Only**

Delete media.

---

## Bulk Email

### POST /bulk-email/create

**Admin Only**

Create bulk email campaign.

---

### POST /bulk-email/get-all

**Admin Only**

Get all campaigns.

---

### POST /bulk-email/send/:id

**Admin Only**

Send bulk email.

---

### DELETE /bulk-email/:id

**Admin Only**

Delete campaign.

---

## Bulk Notification

### POST /bulk-notification/create

**Admin Only**

Create bulk notification.

---

### POST /bulk-notification/get-all

**Admin Only**

Get all bulk notifications.

---

### POST /bulk-notification/send/:id

**Admin Only**

Send bulk notification.

---

### DELETE /bulk-notification/:id

**Admin Only**

Delete bulk notification.

---

## Error Codes

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request / Validation Error |
| 401 | Unauthorized |
| 403 | Forbidden (Role not allowed) |
| 404 | Not Found |
| 409 | Conflict (e.g., duplicate) |
| 429 | Too Many Requests (Rate limit) |
| 500 | Internal Server Error |

---

## Rate Limiting

Global rate limit: 100 requests per 60 seconds.

Some endpoints have specific limits:
- Application create: 3/min
- Blog get-all: 10/min
- School forum: 20/min

---

## Role-Based Access

| Role | Value | Access |
|------|-------|--------|
| SUPER_ADMIN | 1 | Full access to all endpoints |
| TUTOR | 2 | Class, forum, payment management |
| STUDENT | 3 | View classes, forums, own data |

---

**Related Documentation:**
- [PROJECT_KNOWLEDGE.md](PROJECT_KNOWLEDGE.md) - Project overview
- [PROJECT_DATABASE.md](PROJECT_DATABASE.md) - Database schema
- [BEST_PRACTICES.md](BEST_PRACTICES.md) - Coding patterns
