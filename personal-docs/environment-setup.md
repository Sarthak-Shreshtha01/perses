# Environment Setup Guide (Windows)

This document details the local setup requirements and how to get your machine ready for Perses UI development.

---

## 1. Node.js & NPM Version Requirements

The UI packages enforce modern Node and NPM engines:
- **Node.js**: `>= 24` (Target version: `24.19.0` as specified in `ui/.nvmrc`)
- **NPM**: `>= 11` (Target version: `11.17.0` as specified in `ui/package.json`)

### Checking Your Current Versions
Run in PowerShell:
```powershell
node --version
npm --version
```

### Upgrading on Windows
If your Node is currently v22 or lower:
1. **Option A: Official Installer**
   - Download the latest Node.js v24 (LTS or Current) installer from [nodejs.org](https://nodejs.org/).
   - Run the installer and restart your terminal.
2. **Option B: Using Winget**
   ```powershell
   winget install OpenJS.NodeJS -v 24.19.0
   ```
3. **Option C: Using nvm-windows**
   - If using [nvm-windows](https://github.com/coreybutler/nvm-windows):
     ```powershell
     nvm install 24.19.0
     nvm use 24.19.0
     ```

---

## 2. Installing UI Dependencies

Perses uses **NPM Workspaces** and **Turborepo** under the `ui/` folder.

```powershell
cd "d:\Open Source\LFX\perses\ui"
npm install
```

> **Troubleshooting Tip:**
> If you encounter dependency or build cache errors when switching branches, run:
> ```powershell
> npm run clean
> npm run clear-turbo-cache
> npm run reinstall
> ```

---

## 3. Running the Backend API Server

The UI app in `ui/app` expects the Perses API server running on `http://localhost:8080`.

### Option A: Using Docker (Recommended if Go is not installed)
A development container image can be built and run using Docker:
```powershell
# In project root:
docker build -t perses-dev -f Dockerfile.dev .
docker run -d -p 8080:8080 --name perses-backend perses-dev
```
Default credentials:
- **Username:** `admin`
- **Password:** `password`

### Option B: Local Go Build (If Go >= 1.26 installed)
```powershell
# From project root:
go build -o ./bin/perses.exe ./cmd/perses
./bin/perses.exe -config ./dev/config.yaml
```

---

## 4. Starting the Frontend UI

Once dependencies are installed, start the local Rspack dev server:

```powershell
cd "d:\Open Source\LFX\perses\ui"
npm run start
```

Or target the app package specifically:
```powershell
npm run start -w app
```

Open your browser at:
**`http://localhost:3000/`**
