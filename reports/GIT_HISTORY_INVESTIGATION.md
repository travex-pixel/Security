# Git History Investigation

**Scope:** every commit reachable from every branch, tag and pull-request ref of all 33 repositories belonging to `travex-pixel`, captured as bare mirrors on 2026-11-09.
**Question answered:** has anything sensitive been committed and later deleted, left on a side branch, or hidden in history where a scan of the current files would miss it?

## 1. Method

1. Mirror clones (`git clone --mirror`) of all 33 repositories. Mirrors fetch `refs/heads/*`, `refs/tags/*` and `refs/pull/*`, so pull-request heads and merge refs are included.
2. `git rev-list --objects --all` to enumerate every object reachable from any ref, then `git cat-file` to read every unique text blob regardless of whether the file still exists at HEAD. This is the key control: a secret deleted in a later commit still exists as a blob and is caught.
3. `git log --all --diff-filter=D --name-only` to list every path that was deleted at some point, and manual review of the deleted content where the name suggested data (SQL, dumps, config).
4. Branch differencing (`git diff --name-only <HEAD>..<branch>`) for repositories with more than one branch, to find content that exists only on non-default branches.
5. `git fsck --unreachable --no-reflogs` on every mirror to look for dangling blobs or commits.
6. Commit metadata review: author and committer identities, timezones, Claude Code session trailers, and commit messages mentioning secrets, keys, env files, rotation or leaks.
7. Cross-check of every secret-pattern or personal-data hit against HEAD trees to classify each as "current" or "historical only".

## 2. Repository history summary

| Repository | First commit | Last commit | Commits | Refs | Paths deleted at some point | Notes |
|---|---|---|---|---|---|---|
| BNA-HVAC | 2026-10-08 | 2026-10-08 | 3 | main | 0 | single-page site |
| Bin-Night | 2026-10-17 | 2026-10-18 | 3 | ccr-b84ed49c | 0 | |
| Bucket-League | 2026-10-07 | 2026-10-07 | 1 | main | 0 | |
| Buckets | 2026-10-06 | 2026-10-06 | 2 | main | 0 | |
| CB-Accounting | 2026-10-08 | 2026-10-08 | 1 | main | 0 | |
| Callup | 2026-10-09 | 2026-10-09 | 1 | main | 0 | |
| Colony-Go | 2026-10-16 | 2026-10-17 | 9 | ccr-19db92c9 | 0 | |
| Coverwise | 2026-11-09 | 2026-11-09 | 8 | ccr-5294ed25 | 0 | |
| Crush-Gear | 2026-10-10 | 2026-10-10 | 3 | main | 1 | deleted path is a UI file |
| DentalAbroad | – | – | 0 | none | 0 | empty |
| Hold-My-Loot | 2026-10-05 | 2026-11-06 | 53 | main, ccr-e0e37c62 | 0 | branches diverge (see 4) |
| How-Good | 2026-10-10 | 2026-10-10 | 5 | claude/api-apps-games… | 0 | |
| HowGood-Arcade | – | – | 0 | none | 0 | empty |
| Jezza-Secret-Mission | 2026-10-18 | 2026-10-25 | 76 | ccr-660c5996 | 2 | deleted: a test file and a build artefact |
| Moss-and-Moon | 2026-10-16 | 2026-10-17 | 5 | ccr-d7b0f4aa | 0 | |
| NBA-fantasy- | 2026-10-20 | 2026-11-06 | 27 | ccr-1b751505 | 0 | committed `dist/` build |
| NewApp | 2026-10-28 | 2026-11-01 | 12 | ccr-17b9ed7f | 12 | deleted: UI components |
| Reputation-Operations | 2026-11-02 | 2026-11-04 | 16 | ccr-24b3d55a | 10 | deleted: UI/design files; prospect data remains |
| Security | – | – | 0 | none | 0 | empty |
| Servo-Empire | 2026-10-17 | 2026-10-21 | 5 | ccr-b935a743 | 0 | |
| Signal | 2026-10-29 | 2026-11-08 | 17 | ccr-4f079d77 | 0 | |
| SleepMax | 2026-10-29 | 2026-10-29 | 4 | ccr-b04031b1 | 0 | |
| TV | 2026-10-21 | 2026-11-02 | 23 | ccr-907b9d08 | 2 | deleted: placeholder `getprop.txt`; real dumps remain |
| The-Hidden-World | 2026-10-10 | 2026-10-30 | 557 | 11 branches + pull/1 | 4 | large; see 4 |
| Thrive | 2026-10-21 | 2026-10-23 | 9 | ccr-ad122ae0 | 0 | |
| Trading | 2026-11-07 | 2026-11-08 | 15 | ccr-ae0a08d8 | 0 | |
| Tyrants | 2026-10-31 | 2026-11-01 | 14 | ccr-381891cc | 0 | |
| VisaFly | 2026-10-09 | 2026-10-09 | 1 | main | 0 | |
| Yeet-Keep | 2026-10-09 | 2026-10-09 | 1 | main | 0 | |
| fruit (private) | 2026-10-10 | 2026-10-11 | 49 | 4 branches + pull/1 | 0 | |
| hello | 2026-10-20 | 2026-10-22 | 9 | ccr-033ca6d4 | 11 | deleted: `supabase/schema.sql` (schema only, no data) and UI files |
| hello-deployed (private) | 2026-10-20 | 2026-10-20 | 1 | main | 0 | snapshot of hello |
| intelligence (private) | 2026-10-22 | 2026-11-05 | 98 | ccr-60aef891 | 8 | deleted: design files |

Totals: roughly 1,950 commits, 13,584 unique blobs, 8,414 distinct paths ever added. Zero tags anywhere. Zero unreachable objects in any mirror.

## 3. Secrets deleted from files but retained in history

**Result: none identified.**

Every blob that ever existed was scanned, not just the current files. The only secret-shaped content that exists in history but not at HEAD is in The-Hidden-World, and all of it is false positive: URL slugs in research notes that happen to start with `sk-` (for example "sk-for…" inside a US government news URL), JSON field names such as `author` and `token_uri`, and example media manifests. These were read individually and are not credentials.

The following files were deleted at some point and were specifically examined because their names suggested sensitive content:

| Repository | Deleted path | Content at the time of deletion | Finding |
|---|---|---|---|
| hello | `supabase/schema.sql` | table definitions with RLS enabled, no rows | not sensitive |
| TV | `discovery/20261023-000534/getprop.txt` | a placeholder; the real dump was pushed later and is still at HEAD | not a removal of sensitive data |
| Jezza-Secret-Mission | `tests/.walk.tmp.ts`, `tests/shots.battle.ts` | test scaffolding | not sensitive |
| NewApp, Reputation-Operations, intelligence, Crush-Gear | UI and design files | code | not sensitive |

No `.env`, key file, token file, cookie file, settings file or credential store was ever added to any repository, so none could have been deleted.

## 4. Content that exists only on non-default branches

Scans of the default branch alone would miss these. They were included.

| Repository | Branch | Branch-only content of interest | Assessment |
|---|---|---|---|
| The-Hidden-World | `ccr-67645e21-nwgkaq` (also `pull/1/head`, `pull/1/merge`) | `analytics/screenshots/*.jpg` (118 files, the owner's Instagram Insights screenshots), `analytics/transcripts/*.json`, `analytics/reels*.csv`, `analytics/postmortems/*.md`, `analytics/SCOREBOARD.md` | personal/business analytics of the owner's social channel; no credentials; see `PERSONAL_INFORMATION_EXPOSURE.md` |
| The-Hidden-World | `ccr-21716fbe-03vmof` | additional episode scripts | content only |
| Hold-My-Loot | `ccr-e0e37c62-g8o1qe` | `supabase/leaderboard.sql`, `supabase/README.md`, marketing and review documents, Lighthouse reports | schema and policies; no credentials. The README on this branch once instructed pasting a Supabase anon key into `index.html`; in every committed version the slot is empty. |
| fruit (private) | 3 side branches + `pull/1` | episode and render variations | content only |

## 5. Commit metadata

- Committer identities across all repositories: the Claude Code cloud committer (about 1,850 commits), GitHub's web-UI committer (44), the account owner via GitHub's no-reply address (159), a GitHub Actions bot (2) and one Vercel commit. No personal email address appears in any author or committer field.
- Exactly four commits (all in TV, 2026-10-23, Sydney timezone, no Claude session trailer) show the hallmarks of being made from the owner's local machine. All other commits were produced by Claude Code on the web (ephemeral containers) or through the GitHub web interface. This matters for the Claude Code assessment: see `CLAUDE_CODE_EXPOSURE.md`.
- Commit messages: a search of all 3,370 commit subject/body lines for secret, key, token, password, credential, env, leak, rotate, revoke and hardcode found no message describing a secret removal or rotation. Matches were content-related (game features named "secret mission", an episode about AI stop tokens).
- Branch names: `ccr-*` and `claude/*` session branches; nothing sensitive.

## 6. Pull-request refs

Both pull requests' head and merge refs were included in the mirrors and scanned. The PR titles and bodies (retrieved through the GitHub search API) describe creative-content work and contain no credential-shaped strings.

## 7. Force-push and rewritten-history check

- `git fsck --unreachable --no-reflogs`: zero unreachable objects in all 33 mirrors.
- Every branch history is linear or simple-merge; no evidence of `--force` rewrites.
- Limitation: a force-push that happened on GitHub before the mirror was taken leaves objects only on GitHub's side, retrievable by SHA for a time but not by a fresh clone. There is no indication this happened, but it cannot be excluded from a clone alone. The owner can check the Activity view on each repository for "force-pushed" events.

## 8. Conclusions

1. Secrets deleted from files but retained in Git history: **NOT IDENTIFIED IN THE AUDITED SCOPE**. The blob-level sweep covers this case directly.
2. Secrets on side branches or in pull-request refs: **NOT IDENTIFIED IN THE AUDITED SCOPE**.
3. Personal and third-party data in history: present, and in every case still present at the HEAD of some branch (nothing was "removed"). The exposure is therefore current, not merely historical. Details and severity are in `PERSONAL_INFORMATION_EXPOSURE.md`.
4. History rewriting would be needed only if the owner decides to remove the TV device dumps, the Reputation-Operations prospect data or the Instagram analytics from public view; deleting the files in a new commit would leave them retrievable. See `IMMEDIATE_REMEDIATION_PLAN.md` for the safe sequence (make private first, then rewrite, then verify).
