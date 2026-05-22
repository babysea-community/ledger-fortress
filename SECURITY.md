# Security Policy

## Supported versions

`ledger-fortress` is a v0.x production OSS primitive with pre-1.0 release semantics. Security fixes target the latest public release and the `main` branch.

## Reporting a vulnerability

Please report vulnerabilities privately through GitHub's **Report a vulnerability** flow on the public `babysea-community/ledger-fortress` repository. If that flow is unavailable, contact the maintainers at `dev@babysea.ai`.

Do not open a public issue for suspected vulnerabilities. We will acknowledge valid reports as quickly as possible, investigate impact, and publish a fix or mitigation before public disclosure.

## Sentry code guard

The public OSS repository is connected to a private, repository-specific Sentry project for repository ownership, Seer-assisted review, and issue routing. The Sentry organization slug and project slug are intentionally not committed to this public repo.

This repo keeps Sentry as a repository guardrail, not runtime telemetry. It ships `scripts/sentry-project-check.mjs` and a scheduled `Sentry Project Check` workflow that verifies the configured project slug, active status, `other` platform, and Code Guard ownership rules using GitHub Actions secrets only. Use `SENTRY_AUTH_TOKEN`, `SENTRY_ORG`, and `SENTRY_PROJECT` as repository secrets. Local `.sentryclirc` files are ignored by git. No Sentry SDK, DSN, tracing, or runtime telemetry is included in this package.

## Runtime posture

`ledger-fortress` is a backend-only credit ledger. Do not call it directly from
browser or mobile clients. Runtime writes must go through a trusted backend or
service role that has already authenticated the account and authorized the
credit operation.

## Security model

| Boundary | Owner | Security requirement |
| :------- | :---- | :------------------- |
| Payment facts | Stripe | Verify raw webhook payloads with `verifyStripeSignature()` before handling the event. |
| Ledger state | Supabase | Apply all migrations, keep RLS enabled, and mutate balances only through fortress functions. |
| Account authorization | Your backend | Map Stripe customers and generation IDs to account IDs before calling the SDK. |
| Client roles | Supabase `anon` and `authenticated` | Must not read fortress tables or execute fortress RPCs. |
| Recovery jobs | Trusted cron or worker | May call `recoverOrphans()` with backend credentials and must treat callbacks as application state updates. |

## Secret handling

- Keep Supabase service-role keys, Supabase database URLs, direct database passwords, and Stripe secret/webhook keys server-side only.
- Use Stripe test-mode restricted keys for smoke validation. The real-stack smoke harness refuses live Stripe keys.
- Do not commit `.env`, smoke-test result files, database URLs, or webhook payloads containing customer metadata.
- Scope CI secrets to the repository/environment that actually runs the smoke test.
- Rotate Stripe webhook secrets, Stripe API keys, Supabase database credentials, service-role keys, and Sentry code-guard tokens through secret storage. Do not rotate by committing config files.

## Database boundary

- Apply `migrations/003_security.sql` after schema migrations to enable RLS, revoke client grants, set `SECURITY DEFINER` on mutating functions, and lock `search_path`.
- Run `scripts/verify-rls.sh`, `scripts/verify-functions.sh`, and `scripts/verify-anon-denied.sh` against a Supabase project before exposing the ledger through an API.
- Never grant `anon` or `authenticated` table writes to `credits` or `credit_ledger`.
- Do not add refund/dispute clawback behavior unless it is implemented and tested as an explicit extension; current Stripe refunds/disputes are deliberately outside this package.

## Operational guardrails

- Keep reserve, charge, refund, add-credit, alert, and recovery calls on trusted servers.
- Treat `credit_ledger` as immutable audit history. Do not repair financial state by editing ledger rows directly.
- Keep alert delivery and recovery notifications outside the critical reserve path.
- Monitor duplicate Stripe deliveries as normal retries, but investigate unexpected increases in missing-account, missing-subscription, or failed-recovery outcomes.
- Run security verification scripts after migrations, credential rotations, database restores, or privilege changes.
- Keep Package Check green: TypeScript lint, coverage, build, verification-script syntax, package dry-run, and Python package checks.

## Incident response

For suspected ledger compromise, leaked credentials, or incorrect credit movement:

1. Disable affected webhook endpoints or generation dispatch if active abuse is possible.
2. Rotate the exposed Stripe, Supabase, or Sentry credential.
3. Preserve Stripe event IDs, generation IDs, account IDs, and `credit_ledger` rows for investigation.
4. Re-run RLS and function verification scripts against the affected Supabase project.
5. Reconcile balances from Stripe paid events and immutable ledger history before reopening writes.
6. Publish a fix or mitigation before public disclosure when a vulnerability affects the OSS package.

## Data handling

`ledger-fortress` stores account IDs, generation IDs, model identifiers, Stripe-derived idempotency descriptions, balances, alert thresholds, and timestamps. Avoid storing secrets, raw webhook payloads, email addresses, or customer payment details in ledger descriptions. Keep sensitive incident details in private vulnerability reports instead of public issues.
