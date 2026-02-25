# Flow State — Complete Technical Documentation

> **Flow State** is a full-stack, real-time productivity tracking application built for developers and teams who want granular visibility into how they spend their work hours. It combines session-based time tracking, Pomodoro timers, contextual work notes, end-of-day summaries, rich analytics dashboards, and PWA support into a single cohesive product.

---

## Table of Contents

- [Flow State — Complete Technical Documentation](#flow-state--complete-technical-documentation)
  - [Table of Contents](#table-of-contents)
  - [1. Architecture Overview](#1-architecture-overview)
  - [2. Tech Stack](#2-tech-stack)
    - [Backend](#backend)
    - [Frontend](#frontend)
  - [3. Project Structure](#3-project-structure)
  - [4. Backend Deep Dive](#4-backend-deep-dive)
    - [4.1 Server \& Middleware](#41-server--middleware)
    - [4.2 Database Connection](#42-database-connection)
    - [4.3 Data Models](#43-data-models)
      - [4.3.1 User](#431-user)
      - [4.3.2 TimeEntry (Core Tracking)](#432-timeentry-core-tracking)
      - [4.3.3 PomodoroSession](#433-pomodorosession)
      - [4.3.4 WorkNote](#434-worknote)
      - [4.3.5 EODSummary](#435-eodsummary)
      - [4.3.6 Category](#436-category)
    - [4.4 Services (Business Logic)](#44-services-business-logic)
      - [4.4.1 TimeService](#441-timeservice)
      - [4.4.2 AnalyticsService](#442-analyticsservice)
      - [4.4.3 PomodoroService](#443-pomodoroservice)
      - [4.4.4 NotesService](#444-notesservice)
      - [4.4.5 EODService](#445-eodservice)
      - [4.4.6 CategoryService](#446-categoryservice)
    - [4.5 Controllers (HTTP Layer)](#45-controllers-http-layer)
    - [4.6 Routes](#46-routes)
    - [4.7 Complete API Reference](#47-complete-api-reference)
      - [Authentication](#authentication)
      - [Time Tracking](#time-tracking)
      - [Analytics](#analytics)
      - [Pomodoro](#pomodoro)
      - [Notes](#notes)
      - [EOD Summaries](#eod-summaries)
      - [Categories](#categories)
  - [5. Frontend Deep Dive](#5-frontend-deep-dive)
    - [5.1 App Configuration](#51-app-configuration)
    - [5.2 Theming \& Design System](#52-theming--design-system)
    - [5.3 State Management](#53-state-management)
    - [5.4 Hooks Layer](#54-hooks-layer)
      - [useTime.ts](#usetimets)
      - [useHistory.ts](#usehistoryts)
      - [useAnalytics.ts](#useanalyticsts)
      - [usePomodoro.ts](#usepomodorots)
      - [useNotes.ts](#usenotests)
      - [useEOD.ts](#useeodts)
      - [useCategories.ts](#usecategoriests)
      - [useIdle.ts](#useidlets)
      - [usePWA.ts](#usepwats)
    - [5.5 Components](#55-components)
      - [Timer.tsx](#timertsx)
      - [CategoryGrid.tsx](#categorygridtsx)
      - [PomodoroTimer.tsx (517 lines)](#pomodorotimertsx-517-lines)
      - [SessionNotes.tsx](#sessionnotestsx)
      - [EODSummaryPanel.tsx](#eodsummarypaneltsx)
      - [HistoryList.tsx](#historylisttsx)
      - [PWAStatus.tsx](#pwastatustsx)
      - [Analytics Components](#analytics-components)
    - [5.6 Pages \& Routing](#56-pages--routing)
  - [6. Feature Breakdown](#6-feature-breakdown)
    - [6.1 Time Tracking (Core)](#61-time-tracking-core)
    - [6.2 Dynamic Categories](#62-dynamic-categories)
    - [6.3 Pomodoro Timer](#63-pomodoro-timer)
    - [6.4 Contextual Work Notes](#64-contextual-work-notes)
    - [6.5 EOD Summaries](#65-eod-summaries)
    - [6.6 Analytics Dashboard](#66-analytics-dashboard)
    - [6.7 PWA Support](#67-pwa-support)
      - [Manifest (`manifest.json`)](#manifest-manifestjson)
      - [Service Worker (`sw.js`)](#service-worker-swjs)
  - [7. Data Flow \& Architecture Patterns](#7-data-flow--architecture-patterns)
    - [Request Lifecycle](#request-lifecycle)
    - [Polling Architecture](#polling-architecture)
    - [Cache Invalidation Map](#cache-invalidation-map)
  - [8. Database Schema Relationships](#8-database-schema-relationships)
  - [9. Development Setup](#9-development-setup)
    - [Prerequisites](#prerequisites)
    - [Backend](#backend-1)
    - [Frontend](#frontend-1)
    - [Build for Production](#build-for-production)
  - [10. Environment Variables](#10-environment-variables)
    - [Backend (`backend/.env`)](#backend-backendenv)
    - [Frontend](#frontend-2)

---

## 1. Architecture Overview

```
┌───────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                       │
│  Next.js 14 (App Router) + React 18 + TanStack React Query   │
│  Tailwind CSS dark theme · Recharts · Lucide icons            │
│  Service Worker (offline + background sync)                   │
└──────────────────────────┬────────────────────────────────────┘
                           │  HTTP (REST JSON)
                           │  Polling every 5 seconds for live data
                           ▼
┌───────────────────────────────────────────────────────────────┐
│                     BACKEND (API Server)                      │
│  Express 4 + TypeScript (ESM) + Zod validation                │
│  Controller → Service → Model (3-layer architecture)          │
│  Port 5000 (configurable)                                     │
└──────────────────────────┬────────────────────────────────────┘
                           │  Mongoose ODM
                           ▼
┌───────────────────────────────────────────────────────────────┐
│                      MongoDB (Database)                       │
│  Collections: users, timeentries, pomodorosessions,           │
│  worknotes, eodsummaries, categories                          │
└───────────────────────────────────────────────────────────────┘
```

**Communication pattern:** The frontend polls `GET /api/track/today` every 5 seconds (React Query `refetchInterval`) to keep the running timer and daily totals in sync. All mutations (start/stop tracking, save notes, etc.) go through React Query mutations which invalidate relevant query caches on success, triggering immediate refetches.

---

## 2. Tech Stack

### Backend

| Technology | Version | Purpose |
|---|---|---|
| **Node.js** | ≥18 | Runtime |
| **Express** | 4.18 | HTTP framework |
| **TypeScript** | 5.3+ | Type safety, compiled to ES2020 |
| **Mongoose** | 8.0.3 | MongoDB ODM |
| **Zod** | 4.3.6 | Request body validation |
| **dotenv** | 16.3 | Environment variable loading |
| **cors** | 2.8.5 | Cross-origin resource sharing |
| **tsx** | 4.7.0 | Dev server with watch mode (replaces `ts-node`) |

**Module system:** ESM (`"type": "module"` in `package.json`). All internal imports use `.js` extensions for Node ESM compatibility.

### Frontend

| Technology | Version | Purpose |
|---|---|---|
| **Next.js** | 14.0.4 | React framework (App Router) |
| **React** | 18.2 | UI library |
| **TypeScript** | 5+ | Type safety |
| **TanStack React Query** | 5.90 | Server state management, caching, polling |
| **Recharts** | 3.7 | Data visualization (charts) |
| **Tailwind CSS** | 3.3 | Utility-first CSS |
| **Lucide React** | 0.563 | Icon library |
| **date-fns** | 4.1 | Date manipulation |
| **clsx + tailwind-merge** | Latest | Conditional class name merging |

---

## 3. Project Structure

```
EOD maker/
├── Details.md                          ← This file
├── README.md
│
├── backend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── .env                            ← MONGODB_URI, PORT
│   └── src/
│       ├── server.ts                   ← Express entry point
│       ├── config/
│       │   └── mongodb.ts              ← Mongoose connection
│       ├── models/
│       │   ├── User.ts                 ← User schema
│       │   ├── TimeEntry.ts            ← Core tracking schema
│       │   ├── PomodoroSession.ts      ← Daily pomodoro counter
│       │   ├── WorkNote.ts             ← Session-linked notes
│       │   ├── EODSummary.ts           ← End-of-day summaries
│       │   └── Category.ts             ← Dynamic work categories
│       ├── services/
│       │   ├── time.service.ts         ← Start/stop/query time entries
│       │   ├── analytics.service.ts    ← Trends, heatmaps, insights
│       │   ├── pomodoro.service.ts     ← Pomodoro cycle management
│       │   ├── notes.service.ts        ← CRUD for work notes
│       │   ├── eod.service.ts          ← EOD summary generation
│       │   └── category.service.ts     ← Category CRUD + seeding
│       ├── controllers/
│       │   ├── time.controller.ts
│       │   ├── analytics.controller.ts
│       │   ├── pomodoro.controller.ts
│       │   ├── notes.controller.ts
│       │   ├── eod.controller.ts
│       │   └── category.controller.ts
│       └── routes/
│           ├── auth.ts                 ← POST /api/auth/login
│           ├── track.ts               ← /api/track/*
│           ├── analytics.ts           ← /api/analytics/*
│           ├── pomodoro.ts            ← /api/pomodoro/*
│           ├── notes.ts               ← /api/notes/*
│           ├── eod.ts                 ← /api/eod/*
│           └── categories.ts          ← /api/categories/*
│
├── frontend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.js
│   ├── tailwind.config.ts
│   ├── postcss.config.js
│   ├── next-env.d.ts
│   ├── public/
│   │   ├── manifest.json              ← PWA manifest
│   │   ├── sw.js                      ← Service worker
│   │   └── icons/
│   │       ├── icon-192.svg
│   │       └── icon-512.svg
│   ├── app/
│   │   ├── globals.css                ← CSS variables, dark theme
│   │   ├── layout.tsx                 ← Root layout, PWA meta, providers
│   │   ├── page.tsx                   ← Home dashboard
│   │   ├── analytics/
│   │   │   └── page.tsx               ← Analytics dashboard
│   │   └── eod/
│   │       └── page.tsx               ← EOD summary page
│   ├── components/
│   │   ├── Login.tsx                  ← Auth form
│   │   ├── Timer.tsx                  ← Running timer display
│   │   ├── CategoryGrid.tsx           ← Category selector + CRUD
│   │   ├── PomodoroTimer.tsx          ← Full pomodoro widget
│   │   ├── SessionNotes.tsx           ← Context-linked notes
│   │   ├── EODSummaryPanel.tsx        ← EOD editor panel
│   │   ├── HistoryList.tsx            ← 7-day history with note badges
│   │   ├── PWAStatus.tsx              ← Offline banner + install button
│   │   ├── providers.tsx              ← React Query provider
│   │   ├── ui/
│   │   │   ├── button.tsx             ← Variant-based button
│   │   │   └── card.tsx               ← Card primitives
│   │   └── analytics/
│   │       ├── TrendChart.tsx         ← Line chart (daily trend)
│   │       ├── CategoryComparisonChart.tsx  ← Grouped bar chart
│   │       ├── ActivityHeatmap.tsx     ← GitHub-style heatmap
│   │       ├── InsightCards.tsx        ← 5-card insight row
│   │       └── SessionLengthChart.tsx  ← Area chart (avg session)
│   ├── hooks/
│   │   ├── useTime.ts                 ← Today data, start/stop mutations
│   │   ├── useHistory.ts              ← Historical entries
│   │   ├── useAnalytics.ts            ← All analytics queries
│   │   ├── usePomodoro.ts             ← Pomodoro queries/mutations
│   │   ├── useNotes.ts                ← Notes CRUD
│   │   ├── useEOD.ts                  ← EOD queries/mutations
│   │   ├── useCategories.ts           ← Category queries/mutations
│   │   ├── useIdle.ts                 ← Idle detection
│   │   └── usePWA.ts                  ← Service worker registration
│   └── lib/
│       └── utils.ts                   ← cn() class merger
```

---

## 4. Backend Deep Dive

### 4.1 Server & Middleware

**Entry point:** `src/server.ts`

```
dotenv.config()  →  express()  →  cors({ origin: '*' })  →  express.json()  →  dbConnect()  →  mount routes  →  listen(5000)
```

CORS is currently open (`origin: '*'`, `credentials: false`) for development. The server loads `.env` from `backend/.env` (resolved relative to `__dirname` using `fileURLToPath` for ESM compatibility).

### 4.2 Database Connection

`src/config/mongodb.ts` — Reads `MONGODB_URI` from environment. Uses Mongoose's `readyState` check to avoid duplicate connections. Calls `process.exit(1)` on connection failure.

### 4.3 Data Models

All models use Mongoose with TypeScript interfaces extending `Document`. Every model has `timestamps: true` (auto `createdAt`/`updatedAt`).

#### 4.3.1 User

| Field | Type | Constraints |
|---|---|---|
| `name` | `String` | required, unique, trimmed |
| `password` | `String` | required (stored as plain text) |

**Note:** Authentication is intentionally simplified — no bcrypt hashing, no JWT tokens. The login endpoint creates a new user if the name doesn't exist, or checks the password if it does.

#### 4.3.2 TimeEntry (Core Tracking)

This is the central model. Every tracked session — whether manual, Pomodoro work, or Pomodoro break — creates a TimeEntry.

| Field | Type | Default | Notes |
|---|---|---|---|
| `category` | `String` | — | required, free-form (matches user's categories) |
| `startTime` | `Date` | — | required, indexed |
| `endTime` | `Date` | `null` | Set when stopped |
| `date` | `String` | — | `YYYY-MM-DD` format, indexed |
| `durationSeconds` | `Number` | `0` | Calculated on stop: `(endTime - startTime) / 1000` |
| `userId` | `ObjectId` | — | ref: `User`, indexed |
| `description` | `String` | `''` | User-provided context |
| `isIdle` | `Boolean` | `false` | Reserved for idle-detection flagging |
| `history` | `[HistorySubdoc]` | `[]` | Audit trail for manual edits |
| `status` | `Enum` | `'running'` | `'running' | 'completed' | 'paused'` |

**History subdocument** (stored when an entry is manually edited):
| Field | Type |
|---|---|
| `updatedAt` | `Date` |
| `reason` | `String` |
| `previousDuration` | `Number` |
| `previousStartTime` | `Date` |
| `previousEndTime` | `Date` |

**Indexes:**
- `{ userId: 1, date: 1 }` — efficient daily queries
- `{ userId: 1, status: 1 }` — fast running-entry lookups

**Key invariant:** Only one entry can have `status: 'running'` per user at a time. The service layer enforces this by calling `stopEntry()` before `startEntry()`.

#### 4.3.3 PomodoroSession

Tracks daily Pomodoro statistics per user. Actual work/break time is recorded as normal TimeEntries — this model only stores counters and mode preference.

| Field | Type | Default | Constraints |
|---|---|---|---|
| `userId` | `ObjectId` | — | ref: `User`, required |
| `date` | `String` | — | `YYYY-MM-DD`, required |
| `completedPomodoros` | `Number` | `0` | Incremented when work phase ends |
| `completedBreaks` | `Number` | `0` | Incremented when break phase ends |
| `mode` | `Enum` | `'25/5'` | `'25/5' | '50/10' | 'custom'` |
| `customWorkMinutes` | `Number` | `25` | min: 1, max: 180 |
| `customBreakMinutes` | `Number` | `5` | min: 1, max: 60 |

**Index:** `{ userId: 1, date: 1 }` unique — one record per user per day.

#### 4.3.4 WorkNote

Contextual notes linked to either a specific time-tracking session or a calendar day.

| Field | Type | Default | Notes |
|---|---|---|---|
| `userId` | `ObjectId` | — | ref: `User`, indexed |
| `linkedType` | `Enum` | — | `'SESSION'` or `'DAY'` |
| `referenceId` | `String` | — | TimeEntry `_id` (for SESSION) or `YYYY-MM-DD` (for DAY) |
| `content` | `String` | `''` | Current note content |
| `versions` | `[{ content, editedAt }]` | `[]` | Full edit history (previous versions pushed on update) |
| `isDeleted` | `Boolean` | `false` | Soft-delete flag |

**Index:** `{ userId: 1, linkedType: 1, referenceId: 1 }`

**Design decision:** Notes use soft-delete (`isDeleted: true`) so version history is never lost. The `versions` array preserves every previous edit.

#### 4.3.5 EODSummary

End-of-day summaries combining auto-computed metrics with user-editable narrative fields.

| Field | Type | Default | Notes |
|---|---|---|---|
| `userId` | `ObjectId` | — | ref: `User` |
| `date` | `String` | — | `YYYY-MM-DD` |
| `totalHours` | `Number` | `0` | Auto-computed from TimeEntries |
| `productiveHours` | `Number` | `0` | Auto-computed using productive categories |
| `categoryBreakdown` | `Mixed` | `{}` | `Record<string, number>` — category → seconds |
| `summary` | `String` | `''` | User-written daily summary |
| `highlights` | `[String]` | `[]` | Key accomplishments |
| `blockers` | `[String]` | `[]` | Issues/blockers encountered |
| `versions` | `[{ summary, highlights, blockers, editedAt }]` | `[]` | Edit history for user fields |

**Index:** `{ userId: 1, date: 1 }` unique.

**Dual nature:** The metrics fields (`totalHours`, `productiveHours`, `categoryBreakdown`) are recomputed every time the EOD is accessed, ensuring they're always up-to-date. The narrative fields (`summary`, `highlights`, `blockers`) are user-controlled and versioned.

#### 4.3.6 Category

User-specific dynamic work categories with color coding and productivity classification.

| Field | Type | Default | Notes |
|---|---|---|---|
| `userId` | `ObjectId` | — | ref: `User`, indexed |
| `name` | `String` | — | required, trimmed |
| `color` | `String` | `'blue'` | Tailwind color key |
| `isProductive` | `Boolean` | `true` | Used by analytics to calculate productivity scores |
| `order` | `Number` | `0` | Display order in the grid |

**Index:** `{ userId: 1, name: 1 }` unique — no duplicate categories per user.

**Available colors:** `blue`, `green`, `purple`, `orange`, `red`, `gray`, `pink`, `teal`, `cyan`, `yellow`, `indigo`, `emerald`

**Default categories** (auto-seeded on first access):

| Name | Color | Productive | Order |
|---|---|---|---|
| Python | blue | ✅ | 0 |
| SQL | green | ✅ | 1 |
| Midas | purple | ✅ | 2 |
| Datasetu | orange | ✅ | 3 |
| Break | gray | ❌ | 4 |
| TT | red | ❌ | 5 |

---

### 4.4 Services (Business Logic)

The service layer contains all business logic. Controllers never access models directly — they always delegate to services.

#### 4.4.1 TimeService

The core tracking service. Enforces the **single active session** invariant.

| Method | Signature | Behavior |
|---|---|---|
| `startEntry` | `(userId, category, description?) → TimeEntry` | **Stops any running entry first**, then creates a new one with `status: 'running'` and today's date. |
| `stopEntry` | `(userId) → TimeEntry \| null` | Finds the running entry, calculates duration, sets `status: 'completed'` and `endTime`. Returns `null` if nothing was running. |
| `getTodayData` | `(userId) → { entries, totals, runningEntry }` | Queries today's entries by date string. Builds `totals: Record<category, seconds>` from completed entries. **Fallback:** if no running entry found in today's entries, queries all entries for `status: 'running'` (handles UTC midnight boundary). |
| `getHistory` | `(userId, days=7) → DayGroup[]` | Gets completed entries in the last N days, groups by date with per-category totals. |
| `updateEntry` | `(entryId, userId, updates) → TimeEntry` | Updates an entry with ownership check. Pushes previous values to `history[]` if start/end times change. Recalculates `durationSeconds`. |

**Important edge case handling:** The `getTodayData` fallback query prevents sessions from "disappearing" when a session started before UTC midnight but the user checks after midnight. The previous behavior only queried today's date, making active sessions invisible.

#### 4.4.2 AnalyticsService

All analytics computations run on the raw TimeEntry data.

| Method | Signature | Returns |
|---|---|---|
| `getProductivityStats` | `(userId, range)` | `{ totalHours, productiveHours, productivityScore, maxFocusStreak }` — score is percentage, streak is consecutive productive sessions |
| `getDailyTrend` | `(userId, days=30)` | Array of daily data points with total/productive hours, session count, avg session length. **Zero days are included** (pre-filled date range). |
| `getWeeklyCategoryComparison` | `(userId, weeks=4)` | Flat array of `{ week, category, hours }`. Weeks labeled by Monday date (e.g., "Feb 3"). |
| `getHeatmapData` | `(userId, days=365)` | Array of `{ date, hours }` for every day in range. Used for GitHub-style contribution graph. |
| `getInsights` | `(userId)` | Analyzes last 90 days: most productive day-of-week, avg session length, top category, daily average, overall productivity rate. |

All methods use `CategoryService.getProductiveNames(userId)` to determine which categories count as "productive" — this is fully dynamic per user.

#### 4.4.3 PomodoroService

Orchestrates Pomodoro cycles on top of the existing time tracking system.

| Method | Behavior |
|---|---|
| `getTodaySession(userId)` | Find-or-create today's PomodoroSession (defaults to `25/5` mode). |
| `completePomodoro(userId, category)` | 1. Stops the running TimeEntry. 2. Increments `completedPomodoros` via `$inc`. 3. Auto-starts a new "Break" TimeEntry with description "Pomodoro break after {category}". |
| `completeBreak(userId)` | 1. Stops the running break TimeEntry. 2. Increments `completedBreaks`. |
| `setMode(userId, mode, customWork?, customBreak?)` | Updates mode preference. Only stores `customWorkMinutes`/`customBreakMinutes` when `mode === 'custom'`. |

**Key design:** Pomodoro work and break periods create **real TimeEntries**, meaning they show up in daily totals, history, analytics, and EOD summaries — zero data duplication.

#### 4.4.4 NotesService

CRUD for contextual work notes with full version history.

| Method | Behavior |
|---|---|
| `createNote(userId, linkedType, referenceId, content)` | Creates note with empty `versions[]`. |
| `updateNote(noteId, userId, newContent)` | **Pushes current content into `versions[]` before overwriting** — full audit trail. Ownership + soft-delete check. |
| `getNotes(userId, linkedType, referenceId)` | Returns non-deleted notes for the given context, sorted by `createdAt` desc. |
| `deleteNote(noteId, userId)` | Sets `isDeleted = true` (soft delete). Version history preserved. |

#### 4.4.5 EODService

End-of-day summary management with auto-computed metrics.

| Method | Behavior |
|---|---|
| `getOrCreateEOD(userId, date)` | **Always recomputes** `totalHours`, `productiveHours`, `categoryBreakdown` from TimeEntries. Creates new EOD if missing; otherwise refreshes metrics while preserving user-editable fields. |
| `updateEOD(userId, date, updates)` | Calls `getOrCreateEOD` first (ensures metrics are fresh). Snapshots `summary`, `highlights`, `blockers` into `versions[]`. Applies only provided updates. |
| `computeMetrics(userId, date)` | *(private)* Aggregates completed TimeEntries for the date. Uses `CategoryService.getProductiveNames()` to determine productivity. |

#### 4.4.6 CategoryService

Dynamic category management with auto-seeding.

| Method | Behavior |
|---|---|
| `getCategories(userId)` | Fetches sorted by `order`. **If zero categories exist, seeds with 6 defaults** (Python, SQL, Midas, Datasetu, Break, TT). |
| `addCategory(userId, name, color, isProductive)` | Auto-calculates `order` as `max(existing) + 1`. |
| `deleteCategory(categoryId, userId)` | Deletes by ID + userId ownership check. |
| `getProductiveNames(userId)` | Returns `string[]` of category names where `isProductive: true`. **Used by analytics and EOD services** to dynamically classify productivity. |

---

### 4.5 Controllers (HTTP Layer)

Controllers handle HTTP concerns: parsing request params/body, calling the appropriate service method, and formatting the response. All use try-catch with consistent error responses.

**Validation:** Controllers use **Zod schemas** to validate request bodies before passing to services. Invalid payloads return a 400 with the Zod error message.

| Controller | Zod Schemas |
|---|---|
| `time.controller` | `startSchema { userId, category, description? }`, `stopSchema { userId }` |
| `pomodoro.controller` | `completePomodoroSchema { userId, category }`, `completeBreakSchema { userId }`, `setModeSchema { userId, mode, customWorkMinutes? (1-180), customBreakMinutes? (1-60) }` |
| `notes.controller` | `createNoteSchema { userId, linkedType: SESSION\|DAY, referenceId, content }`, `updateNoteSchema { content, userId }` |
| `eod.controller` | `updateEODSchema { userId, summary?, highlights?: string[], blockers?: string[] }` |
| `category.controller` | `addCategorySchema { userId, name: 1-30 chars, color?, isProductive? }` |

---

### 4.6 Routes

All routes are mounted under `/api` on the Express server.

### 4.7 Complete API Reference

#### Authentication

| Method | Path | Body/Query | Response |
|---|---|---|---|
| `POST` | `/api/auth/login` | `{ name, password }` | `{ success, user: { id, name } }` — creates user if not exists |

#### Time Tracking

| Method | Path | Body/Query | Response |
|---|---|---|---|
| `POST` | `/api/track/start` | `{ userId, category, description? }` | `{ success, entry }` |
| `POST` | `/api/track/stop` | `{ userId }` | `{ success, entry }` |
| `GET` | `/api/track/today` | `?userId=` | `{ success, entries[], totals: {}, runningEntry }` |
| `GET` | `/api/track/history` | `?userId=&days=7` | `{ success, history: [{ date, entries, totals, totalSeconds }] }` |
| `PUT` | `/api/track/update` | `{ entryId, userId, updates: { startTime?, endTime?, description?, category? } }` | `{ success, entry }` |

#### Analytics

| Method | Path | Query | Response |
|---|---|---|---|
| `GET` | `/api/analytics/productivity` | `?userId=&range=day\|week\|month` | `{ success, data: { totalHours, productiveHours, productivityScore, maxFocusStreak } }` |
| `GET` | `/api/analytics/trend` | `?userId=&days=30` | `{ success, data: [{ date, totalHours, productiveHours, sessions, avgSessionMinutes }] }` |
| `GET` | `/api/analytics/weekly-categories` | `?userId=&weeks=4` | `{ success, data: [{ week, category, hours }] }` |
| `GET` | `/api/analytics/heatmap` | `?userId=&days=365` | `{ success, data: [{ date, hours }] }` |
| `GET` | `/api/analytics/insights` | `?userId=` | `{ success, data: { mostProductiveDay, avgSessionMinutes, topCategory, ... } }` |

#### Pomodoro

| Method | Path | Body/Query | Response |
|---|---|---|---|
| `GET` | `/api/pomodoro/today` | `?userId=` | `{ success, session }` |
| `POST` | `/api/pomodoro/complete-work` | `{ userId, category }` | `{ success, session, breakEntry }` |
| `POST` | `/api/pomodoro/complete-break` | `{ userId }` | `{ success, session }` |
| `PUT` | `/api/pomodoro/mode` | `{ userId, mode, customWorkMinutes?, customBreakMinutes? }` | `{ success, session }` |

#### Notes

| Method | Path | Body/Query | Response |
|---|---|---|---|
| `POST` | `/api/notes` | `{ userId, linkedType, referenceId, content }` | `{ success, note }` |
| `GET` | `/api/notes` | `?userId=&linkedType=&referenceId=` | `{ success, notes[] }` |
| `PUT` | `/api/notes/:id` | `{ content, userId }` | `{ success, note }` |
| `DELETE` | `/api/notes/:id` | `?userId=` | `{ success, note }` |

#### EOD Summaries

| Method | Path | Body/Query | Response |
|---|---|---|---|
| `GET` | `/api/eod/:date` | `?userId=` | `{ success, eod }` |
| `PUT` | `/api/eod/:date` | `{ userId, summary?, highlights?, blockers? }` | `{ success, eod }` |

#### Categories

| Method | Path | Body/Query | Response |
|---|---|---|---|
| `GET` | `/api/categories` | `?userId=` | `{ success, categories[] }` |
| `POST` | `/api/categories` | `{ userId, name, color?, isProductive? }` | `{ success, category }` |
| `DELETE` | `/api/categories/:id` | `?userId=` | `{ success, category }` |

---

## 5. Frontend Deep Dive

### 5.1 App Configuration

**Next.js:** Version 14 with the App Router pattern (file-based routing under `app/`). No custom webpack config. Module resolution uses `@/*` path alias mapping to the project root.

**TypeScript:** Target ES2017, JSX preserved for Next.js compilation, bundler module resolution.

### 5.2 Theming & Design System

The app uses a **dark-only theme** via Tailwind CSS with CSS custom properties.

**CSS Variables** (defined in `globals.css`):

```css
:root {
  --background: 222.2 84% 4.9%;     /* Near black */
  --foreground: 210 40% 98%;        /* Near white */
  --primary: 210 40% 98%;           /* White-ish */
  --secondary: 217.2 32.6% 17.5%;   /* Dark blue-gray */
  --destructive: 0 62.8% 30.6%;     /* Dark red */
  --border: 217.2 32.6% 17.5%;
  --ring: 212.7 26.8% 83.9%;
  --radius: 0.5rem;
}
```

The `<html>` element always has `className="dark"`. Tailwind's `darkMode: ["class"]` is configured but the app never switches to light mode.

**UI Primitives:**
- `Button` — variant-based (`default`, `destructive`, `outline`, `secondary`, `ghost`) with sizes (`default`, `sm`, `lg`, `icon`). Uses `forwardRef` pattern.
- `Card` / `CardHeader` / `CardTitle` / `CardContent` — composable card primitives with Tailwind classes.
- Icons from `lucide-react` — used throughout for consistent iconography.

### 5.3 State Management

**Server state:** TanStack React Query v5 manages all API data. The `QueryClient` is instantiated via `useState` in the `providers.tsx` client component to avoid SSR issues.

**Client state:** React `useState` for local UI state (form inputs, modals, toggle states). No global client state store — all shared state comes from the server via React Query caches.

**Polling:** `useToday()` polls every 5 seconds (`refetchInterval: 5000`) to keep the dashboard timer, daily totals, and running entry status live.

**Cache invalidation strategy:** Every mutation hook calls `queryClient.invalidateQueries()` on success with the relevant query keys, triggering immediate refetches. For example, `useStartTracking()` invalidates both `['today', userId]` and `['history', userId]`.

### 5.4 Hooks Layer

All API communication is encapsulated in custom hooks. No component ever calls `fetch` directly.

#### useTime.ts

| Export | Type | API | Query Key | Notes |
|---|---|---|---|---|
| `useToday(userId)` | Query | `GET /api/track/today` | `['today', userId]` | **Polls every 5s** |
| `useStartTracking()` | Mutation | `POST /api/track/start` | — | Invalidates `today` + `history` |
| `useStopTracking()` | Mutation | `POST /api/track/stop` | — | Invalidates `today` + `history` |

**Types:** `TimeEntry { _id, category, startTime, endTime?, description? }`, `TodayData { totals: Record<string, number>, runningEntry: TimeEntry | null, entries: TimeEntry[] }`

#### useHistory.ts

| Export | API | Query Key |
|---|---|---|
| `useHistory(userId, days=7)` | `GET /api/track/history` | `['history', userId, days]` |

#### useAnalytics.ts

| Export | API | Query Key |
|---|---|---|
| `useAnalytics(userId, range)` | `GET /api/analytics/productivity` | `['analytics', userId, range]` |
| `useTrend(userId, days)` | `GET /api/analytics/trend` | `['trend', userId, days]` |
| `useWeeklyCategories(userId, weeks)` | `GET /api/analytics/weekly-categories` | `['weeklyCategories', userId, weeks]` |
| `useHeatmap(userId, days)` | `GET /api/analytics/heatmap` | `['heatmap', userId, days]` |
| `useInsights(userId)` | `GET /api/analytics/insights` | `['insights', userId]` |

**Exported types:** `TrendPoint`, `WeeklyCategoryPoint`, `HeatmapPoint`, `Insights`

#### usePomodoro.ts

| Export | Type | API |
|---|---|---|
| `usePomodoroToday(userId)` | Query | `GET /api/pomodoro/today` |
| `useCompleteWork()` | Mutation | `POST /api/pomodoro/complete-work` |
| `useCompleteBreak()` | Mutation | `POST /api/pomodoro/complete-break` |
| `useSetPomodoroMode()` | Mutation | `PUT /api/pomodoro/mode` |

#### useNotes.ts

| Export | Type | API |
|---|---|---|
| `useNotes(userId, linkedType, referenceId)` | Query | `GET /api/notes` |
| `useCreateNote()` | Mutation | `POST /api/notes` |
| `useUpdateNote()` | Mutation | `PUT /api/notes/:id` |
| `useDeleteNote()` | Mutation | `DELETE /api/notes/:id` |

#### useEOD.ts

| Export | Type | API |
|---|---|---|
| `useEOD(userId, date)` | Query | `GET /api/eod/:date` |
| `useUpdateEOD()` | Mutation | `PUT /api/eod/:date` |

#### useCategories.ts

| Export | Type | API |
|---|---|---|
| `useCategories(userId)` | Query | `GET /api/categories` |
| `useAddCategory()` | Mutation | `POST /api/categories` |
| `useDeleteCategory()` | Mutation | `DELETE /api/categories/:id` |

#### useIdle.ts

Browser idle detection via DOM events (`mousedown`, `mousemove`, `keydown`, `scroll`, `touchstart`). Returns a `boolean`. Used for visual indicators only — does **not** auto-stop sessions (that caused a bug where sessions silently vanished when users worked outside the browser).

#### usePWA.ts

Service worker lifecycle management:
- Registers `/sw.js` on mount
- Tracks `isOnline` via `navigator.onLine` + `online`/`offline` events
- Captures `beforeinstallprompt` event for install prompting
- Sends `'REPLAY_SYNC'` to service worker when coming back online
- Exposes `{ isOnline, isInstalled, canInstall, promptInstall() }`

### 5.5 Components

#### Timer.tsx
The live elapsed-time display for the currently running session.
- Calculates elapsed time with a 1-second `setInterval` from `runningEntry.startTime`
- Formats as `HH:MM:SS`
- Shows category name and a destructive Stop button when running
- Shows idle state with a Play icon when nothing is running

#### CategoryGrid.tsx
Interactive category selector with full CRUD capabilities.
- **Displays** categories as colored, clickable buttons in a responsive grid
- **Active state:** The running category gets a colored ring border
- **Add mode:** Inline form with name input, 12-color visual picker, productive/break toggle
- **Edit mode:** Toggle that shows red ✕ delete buttons on each category with confirmation
- **Color mapping:** Each of 12 color keys maps to specific Tailwind hover, border, and text classes
- Non-productive categories show a "(break)" label

#### PomodoroTimer.tsx (517 lines)
The most complex frontend component. A full Pomodoro timer with SVG progress ring.

**Sub-components:**
- `ProgressRing` — SVG circle with `stroke-dashoffset` animated via CSS transition. Blue for work, green for break. Dims to 50% opacity when paused.
- `PomodoroDots` — Red dots showing completed pomodoros today (max 8 visible, overflow count)
- `CustomModeForm` — Inline form for setting custom work/break minutes (1–180 / 1–60)

**State machine:**
```
IDLE → (Start) → WORK → (timer hits 0) → BREAK → (timer hits 0) → IDLE
                    ↑                        ↓
                    └────── (Start again) ────┘
```

**Features:**
- Mode selector: `25/5`, `50/10`, or Custom (with inline form)
- Pause/Resume (freezes countdown, dims ring, shows pulsing "PAUSED" text)
- Cancel (stops the TimeEntry and resets)
- Skip Break (ends break early)
- Audio notification (inline base64 WAV beep) when a phase ends
- `hasStartedRef` prevents the zero-handler from firing on mount (previous bug)
- `configRef` prevents stale closure issues in the timer effect

#### SessionNotes.tsx
Contextual notes tied to the currently running time-tracking session.
- Textarea with Ctrl+Enter shortcut for quick submission
- Displays existing notes as a list with timestamps and edit counts
- Inline editing: click pencil → edit mode with save/cancel
- Delete with confirmation
- Only visible when a timer is actively running

#### EODSummaryPanel.tsx
End-of-day summary editor combining auto-computed metrics with user narrative.
- **Read-only section:** 3-stat grid (Total Hours, Productive Hours, Productivity Score %), category breakdown table
- **Editable section:** Summary textarea, highlights list (add/remove individual items), blockers list (add/remove)
- Tracks dirty state to enable/disable the save button
- Shows version count ("N versions") for audit trail
- Null-guard on `categoryBreakdown` to prevent crashes

#### HistoryList.tsx
7-day history view with per-entry detail.
- Groups entries by date with day headers showing total hours
- Each entry row: category tag, start→end time, duration in minutes
- `SessionNoteBadge` — inline sub-component that queries notes per entry and shows a count badge (e.g., "2 notes")

#### PWAStatus.tsx
Global UI overlays for PWA status:
- **Offline banner:** Fixed yellow bar at top when `!isOnline` with WifiOff icon
- **Install button:** Fixed blue button at bottom-right when `canInstall` is true

#### Analytics Components

| Component | Chart Type | Data Source | Description |
|---|---|---|---|
| `TrendChart` | Recharts `LineChart` | `TrendPoint[]` | Two lines: Total Hours (blue) + Productive Hours (green). Date labels "Feb 3" style. |
| `CategoryComparisonChart` | Recharts `BarChart` | `WeeklyCategoryPoint[]` | Grouped bars per week with one bar per category. 8-color palette. Uses `useMemo` for data pivoting. |
| `ActivityHeatmap` | Custom SVG grid | `HeatmapPoint[]` | GitHub-style contribution graph with 6 intensity levels (gray → green-400). Day-of-week labels, month labels, legend strip. |
| `InsightCards` | 5-card layout | `Insights` | Most Productive Day, Avg Session Length, Top Category, Daily Average, Productivity Rate. Each with Lucide icon and color accent. |
| `SessionLengthChart` | Recharts `AreaChart` | `TrendPoint[]` | Purple gradient area chart showing avg session length (minutes) over time. Filters out zero-session days. |

### 5.6 Pages & Routing

| Route | File | Description |
|---|---|---|
| `/` | `app/page.tsx` | Home dashboard — login gate → timer, categories, pomodoro, notes, today's summary, history |
| `/analytics` | `app/analytics/page.tsx` | Full analytics dashboard — insights, trends, heatmap, category comparison |
| `/eod` | `app/eod/page.tsx` | EOD summary page with date navigator |

All pages read `user` from `localStorage`. No server-side auth middleware — auth is client-side only.

---

## 6. Feature Breakdown

### 6.1 Time Tracking (Core)

The fundamental feature. Users click a category to start tracking, and the system records the session.

**Flow:**
1. User clicks a category in the grid → `prompt()` asks for description → `POST /api/track/start`
2. Backend stops any existing running entry, then creates a new one with `status: 'running'`
3. Frontend polls every 5s, recalculating elapsed time client-side between polls
4. User clicks Stop → `POST /api/track/stop` → backend calculates `durationSeconds`, sets `status: 'completed'`
5. Today's totals and history update immediately via cache invalidation

**Edge cases handled:**
- UTC midnight boundary — running entries aren't lost at date rollover
- Single active session invariant — starting a new session auto-stops the previous one
- Manual entry editing with full audit trail

### 6.2 Dynamic Categories

Users can fully customize their tracking categories.

- **Auto-seeding:** First access creates 6 default categories
- **Add:** Name (1-30 chars), color (12 visual options), productive/break toggle
- **Delete:** With confirmation dialog, ownership verified server-side
- **Duplicate prevention:** Unique index on `{ userId, name }`
- **Productive classification** feeds into analytics and EOD productivity calculations

### 6.3 Pomodoro Timer

A complete Pomodoro implementation that integrates with the existing time tracking system.

**Modes:**
- `25/5` — 25 min work, 5 min break (classic)
- `50/10` — 50 min work, 10 min break
- `Custom` — User-defined durations (work: 1-180 min, break: 1-60 min)

**Lifecycle:**
1. Select mode → Click "Start Pomodoro"
2. Backend creates a TimeEntry for the work phase
3. SVG progress ring counts down, showing time remaining
4. At zero → audio beep → backend stops work entry, increments pomodoro count, auto-starts a "Break" TimeEntry
5. Break countdown completes → backend stops break entry, increments break count → return to idle

**Pause/Resume:** Freezes the countdown. The ring dims and "PAUSED" pulses. The TimeEntry continues running on the backend (time is still being tracked — the pause only affects the countdown).

**Daily tracking:** Red dots show completed pomodoros for the day (8 visible + overflow).

### 6.4 Contextual Work Notes

Notes that are linked to specific tracking sessions, providing context for what was accomplished.

- **Session linking:** Notes are tied to a TimeEntry `_id` via `linkedType: 'SESSION'`
- **Day linking:** Also supports `linkedType: 'DAY'` for day-level notes
- **Version history:** Every edit pushes the previous content to a `versions[]` array — full audit trail
- **Soft delete:** Deleted notes retain their version history
- **UI:** Only visible when a timer is running. Quick-add via Ctrl+Enter.
- **History integration:** HistoryList shows note count badges per entry

### 6.5 EOD Summaries

End-of-day reports combining automated metrics with manual narrative.

**Auto-computed (refreshed on every access):**
- Total hours worked
- Productive hours (based on user's category classifications)
- Category breakdown (category → seconds)
- Productivity score (percentage)

**User-editable (versioned):**
- Summary text (free-form narrative)
- Highlights (array of accomplishments, add/remove individually)
- Blockers (array of issues, add/remove individually)

**Date navigation:** Users can browse EODs for any past date. Cannot navigate into the future.

### 6.6 Analytics Dashboard

A comprehensive analytics page with 6 visualization types.

1. **Insight Cards (top row):** 5 key metrics — Most Productive Day, Avg Session Length, Top Category, Daily Average, Productivity Rate. Computed over the last 90 days.

2. **Productivity Summary:** 4-card grid with Productivity Score %, Focus Streak, Total Hours, Productive Hours. Switchable between day/week/month ranges.

3. **Daily Trend Line Chart:** Two lines (total vs productive hours) over 7/14/30/60 days. Includes zero-value days.

4. **Weekly Category Comparison:** Grouped bar chart showing hours per category per week over the last 4 weeks.

5. **Average Session Length:** Area chart (purple gradient) showing how session duration trends over time.

6. **Activity Heatmap:** GitHub-style contribution graph for the past year. 6 intensity levels from gray (0h) through shades of green to bright green (6h+). Day-of-week labels (Mon, Wed, Fri), month labels, legend.

### 6.7 PWA Support

The app is a **Progressive Web App** — installable on desktop and mobile.

#### Manifest (`manifest.json`)
- **Name:** "Flow State - Time Tracker"
- **Display:** Standalone (no browser chrome)
- **Theme:** `#3b82f6` (blue), Background: `#000000` (black)
- **Icons:** SVG icons at 192×192 and 512×512

#### Service Worker (`sw.js`)

**Caching strategies:**
- **Static assets** (pages, icons, manifest): **Cache-first** — serve from cache, fetch and update cache on miss
- **API GET requests**: **Network-first** — try network, cache successful responses, serve cache if offline
- **API mutations** (POST/PUT): **Try network, queue on failure** — if offline, store the request in a sync queue and return `{ success: true, queued: true }`

**Background sync:**
- Failed mutations are serialized and stored in a dedicated cache store
- When the browser comes back online, the `usePWA` hook sends a `'REPLAY_SYNC'` message to the service worker
- The SW replays all queued mutations in order
- If a replay fails, it stays in the queue for the next sync attempt

**Offline experience:**
- All pages load from cache (the app shell is cached on install)
- API data loads from cached responses
- Mutations queue for later sync
- A yellow banner ("You're offline — changes will sync when you reconnect") appears at the top

---

## 7. Data Flow & Architecture Patterns

### Request Lifecycle

```
User Action
    ↓
React Component (event handler)
    ↓
Custom Hook (useMutation / useQuery)
    ↓
fetch() to Express API
    ↓
Route → Controller (Zod validation) → Service (business logic) → Mongoose Model → MongoDB
    ↓
Response JSON
    ↓
React Query cache update → Component re-render
```

### Polling Architecture

```
useToday() ──[every 5s]──→ GET /api/track/today
                              ↓
                          TimeService.getTodayData()
                              ↓
                          { entries, totals, runningEntry }
                              ↓
                          Cache updated → Timer re-renders with new elapsed time
```

### Cache Invalidation Map

| When... | These query keys are invalidated |
|---|---|
| Start tracking | `['today', userId]`, `['history', userId]` |
| Stop tracking | `['today', userId]`, `['history', userId]` |
| Complete pomodoro work | `['pomodoro', userId]`, `['today', userId]` |
| Complete pomodoro break | `['pomodoro', userId]`, `['today', userId]` |
| Set pomodoro mode | `['pomodoro', userId]` |
| Create/update/delete note | `['notes', ...]` |
| Update EOD | `['eod', userId, date]` |
| Add/delete category | `['categories', userId]` |

---

## 8. Database Schema Relationships

```
User (1)
  ├──(many)── TimeEntry      (userId → User._id)
  ├──(many)── PomodoroSession (userId → User._id, one per day)
  ├──(many)── WorkNote        (userId → User._id, linked to TimeEntry or date)
  ├──(many)── EODSummary      (userId → User._id, one per day)
  └──(many)── Category        (userId → User._id, unique per name)

TimeEntry ←──(linked)── WorkNote  (referenceId = TimeEntry._id when linkedType = 'SESSION')
```

All relationships go through `userId` as a foreign key (Mongoose `ObjectId` ref to `User`). There are no join/populate calls in the codebase — all queries filter by `userId` directly.

---

## 9. Development Setup

### Prerequisites
- **Node.js** ≥ 18
- **MongoDB** instance (local or Atlas)

### Backend

```bash
cd backend
npm install
# Create .env file:
# MONGODB_URI=mongodb://localhost:27017/flowstate
# PORT=5000
npm run dev    # Starts tsx watch mode on port 5000
```

### Frontend

```bash
cd frontend
npm install
npm run dev    # Starts Next.js dev server on port 3000
```

### Build for Production

```bash
# Backend
cd backend
npm run build    # Compiles TypeScript to dist/
npm start        # Runs compiled JavaScript

# Frontend
cd frontend
npm run build    # Next.js production build
npm start        # Starts production server
```

---

## 10. Environment Variables

### Backend (`backend/.env`)

| Variable | Required | Default | Description |
|---|---|---|---|
| `MONGODB_URI` | ✅ | — | MongoDB connection string |
| `PORT` | ❌ | `5000` | Express server port |
| `FRONTEND_URL` | ❌ | — | Currently unused (CORS is `*`) |

### Frontend

| Variable | Required | Default | Description |
|---|---|---|---|
| `NEXT_PUBLIC_API_URL` | ❌ | `http://localhost:5000` | Backend API base URL |

---

*This documentation covers the complete codebase as of February 2026. Every model field, service method, API endpoint, component prop, hook export, and architectural decision is documented above.*
