# Session prompts

Self-contained briefs for a Claude session that runs somewhere other than your
local machine (a remote box, a CI job, a teammate's checkout). Instead of pasting a
long brief, start a session there and point it at the file:

```
git pull
# then, in the session:
#   Read prompts/<task>.md and follow it.
```

Each brief begins by requiring the session to read `../CLAUDE.md` (the rules), then
references the relevant code or doc, so the prompt stays short and the procedure
lives in one place.

## Conventions

- Keep approvals on for anything outward-facing (deploy, publish, send).
- Commit work back through git so it is reviewed.
- Report results plainly: what changed, what passed, what is left.

## Available briefs

- `example_task.md` - a template brief; copy it per task.
