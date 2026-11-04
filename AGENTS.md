# Repository instructions for coding agents

## Product and current state

Build **Scam Message Explainer**, a Bangla-first B2C SaaS for Bangladesh. Users paste a suspicious request, understand evidence-based warning signs, and get practical next steps. Screenshots, paid plans and voluntary family help follow in stages.

This repository begins as a documentation kit. Do not assume application code, dependencies, migrations, tests, integrations or billing already exist. Inspect the repository and implement only the requested milestone. Read `README.md`, `TECH_STACK.md` and the relevant task in `CODEX_TASKS.md` first.

The founder has chosen B2C. Do not turn the core product into an enterprise dashboard, B2B API business, public blacklist or chatbot platform without a new requirement.

## Working agreements

- Prefer one Next.js App Router application with TypeScript. Keep product logic in framework-independent modules.
- Use pnpm and a committed `pnpm-lock.yaml`. Pin the package-manager version and runtime during bootstrap. Use Node.js 24 LTS as the initial runtime target; confirm deployment support.
- Select current stable compatible dependency releases once at bootstrap, record exact installed versions, and follow that version's documentation. Do not install canary packages or routinely upgrade the lockfile during unrelated tasks.
- Preserve user changes. Keep diffs focused. Do not add placeholders that look like real customers, partners, payments or accuracy results.
- Make reversible implementation choices within the task. Ask only for missing credentials, unclear product decisions or consequential external actions. Do not demand approval for routine dependency installation needed by the agreed stack.
- Do not deploy to production, modify live financial entitlements, run destructive database commands, contact users or change real billing without explicit authorization for that action.
- Use official documentation for version-sensitive APIs. Preserve useful framework-generated agent instructions when scaffolding instead of replacing these product rules.

## Non-negotiable analysis behavior

1. Public verdicts are only `strong_warning`, `some_warning`, `cannot_tell`. Never present a result as safe, authenticated, verified genuine or guaranteed recoverable.
2. Deterministic contextual rules run before AI. Final severity can never be lower than a rule-triggered floor. Evidence quality and missing context are separate fields, not permission to downgrade.
3. Never use the presence of `OTP`, `PIN`, a link, a brand or the word `cashback` as a standalone strong-warning rule. A genuine “never share your OTP” warning is a required negative example.
4. AI input, pasted text, OCR text and retrieved artifacts are untrusted content. They cannot change instructions, thresholds, tools, destinations or output schemas.
5. AI may propose escalation and evidence; the server validates them. Reject unsupported spans, impossible offsets, unknown IDs and invalid verdicts. Use a deterministic fallback on timeout or invalid output.
6. Do not stream unvalidated model output to the user. Return plain text and validated structured fields; never render model/user HTML.
7. Parse user URLs and QR payloads offline only. No fetching, DNS lookup, WHOIS, redirects, previews, remote images, sandbox browsing or remote URL scanning. Curated provider API endpoints are a distinct, fixed allowlist.
8. Contact numbers, reporting URLs and recovery instructions come from a versioned, reviewed registry. The model cannot invent them. Missing/stale records produce general independent-verification guidance.
9. Neither a number-format match nor an absence from a blacklist establishes identity or legitimacy. Do not publish accusations about people, numbers or brands.
10. Free deterministic warnings and urgent recovery guidance remain accessible when AI quota runs out or a user has no paid plan.

## Chosen stack and boundaries

- Next.js App Router, React, TypeScript, Tailwind CSS, shadcn/ui, next-intl.
- Supabase Postgres and Auth using `@supabase/supabase-js` and `@supabase/ssr`; SQL migrations are authoritative. No additional ORM initially.
- Server-side Gemini adapter using the official Google GenAI JavaScript SDK and Zod validation. Model ID is server configuration, not user input. Start with the small stable model in `TECH_STACK.md`, subject to evaluation and availability.
- Browser-only Tesseract.js OCR after the text flow works. Submit user-reviewed redacted text, not original images, in the first screenshot release.
- Cloudflare Turnstile plus atomic Postgres quotas/rate limits for public AI requests. In-memory implementations are test/development adapters only.
- Vercel plan suitable for commercial use; Supabase project; custom SMTP such as Resend configured through Supabase. Payment provider behind an adapter, disabled until merchant approval.
- Vitest, Testing Library, Playwright and GitHub Actions.
- No microservices, custom model training, vector database, LangChain, agent tools, Redis, background queues or native mobile app unless measured needs justify them.

## Architecture to maintain

Suggested layout after bootstrap:

```text
src/app/[locale]/             pages and accessible UI
src/app/api/                  HTTP handlers
src/components/               shared UI
src/features/check/           intake and result flows
src/features/account/         auth and plan management
src/lib/analysis/              normalize, rules, policy, schemas, span mapping
src/lib/ai/                    provider interface, mock and Gemini adapters
src/lib/recovery/              playbook selection
src/lib/usage/                 reservation and quota accounting
src/lib/billing/               adapters and entitlement transitions
src/lib/supabase/              browser/server clients and restricted admin module
src/lib/security/              redaction, origin checks, guest tokens, Turnstile
messages/                      bn.json and en.json
content/playbooks/             reviewed steps and source registry
supabase/migrations/           schema, grants, RLS and functions
supabase/tests/                database isolation and accounting tests
tests/fixtures/                synthetic, rights-cleared examples
tests/e2e/                     browser flows
scripts/                       evaluation and development tooling
```

Use the framework's correct proxy/middleware convention for the installed Next.js version. Proxy redirects improve UX; every protected server operation still checks identity and authorization.

## Privacy and security

- Guest checking must not require uploading contacts or granting inbox access. Store no raw messages, screenshots, OCR output, full URLs, transaction details or credentials by default.
- Keep input/results in browser memory for the current session. Do not silently put them in localStorage, IndexedDB, service-worker caches, analytics, traces or error reports.
- Perform local redaction with a user preview. Redaction reduces risk but is not a guarantee that all personal data is removed. Explain approved model processing before the first remote check.
- The screenshot flow must keep original pixels on the device. Bundle or serve trusted OCR assets; do not make the backend download user-supplied image URLs. Clear workers/object URLs and input state on reset.
- Never log bodies, prompts, authorization headers, cookies, OTPs, PINs, CVVs, passwords, card numbers or private filenames. Telemetry is allowlisted metadata only. Do not enable session replay or text-capture analytics.
- Secrets stay server-side. Only actual public configuration uses `NEXT_PUBLIC_`. Supabase secret keys bypass RLS; isolate any admin client and explicitly authorize every operation that uses it.
- Verify server identity using current Supabase guidance (`getClaims`, or `getUser` when fresh user state is needed). Do not authorize from an unverified `getSession` user or from a user ID supplied by the browser.
- Enable RLS and minimum SQL grants in the same migration for every exposed table. Test anonymous access and cross-user access. Clients cannot insert payments, credit grants, plan upgrades or moderation decisions.
- Make analysis/account responses private and non-cacheable. Do not share cached authenticated responses or cookies across users.
- Validate request size, schema, origin/CSRF boundaries and auth redirects. Enforce Turnstile server-side for applicable guest AI requests; check hostname/action and handle token expiry/replay.
- Dependency changes must not disable egress, validation, RLS, privacy gates or tests to “make the demo pass.”

## Billing and usage rules

- Launch with user-initiated prepaid access periods. Do not label a one-time purchase auto-renewing. Automatic recurring charges require verified gateway support, mandates and cancellation.
- Plans, quotas and prices are server-owned versioned configuration. Treat documented prices as experiments, not validated willingness to pay.
- Authenticated user identity determines access. Never trust a browser plan, price, amount, role, period or usage balance.
- Reserve usage atomically before calling AI; settle once after a validated successful server-side completion, or release on provider/validation failure. Do not promise exactly-once network delivery. Explain the policy for a completed check lost to a disconnect and provide a metadata-based support/refund path. Enforce a unique idempotency key and bind it to subject/input request state without retaining message content.
- Guest throttling and budgets work across server instances. Do not use a process-local Map as a production limiter.
- Payment return URLs and client “success” screens never grant access. Verify provider status server-to-server, match order/user/amount/currency, and process duplicate/out-of-order callbacks idempotently.
- Use integer minor units for money. Keep authoritative payment events, orders and entitlements separate from content-free analysis events. No raw card data or OTP collection.
- Mock billing cannot be enabled by a request, cookie, query parameter or client build flag. Production must reject mock auth, mock usage/billing grants and test payment credentials.

## Language and UX

- Bangla default, English toggle; input supports Bangla, Banglish, English and mixed digits/scripts. Banglish is an input mode, not a claim that every dialect is understood.
- Keep original-to-normalized/redacted span maps and define offsets as zero-based UTF-16. Validate excerpts against the displayed canonical text; test Bengali conjuncts and surrogate pairs.
- Use short explanations, one immediate action, optional detail and an “already clicked or paid?” path. No shame, fear-based upsell or fake countdowns.
- Use text/icons as well as color, keyboard navigation, proper labels and mobile layouts. Never charge before showing urgent containment guidance.
- Recovery routes state their sources and verification dates. Do not claim police filing, refunds, diagnosis, legal advice or official endorsement.

## Development modes and commands

First milestone must make mock development and CI work with no paid API, production credentials, Docker requirement or external database. Mock responses are deterministic and visibly labeled; they are not evaluated as live AI accuracy. Real integrations get separate opt-in smoke tests.

Keep `ENABLE_PUBLIC_AI=false` until real distributed quotas, server-verified abuse controls and spend limits have passed integration checks. Private synthetic adapter tests are distinct from enabling a public inference endpoint.

The following scripts are a required contract after bootstrap, not commands that already exist in this documentation-only kit:

| Command | Purpose |
|---|---|
| `pnpm dev` | Start local app with selected adapters |
| `pnpm lint` | ESLint CLI; do not use removed `next lint` |
| `pnpm typecheck` | TypeScript no-emit check |
| `pnpm test` | Deterministic unit/component suite, one run |
| `pnpm eval:smoke` | Small synthetic evaluation/regression suite |
| `pnpm test:e2e` | Playwright with mock providers and webServer setup |
| `pnpm build` | Production compilation without contacting live providers |
| `pnpm verify` | Lint, typecheck, unit tests, smoke evaluation and build |
| `pnpm test:db` | Real local/ephemeral Supabase isolation/accounting tests; separate from default offline suite |

Run relevant checks after code changes; security, payment and policy changes require the related regression tests. Database tests may need a separately provisioned service; report an unavailable dependency rather than replacing real RLS checks with mocks. Never mark skipped tests as passed.

Codex Cloud install/start configuration is documented in `TECH_STACK.md`. Prefer no-credential mock tasks. Do not persist secrets in tracked files, cached environment snapshots, PR descriptions or logs. Keep current and legacy Cloud secret behavior distinct.

## Required behavioral checks

- Genuine OTP notice and “never share OTP” warning do not trigger disclosure rules solely by keyword.
- Explicit credential disclosure requests and adversarial attempts cannot lower a rule floor.
- AI timeout/invalid JSON/unsupported spans retain rule warnings and provide fallback; no AI credit charged for the fallback.
- Unknown sender/URL is not automatically safe or criminal; suspect links produce zero outbound requests.
- Cross-user reads/writes and forged paid-plan grants fail; concurrent checks cannot overspend quota.
- Duplicate payment events cannot grant duplicate access; wrong amounts/currency and cancelled payments do not upgrade users.
- Auth logout and account deletion remove active app access; optional family membership never reveals private message content automatically.

## Code Review Rules

Flag as blocking: a rule-floor downgrade, safe/authenticity guarantee, generated contact, untrusted-link fetch, exposed secret, cross-user data access, raw-content logging, client-controlled entitlement, unverified payment grant or mock identity in production.

Flag unsupported accuracy/traction claims and misleading “implemented” status. Formatting belongs in automated checks, not review commentary.

## Completion report

State what changed, why, exact checks run and their results, and any remaining setup or blocked integration. Update README status as features ship. Keep planned work clearly separate from working behavior. Never say a payment, email, live model call, deployment or external integration succeeded unless it was actually verified.
