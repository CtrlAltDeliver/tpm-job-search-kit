# TPM Toolkit — Starter Kit

An AI-run job-search and interview-prep system for Technical Program Managers.
This is a **starter kit**: the folder structure, the skills, and templates you
fill with your own material. The skills are only as good as the files they read,
so most of the work on day one is setting up *your* content — not the tooling.

> **The one-sentence mental model:** the skills are the last 10%. Your files —
> resume, story bank, company research, trackers — are the 90%. Set those up
> first and the skills come alive. Skip them and `/apply` and `/interview-prep`
> are just generic prompts.

## What's in the box

| Piece | What it does |
|---|---|
| `.claude/skills/apply/` | `/apply` — finds fresh TPM roles, scores them against your rubric, dedups against what you've already applied to, and hands you the top 3 worth your morning (with the story to lead with). |
| `.claude/skills/interview-prep/` | `/interview-prep` — runs a full mock interview round tailored to the actual interviewer, asks questions cold, coaches after each answer, and tracks your weak spots across sessions. |
| `Job-applications-TPM/` | Your application pipeline — trackers, per-company folders, and the discovery engine. |
| `TPM-Stories/` | Your STAR story bank. The mock interview reads this. |
| `company-context/` | Per-company research files (`/apply` and `/interview-prep` both read these). |
| `mock-interviews/` | Feedback from past mock rounds, so the coaching improves over time. |
| `me/` | Your resume and personal goals. |

## Related repos

This kit is the **front door** of a three-repo set — start here, then pull the
other two in as you go:

| Repo | What it is | How it fits |
|---|---|---|
| **[apply-pipeline](https://github.com/CtrlAltDeliver/apply-pipeline)** | The parallel job-discovery engine (Python). | The `/apply` skill here *runs* it. Copy its files into `Job-applications-TPM/` (setup step 4). |
| **[tpm-toolkit](https://github.com/CtrlAltDeliver/tpm-toolkit)** | The wider set of TPM slash-command skills — `score`, `tailor`, `companyresearch`, `referrals`, `interview-prep`. | Drop any of these into `.claude/skills/` alongside `apply` and `interview-prep` for the full workflow. |

## What you need

- **[Claude Code](https://claude.com/claude-code)** — the skills (`SKILL.md` files) are a Claude Code construct. The discovery *engine* is plain Python and runs anywhere, but the routines assume Claude Code. Reading/writing the `.xlsx` trackers and creating `.docx` job-description files use the **`xlsx` and `docx` skills that ship with Claude Code** — nothing extra to install.
- **Python 3.8+** — for the discovery engine (`pip install pyyaml`).
- **Your own connectors, optionally** — these are per-user integrations you enable in *your* Claude Code, authenticated to *your* accounts. There is nothing to download here, and nobody ships theirs to anyone — the skill detects whether each is present and **skips those steps gracefully** when it isn't. `/apply` runs fine without them (discovery + scoring + trackers); connecting them just lights up more:
  - **Gmail** — the inbox sweep (auto-updates application statuses) and the daily summary draft.
  - **Browser (Claude-in-Chrome)** — interviewer LinkedIn lookup and referral search.

> **Why there are no connector "files" in this repo:** an MCP connector is an
> OAuth link to a specific account. It can't be packaged and shipped — and you
> wouldn't want it to be (that would hand your inbox and LinkedIn session to
> anyone who cloned it). Everything that *is* a file is already in these repos;
> connectors you add yourself, once, in your own environment.

## Day-one setup (do these in order)

The order matters. Don't drop the skills in first — set up your content, *then*
point the skills at it.

### 1. Put this folder where you want it and point Claude at it

Clone or copy this whole folder to your machine (e.g. `~/Desktop/context-directory`).
Open Claude Code with this folder as the working directory. The included
`CLAUDE.md` tells Claude how the pieces fit together.

### 2. Add your resume

Drop your latest resume PDF into `me/`. See [`me/README.md`](me/README.md).

### 3. Pull in your existing material

Let Claude gather what you already have scattered around your machine. Run this
prompt:

> "Look through my Desktop, Documents and Downloads folders and find files that
> might be useful context about me or my work — resume, mock interview feedback,
> past STAR stories, job descriptions I've saved. Tell me what they are and ask
> before moving anything. Resume goes in `me/`, mock-interview feedback in
> `mock-interviews/`, and any STAR stories in `TPM-Stories/`."

### 4. Drop in the discovery engine

Copy the Python files from the
[apply-pipeline repo](https://github.com/CtrlAltDeliver/apply-pipeline) into
`Job-applications-TPM/`. You want both halves:

- **Discovery:** `discover.py`, `title_filter.py`, `normalize.py`,
  `linkedin_fallback.py`, `config.py`, `ats-targets.yaml`.
- **Folder/tracker layer** (what steps 2 and 5 of `/apply` call):
  `read_jds.py`, `dedup_check.py`, `purge_opportunities.py`.

Then install deps (PyYAML for discovery, openpyxl for the tracker layer — both in
`requirements.txt`) and smoke-test:

```bash
cd Job-applications-TPM && pip install -r requirements.txt && python3 discover.py --pretty
```

### 5. Tune your rubric

Two files decide what counts as a good role for *you* — edit both:

- `Job-applications-TPM/config.py` (from the pipeline repo) — location rules,
  salary floor, freshness window.
- `.claude/skills/apply/SKILL.md` → the **Fitment rubric** section — your
  must-haves, strong signals, and anti-signals. The shipped version is marked
  with `# EDIT` where you should substitute your own targets (location, comp,
  domains, seniority).

### 6. Seed your trackers and story bank

- Rename `Application tracker.template.xlsx` → `Application tracker.xlsx` and
  `TPM opportunities.template.xlsx` → `TPM opportunities.xlsx` (drop the
  `.template`).
- Copy `priorities.example.md` → `priorities.md` and list your target companies.
- Write at least 2–3 real STAR stories into `TPM-Stories/` and index them in
  `_index.md`. See the example story and `_index.template.md`.

### 7. Try it

```
/apply
```
```
/interview-prep <Company>
```

## What's shipped vs. what you build

Everything here is a **template**. None of it contains anyone else's data. The
discovery engine works out of the box; every other step gets better the more of
*your* material you feed it. Start small — a resume and three stories is enough
to see value on day one — and grow the story bank and company-context files as
you go.
