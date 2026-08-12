# VENER.AI PM - project board

Living tracker. Status markers: 🟢 done, 🔵 in progress, ⚪ todo, 🔴 blocked,
🟡 in review.

Product code and technical docs live in the monorepo one level up
(`/Users/sbenson/Documents/VENER.AI`); its `docs/PROJECT-BOARD.md` holds the
detailed per-service evidence. **This board is the current-status source of
truth**; update it as work moves.

Last full update: **2026-08-12**.

## At a glance

| Item | Priority | Stage | Status | Blocker |
|------|----------|-------|--------|---------|
| Core platform (builder pipeline, inference/RAG, sessions, handoff) | high | build | 🟢 proven E2E | - |
| Voice cloning service (Chatterbox, containerized) | high | build | 🟢 proven | box currently loaned out |
| Talking portrait (EchoMimicV3 + persistent-model server) | high | build | 🟢 proven (WSL warm render 264s/125f) | box currently loaned out |
| Per-twin usage quotas (video allowance, message fair-use) | high | build | 🟢 62 tests green | - |
| Guide avatar clips (pre-rendered interviewer) + hands-free sitting screen | high | build | 🟢 COMPLETE SET RENDERED 2026-08-12: all 44 questions + welcome + 5 enc + 3 idles at native 768² (interviewer named CLIO, KLY-oh, frontal candidate 4); founder script review applied (us→me, off_limits cut); transcript audit 49/49 PASS after 2 auto-caught fixes; join-quality metrics live (seam SSIM + motion ratio, founder-calibrated) | welcome v3 render + founder report pass → bundle |
| Sitting refinement with Sean as test subject | high | validate | 🟢 device test 2026-08-09: clip interview + video quality ACCEPTED for this version → remodel round issued | - |
| Nan's sitting | high | validate | ⚪ deliberately postponed (2026-07-28) | refinement above (R2 consent clip: DONE 2026-08-09, in bundle) |
| Apple Developer enrollment (DUNS 234989532 ready) | high | ship | 🟡 submitted, in review (2026-08-09) | Apple verification |
| Google Play org account | med | ship | 🟢 FULLY VERIFIED as Venerai Ltd organisation (2026-08-10) | - |
| EAS account + project link (slug `venerai`) | high | ship | 🟢 done 2026-07-25 | - |
| TestFlight build + both phones as internal testers | high | ship | ⚪ ready to fire | Apple enrollment + HTTPS backend (R2 consent clip: done) |
| AWS Activate credits | high | fund | 🟢 GRANTED $1,000 (2026-07-31) | - |
| NVIDIA Inception | med | fund | 🔴 rejected (automatic, no reason) | do not re-apply blind |
| AWS deploy (Terraform + deploy script, ECS/RDS/S3/SQS + render queue) | high | ship | 🔵 UNBLOCKED — awaiting AWS credentials; now THE LAUNCH COMPUTE (economics v2: serverless GPU from subscriber #1, L4 VM at ~100 subs) | user: aws configure |
| Async render pull-worker (phase-A GPU: box renders cloud jobs) | med | build | 🟢 built + 68 tests; DEMOTED 2026-08-10 to dev/test + emergency contingency (cloud-GPU launch) | - |
| Marketing site vener.ai (product/founder/terms + sealed interest form) | med | live | 🟢 live; priority-access framing 2026-07-30 | - |
| **Website visual upgrade + real product demos** | **high (URGENT)** | build | 🔵 4 guide clips rendered (B recipe: ag5.5/as1.2/24 steps/loudnorm — the lip-sync A/B winner); site sections LIVE on vener.ai (hero + ask-demo + CTA) | remaining: app screen-recordings S4/S5 |
| Pitch deck (Activate, measured economics) | med | fund | 🟡 economics STALE (subscription-only pricing + cloud-GPU COGS 2026-08-10) — refresh before next investor use | - |
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
| Android build (Beam Pro as test device) | med | ship | ⚪ UNBLOCKED (Play org verified) — ready for closed-test track + Beam Pro APK | - |

## In progress

- 🔵 **App remodel: story-first, interviewer-led** (founder direction 2026-08-09
  after the device test — video quality ACCEPTED for this version, decision log
  entry same date): home = people list with gathering/ready/woven states; she
  leads (no question menus/webforms); weave unlocks with an honest readiness
  assessment (new `GET /twins/{id}/readiness`); daily journal with personalised
  questions (new `GET /twins/{id}/journal/today`, chat-chain over memories, bank
  fallback); uploads tucked away (pipeline still uses letters→persona,
  photo→portrait). BUILT + GATED 2026-08-09 on branch `app-remodel-story-first`
  (d3bf789 backend, df9f599 app): UX gate issued 4 BLOCKs (woven-twin demotion
  on journal answer; mid-recording exit data loss; dishonest skip-close; Home
  one-tap unconfirmed weave) — all fixed + 14 ranked notes applied; privacy
  gate APPROVED consent rewording with required changes R1 (voice/audience/
  photos disclosed in the spoken sentence) + R3 (asked_prompt persisted per
  artifact) — both done, 91+22 tests green; R2 consent clip DONE + bundled.
  2026-08-12: terms & conditions drafting in-app (compliance-review + ui-app-developer). OPEN: founder device pass (stack is live; Expo Go serves the
  branch now), then merge. Flagged founder decision: personalised journal
  questions speak in device TTS, not her voice — guide-voice synth is the
  fast-follow.

## Up next

- ⚪ **Continuity & portability promise** ("your archive outlives us"): FAQ +
  terms copy guaranteeing full export in open formats and a wind-down export
  window — HereAfter's shutdown (competitor-watch.md) makes abandonment the
  question every family will ask; answer it structurally before the pilot.
- 🟢 Batch-render tail-trim: DONE — word-timestamp trim + sentence-aware
  segmentation live in render_guide_clips.py phase 1.5 (trimmed 8+ lines in
  the 2026-08-04 app batch).
- 🔵 **StableAvatar A/B — FOUNDER PRIORITY 2026-08-12**: MIT, Wan-1.3B family,
  single-pass infinite length (no stitching ever), English audio encoder,
  Sync-C ~2x the big names (model-scout digest in monorepo docs/MODELS.md).
  ai-researcher building on the 5070 Ti now: welcome + 60-90s single-pass vs
  the stitched chain. If it wins it becomes the render engine — retiring seams
  structurally (frame-carry v4 made joins PASS but conditioning-frame decay
  blurs later segments; founder verdict). Scout also: LatentSync now Apache
  (mouth rung back on table, needs detector swap spike); Wan2.2-S2V-14B as
  future L4 masters tier; Hunyuan family EXCLUDES EU/UK (dead end).
- ⚪ **Quality envelope — STANDING priority (founder directive 2026-08-04):
  keep pushing render realism on BOTH pipelines.** Renderer-level rungs
  (shared, in effort order): (1) temporal-aware face restoration/SR pass on
  rendered frames — cheapest visible win, mind flicker (per-frame GFPGAN
  flickers; use video-aware restorers); (2) two-stage render: Wan/EchoMimic
  motion → mouth re-render pass (LatentSync/MuseTalk-class) for crisper
  visemes; (3) English-viseme fine-tune of EchoMimicV3 (it ships a
  chinese-wav2vec2 audio encoder — LoRA + English audio-encoder swap on
  HDTF/CelebV-class data, overnight-scale on our cards).
  **Interviewer (guide) track:** apply (1)+(2) to the 18 bundled masters and
  re-bundle (async = polish is free); remaining 26 question lines at the same
  bar; multi-take rendering (2-3 seeds per line, pick the best take — the
  guide is rendered once, watched thousands of times); idle/ambient guide loop
  between questions (ties into ambient living portraits below).
  **Living portrait (twin) track:** same polish passes on portrait moments;
  ~30-60s consented enrollment VIDEO as a sitting step (the Synthesia cheat:
  real footage of the subject speaking) → real-footage idle reels,
  reenactment-style rendering (LivePortrait-class drives real frames), and
  per-twin LoRA later — a product/UX + consent-copy decision, not just ML;
  voice fine-tune tier (StyleTTS2) is this track's audio rung.
- ⚪ "Bring your recordings" import angle for displaced HereAfter families
  (functionally supported today via artifact uploads; positioning work only).
- ⚪ **Ambient living portraits** (founder vision 2026-08-03): the twin's frame
  is always alive — pre-rendered, randomized idle reel (breathing, smile,
  glance, wave, walk-off/empty-frame/walk-back) as a weighted playlist; the
  portrait-moment button plays an "approach" clip first ("they come over").
  Pre-render ~6-10 short clips per twin post-weave (~$2 one-time inside the
  Weave margin, ~5MB download-once). Engine note: EchoMimic is audio-driven —
  idle motion needs the Wan i2v pipeline with motion prompts (same weights,
  new pipeline) or the near-silent-audio trick; walk-off from a bust portrait
  is the stretch goal. Non-verbal only: ambience, not testimony.
- ⚪ **URGENT: website visual upgrade + demos** (now incl. the synthetic guide
  as an alpha-video sprite moving over page elements — waves at the CTA,
  peeks from section edges; render on flat bg → matte → VP9/HEVC alpha) — screen recordings of the app,
  a guide-avatar clip, a living-portrait reply. Constraint: rights-clean faces
  only (generated guide face / Sean's own footage — never Unsplash-derived
  talking renders). Reuses the guide-avatar batch output when verdicts land.
- ⚪ Device retest of the revamped sitting screen on Sean's iPhone (Expo Go + local
  stack, ~15 min): hands-free VAD start/stop, expo-video playback, upload. Gates
  everything sitting-related.
- ⚪ Sean's full self-sitting → iterate on flow annoyances → only then nan.
- ⚪ **Voice fine-tune tier** (the real "more of my voice" answer): per-person
  TTS fine-tuning from the training pipeline's LJSpeech dataset. Licence
  verdicts: StyleTTS2 (MIT) = the candidate; F5-TTS & Fish-Speech weights are
  non-commercial (hard fail); XTTS already rejected. ~20 min of founder audio
  usable today; grows with every answered question. Until then: zero-shot
  Chatterbox with adherence-first defaults (cfg 0.2 / exaggeration 0.4, the
  founder's accent A/B winner 2026-08-03) + combined dual-passage reference
  (both now in production, d34fdca) — mitigates the American drift, cannot
  eliminate it.
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

Two rosters: this repo's PM team (below) and the monorepo's engineering +
specialist team (`.claude/agents/` there, updated 2026-08-12).

| Role | Agent | Type | Focus |
|------|-------|------|-------|
| Builder | `builder` | producer | implement features and prototypes in the monorepo |
| Product critic | `product-critic` | gate | scope, family-user value, honesty of claims |
| Code reviewer | `code-reviewer` | gate | correctness, security, quality |
| Privacy & security critic | `privacy-security-critic` | gate | consent, GDPR, ISO-aligned controls, data handling |
| Model scout | `model-scout` | periodic | model quality + commercial-licence vetting (docs/MODELS.md in monorepo) |
| Funding scout | `funding-scout` | periodic | credits/grants pipeline, application drafting |
| Competitor scout | `competitor-scout` | periodic | legacy-tech market watch (HereAfter, StoryFile, Re;memory, Eternos…) |

Monorepo team (engineering + specialists):

| Agent | Type | Focus |
|-------|------|-------|
| `ai-researcher` | producer | models, prompts, pipeline, RAG, voice, eval, inference cost |
| `backend-platform-engineer` | producer | FastAPI, DB/migrations, Redis, queues, Docker, Terraform |
| `ui-app-developer` | producer | web + Expo apps, recording UX, accessibility |
| `security-engineer` | producer/gate | auth, secrets, GDPR/ISO, abuse cases |
| `user-experience` (2026-08-09) | gate | will this land with grieving families + elderly subjects; blocks feature-first drift |
| `infra-scaling-review` (2026-08-10) | reviewer | quantified growth reviews, scaling-rung triggers (docs/infra-scaling-review.md) |
| `launch-specialist` (2026-08-10) | planner | coordinated store launch, waves, packaging within the never-free law (docs/launch-plan.md) |
| `compliance-review` (2026-08-10) | reviewer | App Store/Play policy + UK/EU law mapped to in-repo evidence |
