---
name: apply
description: Surface the top 3 TPM roles worth applying to today. Refreshes the opportunities backlog, sweeps your inbox for status updates, finds fresh ATS-native listings, scores them against your fitment rubric, and returns only true fitment-5/4 matches with the story to lead with. Use when you say "/apply", "what should I apply to today", or "top 3 jobs today".
---

# Top 3 jobs to apply today

Goal: hand the user **three (or fewer) TPM roles genuinely worth their morning** —
verified open, product-company, matched to their actual story. **Never pad the
list.** Fewer good options is better than three mediocre ones.

This skill orchestrates the discovery engine (the `apply-pipeline` Python scripts
in `Job-applications-TPM/`) plus the tracker/folder bookkeeping around it. It
reads and writes the user's own files — nothing is hardcoded to one person.

> **Setup note:** this skill assumes the discovery engine and trackers are in
> place (see the repo README). Steps that need optional connectors — the Gmail
> inbox sweep (Step 1b) and the summary draft (Step 9) — are marked
> **[needs Gmail]**; the referral lookup (Step 6b) is marked **[needs browser]**.
> When a connector isn't available, skip that step, say so, and continue.

---

## Hard rules for Bash calls

- **Don't `cd "<path>" && …`** — the working directory is already the project
  root. Use relative paths: `bash Job-applications-TPM/run_discovery.sh`.
- **Don't inline `python3 -c "<multi-line>"`** — extend or call a real `.py`/`.sh`
  helper instead.
- Keep redirects (`>`, `2>`) inside wrapper scripts, not on the outer command.

---

## Step 1 — Load state

Load in parallel; skim, don't dump (you only need company + title + status):

1. **Resume:** newest `*Resume*.pdf` in `me/` (don't hardcode the filename).
2. **Goals:** `me/goals.md`.
3. **Applications:** `Job-applications-TPM/Application tracker.xlsx` — every row.
   Roles marked `Applied - awaiting response` are excluded from new suggestions.
4. **Priorities:** `Job-applications-TPM/priorities.md`.
5. **Opportunities backlog:** `Job-applications-TPM/TPM opportunities.xlsx`.
6. **Rejected:** `Rejected/` folder names (don't re-suggest an exact rejected role).

## Step 1a — Promote completed Pending-applications folders

Sweep `Job-applications-TPM/Pending-applications/`. For each company folder:

- If it contains a file with `Resume` in the name (any extension) → the user has
  tailored their resume and is ready. **Move the whole folder** up to
  `Job-applications-TPM/<Company>/`, then update `Application tracker.xlsx`:
  `Date applied` = today, `Status` = `Applied - awaiting response` (append a row
  if none exists, carrying `Title` + `Link` from the opportunities tracker).
- If no Resume file → leave it.

Save the tracker.

## Step 1b — Email sweep **[needs Gmail]**

Fold the whole "job application status" check into `/apply` so the tracker is
fresh before dedup. **If no Gmail connector is available, skip this step and note
it.**

1. Read the last-checked timestamp from `Job-applications-TPM/.last-status-check`
   (plain ISO timestamp) as `T_prev`. If missing, default to the earliest
   `Date applied` in the tracker. Capture the current time as `T_scan_start`.
2. Search Gmail in the window `T_prev → T_scan_start` for messages about any
   company in `Application tracker.xlsx`. Cover: auto-confirmations,
   auto-rejections, recruiter outreach, OA/interview invites, scheduling, offers,
   post-interview rejections. Search **per active company name**, scoped to the
   window; triage on subject/sender metadata first, open the body only when a
   stage change hinges on it.
3. Update the tracker:
   - **Auto-confirmation** → no status change (note only).
   - **Auto-rejection** → `Status = Auto rejection`; move
     `Job-applications-TPM/<Company>/` → `Rejected/<Company>/`.
   - **Recruiter / OA / interview / offer** → `Status` reflects the stage; folder
     stays put.
   - **Post-interview rejection** → `Status = Rejected`; move folder to `Rejected/`.
4. **Save the tracker and finish every folder move FIRST. Only then** write
   `T_scan_start` to `.last-status-check`. (Order is load-bearing: the cursor
   declares "everything up to here is processed" — advancing it before the writes
   commit means a crash silently drops those emails forever. Worst case of doing
   it last is a harmless re-scan.)
5. Capture a short summary (folders moved, statuses changed, anything needing the
   user's input) for Steps 8–9.

Re-read `Application tracker.xlsx` after 1a + 1b before continuing.

## Step 2 — Re-verify the opportunities backlog

Sweep `TPM opportunities.xlsx`:

- **Move rows where `Worth Applying = N` OR `Valid? = N`** to the
  `Rejected opportunities` sheet (append `Rejected on` = today + a one-line
  `Reason`). Renumber `S.No` on the Opportunities sheet.
- **Delete rows where `Applied = Y`** (case-insensitive). Before deleting, make
  sure the role is captured in `Application tracker.xlsx` (append a row if not).
  Renumber `S.No`.
- Rows where `Applied` is blank/`N` are left alone.

The `Rejected opportunities` sheet is **append-only** — never re-surface a role
the user already reviewed and declined. Save the file.

## Step 3 — Discover fresh roles (ATS JSON APIs)

Discovery is JSON-API driven, not search-snippet driven — going straight to each
ATS's public JSON endpoint avoids the stale-snippet problem entirely.

**Run the engine. Do not reconstruct the command:**

```bash
python3 Job-applications-TPM/discover.py --pretty --seen Job-applications-TPM/seen.json
```

(Or `bash Job-applications-TPM/run_discovery.sh` if you've wrapped it.) It loads
`ats-targets.yaml`, fetches each verified company's public JSON endpoint in
parallel (Greenhouse / Lever / Ashby / SmartRecruiters / Workday / HiBob), and
filters every role through:

- **Title filter** (`title_filter.py`) — Senior+ TPM + qualified Program/Project
  Manager (Engineering); excludes EPM, coordinator, AI-titled, non-eng domains.
- **Location + salary filter** (`config.py`) — your geographic rules and comp
  floor. **# EDIT `config.py` for your own location/salary targets.**
- **Freshness** — postings updated within the last N days (`DEFAULT_MAX_AGE_DAYS`).
- **Dedup** — against the tracker, `<Company>/` folders, `Rejected/`, the
  `Rejected opportunities` sheet, and the live backlog (via `seen.json`).

Returns a ranked JSON candidate list with embedded JD text, salary if available,
and dedup notes. **Don't layer extra WebSearch on top of this for verified
companies** — the JSON path is the source of truth. Snippets are not verification.

## Step 3a — LinkedIn fallback for companies with no clean JSON

Companies marked `status: todo` in `ats-targets.yaml` (Workday-hardened, custom
ATSes) have no clean public JSON. Cover them with the fallback:

```bash
python3 Job-applications-TPM/linkedin_fallback.py
```

It queries the LinkedIn guest-view job search for your keywords across your
region, applies the same `title_filter.py` rules, and enriches the top survivors.
These candidates carry a `linkedin` tag and feed the same dedup pipeline. They're
verified only by being a live posting within the freshness window — flag them
`⚠ Verify still open before applying`.

## Step 4 — Score against the fitment rubric

For every verified-open candidate, apply this rubric. Score is the result, not
vibes. **# EDIT every bracketed target below to match your own search.**

### Must-haves (gate — no Fitment ≥3 without ALL of these)

1. **The role is TPM work — judged by JD scope, not the title string.**
   Auto-pass on "Technical Program Manager" / "Technical Project Manager" (any
   seniority below Principal). Adjacent titles ("Senior Program Manager
   (Technical)," "Delivery Lead (Engineering)") pass **only if** the JD shows all
   three: cross-functional execution across eng teams; ownership of technical
   roadmap / dependency management; a technical-fluency bar (architecture, APIs,
   distributed systems, cloud). Hard-excluded regardless of JD: Engineering
   Program Manager (EPM), coordinator-level seats, external-client services
   delivery.
2. **Employer is a product company** — software/hardware product as primary
   revenue. Not consulting, audit, staffing.
3. **Location** — `# EDIT: your acceptable locations / remote rules`.
4. **Posting verified open in the freshness window via ATS-native URL** — no
   aggregator-only sources.

### Strong-positive signals (each pushes Fitment 4→5)

5. **Domain overlap with the resume** — `# EDIT: your domains`.
6. **Scope signal** — "X engineering teams," "drive roadmap across," named
   workstream counts. Specific numbers beat generic "coordinate."
7. **Tech-fluency expectation** — JD names distributed systems / APIs / cloud /
   system-design rounds.
8. **Salary at or above your floor** (`# EDIT`).
9. **Culture/stress signal** — async-first, public eng blog, transparent comp,
   a leadership-principles framework.
10. **Target-list company** (from `priorities.md`).

### Anti-signals (hard veto — drop to Fitment ≤2)

- **AI-centered role** — title contains AI/ML/GenAI/LLM/Agentic, OR the JD's
  primary requirement is AI/ML domain expertise. (`# EDIT/REMOVE if AI roles are
  in-scope for you.`) Soft case still OK: a company that *touches* AI where the
  TPM scope is broader.
- JD is status-meeting / portfolio-reporting work with no engineering-team
  ownership.
- Employer is consulting / audit / staffing.
- Onsite-only in a city requiring relocation you won't do.

### Rubric

| Score | Rule |
|---|---|
| **5** | All 4 must-haves + ≥5 strong-positives + 0 anti-signals + a domain bullseye |
| **4** | All 4 must-haves + ≥3 strong-positives + 0 anti-signals |
| **3** | All 4 must-haves + ≥1 strong-positive |
| **2** | Misses 1 must-have OR has 1 anti-signal |
| **1** | Misses 2+ must-haves OR multiple anti-signals |

**WebFetch budget:** cap JD page fetches (~8/run). Spend them on candidates that
would be top-3 picks or sit on a 3-vs-4 boundary. "Missing metadata" is never on
its own a reason to fetch — note `salary unlisted` and move on.

## Step 5 — Pick the top 3 (or fewer)

- Eligible pool = **Fitment 5 and 4 only**. Never surface a 3 in the daily top 3.
- Rank by Fitment, then by your location preference, then by posting freshness.
- **If fewer than 3 qualify, say so honestly.** Show the 1 or 2 you have and state
  you didn't pad. Never surface 3s to fill space.
- Dedup once more against the tracker, `<Company>/` folders, `Rejected/`, and the
  `Rejected opportunities` sheet. Exact company + normalized-title match → drop
  silently (log for audit). Same company, clearly different role → surface with
  `⚠ already applied to <Company> — verify this is a different role`.

## Step 6 — Warm-story hook

For each pick, name which of the user's flagship stories (from `TPM-Stories/`) is
the best STAR match for the cover letter / first recruiter call, and why it lands.
Read `TPM-Stories/_index.md` to match by tag. **Never invent a story** — if
nothing in the bank fits, say so.

## Step 6a — Readiness gap

For each pick, diff the JD against the resume into three short lines:
- **You clearly meet:** the 2–3 must-haves the resume already proves.
- **Gaps to address:** what the JD stresses that the resume doesn't cover.
- **Emphasize:** the 1–2 things to foreground, tied to the warm story.

Don't rewrite the resume here — that's a separate tailoring step.

## Step 6b — Who can refer **[needs browser]**

Only for picks the user has already committed to (row marked `Worth Applying = Y`)
and only if a browser connector is available. Search the user's own LinkedIn
1st-degree connections at each pick's company, read **only the people-search
result cards** (never open a profile, never solve a CAPTCHA, never enter
credentials), and write the best ask into a `Who can refer` column in
`TPM opportunities.xlsx`. If no browser is connected or LinkedIn walls, hand the
user the search link instead. Skip silently in the no-browser case.

## Step 7 — Output + tracker write

Output format:

```
## Top N to apply today (N = 1, 2, or 3)

### 1. <Company> — <Title> (<Location>, <Remote/Hybrid>)
**Fitment: 5/5** · **Apply:** <ATS URL> · **Salary:** <if known>
**Why it fits:** <must-haves cleared + strongest signals, one sentence>
**Warm story:** <which story to lead with, and why>
**You meet / Gaps / Emphasize:** <the three readiness lines>
**Who can refer:** <best ask, or "link handed", or "—">
<dedup flag if present>
```

Then one sentence on pipeline state: how many backlog rows are still live, how
many fresh ATS-verified roles you scanned, how many made the cut.

**Tracker write:** add new verified-open Fitment-4-or-5 roles to
`TPM opportunities.xlsx` (whether or not they made today's top 3 — they're future
candidates). Don't add Fitment ≤3. Continue `S.No` numbering.

## Step 7a — Promote committed rows into Pending-applications

Sweep `TPM opportunities.xlsx`. For every row where `Worth Applying = Y` AND
`Valid? = Y` AND `Applied` is blank:

- Create `Job-applications-TPM/Pending-applications/<Company>/` if absent (don't
  overwrite an existing folder).
- Inside it, create a JD document named `<Role title>.docx` (sanitize the
  filename). Source the JD from this run's `apply_candidates.json` embedded text
  if present; else WebFetch the `Link`; else write a placeholder .docx with role,
  company, location, link, salary, Fitment, and a note to paste the JD from the
  link.

## Step 8 — Daily log (optional)

If the user keeps `daily-notes/`, append a one-line summary under a
`## Daily applies` heading: `HH:MM — /apply: surfaced N roles (top picks: …).`
Use the user's local timezone (see `CLAUDE.md`).

## Step 9 — Draft a summary email **[needs Gmail]**

If a Gmail connector is available, create a **draft** (never send) to the user's
own address summarizing the picks + any pipeline updates from Steps 1a/1b/7a.
Skip if no Gmail connector.

---

## Failure modes to avoid

- **Don't trust search snippets.** Verify via the ATS-native URL.
- **Don't add EPM roles** even when adjacent.
- **Don't suggest consulting / professional-services firms** even if titled TPM.
- **Don't pad to 3.** One real Fitment-5 beats three fillers.
- **Don't re-suggest applied or rejected roles.**
- **Never fabricate** a story, metric, or JD detail.
