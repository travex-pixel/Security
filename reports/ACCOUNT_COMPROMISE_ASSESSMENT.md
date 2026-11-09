# Account Compromise Assessment

**Question:** could exposed information enable access to the owner's GitHub, Anthropic, Vercel, Supabase, email, cloud, database or payment accounts, and is there evidence that any such access has occurred?

## 1. The four distinctions this report keeps separate

| Term | Meaning | Status for this account |
|---|---|---|
| Credential exposure | a usable secret is present where an unauthorised party could read it | **NOT IDENTIFIED IN THE AUDITED SCOPE** for git content. **POTENTIAL EXPOSURE — REQUIRES VERIFICATION** for credentials handled in Claude Code sessions and on the local machine (see `CLAUDE_CODE_EXPOSURE.md`). |
| Credential validity | whether an exposed secret still works | nothing was found to test; no live validation was performed by rule |
| Unauthorised access | evidence that someone used a credential they should not have | **UNABLE TO DETERMINE** from this engagement; account sign-in and audit logs were not accessible |
| Confirmed compromise | an account is under someone else's control | **no evidence of any kind** was found; this is not the same as proof of absence |

No statement in this report should be read as "an account has been compromised". No evidence supports that.

## 2. What an outsider can learn from the public repositories

An outsider reading the 30 public repositories learns the integration map, not the keys:

- Hosting: Next.js on Vercel (seven projects, several with live Vercel hostnames), static games on GitHub Pages (eight sites) and a custom domain, a Node/WebSocket game server on Fly.io and Google Cloud Run (service name, region, bucket name pattern, and the Cloud project id via a bucket name in a README), Cloudflare Pages for a TV remote page.
- Data: Supabase for several apps (no project refs committed; only placeholders), Upstash/Vercel KV for one, a Postgres pooler for one.
- Third-party APIs: Anthropic (server-side in five apps), Google Maps Places, EODHD market data, ElevenLabs, YouTube Data API via OAuth, Stripe (integrated in code for one product), PostHog analytics.
- CI: which GitHub Actions secrets exist by name (`GCP_SA_KEY`, `GCP_PROJECT_ID`, `YT_CLIENT_ID`, `YT_CLIENT_SECRET`, `YT_REFRESH_TOKEN`, `ELEVENLABS_API_KEY`, `VOICE_VAULT_TOKEN`, `VOICE_VAULT_REPO`), and that one deploy workflow runs on any branch push with a broadly privileged cloud key.
- Identity hints: Sydney, specific phone and TV models, friend nicknames, a probable first name, a probable employer. No email address, no password, no token.

This is a precise target list for phishing and a runbook for what to do after a takeover. It is not, by itself, a way in.

## 3. Realistic attack chains, ranked

| Rank | Chain | Preconditions | Impact if realised | Likelihood | Evidence pointer | Status |
|---|---|---|---|---|---|---|
| 1 | **Phish or hijack the single Google identity** that plausibly owns GitHub, Vercel, Supabase, Google Cloud, YouTube, Fly and Cloudflare, then use the public runbook to reach every platform secret | successful phishing, SIM swap or session theft of one mailbox; weak MFA | estate-wide: push code that auto-deploys, read every Vercel env var, control the cloud project, rewrite repositories | ordinary phishing risk; cannot be measured from here | service map in `APPLICATION_SECURITY_REVIEW.md`; `CREDENTIAL_EXPOSURE_REPORT.md` CR-04 | **structural risk, MEDIUM severity on the engagement scale, HIGH impact if realised**; MFA state UNABLE TO DETERMINE |
| 2 | **Repository write or build-dependency compromise, then abuse of the Cloud Run deploy key** | ability to land a commit on any branch of Jezza-Secret-Mission, or a compromised npm dependency in its build | execution with a service account documented as Storage Admin, Cloud Run Admin, Artifact Registry Admin, Service Account User: control of the cloud project, including the bucket holding party photos | requires prior GitHub access or supply-chain event | Jezza `.github/workflows/deploy-cloudrun.yml` | MEDIUM |
| 3 | **Denial-of-wallet through unauthenticated LLM proxy routes** | the Anthropic key is configured in production; attacker scripts requests past per-instance rate limits | run up API spend; one route allows 16,000 output tokens per call on the most expensive model | easy if routes are live | hello `/api/chat`, Signal `/api/ask`, Trading `/api/ai/review`, Reputation-Operations server actions | MEDIUM if keys are live |
| 4 | **Forge the Reputation-Operations workspace gate** | brute-force a 32-bit unkeyed token or the passcode (no rate limit, not constant-time) | open the founder's workspace; data is browser-local, so payoff is low, but the gate also fronts paid API actions | feasible | `src/proxy.ts`, `workspace/unlock/actions.ts` | LOW |
| 5 | **Enumerate Jezza party codes** (4 characters, `Math.random`, unauthenticated, unthrottled) and read party content and photo URLs of real people | server is running | exposure of selfies and face crops of identifiable people | feasible in under an hour; one real code is published in a committed screenshot | `server/index.ts:135-190`, `shared/engine.ts:196,214,475`, `docs/screenshots/live/host-party.png` | MEDIUM (privacy impact, while the server runs) |
| 6 | **Read or overwrite any Tyrants game room** through anon Supabase policies with `using (true)` | anon key is public by design | list every room code and snapshot, overwrite rooms, fill the free-tier database | trivial | `supabase/migrations/0001_rooms.sql:22-31` | MEDIUM (game data only) |
| 7 | **Home-network reconnaissance using the TV dumps** | attacker already on the owner's LAN | ADB over TCP on an unpatched Android 9 TV is a ready target | requires physical or Wi-Fi proximity | TV `backup/`, `discovery/` | LOW |

Chains checked and found **not** to exist: `pull_request_target` secret exfiltration (no such trigger anywhere); self-hosted runners (none); a service-role or LLM key in any client bundle (none); hardcoded Supabase or anon keys (none); weak RLS on Coverwise or Hold-My-Loot (both are well designed); committed `.env`, key, cookie or credential-store files (none).

## 4. Credential-reuse and identity-concentration risk

The repositories never reveal the owner's email address or any password, so password reuse cannot be assessed from the evidence and is not asserted. What the evidence does show is that one person and one identity are the only control across every platform, and that public documentation names every integration. The consequence of that identity being phished is estate-wide. The protections that matter are phishing-resistant MFA (hardware key or passkey) on the Google account and GitHub, removal of SMS recovery, a review of OAuth application grants and personal access tokens, and branch protection so that a stolen session cannot silently deploy.

## 5. Privilege escalation inside the applications

- No server-trusted client role or JWT-claim flaw was found. Client-side `isHost` or `role` flags in the games govern only local or casual behaviour.
- Jezza host takeover is gated by a hashed, rate-limited PIN; export tokens are 24 bytes from a cryptographic generator with a 15-minute life. Good.
- The private `intelligence` demo has no authentication at all and self-asserted `recordedBy` identities; its data is fictional and the app says access classifications are display-only. This becomes a real problem only if real data is loaded.

## 6. Evidence of misuse

| Source that would show misuse | Available to this engagement | Result |
|---|---|---|
| GitHub security log, sign-in history, PAT and OAuth grant lists | no | UNABLE TO DETERMINE |
| Google account activity, recovery changes | no | UNABLE TO DETERMINE |
| Anthropic console usage and spend | no | UNABLE TO DETERMINE |
| Vercel audit log, deployment list | no | UNABLE TO DETERMINE |
| Supabase auth and database logs | no | UNABLE TO DETERMINE |
| Google Cloud audit logs, IAM bindings | no | UNABLE TO DETERMINE |
| Repository commit history for unexpected authors or branches | yes | no unexpected authors; every commit attributable to the owner, the Claude cloud committer, the GitHub web UI, a GitHub Actions bot or Vercel |
| Repository forks, stars, watchers | yes | zero forks and zero stars on all 33 repositories (low attention so far) |
| Pull requests or issues from strangers | yes | none; both PRs are the owner's |

The repository-side indicators are clean. The account-side indicators must be checked by the owner; the remediation plan lists where.

## 7. What could not be determined without touching live systems

- Whether the Anthropic, Stripe, Supabase service-role, Google Cloud service-account, YouTube OAuth and Vercel credentials are configured in production and still valid.
- Whether Row Level Security behaves at runtime as the committed SQL intends.
- Whether the Jezza server, the private `intelligence` deployment and the Reputation-Operations workspace are publicly reachable and protected.
- Current Google Cloud IAM bindings for the deploy service account.
- Whether the owner's local machine holds any secret in Claude Code settings, transcripts or shell history.

Each of these is a dashboard check the owner can make without any risk; the steps are in `IMMEDIATE_REMEDIATION_PLAN.md`.

## 8. Answers

| Question | Answer |
|---|---|
| Is there evidence that any exposed credential has been misused? | UNABLE TO DETERMINE (no credential was found to misuse; account logs not accessible; repository-side indicators clean) |
| Could someone realistically access any of the owner's accounts or applications? | POTENTIAL EXPOSURE — REQUIRES VERIFICATION. Not from anything in the repositories. The realistic paths are phishing of the single identity (unmeasurable here) and the application weaknesses above (party-code enumeration, Tyrants room policies, unauthenticated LLM routes), which expose game data, photos and API spend rather than accounts. |
| Has any account been compromised? | No evidence found. Not claimed. |
