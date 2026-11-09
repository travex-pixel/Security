# Repository Inventory

**Engagement:** Travex Pixel personal data and credential exposure investigation
**Account:** `github.com/travex-pixel` (user id 338128618, created 2026-10-05)
**Date of evidence capture:** 2026-11-09 (UTC)
**Evidence method:** every repository was mirror-cloned (`git clone --mirror`), which fetches every branch, tag, pull-request ref and reachable commit. No clone was shallow. All scanning was performed against these mirrors; nothing was written to any repository.

## 1. Discovery and authentication

| Check | Result |
|---|---|
| Authenticated GitHub identity | `travex-pixel` (confirmed via the GitHub API `user` endpoint) |
| Repositories reported by GitHub for the account | 33 (30 public, 3 private, 0 archived, 0 forks of other projects) |
| Repositories mirrored with full history | 33 of 33 |
| Public gists | 0 |
| Organisations | none visible |
| Pull requests across the account | 2 (The-Hidden-World #1, Fruit #1), both open |
| Issues | 0 |
| Forks of the account's repositories by others | 0 on every repository |
| Stars | 0 on every repository |

Access limits that affected discovery are recorded in `AUDIT_LIMITATIONS.md`. In short: full git content was available for every repository, but repository-level GitHub API endpoints (pull request text, Actions logs, collaborators, deploy keys, webhooks, GitHub secret-scanning alerts) were reachable only for the four repositories attached to this session (Security, Intelligence, hello-deployed, Fruit). GitHub metadata for the other 29 repositories came from the search API only.

## 2. Inventory

Columns: Commits counts every commit reachable from any ref. "Claude cfg committed" records whether any `.claude/` directory, `CLAUDE.md`, `AGENTS.md` or `.mcp.json` exists at HEAD (see `CLAUDE_CODE_EXPOSURE.md` for the content assessment; no `settings.json`, `settings.local.json` or `.mcp.json` was ever committed in any repository). Scanner columns give raw hit counts before manual triage; every hit was reviewed and is dispositioned in `CREDENTIAL_EXPOSURE_REPORT.md`.

| # | Repository | Visibility | Archived | Fork | Pages | Created | Last push | Commits (all refs) | Branches | Tags | Files @HEAD | Full history | Claude cfg committed | gitleaks | trufflehog | PII sweep | Deployed URL (from GitHub metadata) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Bin-Night | public | false | false | true | 2026-10-17 | 2026-10-18 | 3 | 1 | 0 | 29 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 2 | BNA-HVAC | public | false | false | true | 2026-10-07 | 2026-10-07 | 3 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 3 | Bucket-League | public | false | false | true | 2026-10-07 | 2026-10-07 | 1 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 1 | 0 | yes |  |
| 4 | Buckets | public | false | false | true | 2026-10-06 | 2026-10-06 | 2 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 5 | Callup | public | false | false | true | 2026-10-08 | 2026-10-08 | 1 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 6 | CB-Accounting | public | false | false | true | 2026-10-07 | 2026-10-07 | 1 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 7 | Colony-Go | public | false | false | true | 2026-10-16 | 2026-10-17 | 9 | 1 | 0 | 11 | yes (mirror, not shallow) | none | 2 | 0 | yes |  |
| 8 | Coverwise | public | false | false | false | 2026-11-09 | 2026-11-09 | 8 | 1 | 0 | 100 | yes (mirror, not shallow) | AGENTS.md | 0 | 0 | yes | https://coverwise-taupe.vercel.app |
| 9 | Crush-Gear | public | false | false | true | 2026-10-10 | 2026-10-10 | 3 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 10 | DentalAbroad | public | false | false | false | 2026-11-03 | 2026-11-03 | 0 | 0 | 0 | 0 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 11 | fruit | PRIVATE | false | false | false | 2026-10-10 | 2026-10-11 | 49 | 4 | 0 | 64 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 12 | hello-deployed | PRIVATE | false | false | false | 2026-10-20 | 2026-10-20 | 1 | 1 | 0 | 117 | yes (mirror, not shallow) | AGENTS.md | 0 | 0 | yes | https://hello-deployed-sand.vercel.app |
| 13 | hello | public | false | false | true | 2026-10-20 | 2026-10-22 | 9 | 1 | 0 | 168 | yes (mirror, not shallow) | AGENTS.md | 0 | 0 | yes | https://travex-pixel.github.io/hello/ |
| 14 | Hold-My-Loot | public | false | false | true | 2026-10-05 | 2026-11-06 | 53 | 2 | 0 | 8 | yes (mirror, not shallow) | none | 1 | 0 | yes |  |
| 15 | How-Good | public | false | false | true | 2026-10-10 | 2026-10-10 | 5 | 1 | 0 | 32 | yes (mirror, not shallow) | none | 1 | 0 | yes |  |
| 16 | HowGood-Arcade | public | false | false | false | 2026-10-17 | 2026-10-17 | 0 | 0 | 0 | 0 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 17 | intelligence | PRIVATE | false | false | false | 2026-10-22 | 2026-11-05 | 98 | 1 | 0 | 248 | yes (mirror, not shallow) | none | 0 | 0 | yes | https://projectmemory-omega.vercel.app |
| 18 | Jezza-Secret-Mission | public | false | false | true | 2026-10-18 | 2026-10-25 | 76 | 1 | 0 | 158 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 19 | Moss-and-Moon | public | false | false | true | 2026-10-16 | 2026-10-17 | 5 | 1 | 0 | 7 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 20 | NBA-fantasy- | public | false | false | false | 2026-10-20 | 2026-11-06 | 27 | 1 | 0 | 30 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 21 | NewApp | public | false | false | false | 2026-10-28 | 2026-11-01 | 12 | 1 | 0 | 432 | yes (mirror, not shallow) | .claude, CLAUDE.md | 0 | 0 | yes | https://new-app-one-snowy.vercel.app |
| 22 | Reputation-Operations | public | false | false | false | 2026-11-02 | 2026-11-04 | 16 | 1 | 0 | 329 | yes (mirror, not shallow) | .claude, CLAUDE.md | 0 | 0 | yes | https://reputation-operations.vercel.app |
| 23 | Security | public | false | false | false | 2026-11-09 | 2026-11-09 | 0 | 0 | 0 | 0 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 24 | Servo-Empire | public | false | false | true | 2026-10-17 | 2026-10-21 | 5 | 1 | 0 | 58 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 25 | Signal | public | false | false | false | 2026-10-29 | 2026-11-08 | 17 | 1 | 0 | 520 | yes (mirror, not shallow) | .claude, AGENTS.md | 0 | 0 | yes | https://signal-wine-ten.vercel.app |
| 26 | SleepMax | public | false | false | false | 2026-10-29 | 2026-10-29 | 4 | 1 | 0 | 276 | yes (mirror, not shallow) | agents/skills md only (203 files) | 0 | 0 | yes | https://sleep-max.vercel.app |
| 27 | The-Hidden-World | public | false | false | false | 2026-10-10 | 2026-10-30 | 557 | 11 | 0 | 127 | yes (mirror, not shallow) | none | 0 | 35 | yes |  |
| 28 | Thrive | public | false | false | false | 2026-10-21 | 2026-10-23 | 9 | 1 | 0 | 114 | yes (mirror, not shallow) | AGENTS.md | 3 | 0 | yes |  |
| 29 | Trading | public | false | false | false | 2026-11-07 | 2026-11-08 | 15 | 1 | 0 | 294 | yes (mirror, not shallow) | .claude, AGENTS.md | 0 | 0 | yes | https://trading-nine-rho.vercel.app |
| 30 | TV | public | false | false | false | 2026-10-21 | 2026-11-02 | 23 | 1 | 0 | 618 | yes (mirror, not shallow) | none | 1 | 0 | yes |  |
| 31 | Tyrants | public | false | false | false | 2026-10-31 | 2026-11-01 | 14 | 1 | 0 | 88 | yes (mirror, not shallow) | none | 0 | 0 | yes | https://tyrants-theta.vercel.app |
| 32 | VisaFly | public | false | false | true | 2026-10-08 | 2026-10-08 | 1 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |
| 33 | Yeet-Keep | public | false | false | true | 2026-10-08 | 2026-10-08 | 1 | 1 | 0 | 1 | yes (mirror, not shallow) | none | 0 | 0 | yes |  |

Notes on individual rows:

- **Security, DentalAbroad, HowGood-Arcade** have zero commits. They were created but never pushed to. Security is the repository this engagement writes to.
- **The-Hidden-World** carries 557 commits across 11 branches plus two pull-request refs. It is 2.1 GB because rendered video files were committed. Binary blobs were not opened; all text blobs were.
- **Hold-My-Loot** has two branches whose content differs materially: `main` (deployed to a custom domain) and a `ccr-*` branch holding a Supabase leaderboard. Both were scanned.
- **Fruit** has 37 GitHub Actions runs (render jobs). One job log was sampled for leaked values; the workflow references no repository secrets.
- **Intelligence** (private) has 98 commits and a public Vercel deployment URL recorded in its GitHub metadata.

## 3. Scan coverage statement

For every one of the 33 repositories the following were performed against all refs:

1. gitleaks v8.21.2 (`gitleaks git`, full history, redacted output).
2. trufflehog v3.84.1 (`--bare`, full history, no live verification). A first run without `--bare` scanned zero chunks and was discarded; the re-run processed roughly 48,000 chunks.
3. detect-secrets 1.5.0 over the HEAD tree of each repository (The-Hidden-World excluded for size; its blobs were covered by the custom sweeps).
4. A custom sweep over every unique text blob reachable from any ref (13,584 unique blobs; 11,079 to 15,018 text blobs depending on the partner's size cut-off) for 40 or more secret patterns and for personal-data patterns (emails, Australian phone numbers, ABN/ACN, street addresses, dates of birth, card and account number shapes).
5. A filename sweep of every path that ever existed in any commit (8,414 distinct repository/path pairs) for sensitive names.
6. Independent re-verification by three partner reviewers using their own tooling (see `INDEPENDENT_PARTNER_REVIEW.md`).
7. EXIF and container inspection of all 2,146 image blobs in history.
8. `git fsck --unreachable` on every mirror: zero unreachable objects.

Personal-information scanning was performed on every repository. Secret scanning was performed on every repository. No repository was skipped.

## 4. What was not inventoried

- Commits that were force-pushed away and are no longer reachable from any ref on GitHub. A mirror clone cannot retrieve them. GitHub may still serve them by SHA for a period. No evidence of force-pushes was found (zero unreachable objects, linear histories), but this cannot be proven from a clone.
- Repository-level settings (collaborators, deploy keys, webhooks, Actions secrets, branch protection) for the 29 repositories not attached to this session.
- GitHub's own secret-scanning alert list. The Security repository reports secret scanning and push protection enabled; the other repositories' status could not be read.
