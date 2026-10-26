# PathAble — Civic Accessibility Intelligence & Navigation Platform

> **A competition-quality, full-stack MERN application empowering individuals with mobility challenges to navigate campuses safely while enabling communities and facility administrators to identify, verify, and resolve physical accessibility barriers.**

---

## 1. Executive Summary & Project Pitch

### The Problem
Traditional navigation tools assume users can climb steps, navigate uneven terrain, or push heavy doors. When an elevator fails or an access ramp is obstructed, a wheelchair user or visually impaired student is stranded without notice. Furthermore, campus facilities departments often lack a centralized, crowd-verified system to prioritize and resolve physical accessibility barriers.

### The Innovation
**PathAble** solves this through three integrated capabilities:
1. **Multi-Criteria Accessible Route Engine**: Implements Dijkstra’s algorithm parameterized with mobility profiles (manual wheelchair, powered scooter, cane/walker, visual impairment) that enforces hard physical exclusions (e.g., stairs for wheelchairs, slopes > 8.5%) and applies safety penalties to unverified segments.
2. **Community Verification Lifecycle**: Allows campus community members to report and crowd-verify barrier obstacles (Submitted → Under Review → Verified → In Progress → Resolved) with duplicate voting prevention.
3. **AI-Assisted Triage & Facilities Administration**: Automatically analyzes barrier descriptions to suggest categories, estimate severity, prompt for missing details, generate concise maintenance work-order briefs, and visualize resolution velocity on an interactive analytics dashboard.

### Community Impact
PathAble adheres to a strict principle of safety transparency: **an unknown route is never labeled safe**. Paths are categorized into verified step-free corridors, moderate slope ramps, and stairs/obstacles, providing dignity, autonomy, and security.

---

## 2. System Architecture

```
                                  ┌─────────────────────────────────────────┐
                                  │             React 18 + Vite             │
                                  │      Tailwind CSS / Lucide Icons        │
                                  │   English & Malayalam Localization (ML) │
                                  └────────────────────┬────────────────────┘
                                                       │  HTTP / REST API (JWT Auth)
                                                       ▼
                                  ┌─────────────────────────────────────────┐
                                  │          Express 4.21 API Server        │
                                  │    Node.js v22  •  Rate Limiting / CORS │
                                  └─────┬──────────────┬──────────────┬─────┘
                                        │              │              │
                     ┌──────────────────┴──┐           │       ┌──────┴──────────────────┐
                     ▼                     ▼           ▼       ▼                         ▼
             ┌───────────────┐     ┌──────────────┐ ┌─────────────┐ ┌─────────────────────────┐
             │  Route Engine │     │  AI Service  │ │ Multer Disk │ │     MongoDB Models      │
             │   (Dijkstra)  │     │ (Gemini/NLP) │ │ File Upload │ │ User, BarrierReport,    │
             │Hard Exclusions│     │  Triage Core │ │  5MB Limit  │ │ Facility, CampusNode,   │
             │Penalties/Slope│     │ Rule-Base FB │ └─────────────┘ │ CampusEdge, ReportVote, │
             └───────────────┘     └──────────────┘                 │ AuditLog, Notification  │
                                                                    └───────────┬─────────────┘
                                                                                │
                                                                   ┌────────────┴────────────┐
                                                                   ▼                         ▼
                                                          MongoDB Atlas / Local     MongoMemoryServer
                                                             (Production URI)       (Zero-Config Demo)
```

---

## 3. Technology Stack

- **Frontend**: React 18, Vite, Tailwind CSS, Lucide React, Leaflet, React-Leaflet, Chart.js, React-ChartJS-2.
- **Backend**: Node.js, Express 4, Mongoose 8, JWT (`jsonwebtoken`), bcryptjs, Multer, Morgan, Express-Rate-Limit.
- **Database**: MongoDB (supports MongoDB Atlas, local MongoDB, or automatic embedded zero-config `mongodb-memory-server` for offline testing and live exhibitions).
- **Testing**: Vitest 3, Supertest.
- **AI/NLP**: Server-side Google Gemini 1.5 Flash API proxy with a built-in deterministic heuristic NLP fallback (runs 100% offline without requiring external API keys).

---

## 4. Setup & Installation (Fedora / Linux)

### Prerequisites
- Node.js (v18+ or v22+)
- npm (v9+ or v10+)

### 1. Clone or Open Project
```bash
cd /home/nazal/.gemini/antigravity/scratch/pathable
```

### 2. Configure Environment Variables
Copy backend environment template:
```bash
cp backend/.env.example backend/.env
```

Default `.env` settings:
```env
PORT=5000
NODE_ENV=development
# Leave MONGODB_URI empty to automatically use zero-config in-memory MongoDB for local evaluation:
MONGODB_URI=
JWT_SECRET=pathable_super_secure_jwt_secret_key_change_in_production_123!
JWT_EXPIRES_IN=7d
CLIENT_URL=http://localhost:3000
# Optional Gemini API Key (falls back automatically to built-in NLP engine if empty):
GEMINI_API_KEY=
```

### 3. Install All Dependencies
```bash
npm run install:all
```

---

## 5. Launching the Application

### Option A: Concurrent Development Mode (Recommended)
In terminal 1 (Backend):
```bash
npm run dev:backend
```
In terminal 2 (Frontend):
```bash
npm run dev:frontend
```

Frontend will be accessible at: `http://localhost:3000`
Backend API will be accessible at: `http://localhost:5000/api/v1`

---

## 6. Pre-Seeded Demonstration Accounts

The platform includes pre-seeded demonstration accounts for university exhibitions:

| Role | Account Email | Password | Purpose |
|---|---|---|---|
| **Admin** | `admin@pathable.edu` | `AdminPassword123!` | Full oversight, status lifecycle transitions, resolution velocity analytics, edge closing. |
| **Maintenance** | `maintenance@pathable.edu` | `MaintPassword123!` | Facilities triage, work orders, repair notes, and campus edge updates. |
| **Student** | `student@pathable.edu` | `StudentPassword123!` | Wheelchair user profile, barrier reporting, route calculation, community voting. |

*(The Sign-In page includes **1-Click Fill** buttons for immediate demonstration without typing credentials).*

---

## 7. MongoDB Atlas Configuration Guide

To connect PathAble to a hosted MongoDB Atlas cluster instead of local/in-memory mode:

1. Log in to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
2. Create a free **M0 Sandbox** cluster.
3. Under **Database Access**, create a user with `Read and Write to any database` privileges.
4. Under **Network Access**, add `0.0.0.0/0` (or your static IP address) to the IP access list.
5. Click **Connect** → **Drivers (Node.js)** and copy the connection string:
   ```
   mongodb+srv://<username>:<password>@cluster0.xxxx.mongodb.net/pathable?retryWrites=true&w=majority
   ```
6. Paste into `backend/.env`:
   ```env
   MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.xxxx.mongodb.net/pathable?retryWrites=true&w=majority
   ```
7. Restart the backend. PathAble will automatically detect the database and seed the campus graph.

---

## 8. API Endpoint Documentation (`/api/v1`)

### Authentication
- `POST /api/v1/auth/register`: Register new user with mobility profile.
- `POST /api/v1/auth/login`: Authenticate and receive JWT.
- `GET /api/v1/auth/me`: Get authenticated user profile (`Bearer <token>`).
- `PUT /api/v1/auth/profile`: Update mobility profile and personal preferences.

### Barrier Reports
- `GET /api/v1/reports`: Paginated list of barriers with filters (`category`, `severity`, `status`, `verifiedOnly`, `near`).
- `GET /api/v1/reports/:id`: Get detailed report with vote counts, audit trail, and photo.
- `POST /api/v1/reports`: Submit new barrier report with coordinates, optional photo upload (Multer), and automatic AI triage.
- `PUT /api/v1/reports/:id`: Edit user-submitted report (allowed only before verification).
- `PUT /api/v1/reports/:id/status`: Administrative status transition (`Verified`, `In Progress`, `Resolved`, `Rejected`, `Duplicate`). Links to campus edge.
- `POST /api/v1/reports/:id/vote`: Submit community confirmation or dispute vote (duplicate votes prevented).
- `POST /api/v1/reports/ai-triage`: Real-time AI analysis of title and description.

### Accessible Route Engine & Campus Graph
- `POST /api/v1/routes/calculate`: Multi-criteria Dijkstra calculation between start and destination nodes.
- `GET /api/v1/routes/graph`: Retrieve campus graph nodes and edges with active barrier overlays.
- `PUT /api/v1/routes/edges/:edgeId`: (Admin) Manually block, unblock, or modify walkway attributes.

### Facilities
- `GET /api/v1/facilities`: List campus facilities, zones, and accessibility feature flags.
- `GET /api/v1/facilities/:id`: Detailed facility accessibility specs.

### Administration & Analytics
- `GET /api/v1/admin/analytics`: Resolution rate, average resolution hours, category and severity breakdowns, timeline trend, hotspots.
- `GET /api/v1/admin/audit-logs`: Paginated audit log of institutional status transitions.
- `POST /api/v1/admin/seed-reset`: Development reset of demonstration campus dataset.

---

## 9. Automated Testing & Quality Checks

Run the automated test suite across backend and frontend:
```bash
npm test
```

### Verified Test Results
- **Route Engine Unit Tests**: 5/5 Passing
  - Dijkstra shortest path with stairs
  - Wheelchair profile avoiding stairs and routing via accessible ramp
  - Blocked edge detection and graceful fallback suggestion
  - Disconnected graph fallback
  - Penalty for unverified edges in cautious mode
- **Backend API Integration Tests**: 13/13 Passing
  - Health check endpoint verification
  - User and Administrator authentication
  - Password hashing and invalid credentials rejection
  - Registration with mobility profile
  - Paginated report querying
  - Barrier report creation with AI triage
  - Community confirm voting
  - Duplicate vote prevention
  - Admin/maintenance lifecycle transition
  - Unauthorized user status update rejection
  - Accessible route calculation on campus graph
  - Admin analytics aggregation calculation
- **Frontend Localization Integrity Tests**: 5/5 Passing
  - English and Malayalam dictionary synchronization
  - Complete parity for all categories, mobility profiles, and lifecycle statuses
- **Build Verification**:
  - `npm run build:frontend` compiled with zero errors.

---

## 10. Recommended Live Demonstration Sequence

For a university project exhibition or hackathon evaluation, follow this 5-minute walkthrough:

1. **Step 1: Problem & Landing Page**:
   - Open `http://localhost:3000`.
   - Point out the 3 core questions, live dynamically computed statistics, and civic accessibility mission.
   - Switch language to **Malayalam (മലയാളം)** to demonstrate localization, then toggle **Dark Mode**.
2. **Step 2: Interactive Campus Map & Route Calculation**:
   - Navigate to **Accessibility Map**.
   - Review the Grandview University demonstration campus graph, showing verified step-free corridors, ramps, and facilities (Library, Student Union, Science Complex).
   - In the **Route Planner**, set:
     - Origin: *Main Campus South Gate*
     - Destination: *Central Library Entrance*
     - Mobility Profile: *Standard (Walking)*
     - Mode: *Shortest Distance*
     - Click **Calculate Accessible Route**: Notice the route takes the direct monumental stone stairs (100m).
   - Now switch Mobility Profile to **Manual Wheelchair (Strict Step-Free)**:
     - Click **Calculate Accessible Route**: Observe that the route automatically excludes the stairs and routes around via the gentle 2.1% West Accessible Ramp (140m).
     - Point out the step-by-step turn guidance and the *“Why this route was chosen”* rationale.
3. **Step 3: Community Barrier Reporting with AI Triage**:
   - Navigate to **Report Barrier**.
   - Click on the interactive map to select coordinates near the Student Union or Library.
   - Enter title: *“Hydraulic elevator malfunction in science building”*.
   - Enter description: *“The wheelchair elevator doors are jammed and flashing error code 42 on the ground floor lobby.”*
   - Click **Analyze with AI Assistant**:
     - Observe the AI suggest *Broken Elevator* category and *Critical* severity with explanation.
     - Note the follow-up questions identifying missing details (e.g. alternative detour).
     - Click **Apply** and submit the report.
4. **Step 4: Community Verification Voting**:
   - Find the report on the map or in the list.
   - Click the **Thumbs Up (Confirm)** button to cast a community confirmation vote.
   - Click it again to verify that the system blocks duplicate votes.
5. **Step 5: Administrative Dashboard & Hotspots**:
   - Click **Sign In** and use the **1-Click Demo Admin** button (`admin@pathable.edu`).
   - Open the **Admin Dashboard**:
     - Inspect the live resolution rate %, average resolution time in hours, category doughnut chart, and recurring barrier hotspots.
     - In the **Barrier Management Table**, click **Manage** on an active report.
     - Transition status from `Submitted` → `Verified` → `In Progress` → `Resolved`, enter resolution notes, and save.
     - Check the **Institutional Audit Trail** to view the recorded timestamped audit log.
