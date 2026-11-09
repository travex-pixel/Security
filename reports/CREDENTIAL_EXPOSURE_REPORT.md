# Credential Exposure Report

**Scope:** every blob in every ref of all 33 repositories of `travex-pixel` (30 public, 3 private), plus the two pull requests, the Actions runs reachable through the session's GitHub access, and the committed documentation that describes how credentials are handled.
**Rule observed throughout:** no discovered value was used, tested or sent anywhere. Values are described by prefix and length only.

## 1. Headline

**No genuine, usable secret was found in the current files or the history of any repository.** This conclusion rests on four overlapping sweeps (gitleaks, trufflehog, detect-secrets, and four independently written blob-level pattern sweeps by different reviewers), a removed-line search of every diff, manual reading of every configuration, deployment, SQL, test-fixture and documentation file that could plausibly hold a credential, and an independent auditor's challenge.

What does exist is:
- one analytics write key that is public by design (PostHog project key);
- an inventory of credential **names** that the owner holds on other platforms, which is a target list rather than a leak;
- two CI designs that handle powerful credentials in a way worth hardening;
- documentation that instructs running secrets inline on the command line (placeholders in git; relevant to the Claude Code assessment).

## 2. Scanner results and dispositions

| Scanner / sweep | Raw hits | Disposition after manual review |
|---|---|---|
| gitleaks 8.21.2, all history, 33 repos | 9 | 8 false positives (localStorage key names in four games, a fake JWT in Thrive Playwright tests, a stub private key in a TV unit test); 1 real but public-by-design PostHog project key (CR-01) |
| trufflehog 3.84.1 `--bare`, all history | 35 (all The-Hidden-World, detector YoutubeApiKey, none verified) | all 39-character URL path slugs in research notes; false positives |
| detect-secrets 1.5.0, HEAD trees | about 560 | lockfile hashes (Coverwise), base64 embedded PDFs (VisaFly synthetic fixtures), base32 alphabets for invite codes (hello, Tyrants, Jezza), content hashes (TV), Android GMS config blob, test prose; the PostHog key again |
| Custom blob sweep A (lead), 40+ patterns, every text blob | about 1,500 | 27 "OpenAI key" hits are URL slugs beginning `sk-` in The-Hidden-World; 24 "service_role" hits are the variable name; 2 database URLs are a templated `.env.example`; 1 private key is a test stub; generic assignments are test fixtures, field names and placeholders |
| Custom blob sweep B (credential partner), 44 patterns, 11,107 text blobs | 370 | all placeholders, tests, URL slugs, field names |
| Custom blob sweep C (Claude Code partner), 30 command shapes, 11,079 blobs | 57 | placeholders, `mkdir -p`, variable names, one 25-character test stub |
| Removed-line search (`git log --all -p`, lines beginning `-`, 9 secret shapes) | 0 | nothing was ever committed then removed |
| Filename sweep (8,414 paths ever added) | 2 `.env.example` files | no `.env`, key, token, cookie, settings or credential-store file ever committed |
| GitHub PR bodies (2) | 0 | no credential-shaped strings |
| Fruit Actions job log (sampled) | 0 | only the auto-masked `GITHUB_TOKEN`; the workflow references no repository secrets |

## 3. Findings

### CR-01 — PostHog project API key hardcoded in a public repository
- Repository / location: Hold-My-Loot, `index.html` line 37 (line 12 in early versions), introduced in commit 28e749b (2026-10-09), present at HEAD of both `main` and `ccr-e0e37c62`.
- Category: analytics write key. Value: `phc_` prefix, 47 characters, host `us.i.posthog.com`.
- Accessibility: public repository and shipped to every browser visiting the game's domain. Current and historical.
- Validity: public-by-design. PostHog project keys are intended for client code. No PostHog personal API key (`phx_`) exists anywhere, including commit messages and ref names.
- Consequence: anyone can inject events into the analytics project or burn free-tier quota. Cannot read data or administer the project.
- Severity: **INFORMATIONAL** (public by design; the credential partner rated it LOW, the auditor INFORMATIONAL; signed off as INFORMATIONAL). Confidence: High.
- Action: none required for secrecy. Optionally restrict authorised domains in PostHog. The same file's Supabase slot (`window.HML_BOARD`) is empty in all 23 production versions.

### CR-02 — Deploy workflow uses a long-lived Google Cloud service-account JSON key and runs on push to any branch
- Repository / location: Jezza-Secret-Mission, `.github/workflows/deploy-cloudrun.yml` (at HEAD), `scripts/deploy-cloudrun.sh`, `docs/DEPLOY.md`.
- Category: CI credential handling. No value is committed; the workflow reads `secrets.GCP_SA_KEY` and `secrets.GCP_PROJECT_ID`.
- Detail: the documented service account holds Cloud Run Admin, Cloud Build Editor, Artifact Registry Admin, Storage Admin and Service Account User. The trigger is `push` on `branches: ['**']`, gated only on the secret being present, so any branch push (including agent-generated branches) deploys production with that key. No `permissions:` block; actions pinned by mutable tags; container runs as root.
- Accessibility: public workflow definition; secret value not exposed. Whether the key exists and is still valid: UNABLE TO DETERMINE.
- Consequence if the GitHub account or a build dependency is compromised: broad control of the Google Cloud project (storage bucket holding party photos, Artifact Registry, Cloud Run, impersonation).
- Severity: **MEDIUM** (hardening; no value exposed). Confidence: High.
- Action: switch to Workload Identity Federation (keyless OIDC); scope the service account to least privilege; restrict the trigger to the default branch or `workflow_dispatch`; add `permissions: contents: read`; pin actions by SHA; if the JSON key exists, rotate it as a precaution after the change.

### CR-03 — Private-repository token interpolated into a git clone URL in a workflow
- Repository / location: The-Hidden-World, `.github/workflows/render-episode.yml` (blob 7cc8b405, commit a5fc135, present only on branch `ccr-7a78c02a`).
- Detail: `git clone "https://x-access-token:${VAULT_TOKEN}@github.com/${VAULT_REPO}.git"` where the variable comes from `${{ secrets.VOICE_VAULT_TOKEN }}` via `env:`. The step uses `set +x`, `--quiet`, and deletes the clone's `.git` immediately. GitHub masks the value in logs.
- Consequence: the token sits in process arguments and the environment for the clone's lifetime; a failing clone would print the URL (masked on the log side only).
- Severity: **LOW** (hardening). Confidence: High.
- Action: use `actions/checkout` with `token:` and a read-only fine-grained personal access token for the second repository.

### CR-04 — Inventory of credential names the owner holds outside git (target list, not a leak)
Derived from `process.env`/`os.environ` usage, `.env.example` files, workflow files and `.gitignore` rules. None of these values appears in any repository. This table is the rotation checklist if any platform account or local machine is ever suspected compromised, and it is what a reader of the public repositories learns about which integrations exist.

| Credential name | Repositories | Where it lives |
|---|---|---|
| `ANTHROPIC_API_KEY` | hello, hello-deployed, Trading, Signal, Reputation-Operations, The-Hidden-World (docs) | Vercel env, local `.env.local` |
| `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_URL` | Coverwise (server-only), hello (documented) | Vercel env |
| `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Tyrants, hello, Trading | Vercel env (public by design) |
| `DATABASE_URL` (pooler URL with password) | hello (documented only) | — |
| `EODHD_API_TOKEN` | Trading | Vercel env |
| `CRON_SECRET` | Signal | Vercel env |
| `WORKSPACE_PASSCODE`, `AUDIT_WEBHOOK_URL`, `GOOGLE_MAPS_API_KEY` | Reputation-Operations | Vercel env |
| `STRIPE_SECRET_KEY`, `LICENSE_SIGNING_SECRET` | Thrive | Vercel/Netlify env |
| `KV_REST_API_TOKEN` / Upstash token | NewApp | Vercel integration |
| `BROWSER_WORKFLOW_WORKER_TOKEN` | Coverwise | Vercel env |
| `HELLO_SESSION_SECRET`, `HELLO_FOUNDER_INVITE`, Gmail SMTP app password | hello | env |
| `ELEVENLABS_API_KEY`, `YT_CLIENT_ID`, `YT_CLIENT_SECRET`, `YT_REFRESH_TOKEN`, `VOICE_VAULT_TOKEN`, `VOICE_VAULT_REPO` | The-Hidden-World | GitHub Actions secrets |
| `GCP_SA_KEY`, `GCP_PROJECT_ID` | Jezza-Secret-Mission | GitHub Actions secrets |
| `VERCEL_TOKEN` (`vca_` prefix) | described by vendored skills in Signal and others | local `.env`, `.vercel/` |
| ADB RSA key pair, Android signing keystore | TV | owner's phone or laptop (gitignored, never committed) |
| Private voice-clone profiles | The-Hidden-World | separate private GitHub repository |

Severity: INFORMATIONAL. Confidence: High.

### CR-05 — Live hostnames and project identifiers (public by design)
- A private project's production URL (`projectmemory-omega.vercel.app`) is hardcoded in the private `intelligence` repository and in its GitHub homepage field. No public repository references it.
- Public projects reference their own hosts (a custom domain for Hold-My-Loot, Vercel hosts for Coverwise/Signal/Trading/Tyrants/NewApp/SleepMax/Reputation-Operations, a Fly host and Cloud Run service name for Jezza, GitHub Pages sites). The Jezza README discloses the Google Cloud project id through a bucket name.
- No Vercel project or team ids, Supabase project refs, Firebase ids or service-account emails appear anywhere.
- Severity: INFORMATIONAL. Confidence: High.

### CR-06 — Hardcoded test and placeholder values (recorded so they are not re-reported)
- hello `src/lib/ai/server/chat.test.ts`: a 25-character `sk-ant-` test stub (real keys are about 108 characters).
- Thrive tests: fake JWT `v1.eyJ2IjoxfQ.c2lnbmF0dXJl` (decodes to `{"v":1}` and the word "signature").
- TV `cleanyourtv/tests/test_adb_key_permissions.py`: stub `-----BEGIN PRIVATE KEY-----\nstub`.
- Jezza tests: host PINs `4821`, `777777`, `1111`, `0000` and `secret: '…xxx'` identities. Production host PIN is user-set, hashed and rate-limited.
- Signal vendored skill: `vca_abc123` shown as the bad example.
- `.env.example` files: `sk-ant-...`, `<ref>`, `<password>`, `change-me-32-bytes`, `https://xxxx.supabase.co`, `https://fake.supabase.co`, `phc_REPLACE_WITH_YOUR_PROJECT_KEY`.
- Severity: INFORMATIONAL. Confidence: High.

## 4. Where keys are read, and what reaches the browser

- `ANTHROPIC_API_KEY` is read only in server-side code in hello, hello-deployed, Trading, Signal and Reputation-Operations, and in a CLI in The-Hidden-World. No client component reads it.
- 15 `NEXT_PUBLIC_` / `VITE_` variable names exist across the repositories; all are URLs, analytics project keys, Supabase anon keys or feature flags, which are public by design. None contains SECRET, SERVICE_ROLE, PRIVATE, TOKEN, PASSWORD or API_KEY.
- Supabase service-role keys are used only in server modules (Coverwise `src/lib/supabase/server.ts`; hello connects as database owner through `DATABASE_URL` on the server).
- No build bundles, source maps, Playwright storage-state files, HAR files or trace archives were ever committed. The only `tsbuildinfo` file (Jezza) holds filenames.

## 5. Credentials in test datasets, logs and dumps

- TV device dumps (`backup/`, `discovery/`): contain device identifiers (see `PERSONAL_INFORMATION_EXPOSURE.md`) but no Wi-Fi pre-shared key, no Google account token, no ADB private key and no public IP.
- No `.log`, HAR or trace files with credentials. The Fruit Actions log sampled contains only masked tokens.
- VisaFly and Thrive "documents" are synthetic fixtures generated by code.

## 6. `.gitignore` archaeology (what existed locally but was never committed)

Ignore rules across 19 repositories name `.env*`, `.vercel`, `*.pem`, `tools/adbkey*`, `*.keystore`, `voices/private/`, `data/` and `.data/`. This tells us the owner held, at some time, local environment files, Vercel link files, an ADB key pair, an Android signing keystore and private voice-clone data. None was committed. See `CLAUDE_CODE_EXPOSURE.md` for the gap: no repository ignores `.claude/settings.local.json`.

## 7. Answers to the credential questions

| Question | Answer |
|---|---|
| Passwords in repositories | NOT IDENTIFIED IN THE AUDITED SCOPE |
| API keys or authentication tokens exposed | NOT IDENTIFIED IN THE AUDITED SCOPE, with one public-by-design analytics key noted (CR-01) |
| Secrets in public repositories | NOT IDENTIFIED IN THE AUDITED SCOPE (CR-01 is public by design) |
| Secrets deleted but retained in history | NOT IDENTIFIED IN THE AUDITED SCOPE (removed-line search and blob sweep both empty) |
| Credential validity | not tested by rule; nothing found to test |
| Credentials held on other platforms (CR-04) and possibly pasted into Claude Code sessions | POTENTIAL EXPOSURE — REQUIRES VERIFICATION (see `CLAUDE_CODE_EXPOSURE.md` CC-05) |

## 8. Limitations

- GitHub Actions logs, PR comments and GitHub secret-scanning alerts for 29 of 33 repositories were not readable (session scope).
- Force-pushed, unreachable commits are not in the mirrors.
- Binary blobs were not inspected for embedded text.
- The owner's local machine, Vercel project settings and GitHub Actions secret values were out of reach; whether the named credentials exist or are valid cannot be determined here.
- Pattern-based detection cannot prove the absence of a secret in an unanticipated format.
