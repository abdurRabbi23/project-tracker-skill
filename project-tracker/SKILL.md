---
name: project-tracker
description: Sets up and maintains a lightweight, modular project-memory system (dated run_log.md, per-module logbook/NN_topic.md files, an always-current logbook/HANDOFF.md, and a CLAUDE.md rules file) for any long-running project worked on across many separate chats — thesis, codebase, research, business, or personal project. Use whenever the user wants to start tracking a new project, scaffold a logbook/run-log structure, record or log today's/this session's work, catch up on a project's status before continuing in a new chat, hand off context to a future session, or audit an existing tracking system for staleness or drift. Trigger on "set up tracking", "create a logbook", "log what we did today", "update the run log", "catch me up on this project", "onboard a new chat", or "check if my logbook is up to date" — even without those exact words, if the user is asking to record progress or restore context on a multi-session project.
---

# Project Tracker

## Why this exists

A project that spans many separate chats loses context between them unless the context lives
in files, not in chat history. The failure mode this skill prevents: reopening a project after
a break and not knowing what was decided, what was tried and abandoned, or what to do next —
forcing either a re-read of everything or a repeated mistake.

The fix is not one giant notes file (it becomes unreadable) and not memory alone (it doesn't
survive a new chat). It's a small number of files, each with one job, kept honestly up to date.

## The backbone

- **`run_log.md`** — dated, append-only, chronological. One entry per session. Answers "what
  happened, in what order." Terse, past tense, no deep reasoning.
- **`logbook/NN_topic.md`** — one file per work-stream or module (numbered in rough
  chronological/dependency order). Answers "what's the actual state of X, why, and what's next."
  This is where reasoning and decisions live.
- **`logbook/00_INDEX.md`** — the front door. One-line project description, current status,
  a table of modules with status, links to everything else. Read this first, always.
- **`logbook/HANDOFF.md`** — the one file that gets *overwritten*, never appended to. Current
  status + the immediate next action + settled decisions, short enough to paste into a fresh
  chat's first message. A stale handoff is worse than no handoff, so it must be rewritten the
  moment the next action changes — not left to drift.
- **`CLAUDE.md`** (or the project's instructions file) — rules that apply to *every* session's
  work on this project, no exceptions. This is where project-specific mandatory rules live
  (formatting standards, scope locks, a required review step before saving output) — things
  that must survive across every chat and every module, not just get mentioned once and
  forgotten.

Templates for all of these are in `assets/templates/` (listed at the end of this file). They're
starting points, not molds — field names and structure should bend to fit what the project
actually needs.

## File management

Notes alone aren't enough if the files themselves are disorganized or ambiguous — a logbook
that says "results are in the results folder" is useless if there are three folders it could
mean, or if last month's superseded run is sitting next to this month's in the same directory
with no way to tell them apart. Two conventions handle this:

**A file map, kept in `00_INDEX.md`.** A one-line-per-folder list of what each top-level
directory is for and whether it's canonical (the current, trustworthy version) or working
material. This is what lets a new chat orient in seconds instead of exploring the whole
directory tree. Keep it current the same way the module table is kept current — when a
folder's role changes, update the map in the same pass as the log entry.

**Source of truth + quarantine, for any project that accumulates generated output**
(experiment runs, data pulls, drafts, exports — anything produced more than once where later
versions can supersede or invalidate earlier ones). The rule: exactly one folder is ever the
canonical answer to "where's the current data," and it is named and pointed to unambiguously
from `00_INDEX.md`. Anything superseded, invalid, or excluded does NOT get deleted and does NOT
sit unmarked next to valid material — it moves to its own clearly-named folder (e.g.
`withdrawn_runs/`, `excluded_*/`, `deprecated_*/`) with a `README.md` in that folder stating
what's inside and why it's excluded. This is the difference between "I think this is probably
the right number" and "this is provably the right number" months later when the reasoning has
left working memory. Use `assets/templates/QUARANTINE_README.md.template` for these.

Not every project needs the quarantine half of this — a project that doesn't regenerate data
over time can skip it. Ask during bootstrap rather than assuming.

## Figure out which mode applies

### 1. Bootstrap a new project

Trigger: the user wants tracking set up for a project that doesn't have it yet.

1. Check whether `logbook/` or `run_log.md` already exist in the target folder. If they do,
   stop and tell the user — don't overwrite existing history. Offer the audit mode instead.
   Check for `CLAUDE.md` separately: an existing one is common and is NOT a reason to stop,
   but it must never be overwritten — it may hold instructions the user depends on. Merge the
   tracking-convention and mandatory-rules sections into what's already there, preserving
   every existing line, and show the user the merged result rather than replacing the file
   silently. The same caution applies to any other file the scaffold would land on: read it
   first, merge rather than clobber, and if a clean merge isn't obvious, ask.
2. If the target folder already has files or subfolders in it (the common case — most projects
   aren't bootstrapped into an empty folder), list the actual directory contents before
   interviewing. Use that listing to propose a first draft of the file map and to flag
   candidates for the quarantine convention (old/superseded-looking folders), rather than
   relying on the user to recall folder names from memory. Confirm the draft with the user
   instead of guessing silently.
3. Interview the user before creating anything. Don't guess at any of this:
   - One-sentence description of the project.
   - What are the natural work-streams? Give the user a nudge with examples suited to their
     domain (code project: setup, feature areas, testing, deployment; thesis/research:
     chapters or experiment phases; business: ops, sales, finance). Aim for 3-8 modules to
     start — more can be added later as work branches.
   - Any mandatory rules that must apply to every session's output on this project? Prompt
     with a concrete example if the user is unsure (a formatting standard, a required check
     before saving a deliverable, a scope boundary that must not be quietly crossed). It is
     fine for the answer to be "none yet."
   - Is there existing history to backfill into `run_log.md` as a "Day 1" entry, or starting
     completely fresh?
   - Confirm or correct the draft file map from step 2 (or build it fresh if the folder was
     empty).
   - Will this project accumulate generated output over time — experiment runs, drafts,
     exports, data pulls — where later versions can supersede or invalidate earlier ones? If
     yes, which folder is the canonical current-truth folder, and which existing folders (if
     any) need to move into quarantine now. If the project doesn't generate that kind of
     output, skip this.
4. Scaffold the files, using this destination mapping (all paths relative to the project root):
   - `00_INDEX.md.template` → `logbook/00_INDEX.md`
   - `run_log.md.template` → `run_log.md`
   - `HANDOFF.md.template` → `logbook/HANDOFF.md`
   - `CLAUDE.md.template` → `CLAUDE.md` (merge into an existing one rather than replacing it,
     per step 1)
   - `module.md.template` → `logbook/NN_topic.md`, one copy per module agreed in the interview
   - `QUARANTINE_README.md.template` → `README.md` **inside each quarantine folder** — the
     output file must be named `README.md`, not the template's own filename, since every
     cross-reference elsewhere in this system (the audit checks, the file-map example) expects
     to find `README.md` there.
   Substitute the interview answers into each. Use the numbering and status-icon conventions
   below.
5. Report what was created and confirm there's nothing left for the user to do manually.

### 2. Session-start context load

Trigger: a new chat is starting on an existing tracked project and the user wants Claude
caught up before doing anything else.

1. Read `logbook/00_INDEX.md`.
2. Read `logbook/HANDOFF.md`.
3. Summarize in a few sentences: current status, the active module, the immediate next action.
   Don't re-narrate the whole file back to the user — they wrote it, they know what's in it.
4. Confirm the next action is still right before acting on it. Things may have changed outside
   this chat since the handoff was last written.

### 3. Daily / session record-keeping

Trigger: the user asks to log or record today's/this session's work, or asks for tracking to
be updated after finishing a task.

1. Identify which module(s) the work touched. Ask rather than guess if it's not obvious, or if
   the work spans a module boundary that doesn't exist yet (that's a sign a new module file is
   needed).
2. Append one dated block to `run_log.md` — what happened, past tense, terse. This file is a
   timeline, not a diary; save the reasoning for the module file.
3. Update the matching `logbook/NN_*.md` file's current-state and next-steps sections. This is
   the deep-context layer — write enough that someone with zero memory of the session could
   pick up the thread.
4. If the immediate next action changed, overwrite `logbook/HANDOFF.md` — don't append to it or
   leave the old action sitting alongside the new one.
5. Update `logbook/00_INDEX.md`'s module status table if a module's status changed (e.g. moved
   from active to done).
6. If new files or folders were created, or a folder's role changed, update the file map in
   `00_INDEX.md` in the same pass — a file map that isn't kept current is worse than none,
   because it's actively misleading.
7. If the project uses the source-of-truth/quarantine convention and something generated this
   session supersedes or invalidates earlier output, move the old material into its quarantine
   folder with a `README.md` explaining why — don't leave both versions sitting in the
   canonical folder together.
8. If a mandatory rule in `CLAUDE.md` applies to the work just done, verify it was actually
   followed before considering the log entry complete — a rule that's written down but not
   checked is decoration, not a rule.

### 4. Consistency audit

Trigger: the user asks to check, audit, or clean up the tracking system.

Check for and report:
- A `HANDOFF.md` next action that `run_log.md` shows was already completed.
- A module's status in the `00_INDEX.md` table that doesn't match what the module file itself
  says.
- Module files that haven't been touched despite `run_log.md` entries that clearly reference
  work in them.
- Broken relative file references (a pointer to a file that no longer exists or moved).
- The file map in `00_INDEX.md` not matching the folders that actually exist (missing new
  ones, describing deleted or renamed ones).
- For projects using the source-of-truth/quarantine convention: more than one folder implicitly
  claiming to be canonical, or a quarantine folder without a `README.md` explaining why its
  contents are excluded.
- Violations of rules stated in `CLAUDE.md` — if a rule specifies a mechanical check (e.g. "grep
  for X must return zero"), actually run it rather than eyeballing it.

Fix straightforward drift directly. Flag anything ambiguous for the user rather than guessing
at which version is correct.

## Conventions

- Status icons, used consistently in `00_INDEX.md` and module files: `✅` done · `▶` active,
  start here · `◻` planned or reference-only · `⏳` blocked or deferred.
- Module files are numbered in rough chronological or dependency order (`01_`, `02_`, ...);
  `00_INDEX.md` is always the front door and is never itself numbered as a module.
- `run_log.md` entries are terse and past tense. If an entry is starting to explain *why* a
  decision was made, that reasoning belongs in the module file — link to it instead of
  expanding the log entry.
- Entry headings can be calendar dates (`## 2026-08-12 — ...`) or working-day counters
  (`## Day 12 — ...`); day counters read well for a project with a deadline, dates for one
  worked on irregularly. Either is fine, but pick one at bootstrap and don't mix them — module
  files cross-reference these headings, and mixed formats break that lookup.
- `HANDOFF.md` stays short enough to paste whole into a new chat's opening message. If it's
  grown past a few paragraphs, move settled background into the relevant module file and leave
  only what's still live.
- Never delete superseded or invalid material outright — quarantine it with a reason. Deleting
  destroys the ability to answer "why isn't X in the results" later; quarantining preserves the
  answer while still keeping it out of the canonical folder.

## Templates

- `assets/templates/00_INDEX.md.template`
- `assets/templates/run_log.md.template`
- `assets/templates/HANDOFF.md.template`
- `assets/templates/CLAUDE.md.template`
- `assets/templates/module.md.template`
- `assets/templates/QUARANTINE_README.md.template` — for any folder holding superseded,
  invalid, or excluded material (only needed if the project uses that convention).

Copy the relevant template(s), fill in the placeholders (`{{LIKE_THIS}}`), and delete any
section that doesn't apply to the project rather than leaving it as unfilled boilerplate.
