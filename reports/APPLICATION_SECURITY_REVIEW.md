# Application Security Review

**Scope:** static review of the 17 repositories that are applications with servers, authentication, databases or deployments (Coverwise, hello, hello-deployed, Jezza-Secret-Mission, Hold-My-Loot, Signal, Trading, intelligence, NBA-fantasy-, Reputation-Operations, SleepMax, Thrive, TV, Tyrants, Servo-Empire, Bin-Night, NewApp), plus the service and deployment map across all 33. Static analysis of the HEAD trees in the mirrors; dependency audit against extracted lockfiles. No request was sent to any live system.
**Priority note:** this review is Priority 5 of the engagement. It did not displace the credential and personal-data work, and nothing here is a credential leak.

## 1. Overall picture

The code is unusually careful for prototype work: schema validation on every route, Row Level Security enabled on every Supabase table, service-role keys used only on the server, HMAC-signed cookies where sessions exist, security headers and Content Security Policy on most Next.js apps, honest READMEs about limitations. No hardcoded secret exists in any application. The findings are design-level. **Nothing is rated CRITICAL or HIGH.** The MEDIUM items concern: unauthenticated access to real people's party photos by code enumeration, a disk-fill path, a weak workspace gate fronting paid API actions, wide-open anonymous policies on a game's rooms table, a deploy-on-any-branch workflow holding a broad cloud key, and unauthenticated LLM proxy routes.

## 2. Service and deployment map

| Service | Repositories | Evidence | What a credential for it would unlock |
|---|---|---|---|
| GitHub (Pages, Actions, secrets) | all | workflows in Colony-Go, Jezza, The-Hidden-World, TV, fruit; CNAME in Hold-My-Loot | every site, every Actions secret, push to auto-deploying branches |
| Vercel (hosting, Blob, KV, cron) | Coverwise, Signal, NewApp, Reputation-Operations, Trading, Tyrants, Thrive (alt), hello, intelligence, SleepMax | `vercel.json`, blob and KV tokens read from env, vendored Vercel skills | every server-side env var of every project; redeploy to production |
| Supabase | Coverwise (service role, server only), Tyrants (anon, browser), Hold-My-Loot (anon slot, empty), hello/hello-deployed (schema), Trading (schema, not wired) | `supabase/*.sql`, server modules | service-role key gives full read of insurance applications in Coverwise |
| Google Cloud (Cloud Run, Build, Artifact Registry, Storage) | Jezza | deploy workflow and script | broad project control via the documented service account |
| Fly.io, Render | Jezza | `fly.toml`, `render.yaml` | persistent disk with party state and photos |
| Netlify | Thrive | `netlify.toml`, deploy doc (configured, use unconfirmed) | Stripe secret and licence-signing secret |
| Stripe | Thrive (integrated in code; live use unconfirmed) | server module, checkout and verify routes | refunds, customer data |
| Anthropic API | Signal, Reputation-Operations, Trading, hello, hello-deployed, The-Hidden-World | server-side env reads | spend |
| ElevenLabs, YouTube Data API (OAuth), GitHub fine-grained PAT | The-Hidden-World | workflow secret names | spend; upload to the channel; read a private asset repo |
| PostHog | Hold-My-Loot (live project key), Thrive, NewApp, Servo-Empire, Bin-Night (placeholders) | client code | public project key only |
| Google Maps Places, EODHD, webhook sink | Reputation-Operations, Trading, Reputation-Operations | env names | spend; lead data in transit |
| Android TV over ADB | TV | tooling and dumps | home device, not a cloud credential |

Services mentioned only in planning documents and not wired: Ko-fi, itch.io, Gumroad, Resend, AdMob, app stores.

## 3. Deployed and business-relevant assets

| Asset | Status from repository evidence | Why it matters |
|---|---|---|
| Hold-My-Loot on a custom domain | live; analytics wired; leaderboard prepared on a side branch but switched off on `main` | the one product with real users today |
| Jezza party server on Fly and Cloud Run | configured for a specific past weekend; whether still running is unknown | holds selfies and group photos of identifiable people |
| Coverwise on Vercel | demo mode by default; Supabase optional | collects insurance-application-grade personal data when live |
| Thrive (LodgeReady) | Netlify and Stripe planned, not live per its deploy doc | paid product for a vulnerable user group; processing is in-browser |
| Reputation-Operations (Reputrail) on Vercel | workspace gated by one shared passcode | public repository ships the go-to-market dataset |
| Signal, Trading, NewApp, SleepMax, Tyrants on Vercel | live hostnames recorded in GitHub metadata | unauthenticated AI routes in Signal and Trading |
| intelligence (private) on Vercel | public URL in metadata; no authentication in the app | fictional data; real people's names and photos on the team page |
| TV | tooling for the owner's own TV | public device dumps (see personal-information report) |

## 4. Supabase Row Level Security review

| Repository / file | RLS | Policies | Verdict |
|---|---|---|---|
| Coverwise `supabase/migrations/0001_init.sql` | on, all tables | none; all privileges revoked from anon and authenticated; purge function is security definer with execute revoked | correct and conservative; only the server's service-role key can read applications |
| Hold-My-Loot `supabase/leaderboard.sql` (side branch) | on | anon select limited to a column subset; anon insert validated by a security-definer trigger with shape, plausibility, profanity and rate checks; player ids never readable | good; residual: per-player limits keyed on a client-chosen id |
| Tyrants `supabase/migrations/0001_rooms.sql:22-31` | on | `select using (true)`, `insert with check (true)`, `update using (true) with check (true)` for anon | weak: anyone with the public anon key can list every room code and snapshot, overwrite any room, and insert unbounded snapshot text; contradicts the README's "only people who know the code" model |
| hello `supabase/schema.sql` (historical), hello-deployed (HEAD) | on, 8 tables | owner-scoped policies on four tables; none on the rest (deny by default) | acceptable sketch, marked not applied; the app connects as database owner server-side |
| Trading `supabase/schema.sql` | on | `auth.uid() = user_id` | correct, explicitly not wired |
| Service-role key in client code anywhere? | | never with a `NEXT_PUBLIC_` or `VITE_` prefix in any ref | clean |
| Real project URLs committed? | | placeholders only | clean |

## 5. Findings

| Rank | ID | Application | Finding | Severity | Confidence |
|---|---|---|---|---|---|
| 1 | APP-01 | Jezza-Secret-Mission | `GET /api/party/:code/tv` and `/exists` are unauthenticated and unthrottled (`server/index.ts:135-190`); party codes are 4 characters from a 32-symbol alphabet generated with `Math.random` (`shared/engine.ts:196,214,475`); `/tv` returns every player's nickname, face crop and up to 400 photo URLs, and `/photos/*` is public static. About one million codes are enumerable in under an hour against a service deployed with unauthenticated access. Contradicts the privacy page. | MEDIUM | High |
| 2 | APP-02 | Jezza-Secret-Mission | Disk-fill: anyone can mint an identity, create a party and post roughly 1 MB photos; only a per-socket 60 messages per 10 s limit; `cellar.slice(-200)` (`engine.ts:1147`) evicts entries without deleting files, orphaning them; 1 GB volumes on Fly and Render | MEDIUM | High |
| 3 | APP-03 | Reputation-Operations | Workspace gate cookie is an unkeyed 32-bit FNV hash of the passcode plus its length (`src/proxy.ts:31-37`): deterministic, never expires, leaks length; unlock compare un-rate-limited and not constant-time (`workspace/unlock/actions.ts:14-15`) | MEDIUM | High |
| 4 | APP-04 | Reputation-Operations | Anthropic and Google Places server actions (`workspace/actions.ts:22-49`, `lib/places/lookup.ts:43-70`) sit outside the proxy matcher and have no internal auth or limits: unauthenticated spend. Needs runtime confirmation of action dispatch behaviour. | MEDIUM | Medium |
| 5 | APP-05 | Tyrants | Supabase `rooms` policies `using (true)` for anon select, insert and update (see section 4) | MEDIUM | High |
| 6 | APP-06 | Jezza-Secret-Mission | Cloud Run deploy on push to any branch with a long-lived service-account JSON key; no `permissions:` block; mutable action tags; Dockerfile runs as root | MEDIUM | High |
| 7 | APP-07 | hello, hello-deployed, Signal, Trading | Unauthenticated LLM proxy routes with per-instance in-memory IP limits (hello 20/min, Signal 12/min, Trading 10/min with up to 16,000 output tokens per call on the most expensive model). Per-instance limits multiply across serverless instances and reset on cold start. | LOW to MEDIUM | High |
| 8 | APP-08 | several | Vulnerable production dependencies: `body-parser` below 1.20.9 (Jezza, HIGH denial of service), `zod` below 4.6.6 (Signal lockfile, intelligence pinned 4.1.12; HIGH), `image-size` via `pptxgenjs` (intelligence, HIGH), `@vercel/blob` chain (Signal). Dev-only criticals in `vitest`/`tinypool` (Jezza, intelligence). | LOW to MEDIUM | High |
| 9 | APP-09 | Signal | `/api/refresh` accepts a spoofed `vercel-cron/` User-Agent when `CRON_SECRET` is unset (`src/lib/refresh-auth.ts:27-33`); rebuild and overwrite of a saved edition plus model spend | LOW | High |
| 10 | APP-10 | Coverwise | GET and DELETE of a comparison by UUID with no owner binding or rate limit (`api/comparisons/[id]/route.ts`); `/retry` does not validate `id` and forwards a client-supplied application to the automation worker when no record exists | LOW (real in live mode) | High |
| 11 | APP-11 | hello | Postgres TLS with `rejectUnauthorized: false` unless a CA certificate is configured (`db.ts:37-38`) | LOW | High |
| 12 | APP-12 | Hold-My-Loot | Leaderboard per-player rate limit keyed on a client-chosen id; analytics default-on with opt-out on `main`; version skew between `main`'s submit code and the side-branch SQL | LOW | High |
| 13 | APP-13 | Jezza-Secret-Mission | Link-preview origin from `x-forwarded-host` when `PUBLIC_URL` is unset; player secrets in plaintext `state.json` and backups; acknowledged weak PIN hash | LOW | High |
| 14 | APP-14 | intelligence (private) | No authentication; access classifications displayed, not enforced; any visitor can mutate or reset shared demo state with a self-asserted name; exports default to "restricted" sharing. Fictional data; self-documented. | LOW | High |
| 15 | APP-15 | TV | CI workflow with `contents: write` builds an APK and pushes it back to the triggering branch; mutable action tags | LOW | High |
| 16 | APP-16 | hello-deployed (private) | Dev snapshot: no Origin check on people routes, forwarded-for keyed limits, per-process cookie secret when env unset | INFORMATIONAL | High |
| 17 | APP-17 | NewApp | 32-bit group ids with no auth (by design); PostHog persistence with cookies and no consent gate | INFORMATIONAL | High |
| 18 | APP-18 | Hold-My-Loot, Servo-Empire, Bin-Night, NBA-fantasy- | Heavy `innerHTML` use in static games; the one externally sourced value is stripped of `<` and server-constrained | INFORMATIONAL | High |
| 19 | APP-19 | Jezza-Secret-Mission | README discloses the Google Cloud project id through a bucket name | INFORMATIONAL | High |

### Positives worth recording
- Coverwise: RLS and service-role isolation are a model implementation; zod on every body; consent gate; security headers.
- hello: Supabase email OTP with database-backed, instance-shared rate limits per IP and per email; participant checks on every conversation action; Origin-based CSRF protection; parameterised SQL.
- Thrive: Stripe checkout with an Origin allowlist derived from environment, session-id validation, refund and dispute checks, HMAC licence tokens with timing-safe verification; all document processing in-browser; cookie-free analytics.
- Jezza: zip export restricted by filename regex (no traversal); export tokens from a cryptographic generator with a 15-minute life; host-only actions enforced server-side; host PIN hashed and rate-limited.
- TV: every interpolated ADB value validated by regex; ADB key directory kept at 0700/0600; keys gitignored and never committed; relay bound to loopback.
- No `pull_request_target`, no self-hosted runners, no untrusted event data in `run:` steps anywhere.

## 6. Cross-cutting observations

- **Rate limiting:** hello's people routes and NewApp use shared stores; the LLM routes, Signal, Trading, Thrive checkout, Coverwise and Jezza HTTP are per-instance or absent. On serverless this is a cost and abuse gap rather than a correctness bug.
- **Client IP derivation:** most routes trust the first `x-forwarded-for` hop. Safe behind Vercel and Netlify; spoofable on Cloud Run, Fly and Render (Jezza, and hello if moved).
- **Secrets handling:** all secrets are read from server-side environment; `NEXT_PUBLIC_*` carries only URLs, analytics project keys and Supabase anon keys; `.env*` is gitignored wherever env handling exists; no hardcoded secret in any application.
- **CI:** only Jezza and TV (and the render factories) have workflows. Issues are scope (any-branch deploy), a long-lived cloud key, missing `permissions:` blocks, `contents: write` with push-back, and mutable action tags.
- **Privacy by design:** PostHog is opt-out (Hold-My-Loot `main`) or unconditional (NewApp); Thrive is the model. Jezza photo retention is indefinite by design and should get a default expiry.

## 7. Recommended fixes (ordered)

1. Jezza: raise party codes to at least 6 characters from a cryptographic generator (the engine already supports `codeLength`); require the player handshake or a host-minted signed token for `/tv`; add an HTTP rate limiter to `/api/party/*`; serve photos behind the same check or via signed URLs; add per-player and per-party photo quotas and delete files on eviction; set a default retention period.
2. Jezza CI: Workload Identity Federation instead of a JSON key; least-privilege service account; deploy from one branch or manual dispatch; `permissions: contents: read`; pin actions by SHA; `USER node` in the Dockerfile; update `express`/`body-parser` and `vitest`.
3. Reputation-Operations: HMAC-SHA256 session token with a server secret and expiry, timing-safe compare, rate-limit the unlock action (or use Vercel deployment protection); check the gate cookie inside each server action.
4. Tyrants: replace the unfiltered select with an RPC `get_room(code)`; add a size check on `snapshot`; require a per-room write token; enable the purge job.
5. LLM routes (hello, Signal, Trading, Reputation-Operations): move limits to a shared store (hello already has a `rate_events` table), add a daily spend cap and provider spend alerts, lower `max_tokens` on Trading's review route.
6. Signal: require `CRON_SECRET` in production (fail closed); re-lock `zod` to a fixed version.
7. Coverwise before live mode: bind records to a signed owner token, validate `id` in `/retry`, rate-limit `/api/comparisons*`, reject client-supplied applications when a store is configured.
8. hello: ship the database CA certificate or fail closed in production; move `/api/chat` limits to the database table.
9. TV CI: publish the APK as a release asset from a gated job instead of pushing to the branch; drop `contents: write`; pin actions.
10. intelligence: Vercel password or SSO protection before any real data; server-side roles on actions.
11. Dependencies: update the production packages named in APP-08.

## 8. Limitations

- Server-action dispatch outside the Reputation-Operations proxy matcher (APP-04) is inferred from framework behaviour, not exercised.
- Whether Jezza is live, which deploy target is in use, the production code length, and whether platform-edge rate limiting exists are unknown.
- Whether `CRON_SECRET`, `DATABASE_CA_CERT`, `WORKSPACE_PASSCODE` and `HELLO_SESSION_SECRET` are set in production is unknown.
- Which branch of Hold-My-Loot is deployed to its domain, and whether the Supabase board is enabled there, is unknown.
- Vercel header trust is assumed from platform documentation.
- No dynamic testing, no dependency reachability analysis, no review of non-HEAD branches beyond the two noted in the Git history report.
