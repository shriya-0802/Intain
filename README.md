# Intain-Fintech: Loan Data Verification Copilot

An enterprise-grade, event-sourced verification platform that ingests messy loan records, detects data-quality issues, uses an AI copilot to resolve exceptions, and produces a cryptographically signed, verified dataset.

Built for the **Intain Campus FinTech Challenge 2026 | Full Stack Track**.

## Features

- **Event-Sourced Architecture** — every ingestion, edit, and AI suggestion is appended as an immutable event to a hash-chained `loan_events` ledger.
- **Role-Based Workflows** — separate dashboards for Data Operator, Reviewer, and Data Consumer.
- **Maker/Checker Enforcement** — a single-actor lock prevents the same user from both submitting and approving a change.
- **Hardened Ingestion** — streaming CSV parser with idempotency and sequence guards.
- **AI Copilot** — explains validation exceptions, proposes JSON patches, and can synthesize new self-healing validation rules. Never mutates data directly; a human reviewer must accept every suggestion.
- **Cryptographic Audit Trail** — SHA-256 hash-chained events, verifiable via a "Verify Ledger Integrity" check.

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, TypeScript, Tailwind CSS, React Router, Axios |
| Backend | FastAPI, SQLAlchemy 2.0, Pydantic v2, Alembic |
| Database | SQLite (default, file-based — swappable via `DATABASE_URL`) |
| AI providers | Google Gemini, Anthropic Claude, OpenAI (any one is enough) |
| Auth | JWT (HS256) |

## Prerequisites

Install these before you start:

- **Node.js** v18 or later (v20+ recommended) — [nodejs.org](https://nodejs.org)
- **Python** 3.11 or later
- **pip** (comes with Python)
- **Git**

Check versions:
```bash
node -v
python3 --version
pip --version
```

## 1. Clone the repository

```bash
git clone https://github.com/saanidhyagoyal/Intain-Fintech.git
cd Intain-Fintech
```

## 2. Backend setup

```bash
cd backend
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Backend dependencies (`backend/requirements.txt`)

| Package | Version | Purpose |
|---|---|---|
| fastapi | 0.141.1 | Web framework / API |
| uvicorn | 0.52.4 | ASGI server |
| starlette | 1.6.0 | ASGI toolkit (FastAPI dependency) |
| SQLAlchemy | 2.0.52 | ORM / database layer |
| alembic | 1.19.1 | DB schema migrations |
| pydantic | 2.13.4 | Data validation / schemas |
| pydantic_core | 2.46.4 | Pydantic core |
| python-dotenv | 1.2.3 | Loads `.env` config |
| cryptography | 50.0.1 | SHA-256 hash chaining for the audit ledger |
| anthropic | 1.1.0 | Anthropic Claude API client |
| google-genai | 2.20.0 | Google Gemini API client |
| google-auth | 2.57.0 | Google auth for Gemini |
| openai | 3.6.0 | OpenAI API client |
| httpx / httpx2 | 0.28.1 / 2.12.0 | HTTP client (used by AI SDKs) |
| requests | 2.34.2 | HTTP client |
| tenacity | 9.1.4 | Retry logic |
| websockets | 16.1.1 | WebSocket support |
| *(remaining entries)* | — | Transitive dependencies of the above (annotated-types, anyio, certifi, cffi, charset-normalizer, click, distro, docstring_parser, h11, httpcore, idna, jiter, Mako, MarkupSafe, pyasn1, pyasn1_modules, pycparser, sniffio, truststore, typing-inspection, typing_extensions, urllib3) |

Everything in the table is installed automatically by `pip install -r requirements.txt` — you don't need to install these individually.

### Configure environment variables

Create a `.env` file in the **project root** (same level as `README.md`, one level above `backend/`):

```bash
# ── Database ──────────────────────────────────────────────────
DATABASE_URL=sqlite:///./loan_copilot.db
RESET_DB_ON_STARTUP=True

# ── AI API Keys ──────────────────────────────────────────────
# GEMINI_API_KEY=your_GEMINI_API_KEY_api_key_here
# ANTHROPIC_API_KEY=your_anthropic_api_key_here
# CHATGPT_API_KEY=your_CHATGPT_api_key_here

# ── Security ─────────────────────────────────────────────────
SECRET_KEY=dev_secret_key_12345

# ── Network / URLs ───────────────────────────────────────────
FRONTEND_URL=http://localhost:5173
BACKEND_URL=http://localhost:8000
VITE_BACKEND_URL=http://localhost:8000

# ── File Upload ──────────────────────────────────────────────
UPLOAD_DIR=./uploads

# ── Seed User Credentials (Backend) ─────────────────────────
SEED_OPERATOR_USER=operator
SEED_OPERATOR_PASS=operator123
REVIEWER_A_USERNAME=reviewer_a
REVIEWER_A_PASSWORD=demo_password
REVIEWER_B_USERNAME=reviewer_b
REVIEWER_B_PASSWORD=demo_password
SEED_CONSUMER_USER=consumer
SEED_CONSUMER_PASS=consumer123

# ── Frontend Demo Credentials (VITE_ prefix for Vite) ───────
VITE_API_URL=/api
VITE_DEMO_OPERATOR_USER=operator
VITE_DEMO_OPERATOR_PASS=operator123
VITE_DEMO_REVIEWER_A_USER=reviewer_a
VITE_DEMO_REVIEWER_A_PASS=demo_password
VITE_DEMO_REVIEWER_B_USER=reviewer_b
VITE_DEMO_REVIEWER_B_PASS=demo_password
VITE_DEMO_CONSUMER_USER=consumer
VITE_DEMO_CONSUMER_PASS=consumer123

# Swagger API Documentation
SWAGGER_UI_URL="http://localhost:8000/docs"

```

Every value has a sane default baked into `app/core/config.py`, so a minimal `.env` (or none at all) will still boot the app — but you'll want at least one AI key for the copilot to return real (non-mock) responses.

### Run the backend

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

- API base URL: `http://localhost:8000/api`
- Interactive docs (Swagger): `http://localhost:8000/docs`

On first run, the app auto-creates SQLite tables and seeds four default users (one per role) from the `.env` values above.

## 3. Frontend setup

Open a new terminal tab:

```bash
cd frontend
npm install
```

### Frontend dependencies (`frontend/package.json`)

**Runtime:**

| Package | Version |
|---|---|
| react | ^19.2.8 |
| react-dom | ^19.2.8 |
| react-router-dom | ^7.18.2 |
| axios | ^1.20.0 |
| clsx | ^2.1.1 |
| tailwind-merge | ^3.6.0 |
| lucide-react | ^1.34.0 |

**Dev / build / test tooling:**

| Package | Version |
|---|---|
| vite | ^8.2.2 |
| @vitejs/plugin-react | ^6.1.0 |
| typescript | ~6.0.2 |
| typescript-eslint | ^8.67.0 |
| eslint | ^10.9.0 |
| eslint-plugin-react-hooks | ^7.1.1 |
| eslint-plugin-react-refresh | ^0.5.4 |
| @eslint/js | ^10.0.1 |
| tailwindcss | ^3.4.19 |
| @tailwindcss/forms | ^0.5.11 |
| postcss | ^8.5.26 |
| autoprefixer | ^10.5.4 |
| vitest | ^3.2.7 |
| @testing-library/react | ^16.3.3 |
| @testing-library/jest-dom | ^7.0.1 |
| @testing-library/user-event | ^14.6.6 |
| jsdom | ^27.0.1 |
| @types/react, @types/react-dom, @types/node | latest matching |
| globals | ^17.11.0 |

`npm install` pulls all of the above in one shot from `package.json`/`package-lock.json`.

### Run the frontend

```bash
npm run dev
```

- App URL: `http://localhost:5173`

The Vite dev server proxies/points to `BACKEND_URL` (`http://localhost:8000` by default) for all API calls — make sure the backend is running first.

## Default test credentials

Seeded automatically on backend startup (from `.env`, no hardcoding):

| Role | Username | Password |
|---|---|---|
| Data Operator | `operator` | `operator123` |
| Reviewer A | `reviewer_a` | `demo_password` |
| Reviewer B | `reviewer_b` | `demo_password` |
| Data Consumer | `consumer` | `consumer123` |

## Running tests

**Backend** (from `backend/`, with the virtualenv active):
```bash
pip install pytest pytest-asyncio   # if not already present
pytest tests/
```

**End-to-end** (from project root):
```bash
pytest tests/test_e2e_flow.py
```

**Frontend** (from `frontend/`):
```bash
npm run test
```

## Project structure

```
Intain-Fintech/
├── ARCHITECTURE.md          # System design, data model, API, trade-offs
├── AI_DEVELOPMENT_LOG.md    # Agentic-coding / prompt log
├── tests/                   # Cross-stack end-to-end tests
├── backend/
│   ├── main.py              # FastAPI entry point (CORS, lifespan, seeding)
│   ├── requirements.txt
│   └── app/
│       ├── api/v1/          # Route handlers (auth, upload, loans, exceptions, rules, ai, audit, summary, verified)
│       ├── core/            # config, database, security, cryptography
│       ├── models/          # LoanEvent, ExceptionRecord, ValidationRule, User
│       ├── schemas/         # Pydantic request/response schemas
│       └── services/        # ingestion, validation, event_store, ai_assistant, self_healing
└── frontend/
    ├── package.json
    └── src/
        ├── pages/           # OperatorDash, ReviewerDash, ConsumerDash, AuditTrailDash, RulesDictionaryDash, LoanDetail, LoginPage
        ├── components/      # UploadZone, AIPanel, AuditTimeline, ExceptionCard, Table, Sidebar
        └── api/client.ts    # HTTP client
```

## Documentation

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — event-sourcing design, data model, validation engine, AI controls, and API design.
- [`AI_DEVELOPMENT_LOG.md`](./AI_DEVELOPMENT_LOG.md) — evidence of agentic coding and the human-review loop.
