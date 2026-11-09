# Claude Code Exposure Assessment

**Question:** did the issue described in the November 2026 reporting about Claude Code's "Yes, and don't ask again" approvals, which persist the full approved command string (including any inline password, token or connection string) into `.claude/settings.local.json`, cause any secret to reach this account's GitHub repositories? And what residual risk exists on the owner's own machines?

## 1. Verdict

| Question | Answer | Confidence |
|---|---|---|
| Was `.claude/settings.local.json`, `.claude/settings.json`, `.mcp.json` or `.claude.json` ever committed to any of the 33 repositories, on any branch, at any time? | **No.** NOT IDENTIFIED IN THE AUDITED SCOPE. | High |
| Was a permission allow-list committed under a different filename? | **No.** Content scan of every text blob in history for `"permissions"`, `"allow":`, `"Bash(`, `"mcpServers"`, `"deny":`, `"additionalDirectories"` found zero hits outside vendor skill metadata. | High |
| Do the committed `.claude/` directories contain tokens, project IDs, organisation IDs or team slugs? | **No.** All committed `.claude/` content is vendor-published or project-rule markdown. | High |
| Does the owner's local machine hold a `settings.local.json` with a secret? | **UNABLE TO DETERMINE.** The local machine was not accessible. One repository (TV) shows definite local Claude Code use; others may have been used locally without leaving a trace. | n/a |
| Were secrets pasted into Claude Code web sessions? | **POTENTIAL EXPOSURE — REQUIRES VERIFICATION.** Committed documentation shows which credentials were handled in sessions; transcripts are not visible to this audit. | n/a |

## 2. How the verdict was reached

Three independent methods, each over every ref of all 33 mirrors:

1. **Filename sweep.** `git rev-list --objects --all` and `git log --all --diff-filter=ADR --name-only`, matched against `.claude*`, `.mcp.json`, `settings.json`, `settings.local.json`, `CLAUDE.md`, `AGENTS.md`, `.cursor*`, `.vscode/`, `.idea/`, `.devcontainer*`, `.codex*`, `.aider*`, `.continue/`, `.windsurf*`, `.gemini`, `.roo`, `.cline`, `.kiro`, `.env*`, `.npmrc`, `.netrc`, `.vercel`, `.supabase/`, `cookies.txt`, `*.pem`, `*.key`, `id_rsa`. Result: only `.claude/agents/*.md`, `.claude/skills/**`, `CLAUDE.md`, `AGENTS.md` and `.env.example` ever existed.
2. **Content sweep.** Every unique text blob in history (11,079 blobs) scanned for the settings-JSON markers above and for about 30 command shapes that embed secrets (`--token`, `--password`, `PGPASSWORD=`, credentialed `postgres://` URIs, `supabase login`, `vercel --token`, `gh auth login --with-token`, `curl -H Authorization:`, `--db-url`, `export *_KEY=`, `echo … > .env`, `ANTHROPIC_API_KEY=`, `OPENAI_API_KEY=`, `vercel env add`, `claude mcp add`, `dangerously-skip-permissions`, `allowedTools`). 57 raw hits, all reviewed: placeholders, `mkdir -p` false positives, variable names, a 25-character test stub.
3. **Dangling objects.** `git fsck --unreachable` on every mirror: zero.

## 3. What is committed under `.claude/` (all clean)

| Repository | Committed Claude content | Assessment |
|---|---|---|
| NewApp | `.claude/agents/impeccable-*.md` (4), `.claude/skills/deploy-to-vercel`, `vercel-composition-patterns`, `vercel-optimize`, `vercel-react-best-practices`; `CLAUDE.md` | vendor skill bundles and project rules; env var names only |
| Reputation-Operations | same agent/skill set; `CLAUDE.md` | names `ANTHROPIC_API_KEY`, `WORKSPACE_PASSCODE`, `AUDIT_WEBHOOK_URL` by name only; states secrets stay server-side |
| Signal | 15 skills including `deploy-to-vercel` and **`vercel-cli-with-tokens`** | see finding CC-03 |
| SleepMax, Trading | `deploy-to-vercel`, `impeccable`, `frontend-design` skills | byte-identical vendor files across five repositories; `<team-slug>` placeholders only |
| The-Hidden-World | `.claude/skills/storyforge/SKILL.md`, `CLAUDE.md` | references `ANTHROPIC_API_KEY` by name; `CLAUDE.md` records operator workflow details (phone type, download limits) |
| Coverwise, Thrive, Trading, hello, hello-deployed, Signal | `AGENTS.md` | auto-generated Next.js agent rules |

No `.mcp.json`, hooks, commands or settings files anywhere.

## 4. Findings

### CC-01 — No Claude Code permission or MCP file was ever committed
- Location: all 33 repositories, all refs. Category: Claude Code persistence (the reported scenario). Public/private: both. Current/historical: never.
- Severity: INFORMATIONAL (clean). Confidence: High.

### CC-02 — No repository ignores `.claude/settings.local.json` (latent control gap)
- Location: every `.gitignore` at HEAD and every historical version (35 versions across 19 repositories). Only `.claude/skills/impeccable/scripts/bin/` is ignored (NewApp, Reputation-Operations). 13 repositories have no `.gitignore` at all; 7 do not ignore `.env`; 10 do not ignore `.vercel`.
- Consequence: a single `git add -A` on a machine where Claude Code has written `settings.local.json`, or where `vercel link` has written `.vercel/project.json`, commits it. Claude Code does not add this file to `.gitignore` by default. The absence of a leak so far reflects the fact that nearly all work was done in ephemeral cloud containers, not a control.
- Severity: MEDIUM (latent, no exposure yet). Confidence: High.
- Action: add the ignore block in section 7 to every repository and to the global git ignore file.

### CC-03 — Committed documentation and one vendored skill instruct running secrets inline on the command line
Values in git are placeholders; the risk is the command shape, which "don't ask again" persists verbatim with the real value substituted, and which also lands in shell history and session transcripts.

| Repository | Path and line | Shape |
|---|---|---|
| hello | `docs/AI_CONVERSATION.md:189` | `ANTHROPIC_API_KEY=… npm run eval` |
| hello | `docs/HUMAN_MODE.md:168` | `TEST_DATABASE_URL=postgresql://… npm test` |
| Reputation-Operations | `scripts/enrich-places.mjs:6-7`, `README.md:74-75` | `GOOGLE_MAPS_API_KEY=... node scripts/enrich-places.mjs` |
| The-Hidden-World | `docs/TTS_PROVIDERS.md:27` (commit 1b24436) | `export ELEVENLABS_API_KEY=...` |
| Jezza-Secret-Mission | `docs/DEPLOY.md` | `PROJECT_ID=<id> ./scripts/deploy-cloudrun.sh` (not a secret, but would be persisted) |
| Signal | `.claude/skills/vercel-cli-with-tokens/SKILL.md:12-70` (added bd2c222, 2026-10-29, at HEAD) | Vercel's published skill: `printenv VERCEL_TOKEN`, `grep '^VERCEL_TOKEN=' .env`, `export VERCEL_TOKEN=$(grep … .env | cut -d= -f2-)`, `echo "value" \| vercel env add …`. It explicitly forbids `--token` on the command line, but the `printenv`/`grep` steps print the token into the session transcript and the `echo "value" |` shape puts a secret in the Bash command string. |

- Every historical version of these files was checked (`git log --all -p`): placeholders only (`…`, `...`, `your-key`, `vca_abc123`, `TEST-CODE`).
- Was the Vercel token skill ever invoked? No commit message, no `.vercel/` directory and no deploy URL in Signal suggests it was. Judged not invoked, Medium confidence.
- Severity: LOW for the public footprint; MEDIUM as a local-machine and transcript implication. Confidence: High.
- Action: reword the documentation to read secrets from `.env` files or `read -s`; remove or replace the `vercel-cli-with-tokens` skill; never approve an env-prefixed or `echo <secret> |` command with "don't ask again".

### CC-04 — Local Claude Code use identified for the TV repository; local files are un-audited
- Evidence: commit forensics across all repositories. About 1,850 commits carry the Claude cloud committer at UTC; 53 commits by `travex-pixel` at UTC carry Claude session trailers (cloud); 42 GitHub web-UI commits are at Sydney time. Exactly four commits (TV: 6f4a099, 5f3662c, d859824, 6a4c2eb, 2026-10-23, Sydney time, no trailers, unsigned) commit ADB dumps that can only be produced from a machine on the TV's home network. TV's `tools/relay.md` says "Claude Code on macOS is the usual case" and its `.gitignore` excludes `adbkey*` and `*.keystore`, confirming local private keys existed in that working tree.
- Implication: for TV, a local clone very likely has `.claude/settings.local.json`, `~/.claude/projects/<TV-path>/*.jsonl` transcripts (containing the full device dumps: identifiers, Wi-Fi owner, LAN addresses) and possibly `~/.claude.json` project entries. The TV docs instruct commands with the TV's LAN address inline (`TV_HOST=192.168.x.x …`, `relay.py 192.168.x.x`, Tailscale routes), so a local allow-list would hold home network topology, not credentials. The real local sensitivities are the gitignored ADB private key and the transcripts.
- Absence of local commits elsewhere does not prove absence of local use. Several deployments (Coverwise, intelligence, Jezza on Fly, hello-deployed) have set-up steps not visible in commits, so un-committed local sessions are plausible.
- Severity: MEDIUM (local, unverifiable). Confidence: High that TV was local; Medium for any other repository.

### CC-05 — Credentials that the committed material shows were handled in sessions (precautionary rotation list)
Whether Claude Code web transcripts retain pasted values server-side, and for how long, cannot be verified here. The owner should recall what was pasted into prompts or typed into agent-run shell commands and rotate those specific credentials as a precaution.

| Credential | Evidence that it was handled in a session or shell | Suggested rotation priority |
|---|---|---|
| GCP service-account JSON key (`GCP_SA_KEY`) and `GCP_PROJECT_ID` | Jezza `.github/workflows/deploy-cloudrun.yml:3-6`, `docs/DEPLOY.md:23,30` (`gcloud auth login`) | 1 |
| `SUPABASE_SERVICE_ROLE_KEY`, `DATABASE_URL` with password | hello `docs/HUMAN_MODE.md:75-90,168`; Coverwise `README.md:70-79` | 2 |
| `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID` | Signal vendored skill | 3 |
| `STRIPE_SECRET_KEY`, `LICENSE_SIGNING_SECRET` | Thrive `docs/launch/DEPLOY.md:15-62` | 4 |
| YouTube OAuth client id/secret/refresh token | The-Hidden-World `.github/workflows/upload-youtube.yml` | 5 |
| `ANTHROPIC_API_KEY` | hello `docs/AI_CONVERSATION.md:189`; `.env.example` in hello, hello-deployed, Trading; Reputation-Operations and The-Hidden-World skill/rule files | 6 |
| `ELEVENLABS_API_KEY`, `GOOGLE_MAPS_API_KEY`, `KV_REST_API_TOKEN`, `CRON_SECRET`, `WORKSPACE_PASSCODE`, `AUDIT_WEBHOOK_URL`, `EODHD_API_TOKEN`, `HELLO_SESSION_SECRET`, Gmail SMTP app password | docs and `.env.example` files as listed in `CREDENTIAL_EXPOSURE_REPORT.md` | 7 |

Severity: POTENTIAL EXPOSURE — REQUIRES VERIFICATION. Confidence: n/a (not observable).

## 5. Why the cloud-session workflow mattered here

29 of 30 non-empty repositories have a `ccr-*` or `claude/*` branch as HEAD. In Claude Code on the web, the project-level `settings.local.json` is written inside a container that is destroyed with the session; it reaches GitHub only if the agent commits it, which the evidence shows never happened. The persistence issue described in the reporting therefore applies to the owner's local machines: the TV clone (definite) and any repository the owner also opened locally (possible).

## 6. What these local files contain (structure only, observed on this session's own container, not the owner's)

- `~/.claude.json`: `oauthAccount` (account id, email address, organisation id), machine and user ids, feature caches, and on local installs a `projects` map keyed by absolute path with `allowedTools` (the persisted allow-list), `mcpServers` (with any env or headers given to them) and prompt `history`. Treat as sensitive.
- `~/.claude/projects/<escaped-path>/<session>.jsonl`: full transcripts of every prompt, tool call and tool result, including command output.
- `~/.claude/shell-snapshots/*.sh`: a snapshot of the shell environment at session start, which can include exported variables.
- Project `.claude/settings.local.json`: `{"permissions": {"allow": ["Bash(<exact command string>)", …]}}`. The allow entry is the literal command, so `Bash(PGPASSWORD=hunter2 psql …)` stores the password.

## 7. Safe local self-check for the owner

Run in a normal terminal, not through an agent. Nothing below deletes anything; every command prints only key names or redacted previews.

```bash
# Redaction helper: first 6 characters of anything long, then a marker
red(){ sed -E 's/([A-Za-z0-9_\-]{6})[A-Za-z0-9_\-\.\/+=]{10,}/\1…[redacted]/g'; }

# 1) Every project-level permission file on this machine, filtered to secret-shaped entries
find ~ \( -path '*/node_modules' -o -path '*/Library' -o -path '*/.Trash' \) -prune -o -type f -path '*/.claude/settings.local.json' -print0 2>/dev/null \
 | while IFS= read -r -d '' f; do jq -r --arg f "$f" '(.permissions.allow[]?, .permissions.deny[]?) | "\($f): \(.)"' "$f" 2>/dev/null; done \
 | grep -iE 'token|passw|secret|key=|api_?key|postgres(ql)?://|mysql://|mongodb://|redis://|Authorization|Bearer|sk-|vca_|sbp_|ghp_|github_pat_|AKIA|eyJ' | red
# Any line printed is an allow-rule carrying a credential: delete that one entry, then rotate that credential.

# 2) User-level settings and the global state file
jq -r '.permissions.allow[]?' ~/.claude/settings.json 2>/dev/null | grep -iE 'token|passw|secret|key|://' | red
jq -r '.projects | to_entries[] | "\(.key): \(.value.allowedTools|length) allowedTools, \((.value.mcpServers//{})|keys|join(","))"' ~/.claude.json 2>/dev/null
jq -r '.projects[].allowedTools[]?' ~/.claude.json 2>/dev/null | grep -iE 'token|passw|secret|key|://' | red
jq -r '.projects[].mcpServers? // {} | .. | strings' ~/.claude.json 2>/dev/null | grep -iE 'token|passw|secret|key|Bearer' | red
find ~ -maxdepth 4 -name '.mcp.json' -not -path '*/node_modules/*' 2>/dev/null | while read f; do echo "== $f"; jq -r '.. | strings' "$f" | grep -iE 'token|passw|secret|key|Bearer' | red; done

# 3) Session transcripts: file names only
grep -rlE 'sk-ant-[A-Za-z0-9_-]{20}|vca_[A-Za-z0-9]{20}|sbp_[A-Za-z0-9]{20}|ghp_[A-Za-z0-9]{30}|AKIA[0-9A-Z]{16}|postgres(ql)?://[^ ]+:[^ ]+@|PRIVATE KEY-----' ~/.claude/projects 2>/dev/null
grep -rlE 'android_id|bluetooth_address|wifi' ~/.claude/projects 2>/dev/null      # the TV project's device dumps
grep -oE '"display":"[^"]{0,60}' ~/.claude/history.jsonl 2>/dev/null | grep -iE 'token|passw|secret|key|://' | red

# 4) Shell history and .env files
grep -nE 'PGPASSWORD=|--token|--password|supabase login|gh auth login|vercel .*--token|export [A-Z_]*(KEY|TOKEN|SECRET)=|curl .*-H .Authorization' ~/.zsh_history ~/.bash_history 2>/dev/null | red
find ~ -path '*/node_modules' -prune -o \( -name '.env' -o -name '.env.local' -o -name '.env.*.local' \) -print 2>/dev/null
# For each .env, show key names only:  grep -oE '^[A-Z_]+=' path/.env

# 5) CLI credential stores: presence and permissions only
for f in ~/.config/gh/hosts.yml ~/.vercel/auth.json ~/.supabase/access-token ~/.aws/credentials ~/.npmrc ~/.netrc ~/.docker/config.json ~/.config/gcloud/credentials.db ~/.claude/.credentials.json; do
  [ -e "$f" ] && ls -l "$f"
done
```

How to clean safely:
1. In `settings.local.json` or `~/.claude.json`: delete only the offending allow entry, keep the rest, then rotate that credential. It has also been in shell history and the transcript. Do not delete whole files before reviewing them; they are the evidence of what was exposed.
2. In `~/.claude/projects/*.jsonl`: if a file is flagged, open it and judge the context; if it holds a real secret, rotate it, then delete that session file. Consider setting a short transcript retention period in Claude Code settings.
3. For the TV project: treat the transcript folder as personal data and prune it after review.
4. Going forward: approve commands containing a literal secret with a one-time "Yes", never "don't ask again"; keep secrets in `.env` files and reference them (`VAR="$(cat …)"`) or use the tool's own login flow; keep `~/.claude.json` and `~/.claude/` out of dotfile repositories and cloud-synced folders; `chmod 600` on them.

## 8. Recommended `.gitignore` block for every repository (and `~/.config/git/ignore`)

```gitignore
# Claude Code / agent local state
.claude/settings.local.json
.claude/*.local.json
.mcp.json
.claude/.credentials.json
# IDE/agent files that may hold tokens
.cursor/mcp.json
.vscode/settings.json
.idea/
# Platform CLI state
.vercel/
.vercel
.supabase/
.netlify/
.firebase/
# Secrets
.env
.env.*
!.env.example
*.pem
*.key
*.p12
*.jks
*.keystore
adbkey
adbkey.pub
cookies.txt
```

Verify in each clone (every path should echo; silence means not ignored):

```bash
git check-ignore -v .claude/settings.local.json .mcp.json .vercel/project.json .env .env.local secrets.pem adbkey
git ls-files -- .claude/settings.local.json .mcp.json .vercel .env* | grep -v example   # must print nothing
```

## 9. Limitations specific to this assessment

- The owner's machines were not examined; local presence of secrets is inferred as possible, not observed.
- Claude Code web transcript retention is not observable from this engagement.
- Mirrors reflect the current ref graph; force-pushed branches are not recoverable from a clone.
- Pattern scanning proves the absence of the specific reported artefacts and common command shapes, not of every conceivable secret format.
