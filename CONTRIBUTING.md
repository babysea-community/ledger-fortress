# Contributing

Thanks for improving Ledger Fortress.

Ledger Fortress is an atomic credit-settlement primitive for Stripe + Supabase systems. Good contributions protect idempotency, preserve no-overdraft behavior, and keep payment, database, and service-role boundaries explicit.

## Contribution guidelines

- Keep all contributions under Apache 2.0. By submitting a PR you agree to license it under Apache 2.0.
- Preserve v1 schemas. If a change requires breaking `schemas/credit-event.v1.json` or `schemas/credit-alert.v1.json`, publish a v2 alongside it.
- Treat idempotency as a hard contract. Any SQL function that mutates `credits` or `credit_ledger` must be provably idempotent and tested by calling it twice.
- Cover edge cases when touching `reserve_credits`, `charge_credits`, `refund_credits`, or crash recovery.
- Keep unsupported payment flows visibly outside the package. Do not document clawbacks, dispute handlers, uncollectible debt tracking, or `settle_credits` unless BabySea production implements the same flow and the OSS surface is updated.
- Keep service-role database credentials, Stripe secrets, Sentry auth tokens, webhook payloads, and customer identifiers out of public fixtures and logs.
- Keep the TypeScript and Python SDK behavior in sync when changing ledger operations or payload contracts.
- Prefer focused changes. Avoid unrelated refactors in migrations, SDK code, demos, or deployment docs.

## Documentation standard

Ledger Fortress docs are part of the public contract for this primitive. Keep them factual, operator-ready, and tied to behavior that exists in this repository.

- Start from the README contract: what the primitive is, what it is not, how to deploy it, how to validate it, and how to recover it.
- Use exact SQL function names, table names, schema names, environment variable names, commands, and file paths.
- Use Supabase-first terminology. Use PostgreSQL only for Supabase SQL behavior, PostgreSQL-compatible URLs, `psql`, database client libraries, connection details, or local developer stand-ins.
- Document validation steps beside operational claims. If a guide says a path is production-ready, include the check, workflow, or smoke harness that proves it.
- Keep security guidance concrete: where service-role keys live, which values must not be logged, how keys are rotated, and what should never be posted publicly.
- Update `CHANGELOG.md` for user-visible docs, configuration, security, SDK behavior, schema, deployment, or operations changes.
- Avoid roadmap language in the public contract. New features stay out of README claims until implemented, documented, and validated for this stack.

When a change touches these areas, update the matching docs before opening a PR:

| Change area                    | Required docs to review                                           |
| :----------------------------- | :---------------------------------------------------------------- |
| SQL ledger behavior            | README invariants, `docs/edge-cases.md`, PgTAP tests, SECURITY.md |
| Stripe event handling          | README Stripe sections, `docs/stripe-event-matrix.md`, SDK docs   |
| SDK operation shape            | TypeScript README, Python README, examples, schemas               |
| Security or RLS behavior       | README production readiness, SECURITY.md, verification scripts    |
| Demo or local-stack behavior   | README quick start, Docker Compose docs, demo scripts             |
| Sentry or CI workflows         | README release gates, SECURITY.md, this guide                     |
| Schema or event envelope shape | README events, JSON Schemas, examples, changelog                  |

## Development flow

### SQL

```bash
cd examples/docker-compose-local
docker compose up -d
psql postgresql://fortress:fortress@localhost:5432/fortress
```

### TypeScript SDK

The published SDK targets Node.js 18+ at runtime. Local TypeScript SDK development uses the Vitest 4/Vite 8 toolchain and requires Node.js 20.19+ or 22.12+.

```bash
cd client/typescript
npm install
npm run lint
npm run test:coverage
npm run build
```

### Python SDK

```bash
cd client/python
pip install -e ".[dev]"
ruff check .
pyright
pytest
```

## Before opening a pull request

Run the checks that match your change:

```bash
(cd client/typescript && npm run lint && npm run test:coverage && npm run build)
(cd client/python && ruff check . && pyright && pytest)
```

For ledger or SQL changes, also run the PgTAP invariant suite and the disposable concurrency simulation documented in the README.

## Issue triage

- `bug` - reproducible defect, with logs, a failing test, or a minimal reproduction.
- `proposal` - scoped design idea with the user problem, implementation sketch, and validation path.
- `good first issue` - small, well-scoped change that can be validated without production credentials.

## Conduct

See [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md). Be respectful, assume good faith, and keep discussion focused on the work and the people using it.

## Security-sensitive changes

Open security fixes privately through the process in [`SECURITY.md`](SECURITY.md). Do not include real Stripe keys, private payment data, customer identifiers, database URLs, service-role credentials, webhook payloads, unreleased vulnerability details, or live production data in public issues, pull requests, test fixtures, logs, or screenshots.
