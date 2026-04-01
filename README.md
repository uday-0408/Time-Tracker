# Flow State (EOD Maker)

Flow State is a full-stack productivity tracker for daily work logging and reporting. It combines real-time task tracking, Pomodoro flows, session notes, analytics, and end-of-day reporting in one app.

## What this project does

- Tracks one active work session at a time
- Supports category-based time logging with descriptions
- Provides Pomodoro cycles (25/5, 50/10, custom)
- Captures session-linked notes
- Builds date-based EOD summaries and template reports
- Shows analytics (productivity, trend, weekly categories, heatmap, insights)
- Ships as a Next.js frontend + Express/MongoDB backend

## Current feature set (up to date)

### Time tracking

- Start and stop sessions via API and dashboard UI
- Single-running-entry enforcement on backend
- Daily totals and historical grouped views
- Entry editing support with audit history for key field changes

Important behavior:
- Break is not a normal trackable category anymore.
- Break time is managed only by the Pomodoro system.
- Manual tracking allows these categories: Python, SQL, Midas, Datasetu, TT.

### Pomodoro

- Start work phase for a selected category
- Complete work, complete break, pause, resume, cancel
- Mode support: 25/5, 50/10, and custom durations
- Persisted phase state in MongoDB so timer survives refresh/reopen
- Auto-expire stale phases when fetching today session

### Notes and EOD

- CRUD work notes per user/session context
- EOD summary by date (read + update)
- Report templates support (create/update/delete + selection)

### Analytics

- Productivity summary endpoint
- Daily trend endpoint
- Weekly category breakdown endpoint
- Activity heatmap endpoint
- Insights endpoint

### Frontend UX

- Dashboard with timer, category grid, history, and pie distribution
- Dedicated analytics page with multiple charts
- Dedicated EOD page with date navigation and template panel
- Login persisted in localStorage
- PWA assets included (manifest + service worker)

Important behavior:
- Idle detection is currently used for visual state only, not auto-stopping sessions.

## Tech stack

### Backend

- Node.js + Express 4
- TypeScript (ESM)
- MongoDB + Mongoose
- Zod validation
- bcryptjs auth password hashing
- cors, compression, dotenv

### Frontend

- Next.js 14 (App Router)
- React 18 + TypeScript
- TanStack React Query
- Recharts
- Tailwind CSS
- Lucide React

## Repository layout

```text
EOD maker/
├── backend/
│   ├── src/
│   │   ├── server.ts
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   └── utils/
│   ├── package.json
│   └── tsconfig.json
├── frontend/
│   ├── app/
│   ├── components/
│   ├── hooks/
│   ├── lib/
│   ├── public/
│   └── package.json
├── Details.md
└── README.md
```

## API overview

Base URL (local):
- http://localhost:5000/api

### Auth

- POST /auth/login

### Tracking

- POST /track/start
- POST /track/stop
- POST /track/manual
- GET /track/today?userId=...
- GET /track/history?userId=...&days=...
- PUT /track/update
- PUT /track/regularize/:entryId
- PATCH /track/regularize/:entryId/review

### Time Regularization API (Quick Test)

All examples assume base URL: http://localhost:5000/api

#### 1) Create Manual Entry

Endpoint:
- POST /track/manual

Request:

```json
{
	"userId": "660000000000000000000001",
	"category": "Python",
	"description": "Backfilled coding session",
	"startTime": "2026-04-01T09:00:00.000Z",
	"endTime": "2026-04-01T10:30:00.000Z"
}
```

Success Response (201):

```json
{
	"success": true,
	"entry": {
		"_id": "660000000000000000000101",
		"userId": "660000000000000000000001",
		"category": "Python",
		"description": "Backfilled coding session",
		"startTime": "2026-04-01T09:00:00.000Z",
		"endTime": "2026-04-01T10:30:00.000Z",
		"durationSeconds": 5400,
		"status": "completed",
		"source": "manual",
		"isRegularized": false,
		"regularizationStatus": "approved"
	}
}
```

Common Error (409 overlap):

```json
{
	"success": false,
	"message": "Time entry overlaps with an existing session.",
	"errorType": "request_error"
}
```

#### 2) Regularize Existing Entry

Endpoint:
- PUT /track/regularize/:entryId

Request:

```json
{
	"userId": "660000000000000000000001",
	"startTime": "2026-04-01T09:15:00.000Z",
	"endTime": "2026-04-01T10:45:00.000Z",
	"category": "SQL",
	"description": "Adjusted after review",
	"reason": "Missed 15 minutes before standup"
}
```

Success Response (200):

```json
{
	"success": true,
	"entry": {
		"_id": "660000000000000000000101",
		"category": "SQL",
		"startTime": "2026-04-01T09:15:00.000Z",
		"endTime": "2026-04-01T10:45:00.000Z",
		"durationSeconds": 5400,
		"isRegularized": true,
		"regularizationReason": "Missed 15 minutes before standup",
		"regularizationStatus": "pending",
		"auditHistory": [
			{
				"oldStartTime": "2026-04-01T09:00:00.000Z",
				"oldEndTime": "2026-04-01T10:30:00.000Z",
				"oldCategory": "Python",
				"oldDescription": "Backfilled coding session",
				"changedAt": "2026-04-01T11:00:00.000Z",
				"reason": "Missed 15 minutes before standup"
			}
		]
	}
}
```

Common Error (running entry cannot be edited):

```json
{
	"success": false,
	"message": "Cannot regularize a running entry.",
	"errorType": "request_error"
}
```

#### 3) Approve/Reject Regularization (Optional HR Flow)

Endpoint:
- PATCH /track/regularize/:entryId/review

Request (approve):

```json
{
	"status": "approved"
}
```

Request (reject):

```json
{
	"status": "rejected"
}
```

Success Response (200):

```json
{
	"success": true,
	"entry": {
		"_id": "660000000000000000000101",
		"regularizationStatus": "approved"
	}
}
```

### Analytics

- GET /analytics/productivity?userId=...&range=day|week|month
- GET /analytics/trend?userId=...&days=...
- GET /analytics/weekly-categories?userId=...&weeks=...
- GET /analytics/heatmap?userId=...&days=...
- GET /analytics/insights?userId=...

### Pomodoro

- GET /pomodoro/today?userId=...
- POST /pomodoro/start-work
- POST /pomodoro/complete-work
- POST /pomodoro/complete-break
- POST /pomodoro/pause
- POST /pomodoro/resume
- POST /pomodoro/cancel
- PUT /pomodoro/mode

### Notes

- POST /notes
- PUT /notes/:id
- GET /notes
- DELETE /notes/:id

### EOD

- GET /eod/:date
- PUT /eod/:date

### Categories

- GET /categories
- POST /categories
- DELETE /categories/:id

### Templates

- GET /templates
- GET /templates/:id
- POST /templates
- PUT /templates/:id
- DELETE /templates/:id

## Local setup

### 1) Prerequisites

- Node.js 18+
- npm
- MongoDB (local instance or Atlas)

### 2) Backend setup

```bash
cd backend
npm install
```

Create backend/.env from backend/.env.example:

```env
MONGODB_URI=mongodb://localhost:27017/timetracker
PORT=5000
FRONTEND_URL=http://localhost:3000
```

Run backend:

```bash
npm run dev
```

### 3) Frontend setup

```bash
cd frontend
npm install
```

Create frontend/.env.local from frontend/.env.example:

```env
NEXT_PUBLIC_API_URL=http://localhost:5000
```

Run frontend:

```bash
npm run dev
```

Open:
- http://localhost:3000

## Build and production commands

Backend:

```bash
cd backend
npm run build
npm start
```

Frontend:

```bash
cd frontend
npm run build
npm start
```

## Operational notes

- Backend enables compression and CORS with support for wildcard origins in FRONTEND_URL.
- Caching headers are applied for selected read endpoints (for example, today/history/analytics routes) to reduce repeated load.
- Password handling supports migration from legacy plaintext records to bcrypt hashes on login.

## Additional docs

- See Details.md for deeper architectural and implementation notes.
- See backend/README.md and frontend/README.md for module-specific details.
