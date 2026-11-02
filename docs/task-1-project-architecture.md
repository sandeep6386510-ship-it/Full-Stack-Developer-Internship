# Task 1 — Project Architecture

## 1. Product Idea

**Product name:** FocusFlow  
**Type:** Full-stack student productivity web application

FocusFlow helps college students organize assignments and daily tasks, schedule focus sessions, and review their progress in one place.

### Problem
Students commonly track assignments, study plans, and deadlines across different apps or notes. This makes it easy to miss due dates and difficult to measure study consistency.

### Proposed solution
A simple dashboard where a student can create and prioritize tasks, mark work complete, run a focus timer, and view daily/weekly progress.

### Target users
College students who need a lightweight task and study-planning tool.

### MVP features
- Create, view, edit, and delete tasks
- Assign due date, priority, and status
- Start a focus session for a task
- Track completed tasks and focus minutes
- Responsive dashboard

Out of scope for the first version: social features, payments, AI recommendations, and multi-user collaboration.

## 2. Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React + Vite | Component-based UI |
| Styling | CSS modules / plain CSS | Responsive design |
| Backend | Node.js + Express | REST API and business logic |
| Database | PostgreSQL | Persistent relational data |
| Validation | Zod or express-validator | Validate API inputs |
| Authentication | JWT + password hashing | Secure user sessions (planned) |
| Testing | Vitest, Supertest | Frontend and API tests |
| Deployment | Vercel (frontend), Render (API), managed PostgreSQL | Hosting (planned) |

## 3. High-Level Architecture

```text
Student
  |
  v
React Web Client (pages, components, state)
  | HTTPS / JSON REST API
  v
Node.js + Express API
  |-- Authentication middleware
  |-- Request validation
  |-- Task service
  |-- Focus-session service
  |
  v
PostgreSQL Database
```

### Request flow
1. The user interacts with a React screen.
2. The client sends an HTTP request to the Express API.
3. The API validates the request and checks authentication where required.
4. Service logic reads or updates PostgreSQL.
5. The API returns a JSON response.
6. React updates the displayed state.

## 4. Data Models

The database uses UUID primary keys and timestamps. The following is a proposed MVP schema.

### User
| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| name | VARCHAR(100) | Required |
| email | VARCHAR(255) | Required, unique |
| password_hash | TEXT | Required; never store plaintext |
| created_at | TIMESTAMPTZ | Default current timestamp |
| updated_at | TIMESTAMPTZ | Updated on modification |

### Task
| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| user_id | UUID | Foreign key → User.id, required |
| title | VARCHAR(160) | Required |
| description | TEXT | Optional |
| due_at | TIMESTAMPTZ | Optional |
| priority | VARCHAR(10) | low, medium, high; default medium |
| status | VARCHAR(20) | todo, in_progress, completed; default todo |
| created_at | TIMESTAMPTZ | Default current timestamp |
| updated_at | TIMESTAMPTZ | Updated on modification |

### FocusSession
| Field | Type | Rules |
|---|---|---|
| id | UUID | Primary key |
| user_id | UUID | Foreign key → User.id, required |
| task_id | UUID | Foreign key → Task.id, nullable |
| started_at | TIMESTAMPTZ | Required |
| ended_at | TIMESTAMPTZ | Nullable until session ends |
| duration_minutes | INTEGER | Non-negative; actual completed duration |
| session_type | VARCHAR(15) | focus or break |
| created_at | TIMESTAMPTZ | Default current timestamp |

### Relationships
- One User has many Tasks.
- One User has many FocusSessions.
- One Task can be linked to many FocusSessions.
- Deleting a user should delete their owned tasks and sessions (cascade).
- A task must belong to the same user as its linked focus session.

## 5. REST API Routes

Base URL: `/api/v1`  
Requests and responses use JSON. Protected routes require a valid bearer token.

| Method | Route | Purpose | Auth |
|---|---|---|---|
| POST | /auth/register | Create an account | No |
| POST | /auth/login | Authenticate and return token | No |
| GET | /auth/me | Get current user profile | Yes |
| GET | /tasks | List current user's tasks; optional status filter | Yes |
| POST | /tasks | Create a task | Yes |
| GET | /tasks/:id | Get one owned task | Yes |
| PATCH | /tasks/:id | Update task fields/status | Yes |
| DELETE | /tasks/:id | Delete an owned task | Yes |
| POST | /focus-sessions | Start/create a focus session | Yes |
| PATCH | /focus-sessions/:id/finish | Finish session and record duration | Yes |
| GET | /focus-sessions | List sessions; optional date range | Yes |
| GET | /progress/summary | Get completed task and focus-minute totals | Yes |

### Example: Create task

**Request** `POST /api/v1/tasks`
```json
{
  "title": "Practice Java collections",
  "description": "Review List, Set, and Map",
  "due_at": "2026-11-03T18:00:00+05:30",
  "priority": "high"
}
```

**Response** `201 Created`
```json
{
  "data": {
    "id": "generated-uuid",
    "title": "Practice Java collections",
    "description": "Review List, Set, and Map",
    "due_at": "2026-11-03T12:30:00Z",
    "priority": "high",
    "status": "todo"
  }
}
```

The UUID and response values above are illustrative.

### API conventions
- Return `200` for successful reads/updates, `201` for creation, and `204` for successful deletion.
- Return `400` for invalid input, `401` for unauthenticated requests, `404` for missing/non-owned resources, and `500` for unexpected server errors.
- Never reveal whether another user's resource exists.
- Validate and normalize all user input.
- Use pagination for task/session lists as the application grows.

## 6. Frontend Screens

### 1. Welcome / Sign in
- Brief product introduction
- Register and sign-in forms
- Navigation to Dashboard after successful authentication

### 2. Dashboard
- Today's task list and status summary
- Task filters
- Quick actions: Add Task, Start Focus, View Progress
- Selecting a task opens Task Details

### 3. Add / Edit Task
- Title and description fields
- Due date selector
- Priority and status controls
- Save and Cancel actions

### 4. Task Details
- Task description, deadline, priority, and status
- Edit / Delete actions
- Start Focus action for the selected task

### 5. Focus Timer
- Selected task and countdown
- Start, Pause, Resume, and Finish controls
- On finish, persist the session and show a completion state

### 6. Progress
- Tasks completed today
- Total focus minutes
- Weekly summary visualization
- Link back to Dashboard

## 7. Navigation Flow

```text
Welcome / Sign in
       |
       v
   Dashboard <----------------------> Progress
       |
       +---- Add / Edit Task -----> Dashboard
       |
       +---- Task Details
                 |
                 +---- Edit Task
                 |
                 +---- Start Focus ----> Focus Timer
                                           |
                                           +---- Finish ----> Progress
```

## 8. Suggested Repository Structure

```text
focusflow/
├── client/
│   └── src/
│       ├── components/
│       ├── pages/
│       ├── services/       # API client
│       └── App.jsx
├── server/
│   └── src/
│       ├── routes/
│       ├── controllers/
│       ├── services/
│       ├── middleware/
│       └── app.js
├── database/
│   └── migrations/
├── docs/
│   └── task-1-project-architecture.md
├── .env.example
└── README.md
```

## 9. Security, Reliability, and Future Improvements

- Hash passwords using a modern password-hashing algorithm (e.g. bcrypt or Argon2).
- Keep secrets in environment variables; do not commit real credentials.
- Use HTTPS in production and rate-limit authentication endpoints.
- Enforce ownership checks for every task and focus-session operation.
- Add database indexes on `user_id`, `status`, and `due_at` where query patterns justify them.
- Add automated tests for validation, authentication, and ownership.
- Future enhancements: reminders, categories, recurring tasks, calendar integration, and cloud sync.

## 10. Deliverables

- Architecture document: this file
- Starter repository: https://github.com/sandeep6386510-ship-it/Full-Stack-Developer-Internship
- LinkedIn demo/explanation: to be recorded and posted by the project owner; add the post URL to the task submission form after publishing.
