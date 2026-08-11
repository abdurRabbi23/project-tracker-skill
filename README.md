# project-tracker

A Claude Skill that sets up and maintains a lightweight, modular project-memory system for any
long-running project — a thesis, a codebase, a research project, a business, a personal
project — that gets worked on across many separate Claude chats.

## Why

A project that spans many separate chats loses context between them unless the context lives
in files, not in chat history. Reopen a project after a break and you don't know what was
decided, what was tried and abandoned, or what to do next — so you either re-read everything or
repeat a mistake. This skill fixes that with a small number of files, each with one job, kept
honestly up to date:

- **`run_log.md`** — a dated, append-only timeline of what happened, in order.
- **`logbook/NN_topic.md`** — one file per work-stream, holding the actual reasoning: goals,
  decisions, current state, next steps.
- **`logbook/00_INDEX.md`** — the front door: one-line project description, current status, a
  module table, and a file map of where things live on disk.
- **`logbook/HANDOFF.md`** — always overwritten, never appended to. The current status and the
  immediate next action, short enough to paste into a new chat's first message.
- **`CLAUDE.md`** — mandatory project rules that apply to every session, no exceptions.

It also covers file management: a "where things live" map so a new chat can orient without
exploring the whole folder, and an optional source-of-truth/quarantine convention for projects
that regenerate output over time (experiment runs, drafts, exports) — one canonical folder,
everything superseded or invalid moved out with a `README.md` explaining why, never silently
deleted.

## What it does

Once installed, ask Claude (in a chat, in Cowork, or in Claude Code) to:

- **Set up tracking for a new project** — Claude interviews you (project description,
  work-streams, mandatory rules, existing folders, whether you need the quarantine convention)
  and scaffolds the full file structure.
- **Catch you up at the start of a new chat** — Claude reads the index and handoff note and
  summarizes current status and the next action before doing anything else.
- **Log today's/this session's work** — Claude updates `run_log.md`, the relevant module file,
  the handoff note, and the file map, all in the same pass.
- **Audit the tracking system** — Claude checks for drift: a stale handoff, a module status
  that doesn't match its own file, a file map that doesn't match the real folders, a quarantine
  folder missing its README, broken references.

See `project-tracker/SKILL.md` for the full behavior and `project-tracker/assets/templates/`
for the file templates it scaffolds from.

## Repository structure

```
project-tracker/                          the skill itself — this is what you install
├── SKILL.md                               triggers, behavior, conventions
└── assets/templates/
    ├── 00_INDEX.md.template
    ├── run_log.md.template
    ├── HANDOFF.md.template
    ├── CLAUDE.md.template
    ├── module.md.template
    └── QUARANTINE_README.md.template
project-tracker.skill                      the same folder, pre-packaged as a .zip for upload
README.md                                  this file
LICENSE                                    MIT
```

## Requirements

No external dependencies, no scripts to install, nothing to build — it's a folder of Markdown
files. All you need is one of the following:

- **Claude.ai, Claude Desktop, or Cowork** — any plan (Free, Pro, Max, Team, Enterprise) with
  **Code execution and file creation** enabled (`Settings > Capabilities`). Skills are currently
  in beta on some surfaces; see [Anthropic's skills documentation](https://support.claude.com/en/articles/12512180-use-skills-in-claude)
  if something below doesn't match what you see.
- **Claude Code** — any recent version with skills support (`code.claude.com/docs/en/skills`).

## Installation

### Option A — Claude.ai / Claude Desktop / Cowork

1. Make sure **Code execution and file creation** is on: `Settings > Capabilities`.
2. Go to `Customize > Skills`.
3. Click **+**, then **+ Create skill**, then **Upload a skill**.
4. Upload `project-tracker.skill` from this repo (it's already packaged — no zipping needed).
5. The skill appears in your skills list. Toggle it on.

### Option B — Claude Code

Personal skill (available in every project):
```bash
git clone https://github.com/<your-username>/project-tracker-skill.git
mkdir -p ~/.claude/skills
cp -r project-tracker-skill/project-tracker ~/.claude/skills/project-tracker
```

Project-scoped skill (checked into one repo, shared with anyone who clones it):
```bash
mkdir -p .claude/skills
cp -r /path/to/project-tracker-skill/project-tracker .claude/skills/project-tracker
```

### Rebuilding the `.skill` file yourself

If you edit `project-tracker/` and want a fresh `.skill` package, a `.skill` file is just a zip
of the skill folder:
```bash
cd project-tracker-skill
zip -r project-tracker.skill project-tracker -x "*.DS_Store"
```

## Usage

Once installed and toggled on, just ask for what you want — you don't need to name the skill:

- *"Set up tracking for this project"*
- *"Log what we did today"*
- *"Catch me up on this project"*
- *"Check if my logbook is up to date"*

## License

MIT — see `LICENSE`.
