# VENER.AI PM - project board

Living tracker. Status markers: 🟢 done, 🔵 in progress, ⚪ todo, 🔴 blocked,
🟡 in review.

Product code and technical docs live in the monorepo one level up
(`/Users/sbenson/Documents/VENER.AI`); its `docs/PROJECT-BOARD.md` holds the
detailed per-service evidence. **This board is the current-status source of
truth**; update it as work moves.

Last full update: **2026-08-02** (evening).

## At a glance

| Item | Priority | Stage | Status | Blocker |
|------|----------|-------|--------|---------|
| Core platform (builder pipeline, inference/RAG, sessions, handoff) | high | build | 🟢 proven E2E | - |
| Voice cloning service (Chatterbox, containerized) | high | build | 🟢 proven | box currently loaned out |
| Talking portrait (EchoMimicV3 + persistent-model server) | high | build | 🟢 proven (WSL warm render 264s/125f) | box currently loaned out |
| Per-twin usage quotas (video allowance, message fair-use) | high | build | 🟢 62 tests green | - |
| Guide avatar clips (pre-rendered interviewer) + hands-free sitting screen | high | build | 🔵 committed (3913a82), device-untested | GPU box loan; quality/voice verdicts |
| Sitting refinement with Sean as test subject | high | validate | ⚪ next up | device retest first |
| Nan's sitting | high | validate | ⚪ deliberately postponed (2026-07-28) | refinement above |
| Apple Developer enrollment (DUNS 234989532 ready) | high | ship | 🔴 user action | Apple verification days |
| EAS account + project link (slug `venerai`) | high | ship | 🟢 done 2026-07-25 | - |
| TestFlight build + both phones as internal testers | high | ship | ⚪ ready to fire | Apple enrollment + HTTPS backend |
| AWS Activate credits | high | fund | 🟢 GRANTED $1,000 (2026-07-31) | - |
| NVIDIA Inception | med | fund | 🔴 rejected (automatic, no reason) | do not re-apply blind |
| AWS deploy (Terraform + deploy script, ECS/RDS/S3/SQS + render queue) | high | ship | 🔵 UNBLOCKED — awaiting AWS account credentials on the Mac | user: aws configure |
| Async render pull-worker (phase-A GPU: box renders cloud jobs) | high | build | 🟢 built + 68 tests (13c34a2); live test at AWS deploy | - |
| Marketing site vener.ai (product/founder/terms + sealed interest form) | med | live | 🟢 live; priority-access framing 2026-07-30 | - |
| **Website visual upgrade + real product demos** | **high (URGENT)** | build | ⚪ todo | rights-clean demo faces (guide face decision) |
| Pitch deck (Activate, measured economics) | med | fund | 🟢 final PDF | - |
| GPU box hardware (bad RAM stick B) | med | ops | 🔴 Corsair RMA (full kit must ship) | user RMA + box downtime |
| Second GPU: RTX 3060 12GB dedicated voice card | med | ops | 🟢 installed 2026-08-01; cohabitation PROVEN (render 100% + synth 72% simultaneous); parking retired | - |
| Local models service (Whisper + embeddings + Kokoro on 3060) + NIM chat primary | high | build | 🟢 live 2026-08-02 — product survives dead HF account (fallbacks intact) | - |
| Local chat floor: Qwen3-4B-Instruct-2507 (llama.cpp, 3060, 63+ tok/s) | high | build | 🟢 live — chain NIM→local→HF; the floor carried an ENTIRE weave when NIM was cold | - |
| **MILESTONE: first twin built by the product's own app, end-to-end** (founder, 17 answers, 49 memories, portrait from in-app photo, $0 external AI) | high | validate | 🟢 2026-08-02 | - |
| Chat round 2: voice-coming indicator, speak-to-talk (local Whisper), in-chat portrait moments | high | build | 🟢 c79a970 — device test next | - |
| Admin monitor /admin: services+homes, users, twins, storage, LIVE LOG WINDOW, twin RESET/ERASE (GDPR) buttons; OTP login over REAL EMAIL (SMTP2GO) | med | build | 🟢 live, E2E verified | - |
| Voice pack for founder twin (8 curated refs, rotation live) | med | build | 🟢 published — likeness A/B by founder pending | - |
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
- 🟡 **AWS Activate re-application submitted 2026-07-29** (site made eligible first:
  product/founder/company/terms + company no. 17335051; launch date given: Oct 2026;
  products: ECS/ALB/RDS/ElastiCache/S3/SQS/ECR/KMS/ACM/CloudWatch + EC2 G6 L4 GPU).
  Site reframed 2026-07-30: waitlist → "register your interest / priority access".

## Up next

- ⚪ **Continuity & portability promise** ("your archive outlives us"): FAQ +
  terms copy guaranteeing full export in open formats and a wind-down export
  window — HereAfter's shutdown (competitor-watch.md) makes abandonment the
  question every family will ask; answer it structurally before the pilot.
- ⚪ "Bring your recordings" import angle for displaced HereAfter families
  (functionally supported today via artifact uploads; positioning work only).
- ⚪ **URGENT: website visual upgrade + demos** — screen recordings of the app,
  a guide-avatar clip, a living-portrait reply. Constraint: rights-clean faces
  only (generated guide face / Sean's own footage — never Unsplash-derived
  talking renders). Reuses the guide-avatar batch output when verdicts land.
- ⚪ Device retest of the revamped sitting screen on Sean's iPhone (Expo Go + local
  stack, ~15 min): hands-free VAD start/stop, expo-video playback, upload. Gates
  everything sitting-related.
- ⚪ Sean's full self-sitting → iterate on flow annoyances → only then nan.
- ⚪ Voice pack curation v2: the 8 phone-mic refs lost to the single clean
  reference (founder A/B 2026-08-02) — stricter quality gating (loudness/SNR
  floor, min duration, K=3-4), and prefer the base ref until a pack proves
  itself; re-record reading passages in quiet as the real fix.
- ⚪ Synthetic warmup render at talking-head deploy (first render after spawn pays
  one-time T5 encode; make it invisible).
- ⚪ Local→cloud data export script (pg_dump + media→S3) so pre-AWS sittings carry
  over (≈1 day, build when AWS lands).
- ⚪ Corpus-growth metrics ("Ark" evidence for seed stage): hours/twin over time,
  sittings, conversations by twin age, churn by twin age.
- ⚪ **Daily question / diary mode** (founder idea, 2026-08-02): after the twin
  exists, one gentle question a day (unanswered bank + generated follow-ups) or
  a freeform "tell me about today" diary note — a 2-minute ritual that deepens
  the twin with dated, episodic memories. Pairs naturally with the daily
  portrait moment (same cadence) and IS the Ark growth engine + retention
  mechanic. Needs: notification permission, dated-memory support in retrieval,
  question-picker endpoint.

## Blocked

- 🔴 **Apple enrollment** — user to enroll (individual = fastest, convertible later;
  org required before public release, App Review 5.1.1(ix)).
- 🔴 **AWS deploy** — credits GRANTED; needs user to configure AWS credentials
  locally, then run prompts/aws-deploy.md (confirm-before-apply). Budget note:
  $1,000 ≈ 3–6 months of the starter footprint WITHOUT the GPU tier — deploy
  core stack first (Kokoro fallback voice), GPU tier deliberately deferred.
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
