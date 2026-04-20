# 📘 RoamTrip API Documentation

> **Base URL:** `http://localhost:8080`  
> **Content-Type:** `application/json`  
> **Authentication:** Bearer Token (JWT) via header `Authorization: Bearer <accessToken>`

---

## 📑 Table of Contents

1. [Authentication](#1-authentication)
2. [User Management](#2-user-management)
3. [Department](#3-department)
4. [Project](#4-project)
5. [Task](#5-task)
6. [Time Log](#6-time-log)
7. [Skill](#7-skill)
8. [Employee Skill](#8-employee-skill)
9. [Notification](#9-notification)
10. [Workload](#10-workload)
11. [API Flows by Use Case](#11-api-flows-by-use-case)
12. [Enum Reference](#12-enum-reference)
13. [Response Wrapper](#13-response-wrapper)

---

## 1. Authentication

> Base path: `/api/auth`  
> Most endpoints in this group **do not require** a token.

---

### 1.1 Register Account
**`POST /api/auth/register`**

Submit a registration request. The system will send a verification email to the registered email address.

**Request Body:**
```json
{
  "email": "user@example.com",    // required, valid email format
  "password": "Secret123",        // required
  "name": "Nguyen Van A"          // required, max 255 characters
}
```

**Response:** `202 Accepted`
```json
"Registration successful. Please check your email to verify your account."
```

---

### 1.2 Verify Email (via link in email)
**`GET /api/auth/verify-email?token=<token>`**

User clicks the link in the email → backend redirects to frontend `/login?verified=true`.

**Query Params:**
| Param | Required | Description |
|-------|----------|-------------|
| `token` | ✅ | Verification token sent in the email |

**Response:** `302 Found` → redirect to `{frontendBaseUrl}/login?verified=true`

---

### 1.3 Login
**`POST /api/auth/login`**

**Request Body:**
```json
{
  "email": "user@example.com",  // required
  "password": "Secret123"       // required
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600,
  "user": {
    "id": 1,
    "email": "user@example.com",
    "username": "nguyenvana",
    "fullName": "Nguyen Van A",
    "phone": "0901234567",
    "avatarUrl": null,
    "departmentId": 2,
    "position": "Developer",
    "role": "TEAM_MEMBER",
    "verified": true,
    "active": true
  }
}
```

---

### 1.4 Refresh Access Token
**`POST /api/auth/refresh`**

Use the `refreshToken` to obtain a new `accessToken` when the current one expires.

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** `200 OK` → same structure as the Login response (`AuthResponse`)

---

### 1.5 Logout
**`POST /api/auth/logout`** 🔒 _(Requires token)_

Invalidates the current session.

**Response:** `200 OK`
```json
"Logged out"
```

---

### 1.6 Send Email Verification OTP
**`POST /api/auth/send-otp`**

Sends a 6-digit OTP code to the provided email address for verification.

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response:** `200 OK` `"OTP sent successfully"`

---

### 1.7 Verify OTP
**`POST /api/auth/verify-email-otp`**

**Request Body:**
```json
{
  "email": "user@example.com",
  "otp": "123456"   // 6-digit code
}
```

**Response:** `200 OK` `"Email verified successfully"`

---

### 1.8 Forgot Password
**`POST /api/auth/forgot-password`**

Sends a password reset link to the provided email address.

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response:** `200 OK` `"Password reset email sent"`

---

### 1.9 Reset Password
**`POST /api/auth/reset-password`**

**Request Body:**
```json
{
  "token": "<reset_token_from_email>",
  "newPassword": "NewSecret123"   // min 8 characters, must contain both letters and numbers
}
```

**Response:** `200 OK` `"Password reset successful"`

---

## 2. User Management

> Base path: `/api/users`  
> 🔒 All endpoints require a Bearer Token.

---

### 2.1 Get Current User Profile
**`GET /api/users/me`**

**Response:** `200 OK`
```json
{
  "id": 1,
  "email": "user@example.com",
  "username": "nguyenvana",
  "fullName": "Nguyen Van A",
  "phone": "0901234567",
  "avatarUrl": null,
  "departmentId": 2,
  "position": "Senior Developer",
  "role": "PROJECT_MANAGER",
  "verified": true,
  "active": true
}
```

---

### 2.2 Update Own Profile
**`POST /api/users/me/update`**

**Request Body:**
```json
{
  "name": "Nguyen Van B"   // max 255 characters
}
```

**Response:** `200 OK` → `UserMeResponse`

---

### 2.3 Change Password
**`POST /api/users/me/password`**

**Request Body:**
```json
{
  "currentPassword": "OldSecret123",
  "newPassword": "NewSecret456"   // min 8 characters, must contain both letters and numbers
}
```

**Response:** `200 OK` `"Password changed successfully"`

---

### 2.4 Create New User (Admin)
**`POST /api/users`** 🛡️ _ADMIN only_

**Request Body:**
```json
{
  "email": "newuser@example.com",       // required
  "password": "Secret123",              // required, min 8 characters
  "fullName": "Tran Thi B",            // required, max 100 characters
  "username": "tranthib",              // optional, max 100 characters
  "phone": "0987654321",               // optional, max 20 characters
  "departmentId": 2,                   // optional
  "position": "Junior Developer",      // optional, max 100 characters
  "role": "TEAM_MEMBER"               // default: TEAM_MEMBER
}
```

**Response:** `201 Created` → `UserMeResponse`

---

### 2.5 Update User (Admin / HR)
**`PUT /api/users/{id}`** 🛡️ _ADMIN or HR_

> HR cannot change the `role` field.

**Path Params:** `id` - ID of the user to update

**Request Body:**
```json
{
  "fullName": "Tran Thi B Updated",
  "username": "tranthib_new",
  "phone": "0111222333",
  "departmentId": 3,
  "position": "Senior Developer",
  "role": "PROJECT_MANAGER"   // Only ADMIN can change this
}
```

**Response:** `200 OK` → `UserMeResponse`

---

### 2.6 Get All Users
**`GET /api/users`** 🛡️ _ADMIN or HR_

**Response:** `200 OK`
```json
[
  { /* UserMeResponse */ },
  { /* UserMeResponse */ }
]
```

---

### 2.7 Get User by ID
**`GET /api/users/{id}`** 🛡️ _ADMIN, HR, PROJECT_MANAGER_

**Response:** `200 OK` → `UserMeResponse`

---

### 2.8 Activate User Account
**`POST /api/users/{id}/activate`** 🛡️ _ADMIN only_

**Response:** `200 OK` → `UserMeResponse` with `active: true`

---

### 2.9 Deactivate User Account
**`POST /api/users/{id}/deactivate`** 🛡️ _ADMIN only_

**Response:** `200 OK` → `UserMeResponse` with `active: false`

---

## 3. Department

> Base path: `/api/departments`  
> 🔒 Requires Bearer Token.

---

### 3.1 Get All Departments
**`GET /api/departments`**

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "name": "Engineering",
    "description": "Software engineering team",
    "managerId": 5,
    "createdAt": "2025-01-01T00:00:00"
  }
]
```

---

### 3.2 Get Department by ID
**`GET /api/departments/{id}`**

**Response:** `200 OK` → `DepartmentResponse`

---

### 3.3 Create Department
**`POST /api/departments`** 🛡️ _ADMIN or HR_

**Request Body:**
```json
{
  "name": "Marketing",         // required, max 100 characters
  "description": "Marketing team",
  "managerId": 3               // optional
}
```

**Response:** `201 Created` → `DepartmentResponse`

---

### 3.4 Update Department
**`POST /api/departments/{id}/update`** 🛡️ _ADMIN or HR_

**Request Body:** same structure as Create Department

**Response:** `200 OK` → `DepartmentResponse`

---

### 3.5 Delete Department
**`POST /api/departments/{id}/delete`** 🛡️ _ADMIN only_

**Response:** `200 OK`

---

## 4. Project

> Base path: `/api/projects`  
> 🔒 Requires Bearer Token.

---

### 4.1 Get All Projects
**`GET /api/projects`**

Can be filtered by status.

**Query Params:**
| Param | Required | Description |
|-------|----------|-------------|
| `status` | ❌ | Filter by status: `PLANNING`, `IN_PROGRESS`, `ON_HOLD`, `COMPLETED`, `CANCELLED` |

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "name": "RoamTrip Platform",
    "code": "RTP",
    "description": "Main platform project",
    "status": "IN_PROGRESS",
    "priority": "HIGH",
    "startDate": "2025-01-01",
    "endDate": "2025-12-31",
    "actualEndDate": null,
    "managerId": 2,
    "createdAt": "2025-01-01T00:00:00"
  }
]
```

---

### 4.2 Get Project by ID
**`GET /api/projects/{id}`**

**Response:** `200 OK` → `ProjectResponse`

---

### 4.3 Create Project
**`POST /api/projects`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Request Body:**
```json
{
  "name": "New Project",          // required, max 200 characters
  "code": "NP001",                // optional, max 50 characters
  "description": "Project desc",
  "status": "PLANNING",           // default: PLANNING
  "priority": "HIGH",             // default: MEDIUM
  "startDate": "2025-03-01",
  "endDate": "2025-09-30",
  "managerId": 2                  // required
}
```

**Response:** `201 Created` → `ProjectResponse`

---

### 4.4 Update Project
**`POST /api/projects/{id}/update`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Request Body:** same structure as Create Project

**Response:** `200 OK` → `ProjectResponse`

---

### 4.5 Delete Project
**`POST /api/projects/{id}/delete`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Response:** `200 OK`

---

### 4.6 Get Project Members
**`GET /api/projects/{id}/members`**

**Response:** `200 OK`
```json
[
  {
    "id": 10,
    "projectId": 1,
    "userId": 5,
    "userFullName": "Nguyen Van A",
    "roleInProject": "DEVELOPER",
    "allocatedEffortPercent": 80,
    "joinDate": "2025-01-15",
    "leaveDate": null,
    "note": "Lead developer"
  }
]
```

---

### 4.7 Add Member to Project
**`POST /api/projects/{id}/members`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Request Body:**
```json
{
  "userId": 5,                     // required
  "roleInProject": "DEVELOPER",   // default: MEMBER
  "allocatedEffortPercent": 80,   // 0-100
  "joinDate": "2025-01-15",
  "note": "Lead developer"
}
```

**Response:** `201 Created` → `ProjectMemberResponse`

---

### 4.8 Update Member Information
**`POST /api/projects/{id}/members/{memberId}/update`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Request Body:** same structure as Add Member

**Response:** `200 OK` → `ProjectMemberResponse`

---

### 4.9 Remove Member from Project
**`POST /api/projects/{id}/members/{memberId}/delete`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Response:** `200 OK`

---

## 5. Task

> Base paths: `/api/tasks` and `/api/projects/{projectId}/tasks`  
> 🔒 Requires Bearer Token.

---

### 5.1 Get Tasks of a Project (nested route)
**`GET /api/projects/{projectId}/tasks`**

**Response:** `200 OK` → `List<TaskResponse>`

---

### 5.2 Get Tasks Assigned to Me
**`GET /api/tasks/my`**

**Response:** `200 OK` → `List<TaskResponse>`

---

### 5.3 Get Task by ID
**`GET /api/tasks/{id}`**

**Response:** `200 OK`
```json
{
  "id": 1,
  "projectId": 1,
  "title": "Implement login screen",
  "description": "Build the login UI",
  "type": "FEATURE",
  "status": "IN_PROGRESS",
  "priority": "HIGH",
  "estimatedHours": 8.0,
  "actualHours": 3.5,
  "startDate": "2025-02-01",
  "dueDate": "2025-02-07",
  "completedAt": null,
  "assigneeId": 5,
  "assigneeName": "Nguyen Van A",
  "reporterId": 2,
  "skillRequirements": [
    {
      "id": 1,
      "skillId": 3,
      "skillName": "React",
      "minimumLevel": "INTERMEDIATE",
      "isRequired": true
    }
  ],
  "createdAt": "2025-01-20T10:00:00"
}
```

---

### 5.4 Create Task
**`POST /api/tasks`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Request Body:**
```json
{
  "projectId": 1,              // required
  "title": "Implement login",  // required, max 300 characters
  "description": "Build login UI with JWT support",
  "type": "FEATURE",           // default: FEATURE
  "priority": "HIGH",          // default: MEDIUM
  "estimatedHours": 8.0,
  "startDate": "2025-02-01",
  "dueDate": "2025-02-07",
  "assigneeId": 5,
  "skillRequirements": [
    {
      "skillId": 3,           // required
      "minimumLevel": "INTERMEDIATE",  // default: INTERMEDIATE
      "isRequired": true      // default: true
    }
  ]
}
```

**Response:** `201 Created` → `TaskResponse`

---

### 5.5 Update Task
**`POST /api/tasks/{id}/update`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Request Body:** same structure as Create Task

**Response:** `200 OK` → `TaskResponse`

---

### 5.6 Update Task Status
**`POST /api/tasks/{id}/status`** _(Any authenticated user)_

**Request Body:**
```json
{
  "status": "IN_PROGRESS",  // required: TODO, IN_PROGRESS, IN_REVIEW, DONE, BLOCKED
  "note": "Starting implementation"
}
```

**Response:** `200 OK` → `TaskResponse`

---

### 5.7 Delete Task
**`POST /api/tasks/{id}/delete`** 🛡️ _ADMIN or PROJECT_MANAGER_

**Response:** `200 OK`

---

### 5.8 Get Task Status Change History
**`GET /api/tasks/{id}/history`**

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "oldStatus": "TODO",
    "newStatus": "IN_PROGRESS",
    "changedBy": 5,
    "note": "Starting implementation",
    "changedAt": "2025-02-01T09:00:00"
  }
]
```

---

## 6. Time Log

> Base path: `/api/time-logs`  
> 🔒 Requires Bearer Token.

---

### 6.1 Get My Time Logs
**`GET /api/time-logs/my`**

**Query Params:**
| Param | Required | Description |
|-------|----------|-------------|
| `from` | ❌ | Start date filter (format: `YYYY-MM-DD`) |
| `to` | ❌ | End date filter (format: `YYYY-MM-DD`) |

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "taskId": 10,
    "taskTitle": "Implement login screen",
    "userId": 5,
    "logDate": "2025-02-03",
    "hoursSpent": 4.5,
    "description": "Completed UI layout",
    "createdAt": "2025-02-03T17:00:00"
  }
]
```

---

### 6.2 Get Time Logs by Task
**`GET /api/time-logs/task/{taskId}`**

**Response:** `200 OK` → `List<TimeLogResponse>`

---

### 6.3 Log Work Time
**`POST /api/time-logs`**

**Request Body:**
```json
{
  "taskId": 10,             // required
  "logDate": "2025-02-03", // required
  "hoursSpent": 4.5,       // required, 0.1 - 24.0
  "description": "Completed UI layout"
}
```

**Response:** `201 Created` → `TimeLogResponse`

---

### 6.4 Update Time Log
**`POST /api/time-logs/{id}/update`**

**Request Body:** same structure as Create Time Log

**Response:** `200 OK` → `TimeLogResponse`

---

### 6.5 Delete Time Log
**`POST /api/time-logs/{id}/delete`**

**Response:** `200 OK`

---

## 7. Skill

> Base path: `/api/skills`  
> 🔒 Requires Bearer Token.

---

### 7.1 Get All Skills
**`GET /api/skills`**

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "name": "React",
    "category": "Frontend",
    "description": "React.js framework"
  }
]
```

---

### 7.2 Get Skill by ID
**`GET /api/skills/{id}`**

**Response:** `200 OK` → `SkillResponse`

---

### 7.3 Create Skill
**`POST /api/skills`** 🛡️ _ADMIN or HR_

**Request Body:**
```json
{
  "name": "TypeScript",     // required, max 100 characters
  "category": "Frontend",  // optional, max 100 characters
  "description": "TypeScript programming language"
}
```

**Response:** `201 Created` → `SkillResponse`

---

### 7.4 Update Skill
**`POST /api/skills/{id}/update`** 🛡️ _ADMIN or HR_

**Request Body:** same structure as Create Skill

**Response:** `200 OK` → `SkillResponse`

---

### 7.5 Delete Skill
**`POST /api/skills/{id}/delete`** 🛡️ _ADMIN only_

**Response:** `200 OK`

---

## 8. Employee Skill

> Base path: `/api/users`  
> 🔒 Requires Bearer Token.

---

### 8.1 Get Skills of a Specific User
**`GET /api/users/{userId}/skills`** 🛡️ _Any authenticated user_

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "userId": 5,
    "skillId": 3,
    "skillName": "React",
    "skillCategory": "Frontend",
    "level": "INTERMEDIATE",
    "yearsOfExperience": 2.5,
    "note": "Used in 3 projects"
  }
]
```

---

### 8.2 Get My Skills
**`GET /api/users/me/skills`**

**Response:** `200 OK` → `List<EmployeeSkillResponse>`

---

### 8.3 Add Skill to My Profile
**`POST /api/users/me/skills`**

**Request Body:**
```json
{
  "skillId": 3,             // required
  "level": "INTERMEDIATE", // required: BEGINNER, INTERMEDIATE, ADVANCED, EXPERT
  "yearsOfExperience": 2.5,
  "note": "Used in 3 projects"
}
```

**Response:** `201 Created` → `EmployeeSkillResponse`

---

### 8.4 Update My Skill
**`POST /api/users/me/skills/{skillId}/update`**

**Request Body:** same structure as Add My Skill

**Response:** `200 OK` → `EmployeeSkillResponse`

---

### 8.5 Remove My Skill
**`POST /api/users/me/skills/{skillId}/delete`**

**Response:** `200 OK`

---

### 8.6 Add Skill for a User (Admin/HR)
**`POST /api/users/{userId}/skills`** 🛡️ _ADMIN or HR_

**Request Body:** same structure as Add My Skill

**Response:** `201 Created` → `EmployeeSkillResponse`

---

## 9. Notification

> Base path: `/api/notifications`  
> 🔒 Requires Bearer Token.  
> All endpoints automatically fetch notifications for the currently authenticated user.

---

### 9.1 Get All Notifications
**`GET /api/notifications`**

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "type": "TASK_ASSIGNED",
    "title": "New task assigned",
    "content": "You have been assigned to task: Implement login",
    "referenceType": "TASK",
    "referenceId": 10,
    "isRead": false,
    "readAt": null,
    "createdAt": "2025-02-01T09:00:00"
  }
]
```

---

### 9.2 Get Unread Notifications
**`GET /api/notifications/unread`**

**Response:** `200 OK` → `List<NotificationResponse>` (only unread notifications)

---

### 9.3 Count Unread Notifications
**`GET /api/notifications/unread/count`**

**Response:** `200 OK`
```json
{
  "count": 5
}
```

---

### 9.4 Mark Notification as Read
**`POST /api/notifications/{id}/read`**

**Response:** `200 OK` → `NotificationResponse` (with `isRead: true`, `readAt: <timestamp>`)

---

### 9.5 Mark All Notifications as Read
**`POST /api/notifications/read-all`**

**Response:** `204 No Content`

---

## 10. Workload

> Base path: `/api/workload`  
> 🔒 Requires Bearer Token.

---

### 10.1 View Team Workload
**`GET /api/workload/team`** 🛡️ _ADMIN, HR, PROJECT_MANAGER_

**Query Params:**
| Param | Required | Description |
|-------|----------|-------------|
| `date` | ❌ | Snapshot date to view (format: `YYYY-MM-DD`). Defaults to the latest date. |

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "userId": 5,
    "userFullName": "Nguyen Van A",
    "snapshotDate": "2025-02-01",
    "totalAllocatedHours": 40.0,
    "totalActualHours": 35.5,
    "capacityUsedPercent": 88.75,
    "projectCount": 2,
    "activeTaskCount": 5
  }
]
```

---

### 10.2 View My Workload
**`GET /api/workload/me`**

**Response:** `200 OK` → `List<WorkloadSnapshotResponse>` (historical snapshots)

---

### 10.3 View Workload of a Specific User
**`GET /api/workload/users/{userId}`** 🛡️ _ADMIN, HR, PROJECT_MANAGER_

**Response:** `200 OK` → `List<WorkloadSnapshotResponse>`

---

### 10.4 Manually Take Workload Snapshot
**`POST /api/workload/snapshot?userId={userId}`** 🛡️ _ADMIN or HR_

**Query Params:**
| Param | Required | Description |
|-------|----------|-------------|
| `userId` | ✅ | ID of the user to snapshot |

**Response:** `200 OK` → `WorkloadSnapshotResponse`

---

### 10.5 View Burnout Logs (System-wide)
**`GET /api/workload/burnout`** 🛡️ _ADMIN, HR, PROJECT_MANAGER_

**Response:** `200 OK`
```json
[
  {
    "id": 1,
    "userId": 5,
    "userFullName": "Nguyen Van A",
    "evaluatedAt": "2025-02-01T08:00:00",
    "riskScore": 78,
    "riskLevel": "HIGH",
    "overdueTaskCount": 3,
    "capacityUsedAvg": 94.5,
    "isAlertSent": true,
    "note": "Overloaded for 2 weeks"
  }
]
```

---

### 10.6 View Burnout History of a Specific User
**`GET /api/workload/burnout/users/{userId}`** 🛡️ _ADMIN, HR, PROJECT_MANAGER_

**Response:** `200 OK` → `List<BurnoutLogResponse>`

---

## 11. API Flows by Use Case

### 🔐 Flow 1: Register & Verify Account

```
1. POST /api/auth/register          → Register account
2. [User clicks link in email]    
   GET  /api/auth/verify-email?token=xxx → Verify email
   [Or use OTP]
   POST /api/auth/send-otp          → Send OTP
   POST /api/auth/verify-email-otp  → Verify OTP
3. POST /api/auth/login             → Login, retrieve accessToken + refreshToken
4. [Store token in localStorage/cookie]
```

---

### 🔄 Flow 2: Refresh Token

```
1. [accessToken expires → 401 response]
2. POST /api/auth/refresh { refreshToken } → Get new accessToken
3. [Retry original request with new accessToken]
4. [If refreshToken also expired → redirect to /login]
```

---

### 🔑 Flow 3: Forgot Password

```
1. POST /api/auth/forgot-password { email }       → Send reset email
2. [User clicks link in email]
3. POST /api/auth/reset-password { token, newPassword } → Set new password
4. POST /api/auth/login                           → Log in again
```

---

### 👤 Flow 4: Employee Management (Admin/HR)

```
1. GET  /api/departments               → Get department list
2. POST /api/users { ... }             → Create new employee (Admin)
3. GET  /api/users                     → List all employees
4. PUT  /api/users/{id} { ... }        → Update employee info
5. POST /api/users/{id}/activate       → Activate account
   POST /api/users/{id}/deactivate     → Deactivate account
```

---

### 📁 Flow 5: Project Management (Project Manager)

```
1. POST /api/projects { ... }                          → Create new project
2. GET  /api/users                                     → Get employee list
3. POST /api/projects/{id}/members { userId, role }    → Add member to project
4. GET  /api/skills                                    → Get skill list
5. POST /api/tasks { projectId, title, skillRequirements } → Create task and assign to member
6. GET  /api/projects/{id}/members                     → Check project members
7. GET  /api/projects/{id}/tasks                       → View project tasks
```

---

### ✅ Flow 6: Do Work (Team Member)

```
1. GET  /api/tasks/my             → View assigned tasks
2. GET  /api/tasks/{id}           → View task details
3. POST /api/tasks/{id}/status { status: "IN_PROGRESS" } → Start working
4. POST /api/time-logs { taskId, logDate, hoursSpent }   → Log work time
5. POST /api/tasks/{id}/status { status: "IN_REVIEW" }   → Move to review
6. POST /api/tasks/{id}/status { status: "DONE" }        → Mark as complete
```

---

### 📊 Flow 7: Workload Monitoring & Management (Admin/HR/PM)

```
1. GET  /api/workload/team?date=2025-02-01   → View team workload by date
2. GET  /api/workload/users/{userId}         → View workload history of a specific employee
3. GET  /api/workload/burnout                → View burnout warning list
4. POST /api/workload/snapshot?userId={id}   → Manually take snapshot when needed
```

---

### 🔔 Flow 8: Notifications (Real-time polling)

```
1. GET  /api/notifications/unread/count → Poll periodically (e.g., every 30s) to check for new notifications
2. GET  /api/notifications/unread       → When new notifications exist, fetch the list
3. POST /api/notifications/{id}/read   → When user clicks a notification, mark as read
4. POST /api/notifications/read-all    → When user clicks "mark all as read"
```

---

### 💼 Flow 9: Employee Skill Management

```
[Admin/HR manages the skill catalog]
1. POST /api/skills { name, category }      → Create new skill
2. GET  /api/skills                         → Get skill list

[Employee updates personal skills]
3. GET  /api/users/me/skills                → View current skills
4. POST /api/users/me/skills { skillId, level } → Add skill
5. POST /api/users/me/skills/{skillId}/update { level } → Update skill level
```

---

## 12. Enum Reference

### UserRole
| Value | Description |
|-------|-------------|
| `ADMIN` | System administrator |
| `HR` | Human resources |
| `PROJECT_MANAGER` | Project manager |
| `TEAM_MEMBER` | Team member |

### ProjectStatus
| Value | Description |
|-------|-------------|
| `PLANNING` | Planning phase |
| `IN_PROGRESS` | In progress |
| `ON_HOLD` | On hold |
| `COMPLETED` | Completed |
| `CANCELLED` | Cancelled |

### ProjectPriority
| Value | Description |
|-------|-------------|
| `LOW` | Low |
| `MEDIUM` | Medium |
| `HIGH` | High |
| `CRITICAL` | Critical |

### ProjectRoleInProject
| Value | Description |
|-------|-------------|
| `MANAGER` | Project manager |
| `TECH_LEAD` | Technical lead |
| `DEVELOPER` | Developer |
| `DESIGNER` | Designer |
| `QA` | Quality assurance |
| `MEMBER` | Member |

### TaskStatus
| Value | Description |
|-------|-------------|
| `TODO` | Not started |
| `IN_PROGRESS` | In progress |
| `IN_REVIEW` | Under review |
| `DONE` | Done |
| `BLOCKED` | Blocked |

### TaskType
| Value | Description |
|-------|-------------|
| `FEATURE` | New feature |
| `BUG` | Bug fix |
| `IMPROVEMENT` | Improvement |
| `RESEARCH` | Research |
| `TASK` | General task |

### TaskPriority
| Value | Description |
|-------|-------------|
| `LOW` | Low |
| `MEDIUM` | Medium |
| `HIGH` | High |
| `CRITICAL` | Critical |

### SkillLevel
| Value | Description |
|-------|-------------|
| `BEGINNER` | Beginner |
| `INTERMEDIATE` | Intermediate |
| `ADVANCED` | Advanced |
| `EXPERT` | Expert |

### RiskLevel (Burnout)
| Value | Description |
|-------|-------------|
| `LOW` | Low risk |
| `MEDIUM` | Medium risk |
| `HIGH` | High risk |
| `CRITICAL` | Critical |

---

## 13. Response Wrapper

All API responses are wrapped by `ApiResponse`:

```json
{
  "code": 200,
  "message": "Success",
  "data": { /* actual data */ }
}
```

### Error Response
```json
{
  "code": 400,
  "message": "Validation failed",
  "data": null
}
```

### Common HTTP Status Codes
| Status | Meaning |
|--------|---------|
| `200 OK` | Success |
| `201 Created` | Created successfully |
| `202 Accepted` | Request accepted |
| `204 No Content` | Success, no data returned |
| `302 Found` | Redirect |
| `400 Bad Request` | Invalid request / Validation error |
| `401 Unauthorized` | Not logged in or token expired |
| `403 Forbidden` | Access denied |
| `404 Not Found` | Resource not found |
| `500 Internal Server Error` | Server error |

---

## 🔐 Permission Overview

| API Group | PUBLIC | TEAM_MEMBER | HR | PROJECT_MANAGER | ADMIN |
|-----------|:------:|:-----------:|:--:|:---------------:|:-----:|
| Auth (register, login...) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Personal profile | ❌ | ✅ | ✅ | ✅ | ✅ |
| User management | ❌ | ❌ | ✅ (cannot change role) | ❌ | ✅ |
| Department CRUD | ❌ | ❌ | ✅ | ❌ | ✅ |
| Project CRUD | ❌ | ❌ | ❌ | ✅ | ✅ |
| Project members | ❌ | 👁️ | ❌ | ✅ | ✅ |
| Task CRUD | ❌ | ❌ | ❌ | ✅ | ✅ |
| Task status update | ❌ | ✅ | ✅ | ✅ | ✅ |
| Time logs | ❌ | ✅ | ✅ | ✅ | ✅ |
| Skill master data | ❌ | 👁️ | ✅ | 👁️ | ✅ |
| Employee skills | ❌ | ✅ (of self) | ✅ | 👁️ | ✅ |
| Notifications | ❌ | ✅ | ✅ | ✅ | ✅ |
| Workload reports | ❌ | 👁️ (of self) | ✅ | ✅ | ✅ |

> **Legend:** ✅ Full access | 👁️ Read-only | ❌ No access
