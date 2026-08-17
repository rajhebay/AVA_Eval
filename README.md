# AVA Eval Test Platform

**⚠️ Status: Engineering POC / internal prototype — not production-ready.** See [docs/reference/REQUIREMENT_TRACEABILITY.md](docs/reference/REQUIREMENT_TRACEABILITY.md) and [docs/reference/SECURITY_REVIEW.md](docs/reference/SECURITY_REVIEW.md) for exactly what's implemented, partial, or blocked.

A functional/regression + LLM-judge scoring + deterministic policy-check + performance-testing platform for the AVA Eval conversational agent.

## Documentation

Full documentation lives in [docs/](docs/README.md). Start with:
- [docs/onboarding/NEW_ENGINEER_GUIDE.md](docs/onboarding/NEW_ENGINEER_GUIDE.md) — new engineer walkthrough
- [docs/getting-started/QUICK_START.md](docs/getting-started/QUICK_START.md) — first run in a few minutes
- [docs/architecture/PLATFORM_ARCHITECTURE.md](docs/architecture/PLATFORM_ARCHITECTURE.md) — how it all fits together

## Quick start

```powershell
# 1) Python dependencies
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# 2) Configure environment
Copy-Item .env.example .env

# 3) Start the mock AVA API (terminal 1)
python tools/mock_api.py

# 4) Start the API backend (terminal 2)
$env:PYTHONPATH = "."
python -m uvicorn apps.api.main:app --host 127.0.0.1 --port 8010 --reload

# 5) Start the web dashboard (terminal 3)
cd apps/web
npm install
npm run dev
```

Dashboard: `http://127.0.0.1:3001`. Full detail (restart/health-check commands, Chomsky/judge configuration): [docs/operations/RUNNING_THE_PLATFORM.md](docs/operations/RUNNING_THE_PLATFORM.md) and [docs/getting-started/INSTALLATION.md](docs/getting-started/INSTALLATION.md).

## Platform modules

| Path | Purpose |
|---|---|
| `apps/api` | FastAPI control plane — runs, cases, metrics, reports, settings |
| `apps/runner` | Functional/eval run orchestration |
| `apps/perf` | Performance run tooling (Locust/JMeter, script quarantine) |
| `apps/web` | Next.js dashboard |
| `packages/evaluation` | Evaluation orchestrator, pluggable engines, policy gate |
| `packages/judge` | Chomsky/OpenAI/heuristic judge client |
| `packages/shared` | Canonical models/settings shared across modules |
| `data/cases` | Test case YAML registry |
| `data/runs` | Persisted run artifacts |
| `reports/` | Generated report exports |

Full directory reference: [docs/onboarding/REPOSITORY_MAP.md](docs/onboarding/REPOSITORY_MAP.md).

## Functional philosophy

- Deterministic hard checks are exact and rule-based, run independently of the LLM judge.
- The LLM judge (Chomsky by default) is fail-closed: unavailable/malformed judge output never silently passes.
- Functional and performance execution paths are fully isolated — no shared infrastructure or verdicts.
- Raw request/response/tool-trace/judge evidence is preserved for every run.

Details: [docs/testing/HARD_CHECKS.md](docs/testing/HARD_CHECKS.md), [docs/testing/RUBRIC_AND_LLM_JUDGE.md](docs/testing/RUBRIC_AND_LLM_JUDGE.md).

## Chomsky integration contract

- Endpoint: `https://chomskygw.vip.qa.ebay.com/api/v1/openai/chat/completions`
- Required header: `X-Ebay-Chomsky-Model-Name`
- Auth: `CHOMSKYGW_API_KEY` (primary), optional `CHOMSKYGW_TOKEN_URL` fallback

Full contract and alternate providers: [docs/integrations/CHOMSKY.md](docs/integrations/CHOMSKY.md).

## Running tests

```powershell
pytest tests/unit/ -v            # offline unit tests
pytest -n 4 -m regression -q -ra # functional regression suite (needs mock API running)
```

Full dev workflow (linting, adding cases, migrations): [docs/onboarding/DEVELOPMENT_GUIDE.md](docs/onboarding/DEVELOPMENT_GUIDE.md).

## Performance testing

```powershell
python -m locust -f perf/locustfile_ava_api.py --host http://127.0.0.1:8000 --users 5 --spawn-rate 1 --run-time 5m --headless --csv perf/results/ava_eval_perf
```

Details: [docs/testing/PERFORMANCE_TESTING.md](docs/testing/PERFORMANCE_TESTING.md).

## Reports

- Functional exports: JSONL, CSV, XLSX, HTML
- Performance exports: JSON, HTML

Details: [docs/operations/REPORTING.md](docs/operations/REPORTING.md).

## Cleanup policy

Generated artifacts should never be committed: `.venv/`, `__pycache__/`, `.pytest_cache/`, `perf/results/*.csv`, `reports/eval-*/`, `reports/fun-*/`, `data/runs/*`, `data/security_runs/*.json`. Runtime directories are kept in git via a `.gitkeep` placeholder only. See `.gitignore`.

## Something broken?

Check [docs/operations/TROUBLESHOOTING.md](docs/operations/TROUBLESHOOTING.md) before debugging from scratch — it covers the most common real issues (Celery queued-forever, mock API down, judge silently falling back, Locust false-failures, and more).
"# AVA_Eval" 
