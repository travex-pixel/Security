# Immediate Remediation Plan

**Principle:** nothing in this plan has been done for you. This engagement was read-only and made no change to any repository, account or service. Every step below is for the owner to carry out or approve.

**Context that shapes the plan:** no genuine credential was found in any repository or its history. There is therefore no credential whose revocation is mandatory because of this investigation. The plan has three parts: (A) confirmed exposures to remove now, (B) precautionary credential and account actions, and (C) hardening. Within each part, items are ordered by what you can do today with certainty first.

## A. Confirmed exposures to remove now (act today)

### A1. Reputation-Operations: third-party business dossier (public)
1. **What needs protecting:** 50 researched businesses, about seven named individuals, critical commentary and cold-call scripts; the seed JSON compiled into the public client bundle.
2. **Steps:** make the repository private immediately (Settings, Danger Zone). Then remove `research/`, `business/call-brief-*` and `src/lib/data/prospects.seed.json` from history (see D below), recreate the repository or force-push the rewritten history, and keep the research in a private store with named individuals stripped. If the site is deployed, redeploy from the cleaned tree so the workspace chunk no longer contains the records.
3. **Affected service:** GitHub; Vercel deployment.
4. **Access logs:** not applicable for repository content. Public exposure window: 2026-11-02 to the day you act; zero forks and stars reduce the chance of copies but do not exclude search-engine or archive caches. Request removal of cached copies from search engines after the rewrite if a listed business complains.
5. **Other items exposed by the same route:** none beyond the files listed.
6. **History cleanup appropriate:** yes.
7. **Further investigation:** none; counts are in `PERSONAL_INFORMATION_EXPOSURE.md`.

### A2. TV: your television's hardware identifiers and home network details (public)
1. **What needs protecting:** device serial, `android_id`, Bluetooth MAC and room name, LAN layout, installed apps, ADB-over-network enabled.
2. **Steps:** make the repository private or remove `backup/` and `discovery/` and rewrite history (D). Treat the serial and MAC as permanently disclosed (exposure window from 2026-10-23). On the TV, turn off ADB debugging and network debugging when not in use, and apply any available firmware update.
3. **Affected service:** GitHub; your home network.
4. **Access logs:** none available. **Other items:** none. **History cleanup:** yes. **Further investigation:** run the local self-check in `CLAUDE_CODE_EXPOSURE.md` section 7 on the machine you used for the TV work; its transcripts contain the same dumps.

### A3. Jezza-Secret-Mission: party server and photos of real people
1. **What needs protecting:** selfies and group photos of identifiable friends held on the server's volume or bucket; the published party code in `docs/screenshots/live/host-party.png`.
2. **Steps:** confirm in the Fly.io, Render and Google Cloud consoles whether the service is still running; stop it and delete the data volume and bucket contents (the privacy page promises deletion). Replace the screenshot with one showing a test code. If the game will be reused, apply the code-length, authentication and rate-limit fixes in `APPLICATION_SECURITY_REVIEW.md` APP-01 before the next event.
3. **Affected services:** Fly.io, Google Cloud Run and Storage, Render.
4. **Access logs:** check Cloud Run request logs and the bucket's access logs for requests to `/api/party/<code>/tv` and `/photos/` from unexpected sources if the service was left running. **History cleanup:** only for the screenshot if you wish. **Further investigation:** none unless logs show access.

### A4. intelligence: real people presented as the product team on a public deployment URL
1. **What needs protecting:** names, titles and headshots of a real firm's leadership, including one person not on the firm's public page.
2. **Steps:** open the Vercel project and enable Deployment Protection (password or Vercel authentication) or remove the deployment until the firm has agreed to be named. Remove the eleventh person's photo unless they consented. Keep the repository private.
3. **Affected service:** Vercel. **Access logs:** Vercel analytics or logs show whether the page has been visited. **History cleanup:** not required (private repository). **Further investigation:** none.

### A5. Lower-priority public data (do when convenient)
- The-Hidden-World: delete or rewrite the side branch `ccr-67645e21-nwgkaq` (and close the associated pull request refs) if the analytics screenshots and reel data should not be public; strip EXIF from future screenshots.
- Signal `PRODUCT.md`: genericise the named company if that association should not be public.
- Jezza fixtures and README: replace real nicknames and event details with generic ones before further promotion.
- BNA-HVAC and CB-Accounting: add an "unofficial redesign concept" note or make private.

## B. Precautionary credential and account actions (this week)

No credential was found exposed, so none of these is mandatory because of a leak. They are justified because (1) the committed documentation shows these credentials were handled in shell commands or Claude Code sessions whose transcripts this audit could not see, (2) one identity controls every platform and its protection state is unknown, and (3) the cost of rotation is low.

### B1. Protect the root identity first
- Google account: enable passkeys or a hardware security key; remove SMS as a recovery or second factor; review Security activity, connected apps and app passwords (a Gmail SMTP app password is referenced in the hello documentation).
- GitHub: enable two-factor with a hardware key or passkey; review Settings, Sessions; review Personal access tokens and Authorized OAuth and GitHub Apps; review Security log for unexpected events; enable vigilant mode for commit signing if desired.
- Vercel, Supabase, Google Cloud, Fly.io, Cloudflare, PostHog, Anthropic: confirm each uses the hardened identity or has its own MFA; review team members and tokens.

### B2. Review where secrets actually live, then rotate in this order
Do this in each dashboard. If a secret is present and you cannot be certain it was never pasted into a prompt or shell command approved with "don't ask again", rotate it.

| Priority | Credential | Where to look | Why it is first |
|---|---|---|---|
| 1 | Google Cloud service-account JSON key (`GCP_SA_KEY`) | Google Cloud IAM, Service accounts, Keys; GitHub repository secrets for Jezza | broadest privilege; replace with Workload Identity Federation and delete the key |
| 2 | Supabase service-role key and database password | Supabase project settings, API and Database | full read of application data in Coverwise |
| 3 | Vercel personal access token | Vercel account, Tokens | the vendored skill reads it from `.env` and prints it |
| 4 | Stripe secret key (if live), licence-signing secret | Stripe dashboard, Developers, API keys; Thrive env | payments |
| 5 | YouTube OAuth client secret and refresh token | Google Cloud, Credentials; GitHub secrets for The-Hidden-World | long-lived channel access |
| 6 | Anthropic API key | Anthropic console | spend; set a workspace spend limit while there |
| 7 | ElevenLabs, Google Maps, Upstash/KV, EODHD, `CRON_SECRET`, `WORKSPACE_PASSCODE`, `AUDIT_WEBHOOK_URL`, `HELLO_SESSION_SECRET`, Gmail app password, GitHub fine-grained PAT for the voice vault | respective consoles; Vercel project env; GitHub secrets | lower value |

After rotating, update the Vercel project environment variables and GitHub Actions secrets, and redeploy.

### B3. Review access logs for misuse (no evidence was found, but these are the places that would show it)
- GitHub Security log; Google account activity; Vercel audit log and deployment list for deployments you did not make; Supabase auth logs and database activity; Google Cloud audit logs and billing; Anthropic usage; Stripe events; Fly and Cloud Run request logs for the party endpoints.
- If any of these shows access you did not perform: treat it as an incident. Rotate every credential on that platform, revoke all sessions, change the account password, and preserve the logs before cleaning up. Escalate to the platform's support and, for payment or customer-data systems, consider your notification obligations.

### B4. Local machine self-check
Run the procedure in `CLAUDE_CODE_EXPOSURE.md` section 7 on every machine where you have used Claude Code, especially the one used for the TV repository. Delete any allow-list entry that carries a secret, rotate that secret, prune transcripts that contain secrets or device dumps, and add the `.gitignore` block from section 8 globally.

## C. Hardening (next two weeks)

1. **Deploy workflows:** Jezza `deploy-cloudrun.yml` and `pages.yml`, Colony-Go `pages.yml`, TV `build-launcher.yml`: restrict to the default branch or manual dispatch; add `permissions:` blocks with least privilege; pin actions by commit SHA; replace the service-account key with Workload Identity Federation; publish the TV APK as a release asset instead of pushing to the branch.
2. **Branch protection:** set a protected default branch on repositories with deployments so a stolen session cannot deploy silently.
3. **Supabase:** fix Tyrants' room policies before the game is promoted; verify Coverwise and Hold-My-Loot policies in the dashboard with `set role anon` test queries.
4. **LLM routes:** shared-store rate limiting, daily budget caps, provider spend alerts; lower `max_tokens` on Trading's review route; gate Reputation-Operations server actions inside the action.
5. **Reputation-Operations gate:** HMAC-signed session with expiry, timing-safe compare, rate-limited unlock, or Vercel deployment protection.
6. **Signal:** require `CRON_SECRET` in production; re-lock `zod`.
7. **Dependencies:** update `express`/`body-parser` (Jezza), `zod` (Signal, intelligence), `pptxgenjs`/`image-size` (intelligence), `vitest` (dev).
8. **Documentation:** reword every instruction that puts a secret inline on a command line; remove or replace the `vercel-cli-with-tokens` skill in Signal.
9. **Visibility defaults:** create new repositories as private unless publication is intended; review the 30 public repositories and make private those that were never meant to be public (client mock-ups, device tooling, personal projects).
10. **Repository secret scanning:** enable GitHub secret scanning and push protection on every repository (the Security repository shows it enabled; the others could not be read).

## D. How to remove data from Git history safely

Deleting a file in a new commit does not remove it; the blob remains retrievable. Sequence:

1. Make the repository private first so the window closes while you work.
2. Clone a fresh mirror as a backup and keep it offline as evidence: `git clone --mirror <url> backup.git`.
3. Rewrite history with `git filter-repo --invert-paths --path research/ --path business/call-brief-first-three.md --path src/lib/data/prospects.seed.json` (adjust paths per repository), or recreate the repository from a clean tree if the history has no value.
4. Force-push all refs (`git push --force --all && git push --force --tags`), then delete and recreate any pull requests whose refs hold the old blobs, or ask GitHub Support to purge unreachable objects.
5. Verify: fresh clone, `git log --all --diff-filter=A --name-only | grep <path>` must print nothing; `git rev-list --objects --all | grep <path>` must print nothing.
6. Only then decide whether to make the repository public again.

## E. Verification checklist

| Action | How to verify it worked |
|---|---|
| Repository made private | repository page shows the lock badge; anonymous visit returns 404 |
| History rewritten | fresh clone, object and path searches above return nothing |
| Party server stopped and data deleted | console shows no running instance; bucket and volume empty |
| Deployment protection enabled | the Vercel URL prompts for authentication from a private browser window |
| MFA enabled | security settings show a hardware key or passkey and no SMS factor |
| Secrets rotated | old value shown as revoked or deleted in each console; new value present in Vercel env and GitHub secrets; redeploy succeeds |
| Workflows restricted | workflow file shows a single branch trigger and a `permissions:` block; a test push to a side branch triggers no deploy |
| `.gitignore` in place | `git check-ignore -v .claude/settings.local.json .env .vercel/project.json` echoes each path |
| Local Claude Code clean | the self-check pipeline prints nothing |

## F. What this plan does not do

- It does not change anything on your behalf.
- It does not declare any account compromised; no evidence supports that.
- It does not cover the owner's local machine beyond the self-check, because that machine was not accessible to this engagement.
