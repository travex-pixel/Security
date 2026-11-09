# Independent Partner Review

This document records the three-round review process: six independent investigations, an adversarial round in which specialists challenged one another and the independent auditor challenged everyone, and a final sign-off in which each material finding was tested against evidence. Consensus was not treated as evidence; each decision below cites what was checked.

## 1. The review board

| Partner | Role | Independent mandate | Round 1 output |
|---|---|---|---|
| 1 | Chief Information Security Officer | overall exposure, business consequence, risk concentration, remediation priority | service map of 17 external services, deployed-asset table, Supabase policy review, top-5 worries |
| 2 | Credential forensics specialist | every credential-like value in every ref, classified as genuine, placeholder, test, public-by-design or unknown | own blob store of 15,018 index rows (11,107 unique text blobs), 44-pattern sweep, detect-secrets, manual read of every deploy, SQL, workflow and env file in every historical version, `.gitignore` archaeology, GitHub API checks |
| 3 | Personal data and privacy forensics specialist | personal information about the owner and third parties, across text, images, documents, metadata and deleted files | regex sweeps over 11,108 text blobs, EXIF over 2,146 images, visual inspection of screenshots, PDF/DOCX fixture checks, container inspection of misnamed media |
| 4 | Claude Code and developer-environment specialist | the reported "don't ask again" persistence scenario and related workflow exposure | filename, diff-filter and content sweeps for settings markers across 11,079 blobs; commit forensics separating cloud from local work; self-check procedure |
| 5 | Attack-surface and account-takeover risk assessor (passive) | what an outsider learns and what realistic paths to compromise exist | public-footprint intelligence, authentication design review, ranked attack chains, checked-absent vectors |
| Application security reviewer | static review of the 17 application repositories | 19 findings, dependency audit against extracted lockfiles |
| 6 | Independent forensic auditor | challenge every partner; find what was missed; resolve conflicts; sign a consolidated register | own enumeration (13,584 blobs, 11,105 text), entropy scan, base64 decode, removed-line scan, metadata channels, env-under-other-name search, gap checks |

The lead investigator additionally ran the baseline (gitleaks, trufflehog, detect-secrets, a 40-pattern blob sweep, filename inventory) and verified every automated hit by hand before the partners began.

## 2. Round 1: independent findings in brief

- **Credential forensics:** no genuine usable secret anywhere in history; one public-by-design PostHog project key; two CI designs worth hardening; an inventory of secret names held elsewhere.
- **Privacy forensics:** no owner core personal data; a public third-party business dossier (Reputation-Operations); the owner's TV device dumps (TV); real event and nicknames (Jezza); Instagram analytics screenshots on a side branch (The-Hidden-World); real executives in a private repo with a public deployment URL (intelligence); synthetic fixtures confirmed synthetic (Thrive, VisaFly).
- **Claude Code:** the reported scenario did not materialise in the GitHub footprint; no repository ignores the local settings file; committed docs instruct inline-secret commands; one repository (TV) shows definite local use.
- **Attack surface:** small external surface; no `pull_request_target`; no key in any client bundle; realistic risks are identity concentration, an over-privileged deploy key on any-branch push, unauthenticated LLM routes, a weak workspace gate and party-code enumeration.
- **CISO:** no live credential; exposure is structural and data-related; one identity controls every platform; no review gate; high velocity AI-driven development.
- **Application security:** nothing CRITICAL or HIGH; six MEDIUM design issues; code quality high overall.

## 3. Round 2: adversarial challenges and outcomes

| Challenge | Raised by | Against | Outcome |
|---|---|---|---|
| "The TV dumps expose a Bluetooth MAC" versus "no MACs found" | CISO vs privacy specialist | each other | **Resolved for the CISO.** `settings-secure.txt` line 15 in both `backup/` and `discovery/` holds `bluetooth_address`. The privacy specialist had grepped the wrong file and corrected the report. Independently re-verified by the lead and the auditor. |
| Identity concentration rated CRITICAL | lead and auditor | CISO | **Re-rated.** CRITICAL on this engagement's scale requires confirmed exposure or compromise. CISO re-rated to MEDIUM (structural) with HIGH impact if realised; the auditor argued HIGH. Final: MEDIUM, see section 4. |
| "VOICE_VAULT_TOKEN embedded in a clone URL" | lead and auditor | credential specialist | **Downgraded to LOW hardening.** It is a `${{ secrets.* }}` expression via `env:`, masked by Actions, with `set +x`, `--quiet` and immediate deletion of the clone's `.git`. Residual: token in process arguments for the clone's lifetime. |
| Are the inline-secret docs placeholders in every historical version? | lead | credential specialist | **Confirmed.** `git log --all -p` on all six paths: placeholders only in every version. |
| Does the blob sweep cover committed-then-removed secrets? | lead | credential specialist and auditor | **Confirmed twice.** Removed-line search over every diff in all 33 repositories: the only removed secret-shaped line is Thrive's fake test JWT. |
| Exact counts for the prospect dataset | lead and auditor | privacy specialist | **Quantified.** 50 rows per CSV; identifiable individuals: 1 (v1), 5 (v2), about 7 distinct across all files, all from published sources; personal emails 0; phones 1 and 3 rows (business lines); premises addresses 1 and 13 to 14 rows; negative commentary aimed at businesses, not individuals, in roughly 17 to 22 rows; the seed JSON ships in the public client bundle outside the passcode gate. Privacy specialist lowered the personal-data component to LOW and kept MEDIUM for reputational risk; auditor argued MEDIUM to HIGH. |
| Are the real executives framed as fictional cast or the real team? | lead and auditor | privacy specialist | **Real team.** The firm and its people are presented as the product's proposed owners and leads; the fictional labelling applies to the project dataset only. No public repository links to the private deployment URL. |
| Did anyone look at enough screenshots? | lead and auditor | privacy specialist | **Extended.** 14 Instagram screenshots viewed by the privacy specialist, 9 more by the auditor: no handle, name, email or notification text. Jezza live screenshots and Hold-My-Loot review images: nicknames and cartoon characters only. New finding by the auditor: one Jezza screenshot publishes a real party code. |
| Absence of local commits is not absence of local Claude Code use | lead | Claude Code specialist | **Accepted and written in.** Self-check extended to every clone on the machine. |
| Was the vendored Vercel token skill ever invoked? | lead | Claude Code specialist | **Judged not invoked** (Medium confidence): no commit, `.vercel/` directory or deploy URL indicates it. |
| `.gitignore` gap rated MEDIUM; local TV use rated MEDIUM | auditor | Claude Code specialist | **Downgraded** to LOW (hygiene) and INFORMATIONAL (inference) respectively. |
| Workspace gate rated MEDIUM | auditor (agreeing with attack-surface assessor) | application security reviewer | **Downgraded to LOW.** The gate protects a browser-local tool and a dataset already public in the repository. |
| TV dumps rated LOW | auditor | credential and privacy specialists | **Raised to MEDIUM.** Confirmed, public, current, persistent hardware identifiers of a device in the owner's home, plus ADB-over-network enabled, requiring history rewrite to remove. |
| PostHog key rated LOW | auditor | credential specialist | **Set to INFORMATIONAL** (public by design). |
| Netlify and Stripe listed as in use | lead | CISO | **Relabelled.** Netlify: configured in Thrive, use unconfirmed. Stripe: integrated in Thrive code, live use unconfirmed. Others: referenced in docs only. |

## 4. What the auditor checked that nobody else had

| Check | Result |
|---|---|
| Entropy scan of all string literals of 20+ characters above 4.3 bits, excluding URLs and hashes | 825 strings, all identifiers, alphabets or constants; the only key-prefixed string is the PostHog project key |
| Decoding every base64 run of 40+ characters and searching for key, token, password, BEGIN, postgres | 0 hits |
| Removed lines in every diff (`git log -p --all`) | only the fake test JWT |
| Git notes, stash refs, odd refs, loose objects, author fields, commit messages | clean; 931 Claude session URLs in public trailers (identifiers, not secrets) |
| `.env` content committed under another filename (lines shaped `KEY=value`) | 88 hits, all shell variables, URLs or placeholders |
| Both pull-request bodies | clean |
| Committed build output (`dist/`), source maps, lockfile registry tokens (15 lockfiles), `tsbuildinfo`, Playwright auth state, HAR, traces, database files | none leak anything |
| The APK committed by TV's workflow | no embedded URLs or keys; debug-signed |
| `tests/.walk.tmp.ts` and `docs/CHALLENGES-MESSENGER.txt` in Jezza | clean |
| A YouTube release ledger that would link GitHub to the channel | does not exist; no owner video IDs anywhere |
| Forks, stars, watchers on all 33 repositories | zero, limiting the already-copied exposure window |

## 5. Round 3: sign-off decisions

The lead investigator made the final call on each material finding after reviewing the evidence cited by all partners. Where partners disagreed, both views are recorded.

| Final ID | Finding | Sensitive? | Actually exposed? | Who can access | Historical or ongoing | Credential usable? | Evidence of misuse | Final severity | Confidence |
|---|---|---|---|---|---|---|---|---|---|
| F-00 | No genuine credential ever committed to any repository | n/a | no | n/a | n/a | nothing to use | none | clean | High |
| F-01 | Third-party business dossier with critical commentary and about seven named individuals, in public repo and client bundle (Reputation-Operations) | yes (third-party; commercial and reputational) | yes | anyone | ongoing since 2026-11-02 | n/a | none | MEDIUM (auditor: MEDIUM to HIGH) | High |
| F-02 | Owner's TV serial, android_id, Bluetooth MAC and name, home LAN layout, apps, ADB enabled (TV) | yes (persistent device identifiers) | yes | anyone | ongoing since 2026-10-23 | n/a | none | MEDIUM (raised from LOW) | High |
| F-03 | Tyrants Supabase room policies `using (true)` for anon | design flaw | yes (public code; anon key public by design) | anyone with the app's anon key | ongoing | n/a | none | MEDIUM (game data) | High |
| F-04 | Jezza party read and photo URLs by 4-character code, unauthenticated; a real code published in a screenshot | yes (third-party photos) | yes, while the server runs | anyone | ongoing if server up | n/a | none | MEDIUM (conditional) | High |
| F-05 | Jezza deploy on any branch with a long-lived broad cloud key | design | key value not exposed | repository writers; CI | ongoing | unknown whether key exists | none | MEDIUM (hardening) | High |
| F-06 | Real executives presented as product team in private repo with public deployment URL (intelligence) | yes (third-party identities, consent) | repo private; deployment unverified | unknown | ongoing | n/a | none | MEDIUM (uncertain accessibility); LOW if protected | Medium |
| F-07 | Unauthenticated LLM proxy routes with per-instance limits (Trading, hello, Signal, Reputation-Operations) | cost | yes in code; keys deployed unknown | anyone | ongoing | n/a | none | MEDIUM if keys live, else LOW | Medium |
| F-ID | Single identity controlling every platform; MFA state unknown | structural | not an exposure | n/a | ongoing | n/a | none | MEDIUM (CISO after Round 2) versus HIGH (auditor); signed off MEDIUM with HIGH impact if realised | Low on current state |
| F-08 | PostHog project key in Hold-My-Loot | public by design | yes | anyone | ongoing | usable only to inject events | none | INFORMATIONAL | High |
| F-09 | Owner's Instagram analytics screenshots and CSVs on a side branch (The-Hidden-World) | mildly | yes | anyone | ongoing on branch | n/a | none | LOW | High |
| F-10 | Friend nicknames, event region and past date (Jezza) | mildly | yes | anyone | ongoing | n/a | none | LOW | High |
| F-11 | Named real employer or client in a product brief (Signal) | mildly | yes | anyone | ongoing | n/a | none | LOW | Medium |
| F-12 | Real client businesses' public contact data in mock-ups (BNA-HVAC, CB-Accounting) | public business info | yes | anyone | ongoing | n/a | none | INFORMATIONAL | High |
| F-13 | TV workflow with `contents: write` pushing a debug-signed APK | design | yes | n/a | ongoing | n/a | none | LOW | High |
| F-14 | Claude session URLs in 931 public commit trailers | identifiers | yes | anyone (links open only for the owner) | ongoing | no | none | INFORMATIONAL | High |
| F-15 | Docs and one vendored skill instruct inline-secret commands (placeholders in git) | pattern | placeholders only | n/a | ongoing | n/a | none | LOW (public); local and transcript implication | High |
| F-16 | No repository ignores `.claude/settings.local.json`; several do not ignore `.env` or `.vercel` | hygiene | no exposure | n/a | ongoing | n/a | none | LOW | High |
| F-17 | Workspace gate token is an unkeyed 32-bit hash (Reputation-Operations) | design | yes | anyone | ongoing | n/a | none | LOW | High |
| F-18 | Possible secrets in the owner's local Claude Code settings or transcripts, and in web-session transcripts | unknown | not observable | owner's machine; service | unknown | unknown | unknown | UNABLE TO DETERMINE; precautionary rotation list provided | n/a |

No partner found evidence of actual compromise or misuse. Every use of "rotate" in the remediation plan is precautionary and labelled as such.

## 6. Disagreements left on the record

1. **Identity concentration:** the auditor would rate it HIGH as a governance risk; the CISO (after challenge) and the lead rate it MEDIUM because the engagement scale ties severity to confirmed exposure and the current MFA state is unknown. All agree the impact if realised is estate-wide and that phishing-resistant MFA is the single most valuable control.
2. **Reputation-Operations dossier:** the privacy specialist separates a LOW personal-data component from a MEDIUM reputational one; the auditor would merge them at MEDIUM to HIGH. All agree it is the first data item to take private.
3. **TV dumps:** the credential and privacy specialists rated LOW; the auditor and lead raised to MEDIUM. The disagreement is about whether device identifiers count as "meaningful personal information"; nobody disputes the facts.

## 7. Auditor's statement on completeness

The auditor's own enumeration matched the lead's (13,584 blobs; 11,105 text blobs versus 11,079 to 11,107 depending on size cut-off), confirmed the absence of hidden refs, notes, stashes and loose objects, and found no credential by five methods the other partners had not used. The auditor's additional checks (build output, source maps, lockfile tokens, Playwright state, APK contents, PR bodies, release ledger, more screenshots) found one new LOW item (the published party code) and no new credential or personal-data exposure. The auditor signed the register in section 5 subject to the limitations in `AUDIT_LIMITATIONS.md`.
