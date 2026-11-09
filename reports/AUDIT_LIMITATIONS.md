# Audit Limitations

This engagement examined everything that could be reached from the authorised, read-only position described below. The conclusions in every other report are bounded by these limits. Where a limit prevents a question from being answered, the executive summary says so with the label **UNABLE TO DETERMINE** rather than implying safety.

## 1. What was fully in scope and fully examined

| Area | Coverage |
|---|---|
| Repository discovery | All 33 repositories reported by GitHub for the account (30 public, 3 private). Confirmed against the authenticated `user` endpoint (30 public repos) and the search API (33 total). |
| Git content | Complete history of all 33 repositories: every branch, tag, pull-request ref and reachable commit, captured as bare mirrors. 13,584 unique blobs. Zero shallow clones. Zero unreachable objects. |
| Secret scanning | gitleaks, trufflehog (bare mode), detect-secrets, plus four independent custom sweeps over every text blob in history. Every hit was manually dispositioned. |
| Personal-data scanning | Regex sweeps over every text blob, manual reading of every data-bearing file (CSV, JSON seeds, SQL, device dumps, docs, test fixtures), EXIF/XMP inspection of all 2,146 image blobs, visual inspection of representative screenshots, container inspection of misnamed media. |
| Claude Code artefacts | Filename and content sweeps of every blob for settings files, permission allow-lists, MCP configuration and inline-secret command shapes. |
| Application code | Static review of the 17 repositories that are applications with servers, auth, databases or deployments; dependency audit against extracted lockfiles. |
| GitHub metadata | Repository visibility, archive and fork status, default branch, Pages flag, homepage, creation and push dates for all 33; PR and issue lists via the search API; Actions workflows and runs for the four attached repositories. |

## 2. What could not be examined, and why

### 2.1 The owner's local development environment (not accessible)
This session ran in an ephemeral cloud container. The `~/.claude` directory, `~/.claude.json`, project-level `.claude/settings.local.json` files, shell history, `.env` files, `~/.vercel`, `~/.supabase`, `~/.config/gh`, `~/.aws`, SSH keys and browser state on the owner's own machines were **not** reachable and were **not** examined. The Claude Code "don't ask again" persistence issue is primarily a local-machine issue. The GitHub footprint shows it did not reach any repository; whether a local `settings.local.json` holds a secret is **UNABLE TO DETERMINE** from here. A safe self-check procedure is provided in `CLAUDE_CODE_EXPOSURE.md`.

### 2.2 Claude Code web session transcripts (not accessible)
Most work in these repositories was done through Claude Code on the web. Any secret pasted into a prompt or printed by a command in those sessions would be in the session transcript held by the service, not in git. This audit cannot see those transcripts. The committed documentation shows which credentials were handled in sessions (see `CLAUDE_CODE_EXPOSURE.md`); their exposure through transcripts is **POTENTIAL EXPOSURE — REQUIRES VERIFICATION** and is addressed by precautionary rotation in the remediation plan.

### 2.3 GitHub repository-level API for 29 of 33 repositories (blocked by session policy)
The session's GitHub access was scoped to the Security repository plus the three private repositories attached during the engagement. For the other 29 repositories, these endpoints returned "not enabled for this session" or "not permitted through this proxy":
- pull request bodies and review comments (only titles/bodies via the search API were readable, and both PRs were checked that way);
- Actions run logs and job logs (only Jezza-Secret-Mission, Colony-Go, TV, The-Hidden-World and fruit have workflows; fruit's runs were checked; the others' logs were not);
- collaborators, deploy keys, webhooks, Actions secrets names, branch protection, GitHub secret-scanning alerts, Dependabot alerts;
- the Events/Activity feed that would show force-pushes.
Even for the four attached repositories, collaborator, deploy-key, webhook, secret-scanning and Actions-secret endpoints were blocked by the proxy. Only `security_and_analysis` on the Security repository was readable (secret scanning and push protection enabled).

### 2.4 Live systems (deliberately not touched)
No request was sent to any deployed application, Supabase project, Vercel deployment, Cloud Run service, Fly app, GitHub Pages site or third-party API, and no discovered value was tested for validity. Consequently the following are **UNABLE TO DETERMINE** from this engagement and must be checked by the owner in the relevant dashboards:
- whether any environment variable or GitHub Actions secret named in the code is actually configured, and whether it is still valid;
- whether Supabase Row Level Security behaves at runtime as the committed SQL intends;
- whether the private `intelligence` project's public Vercel URL is reachable without protection, and whether the Reputation-Operations workspace passcode is set in production;
- whether the Jezza party server is still running and still holds photos;
- which branch of Hold-My-Loot is deployed to its custom domain;
- Google Cloud IAM bindings for the Jezza deploy service account.

### 2.5 Account security state (not accessible)
MFA status, passkeys, recovery options, active sessions, OAuth application grants, personal access tokens and sign-in logs for GitHub, Google, Anthropic, Vercel, Supabase, Google Cloud, Fly.io, Cloudflare and PostHog were not visible. Evidence of misuse of any credential therefore cannot be assessed here; the remediation plan tells the owner where to look.

### 2.6 Unreachable or force-pushed history
A mirror clone retrieves only objects reachable from refs on GitHub. Commits that were force-pushed away before 2026-11-09 are not in the evidence store. No indicator of force-pushing was found (zero unreachable objects, linear histories), but absence cannot be proven from a clone.

### 2.7 Binary content
Video files (including The-Hidden-World's 2 GB of renders), audio, fonts and archives were not opened for embedded text. Images were inspected for metadata and a sample was viewed; not every one of the 2,146 images was looked at by eye. PDFs committed as fixtures were parsed and found synthetic.

### 2.8 Pattern-based detection
Regex and entropy scanning cannot prove the absence of a secret in an unanticipated format. Four independent sweeps with different pattern sets, plus manual reading of every configuration, deployment, SQL, test-fixture and documentation file, reduce but do not eliminate this gap.

### 2.9 Identity inferences
No authoritative identity document exists in the repositories. Statements about the owner's probable first name, city, devices and social circle are inferences from nicknames, timezones and device metadata, and are described at that confidence in `PERSONAL_INFORMATION_EXPOSURE.md`.

## 3. Tooling notes that affect interpretation

- The first trufflehog run (without `--bare`) scanned zero chunks and reported zero findings; it was discarded and re-run. Reports from any earlier scan of these repositories that relied on trufflehog in that mode should be treated as having found nothing because they looked at nothing.
- detect-secrets was run on HEAD trees only; history coverage came from the blob-level sweeps.
- gitleaks redacted 90 percent of each matched value in its output; the remaining prefixes were used only to classify hits.

## 4. What this means for the conclusions

- "NOT IDENTIFIED IN THE AUDITED SCOPE" for secrets in git is a strong result: the scope is complete for git content and the methods overlap.
- Statements about the local machine, session transcripts, account security and live systems are explicitly **not** covered and are labelled UNABLE TO DETERMINE or POTENTIAL EXPOSURE — REQUIRES VERIFICATION. The owner should complete the self-checks in `CLAUDE_CODE_EXPOSURE.md` and the dashboard checks in `IMMEDIATE_REMEDIATION_PLAN.md` before treating this engagement as a full clean bill of health.
