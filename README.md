# GraphOne Intelligence Pipeline

[![CI](https://github.com/rajaryan1111/graphone-intelligence-pipeline/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/rajaryan1111/graphone-intelligence-pipeline/actions/workflows/ci.yml)

An async, fault-tolerant data-ingestion platform for **AI research papers, news, jobs, startups, and products**. The project focuses on reliable ingestion rather than simply scraping data: sources are validated, records are deduplicated, failures are accounted for, and provenance is preserved.

## Why this project matters

The interesting engineering problem is not fetching JSON. It is building a pipeline that remains predictable when upstream sources fail, rate-limit, return malformed data, or change shape.

### Engineering highlights

- **Async ingestion** with worker pools and retry/backoff.
- **Source adapters** for research, news, jobs, startups and products.
- **Validation** with Pydantic schemas before persistence.
- **Idempotent storage** with SQLAlchemy and PostgreSQL/SQLite support.
- **Provenance tracking** for raw source documents.
- **LLM orchestration** with multiple providers, fallback handling and structured outputs.
- **Entity resolution** using normalization, aliases, fuzzy matching and an auditable mapping log.
- **Failure accounting** so rejected records are visible instead of silently disappearing.
- **Automated testing** with mocked upstream responses and opt-in live integration tests.

## Current status

The latest recorded implementation covers the core ingestion platform, research/news/jobs pipelines, LLM orchestration, startup/product pipelines and entity resolution.

| Area | Status |
|---|---|
| Async HTTP + retry/backoff | ✅ |
| Database models + repositories | ✅ |
| Research pipeline | ✅ |
| News pipeline | ✅ |
| Jobs pipeline | ✅ |
| LLM provider orchestration | ✅ |
| Startups + products pipelines | ✅ |
| Entity resolution | ✅ |
| Quality/export/final-audit phases | 🚧 Planned |

The repository records **452 passing unit tests with 2 integration tests deselected** at the latest documented verification point. Live integration tests are opt-in because they depend on external services.

## Architecture

```text
External APIs / RSS / Sitemaps
            │
            ▼
     Async source adapters
            │
      retry + backoff
            │
            ▼
   extraction / enrichment
            │
            ▼
      schema validation
            │
       ┌────┴────┐
       ▼         ▼
 raw provenance  entity resolution
       │         │
       └────┬────┘
            ▼
     idempotent storage
      PostgreSQL / SQLite
            │
            ▼
     metrics / exports
```

## Example reliability behavior

The pipeline distinguishes between **real access failures and rate limits**. For example, GitHub can signal anonymous rate limiting through `403` plus `X-RateLimit-Remaining: 0`; the HTTP layer handles that differently from a genuine permanent `403`.

The project also avoids fabricating missing records. If an upstream source returns fewer valid records than requested, the pipeline records the shortfall instead of padding the result set with invented data.

## Data-source handling

The research pipeline uses official/public sources such as **arXiv, OpenAlex and GitHub enrichment**. News and jobs use a mixture of APIs, RSS feeds and structured sitemap data.

A retired Papers With Code adapter is retained as disabled historical code and has been replaced by OpenAlex. The disabled adapter is not used in the active pipeline.

## Tech stack

**Python 3.11+** · asyncio · httpx · Pydantic v2 · SQLAlchemy 2 · PostgreSQL · SQLite · Redis · pytest · pytest-asyncio · respx · structlog · Docker Compose

## Repository structure

```text
src/
├── config/          settings, logging, source registry
├── crawlers/        HTTP client and source adapters
├── extraction/      dates, URLs, external enrichment
├── validation/      Pydantic validation
├── storage/         SQLAlchemy models and repositories
├── pipeline/        worker pools and vertical orchestration
├── llm/             provider abstraction and orchestration
├── resolution/      entity matching / mapping
└── export/          export-related modules

tests/               unit + opt-in integration tests
fixtures/            captured upstream response fixtures
```

## Run locally

```bash
cp .env.example .env
pip install -r requirements.txt
docker compose up -d postgres redis
```

Run the test suite:

```bash
pytest
```

Opt into live integration checks only when external access is available:

```bash
pytest -m integration
```

Run a pipeline from the CLI:

```bash
python -m src.main --help
python -m src.main --vertical research --target 1000 --workers 50
```

## Engineering principles

1. **Never invent upstream data.** Missing or invalid fields cause rejection rather than silent fabrication.
2. **Make failures observable.** Counters and structured errors explain where records were lost.
3. **Validate before persistence.** External data crosses a typed validation boundary before entering storage.
4. **Prefer idempotent operations.** Re-running a pipeline should not duplicate records.
5. **Keep providers replaceable.** External LLM/API integrations sit behind application-level interfaces.
6. **Separate demo claims from production claims.** External-service availability and benchmark results are documented with their limitations.

## Limitations

- Some pipelines depend on external services and their current availability/rate limits.
- A production deployment would need a proper migration workflow instead of relying on development-time schema creation.
- Live integration tests are intentionally separated from the deterministic unit suite.
- Quality/export/final-audit work remains future scope.

## Author

Built by **Raj Aryan** as an AI/data-engineering project focused on reliability, validation and production-oriented pipeline design.
