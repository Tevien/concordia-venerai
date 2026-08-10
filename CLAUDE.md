# VENER.AI PM - working rules

Binding guidance for any Claude session in this repo. Keep it short and real; edit
it as the work teaches you what matters.

## Rules

- **Never commit secrets.** No API keys, tokens, passwords, `.env` files, or
  customer data. Use the secret manager / env vars; `.gitignore` covers the common
  cases, but check before you commit.
- **Outward-facing actions need confirmation.** Deploying, publishing a release,
  sending to customers, posting publicly, or anything hard to reverse: confirm
  first unless explicitly told to proceed. Approval for one such action does not
  extend to the next.
- **Keep changes reviewable.** Small, focused commits with clear messages. Branch
  off the default branch (main) for non-trivial work.
- **Report outcomes faithfully.** If tests fail, say so with the output. If a step
  was skipped, say that. Done-and-verified is stated plainly, not implied.

## VENER.AI-specific rules (binding)

- **Brand:** the name is always **VENER.AI** — never bare "VENER". Company:
  VENERAI LTD, England & Wales, no. 17335051. Name from Latin *venerari*.
  **Spoken form: "Vennerai", one word** — any text sent to a TTS engine
  substitutes it (helper: `vener_common.hf.speakable`; app: `src/speech.js`).
- **Claims:** "ISO 27001/17/18-**aligned** controls", never "certified".
  Participation is never "free" — invitations carry a cost contribution.
  Public claims must be true today, not aspirational ("built, not promised").
- **Consent is product law:** no recorded consent → no twin; the twin never
  invents memories; estimates are labelled estimates. Any change touching these
  goes through `privacy-security-critic` before shipping.
- **Where things live:** product code + technical docs = the monorepo at
  `/Users/sbenson/Documents/VENER.AI` (this repo sits inside it but is its own
  git repo, gitignored by the monorepo). This repo = status, decisions, plans.
  **Update `docs/project_board.md` and `docs/decisions.md` as part of finishing
  any significant piece of work, not after.**
- **The GPU box** (dual-boot: `tevsockswin` WSL / `tevsockslin` Ubuntu, RTX
  5070 Ti 16 GB, cu128 required) is shared with other projects and has one RAM
  stick pending RMA — ask before heavy use; park `vener-voice` during renders;
  checksum-verify anything that predates 2026-07-14.
- **No remote push** for the monorepo or this repo until the user sets remotes
  (explicit choice).

## The agent team and posture

The roster is versioned in `.claude/agents/`:

- **Producers** (`builder`): do the work - implement, draft, prototype.
- **Critic gates** (`product-critic`, `code-reviewer`, `privacy-security-critic`):
  review before things ship. The loop is: a producer builds, then the critics
  gate, revising until all pass. Privacy/consent-touching changes always include
  the privacy gate.
- **Scouts** (`competitor-scout`, `model-scout`, `funding-scout`): periodic
  watches - market, model licensing/quality, credits & grants - each maintaining
  its doc.

Posture by context:

- **Working locally (planning, building, reviewing):** use the full team freely.
- **In CI / production / a customer-facing environment:** run lean. Do the narrow
  task and do not spawn the planning agents. Fewer moving parts where it matters.

## Decisions and tracking

- Record non-obvious choices in `docs/decisions.md` (what, why, alternatives).
- Keep `docs/project_board.md` current as work moves.
- Log stakeholder meetings in `docs/meetings.md` with dates.
