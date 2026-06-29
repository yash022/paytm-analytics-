# PayAnalytics — Project Map

Payment analytics portal: React dashboard + FastAPI backend + MongoDB + optional LangGraph/Ollama chat.

---

## 1. Folder Structure

```
payment-analytics-frontend/
├── backend/
│   ├── app/
│   │   ├── main.py                 # FastAPI entry
│   │   ├── config.py               # Env constants (Mongo URL, SLA)
│   │   ├── settings.py             # Pydantic settings (LLM, auth, Redis)
│   │   ├── api/                    # HTTP route handlers
│   │   ├── agents/                 # LangGraph chat pipeline
│   │   │   ├── graph.py
│   │   │   ├── state.py
│   │   │   ├── llm.py
│   │   │   ├── nodes/              # planner, router, validator, run_mongo, critic
│   │   │   ├── agents/             # analytics, refund, anomaly, report, mongo_plan
│   │   │   └── tools/              # mongo_executor, analytics_tools
│   │   ├── auth/                   # JWT, users, RBAC deps
│   │   ├── cache/                  # Redis client + cached_fetch wrapper
│   │   ├── db/                     # Motor client, seed, CSV loaders
│   │   └── services/               # metrics, alerts, ingest, anomaly, insights
│   ├── data/sample/                # transactions_all.csv, refunds_all.csv
│   ├── scripts/seed.sh
│   ├── requirements.txt
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── main.jsx                # React bootstrap
│   │   ├── App.jsx                 # Routes
│   │   ├── api/client.js           # Axios API layer
│   │   ├── context/                # AuthContext, BackendContext
│   │   ├── pages/                  # Dashboard, Chat, Upload, Login
│   │   ├── components/             # Charts, layout, UI primitives
│   │   └── lib/                    # utils, formatDateTime
│   ├── vite.config.js
│   └── package.json
├── docs/PRD.md
└── scripts/dev.sh
```

---

## 2. Purpose of Major Folders

| Folder | Purpose |
|--------|---------|
| `backend/app/api/` | REST endpoints; thin handlers delegating to services or agents |
| `backend/app/agents/` | LangGraph state machine for natural-language chat |
| `backend/app/services/` | Core business logic: metrics, alerts, ingest, anomalies |
| `backend/app/db/` | MongoDB connection, indexes, seeding, CSV import |
| `backend/app/auth/` | JWT login, user seeding, role-based access |
| `backend/app/cache/` | Redis caching for metrics/health endpoints |
| `backend/data/sample/` | Demo transaction and refund datasets |
| `frontend/src/pages/` | Top-level screens (dashboard, chat, upload) |
| `frontend/src/components/` | Reusable UI: charts, nav, dashboard panels |
| `frontend/src/context/` | Global auth state and backend health polling |
| `docs/` | Product requirements and architecture notes |

---

## 3. Major Files (1–2 lines each)

### Backend — Core

| File | Purpose |
|------|---------|
| `main.py` | FastAPI app; startup hooks for indexes, users, Redis, Ollama warmup |
| `config.py` | Legacy module-level env vars (Mongo URL, SLA hours, hot data window) |
| `settings.py` | Central Pydantic config: LLM provider, JWT, Redis TTLs, feature flags |

### Backend — API

| File | Purpose |
|------|---------|
| `api/router.py` | Registers all API sub-routers |
| `api/chat.py` | `POST /chat` — runs LangGraph pipeline, returns answer + metadata |
| `api/metrics.py` | `GET /metrics/summary` — dashboard KPIs |
| `api/insights.py` | `GET /insights/summary` — optional LLM rail insights |
| `api/alerts.py` | `GET /alerts` — merchant anomaly + refund SLA alerts |
| `api/health.py` | `GET /health` — DB, Redis, LLM, transaction counts |
| `api/ingest.py` | `POST /ingest` — multipart CSV/JSON upload |
| `api/auth.py` | Login, logout, `/me` |
| `api/sync.py` | Simulated sync status and trigger |
| `api/archive.py` | Move old transactions to archive collection |
| `api/masters.py` | CRUD for banks, merchants, payment types |

### Backend — Agents (LangGraph)

| File | Purpose |
|------|---------|
| `agents/graph.py` | Builds and compiles the LangGraph; `run_chat()` entry point |
| `agents/state.py` | TypedDict shared state passed between nodes |
| `agents/llm.py` | Ollama/Gemini client, reachability probes, `invoke_text()` |
| `nodes/planner.py` | NL question → structured plan (banks, days, payment_type) |
| `nodes/router.py` | Classifies route: stats / refund / both / anomaly / clarify |
| `nodes/validator.py` | Validates query spec before execution |
| `nodes/run_mongo.py` | Calls `mongo_executor.execute_plan()` |
| `nodes/critic.py` | Final QA pass; strips fences, catches contradictions |
| `agents/mongo_plan_agent.py` | Wraps plan into JSON query spec |
| `agents/analytics_agent.py` | Computes failure summaries, spikes, comparisons |
| `agents/refund_agent.py` | Aggregates pending/breached/at-risk refunds |
| `agents/anomaly_agent.py` | Attaches anomaly detection results to state |
| `agents/report_agent.py` | Synthesizes final answer (LLM or rules template) |
| `tools/mongo_executor.py` | Deterministic MongoDB aggregation pipelines |
| `tools/analytics_tools.py` | Pure functions: failure summary, spike detection, bank compare |

### Backend — Services & Data

| File | Purpose |
|------|---------|
| `services/metrics_service.py` | Dashboard aggregations on transactions/refunds |
| `services/anomaly_service.py` | Merchant failure shift detection vs baseline |
| `services/alerts_service.py` | Combines anomaly + SLA alerts for API |
| `services/ingest_service.py` | Validates and upserts uploaded files |
| `services/insights_service.py` | Generates rail insights (rules or LLM) |
| `services/time_windows.py` | Helpers for “since N days/hours before latest txn” |
| `services/chat_service.py` | **Legacy** rules-only chat; not wired to API |
| `db/database.py` | Motor async client, indexes, `db_ok()` |
| `db/seed.py` | Loads sample CSVs into MongoDB |
| `db/csv_loaders.py` | Parses transaction/refund CSV rows |
| `db/master_data.py` | Seeds banks, merchants, currencies |
| `auth/deps.py` | `get_current_user` FastAPI dependency |
| `auth/service.py` | User CRUD, default user seeding |
| `auth/security.py` | Password hashing, JWT encode/decode |
| `cache/redis_client.py` | Redis connection lifecycle |
| `cache/service.py` | `cached_fetch()` decorator for API responses |

### Frontend

| File | Purpose |
|------|---------|
| `App.jsx` | Route tree: login, dashboard, chat, upload (RBAC) |
| `api/client.js` | Axios instance + all API call helpers |
| `context/AuthContext.jsx` | JWT storage, login/logout, role checks |
| `context/BackendContext.jsx` | Polls `/health` every 30s; exposes `connected`, `llmOn` |
| `pages/Dashboard.jsx` | Loads metrics + insights; charts and alert panels |
| `pages/Chat.jsx` | Chat UI; sends questions to `POST /chat` |
| `pages/Upload.jsx` | CSV upload form for engineers |
| `pages/Login.jsx` | Email/password login |
| `components/DashboardPanels.jsx` | KPI cards, critical alerts, refund summary |
| `components/BankFailureChart.jsx` | Recharts bank failure visualization |
| `components/TransactionHealthChart.jsx` | Rail health over time |
| `components/MerchantLeaderboard.jsx` | Top merchants by failure rate |

### Data & Docs

| File | Purpose |
|------|---------|
| `data/sample/transactions_all.csv` | Sample payment transactions |
| `data/sample/refunds_all.csv` | Sample refunds with SLA fields |
| `docs/PRD.md` | Full product spec, flows, API reference |

---

## 4. Complete Request Flow (User Query → Response)

### Chat path (`POST /chat`)

```
1. User types question in Chat.jsx
2. sendChatMessage() → POST /chat { message } (JWT attached)
3. api/chat.py validates auth, calls run_langgraph_chat(question) in thread pool
4. graph.py invokes compiled LangGraph with { question, query_retry_count: 0 }

   LangGraph nodes (in order):
   a. planner      → extracts plan (banks, days, payment_type, date range)
   b. router       → sets route + agent_type (stats/refund/both/anomaly/clarify)
   c. [branch]
      - clarify   → canned examples → END
      - refund    → refund_agent → report → critic → END
      - stats/both/anomaly:
        mongo_plan → validator → run_query (mongo_executor)
        → analytics OR anomaly agent
        → [both: refund_agent]
        → report_agent (LLM narrative or rules markdown)
        → critic → END

5. graph.py returns final_answer, route, agent metadata, refund_findings
6. chat.py maps to ChatResponse (adds refund_summary counts)
7. Chat.jsx renders answer with agent badge, validation tag, data_as_of
```

### Dashboard path (no LangGraph)

```
Dashboard.jsx → GET /metrics/summary + GET /insights/summary
  → metrics_service / insights_service → MongoDB aggregations → JSON → charts
```

---

## 5. Top 10 Files to Understand First

| # | File | Why |
|---|------|-----|
| 1 | `backend/app/agents/graph.py` | Defines the entire chat pipeline and routing logic |
| 2 | `backend/app/api/chat.py` | HTTP entry point for chat |
| 3 | `backend/app/agents/nodes/planner.py` | How NL becomes query filters |
| 4 | `backend/app/agents/tools/mongo_executor.py` | Actual MongoDB queries executed |
| 5 | `backend/app/agents/agents/report_agent.py` | How final answers are produced |
| 6 | `backend/app/services/metrics_service.py` | Dashboard data logic |
| 7 | `frontend/src/pages/Chat.jsx` | Chat UX and API integration |
| 8 | `frontend/src/pages/Dashboard.jsx` | Main operational UI |
| 9 | `backend/app/db/seed.py` | How sample data gets loaded |
| 10 | `backend/app/settings.py` | LLM, auth, cache, and feature toggles |

---

## 6. Top 10 Potentially Over-Engineered Files

| # | File | Why it may be excessive |
|---|------|-------------------------|
| 1 | `agents/graph.py` | 10+ nodes for chat; critic/validator/retry add complexity for modest safety gains |
| 2 | `agents/mongo_plan_agent.py` | Thin JSON wrapper; could be inlined into planner or run_query |
| 3 | `nodes/validator.py` | Mostly passes through; real validation is in mongo_executor |
| 4 | `nodes/critic.py` | Minimal checks; report_agent already has rules fallback |
| 5 | `agents/agents/analytics_agent.py` | Thin wrapper over analytics_tools + optional LLM call |
| 6 | `services/chat_service.py` | Duplicate legacy chat path; unused by API |
| 7 | `config.py` + `settings.py` | Two config systems with overlapping Mongo/SLA vars |
| 8 | `settings.py` LLM flags | 10+ booleans (`llm_for_*`, `gemini_for_*`) for 4 stages |
| 9 | `agents/llm.py` | Heavy probing/caching for a demo app |
| 10 | `api/archive.py` + `sync.py` + `masters.py` | Enterprise patterns (archive, sync, CRUD) beyond core demo scope |
