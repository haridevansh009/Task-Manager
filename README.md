# Task Management App — Frontend

A modern, responsive task management application built to help users organize, manage, and track their tasks efficiently. The frontend provides multiple task visualization modes, allowing users to manage their workflow according to their preferences.

## 🚀 Features

- **List View:** View and manage tasks in a structured list.
- **Grid View:** Display tasks using a clean card-based layout.
- **Kanban Board:** Organize tasks into workflow columns and track their progress.
- **Section View:** Group tasks by status, such as To Do, In Progress, and Done.
- **Task Management:** Create, view, edit, and delete tasks.
- **Responsive UI:** Designed to work across desktop, tablet, and mobile devices.
- **Status Tracking:** Monitor task progress through different workflow stages.
- **Reusable Components:** Maintainable and modular frontend architecture.

## 🛠️ Tech Stack

- React.js
- JavaScript
- HTML5
- CSS3
- Vite
- REST API integration

## 📁 Project Structure

```text
frontend/
├── public/
├── src/
│   ├── components/
│   ├── pages/
│   ├── assets/
│   ├── services/
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── .env.example
├── package.json
├── vite.config.js
└── README.md
```

*Note: Adjust the folder structure to match your actual project files.*

## ⚙️ Getting Started

### 1. Clone the Repository

```bash
git clone YOUR_REPOSITORY_URL
cd frontend
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment Variables

Create a `.env` file in the frontend root directory if your application requires environment variables.

```env
VITE_API_URL=http://localhost:5000/api
```

Replace the URL with your actual backend API address.

### 4. Start the Development Server

```bash
npm run dev
```

Open the local URL displayed in your terminal, usually `http://localhost:5173`.

## 🔗 Backend Integration

The frontend can communicate with the backend through REST APIs to perform task-related operations. Ensure the backend server is running and the API URL is configured correctly.

## 🔐 Security Notes

- Never store private API keys or backend secrets in frontend environment variables.
- Validate user input before submitting data.
- Use HTTPS in production.
- Keep authentication tokens and sensitive information secure.

## 📌 Future Improvements

- Drag-and-drop task management.
- Search, filtering, and sorting.
- Task deadlines and priority labels.
- User authentication and personalized dashboards.
- Real-time task updates.

## 👩‍💻 Development

This project is designed with a component-based architecture to make the interface easier to maintain, extend, and improve.

---

**Built with React and Vite.**
# Taskflow Backend

Node.js + Express + MongoDB (Mongoose), JWT auth, server-side SLA tracking.

## Setup
    cp .env.example .env     # set MONGODB_URI and JWT_SECRET
    npm install
    npm start                # http://localhost:5000
    npm test                 # SLA unit tests

Without SMTP_* settings, the password-reset link is printed in the server console (dev only).

## API
Auth (rate limited): `POST /api/auth/signup | login | logout | forgot-password | reset-password`
Tasks (Bearer token required, every query is scoped to the logged-in user):

| Method | Path | Purpose |
|---|---|---|
| GET | /api/tasks | list. Query: q, status, priority, assignee, dateField (dueDate\|createdAt), from, to, page, limit (max 100). All filters combine with AND. Returns `{ tasks, total, page, pages }` |
| GET | /api/tasks/stats | same filters. Returns `{ total, byStatus, avgSlaMs }` |
| POST | /api/tasks | create (title required) |
| GET | /api/tasks/:id | details incl. `stageHistory` |
| PATCH | /api/tasks/:id | edit title, description, priority, assignee, dueDate (status is rejected here) |
| PATCH | /api/tasks/:id/status | stage transition `{ status }` |
| DELETE | /api/tasks/:id | delete |

## SLA rules
- Stored per task: `stageTimeSpentMs {TODO, IN_PROGRESS, DONE}`, `currentStageEnteredAt`, `stageHistory[]`. All times use the server clock, in milliseconds.
- Leaving a stage adds `now - currentStageEnteredAt` to that stage and pushes a history record. Returning to a stage continues its old total.
- DONE is final: its duration is always 0 and the clock stops. Reopening a DONE task starts the clock for the new stage; old totals are kept.
- Live SLA = stored durations + time since entering the active stage (0 extra when DONE). Total = TODO + IN_PROGRESS + DONE.
- Transitions are one compare-and-set `findOneAndUpdate` on (id, user, current status, entry time), so two racing requests cannot add the same elapsed time twice. A lost race re-reads and retries (max 3), then returns 409.
- Average SLA = sum(total SLA) / N over the filtered set. Active tasks count their live time, DONE tasks are frozen.

## Notes
- Password reset: 32-byte random token, only its SHA-256 hash is stored, expires in 1 hour, cleared atomically on use (single use).
- JWT is stateless: logout only drops the token on the client, and old tokens stay valid until they expire (7 days), even after a password reset.
