# Sequential Codex Cloud build tasks

Use one task at a time. Finish, inspect the diff and checks, then merge before moving on. These tasks define a B2C product; there is no need to build enterprise tenancy or a public B2B API first. Timings depend on implementation and review, not a guaranteed two-day production launch.

## Task 1: Bootstrap and a usable mock flow

**Paste into Codex:**

```text
Read AGENTS.md, README.md and TECH_STACK.md. Implement Task 1 only.
Inspect existing files first. Scaffold the application safely in a temporary
directory if create-next-app cannot run in this non-empty repository, then
bring in needed source/config files while preserving these docs and .git.
Use Next.js App Router, TypeScript, Tailwind, shadcn/ui and next-intl,
Bangla default with English toggle. Pin compatible stable versions and pnpm.
Build landing/paste, result, recovery and honest planned-pricing screens.
Use deterministic mock AI/auth/data and disabled billing. Display demo mode.
Implement the required scripts and a GitHub Actions workflow. No API keys,
external database or Docker may be required for default development/tests.
Run checks, inspect mobile UI, and update README with actual status.
```

**Acceptance:** keyboard/mobile flow works; three supported verdicts; genuine “never share OTP” example; mock state explicit; useful free recovery branches; `pnpm verify` and mock Playwright flow run. Fonts/fixtures are local; build needs no live service. Missing dependencies/checks are reported honestly.

## Task 2: Deterministic analysis and evidence contract

```text
Implement Task 2 only. Build typed normalization, secret masking, source-span
mapping, contextual rules, Zod schemas, non-downgrade policy and reviewed
playbook registry. Rules must distinguish disclosure requests from quoted
warnings and genuine OTP notifications. Unknown context means cannot_tell,
never safe. Keep every suspect URL offline. Add small synthetic fixtures
covering Bangla, Banglish, English, mixed numerals and prompt injection.
Implement eval:smoke; clearly label synthetic benchmark limitations.
```

**Acceptance:** floor cannot be lowered through any output path; correct canonical text highlighting; no model-generated contact; no external request to any pasted domain; input limits enforced; selected recovery steps match exposure. Tests exercise behavior, not just the existence of functions.

## Task 3: Real AI adapter with guarded failure

```text
Implement Task 3 only. Add the official server-side Google GenAI adapter
behind the existing mock interface. Follow the installed SDK's official
structured-output documentation. Configure model ID on the server; use
Zod plus semantic evidence/action validation. Bound token usage, timeout
and retries. Do not stream unvalidated output. Keep default CI mocked and
no-key startup working. Add an opt-in synthetic live smoke command; do not
run it without configured test access and an explicit spend-limited task.
```

**Acceptance:** timeout, malformed output, invented span/contact and downgrade attempts trigger valid fallback. No key in client bundle or logs. Results distinguish mock/rules-only/AI modes. Missing key is an honest disabled-adapter state, not a fake live response. Measure actual latency/token cost when live testing is authorized.

## Task 4: Accounts, RLS, distributed quotas and public-beta hardening

```text
Implement Task 4 only. Add Supabase SQL migrations, minimum grants/RLS,
SSR auth, email magic links, protected account routes, owner isolation,
deletion workflow and server-owned plan configuration. Build atomic usage
reservation/settlement/release with idempotency and bounded expiration.
Add server-verified Turnstile and guest/global AI spend limits. Keep rules
and recovery public when quota or AI is unavailable. No raw check history.
Add real database isolation/concurrency tests in a separate service-backed
suite, while preserving offline mock CI. Document SMTP/redirect setup.
```

**Acceptance:** user A cannot access B's rows; anonymous clients cannot mutate billing/usage; client cannot choose owner/plan; concurrent requests cannot overspend; retries do not double-charge; AI failure releases reserved units; default email restrictions acknowledged; public AI remains disabled until configured distributed protections work. Do not apply migrations to production.

## Task 5: Browser-only screenshot intake

```text
Implement Task 5 only. Add lazy-loaded Tesseract.js with trusted locally
served Bengali/English assets, browser worker cancellation, safe image
limits, editable OCR text and redaction preview. Original image pixels,
filenames and metadata must never be submitted to the backend in this
release. Submit only user-reviewed redacted text through the same analysis
contract. Provide paste/manual correction fallback. Test on modest mobile
viewports and representative non-sensitive screenshots.
```

**Acceptance:** browser network test confirms no screenshot body leaves device; invalid/huge/unsupported files rejected; slow OCR cancel/reset works; mock mode needs no remote OCR download; text highlights align with reviewed text. Do not claim OCR authenticates screenshots or eliminates all PII.

## Task 6: Approved prepaid payments and entitlement accounting

```text
Implement Task 6 only using the approved merchant provider and its current
official sandbox documentation. Default billing stays disabled. Add server
orders, hosted checkout, authenticated provider status verification,
immutable plan/amount/currency matching, idempotent events, entitlement
grant/expiry and refund/reversal transitions. Never grant from a success
redirect or frontend flag. Test duplicate and out-of-order events, wrong
amount/currency, forged callbacks, cancellation and reconciliation.
Keep sandbox/live configuration strictly separate. Do not activate live
payments or modify real entitlements without explicit authorization.
```

**Acceptance:** contract and integration tests prove exactly-once grants; only verified payments extend access; refund/reversal cannot be overwritten by a late success; UI states access expiry/user-initiated renewal accurately; support/refund flow exists. Merchant approval and a verified end-to-end live test are launch gates, not assumptions.

## Task 7: Evaluation and a small paid cohort

```text
Implement Task 7 only. Create a reproducible evaluation report from a
campaign-held-out, rights-cleared labeled set. Separate synthetic cases,
genuine warnings/promotions, scams and ambiguous inputs. Report recall,
strong-warning precision, genuine-message false alarms, abstention and
sample limits, by language/input type. Add sanitized operating metrics
and user-visible feedback without content logging. Review production
configuration, budgets, recovery-source freshness, account deletion,
commercial deployment plan and launch checklist. No fabricated traction.
```

**Acceptance:** exact reproducible counts and failure examples; zero known floor-downgrade/contact-invention/link-fetch paths; actual DB/billing integration tests pass; rollout is limited to an agreed cohort with spend controls. Marketing accuracy claims cannot exceed measured evidence.

## Task 8: Optional family product, after individual validation

```text
Implement only after the founder confirms the family experiment. Add a
paying organizer, consenting invitations, member roles, shared usage pool,
revocation and per-item voluntary redacted sharing. An organizer cannot
read members' checks or monitor their inbox by default. Test membership
authorization, invitation expiry/replay, quota concurrency and immediate
loss of access after revocation. Keep individual privacy boundaries.
```

**Acceptance:** household billing rights and message-viewing rights are separate; no private result leaks; shared quota is not duplicated per member. Do not add contacts upload, SMS interception or automatic family alerts as shortcuts.

## Task-completion response to require

For every task, ask Codex to state:

- Working behavior added and files changed.
- Checks actually run and their results.
- Mocked, skipped or blocked external dependencies.
- Manual setup still needed, with no credentials in the response.
- The next smallest task and any material risk discovered.

Do not use “build the whole SaaS” as the first prompt. The phase boundaries are what keep agent-generated changes understandable and testable.
