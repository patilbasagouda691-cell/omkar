# Smart Gmail — Full-Stack Email Organization & Retention System

Smart Gmail is a modern, production-style web application designed to connect securely with Google's Gmail API and provide an intelligent email classification, retention management, and safe cleanup workflow.

![Smart Gmail Architecture](https://img.shields.io/badge/Stack-React%20%7C%20Node.js%20%7C%20Express%20%7C%20MongoDB%20%7C%20Gmail%20API-blue)

---

## 🌟 Key Features

1. **Smart Email Classification**
   - Classify incoming emails as **Important**, **Not Important (Other)**, or **Pending Review (Decide Later)**.
   - Built-in heuristic AI recommendation engine providing confidence ratings and reasoning cards.
   - Complete user override control—AI suggestions are recommendations only and never permanently delete messages.

2. **Custom Email Retention Engine**
   - Configurable retention periods: **30 Days**, **60 Days** (Default), **90 Days**, **180 Days**, or **Custom Days**.
   - Expiration dates calculated strictly from the **original received date** of each email.
   - Re-classifying or updating an email does **NOT** reset its received timestamp.

3. **Safe Automated Cleanup & Review Queue**
   - **Expiring Soon Warning**: Highlight emails reaching expiration within 7 days.
   - **Expired Emails Review Queue**: Review expired emails before any deletion occurs.
   - **Idempotent Cron Job**: Background worker (`node-cron`) running daily at 2:00 AM to process retention expirations.
   - **Data Safety Rule**: Original Gmail emails are **NEVER** automatically permanently deleted without explicit user confirmation (`?confirm=true`).

4. **Complete Email Client Functionality**
   - Read emails with full HTML/plain-text rendering & attachment downloads.
   - Compose & Send emails via official Gmail API (or simulated Demo Mode).
   - Star/Unstar, Soft Trash, Restore, and Search by sender, recipient, subject, content, and classification.

5. **Demo Mode (Runs Out-of-the-Box)**
   - Fully functional without requiring active Google OAuth credentials or local MongoDB daemons.
   - Provides realistic sample dataset (Placement cell, Project updates, Amazon recruitment, Bank statements, Security alerts, etc.).

---

## 🛠 Tech Stack

### Frontend
- **Framework**: React 18 with Vite
- **Styling**: Tailwind CSS 3
- **Icons**: Lucide React
- **Routing**: React Router DOM v6
- **HTTP Client**: Axios

### Backend
- **Runtime**: Node.js & Express.js
- **Database**: MongoDB & Mongoose (with in-memory fallback for Demo Mode)
- **Authentication**: Google OAuth 2.0 & JWT (`jsonwebtoken`)
- **APIs**: Official Google Gmail API (`googleapis`, `google-auth-library`)
- **Jobs & Security**: `node-cron`, `helmet`, `cors`, `express-rate-limit`

---

## 📁 Project Structure

```
smart-gmail/
│
├── client/                     # Frontend React application
│   ├── src/
│   │   ├── components/         # Reusable UI components (EmailRow, EmailReader, ComposeModal, etc.)
│   │   ├── pages/              # Route pages (Dashboard, Important, Other, Pending, Expired, Settings, etc.)
│   │   ├── layouts/            # MainLayout wrapper
│   │   ├── context/            # AuthContext & EmailContext
│   │   ├── services/           # Axios API client
│   │   ├── App.jsx             # React Router setup
│   │   ├── main.jsx
│   │   └── index.css           # Tailwind CSS
│   ├── public/
│   ├── package.json
│   ├── vite.config.js
│   └── tailwind.config.js
│
├── server/                     # Backend Node.js Express server
│   ├── config/                 # DB & Google OAuth configurations
│   ├── controllers/            # Route controllers (email, retention, cleanup, dashboard, auth)
│   ├── routes/                 # Express REST API routes
│   ├── models/                 # Mongoose models (User, EmailMetadata, UserSettings, CleanupLog)
│   ├── middleware/             # Auth & JWT verification middleware
│   ├── services/               # Gmail API service, Sync service, Cleanup service, AI engine, Demo store
│   ├── jobs/                   # node-cron scheduled cleanup job
│   ├── server.js               # Main Express entry point
│   ├── package.json
│   └── .env.example
│
├── package.json                # Root package with concurrently scripts
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start & Installation

### Prerequisites
- Node.js (v18+)
- npm (v9+)
- (Optional) MongoDB local or Cloud URI
- (Optional) Google Cloud OAuth Client Credentials

### 1. Install All Dependencies

Run the single root script to install dependencies across root, server, and client:

```bash
npm run install-all
```

### 2. Environment Setup

Create `server/.env` based on `server/.env.example`:

```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/smart-gmail
JWT_SECRET=super_secret_jwt_key_smart_gmail_2026
GOOGLE_CLIENT_ID=your_google_client_id_here.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_google_client_secret_here
GOOGLE_REDIRECT_URI=http://localhost:5000/api/auth/google/callback
CLIENT_URL=http://localhost:5173
DEFAULT_RETENTION_DAYS=60
DEMO_MODE=true
```

### 3. Run Development Servers

Start both Frontend (Vite) and Backend (Express) concurrently:

```bash
npm run dev
```

- **Frontend**: [http://localhost:5173](http://localhost:5173)
- **Backend API**: [http://localhost:5000/api](http://localhost:5000/api)

---

## 🌐 Google Cloud & Gmail API Setup Guide

To connect real Gmail accounts to Smart Gmail:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project (e.g., `Smart Gmail App`).
3. Navigate to **APIs & Services > Library** and enable **Gmail API**.
4. Configure the **OAuth Consent Screen**:
   - User Type: External or Internal.
   - Add scopes: `.../auth/gmail.readonly`, `.../auth/gmail.send`, `.../auth/gmail.modify`, `.../auth/userinfo.email`, `.../auth/userinfo.profile`.
   - Add test users (your Gmail address).
5. Go to **Credentials > Create Credentials > OAuth client ID**:
   - Application type: **Web application**.
   - Authorized redirect URIs: `http://localhost:5000/api/auth/google/callback`.
6. Copy the **Client ID** and **Client Secret** into your `server/.env` file.

---

## 📡 REST API Documentation

| Endpoint | Method | Description |
| :--- | :--- | :--- |
| `/api/auth/google` | `GET` | Initiate Google OAuth authorization flow |
| `/api/auth/google/callback` | `GET` | Google OAuth callback handler |
| `/api/auth/me` | `GET` | Fetch authenticated user details |
| `/api/emails` | `GET` | Fetch inbox emails (supports `classification`, `isStarred`, `search`, `page`) |
| `/api/emails/:id` | `GET` | Fetch single email details (marks email as read) |
| `/api/emails/sync` | `POST` | Synchronize Gmail inbox into database |
| `/api/emails/send` | `POST` | Send email via Gmail API or Demo fallback |
| `/api/emails/:id/classification` | `PATCH` | Update email classification (`IMPORTANT`, `NOT_IMPORTANT`, `PENDING`) |
| `/api/emails/:id/star` | `PATCH` | Toggle email star status |
| `/api/emails/:id/trash` | `POST` | Move email to soft trash |
| `/api/emails/:id/restore` | `POST` | Restore email from trash |
| `/api/emails/:id/permanent` | `DELETE` | Permanently delete email (`?confirm=true`) |
| `/api/settings/retention` | `GET / PATCH` | Read or update retention period (30/60/90/180/custom) |
| `/api/cleanup/expired` | `GET` | Fetch expired emails queue |
| `/api/cleanup/expiring` | `GET` | Fetch emails expiring within 7 days |
| `/api/cleanup/run` | `POST` | Trigger daily cleanup job manually |
| `/api/cleanup/logs` | `GET` | Fetch audit logs |
| `/api/dashboard/stats` | `GET` | Fetch aggregate stats for dashboard cards |

---

## 🧪 Manual Testing Checklist

- [x] **Demo Mode Launch**: Run `npm run dev` and navigate to `http://localhost:5173`. App should load instantly with sample emails.
- [x] **Navigation**: Click between Dashboard, Important Inbox, Other Inbox, Pending Classification, Starred, Sent, Trash, Expired, and Settings.
- [x] **Email Triaging**: Change classification of an email from Pending to Important or Other. Verify counter updates in real time.
- [x] **Compose & Send**: Click "+ Compose", fill in To, Subject, Body, and click Send. Verify email appears in Sent folder.
- [x] **Retention Configuration**: Go to Settings, change retention period from 60 days to 30 days or Custom 90 days. Verify Toast notification confirms recalculation.
- [x] **Expired Queue & Extension**: Go to Expired Emails, click "+60d Retention" to restore an expired email back to active status.
- [x] **Permanent Deletion**: Delete an email from Trash/Expired queue, verify confirmation dialog requires explicit confirmation.
