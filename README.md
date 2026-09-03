# OpsFlow AI

<div align="center">

**AI Middleware for Small Business Operations**

[![Python Version](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-green.svg)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15+-336791.svg)](https://www.postgresql.org/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991.svg)](https://openai.com/)
[![Docker](https://img.shields.io/badge/Docker-24+-2496ED.svg)](https://www.docker.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Stop being the integration between your tools. Let AI handle the glue work.**

</div>

---

## 📌 TL;DR

> *"Right now, you are the integration between your tools."*

Small teams run on a patchwork—tasks in one tool, docs in another, decisions buried in Slack threads—with an operator manually stitching it all together. That stitching quietly eats **10–15 hours a week**.

**OpsFlow AI** sits between your existing tools (Gmail, Calendar, Slack) and an AI orchestration layer. It automates the glue work:

- 🧹 Reads emails → Creates tasks & drafts replies
- 🔔 Detects stale deals → Drafts follow-ups
- 📅 Compiles daily briefs from meetings, tasks, and comms
- 📝 Summarizes meetings → Extracts action items

**No change in how you work. No new apps to learn. Just less busywork.**

---

## 🎯 Why This Matters

| The Old Way | The OpsFlow Way |
| :--- | :--- |
| Start day digging through 60+ emails | Agent triages inbox, surfaces what needs you |
| CRM updates from memory, days later | Databases self-update from emails & meetings |
| Friday status reports from scattered docs | Daily brief compiles itself before you open your laptop |
| Chasing meeting follow-ups manually | AI meeting notes + action items routed automatically |

**The result:** Teams save **8–12 hours per week**. At a $38/hour labor cost, that's **~$20,000/year per employee** reclaimed.

---

## ✨ Features

| Agent | What It Does | Trigger |
| :--- | :--- | :--- |
| **📬 Inbox Triage** | Classifies emails (URGENT/ACTION/FYI/NOISE), drafts replies, creates tasks | Every 15 min or webhook on new email |
| **📉 Deal Staler** | Detects contacts with no touch in 5+ days, drafts follow-ups | Daily at 9:00 AM |
| **📋 Daily Brief** | Compiles meetings, top priorities, stale deals, and yesterday's highlights | Daily at 8:00 AM |
| **📝 Meeting Notes** | Transcribes, summarizes, extracts action items with owners | After calendar event ends |
| **🧠 RAG Knowledge** | Semantic search across company policies, processes, and decisions | On-demand via API |

---

## 🏗️ Architecture

```mermaid
flowchart TB
    subgraph Ingestion["📥 INGESTION LAYER"]
        Gmail[Gmail API]
        Calendar[Calendar API]
        Slack[Slack SDK]
        Webhooks[Webhooks]
    end

    subgraph Storage["🗄️ KNOWLEDGE GRAPH"]
        Tasks[(Tasks)]
        Contacts[(Contacts)]
        Meetings[(Meetings)]
        Knowledge[(Knowledge Pages)]
        Vector[(pgvector)]
    end

    subgraph Agents["🤖 AI AGENTS"]
        Triage[Triage Agent]
        Staler[Deal Staler]
        Brief[Briefing Agent]
        Meeting[Meeting Notes Agent]
    end

    subgraph Output["📤 OUTPUT LAYER"]
        Email[Email Briefs]
        Drafts[Drafts Folder]
        Dashboard[Streamlit Dashboard]
        SlackOut[Slack Notifications]
    end

    Orchestrator[⏱️ Orchestration Layer<br>Celery Beat + Redis]

    Ingestion --> Storage
    Ingestion --> Agents
    Storage --> Agents
    Agents --> Output
    Orchestrator --> Agents
    Webhooks --> Agents
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Backend** | Python 3.11 + FastAPI | REST APIs, webhook handlers, validation |
| **Database** | PostgreSQL 15 + pgvector | Structured data + vector embeddings for RAG |
| **Orchestration** | Celery + Redis | Scheduled jobs, async task queue |
| **AI/LLM** | OpenAI GPT-4o / Anthropic Claude | Classification, drafting, summarization |
| **Email** | Gmail API / Microsoft Graph | Read emails, create drafts |
| **Calendar** | Google Calendar API / Microsoft Graph | Read events, extract attendees |
| **Slack** | Slack SDK | Read messages, send notifications |
| **RAG** | pgvector + Gemini embeddings | Semantic search |
| **Frontend** | Streamlit | Lightweight admin dashboard |
| **Infra** | Docker Compose | Containerized, one-command deploy |
| **Monitoring** | Prometheus + Grafana | Metrics, logs, alerts |

---

## 📦 Getting Started

### Prerequisites

- Docker & Docker Compose installed
- Google Cloud Console project with Gmail & Calendar APIs enabled
- OpenAI API key (or Anthropic)

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/opsflow-ai.git
cd opsflow-ai
```

### 2. Configure Environment

Copy the example environment file and fill in your credentials:

```bash
cp .env.example .env
```

Edit `.env` with your API keys and OAuth credentials:

```env
# LLM Provider
OPENAI_API_KEY=sk-...

# Google OAuth (Gmail + Calendar)
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GOOGLE_REFRESH_TOKEN=...

# Database
DATABASE_URL=postgresql://opsflow:password@postgres:5432/opsflow
REDIS_URL=redis://redis:6379/0

# Optional: Slack
SLACK_BOT_TOKEN=xoxb-...
SLACK_CHANNEL_ID=...
```

### 3. Launch with Docker Compose

```bash
docker-compose up -d
```

This starts:
- PostgreSQL (with pgvector)
- Redis (message broker)
- FastAPI backend (port `8000`)
- Celery worker (background tasks)
- Celery Beat scheduler (cron jobs)
- Streamlit dashboard (port `8501`)

### 4. Verify It's Running

```bash
curl http://localhost:8000/health
# Should return: {"status":"healthy"}
```

Access the dashboard at [http://localhost:8501](http://localhost:8501)

### 5. Run Your First Agent (Manually)

```bash
# Trigger the Triage Agent
curl -X POST http://localhost:8000/api/agents/triage/run

# Generate today's brief
curl -X POST http://localhost:8000/api/agents/brief/run
```

---

## 🔐 Security & Data Privacy

- **OAuth 2.0 only:** No passwords stored; tokens encrypted at rest.
- **Least privilege:** Agents read only what they need; email writes limited to drafts (never auto-send).
- **Audit trail:** Every action is logged to the `audit_log` table.
- **Zero data retention:** LLM providers do not store your data (opt-in zero-retention policies).
- **Self-hosted option:** Keep all data within your infrastructure.
- **Emergency kill switch:** Revoke agent access instantly.

---

## 📂 Project Structure

```
opsflow-ai/
├── backend/                     # FastAPI application
│   ├── app/
│   │   ├── api/                 # REST endpoints & webhooks
│   │   ├── core/                # Config, DB, security
│   │   ├── models/              # SQLAlchemy ORM models
│   │   ├── ingestion/           # Gmail, Calendar, Slack clients
│   │   ├── agents/              # AI agent logic & prompts
│   │   ├── rag/                 # Embeddings & vector retrieval
│   │   └── services/            # Business logic layer
│   └── Dockerfile
│
├── scheduler/                   # Celery Beat configuration
│   ├── beat_schedule.py         # Scheduled task definitions
│   └── Dockerfile
│
├── worker/                      # Celery worker
│   └── Dockerfile
│
├── dashboard/                   # Streamlit admin UI
│   ├── app.py
│   └── pages/
│
├── infra/                       # Nginx, Prometheus, Grafana configs
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 🧪 API Endpoints (Preview)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Service health check |
| `POST` | `/api/webhooks/gmail` | Gmail push notification receiver |
| `POST` | `/api/webhooks/calendar` | Calendar event update receiver |
| `POST` | `/api/agents/triage/run` | Manually trigger inbox triage |
| `POST` | `/api/agents/brief/run` | Manually generate daily brief |
| `POST` | `/api/agents/staler/run` | Manually check stale deals |
| `GET` | `/api/tasks` | List all tasks |
| `GET` | `/api/contacts` | List all contacts |
| `GET` | `/api/audit/logs` | View audit trail |
| `PUT` | `/api/settings` | Update agent configurations |

Full OpenAPI documentation available at `http://localhost:8000/docs`

---

## 🙏 Acknowledgments

- Inspired by the [Notion AI Operations Playbook](https://www.notion.com/ai-operations-playbook) (July 2026 Edition)
- Built on the shoulders of giants: OpenAI, FastAPI, Celery, and the open-source community

---

<div align="center">

**Built with ❤️ for small business operators everywhere.**

</div>
```
