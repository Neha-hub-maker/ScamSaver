# Technical stack and build decisions

**Decision date: 4 November 2026.** These are proposed implementation decisions for a Bangla-first B2C SaaS, not a statement that the application has been built. Version-sensitive behavior was checked against the official references below. Pin actual package versions during bootstrap; do not treat `latest` as a permanent production dependency.

## 1. Recommended shape

Use a **single Next.js application with separate internal modules**, one managed Postgres database and one server-side AI adapter. The same application serves the landing page, checker, recovery content, accounts and billing. This reduces the number of deployments and languages a solo founder must maintain.

Do not start with a separate Express/FastAPI backend, microservices, an agent framework, vector search or custom model training. The initial reasoning task needs structured classification/explanation, not autonomous browsing. Add infrastructure only after a measured limitation appears.

| Area | Selected approach | Why / tradeoff |
|---|---|---|
| Runtime | Node.js 24 LTS, exact package-manager version pinned in package.json | Mature supported runtime; validate host support. Newer is not automatically better for a first product [D3] |
| Framework | Current stable Next.js App Router + React + TypeScript | One full-stack deployment and common coding-agent tooling. Pin compatible versions after bootstrap [D4] |
| Styling | Tailwind CSS + shadcn/ui | Fast, editable component source. Use only needed components; avoid a second component framework |
| Internationalization | next-intl; `bn` and `en` UI locales | Typed dictionaries and consistent Bangla/English output. Banglish remains supported input, not a separate UI locale [D5] |
| Database | Supabase Postgres, SQL migrations, generated TypeScript types | One authoritative source for identity-linked quotas, payments and access. No Prisma/Drizzle initially |
| Auth | Supabase Auth via `@supabase/ssr`; email magic link first | Keep auth and RLS integrated. Google OAuth can follow if users request it; production email needs custom SMTP [D6–D9] |
| Rules | Plain TypeScript, versioned predicates and policy | Deterministic behavior is testable without a paid model |
| AI | Official Google GenAI JavaScript SDK (`@google/genai`), server-side, provider interface | Start with configurable `gemini-2.5-flash-lite` if still available and it passes the project evaluation. Record replacement decisions; do not silently switch models [D10–D11] |
| Validation | Zod for API input, model response and environment configuration | A JSON schema does not guarantee truthful evidence or valid actions; validate business semantics too [D10] |
| Screenshots | Tesseract.js browser worker, Bengali/English OCR assets, user correction and redaction | Keeps original pixels off the server in the first image release. OCR quality and download size must be measured on modest phones [D12] |
| Abuse prevention | Turnstile + signed guest cookie + distributed database limits | Challenge verification and quotas happen server-side. A cookie alone cannot prevent repeat abuse [D13] |
| Payments | SSLCOMMERZ adapter after merchant approval; prepaid plans first | Local collection path, but SaaS eligibility and renewal terms need written confirmation. bKash documents subscription capability; access/fees remain merchant-specific [D14–D15] |
| Hosting | Vercel plan permitting commercial use, Supabase managed project | Direct Next.js workflow. Vercel Hobby restricts use to noncommercial personal projects, so do not assume it covers this business [D16] |
| Email | Supabase custom SMTP, initially Resend if its account/domain requirements fit | No email vendor API needed in app code for the first auth flow; configure SMTP in Supabase [D9] |
| Tests | Vitest + Testing Library + Playwright; Supabase database tests separately | Fast deterministic PR checks, browser flows and real SQL isolation tests serve different purposes |
| Automation | GitHub Actions, lockfile install, preview deployment | Repeatable checks; no paid model/network calls in default CI |
| Monitoring | Allowlisted structured counters and sanitized errors | Track latency, cost, verdict category and failures. Add an error vendor only with content capture disabled and verified |

## 2. Routes and core flow

Proposed route contract:

| Route | Responsibility |
|---|---|
| `/bn`, `/en` | Landing page and paste intake, usable as a guest |
| `/bn/check`, `/en/check` | Input/result UI, no public shareable result identifier |
| `/bn/recover`, `/en/recover` | Free approved exposure guidance, no AI dependency |
| `/bn/pricing`, `/en/pricing` | Clearly labeled live offer only after it exists; honest limits/renewal |
| `/bn/account`, `/en/account` | Protected plan, remaining usage, privacy and deletion controls |
| `/auth/callback` | Verified auth callback; local allowlisted redirect paths |
| `/api/analyze` | Input validation, abuse check, atomic usage reservation, rules, optional AI and final policy |
| `/api/billing/checkout` | Authenticated server-owned order creation; provider adapter |
| `/api/billing/notify` | Provider-defined notification verification and reconciliation, no browser plan trust |
| `/api/account/delete` | Fresh authorization and a documented deletion workflow |

All analysis responses use `Cache-Control: private, no-store`. Do not put pasted content in URLs, page titles, public result slugs or error messages. Public static recovery content may be cached independently.

Flow: intake → local secret masking/preview → canonical text and span map → rules/floor → quota-reserved remote inference if requested and permitted → validate model evidence → final policy → approved next action. When AI is unavailable or quota is exhausted, return the deterministic warning and free recovery link, with an honest reduced-analysis state.

## 3. Output contract

Use a Zod schema and generate the model's supported JSON schema from a deliberately simple subset. Keep the internal provider output separate from the public result.

```ts
type Verdict = 'strong_warning' | 'some_warning' | 'cannot_tell';
type RuleFloor = 'none' | 'some_warning' | 'strong_warning';
type EvidenceQuality = 'adequate' | 'partial' | 'unreadable';

interface AnalysisResult {
  verdict: Verdict;
  ruleFloor: RuleFloor;
  evidenceQuality: EvidenceQuality;
  analysisMode: 'mock_demo' | 'rules_only' | 'rules_and_ai';
  reasons: Array<{
    id: string;
    startUtf16: number;
    endUtf16: number;
    excerpt: string;
    explanation: string;
    source: 'rule' | 'model';
  }>;
  approvedActionIds: string[];
  missingContext: string[];
  versions: { rules: string; prompt: string; model: string; playbook: string };
}
```

Validate each excerpt against the displayed redacted canonical text. Do not highlight a guessed original screenshot region. Reject unknown actions and contact fields. A strong rule stays strong even if the model says it cannot tell. A model that sees no warning never creates a safe verdict. Exposure urgency is a separate recovery concern and must not be confused with classification confidence.

Start with one non-streamed model request, bounded input/output, a short timeout and at most one carefully controlled transient retry. Do not use tools, search grounding, browsing, function execution or model-generated URLs. A provider adapter lets the model change after evaluation without rewriting policy or billing.

## 4. Storage: accounts are durable; checks are transient

| Entity | Allowed storage | Access and protections |
|---|---|---|
| `profiles` | Auth ID, locale, consent versions and account preferences | Owner read/update allowed fields; no role/plan assignment from client |
| `plans` | Versioned server-owned price, duration and quota configuration | Public read of published offer only; privileged writes |
| `orders` | User ID, immutable plan version, amount in integer minor units, BDT currency, provider reference and status | User can view own order; server creates/changes status |
| `payment_events` | Provider event identity/status, verification outcome and minimal reference | Restricted server writes; deduplicated; never store full callback bodies containing unnecessary PII |
| `entitlements` | Subject, plan version, starts/expires timestamps and grant origin | Subject read; authorized server-only grant/revoke |
| `usage_periods` | Subject, window, quota, consumed/reserved counts | Atomic update; no direct client mutations |
| `analysis_requests` | Random request ID, subject/idempotency binding, reservation state, cost/latency and version IDs | No message, result, OCR text or full URL. Short documented retention |
| `guest_limits` | Signed random guest subject plus rotating keyed IP-derived throttle identifiers | Raw IP not persisted. Hashing does not create anonymity; use short TTL and legal/privacy review |
| `playbooks` / source files | Reviewed actions, official URLs, sources, timestamps and expiry | Read-only runtime content; code/reviewer controlled updates |
| `privacy_requests` | Request identity, type and completion state | Restricted workflow; billing-record retention exceptions explained |

Keep schema/grants/RLS/functions in migrations and generate matching TypeScript types. User-scoped operations use a verified session and owner policies. Server secret clients bypass RLS, so tightly isolate them and explicitly authorize their requests. Clients must never be able to grant themselves plan access or credits. [D6–D8]

Proposed retention defaults, to review before production: usage/diagnostic request records 30 days; guest throttle identifiers at most 48 hours; original inputs/results browser memory only until reset/tab closure. Legal/accounting requirements determine payment-record retention separately. Do not promise deletion of every financial record immediately if retention is legally required.

## 5. Atomic quotas and prepaid access

Treat these as **experiment configuration**, not implemented offers:

- Free: three remote text checks per Bangladesh calendar day; deterministic warnings and recovery content remain available.
- Pro: BDT 99 for 30 days, an initial experiment of 120 usage units; text costs one unit, screenshot-derived check three.
- Family later: BDT 199 for 30 days, five consenting members sharing a proposed 400-unit pool. No automatic sharing of check content.

All prices/quotas above are founder hypotheses. Adjust before public sales using measured provider cost, OCR usability, support load and real purchase behavior. Do not implement family membership in the first paid release simply because a proposed plan is listed.

Use an atomic Postgres function/transaction to reserve units before inference. It must authorize subject, enforce limits, deduplicate request identity, and prevent concurrent overspend. A validated successful server-side completion settles the usage record once; invalid output/timeout releases the reservation. Network delivery itself cannot be guaranteed exactly once. A disconnected client can inspect minimal request status, but the server cannot retrieve an unstored result. Do not rerun paid inference automatically; explain the policy, provide a metadata-based credit-support path and tell the user when a new check is needed. Keep request state rather than result content.

Use a per-subject idempotency key and a short-lived keyed input fingerprint only to detect mismatched retries; do not store content or a reusable unkeyed message hash. Expired reservations are reconciled through a bounded maintenance operation. Security-definer SQL functions require restricted execution grants, a fixed search path and explicit subject checks.

Daily free windows reset at midnight `Asia/Dhaka`; timestamps are stored in UTC and UI shows the local reset. Prepaid periods are exact server-owned start/end instants. Specify renewal behavior: extend from the later of the current expiry and verified purchase time, once per verified order, unless the displayed offer explicitly says otherwise. Test timezone edges and overlapping purchases.

Payment workflow: server creates immutable order → approved hosted checkout → provider notification or user-initiated verification → server-to-server status check → match order/user/amount/currency → idempotent entitlement grant. Reconcile pending orders and handle refunds/chargebacks with a status transition table so late success cannot overwrite a previously verified reversal. No real plan grant from a demo “Pay” button.

## 6. Screenshot choice and privacy tradeoff

First image implementation: load a local image into a browser worker, OCR with trusted bundled Bengali/English assets, show editable extracted text, offer masking, and send only the reviewed redacted text. Original images, EXIF and filenames are not submitted to the application backend. This avoids a server upload/storage pipeline while still requiring clear remote text-processing disclosure.

Proposed limits: one JPEG/PNG/WebP up to 5 MB; resize long edge to 2,048 pixels; no PDFs, SVG, animated images, APKs or remote image URLs. Test actual decoding/runtime support. Cap OCR work/time and allow cancellation; slow/failed OCR offers paste/edit fallback, not fabricated text. Large OCR assets should load only when needed and be served from trusted app-controlled paths with reviewed licenses. [D12]

Test Bengali conjuncts, mixed digits, dark screenshots, blur and cropped sender context on target phones. Add cloud vision only if measured OCR failures justify it and appropriate redaction, image-processing consent, vendor retention and legal review are complete. Do not quietly switch local OCR into image upload.

## 7. Codex Cloud and CI setup

### No-credential default

The first tasks use `APP_MODE=development`, `AI_PROVIDER=mock`, `AUTH_MODE=mock`, `DATA_BACKEND=memory`, `BILLING_PROVIDER=disabled`, and `ENABLE_PUBLIC_AI=false`. These are server-controlled modes, never browser-selectable. The UI prominently says demo/mock analysis. No fake account session may work in a real production deployment. Enable public remote AI only after distributed abuse controls, quotas and spend caps pass integration checks; opt-in private adapter smoke tests do not enable public traffic.

Mock adapters must be deterministic and preserve the same typed interfaces. They prove UX and policy wiring, not real detection quality, RLS correctness or provider integration. Database and live-provider tests are opt-in, against test resources only.

### Current Cloud environments

Official OpenAI documentation currently describes creating an environment, letting Codex inspect/install/test dependencies, reviewing the setup report and publishing it. Install scripts prepare dependencies; start instructions start/check services. New tasks use the published setup; republish after material setup changes. [D1]

After bootstrap, configure installation to:

```bash
pnpm install --frozen-lockfile
pnpm exec playwright install --with-deps chromium
```

The package manager must already match package.json's pinned version; include the environment's verified installation step if it does not. Do not depend on globally installed tools or assume Corepack is present. The first bootstrap creates the lockfile; only subsequent installs use `--frozen-lockfile`.

If browser/system downloads are unavailable, use a prepared environment image or a CI browser-install step and report the limitation. Do not call an unrun browser suite passed. The default application/test build must not fetch Google Fonts, OCR assets, live AI or a production database; bundle approved assets during setup and self-host fonts.

For ordinary tasks, allow package-manager destinations and only other explicitly needed documentation/download hosts. No internet access is needed at runtime for the mock checker. A domain permitted in environment settings is not permission to visit suspect URLs in the application.

### Optional live integration testing

Current Cloud environments distinguish directly supplied variables from HTTP/network secrets whose placeholders are substituted for allowed HTTPS destinations. A network secret name must not collide with a separately configured direct variable. Use limited test credentials and exact provider domains; verify SDK compatibility rather than assuming proxy substitution works. [D1]

Never include production payment access or customer data in routine tasks. Prefer no live key until the provider adapter is ready. For Gemini tests, supply the credential through the supported secure mechanism, permit only the official API host and run synthetic examples with strict spend limits. Runtime production keys belong in hosting/provider secret settings, not GitHub source files.

### Legacy Cloud environment interface

If your account shows the older setup/maintenance-script environment, its documentation says encrypted secrets are available only during setup and removed before the agent phase. Do not copy them into `.env.local` or cached files to bypass that boundary. Keep default tasks mocked and use an approved test environment/configuration for any runtime integration checks. Follow the interface/documentation you actually have. [D2]

### CI command contract

Task 1 implements lint, typecheck, unit/component tests, `eval:smoke`, browser tests and build scripts. `pnpm verify` runs deterministic checks and production compilation with mock configuration; `pnpm test:e2e` starts the app through Playwright's webServer configuration. `pnpm test:db` runs a separately provisioned real database suite. GitHub Actions installs with the committed lockfile and uses no production secrets for untrusted PRs.

Use ephemeral test auth/DB projects or local Supabase for actual RLS and accounting checks. Do not run migrations against production from a PR. Avoid a global Docker dependency for the default coding task.

## 8. Production gates and practical order

1. **Text beta:** rules, explanations, recovery, mock UI and deterministic regressions.
2. **Real AI:** server adapter, semantically validated evidence, latency/cost measurements and hard spend caps.
3. **Private account beta:** Supabase/RLS, email delivery, atomic quota reservations, Turnstile, privacy controls.
4. **Screenshot beta:** local OCR, review/redaction and mobile usability, with no original-image uploads.
5. **First paid cohort:** approved hosted checkout, prepaid access, reconciled entitlements, support/refund process and actual purchase evidence.
6. **Retention work:** improve event-triggered usefulness; test family convenience only after individual users return/pay. Do not measure an episodic protection product solely by daily active use.

Production configuration must reject mock auth/data and fake payment grants, enforce HTTPS and approved origins, use suitable model terms and commercial hosting, and pass actual cross-user and payment/usage integration tests. A free public beta may leave billing disabled; that does not waive AI-abuse and privacy requirements.

No platform pricing is promised here. Budget for commercial hosting, managed database/backups, suitable AI processing, domain/email, payment onboarding/fees and human support. Confirm current quotes before purchase. “Free tier” is not a guarantee of commercial permission, privacy suitability, quota or reliability.

## 9. Official references

Checked 4 November 2026. References establish documented capabilities, not that this kit implements them.

| ID | Documentation |
|---|---|
| D1 | [OpenAI: current Cloud environments](https://learn.chatgpt.com/docs/environments/cloud-environments) and [AGENTS.md discovery](https://learn.chatgpt.com/docs/agent-configuration/agents-md) |
| D2 | [OpenAI: legacy Cloud environment behavior](https://developers.openai.com/codex/environments/cloud-environment) |
| D3 | [Node.js release support](https://nodejs.org/en/about/previous-releases) |
| D4 | [Next.js installation and commands](https://nextjs.org/docs/app/getting-started/installation) |
| D5 | [next-intl App Router setup](https://next-intl.dev/docs/getting-started/app-router) |
| D6 | [Supabase server-side Next.js auth](https://supabase.com/docs/guides/auth/server-side/nextjs) |
| D7 | [Supabase API keys and privileges](https://supabase.com/docs/guides/api/api-keys) |
| D8 | [Supabase RLS and grants](https://supabase.com/docs/guides/database/postgres/row-level-security) |
| D9 | [Supabase custom SMTP requirements](https://supabase.com/docs/guides/auth/auth-smtp) |
| D10 | [Gemini structured output](https://ai.google.dev/gemini-api/docs/generate-content/structured-output) |
| D11 | [Google GenAI JavaScript SDK](https://github.com/googleapis/js-genai) and [Gemini pricing/model information](https://ai.google.dev/gemini-api/docs/pricing) |
| D12 | [Tesseract.js official project](https://github.com/naptha/tesseract.js) |
| D13 | [Turnstile server-side verification](https://developers.cloudflare.com/turnstile/get-started/server-side-validation/) |
| D14 | [SSLCOMMERZ public pricing](https://sslcommerz.com/pricing/) and [developer documentation](https://developer.sslcommerz.com/doc/v4/) — verify merchant-specific terms before use |
| D15 | [bKash business features](https://www.bkash.com/en/business) — capability listing is not merchant approval |
| D16 | [Vercel Hobby commercial-use restriction](https://vercel.com/docs/plans/hobby) |

Bangladesh legal applicability, data-processing/vendor terms, commercial data rights, merchant eligibility and refund/tax obligations require qualified review before production. This technical plan makes no legal-compliance certification.
