# ledger-fortress TypeScript SDK

Atomic Stripe + Supabase credit ledger for async AI workloads.

## Runtime boundary

Use this package from trusted backend code only. It opens a PostgreSQL-compatible Supabase database connection through `pg` and calls the fortress SQL functions. Do not ship database URLs, service-role keys, or this mutation surface to browser or mobile clients.

## Install from source

Until the public npm package is published, build the SDK from the repository:

```bash
cd client/typescript
npm install
npm run build
```

Then install the local package into your backend application:

```bash
npm install /path/to/ledger-fortress/client/typescript
```

## Basic lifecycle

```typescript
import { LedgerFortress } from 'ledger-fortress';

const fortress = new LedgerFortress({
  databaseUrl: process.env.SUPABASE_DATABASE_URL ?? process.env.DATABASE_URL!,
});

const reserved = await fortress.reserve({
  accountId,
  generationId,
  amount: 0.062,
  model: 'flux-schnell',
});

if (!reserved) {
  throw new Error('insufficient_credits');
}

try {
  await runGeneration();
  await fortress.charge({ accountId, generationId, amount: 0.062, model: 'flux-schnell' });
} catch (error) {
  await fortress.refund({ accountId, generationId, amount: 0.062, model: 'flux-schnell' });
  throw error;
}
```

## API surface

- `canGenerate()` and `getBalance()` for backend reads.
- `addCredits()`, `reserve()`, `charge()`, and `refund()` for the supported credit lifecycle.
- `listLedger()` and `buildEvent()` for audit/event integration.
- `recoverOrphans()` for backend cron recovery.
- `setAlertSettings()`, `getAlertSettings()`, `checkAlerts()`, and `resetAlerts()` for low-balance alert state.
- `getPlanCredits()` for explicit Stripe Price ID credit lookup when you use plan-based grant resolvers.

## Stripe helpers

Import `ledger-fortress/stripe` for `verifyStripeSignature()` and `createStripeWebhookHandler()`. The helper handles only `invoice.paid`, `checkout.session.completed`, and `checkout.session.async_payment_succeeded`; refund, dispute, chargeback, and debt workflows stay application-owned.

## Verification

```bash
npm run lint
npm test
npm run build
```

Use a disposable Supabase project or local PostgreSQL developer stand-in before running database tests:

```bash
LEDGER_FORTRESS_CONFIRM_DISPOSABLE_DB=1 npm run test:db:concurrency
```

## More docs

- https://github.com/babysea-community/ledger-fortress#readme
- https://github.com/babysea-community/ledger-fortress/blob/main/docs/stripe-integration.md
- https://github.com/babysea-community/ledger-fortress/blob/main/docs/architecture.md

License: Apache-2.0.
