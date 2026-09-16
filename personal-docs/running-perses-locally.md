# Running Perses Locally: Frontend, Backend & User Guide

This guide provides a comprehensive walkthrough on how to start both the backend API server and frontend UI, how they communicate, and how to navigate and use Perses once it is running.

---

## 🏗️ Architecture Overview for Local Development

Perses runs as two separate processes in development:

```text
┌──────────────────────────────────────┐       ┌──────────────────────────────────────┐
│        Frontend (Rspack Dev)         │       │          Backend API Server          │
│        http://localhost:3000         │       │        http://localhost:8080         │
│                                      │       │                                      │
│  UI App (React + MUI)                │       │  Go HTTP Server (Echo framework)     │
│  Dev Proxy:                          │ API   │  Config: dev/config.yaml             │
│  /api, /proxy, /plugins ─────────────┼──────>│  Database: dev/local_db (file store) │
│                                      │       │  Seed Data: dev/data                 │
└──────────────────────────────────────┘       └──────────────────────────────────────┘
```

The frontend dev server on port `3000` automatically proxies all `/api`, `/proxy`, and `/plugins` requests to `http://localhost:8080`, so there are no CORS complications.

---

## 1. Running the Backend Server

The backend runs on **`http://localhost:8080`**. It uses `dev/config.yaml` and seeds sample projects, dashboards, and users from `dev/data/` into a local file database at `dev/local_db/`.

### Default Login Credentials
- **Username:** `admin`
- **Password:** `password`

Choose one of the following methods to start the backend:

### Option A: Using Docker (Quickest if Go is not installed)

You can run Perses in a local Docker container:

```powershell
# From the project root:
docker build -t perses-dev -f Dockerfile.dev .
docker run -d --name perses-backend -p 8080:8080 perses-dev
```

To stop or view logs:
```powershell
docker logs -f perses-backend
docker stop perses-backend
```

---

### Option B: Native Go (Recommended for backend or full-stack work)

**Prerequisites:** Go `1.26.x` installed and in your PATH.

#### On Windows (Git Bash):
```bash
# 1. Generate embedded UI assets (required because ui/embed.go is gitignored)
./scripts/compress_assets.sh

# 2. Build the backend binary
go build -o ./bin/perses.exe ./cmd/perses

# 3. Start the server with the development config
./bin/perses.exe --config ./dev/config.yaml --log.level=debug
```

#### On Windows (PowerShell):
```powershell
# 1. Generate embedded UI assets via Git Bash (or run 'make assets-compress')
& "C:\Program Files\Git\bin\bash.exe" ./scripts/compress_assets.sh

# 2. Build the backend binary
go build -o ./bin/perses.exe ./cmd/perses

# 3. Start the server with the development config
.\bin\perses.exe --config .\dev\config.yaml --log.level=debug
```

> [!NOTE]
> If you skip generating the static assets, Go will fail to compile with:
> `ui\endpoint.go:46:35: undefined: embedFS`
> This happens because `ui/embed.go` defines `embedFS` and is generated at build time.

#### On Linux / macOS:
```bash
./scripts/api_backend_dev.sh
# Or manually:
./scripts/compress_assets.sh
go build -o ./bin/perses ./cmd/perses
```

### Verifying the Backend is Healthy
Open your browser or run:
```powershell
curl http://localhost:8080/api/v1/health
```
It should return a `200 OK` or JSON status.

---

## 2. Running the Frontend Web App

The frontend is a React application located in the `ui/` directory.

### Prerequisites
- **Node.js**: `>= 24` (see `ui/.nvmrc`: `v24.19.0`)
- **NPM**: `>= 11`

### Step 1: Install Dependencies
Open a terminal in the `ui/` folder:
```powershell
cd "d:\Open Source\LFX\perses\ui"
npm install
```

### Step 2: Start the Development Server

#### On Windows:
```powershell
# Run directly with npm workspaces (avoids Turborepo space-escaping bug on Windows):
npm run start -w app
```
*(Alternatively: `cd app` then `npm run start`)*

#### On Linux / macOS:
```bash
npm run start
```

### Step 3: Open the Web Application
Navigate to:
👉 **`http://localhost:3000/`**

---

## 3. How to Use Perses: A Guided Tour

### 🔑 Step 1: Sign In
1. Opening `http://localhost:3000` will redirect you to `/sign-in` (if native authentication is enabled).
2. Enter the credentials:
   - **Username:** `admin`
   - **Password:** `password`
3. Click **Sign In**.

---

### 🏠 Step 2: Explore the Home Page
Once logged in, the Home page displays:
- **Important Dashboards:** Pre-seeded dashboards such as `Demo`, `ThreeSignals`, `NodeExporter`, and `Benchmark`.
- **Recent Dashboards:** Dashboards you have visited recently.
- **Starred Dashboards:** Dashboards you have marked with a star for quick access.

---

### 📁 Step 3: Projects Management
In Perses, dashboards and configurations belong to **Projects**.
1. Click **Projects** in the top navigation or sidebar.
2. You will see pre-configured projects like `perses` and `testing`.
3. **Create a New Project:**
   - Click the **+ Add Project** button.
   - Enter a name (e.g. `my-test-project`) and description.
   - Click **Save**.

---

### 📊 Step 4: Viewing & Editing Dashboards
1. Open the `perses` project and click on the **`Demo`** dashboard.
2. **Dashboard Controls:**
   - **Time Range Picker (top right):** Change time window (e.g., *Last 1 hour*, *Last 6 hours*, or custom dates).
   - **Refresh Interval:** Set auto-refresh (e.g., *Off*, *30s*, *1m*).
   - **Variables:** Use dropdown selectors at the top to filter panel queries dynamically.
3. **Editing the Dashboard:**
   - Click the **Pencil (Edit)** icon on the top right.
   - **Rearrange panels:** Click and drag panel headers to change layout.
   - **Resize panels:** Drag the bottom-right corner of any panel.
   - **Add a Panel:** Click **+ Add Panel**, select a visualization (e.g., *Time Series*, *Gauge*, *Stat*, or *Markdown*), configure the query and visual options, and click **Apply**.
   - **Save:** Click **Save** in the top action bar to persist changes.

---

### 🔍 Step 5: Explore Mode (Ad-Hoc Querying)
1. Click **Explore** in the navigation bar (`/explore`).
2. Explore mode lets you test PromQL queries and view live data without having to create or modify a dashboard.
3. Select a datasource and type a query (e.g. `up` or `node_cpu_seconds_total`) to render charts instantly.

---

### ⚙️ Step 6: Administration & Settings
Click the **Admin** link in the navigation menu (`/admin`):
- **Users:** View existing users or create new accounts.
- **Roles & Role Bindings:** Configure Role-Based Access Control (RBAC) permissions.
- **Global Datasources:** Manage datasources (such as Prometheus endpoints) shared across all projects.
- **Global Variables:** Variables accessible by all dashboards in any project.

---

## 4. Resetting Sample Data

If you modify or delete dashboards/projects and want to reset the database back to initial state:

1. Stop the backend server.
2. Delete the contents of `dev/local_db`:
   ```powershell
   # In project root:
   Remove-Item -Recurse -Force dev/local_db/*
   ```
3. Restart the backend server. It will automatically re-populate from `dev/data/`.

---

## 5. Common Troubleshooting & FAQ

| Problem | Cause | Solution |
| :--- | :--- | :--- |
| **Network Error / 502 Bad Gateway on API calls** | Backend server is not running on port 8080 | Start the backend via Go or Docker before opening the frontend. |
| **Port 3000 or 8080 already in use** | Another process is holding the port | Run `Get-Process -Id (Get-NetTCPConnection -LocalPort 3000).OwningProcess | Stop-Process` or specify a different port: `PORT=3001 npm run start`. |
| **TypeScript / Turborepo build cache errors** | Stale artifacts after switching git branches | In `ui/`, run `npm run clean`, `npm run clear-turbo-cache`, and `npm install`. |
| **`turbo run start` exits with code 1 (`@perses-dev/app#start exited (1)`)** | Turborepo unquoted space bug with `C:\Program Files\nodejs\node_modules\npm\bin\npm-cli.js` | Run via npm workspace directly: `npm run start -w app` (or `cd app && npm run start`). |
| **Node.js engine warning / syntax errors** | Using older Node.js version (< 24) | Upgrade Node.js to `24.x` as detailed in [environment-setup.md](./environment-setup.md). |
