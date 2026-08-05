# Job-applications-TPM/

Your application pipeline. Contents:

## Trackers (rename off `.template` before first use)

- **`Application tracker.xlsx`** — one row per role you've applied to. Columns:
  `Company`, `Title`, `Link`, `Date applied`, `Status`. `/apply`'s email sweep
  updates `Status` from your inbox.
- **`TPM opportunities.xlsx`** — backlog of roles found but not yet applied to.
  Two sheets: `Opportunities` and `Rejected opportunities`. You mark
  `Worth Applying` (Y/N) and `Valid?` (Y/N); `/apply` promotes the Y/Y rows into
  `Pending-applications/`.

## `priorities.md`

Your ranked target companies. Copy from `priorities.example.md`.

## The discovery engine (from the apply-pipeline repo)

Drop these in from <https://github.com/CtrlAltDeliver/apply-pipeline>:
`discover.py`, `title_filter.py`, `normalize.py`, `linkedin_fallback.py`,
`config.py`, `ats-targets.yaml`, `requirements.txt`.

`ats-targets.yaml` maps each target company to its ATS platform + slug. Start
from `ats-targets.example.yaml` here, then expand it.

## `Pending-applications/<Company>/`

The staging area. `/apply` creates a folder here per role worth applying to,
with the JD inside. When you've tailored your resume for it, drop the resume file
(filename containing `Resume`) into the folder — the next `/apply` run promotes
the folder up to `Job-applications-TPM/<Company>/` and marks it applied.
