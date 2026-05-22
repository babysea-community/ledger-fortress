# ledger-fortress Python SDK

Atomic Stripe + Supabase credit ledger for async AI workloads.

## Runtime boundary

Use this package from trusted backend code only. It opens a PostgreSQL-compatible Supabase database connection through `psycopg2` and calls the fortress SQL functions. Do not ship database URLs, service-role keys, or this mutation surface to browser or mobile clients.

## Install from source

Until the public PyPI package is published, install the SDK from the repository:

```bash
cd client/python
pip install -e .
```

For local quality checks:

```bash
pip install -e ".[dev]"
ruff check .
pyright
```

## Basic lifecycle

```python
import os

from ledger_fortress import LedgerFortress

fortress = LedgerFortress(
    database_url=os.environ.get("SUPABASE_DATABASE_URL") or os.environ["DATABASE_URL"],
)

reserved = fortress.reserve(
    account_id=account_id,
    generation_id=generation_id,
    amount=0.062,
    model="flux-schnell",
)

if not reserved:
    raise RuntimeError("insufficient_credits")

try:
    run_generation()
    fortress.charge(
        account_id=account_id,
        generation_id=generation_id,
        amount=0.062,
        model="flux-schnell",
    )
except Exception:
    fortress.refund(
        account_id=account_id,
        generation_id=generation_id,
        amount=0.062,
        model="flux-schnell",
    )
    raise
```

## API surface

- `can_generate()` and `get_balance()` for backend reads.
- `add_credits()`, `reserve()`, `charge()`, and `refund()` for the supported credit lifecycle.
- `list_ledger()` and `build_event()` for audit/event integration.
- `recover_orphans()` for backend cron recovery.
- `set_alert_settings()`, `get_alert_settings()`, `check_alerts()`, and `reset_alerts()` for low-balance alert state.
- `get_plan_credits()` for explicit Stripe Price ID credit lookup when you use plan-based grant resolvers in your application.

## More docs

- https://github.com/babysea-community/ledger-fortress#readme
- https://github.com/babysea-community/ledger-fortress/blob/main/docs/stripe-integration.md
- https://github.com/babysea-community/ledger-fortress/blob/main/docs/architecture.md

License: Apache-2.0.
