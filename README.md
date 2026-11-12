# JobRadar

A job-posting aggregation and matching API. Pulls postings from several sources on a schedule,
removes cross-posted duplicates, and ranks what's left against a candidate profile instead of
leaving that to a keyword search.

**Stack:** Python 3.12 · FastAPI · PostgreSQL · SQLAlchemy · Celery · Redis · Docker · pytest

**Status:** not started · Python track, lower priority than the three Java projects

---

## Why this project

Every other project in this portfolio is Java. This one exists to back the Python resume
variant with something real, and because the ingest-dedupe-score pipeline is a pattern that
shows up constantly in real backends and is easy to fake badly — a dedupe that only catches
exact string matches, or a "matching" that's really just a `WHERE stack = 'Java'` filter.

## The questions this project has to let you answer

Answers go in `NOTES.md` as you build. Don't reconstruct them the night before an interview.

- What makes two postings from different sources "the same job"? What's your similarity
  check, and what's a real pair it gets wrong?
- Your ingestion runs on a Celery schedule. What happens if a source is down for one run —
  does the job fail, retry, or skip silently? What happens if it's slow enough to still be
  running when the next scheduled run fires?
- Scoring ranks postings against a candidate profile. What are the actual inputs, and what's
  the formula? Could you defend why it's not just a weighted keyword count?
- Where does Redis sit in this — what's cached, what invalidates it, and what's the
  consequence of serving a stale cache entry here versus in the ledger project?
- Show me the schema. Why does a posting belong to a `source` the way an expense belongs to
  a group?
- What's actually async here versus the bulk-import service's background worker, and why the
  different tool (Celery vs a JVM thread pool)?

---

## Build plan

One milestone per sitting, one commit per real day.

| # | Milestone | Done when |
|---|---|---|
| M0 | Scaffold — FastAPI + Postgres (SQLAlchemy) + Docker Compose, `/health` | App boots against a real Postgres |
| M1 | `Source` and `Posting` models; a seeded fixture source ingested manually | Postings persist with their source |
| M2 | Celery beat schedule — periodic ingestion task, idempotent re-runs | Running the task twice doesn't duplicate rows |
| M3 | De-duplication across sources | A planted near-duplicate pair collapses to one posting |
| M4 | Candidate-profile scoring + `GET /postings?...` filtered, ranked search | Two different profiles get different orderings |
| M5 | Redis cache-aside on search, with documented invalidation | Cache hit/miss visible; staleness window stated |
| M6 | pytest suite (ingestion, dedupe, scoring) in GitHub Actions CI | 40+ tests green in CI, not just locally |
| M7 | Frontend — search/filter UI calling the API | Can search and filter postings in a browser |
| M8 | Dockerfile, `compose.yaml`, deploy backend + frontend | Live URL for both |

---

## Setup

```bash
cd ~/projects/job-radar
python3 -m venv .venv && source .venv/bin/activate
pip install fastapi "uvicorn[standard]" sqlalchemy psycopg[binary] celery redis pytest httpx
```

```bash
podman run -d --name jobradar-pg -p 5434:5432 \
  -e POSTGRES_PASSWORD=jobradar -e POSTGRES_DB=jobradar docker.io/library/postgres:17
podman run -d --name jobradar-redis -p 6380:6379 docker.io/library/redis:7
```

> Ports **5434**/**6380** — the ledger and import projects already hold 5432/5433 and default
> Redis. On 8 GB of RAM, stop whichever project's containers you're not actively working on.

## Git

```bash
git add -A && git commit -m "docs: project spec and build plan"
```
