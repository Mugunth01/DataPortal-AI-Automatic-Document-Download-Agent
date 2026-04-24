# 🏛️ DataPortal AI — Automatic Document Download Agent

> **Hackathon Project #5** · Automatic Document Download Agent  
> An end-to-end AI-powered platform that automatically downloads county property tax rolls, syncs data into SQL Server, and provides a conversational AI interface for querying and analysis.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [Environment Variables](#environment-variables)
- [How It Works](#how-it-works)
- [API Endpoints](#api-endpoints)
- [County Support](#county-support)
- [Configuration](#configuration)
- [Running with ngrok](#running-with-ngrok)

---

## 🧠 Overview

DataPortal AI is an intelligent automation platform built for county property tax data management. It:

- **Automatically downloads** property roll CSVs/XLSXs from 13 county portals (via HTTP + Playwright)
- **Syncs data** into SQL Server by calling stored procedures (`USP_Update_DateCoded_Fast`)
- **Provides an AI chat interface** (powered by a local Ollama LLM) to query the database in plain English
- **Tracks every run** with sidecar JSON logs visible in a live dashboard

---

## ✨ Features

| Feature | Description |
|---|---|
| 🤖 AI Chat | Natural language → SQL → Results via local Ollama LLM |
| 📥 Auto Download | Scheduled HTTP + Playwright downloads from 13 county portals |
| 📄 Manual Upload | Drag-and-drop CSV/XLSX → instant DB update |
| 📊 Automation Dashboard | Live download logs + DB update logs with IST timestamps |
| 🔍 PID Explorer | Search, filter, and export matched/unmatched property IDs |
| 🌙 Dark / Light Mode | Full theme toggle |
| 💬 Chat History | Persistent saved chats (up to 5), load/delete anytime |

---

## 📁 Project Structure

```
dataportal/
│
├── frontend/                        # React application (Create React App)
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── App.jsx                  # Root component — layout, sidebar, routing, chat
│   │   ├── AutomationPage.jsx       # Download logs + DB update logs dashboard
│   │   ├── DocumentsPage.jsx        # Manual CSV/XLSX upload + status tracking
│   │   ├── config.js                # ⚙️  API base URL + ngrok headers (edit this!)
│   │   ├── App.css
│   │   ├── index.css
│   │   ├── index.js                 # React entry point
│   │   └── reportWebVitals.js
│   ├── package.json
│   └── .gitignore
│
└── backend/                         # Python FastAPI application
    ├── main.py                      # FastAPI app — all routes, DB logic, Ollama integration
    ├── autodownload.py              # Scheduler — HTTP + Playwright county downloads
    ├── chat_history.py              # Chat persistence (chats.json read/write helpers)
    ├── documents_routes.py          # (Reference) Document upload route — merged into main.py
    ├── chats.json                   # Persisted chat sessions (auto-generated)
    ├── .env                         # 🔒 Credentials for Playwright counties (never commit!)
    ├── autodownload.log             # Runtime log file (auto-generated)
    ├── cad_scheduler.log            # Scheduler log (auto-generated)
    ├── requirements.txt             # Python dependencies
    └── venv/                        # Virtual environment (not committed)
```

---

## 🛠️ Tech Stack

### Frontend
- **React 18** (Create React App)
- **Recharts** — Area, Bar, Pie charts
- **Lucide React** — Icon set
- Pure inline CSS (no external CSS framework)

### Backend
- **FastAPI** — REST API framework
- **Uvicorn** — ASGI server
- **pyodbc** — SQL Server connection
- **Playwright** — Browser automation for protected county portals
- **Requests** — HTTP downloads with proxy support
- **Ollama** (`gemma4:e2b`) — Local LLM for natural language → SQL
- **openpyxl** — XLSX parsing

### Infrastructure
- **SQL Server** (`TaxrollStaging`) — Property tax database
- **Stored Procedure** `USP_Update_DateCoded_Fast` — Core DB sync logic
- **ngrok** — Tunnel for local dev / demo

---

## ✅ Prerequisites

### Backend
- Python 3.9+
- SQL Server with ODBC Driver 13
- [Ollama](https://ollama.ai) running locally with `gemma4:e2b` pulled
- Playwright browsers installed
- Network access to `\\sanserver\PFBA Fileserver\...` shared drive

### Frontend
- Node.js 18+
- npm or yarn

---

## 🚀 Getting Started

### Backend Setup

```bash
# 1. Clone the repository
git clone https://github.com/your-org/dataportal-ai.git
cd dataportal-ai/backend

# 2. Create and activate virtual environment
python -m venv venv

# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Install Playwright browsers
playwright install chromium

# 5. Configure environment variables
cp .env.example .env
# Edit .env with your county portal credentials (see Environment Variables section)

# 6. Start the FastAPI server
uvicorn main:app --reload --port 8000

# 7. (Optional) Run the auto-downloader in a separate terminal
python autodownload.py
```

The backend will be available at: `http://localhost:8000`  
Interactive API docs: `http://localhost:8000/docs`

---

### Frontend Setup

```bash
# 1. Navigate to frontend directory
cd dataportal-ai/frontend

# 2. Install dependencies
npm install

# 3. Configure the API URL
# Edit src/config.js and set the API constant to your backend URL:
#   export const API = "http://localhost:8000";
#   (or your ngrok URL if tunneling)

# 4. Start the development server
npm start
```

The frontend will be available at: `http://localhost:3000`

---

## 🔐 Environment Variables

Create a `.env` file in the `backend/` directory:

```env
# Shared login email for Playwright counties
EMAIL=your-login@example.com

# Ellis County
ELLIS_URL=https://...
ELLIS_PASSWORD=your-password

# Hidalgo County
HIDALGO_URL=https://...
HIDALGO_PASSWORD=your-password

# Potter-Randall County
POTTER_RANDALL_URL=https://...
POTTER_RANDALL_PASSWORD=your-password

# Rockwall County
ROCKWALL_URL=https://...
ROCKWALL_PASSWORD=your-password

# Webb County
WEBB_URL=https://...
WEBB_PASSWORD=your-password

# Denton County
DENTON_URL=https://...
DENTON_PASSWORD=your-password

# MCAD
MCAD_URL=https://...
MCAD_PASSWORD=your-password

# Travis County
TRAVIS_URL=https://...
TRAVIS_PASSWORD=your-password
```

> ⚠️ **Never commit `.env` to version control.** It is already in `.gitignore`.

---

## ⚙️ Frontend Config

Edit `frontend/src/config.js` to point to your backend:

```js
// For local development
export const API = "http://localhost:8000";

// For ngrok tunnel (update when URL changes)
export const API = "https://xxxx-xxx-xxx.ngrok-free.app";

export const NGROK_HEADERS = {
  "ngrok-skip-browser-warning": "true",
};
```

> This is the **only file** you need to edit when the ngrok URL changes.

---

## 🔄 How It Works

### Automatic Download Flow

```
autodownload.py runs every 4 minutes
        │
        ├── For each HTTP county (Bell, Brewster, Brooks, Brown, Starr)
        │       └── requests + proxy → CSV download
        │
        └── For each Playwright county (Ellis, Hidalgo, Potter-Randall, ...)
                └── Chromium browser → login → Mass Download → XLSX
        │
        ▼
  File saved to \\sanserver\...\{county_code}\{filename}
        │
        ▼
  DB Update (update_database)
    ├── DELETE FROM PTAX_AofA_Staging WHERE CountyName = ?
    ├── INSERT INTO PTAX_AofA_Staging (bulk)
    └── EXEC USP_Update_DateCoded_Fast @CountyName = ?
        │
        ▼
  Sidecar JSON written ({filename}.log.json)
  → Read by /automation/download-logs and /automation/db-logs
```

### Manual Upload Flow (Documents page)

```
User drops CSV/XLSX
        │
        ▼
POST /documents/upload
        ├── Extract property IDs from file
        ├── Run same DB update (staging → SP)
        └── Return stats (matched / unmatched / updated / expired / inserted)
        │
        ▼
Frontend shows live result cards + PID Explorer modal
```

### AI Chat Flow

```
User types question
        │
        ▼
POST /ask
        ├── generate_sql()  →  Ollama LLM (gemma4:e2b)
        ├── fix_sql()       →  safety check + TOP limit
        ├── execute_sql()   →  pyodbc → SQL Server
        └── format_response() → table / list / single value
        │
        ▼
Chat message rendered with structured blocks
Chat saved to chats.json
```

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/ask` | Natural language query → SQL → results |
| `GET` | `/chats` | List all saved chats |
| `POST` | `/chats` | Create new chat |
| `GET` | `/chats/{id}` | Load a specific chat |
| `DELETE` | `/chats/{id}` | Delete a chat |
| `PATCH` | `/chats/{id}/rename` | Rename a chat |
| `POST` | `/documents/upload` | Upload CSV/XLSX for DB update |
| `GET` | `/documents/status/{id}` | Poll upload status |
| `GET` | `/automation/download-logs` | Download run history (with date filter) |
| `GET` | `/automation/db-logs` | DB update history (today's latest per county) |
| `GET` | `/automation/download-file` | Stream a raw downloaded file |
| `POST` | `/automation/trigger` | Manually trigger a county download |
| `GET` | `/automation/debug-files` | Debug file listing with IST timestamps |

---

## 🗺️ County Support

| County | Strategy | Source |
|---|---|---|
| Bell | HTTP + Proxy | esearch.bellcad.org |
| Brewster | HTTP + Proxy | esearch.brewstercotad.org |
| Brooks | HTTP + Proxy | esearch.brookscad.org |
| Brown | HTTP + Proxy | esearch.brown-cad.org |
| Starr | HTTP + Proxy | esearch.starrcad.org |
| Ellis | Playwright | Login required |
| Hidalgo | Playwright | Login required |
| Potter-Randall | Playwright | Login required |
| Rockwall | Playwright | Login required |
| Webb | Playwright | Login required |
| Denton | Playwright | Login required |
| MCAD | Playwright | Login required |
| Travis | Playwright | Login required |

---

## 🌐 Running with ngrok

To expose both servers for remote access or demos:

```bash
# Terminal 1 — Backend
uvicorn main:app --reload --port 8000

# Terminal 2 — Frontend
npm start

# Terminal 3 — ngrok for backend
ngrok http 8000

# Terminal 4 — ngrok for frontend
ngrok http 3000
```

After getting the ngrok URLs:

1. Update `frontend/src/config.js` → set `API` to the **backend** ngrok URL
2. Update `backend/main.py` → add the **frontend** ngrok URL to `FRONTEND_ORIGINS`
3. Restart both servers

---

## 📝 Notes

- All timestamps are in **IST (Asia/Kolkata, UTC+5:30)**
- The AI chat only allows `SELECT` queries — `DROP`, `DELETE`, `UPDATE`, `INSERT` are blocked
- Sidecar JSON files (`.log.json`) are the source of truth for the Automation dashboard
- Chat history is stored locally in `chats.json` — limited to 5 recent sessions in the sidebar
- The auto-downloader runs every **4 minutes** (`INTERVAL_MINUTES * 240` seconds)

---

## 🤝 Team

Built for the internal hackathon by **Team DataPortal**.

---

*Made with ❤️ using FastAPI + React + Ollama*
