# VENER.AI PM - project board

Living tracker. Status markers: 🟢 done, 🔵 in progress, ⚪ todo, 🔴 blocked,
🟡 in review.

Product code and technical docs live in the monorepo one level up
(`/Users/sbenson/Documents/VENER.AI`); its `docs/PROJECT-BOARD.md` holds the
detailed per-service evidence. **This board is the current-status source of
truth**; update it as work moves.

Last full update: **2026-07-28**.

## At a glance

| Item | Priority | Stage | Status | Blocker |
|------|----------|-------|--------|---------|
| Core platform (builder pipeline, inference/RAG, sessions, handoff) | high | build | 🟢 proven E2E | - |
| Voice cloning service (Chatterbox, containerized) | high | build | 🟢 proven | box currently loaned out |
| Talking portrait (EchoMimicV3 + persistent-model server) | high | build | 🟢 proven (WSL warm render 264s/125f) | box currently loaned out |
| Per-twin usage quotas (video allowance, message fair-use) | high | build | 🟢 62 tests green | - |
| Guide avatar clips (pre-rendered interviewer) + hands-free sitting screen | high | build | 🔵 code done, device-untested | GPU box loan; quality/voice verdicts |
| Sitting refinement with Sean as test subject | high | validate | ⚪ next up | device retest first |
| Nan's sitting | high | validate | ⚪ deliberately postponed (2026-07-28) | refinement above |
| Apple Developer enrollment (DUNS 234989532 ready) | high | ship | 🔴 user action | Apple verification days |
| EAS account + project link (slug `venerai`) | high | ship | 🟢 done 2026-07-25 | - |
| TestFlight build + both phones as internal testers | high | ship | ⚪ ready to fire | Apple enrollment + HTTPS backend |
| AWS Activate credits (via Spendbase) | high | fund | 🔵 re-apply | site eligibility FIXED 2026-07-28 |
| AWS deploy (Terraform + deploy script, ECS/RDS/S3/SQS) | high | ship | 🔴 written+validated, awaiting credentials | Activate credits |
| Marketing site vener.ai (product/founder/terms + sealed waitlist) | med | live | 🟢 live, Activate-eligible | - |
| Pitch deck (Activate, measured economics) | med | fund | 🟢 final PDF | - |
| GPU box hardware (bad RAM stick B) | med | ops | 🔴 Corsair RMA (full kit must ship) | user RMA + box downtime |
| Full-twin training (voice pack live; persona/face LoRA trainers) | med | build | 🔵 voice pack live; LoRA runs need CUDA time | GPU box availability |
| Engineering hardening backlog (Alembic, media-leak on delete, resumable uploads, retrieval eval) | med | build | ⚪ backlog | - |
| Android build | low | ship | ⚪ later | iOS first |

## In progress

- 🔵 **Guide avatar (Option B)**: batch driver + serve-protocol renderer work; female
  guide voice generated (Kokoro ref → Chatterbox clone); quality A/B samples on the
  Mac (`data/guide-*.mp4`) awaiting Sean's three verdicts (voice ✓/✗, steps tier,
  face choice — current portrait is a placeholder stock photo, shipped guide needs a
  generated/licensed face). Batch = resumable; ~5h at interactive settings, ~2 days
  at premium on WSL. Paused while the GPU box is loaned out.
- 🔵 **AWS Activate re-application**: Spendbase rejected 2026-07-28 (site was
  waitlist-only). Site fixed and live the same day (product/founder/company/terms +
  company no. 17335051). Re-apply via Spendbase or direct aws.amazon.com/activate.

## Up next

- ⚪ Device retest of the revamped sitting screen on Sean's iPhone (Expo Go + local
  stack, ~15 min): hands-free VAD start/stop, expo-video playback, upload. Gates
  everything sitting-related.
- ⚪ Sean's full self-sitting → iterate on flow annoyances → only then nan.
- ⚪ Synthetic warmup render at talking-head deploy (first render after spawn pays
  one-time T5 encode; make it invisible).
- ⚪ Local→cloud data export script (pg_dump + media→S3) so pre-AWS sittings carry
  over (≈1 day, build when AWS lands).
- ⚪ Corpus-growth metrics ("Ark" evidence for seed stage): hours/twin over time,
  sittings, conversations by twin age, churn by twin age.

## Blocked

- 🔴 **Apple enrollment** — user to enroll (individual = fastest, convertible later;
  org required before public release, App Review 5.1.1(ix)).
- 🔴 **AWS deploy** — waiting on Activate credits (re-application pending).
- 🔴 **GPU work** (guide clips, LoRA training, cloned-voice serving) — box loaned to
  another project; also stick-B RMA will take the box down when shipped.
- 🔴 **iOS goal externals** — Apple + HTTPS backend (AWS, or Tailscale Serve interim
  which requires Tailscale app on both phones).

## Team

| Role | Agent | Type | Focus |
|------|-------|------|-------|
| Builder | `builder` | producer | implement features and prototypes in the monorepo |
| Product critic | `product-critic` | gate | scope, family-user value, honesty of claims |
| Code reviewer | `code-reviewer` | gate | correctness, security, quality |
| Privacy & security critic | `privacy-security-critic` | gate | consent, GDPR, ISO-aligned controls, data handling |
| Model scout | `model-scout` | periodic | model quality + commercial-licence vetting (docs/MODELS.md in monorepo) |
| Funding scout | `funding-scout` | periodic | credits/grants pipeline, application drafting |
| Competitor scout | `competitor-scout` | periodic | legacy-tech market watch (HereAfter, StoryFile, Re;memory, Eternos…) |
