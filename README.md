# SLCTS — Smart Logistics & Cargo Transit System

SLCTS is a full-stack logistics platform for managing cargo transit operations in Uganda. It connects three applications through a shared Core API:

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Admin Command Dashboard** | React 19 · TypeScript · Vite · Tailwind CSS | Fleet oversight, driver verification, bookings, financial ledger |
| **Driver Mobile Portal** | Flutter · Dart · BLoC | Driver onboarding, trip accept/complete, GPS telemetry |
| **Core API** | Node.js · TypeScript · Express · PostgreSQL | REST authentication, business logic, WebSocket dispatch |

### What the system does

- **Admins** log in via the web dashboard to view live fleet telemetry on a map, approve drivers, monitor bookings, and release escrow payments from the transaction ledger.
- **Drivers** register by phone on the mobile app, go online to receive booking offers, accept trips, stream GPS coordinates, and mark deliveries complete.
- **The API** enforces JWT role-based access, stores all data in PostgreSQL with strict UGX decimal rules (15% platform commission), and pushes real-time booking offers over WebSocket.

```
┌─────────────────┐     REST /api      ┌─────────────────┐
│  Admin Web UI   │◄──────────────────►│                 │
│  (port 5173)    │     WebSocket      │   Core API      │
└─────────────────┘◄──────────────────►│  HTTP :3001     │
                                         │  WS   :4000     │
┌─────────────────┐     REST /api      │                 │
│  Flutter Driver │◄──────────────────►│   PostgreSQL    │
│  App (mobile)   │     WebSocket      └─────────────────┘
└─────────────────┘
```

---

## Requirements

Install the following before running the system locally.

### Core (required for API + Admin dashboard)

| Tool | Version | Download |
|------|---------|----------|
| **Node.js** | 18+ | [nodejs.org](https://nodejs.org/) |
| **npm** | 9+ | Included with Node.js |
| **PostgreSQL** | 14+ | [postgresql.org](https://www.postgresql.org/download/) |
| **psql CLI** | — | Included with PostgreSQL |

### Admin dashboard (additional)

| Tool | Purpose |
|------|---------|
| **Google Maps JavaScript API key** | Live fleet map — [Google Cloud Console](https://console.cloud.google.com/google/maps-apis) |

### Driver mobile app (optional — for mobile testing)

| Tool | Version | Download |
|------|---------|----------|
| **Flutter SDK** | ≥ 3.3 | [flutter.dev](https://flutter.dev/docs/get-started/install) |
| **Android Studio** | latest | Android emulator and SDK |
| **Xcode** | latest (macOS) | iOS Simulator |
| **Chrome** | latest | Fastest local Flutter web testing |

### Physical device testing (optional)

| Tool | Purpose |
|------|---------|
| **ngrok** or similar | Tunnel localhost API to a phone on another network — see [frontend/README.md](frontend/README.md) |

---

## Project structure

```
SCLST/
├── backend/          # Node.js Core API (Express + PostgreSQL + WebSocket)
├── src/              # React Admin Dashboard
├── mobile/           # Flutter Driver App
├── frontend/         # Frontend portal documentation
├── package.json      # Admin web dependencies
└── .env.example      # Admin web environment template
```

---

## Installation & run guide

Follow these steps **in order**. Three terminal windows are needed for the full system.

### Step 1 — Clone the repository

```bash
git clone https://github.com/kAWAMIRIA23/SCLST.git
cd SCLST
```

### Step 2 — Set up PostgreSQL

**Windows (PowerShell):**

```powershell
cd backend
.\setup-db.ps1
```

**Manual (any OS):**

```bash
psql -U postgres -c "CREATE DATABASE slcts;"
psql postgresql://postgres:YOUR_PASSWORD@localhost:5432/slcts -f backend/sql/schema.sql
```

### Step 3 — Configure the backend

Create `backend/.env` (copy from `backend/.env.example`):

```env
DATABASE_URL=postgresql://postgres:YOUR_PASSWORD@localhost:5432/slcts
JWT_SECRET=your_minimum_64_character_random_secret_string_change_this_in_production
PORT=3001
WS_PORT=4000
JWT_EXPIRES_IN=8h
BCRYPT_SALT_ROUNDS=12
NODE_ENV=development
```

Install dependencies and seed the admin user:

```bash
cd backend
npm install
npm run seed:admin
```

Default admin credentials:

- **Email:** `admin@slcts.ug`
- **Password:** `AdminSecure@2025`

### Step 4 — Start the Core API

```bash
cd backend
npm run dev
```

You should see:

```
SLCTS API listening on http://localhost:3001
Telemetry WebSocket listening on ws://localhost:4000/telemetry
```

Leave this terminal running.

### Step 5 — Configure and start the Admin Dashboard

Open a **new terminal** from the repository root.

Create `.env` (copy from `.env.example`):

```env
VITE_GOOGLE_MAPS_API_KEY=your_google_maps_api_key
VITE_WS_URL=ws://localhost:4000/telemetry
```

Install and run:

```bash
npm install
npm run dev
```

Open **http://localhost:5173** and log in with the admin credentials above.

### Step 6 — Start the Driver Mobile App (optional)

Open a **third terminal**:

```bash
cd mobile
flutter pub get
flutter run -d chrome \
  --dart-define=BASE_URL=http://localhost:3001 \
  --dart-define=WS_URL=ws://localhost:4000/telemetry
```

On **Windows**, you can use the helper script:

```powershell
cd mobile
.\run-chrome.ps1
```

---

## Quick reference — ports & URLs

| Service | URL |
|---------|-----|
| Admin Dashboard | http://localhost:5173 |
| Core API (HTTP) | http://localhost:3001 |
| WebSocket Telemetry | ws://localhost:4000/telemetry |
| PostgreSQL | localhost:5432 / database `slcts` |

---

## Production build

```bash
# Backend
cd backend && npm run build && npm start

# Admin web
npm run build && npm run preview
```

---

## Further documentation

| Document | Contents |
|----------|----------|
| [backend/README.md](backend/README.md) | API architecture, `.env` reference, route manifest |
| [frontend/README.md](frontend/README.md) | Admin + Flutter setup, ngrok mobile bridging |
| [backend/sql/schema.sql](backend/sql/schema.sql) | PostgreSQL DDL — tables, enums, CHECK constraints |

---

## License

Private — all rights reserved.
