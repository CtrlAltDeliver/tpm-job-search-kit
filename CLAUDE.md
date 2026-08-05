# Project instructions

This is a personal TPM job-search + interview-prep workspace. The AI reads the
folders below to run two routines: `/apply` (find and score roles) and
`/interview-prep` (mock interviews). Edit this file to match how *you* work.

## Folder map

- `me/` — your resume (latest PDF) and `goals.md`. The routines score roles and
  pick stories against these.
- `Job-applications-TPM/` — the application pipeline:
  - `Application tracker.xlsx` — roles you've applied to and their status.
  - `TPM opportunities.xlsx` — backlog of roles found but not yet applied to.
  - `priorities.md` — your ranked target companies.
  - `Pending-applications/<Company>/` — roles queued to apply to (drop your
    tailored resume in the folder and the next `/apply` run promotes it).
  - `ats-targets.yaml` + the discovery Python scripts (from the apply-pipeline repo).
- `Rejected/<Company>/` — roles that were rejected (post-application).
- `company-context/<slug>.md` — per-company research (one file per company).
- `TPM-Stories/` — your STAR story bank + `_index.md`.
- `mock-interviews/` — feedback from past mock rounds.
- `daily-notes/` — optional running log, one file per day.

## Conventions

- **Resume:** always read the newest `*Resume*.pdf` in `me/` — don't hardcode a
  filename.
- **Timezone:** timestamps in daily notes use your local timezone. Set yours
  here: `# TIMEZONE = America/Denver` — edit to your own.
- **Never fabricate.** Don't invent metrics, stories, or role details. If a story
  is missing a data point, flag it and ask — don't make one up.
- **Don't pad.** `/apply` returns *fewer* than 3 roles rather than padding with
  mediocre ones.

## Reminders / to-dos (optional)

If you want the AI to track your to-dos, keep them in a `reminders.md` at the
root, organized by urgency. Capture proactively when you mention a task.
