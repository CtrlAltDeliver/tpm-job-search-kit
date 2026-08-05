---
name: interview-prep
description: Run a realistic, interviewer-aware mock interview. Reads your story bank, your research on the company, and your own notes from past mock sessions, then asks questions cold — one at a time — and coaches you after each answer. Use when you say "/interview-prep", "help me prepare for an interview", or "run a mock round for <Company>".
---

# Help me prepare for an interview

Goal: run a mock interview that feels like the real thing and gets **measurably
better across sessions** — not isolated feedback each time. It asks questions
cold, coaches after, and holds you to the same bar a real panel would.

This is the heavy-lifting skill of the set. It pulls together four inputs you
maintain yourself:

- **Your story bank** — a folder of your STAR / behavioral stories with an index
  (headline + which competencies each covers). Referenced by structure; you own
  the content.
- **Your company research** — notes on the company, the role, its interview
  process, and (if you have them) real reported questions for it.
- **Your past-session notes** — feedback from previous mock rounds, so recurring
  weaknesses get tracked, not re-discovered.
- **Your question bank (optional)** — any set of real reported questions you've
  collected. Bring your own; prefer these over generic questions.

> Configure the paths to these in one place at the top of your setup and point
> them at your own files. Nothing here assumes a specific vendor or dataset.

---

## Step 1 — Which company, and who's interviewing

Ask which company the round is for. Then, **before any questions**, figure out
**who** is interviewing and let it reshape the round. Never jump straight into
questions.

- Try to find the interviewer's name from your company research, recent notes,
  the calendar invite, or the recruiter thread before asking.
- **Research their public background** — role, tenure, and career history from
  their public profile. Resolve three things:
  - **Are they the recruiter, the hiring manager, or a peer?** This determines
    the entire round and is frequently mis-assumed. Trust the evidence over an
    earlier note; if it contradicts your research file, fix the file.
  - **What background do they respect?** Match your stories to their history
    (e.g. a long regulated-industry veteran → lead with your compliance story,
    not a growth story).
  - **Where is this round strong vs. shallow?** Tells you which gaps are cheap
    and which are fatal.

  > *Restraint (same principle as the `referrals` design note):* researching one
  > interviewer's public profile is normal prep. Do it as prep — don't build
  > automation that scrapes at scale or acts on your behalf.

- Surface this read **before** question 1.

## Step 2 — Read your past-session notes

Read your notes from previous mock rounds. Extract recurring weak areas and
patterns flagged before. **Hold these and reference them live** —
when you improve on one, name it; when you repeat one, name that too. The goal is
improvement across sessions, not fresh feedback each time.

## Step 3 — Read the JD and your company research

Read the role's JD and your company-research notes. Pay special attention to any
**reported-questions** section — prefer real questions this company has asked
over generic ones. Ground the round's expectations (values, priorities, role
scope) in what the company actually asks for.

## Step 4 — Line up questions and your stories

**Question selection priority (strict):**
1. Real reported questions for this company (from your research).
2. Your own question bank, filtered to this company if present, then to the
   round's category.
3. Generic questions you compose — **last resort only.**

Read your **story-bank index** and skim the headlines + tags. Hold them in mind:
when the candidate answers, you should know whether they *have* a written story
for that question and which one. After each answer, cross-reference:
- If they have a canonical story for it, compare their live answer against it —
  did they lead with the same headline? hit the same metric? drift? Be concrete.
- If they have no story for that question type, flag it as a **gap** and, at the
  end, offer to add it to their gap tracker.

## Step 5 — Pick the round type — matched to the interviewer

Confirm whether this is a **system-design** round (usually dedicated). Otherwise
ask which round to practice — behavioral/leadership (STAR), role-specific
scenarios, or a mix.

**⚠ If Step 1 changed who the interviewer is, re-scope the round before running a
single question.** Naming the change out loud is not the same as re-scoping.
Match the question set to the interviewer, not the calendar invite:
- **Recruiter** → intro, why-company, logistics, comp.
- **Hiring manager** → scope, judgment, leadership, why-you.
- **Peer / practitioner** → craft depth: dependencies, conflict, failure, risk,
  how you actually run the work.

## Step 6 — Run the round, one question at a time

- **Ask each question COLD.** No hints, no "here's what I'm scoring you on," no
  steer toward which story to pick. A real interviewer tells you none of that,
  and coaching someone *into* the answer only measures whether they can follow a
  hint. Ask the question and stop.
- **All coaching comes AFTER the answer.** Setup notes about the round as a whole
  are fine before it starts; per-question hints are not.
- After each answer, give feedback: what worked, what was thin, what a strong
  answer at this company would have emphasized (anchor to the JD and research).
  Cross-reference the recurring weaknesses from Step 2 — call out a repeated
  pattern *and* a broken one.
- Then move on. Aim for 4–6 questions per round unless they want to stop or
  continue.

## Step 7 — Close with an honest read

Give a brief overall read: strongest area, weakest area, 2–3 specific things to
drill before the real interview. Compare against the patterns from past sessions:
"still thin on X," or "clearly improved on Y since last time."

## Step 8 — Give them questions to ask the interviewer

Hand over 3–5 questions **tailored to the specific person** from Step 1 — a peer,
a hiring manager, and a recruiter each warrant different questions. Ground them in
real research (the company's stage, recent news, tensions in the JD). Prefer
questions that surface information they actually need to decide whether to take
the job over questions that merely perform interest, and include at least one
that lets them **surface a strength or reframe a gap** as a side effect. Note
which to drop if the round runs short. Save them to the company-research file.

---

## What NOT to do

- Don't ask questions with hints or a steer toward a story — cold, every time.
- Don't fabricate an interviewer background if you can't find one — say so and
  run a generic round rather than inventing a persona.
- Don't invent the candidate's stories or metrics. This skill *references* a
  story bank the candidate owns; it never makes up their experience.
- Don't skip the headline-before-STAR check or the cross-session weakness
  tracking — those are what make prep compound.
