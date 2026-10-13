# Frankie Fleur — Architecture

**Status:** Accepted technical direction, 14 October 2026. This describes choices made before implementation; adjust it through reviewed architecture changes rather than silently substituting technology.

## Product and engineering principles

Frankie Fleur is a small florist storefront that may evolve into a business operations app. Ship a **functional proof of concept with basic UI** first; improve the customer and admin experience iteratively with the business owner. Admin users must be able to create, edit, publish and unpublish products (including photographs) **without a code deployment**.

Priorities: low costs, fast delivery for a solo developer, secure payments and customer data, fast mobile storefront, professional quality gates, and maintainable code. Avoid speculative complexity. If meaningful growth eventually calls for relational inventory and profitability modelling, revisit the Firebase choice then rather than preemptively building a migration architecture.

## Technology decisions

| Area | Choice |
| --- | --- |
| Web application | Next.js App Router + React + strict TypeScript |
| UI | Tailwind CSS + shadcn/ui |
| Backend | Next.js server routes / server-side logic (modular monolith) |
| Database | Firebase Cloud Firestore |
| Authentication | Firebase Authentication; initially restricted admin Google sign-in |
| Product photography | Firebase Cloud Storage |
| Payment | Hosted Stripe Checkout |
| Initial hosting | Netlify |
| Unit tests | Vitest |
| Integration / permission tests | Firebase Emulator Suite |
| End-to-end | Playwright |
| CI/CD | GitHub Actions |
| Web performance | Lighthouse CI + field Core Web Vitals |
| Infrastructure as code | Terraform for supported GCP/Firebase resources; Firebase CLI for rules and indexes |

**Why Next.js, not Astro?** Astro already works well for the business owner's portfolio and can be exceptionally fast for static content. This project includes substantial React administration and full-stack capabilities, while Next.js enables a single application and production Next.js experience. Treat mobile performance as a quality gate, not an assumed property of the framework.

**Why Firebase rather than Neon/PostgreSQL + R2?** Faster delivery, familiar tools and integrated Firestore/Auth/Storage suit v1. Accept some vendor coupling; reconsider only when real business requirements justify the cost of engineering change.

## Scope and architecture

One Next.js application, organised into domain-focused features such as products, checkout, orders and admin. Do not start with microservices, a separate API, a headless CMS, Nx or a complicated monorepo. Put business logic, pricing, validation and privileged Firebase operations in server-side modules, not React components.

Initial vertical slices: admin authentication and authorisation; product CRUD with draft/published status and image uploads; public catalogue and product details; cart; Stripe Checkout; verified Stripe webhook and persisted orders; simple admin order view. Later possibilities (not initial scope): flower-market purchasing sheets, tasks, inventory, analytics, cost/profitability and supplier insights.

Use Firestore to store product descriptions, prices and Storage object references; store actual images in Firebase Storage. Admin image uploads must work from phones and computers. Validate upload type/size, optimise sizes/formats as practical, and enforce Storage permissions. Publish changes through data writes with a deliberate strategy for Next.js caching/revalidation so a new product appears without rebuilding the site.

Prefer Server Components on public storefront pages and small interactive client islands for cart, selectors and other necessary interaction. The admin area can use richer React interactivity.

## Three environments, two real Firebase projects

| Environment | Firebase | Stripe | Application |
| --- | --- | --- | --- |
| Local and CI | Emulator Suite (Auth, Firestore, Storage), `demo-*` project ID | Mocks or test mode | Local Next.js / CI |
| Staging | Dedicated Firebase project | Test mode | Staging Netlify site/context |
| Production | Separate Firebase project | Live mode | Production Netlify site/context |

- Never point local emulator tests, PR previews or staging at production data, credentials or Stripe live mode.
- Validate expected environment, Firebase project ID and Stripe mode on startup/deploy; fail closed on mismatch.
- Use explicit Firebase project aliases/flags; never rely on an accidentally selected CLI default.
- Version-control Firestore/Storage rules and Firestore indexes.
- Use deterministic local/staging seed data. Run destructive E2E tests only against isolated emulators, not shared staging or production.
- A future production-to-staging refresh must anonymise customer data and account for Firestore, Auth and Storage separately; prohibit sending real notifications and payments.
- Firebase Storage requires billing-enabled project configurations; enable budgets/alerts while recognising alerts do not automatically cap spending.

## Infrastructure as code

**Terraform** should provision long-lived resources where the Google/Firebase providers support them reliably: projects, enabled APIs, Firestore databases, Storage buckets, app registrations, IAM/service accounts and supported Identity Platform configuration. Some Firebase-related provider coverage may change or be preview; check current provider documentation before implementing. Keep staging and production Terraform state separate, with shared modules and per-environment variables. Use protected remote state; protect production data resources against accidental deletion. Never commit credentials or state.

**Firebase CLI** handles Firestore indexes and Firestore/Storage rules tracked in Git (`firebase.json`, `.firebaserc`, rules and index definitions), explicitly deployed to staging or production. Avoid manual Firebase Console drift; document and reconcile unavoidable bootstrap or emergency edits.

**GitHub Actions** runs Terraform format/validate and a readable plan on relevant PRs. Applying to production should require trusted identities, reviewed plans and protected approval. Prefer Workload Identity Federation/OIDC instead of long-lived service account keys where supported. Initial Google Cloud billing and auth/bootstrap setup may require one-time manual steps; document them. Deployment settings and secrets should be managed through appropriate environment-scoped facilities rather than source control.

## Security and payments

Restrict admin access to authorised users and enforce this in server operations as well as relevant Firestore/Storage rules. Do not trust browser-provided prices or user roles. Checkout sessions are created server-side with validated products and prices. Use Stripe-hosted payment collection, verified webhook signatures, idempotent webhook processing and server-controlled order state. Never handle raw payment-card information. Limit access to customer order data.

## CI, testing and release quality

Every PR should pass:
1. Formatting, ESLint, TypeScript strict checking.
2. Vitest unit tests.
3. Firebase emulator-backed integration and Firestore/Storage security rule tests.
4. Playwright critical-path E2E against emulators (initially Chromium).
5. Production build.
6. Lighthouse CI on representative public pages once they exist.

Capture/upload Playwright traces, screenshots and reports when tests fail. Use deterministic fixtures; keep routine tests independent of live Firebase and external Stripe where possible, with separately scoped Stripe test-mode integration checks.

Use GitHub Actions as the release gate, not independent Netlify auto-publishing that can bypass tests. Protect main with required checks/PR workflow where repository settings permit. Use separate staging/production deployments, environment-scoped secrets and explicit production approval where appropriate.

## Performance, accessibility and SEO

Target **95+ Lighthouse mobile performance**, **95+ accessibility**, **100 SEO** on representative storefront pages; target good field Core Web Vitals at p75 (LCP ≤2.5s, INP ≤200ms, CLS ≤0.1). These are goals, not guarantees. Focus on responsive image optimisation, reserved image dimensions, minimal client JS, caching/revalidation and stable layout.

## Open details

Exact Firebase project IDs, billing linkage, Terraform state backend/bootstrap, Node/package manager versions, Netlify site setup, auth session approach, image transformation/CDN implementation and deployment/promotion specifics are yet to be selected. Make these explicit during foundation work instead of treating an illustrative configuration as decided.
