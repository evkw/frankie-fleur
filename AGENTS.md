# AGENTS.md — Frankie Fleur

**Read [docs/architecture.md](docs/architecture.md) before modifying code or infrastructure.** It records agreed decisions and deferred questions. Update it when material architectural choices change.

## Project intent

Build a small, reliable florist ecommerce storefront and an admin application. Start with a working proof of concept and barebones UI; refine design with the business owner. Product updates and image uploads must never require a code release. Keep costs and operational burden low while upholding production-quality engineering standards.

## Stack guardrails

- Next.js App Router, React, strict TypeScript, Tailwind CSS, shadcn/ui.
- Firebase Firestore, Firebase Authentication and Firebase Cloud Storage.
- Stripe-hosted Checkout and Netlify hosting initially.
- GitHub Actions, Vitest, Firebase Emulator Suite, Playwright and Lighthouse CI.
- Terraform for supported cloud infrastructure; Firebase CLI for rules and indexes.
- Prefer a modular monolith, server-side authoritative business logic and lean server-rendered public pages.
- Do **not** introduce PostgreSQL, extra APIs/services, headless CMS, Nx or microservices without an explicit decision.

## Environment safeguards

- **Local/CI:** emulator-only Firebase with `demo-*` ID; no real production services.
- **Staging:** separate Firebase project, Stripe test mode and Netlify environment.
- **Production:** separate Firebase project, Stripe live mode and Netlify environment.
- Fail closed on environment/project mismatch. Never expose live credentials to PR previews or use shared staging/production for destructive tests.
- Keep Firestore/Storage security rules and indexes in Git. Explicitly select target project for deploy.
- Store secrets securely; never commit keys, Terraform state or real customer data. Keep staging/production Terraform state separate and protect production data from destructive changes.

## Coding and release standards

- Small, reviewable PRs with acceptance criteria and relevant automated tests.
- Vitest for business logic; Firebase emulators for integrations and security rules; Playwright for critical user flows.
- Format, lint, type-check, test, build and check relevant Lighthouse budgets through CI.
- Gate all production deploys on successful GitHub Actions and appropriate approvals; never bypass the checks through direct Netlify publishing.
- Run Terraform fmt/validate/plan on infrastructure PRs; protect production apply, prefer federated short-lived credentials.
- Admin operations require server-side authorisation. Never trust client cart prices. Validate Stripe webhook signatures and make order fulfilment idempotent.
- Optimise mobile storefront JavaScript and product images; respect accessibility.
- Avoid building deferred features until they serve the business.

## Initial POC

Admin sign-in; product drafts and publishing; multi-image uploads; catalogue/product pages; cart; Stripe Checkout; verified order creation; basic admin order handling.

See [docs/architecture.md](docs/architecture.md) for rationale, environments, performance targets, IaC boundaries and unresolved implementation details.
