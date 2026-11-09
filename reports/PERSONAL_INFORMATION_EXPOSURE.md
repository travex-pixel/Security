# Personal Information Exposure

**Scope:** every blob in every ref of all 33 repositories (30 public, 3 private), image metadata for all 2,146 image blobs, commit messages, branch names, deleted files and pull-request text.
**Redaction rule:** this report describes categories, locations and counts. It does not reproduce names, addresses, phone numbers, identifiers or the owner's inferred personal details beyond city level.

## 1. Headline

**The account owner's own core personal data (real full name, personal email, phone number, home address, date of birth, identity documents, financial accounts, family names) was not found anywhere in any repository.** NOT IDENTIFIED IN THE AUDITED SCOPE.

What is exposed is secondary and third-party information, all in public repositories and all still present at the HEAD of some branch:

| ID | What | Where | Severity | Confidence |
|---|---|---|---|---|
| PI-01 | Researched dossier on 50 real small businesses with critical commentary, sales scripts, and a handful of named owners | Reputation-Operations (public) | MEDIUM reputational / LOW personal-data | High |
| PI-02 | Owner's home TV hardware identifiers (serial, android_id, Bluetooth MAC and name), home network layout, installed apps, locale and timezone | TV (public) | LOW | High |
| PI-03 | Real social-event details and friends' nicknames in a party game | Jezza-Secret-Mission (public) | LOW | High |
| PI-04 | Owner's Instagram Insights screenshots and reel analytics, with phone model in EXIF | The-Hidden-World (public, side branch only) | LOW | High |
| PI-05 | Real executives of a real advisory firm presented as the team behind a product, with headshots, including one person not on the firm's public page | intelligence (private repo; public Vercel URL) | LOW (MEDIUM if the deployment is live and unprotected) | Medium |
| PI-06 | Two real small businesses' public contact details and two tradespeople's first names in site mock-ups | BNA-HVAC, CB-Accounting (public) | INFORMATIONAL | High |
| PI-07 | A real employer or client company named in a product brief | Signal (public) | LOW | Medium |
| PI-08 | True-crime scripts about real cases naming real people from public records | The-Hidden-World (public) | INFORMATIONAL | High |

## 2. What a stranger can infer about the owner (aggregated, inference only)

- Probable first name: a nickname used as the default host in a party game's test fixtures, consistent with the account handle. Medium confidence.
- City: Sydney, Australia. From locale and timezone in the TV dumps, Sydney-time references in several projects, and the geography of all researched businesses. High confidence at city level; suburb not determinable.
- Devices: a specific Samsung phone model (from screenshot EXIF software tags) and a specific TCL Android TV in a "Family Room" (from dumps).
- Social circle: a friend group known by nicknames, one of whom had a bucks weekend in a named wine region on a date in late October 2026 (now past) and attended a named university.
- Work: a product brief names a real Australian company as the user's employer; a private repository names a real advisory firm. Which of these is the owner's employer is not stated.
- Side project: a true-crime short-video channel; the handle never appears in text or in any inspected screenshot.
- Commit identities: all no-reply addresses. No personal email anywhere.

None of this is a direct identity leak, but together it narrows the owner's identity considerably for anyone who already knows the handle.

## 3. Findings in detail

### PI-01 — Third-party business dossier published in a public repository
- Repository: Reputation-Operations (public), default branch at HEAD. Files: `research/prospect-list.csv` (50 rows, 24 columns), `research/prospect-list-v2.csv` (50 rows, 32 columns), `src/lib/data/prospects.seed.json` (same 50 records, imported by a client-side module and therefore compiled into the public JavaScript bundle of the workspace page), `research/prospect-audits/01..15-*.md` (business named in each filename), `research/raw-prospect-notes.md`, `research/prospect-intelligence-notes.md`, `business/call-brief-first-three.md`. Added 2026-11-02.
- Content: real business names, suburbs, websites, review ratings and counts, "recurring complaints", "cx weaknesses", "potential reputation problems", "ability to pay" style scoring, personalised cold-call openers. Exact counts from the Round 2 re-check: rows naming an identifiable person: 1 (v1) and 5 (v2), plus four to five names in the call brief and one audit, all taken from published sources; rows with any phone number: 1 (v1) and 3 (v2), all business landlines or 1300 numbers; rows with a personal email: 0; rows with a mobile number: 0; rows with a business premises address: 1 (v1) and 14 (v2); reputation-problem statements naming an individual rather than a business: 0. Several ABN-pattern matches in a name-clearance note.
- Accessibility: public repository, current and historical. The in-app passcode gate does not stop the seed JSON being served as a static chunk if the site is deployed (deployment not verified).
- Validity: genuine desk research about real businesses, by the repository's own notes.
- Consequence: a downloadable, ranked, critical dossier on 50 identifiable small businesses with sales scripts, published under the owner's account next to a document promising a "compliant cold-contact approach". Personal-data content is limited (business contact details and a few owner names from published sources), so the Privacy Act exposure is modest; the reputational and commercial exposure for the owner if a listed business finds it is the larger risk. This is not legal advice.
- Severity: MEDIUM (reputational/commercial), LOW (personal data). Confidence: High.
- Action: make the repository private or move `research/`, `business/call-brief-*` and the seed JSON out of the public repository and purge history; strip named individuals from retained research; if the site is live, confirm the workspace bundle is not served to anonymous visitors.

### PI-02 — Owner's TV device dumps
- Repository: TV (public), default branch at HEAD. Files: `backup/20261023-003417/getprop.txt` (device serial on two lines), `backup/20261023-003417/settings-secure.txt` and `discovery/20261023-000534/settings-secure.txt` (line 8 `android_id`, line 15 `bluetooth_address`, line 16 `bluetooth_name`), `settings-global.txt`, `packages-user.txt`, `backup/manifest.json`, `discovery/*` (ADB target addresses on the home network), `docs/03-device-report.md`. Commits 1a47d05, 6f4a099, 5f3662c (2026-10-22/23, made from the owner's local machine).
- Content: full hardware serial, `android_id`, Bluetooth MAC address, Bluetooth name describing a room in the home, two private LAN address ranges, ADB over Wi-Fi enabled, developer settings enabled, non-market installs enabled, full installed and disabled app lists (17 user apps including streaming subscriptions), build fingerprint showing Android 9 with a 2021 security patch, locale and timezone. A home-screen screenshot shows a Google account avatar initial but no email.
- Not present (verified): Wi-Fi network name or password, Google account email or tokens, ADB private key, public IP address. The base64 `anomaly_config` value is standard Google services configuration, not a secret.
- Accessibility: public, current. Validity: genuine (the owner's own device per the docs).
- Consequence: persistent device fingerprinting, household profiling (subscriptions, viewing apps), a ready reconnaissance package for anyone who reaches the home network (an unpatched TV with ADB over TCP enabled), and serial-number material usable in warranty or support social engineering. Not remotely exploitable from the repository alone.
- Severity: LOW. Confidence: High.
- Action: remove `backup/` and `discovery/` from the public repository (or make the repository private) and rewrite history for the serial, `android_id` and MAC; disable ADB over network on the TV when not in use. Note the upstream `cleanyourtv` project author named in its licence is a third party, not the owner.

### PI-03 — Real social event and friends' nicknames in a party game
- Repository: Jezza-Secret-Mission (public), at HEAD. Files: `README.md`, `docs/CHALLENGES-MESSENGER.txt`, `docs/MISSIONS.md`, `shared/missions.ts`, `shared/themes.ts`, test fixtures and simulations with arrays of nicknames, one commit body naming a scorer, one commit body describing a real birthday-party anecdote. A GitHub Pages demo is published.
- Content: the honouree's nickname, his university, the wine region of the weekend and its date, 16 "lore" challenges referencing his school, work, old nicknames, first car and middle name (as prompts, not answers), and a dozen friends' nicknames. No surnames, no venue name or address, no real photos (live screenshots show only test nicknames and a placeholder image), no state or photo files ever committed. The server stores photos on a hosting volume, never in git.
- Accessibility: public, current and historical. Event date has passed.
- Severity: LOW. Confidence: High.
- Action: optional. Replace real nicknames in fixtures with generic ones before further promotion; ensure the hosted data directory (selfies, "cellar" photos) is deleted as the privacy page promises.

### PI-04 — Owner's Instagram analytics screenshots and reel data
- Repository: The-Hidden-World (public), branch `ccr-67645e21-nwgkaq` only (also the pull/1 refs); not on the default branch. Files: `analytics/screenshots/*.jpg` (118 files), `analytics/screenshots/MANIFEST.txt`, `analytics/reels.csv`, `analytics/reels_summary.csv`, `analytics/SCOREBOARD.md`, `analytics/transcripts/*.json`, `analytics/postmortems/*.md`.
- Content: 112 Samsung phone screenshots of "Reel insights" sub-screens (views, like/skip/share rates, audience age bands, follower share), with EXIF software tag and original timestamps retained; no GPS. Status bars show only icon-level information (messaging apps, Wi-Fi, battery, one in-progress call timer). 14 screenshots were viewed across two rounds: none shows the account handle, display name, profile photo or follower count. All 118 transcripts: no handle, audience countries and cities null. Six files named `.jpg` are 52 to 64 second AAC audio tracks of episode narration (not voice memos).
- Accessibility: public, branch-only. Validity: genuine.
- Consequence: links the GitHub identity to a specific small creator channel's performance data and the owner's phone model and analytics-checking times. Competitive intelligence; weak routine inference.
- Severity: LOW. Confidence: High.
- Action: optional. Strip EXIF from screenshots or keep raw analytics out of public branches; rename the six audio files.

### PI-05 — Real executives presented as a product's team in a private repository with a public deployment URL
- Repository: intelligence (private), at HEAD. Files: a TypeScript module listing 11 named people with titles and credentials, `public/team/*.jpg` (11 headshots, no EXIF), a "why this firm" document, downloadable decks and spreadsheets "Prepared for [firm] Review, Prepared by [initials]". GitHub metadata records a public Vercel homepage; no public repository links to it.
- Content: names, roles and biographies of 10 executives of a real Australian advisory firm (public on the firm's own site, per the repository) plus one person described as "Director of AI" who is not on the firm's published team page, with his photo. They are framed as the actual proposed delivery team and owner of the product, not as characters in the repository's fictional project data (which is clearly labelled fictional).
- Accessibility: repository private; deployment reachability not verified (no requests were sent).
- Consequence: a web page presenting a firm's leadership as endorsing or owning a proposal they may not have approved; use of one person's photo beyond what the firm publishes. Reputational and misrepresentation exposure for the owner.
- Severity: LOW as found; MEDIUM if the deployment is confirmed live and unprotected. Confidence: Medium.
- Action: check the Vercel project; enable deployment protection or take it down until the firm consents; remove the eleventh person's photo unless agreed; keep the repository private.

### PI-06 — Real businesses' public contact details in site mock-ups
- Repositories: BNA-HVAC and CB-Accounting (public, `main`, 2026-10-08), one `index.html` each.
- Content: each business's street address, a mobile number (repeated 21 times in one page), an info@ email, legal entity name, a trade licence number, and two technicians' first names. All copied from the businesses' public websites (one page says "verified against directories").
- Severity: INFORMATIONAL. Confidence: High.
- Action: none required; consider labelling the pages as unofficial redesign concepts.

### PI-07 — Named real company as the user's employer in a product brief
- Repository: Signal (public), `PRODUCT.md` (2026-10-29 to HEAD): "a senior commercial leader at [named Australian distributor]… opens the product most mornings"; design tokens derived from that company's branding. Also a public call brief in Reputation-Operations describing the founder as "a mate".
- Severity: LOW. Confidence: Medium.
- Action: optional; genericise the company name if the association should not be public.

### PI-08 — True-crime content about real people
- Repository: The-Hidden-World (public), scripts and research for about 100 real criminal cases naming perpetrators and sometimes victims; one historical email of a convicted person quoted from a public court exhibit; research notes mark unsourced names as "do not use".
- Severity: INFORMATIONAL (editorial sensitivity, not privacy). Confidence: High.

## 4. Checked and found clean

- **Thrive** test fixtures named like a payslip, a bank statement, a HEIC photo, a Bali holiday image and a relationship statement: all generated by `tests/fixtures/generate.py`; the photo is a synthetic colour field with a test caption; PDFs are blank library output.
- **VisaFly** embedded "passport", "birth certificate", "national ID" and "police check" documents: 2 KB synthetic PDFs produced by a report library, and a specimen ID image watermarked as fictional. Clean.
- **Bin-Night**: a game; no address, council, suburb or bin schedule anywhere in history.
- **Trading**: synthetic market data; no broker, account or portfolio data.
- **fruit** (private): animation project; only third-party font-licence emails.
- **hello**, **hello-deployed** (private), **Coverwise**, **Tyrants**, **Hold-My-Loot**: schemas only, no seeded personal rows; review screenshots show fictional cartoon characters.
- **NBA-fantasy-**: public league data only. **Signal** fixtures: public RSS feeds.
- Small prototypes (NewApp, SleepMax, Servo-Empire, Moss-and-Moon, Colony-Go, How-Good, Crush-Gear, Buckets, Bucket-League, Callup, Yeet-Keep): code only.
- Consumer email domains across all blobs: one hit, a third-party address quoted from a public court exhibit. No owner personal email.
- EXIF across all 2,146 image blobs: GPS in none; camera make or model in none; only the Android software tag and timestamps on the Instagram screenshots and three synthetic date tags on the Thrive fixture.
- First-person date-of-birth, birthday or family-member statements: none.
- Deleted files: none carried personal data.
- Jezza live screenshots and Hold-My-Loot review images: test nicknames and cartoon crews only; no real faces.

## 5. Public versus private, current versus historical

| Finding | Repository visibility | At HEAD of some branch | Historical-only |
|---|---|---|---|
| PI-01 | public | yes (default branch) | no |
| PI-02 | public | yes (default branch) | no |
| PI-03 | public | yes | no |
| PI-04 | public | yes (side branch only, not the default branch) | no |
| PI-05 | private repository; public deployment URL | yes | no |
| PI-06, PI-07, PI-08 | public | yes | no |

Nothing personal was "removed but left in history": every item is still live. Removing it will require history rewriting after the repository is made private, as set out in the remediation plan.

## 6. Limitations

- The owner's machine, browser storage, hosting data volumes and live sites were not accessible; exposure through deployed applications is inferred from code, not observed.
- Repository metadata and PR text for the 30 public repositories came only from the search API.
- Audio was not transcribed; video was not opened; not every image was viewed by eye (EXIF was read for all).
- Identity statements are inferences from nicknames, timezones and device metadata.
