# Executive Summary

**Engagement:** Travex Pixel personal data and credential exposure investigation
**Subject:** the GitHub account `travex-pixel` (33 repositories: 30 public, 3 private) and its associated development footprint
**Date:** 2026-11-09
**Nature:** independent, read-only forensic review by six specialist functions with a three-round challenge process. No credential was used or tested. No repository, account or service was changed. All evidence in the reports is redacted.

## 1. The answer in one paragraph

Nothing in your 33 repositories, on any branch or anywhere in their history, contains a password, API key, token, private key or database credential of yours. Four independent sweeps, three scanners and an adversarial audit agree on that. The Claude Code "don't ask again" issue did not put a settings file or an approved command into any repository. What you have exposed, publicly and still live today, is not credentials but data: a researched dossier on 50 real businesses with critical commentary, your television's hardware identifiers and home network layout, and smaller items such as friends' nicknames, a real party code and your Instagram analytics. Separately, a private repository presents a real firm's executives as your product team on a public deployment address. No evidence of any account being accessed by anyone else was found, but the places that would show such access (your account sign-in logs, MFA state, local machine, Claude Code session transcripts) were outside what this engagement could see, so those are marked as unable to determine rather than safe.

## 2. The ten executive questions

| # | Question | Verdict | Basis |
|---|---|---|---|
| 1 | Have any of my passwords been found in my repositories? | **NOT IDENTIFIED IN THE AUDITED SCOPE** | every blob in every ref of all 33 repositories scanned by seven methods; only placeholders and test values exist |
| 2 | Have any API keys or authentication tokens been exposed? | **NOT IDENTIFIED IN THE AUDITED SCOPE** | same coverage; the only key-like value is a PostHog project key that is public by design |
| 3 | Have any personal details or sensitive documents been exposed? | **CONFIRMED** (secondary and third-party data; not your core identity data) | your TV's serial, Android id, Bluetooth MAC and room name and home LAN layout (TV, public); a dossier on 50 real businesses and about seven named owners (Reputation-Operations, public); real event details and nicknames (Jezza, public); your Instagram analytics screenshots (The-Hidden-World, side branch, public); real executives' names and photos (intelligence, private repo, public deployment URL). Your real name, personal email, phone, home address, date of birth, identity documents and financial details were not found anywhere. Document-like files in Thrive and VisaFly are synthetic fixtures. |
| 4 | Has any information been exposed through Claude Code configuration files? | **NOT IDENTIFIED IN THE AUDITED SCOPE** for GitHub; **UNABLE TO DETERMINE** for your local machine | no `.claude/settings.json`, `settings.local.json`, `.mcp.json` or allow-list content was ever committed anywhere; committed `.claude/` folders hold only vendor and project markdown. Your local machine was not accessible; a self-check is provided. |
| 5 | Have any secrets been committed to public GitHub repositories? | **NOT IDENTIFIED IN THE AUDITED SCOPE** | the one public-by-design analytics key aside |
| 6 | Have secrets been deleted from files but retained in Git history? | **NOT IDENTIFIED IN THE AUDITED SCOPE** | blob-level sweep of history and a removed-line search of every diff; zero unreachable objects |
| 7 | Is there evidence that any exposed credential has been misused? | **UNABLE TO DETERMINE** | no credential was found to misuse; repository-side indicators are clean (no unexpected authors, zero forks); account sign-in and audit logs were not accessible |
| 8 | Could someone realistically access any of my accounts or applications? | **POTENTIAL EXPOSURE — REQUIRES VERIFICATION** | not through anything in the repositories. The realistic paths are phishing of the single identity that controls every platform (its MFA state is unknown), and application weaknesses that expose game data, party photos and API spend rather than accounts (party-code enumeration, open room policies in Tyrants, unauthenticated AI routes). |
| 9 | What should I change or revoke immediately? | see section 4 | no mandatory revocation arises from a found leak; take the public data private today; harden the root identity; precautionary rotation of credentials that documentation shows were handled in shell commands or sessions |
| 10 | What is the overall residual risk after the investigation? | **MEDIUM**, reducible to LOW by the actions in section 4 | structural (one identity, no review gate, deploy-on-any-branch with a broad cloud key) and data-related (public third-party and device data), not credential leakage |

## 3. Severity register (signed off after the challenge rounds)

| Severity | Count | Items |
|---|---|---|
| CRITICAL | 0 | none |
| HIGH | 0 | none (the auditor would place the business dossier and identity concentration at HIGH; the lead and CISO hold MEDIUM because the engagement scale reserves HIGH for confirmed exposure of meaningful personal information or credentials; the disagreement is recorded in `INDEPENDENT_PARTNER_REVIEW.md`) |
| MEDIUM | 8 | third-party business dossier (Reputation-Operations); TV device identifiers; Tyrants anonymous room policies; Jezza party-code enumeration and photo exposure (while the server runs); Jezza deploy workflow with a broad long-lived cloud key on any branch; real executives on a public deployment URL (intelligence, uncertain accessibility); unauthenticated LLM routes (if keys are deployed); single identity controlling every platform |
| LOW | 8 | Instagram analytics on a side branch; friend nicknames and past event; named employer in a brief; TV CI pushing a debug APK; inline-secret command patterns in docs and one vendored skill; `.gitignore` gaps; weak workspace gate; token-in-URL clone pattern in one workflow |
| INFORMATIONAL | 5 | public PostHog project key; client businesses' public contact details; Claude session URLs in commit trailers; private project hostname in a private repo; local Claude Code use inferred for one repository |
| UNABLE TO DETERMINE | 1 | secrets in local Claude Code settings or transcripts, and in web-session transcripts |

## 4. What to do, in order

**Today**
1. Make Reputation-Operations private; then purge the research files, call brief and seed JSON from history and redeploy. This is the one item with third-party reputational and commercial consequences for you.
2. Make TV private or remove the `backup/` and `discovery/` folders and rewrite history; disable ADB over network on the TV. Treat the serial and MAC as disclosed.
3. Confirm the Jezza party server is stopped and its photos deleted; replace the screenshot that shows a real party code.
4. Put Vercel deployment protection on the private `intelligence` project, or take it down until the named firm consents.
5. Enable passkey or hardware-key MFA on your Google account and GitHub; remove SMS recovery; review connected apps, tokens and sessions.

**This week**
6. Review each platform console for the credentials listed in `IMMEDIATE_REMEDIATION_PLAN.md` B2 and rotate, in order, the Google Cloud service-account key (replace with Workload Identity Federation), the Supabase service-role key, the Vercel token, Stripe keys if live, the YouTube refresh token and the Anthropic key. These rotations are precautionary: nothing shows these values leaked, but the documentation shows they were handled in shell commands and sessions this audit could not see.
7. Run the local self-check in `CLAUDE_CODE_EXPOSURE.md` on every machine where you used Claude Code, especially the one used for the TV work.
8. Check account activity logs on GitHub, Google, Vercel, Supabase, Google Cloud and Anthropic for anything you did not do.

**Next two weeks**
9. Restrict deploy workflows to the default branch, add least-privilege permissions, pin actions, protect default branches.
10. Fix Tyrants' Supabase policies; add shared rate limits and spend caps to the AI routes; fix the Reputation-Operations gate; update vulnerable dependencies; reword documentation that puts secrets on the command line and remove the token-reading Vercel skill.
11. Add the recommended `.gitignore` block to every repository and to your global git ignore; default new repositories to private.

## 5. What this engagement could not see

Your local machines, Claude Code web session transcripts, account security settings and logs, live deployments and databases, repository-level GitHub settings for 29 of 33 repositories, and any history force-pushed away before today. None of these showed a problem because none of them was visible. Details are in `AUDIT_LIMITATIONS.md`. Until the self-check and dashboard reviews in section 4 are done, the correct reading of this report is "no exposure found in everything that could be examined", not "nothing is exposed".

## 6. Reports in this set

| File | Contents |
|---|---|
| `EXECUTIVE_SUMMARY.md` | this document |
| `REPOSITORY_INVENTORY.md` | all 33 repositories with visibility, history depth, scan coverage |
| `CREDENTIAL_EXPOSURE_REPORT.md` | every credential-pattern hit and its disposition; inventory of credential names held elsewhere |
| `PERSONAL_INFORMATION_EXPOSURE.md` | personal and third-party data findings, redacted |
| `CLAUDE_CODE_EXPOSURE.md` | the "don't ask again" scenario, verdict, and the local self-check |
| `GIT_HISTORY_INVESTIGATION.md` | deleted files, side branches, unreachable objects, force-push check |
| `ACCOUNT_COMPROMISE_ASSESSMENT.md` | realistic attack chains, evidence of misuse, what could not be determined |
| `APPLICATION_SECURITY_REVIEW.md` | static review of 17 applications, 19 findings, positives |
| `IMMEDIATE_REMEDIATION_PLAN.md` | ordered actions, rotation list, history-cleanup procedure, verification checklist |
| `INDEPENDENT_PARTNER_REVIEW.md` | the three-round challenge process, conflicts resolved, signed register |
| `AUDIT_LIMITATIONS.md` | everything out of reach and how it bounds the conclusions |
