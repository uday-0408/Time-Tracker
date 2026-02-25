# Enterprise Time Tracker - Flow State

A robust, enterprise-grade MENN stack time tracking application featuring advanced analytics, productivity scoring, intelligent idle detection, and audit history tracking. Built with modern web technologies and enterprise-level architecture patterns.

---

## 📑 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Core Functionality](#core-functionality)
- [API Documentation](#api-documentation)
- [Data Models](#data-models)
- [Frontend Components](#frontend-components)
- [Setup & Installation](#setup--installation)
- [Development Workflow](#development-workflow)
- [Environment Variables](#environment-variables)

---

## 🎯 Overview

**Flow State** is a professional time tracking application designed for developers, data analysts, and knowledge workers who need granular insights into their productivity patterns. The application automatically tracks time spent across different work categories, detects idle periods, calculates productivity scores, and provides comprehensive analytics to help users optimize their workflow.

### Main Capabilities:
- ⏱️ Real-time activity tracking with live timer
- 📊 Advanced analytics dashboard with visual charts
- 🎯 Productivity scoring based on configurable productive categories
- 🔄 Automatic idle detection and session pausing
- 📝 Audit history for time entry modifications
- 📈 Focus streak tracking to monitor consistency
- 🗂️ Historical data analysis with customizable date ranges

---

## ✨ Key Features

### 1. **Single-Session Time Tracking**
- Start and stop time tracking for different work categories
- Only one active session allowed at a time (enforced at backend)
- Real-time elapsed timer updated every second
- Categories: Python, SQL, Midas, Datasetu, Break, TT
- Optional description for each time entry
- Visual feedback with color-coded category buttons

### 2. **Intelligent Idle Detection**
- Monitors user activity (mouse, keyboard, touch, scroll)
- Automatically stops active session after 10 minutes of inactivity
- Prevents accidental time logging when away from computer
- Client-side implementation for immediate response
- Activity events tracked: `mousedown`, `mousemove`, `keydown`, `scroll`, `touchstart`

### 3. **Advanced Analytics Dashboard**
- **Productivity Score**: Percentage of time spent on productive vs non-productive activities
- **Focus Streak**: Maximum consecutive productive sessions
- **Total Hours**: Aggregate time tracked in selected period
- **Productive Hours**: Time spent on productive categories
- **Visual Charts**: Bar charts using Recharts library
- **Time Range Selection**: Daily, weekly, or monthly views
- **Color-coded Metrics**: Green for productive, blue for total, purple highlights

### 4. **Audit History Tracking**
- Every time entry modification is logged
- Tracks previous duration, start time, and end time
- Records reason for change and timestamp
- Enables accountability and correction tracking
- Useful for billing, compliance, and retrospectives

### 5. **React Query Integration**
- Optimistic UI updates
- Automatic background refetching every 5 seconds
- Smart caching to reduce network requests
- Invalidation on mutations for consistency
- DevTools integration for debugging
- Seamless loading and error states

### 6. **Modern, Responsive UI**
- Dark mode with grayscale aesthetics
- Gradient backgrounds and glassy card effects
- Mobile-responsive design (grid layouts adapt to screen size)
- Lucide React icons for consistent iconography
- Tailwind CSS for utility-first styling
- Shadcn-UI inspired component library
- Smooth transitions and hover effects

### 7. **User Authentication**
- Simple name/password based login
- Auto-creation of new users on first login
- Case-insensitive username matching
- Session persistence using localStorage
- User-specific data isolation

### 8. **Daily Summary & History**
- Today's focus view showing time per category
- Historical entries grouped by date
- Total time calculation per day
- Sortable and filterable history list
- Last 7 days history by default with configurable range

---

## 🏗️ Architecture

### Design Pattern: Controller-Service-Repository

```
┌─────────────────┐
│   Frontend      │
│   (Next.js)     │
└────────┬────────┘
         │ HTTP/REST
┌────────▼────────┐
│   Routes        │  ← Express route handlers
└────────┬────────┘
┌────────▼────────┐
│  Controllers    │  ← Request/response handling, validation
└────────┬────────┘
┌────────▼────────┐
│   Services      │  ← Business logic layer
└────────┬────────┘
┌────────▼────────┐
│   Models        │  ← Mongoose schemas & database interaction
└────────┬────────┘
         │
┌────────▼────────┐
│   MongoDB       │
└─────────────────┘
```

### Backend Architecture:
- **Routes**: Define API endpoints and HTTP methods
- **Controllers**: Handle HTTP requests, validate input with Zod schemas
- **Services**: Contain business logic, isolated from HTTP layer
- **Models**: Mongoose schemas with TypeScript interfaces

### Frontend Architecture:
- **App Router**: Next.js 14 file-based routing
- **Components**: Reusable UI components with Shadcn-UI patterns
- **Hooks**: Custom React hooks for data fetching (React Query)
- **State Management**: React Query for server state, useState for local state

---

## 🛠️ Tech Stack

### Frontend
| Technology | Version | Purpose |
|------------|---------|---------|
| **Next.js** | 14.0.4 | React framework with App Router |
| **React** | 18.2.0 | UI library |
| **TypeScript** | 5.x | Type safety |
| **TanStack React Query** | 5.90.20 | Server state management |
| **Recharts** | 3.7.0 | Data visualization |
| **Tailwind CSS** | 3.3.0 | Utility-first CSS |
| **Lucide React** | 0.563.0 | Icon library |
| **date-fns** | 4.1.0 | Date manipulation |

### Backend
| Technology | Version | Purpose |
|------------|---------|---------|
| **Node.js** | 20+ | JavaScript runtime |
| **Express** | 4.18.2 | Web framework |
| **TypeScript** | 5.3.3 | Type safety |
| **MongoDB** | - | NoSQL database |
| **Mongoose** | 8.0.3 | MongoDB ODM |
| **Zod** | 4.3.6 | Schema validation |
| **tsx** | 4.7.0 | TypeScript execution |
| **cors** | 2.8.5 | Cross-origin requests |
| **dotenv** | 16.3.1 | Environment variables |

---

## 📁 Project Structure

```
EOD maker/
├── backend/
│   ├── src/
│   │   ├── server.ts                  # Express app entry point
│   │   ├── config/
│   │   │   └── mongodb.ts             # MongoDB connection setup
│   │   ├── controllers/
│   │   │   ├── time.controller.ts     # Time tracking request handlers
│   │   │   └── analytics.controller.ts # Analytics request handlers
│   │   ├── services/
│   │   │   ├── time.service.ts        # Time tracking business logic
│   │   │   └── analytics.service.ts   # Analytics calculations
│   │   ├── models/
│   │   │   ├── TimeEntry.ts           # Time entry schema with history
│   │   │   └── User.ts                # User schema
│   │   └── routes/
│   │       ├── auth.ts                # Authentication routes
│   │       ├── track.ts               # Time tracking routes
│   │       └── analytics.ts           # Analytics routes
│   ├── package.json
│   ├── tsconfig.json
│   └── .env                           # Environment variables
│
├── frontend/
│   ├── app/
│   │   ├── layout.tsx                 # Root layout with providers
│   │   ├── page.tsx                   # Dashboard page (home)
│   │   ├── globals.css                # Global styles
│   │   └── analytics/
│   │       └── page.tsx               # Analytics page
│   ├── components/
│   │   ├── Timer.tsx                  # Real-time timer display
│   │   ├── CategoryGrid.tsx           # Category selection buttons
│   │   ├── HistoryList.tsx            # Historical entries list
│   │   ├── Login.tsx                  # Authentication form
│   │   ├── providers.tsx              # React Query provider
│   │   └── ui/
│   │       ├── button.tsx             # Reusable button component
│   │       └── card.tsx               # Reusable card component
│   ├── hooks/
│   │   ├── useTime.ts                 # Time tracking queries/mutations
│   │   ├── useAnalytics.ts            # Analytics queries
│   │   ├── useHistory.ts              # History queries
│   │   └── useIdle.ts                 # Idle detection hook
│   ├── lib/
│   │   └── utils.ts                   # Utility functions (cn, etc.)
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.js
│   ├── tailwind.config.ts
│   └── postcss.config.js
│
└── README.md
```

---

## ⚙️ Core Functionality

### Time Tracking Flow

1. **Starting a Session**:
   ```
   User clicks category → Prompts for description → 
   API call to /api/track/start → Stops any running entry → 
   Creates new entry with status 'running' → 
   React Query invalidates cache → UI updates
   ```

2. **Active Session**:
   ```
   Timer component polls start time → Calculates elapsed seconds → 
   Displays HH:MM:SS format → Updates every second → 
   Idle hook monitors activity → Auto-stops after 10min inactivity
   ```

3. **Stopping a Session**:
   ```
   User clicks stop → API call to /api/track/stop → 
   Finds running entry → Calculates duration → 
   Sets endTime and status 'completed' → 
   React Query invalidates → Timer resets
   ```

### Analytics Calculation

**Productivity Score Formula**:
```
Productive Categories: ['Python', 'SQL', 'Midas', 'Datasetu']
Productivity Score = (Productive Seconds / Total Seconds) × 100
```

**Focus Streak Logic**:
- Increments for each consecutive productive session
- Resets to 0 on non-productive session
- Tracks maximum streak achieved in time range

### Data Persistence

- All time entries stored with UTC timestamps
- Date field stores ISO date string (YYYY-MM-DD)
- Compound indexes on `{userId, date}` and `{userId, status}` for query performance
- Audit history stored as embedded documents in time entries

---

## 🔌 API Documentation

### Base URL
```
http://localhost:5000/api
```

### Authentication

#### POST `/auth/login`
**Description**: Login or register a user

**Request Body**:
```json
{
  "name": "John Doe",
  "password": "password123"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "user": {
    "id": "6789abcd1234567890abcdef",
    "name": "John Doe"
  }
}
```

**Error Response** (401 Unauthorized):
```json
{
  "success": false,
  "message": "Invalid password"
}
```

---

### Time Tracking

#### POST `/track/start`
**Description**: Start a new time tracking session

**Request Body**:
```json
{
  "userId": "6789abcd1234567890abcdef",
  "category": "Python",
  "description": "Working on authentication module"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "entry": {
    "_id": "entry123",
    "userId": "user123",
    "category": "Python",
    "description": "Working on authentication module",
    "startTime": "2026-02-24T10:30:00.000Z",
    "endTime": null,
    "status": "running",
    "date": "2026-02-24",
    "durationSeconds": 0
  }
}
```

#### POST `/track/stop`
**Description**: Stop the currently running session

**Request Body**:
```json
{
  "userId": "6789abcd1234567890abcdef"
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "entry": {
    "_id": "entry123",
    "category": "Python",
    "startTime": "2026-02-24T10:30:00.000Z",
    "endTime": "2026-02-24T11:45:00.000Z",
    "durationSeconds": 4500,
    "status": "completed"
  }
}
```

#### GET `/track/today?userId={userId}`
**Description**: Get today's entries and totals for a user

**Response** (200 OK):
```json
{
  "success": true,
  "entries": [...],
  "totals": {
    "Python": 4500,
    "SQL": 3600,
    "Break": 900
  },
  "runningEntry": {
    "_id": "entry456",
    "category": "SQL",
    "startTime": "2026-02-24T14:00:00.000Z"
  }
}
```

#### GET `/track/history?userId={userId}&days={days}`
**Description**: Get historical entries grouped by date

**Parameters**:
- `userId` (required): User ID
- `days` (optional): Number of days to retrieve (default: 7)

**Response** (200 OK):
```json
{
  "success": true,
  "history": [
    {
      "date": "2026-02-24",
      "entries": [...],
      "totals": {
        "Python": 7200,
        "SQL": 3600
      },
      "totalSeconds": 10800
    }
  ]
}
```

#### PUT `/track/update`
**Description**: Update a time entry (creates audit history)

**Request Body**:
```json
{
  "entryId": "entry123",
  "userId": "user123",
  "updates": {
    "startTime": "2026-02-24T10:00:00.000Z",
    "endTime": "2026-02-24T11:00:00.000Z",
    "description": "Updated description",
    "category": "Python"
  }
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "entry": {
    "_id": "entry123",
    "durationSeconds": 3600,
    "history": [
      {
        "updatedAt": "2026-02-24T12:00:00.000Z",
        "reason": "Manual edit",
        "previousDuration": 4500,
        "previousStartTime": "2026-02-24T10:30:00.000Z",
        "previousEndTime": "2026-02-24T11:45:00.000Z"
      }
    ]
  }
}
```

---

### Analytics

#### GET `/analytics/productivity?userId={userId}&range={range}`
**Description**: Get productivity statistics

**Parameters**:
- `userId` (required): User ID
- `range` (required): 'day' | 'week' | 'month'

**Response** (200 OK):
```json
{
  "success": true,
  "data": {
    "totalHours": "45.50",
    "productiveHours": "38.25",
    "productivityScore": 84,
    "maxFocusStreak": 12
  }
}
```

---

## 📊 Data Models

### TimeEntry Schema
```typescript
{
  _id: ObjectId,
  userId: ObjectId,              // Reference to User
  category: String,              // Python, SQL, Midas, etc.
  description: String,           // Optional task description
  startTime: Date,              // ISO timestamp
  endTime: Date | null,         // ISO timestamp or null if running
  date: String,                 // ISO date string (YYYY-MM-DD)
  durationSeconds: Number,      // Calculated duration
  status: 'running' | 'completed' | 'paused',
  isIdle: Boolean,              // If stopped due to inactivity
  history: [                    // Audit trail
    {
      updatedAt: Date,
      reason: String,
      previousDuration: Number,
      previousStartTime: Date,
      previousEndTime: Date
    }
  ],
  createdAt: Date,              // Auto-generated
  updatedAt: Date               // Auto-generated
}
```

**Indexes**:
- `{ userId: 1, date: 1 }` - Fast daily queries
- `{ userId: 1, status: 1 }` - Find running entries
- `{ startTime: 1 }` - Time-range queries

### User Schema
```typescript
{
  _id: ObjectId,
  name: String,         // Username (unique, case-insensitive)
  password: String,     // Plain text (⚠️ use bcrypt in production)
  createdAt: Date,
  updatedAt: Date
}
```

---

## 🎨 Frontend Components

### Core Components

#### `<Timer />`
- **Purpose**: Display active session with live countdown
- **Props**: `runningEntry`, `onStop`, `isLoading`
- **Features**: 
  - Real-time elapsed time calculation
  - HH:MM:SS format display
  - Large, readable font with gradient card
  - Stop button with loading state

#### `<CategoryGrid />`
- **Purpose**: Category selection buttons
- **Props**: `onStart`, `activeCategory`, `isLoading`
- **Features**:
  - 2-3 column responsive grid
  - Color-coded categories
  - Active state highlighting
  - Hover animations (scale)

#### `<HistoryList />`
- **Purpose**: Display past time entries
- **Features**:
  - Date grouping
  - Time formatting
  - Category badges
  - Total time per day

#### `<Login />`
- **Purpose**: Authentication form
- **Props**: `onLogin`
- **Features**:
  - Simple name/password inputs
  - Auto-creates new users
  - Error handling

### Custom Hooks

#### `useTime(userId)`
- **Returns**: Today's data with 5-second polling
- **Methods**: `useStartTracking()`, `useStopTracking()`
- **Invalidations**: Auto-refreshes on mutations

#### `useAnalytics(userId, range)`
- **Returns**: Productivity stats for given range
- **Caching**: React Query caches by userId and range

#### `useIdle(timeoutSeconds)`
- **Returns**: Boolean indicating idle state
- **Default**: 600 seconds (10 minutes)
- **Events**: Tracks mouse, keyboard, touch, scroll

---

## 🚀 Setup & Installation

### Prerequisites
- Node.js 20+ 
- MongoDB (local or Atlas)
- npm or yarn

### Backend Setup

1. **Navigate to backend directory**:
   ```bash
   cd backend
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Create `.env` file** in `backend/` directory:
   ```env
   MONGODB_URI=mongodb://localhost:27017/time-tracker
   PORT=5000
   FRONTEND_URL=http://localhost:3000
   ```

4. **Run development server**:
   ```bash
   npm run dev
   ```
   Server will start on `http://localhost:5000`

5. **Build for production** (optional):
   ```bash
   npm run build
   npm start
   ```

### Frontend Setup

1. **Navigate to frontend directory**:
   ```bash
   cd frontend
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Create `.env.local` file** (optional):
   ```env
   NEXT_PUBLIC_API_URL=http://localhost:5000
   ```

4. **Run development server**:
   ```bash
   npm run dev
   ```
   Application will start on `http://localhost:3000`

5. **Build for production** (optional):
   ```bash
   npm run build
   npm start
   ```

### MongoDB Setup

**Option 1: Local MongoDB**
```bash
# Install MongoDB and start service
mongod --dbpath /path/to/data
```

**Option 2: MongoDB Atlas**
```
1. Create free cluster at mongodb.com/cloud/atlas
2. Get connection string
3. Update MONGODB_URI in .env
```

---

## 💻 Development Workflow

### Running the Full Stack
```bash
# Terminal 1 - Backend
cd backend
npm run dev

# Terminal 2 - Frontend
cd frontend
npm run dev
```

### Adding a New Feature

1. **Backend**:
   - Create service method in `services/`
   - Add controller in `controllers/`
   - Define route in `routes/`
   - Update model if needed

2. **Frontend**:
   - Create React Query hook in `hooks/`
   - Build UI component in `components/`
   - Add to page in `app/`

### Code Style
- **TypeScript**: Strict mode enabled
- **Formatting**: Standard Prettier config
- **Naming**: camelCase for variables, PascalCase for components
- **File naming**: kebab-case for utilities, PascalCase for components

---

## 🔐 Environment Variables

### Backend `.env`
```env
# Database
MONGODB_URI=mongodb://localhost:27017/time-tracker

# Server
PORT=5000

# CORS (optional)
FRONTEND_URL=http://localhost:3000
```

### Frontend `.env.local`
```env
# API endpoint
NEXT_PUBLIC_API_URL=http://localhost:5000
```

---

## 🎯 Key Design Decisions

### Why Controller-Service Pattern?
- **Separation of concerns**: HTTP logic separate from business logic
- **Testability**: Services can be unit tested independently
- **Reusability**: Services can be called from multiple controllers

### Why React Query?
- **Automatic caching**: Reduces unnecessary API calls
- **Background refetching**: Keeps data fresh
- **Optimistic updates**: Instant UI feedback
- **Built-in loading/error states**: Less boilerplate

### Why Single Active Session?
- **Prevents time overlap**: One task at a time
- **Cleaner analytics**: No ambiguous time allocation
- **Better focus tracking**: Encourages mindful task switching

### Why Idle Detection?
- **Accuracy**: Prevents logging time when away
- **Automatic**: No manual intervention needed
- **Configurable**: Can adjust timeout threshold

---

## 📈 Future Enhancements

- [ ] Export reports to CSV/PDF
- [ ] Team collaboration and shared dashboards
- [ ] Integration with calendar apps (Google Calendar, Outlook)
- [ ] Browser extension for tracking
- [ ] Mobile app (React Native)
- [ ] Custom category creation
- [ ] Goal setting and notifications
- [ ] Tag system for granular categorization
- [ ] Pomodoro timer integration
- [ ] Real-time collaboration with WebSockets
- [ ] Advanced reporting (weekly/monthly summaries)
- [ ] Bcrypt password hashing for production
- [ ] JWT authentication with refresh tokens
- [ ] Role-based access control (admin/user)

---

## 📝 License

This project is for educational and personal use.

---

## 👤 Author

Built with ❤️ for productivity enthusiasts
