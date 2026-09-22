# Contributing

Thanks for contributing to GraphOne / FrontierAtlas.

## Principles

- Preserve source provenance and explicit failure accounting.
- Do not pad source results to manufacture target counts.
- Keep ingestion idempotent and schema validation explicit.
- Treat upstream API behavior as unreliable and handle failures without inventing records.

## Development

1. Make a focused branch.
2. Run the relevant unit/integration tests for the vertical or crawler you changed.
3. Run the full test suite before a broad refactor or release-oriented change.
4. Document new source adapters, environment variables, or failure modes.

## Pull requests

Explain the pipeline stage affected, sources touched, tests run, and any known upstream limitations. Changes to entity resolution, persistence, or LLM orchestration should include regression coverage where practical.
