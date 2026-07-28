# Brief: first AWS production deploy

You are a Claude session with the VENER.AI monorepo checked out (this PM repo
lives inside it at `concordia-venerai/`). Run this when AWS credentials exist
(Activate credits or a paid account).

## Before anything else

Read `CLAUDE.md` here and in the monorepo root and obey both. Deploying is
outward-facing: confirm with Sean before `terraform apply` and before the deploy
script. Never commit secrets (AWS keys stay in env/aws-cli config).

## Task

1. Verify credentials: `aws sts get-caller-identity` (a dockerized
   `amazon/aws-cli` fallback is wired in `scripts/deploy-aws.sh` — the Mac's
   native CLIs were broken as of 2026-07).
2. `cd infra/aws && terraform init && terraform apply -var hf_token=...`
   `[-var alb_certificate_arn=...]` — an ACM cert (or Cloudflare-fronted domain)
   is REQUIRED for iOS ATS; plan for HTTPS from day one. Region: eu-west-1
   (data-residency promise on the site).
3. `scripts/deploy-aws.sh ACCOUNT_ID` and note `terraform output app_url`.
4. Smoke test: register → build a tiny twin (needs HF_TOKEN) → chat → /speak
   (Kokoro fallback is correct without the GPU tier).
5. Set `apps/mobile/app.json` → `expo.extra.serverUrl` to the HTTPS URL; commit.
6. If pre-AWS local sittings exist: build and run the local→cloud export
   (pg_dump + media volume → S3) — design note in the board's "Up next".
7. Update this repo: `docs/project_board.md` (AWS rows → 🟢), decision log if
   anything non-obvious came up.

## Definition of done

- `app_url` serves the web app over HTTPS; register/build/chat smoke passes.
- Mobile `serverUrl` committed; boards updated; Sean told the monthly cost
  estimate vs the deck's $250–650 starter figure.

## Report back

What was deployed, the URL, actual first-month cost estimate, anything skipped
or failing, and what remains for the TestFlight brief.
