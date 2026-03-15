# NexSettle - AI-Powered Insurance Claims Automation Platform

> **Tech Stack:** Django · LangGraph · CrewAI · Gemini 2.5 Flash · Tesseract OCR · MongoDB · Vanilla HTML/CSS/JS

![Django](https://img.shields.io/badge/Django-5.1-092E20?logo=django&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Only-47A248?logo=mongodb&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-Orchestration-1F6FEB)
![CrewAI](https://img.shields.io/badge/CrewAI-Agentic_AI-6A5ACD)
![Gemini](https://img.shields.io/badge/Gemini-2.5_Flash-4285F4?logo=google&logoColor=white)
![OCR](https://img.shields.io/badge/OCR-Tesseract-5C3EE8)
![Frontend](https://img.shields.io/badge/Frontend-HTML%2FCSS%2FJS-E34F26?logo=html5&logoColor=white)
![CI](https://img.shields.io/badge/CI-GitHub_Actions-2088FF?logo=githubactions&logoColor=white)
![Deploy](https://img.shields.io/badge/Deploy-Render-46E3B7?logo=render&logoColor=000000)
![Docker](https://img.shields.io/badge/Docker-Supported-2496ED?logo=docker&logoColor=white)

---
## Table of Contents

1. [Project Overview](#-project-overview)
2. [Architecture](#-architecture)
3. [Prerequisites](#-prerequisites)
4. [Quick Start (Windows)](#-quick-start-windows)
5. [Manual Setup](#-manual-setup)
6. [Docker Setup](#-docker-setup)
7. [Environment Variables](#-environment-variables)
8. [API Reference](#-api-reference)
9. [Frontend Pages](#-frontend-pages)
10. [MongoDB Collections](#-mongodb-collections)
11. [AI Pipeline](#-ai-pipeline)
12. [Credentials (Default)](#-credentials-default)
13. [Project Structure](#-project-structure)

---

## Project Overview

NexSettle automates the entire insurance claim lifecycle:

| Step | Technology | Description |
|------|-----------|-------------|
| Document Upload | Django REST | PDF/PNG/JPG/JPEG multipart upload |
| OCR | Tesseract OCR + pdf2image | Text extraction from scanned docs |
| Classification | Keyword Detection | Identifies 7+ document types |
| Data Extraction | Gemini 2.5 Flash | Structured JSON from document text |
| Fraud Detection | Rule-based + AI | Cross-doc validation, format checks |
| Policy Verification | MongoDB lookup | Aadhaar/PAN cross-check |
| Claim Estimation | Business rules | Natural/Accidental death payout |
| PDF Report | ReportLab | Auto-generated settlement report |
| Storage | MongoDB | All data in `nexsettle_db` |

---

## Architecture

```

```

---

## Prerequisites

| Requirement | Version | Download |
|------------|---------|----------|
| Python | 3.12+ | https://python.org |
| MongoDB | 7.0+ | https://mongodb.com/try/download/community |
| Tesseract OCR | 5.x | https://github.com/UB-Mannheim/tesseract/wiki *(Windows)* |
| Poppler | latest | https://github.com/oschwartz10612/poppler-windows *(Windows)* |
| Git | any | https://git-scm.com |

> **Windows Tesseract:** After installing, Tesseract is typically at:
> `C:\Program Files\Tesseract-OCR\tesseract.exe`
> This path is already pre-configured in `backend\.env`.

> **Windows Poppler:** After downloading, add the `bin/` folder to your system PATH so `pdf2image` can find `pdftoppm`.

---

## Quick Start (Windows)

```batch
# 1. Clone the repo
git clone <repo-url>
cd NexSettle_Project

# 2. Start MongoDB (if not running as a service)
mongod --dbpath C:\data\db

# 3. Run the backend (auto-creates venv + installs deps)
run.bat

# 4. In a new terminal, seed the default admin account
seed_admin.bat

# 5. Open the app
# Frontend and API are both served from Django
# Open http://localhost:8000
```

> **Set your `GEMINI_API_KEY`** in `backend\.env` before running â€” otherwise AI extraction will be skipped and regex fallback is used.

---

## Manual Setup

### 1. Create virtual environment

```powershell
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure environment

```powershell
copy .env.example .env
# Edit .env and fill in GEMINI_API_KEY, email credentials
```

### 3. Create media directories

```powershell
mkdir media\claims
mkdir media\reports
```

### 4. Seed admin account

```powershell
python scripts\seed_admin.py
```

### 5. Start Django

```powershell
python manage.py runserver
```

Backend + frontend are available at **http://localhost:8000**

### 6. Open app

Open `http://localhost:8000` directly in your browser.

> **CORS Note:** Django is configured with `CORS_ALLOW_ALL_ORIGINS = True` for local development.

---

## Docker Setup

```bash
# Build and start all services
docker-compose up --build

# Seed admin (in a separate terminal)
docker exec nexsettle_backend python scripts/seed_admin.py

# Stop
docker-compose down
```

Services:
- `nexsettle_backend` â†’ Django at http://localhost:8000
- `nexsettle_mongo` â†’ MongoDB at localhost:27017

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SECRET_KEY` | (dev key) | Django secret key |
| `DEBUG` | `True` | Django debug mode |
| `MONGO_URI` | `mongodb://localhost:27017/` | MongoDB connection string |
| `MONGO_DB_NAME` | `nexsettle_db` | Database name |
| `JWT_SECRET` | (dev key) | JWT signing secret |
| `GEMINI_API_KEY` | `""` | Google Gemini API key *(required for AI extraction)* |
| `EMAIL_HOST` | `smtp.gmail.com` | SMTP server |
| `EMAIL_HOST_USER` | `""` | Gmail address for OTP emails |
| `EMAIL_HOST_PASSWORD` | `""` | Gmail App Password |
| `TESSERACT_CMD` | `C:\Program Files\Tesseract-OCR\tesseract.exe` | Tesseract path (Windows) |

> Get a Gemini API key at: https://aistudio.google.com/app/apikey

---

## API Reference

### Authentication  `POST /api/auth/`

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/auth/register/` | POST | âŒ | Register new claimant |
| `/api/auth/verify-otp/` | POST | âŒ | Verify email OTP |
| `/api/auth/resend-otp/` | POST | âŒ | Resend OTP |
| `/api/auth/login/` | POST | âŒ | Login (returns JWT) |
| `/api/auth/logout/` | POST | âœ… | Logout |
| `/api/auth/profile/` | GET | âœ… | Get profile |

### AI Pipeline  `POST /api/pipeline/`

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/pipeline/process/` | POST | âœ… User | Upload files & run full AI pipeline |

**Request:** `multipart/form-data` with one or more `files` fields  
**Response:** Full extraction JSON with documents, fraud flag, estimation

### Claims  `GET /api/claims/`

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/claims/list/` | GET | âœ… User | List own claims |
| `/api/claims/all/` | GET | âœ… Admin | All claims (paginated) |
| `/api/claims/<id>/` | GET | âœ… User | Get single claim |
| `/api/claims/<id>/status/` | PATCH | âœ… Agent/Admin | Update claim status |

### Reports  `GET /api/reports/`

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/reports/<id>/download/` | GET | âœ… | Download PDF report |

### Agent  `POST /api/agent/`

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/agent/login/` | POST | âŒ | Agent login |
| `/api/agent/claims/` | GET | âœ… Agent | Pending claims queue |
| `/api/agent/claims/<id>/review/` | POST | âœ… Agent | Submit review |

### Admin  `POST /api/admin-panel/`

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/admin-panel/login/` | POST | âŒ | Admin login |
| `/api/admin-panel/dashboard/` | GET | âœ… Admin | Platform stats |
| `/api/admin-panel/claims/` | GET | âœ… Admin | All claims with filters |
| `/api/admin-panel/claims/<id>/approve/` | PATCH | âœ… Admin | Approve claim |
| `/api/admin-panel/claims/<id>/reject/` | PATCH | âœ… Admin | Reject claim |
| `/api/admin-panel/agents/create/` | POST | âœ… Admin | Create agent account |

---

## Frontend Pages

| File | URL | Role |
|------|-----|------|
| `frontend/index.html` | Landing page | Public |
| `frontend/pages/login.html` | Claimant login | Public |
| `frontend/pages/register.html` | Registration + OTP | Public |
| `frontend/pages/verify-otp.html` | Email verification | Public |
| `frontend/pages/dashboard.html` | Claimant dashboard | User |
| `frontend/pages/agent-login.html` | Agent login | Public |
| `frontend/pages/agent-dashboard.html` | Agent claims queue | Agent |
| `frontend/pages/admin-login.html` | Admin login | Public |
| `frontend/pages/admin-dashboard.html` | Full admin panel | Admin |

---

## MongoDB Collections

Database: **`nexsettle_db`**

| Collection | Purpose |
|-----------|---------|
| `users` | Claimant accounts |
| `otp_verification` | OTP codes for email verification |
| `policy_holder_data` | Pre-existing insurance details |
| `nominee_details` | Nominee information |
| `claims` | Processed claims with AI results |
| `claim_documents` | File paths for uploaded documents |
| `agents` | Agent accounts |
| `admins` | Admin accounts |
| `fraud_logs` | Fraud detection audit trail |

---

## AI Pipeline

```

```

### Supported Document Types

| Type | Key Fields Extracted |
|------|---------------------|
| `death_certificate` | full_name, date_of_death, cause_of_death, certificate_number, issuing_authority |
| `aadhaar` | aadhaar_number (masked to ****1234) |
| `pan` | pan_number (masked to *****4F) |
| `bank` | account_number, ifsc_code, bank_name, account_holder_name |
| `policy` | policy_number, sum_assured, nominee_name, dates |
| `fir` | fir_number, police_station, incident_description |
| `hospital_record` | diagnosis, treatment_summary, cause_of_death |
| `newspaper_clipping` | headline, publication_date, incident_description |

---

## Credentials (Default)

> Created by running `seed_admin.bat` or `python scripts/seed_admin.py`

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@nexsettle.org | Admin@123 |

**Agents** are created by the admin through the Admin Portal â†’ Agent Management.

**Claimants** self-register at `frontend/pages/register.html`.

---

## Project Structure

```

```

---

## Development Notes

### CORS & CSRF
- `CORS_ALLOW_ALL_ORIGINS = True` is set for local dev.
- JWT in `Authorization: Bearer <token>` header handles auth â€” no CSRF required.

### Fallback Extraction
- If Gemini API key is missing or the call fails, **regex-based extraction** runs automatically as a fallback.

### OCR Confidence Threshold
- If Tesseract confidence < 0.6 â†’ `status: "failed_ocr"` + user prompt to re-upload.
- PDFs with embedded text bypass Tesseract (confidence = 0.95).

### Claim Estimation Rules
- **Natural Death:** 100% of Sum Assured
- **Accidental Death:** 200% (double indemnity)
- **Fraud Detected:** â‚¹0 payout

### Data Masking (MongoDB storage)
- Aadhaar: `123412341234` â†’ `********1234`
- PAN: `ABCDE1234F` â†’ `*****1234F`
- Bank Account: `123456789012` â†’ `XXXXXXXX9012`

---

*Built with â¤ï¸ using Django Â· LangGraph Â· Gemini 2.5 Flash Â· MongoDB*

---

## CI/CD (GitHub Actions)

This repo now includes:

- `.github/workflows/ci.yml`
  - Runs on push/PR
  - Installs dependencies
  - Compiles Python modules
  - Runs Django checks
  - Runs Mongo-backed smoke flow (`smoke_test_flow`)

- `.github/workflows/deploy-render.yml`
  - Triggers Render deployment after successful CI on `main/master`
  - Also supports manual trigger (`workflow_dispatch`)
  - Requires GitHub secret: `RENDER_DEPLOY_HOOK_URL`

### Required GitHub Secret

Add this in GitHub repo settings:

- `RENDER_DEPLOY_HOOK_URL` = your Render deploy hook URL

---

## Render Deployment

Deployment files included:

- `render.yaml` (Render Blueprint)
- `backend/start_render.sh` (startup script)
- `backend/Procfile`
- `backend/runtime.txt`
- `backend/.env.example`

### Deploy Steps

1. Push this repository to GitHub.
2. In Render, create a new **Blueprint** from the repo.
3. Render will detect `render.yaml` and create the web service.
4. Set required env vars in Render dashboard:
   - `MONGO_URI` (MongoDB Atlas/local tunnel URI)
   - `GEMINI_API_KEY`
   - `EMAIL_HOST_USER` and `EMAIL_HOST_PASSWORD` (if email OTP is needed)
5. Deploy.

Health check endpoint:

- `/api/health/`

App endpoint:

- `/` (frontend served by Django)

---

## One-command Bootstrap

For local/project initialization:

```powershell
cd backend
python manage.py bootstrap_project
python manage.py runserver
```

---
