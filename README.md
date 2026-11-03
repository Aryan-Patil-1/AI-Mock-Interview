# InterviewForge — AI-Powered Mock Interview Platform with RAG

![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![React](https://img.shields.io/badge/React-19.2-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript-6.0-3178C6?style=for-the-badge&logo=typescript&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-4.2-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-336791?style=for-the-badge&logo=postgresql&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=for-the-badge&logo=openai&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=for-the-badge&logo=langchain&logoColor=white)
![LangGraph](https://img.shields.io/badge/LangGraph-FF6F00?style=for-the-badge&logoColor=white)
![RAG](https://img.shields.io/badge/RAG-Retrieval--Augmented_Generation-7C3AED?style=for-the-badge)
![ChromaDB](https://img.shields.io/badge/ChromaDB-FF6B35?style=for-the-badge&logoColor=white)

A full-stack AI-powered mock interview platform built on **Retrieval-Augmented Generation (RAG)**. Upload a resume and a job description; the platform indexes your resume into ChromaDB, retrieves the most relevant context at interview time, and uses **GPT-4o + LangGraph** to generate questions grounded in your real experience. Practise hands-free in **voice mode** and track weak areas across sessions with **Mem0**.

![InterviewForge Screenshot](docs/images/screenshot.png)

---

## Features

- **RAG Pipeline** — resume PDFs are chunked and indexed into ChromaDB; retrieved at interview start to ground question generation in your actual experience
- **AI Question Generation** — GPT-4o + LangGraph generate behavioural, coding, and written questions personalised to the role, resume, and known weak areas
- **Voice Interview Mode** — fully hands-free using browser-native SpeechRecognition (STT) and SpeechSynthesis (TTS)
- **AI Answer Evaluation** — GPT-4o scores each answer 0–10 with specific, actionable feedback
- **Cross-Session Memory** — Mem0 stores strengths and weak areas across sessions and injects them so the AI focuses on where you need work
- **Async Evaluation Worker** — Redis + RQ runs answer scoring in the background
- **Interview Reports** — total score, strengths, weaknesses, per-question breakdown
- **Session Resume** — pick up an incomplete interview from the dashboard
- **JWT Authentication** — register, login, persistent sessions
- **Responsive UI** — Tailwind CSS 4.2 + shadcn/ui, works on desktop and mobile

---

## AI & RAG Pipeline

The core of InterviewForge is a Retrieval-Augmented Generation pipeline that grounds interview questions in your actual experience.

```
Resume PDF ──► Chunker ──► OpenAI embeddings ──► ChromaDB
                                                    │
                                                    ▼
User starts interview ──► Retriever ──► Top-k resume chunks
                                                    │
                                                    ▼
                              LangGraph state machine
                                                    │
                                ┌───────────────────┼───────────────────┐
                                ▼                   ▼                   ▼
                        Generate question     Score answer       Update memory (Mem0)
                          (GPT-4o)            (GPT-4o, 0–10)     across sessions
```

- **Indexing** — resume PDFs are chunked, embedded with OpenAI, and stored in ChromaDB at upload time
- **Retrieval** — at the start of every interview, top-k resume chunks plus role + Mem0 weak-area context are pulled in to ground question generation
- **Orchestration** — a LangGraph state machine drives the loop: generate → ask → evaluate → record → next, with answer scoring offloaded to a Redis + RQ background worker
- **Memory** — Mem0 persists strengths and weaknesses across sessions so the AI keeps focusing on areas that need work

---

## Tech Stack

| Layer | Technology |
|---|---|
| **AI / LLM** | **OpenAI GPT-4o, LangChain, LangGraph** |
| **RAG / Vector DB** | **ChromaDB, OpenAI embeddings** |
| **Long-term Memory** | **Mem0** |
| Frontend | React 19.2, TypeScript 6.0, Vite 8.0, Tailwind CSS 4.2 |
| UI | shadcn/ui, Base UI, Lucide React |
| State | Zustand 5.0, TanStack Query 5.100 |
| Routing | React Router 7.14 |
| HTTP Client | Axios 1.15 |
| Backend | Python 3.12, FastAPI, SQLAlchemy (async) |
| Database | PostgreSQL 17 (asyncpg), Alembic |
| Cache / Queue | Redis 7, RQ |
| Auth | JWT (python-jose), bcrypt |
| Voice | Web Speech API (browser-native STT + TTS) |
| Logging | Loguru |

---

## Project Structure

```
├── backend/                       # FastAPI server
│   ├── app/
│   │   ├── api/
│   │   │   ├── controllers/       # auth, interview, sessions, settings
│   │   │   └── routes/            # Thin route handlers (auth, interview, sessions, settings)
│   │   ├── core/                  # JWT auth, Pydantic config, async DB session
│   │   ├── models/                # SQLAlchemy ORM, Pydantic schemas, session state, user helpers
│   │   ├── services/              # RAG, evaluator, LangGraph flow, Mem0 memory, session manager
│   │   ├── workers/               # RQ async evaluation worker
│   │   └── main.py                # FastAPI app + router wiring
│   ├── seed.py                    # Demo data seeder
│   ├── Dockerfile
│   ├── requirements.txt
│   └── .env.example
│
├── frontend/                      # React + TypeScript SPA
│   ├── src/
│   │   ├── api/                   # Axios client (JWT interceptor), mutations, request helpers
│   │   ├── components/            # Layout, ProtectedRoute, shadcn/ui primitives
│   │   ├── lib/                   # ROUTES constants, utilities
│   │   ├── pages/                 # auth, dashboard, home, interview, landing, not-found, report, settings
│   │   │                          # (each page has its own components/ subfolder + Zustand store where needed)
│   │   ├── shared/                # hooks (useSpeechRecognition, useSpeechSynthesis), stores (useAuthStore)
│   │   ├── types/                 # auth, common, interview, session, user
│   │   ├── App.tsx
│   │   ├── index.css
│   │   └── main.tsx
│   ├── components.json            # shadcn/ui config
│   ├── eslint.config.js
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json              # + tsconfig.app.json, tsconfig.node.json
│   ├── vite.config.ts
│   └── .env.example
│
├── docs/images/                   # Screenshots
├── docker-compose.yml             # PostgreSQL + Redis + ChromaDB + backend + worker
├── .env.example
├── .gitignore
└── README.md
```

---

## Getting Started

### Prerequisites

- [Node.js 20+ (tested on 24.13.0)](https://nodejs.org)
- [Python 3.12+](https://www.python.org/downloads/)
- [Docker](https://www.docker.com) — for PostgreSQL, Redis, and ChromaDB
- [OpenAI API key](https://platform.openai.com)

### 1. Clone the repository

```bash
git clone https://github.com/saadshahidit/interview-forge.git
cd interview-forge
```

---

### 2. Set up environment variables

**Root `.env`:**

```env
OPENAI_API_KEY=your_openai_api_key_here
DATABASE_URL=postgresql+asyncpg://interviewforge:interviewforge@localhost:5432/interviewforge
REDIS_URL=redis://localhost:6379
CHROMA_HOST=localhost
CHROMA_PORT=8100
VITE_API_URL=http://localhost:8000
```

**`backend/.env`:**

```env
OPENAI_API_KEY=your_openai_api_key_here
DATABASE_URL=postgresql+asyncpg://interviewforge:interviewforge@localhost:5432/interviewforge
REDIS_URL=redis://localhost:6379
CHROMA_HOST=localhost
CHROMA_PORT=8100
JWT_SECRET=your_long_random_secret_here
```

**`frontend/.env`:**

```env
VITE_API_URL=http://localhost:8000
```

### 3. Start services

**Option A — Docker (recommended):**

```bash
docker compose up -d
```

This starts PostgreSQL, Redis, ChromaDB, the backend API, and the RQ evaluation worker.

**Option B — Local services:** install PostgreSQL 17, Redis 7, and ChromaDB locally (or use cloud equivalents) and update the URLs in `.env` accordingly.

### 4. Install dependencies and start

```bash
# Backend
cd backend
python -m venv .venv
source .venv/bin/activate         # macOS/Linux
# .venv\Scripts\activate          # Windows
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

# Frontend (new terminal)
cd frontend
npm install
npm run dev
```

### 5. Seed demo data (optional)

```bash
cd backend && python seed.py
```

> Demo login — `sarah@interviewforge.com` / `password123` (5 sessions: 3 completed, 2 in progress)

Open `http://localhost:5173` in your browser.

---

## API Reference

### Auth — `/api/auth`

| Method | Endpoint | Description |
|---|---|---|
| POST | `/signup` | Register a new user |
| POST | `/login` | Login and receive a JWT |
| GET | `/me` | Get current user (auth required) |

### Interview — `/api/interview` (auth required)

| Method | Endpoint | Description |
|---|---|---|
| POST | `/start` | Start session and get first question |
| POST | `/answer` | Submit answer and get next question |
| GET | `/status/:id` | Get current session state |
| GET | `/report/:id` | Get completed session report |

### Sessions — `/api/sessions` (auth required)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/` | List all sessions for the user |
| GET | `/:id` | Get session detail with history |

### Settings — `/api/settings` (auth required)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/profile` | Get user profile |
| POST | `/resume` | Upload resume PDF |

---

## Deployment

| Part | Platform |
|---|---|
| Frontend | [Vercel](https://vercel.com) |
| Backend | [Render](https://render.com) |
| Database | [Supabase](https://supabase.com) (PostgreSQL) |
| Cache | [Upstash](https://upstash.com) (Redis) |
| Vector DB | ChromaDB (self-hosted or cloud) |

When deploying, set the environment variables on each platform:
- On Render: set `OPENAI_API_KEY`, `DATABASE_URL`, `REDIS_URL`, `CHROMA_HOST`, `CHROMA_PORT`, `JWT_SECRET`
- On Vercel: set `VITE_API_URL` to your live Render backend URL (e.g. `https://your-app.onrender.com`)

---

## License

MIT
