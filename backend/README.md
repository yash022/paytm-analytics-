# PayAnalytics Backend

**MongoDB + LangGraph** — FastAPI + Motor for data; LangGraph agents for chat analysis (optional Ollama/Gemini).

## Structure

```
app/
├── main.py           # FastAPI entry
├── config.py         # MONGODB_URL, SLA_HOURS, …
├── api/
│   ├── router.py     # Registers all routes
│   ├── health.py
│   ├── metrics.py
│   └── …
├── db/
│   ├── database.py   # Motor client + indexes
│   ├── seed.py
│   ├── csv_loaders.py
│   └── master_data.py
├── agents/           # LangGraph graph, planner, Mongo executor, LLM
└── services/         # metrics, ingest, alerts, …
```

## Run

```bash
source .venv/bin/activate
pip install -r requirements.txt
python -m app.db.seed
# Optional: ollama pull llama3.2
uvicorn app.main:app --reload --port 8020
```

Chat uses LangGraph (`POST /chat`). Check `GET /health` for `langgraph: enabled` and `llm_configured`.

API docs: http://127.0.0.1:8020/docs

## Environment

Copy `.env.example` → `.env` (optional). Defaults work for local Mongo.

## Sample data

`data/sample/transactions_all.csv`  
`data/sample/refunds_all.csv`
