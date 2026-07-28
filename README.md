# VENER.AI PM

Project management and source of truth for VENER.AI — living portraits of the people you love

Maintained by Sean Benson (VENERAI LTD).

## What this repo is

The **status, decisions and plans** source of truth for VENER.AI, designed so
quality survives conversation compaction: any fresh Claude session (or human)
reads this repo and knows exactly where the work stands and why. Product code
and technical docs live in the **monorepo one level up**
(`/Users/sbenson/Documents/VENER.AI` — services, apps, infra, `docs/`); this
repo sits inside it as an independent git repo, gitignored by the monorepo.

Seeded 2026-07-28 with the full state of the founding build (2026-07-03 →
2026-07-28): platform proven end-to-end, GPU services working, site live,
funding pipeline in motion. Start with `docs/project_board.md`.

## Layout

```
CLAUDE.md            binding rules (incl. VENER.AI brand/consent law) + agent posture
docs/
  roadmap.md         vision, now / next / later, milestones
  project_board.md   the living tracker — START HERE
  meetings.md        rolling dated record of stakeholder meetings
  decisions.md       decision log: the "why" of every non-obvious choice
prompts/             self-contained briefs (aws-deploy, testflight-ship, …)
.claude/agents/      the versioned agent team (producers, gates, scouts)
src/                 (unused — product code lives in the monorepo)
assets/              brand assets
```

## Workflow

1. Edit or write here (code, a doc, a decision), commit, push.
2. A teammate or a remote / CI session pulls and continues.
3. Keep `docs/project_board.md` current; log meetings in `docs/meetings.md` and
   choices in `docs/decisions.md`.

## Getting started

```bash
# this repo is docs + agents only; the product stack is in the monorepo:
cd .. && make up          # local stack (needs HF_TOKEN in .env)
cd .. && make test        # 62 tests across the two services
```

## The agent team

Versioned in `.claude/agents/`: producers build, critic gates review before things
ship, and scouts keep watch on the market. See `CLAUDE.md` for how and when to use
them.
