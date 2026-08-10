# VENER.AI PM - decision log

Lightweight record of non-obvious choices (ADR style), so the "why" survives.
Newest first. One entry per decision; keep it short.

---

## 2026-08-10 - The interviewer is named Clio

**Status:** accepted (founder naming)

**Context:** the synthetic interviewer needed a name — "she" was never
introduced (UX gate finding), and "our interviewer" is generic.

**Decision:** she is **Clio**, after the Greek Muse of history — the keeper of
what deserves remembering, which is literally her job. User-facing copy
introduces her by name once (the begin button), then "she" is earned. Code
identifiers stay `guide*` (stable API); the name is a copy-level fact. Pronunciation RESOLVED 2026-08-10 by
voice-preview A/B: she is "KLY-oh" (classical muse) — every TTS path
respells to "Cly-oh" via speakable(); written surfaces keep "Clio".

---

## 2026-08-10 - Launch economics v2: subscription-only pricing, cloud-GPU launch

**Status:** accepted (founder direction; supersedes both 2026-07-31 decisions below)

**Context:** the up-front $30 Weave fee existed to cover onboarding compute,
but AWS bills in arrears — cash-flow only requires subscription income to
land before the AWS bill, which monthly-in-advance billing guarantees. An
up-front charge is pure friction against traction. On infra: the home box was
the launch compute in phase-A; the founder now designates local GPUs as
dev/test only.

**Decision:**
1. **Pricing: $10/mo subscription only, charged at signup** — no up-front
   fee. The weave's compute cost (~$2-3 serverless) is absorbed by the first
   month. Never-free is preserved: the first charge happens at signup, before
   the sitting; there is no $0 window. Worst-case churn exposure is ~$2-3 per
   month-one canceller — an acceptable acquisition cost. Extra moments, when
   they arrive, are priced above their serverless cost (~$0.25/moment floor).
2. **Launch compute is AWS, data is AWS.** Serverless per-second GPU
   (Modal/RunPod-class, ~$0.15-0.25/moment) from subscriber #1 so cost tracks
   usage; a dedicated full-time L4 VM becomes cheaper at roughly ~100 active
   subscriptions and is adopted then. Local GPUs: product development and
   testing only; the SQS pull-worker survives as an emergency contingency
   valve, not a launch dependency.

**Arithmetic check (recorded honestly):** 100 × $10 = $1,000 MRR gross vs
full-time L4 ≈ $580-660/mo on-demand (≈$350-400 with a savings plan) + core
stack ~$150-250 — the founder's "100 subs covers the L4" holds on GROSS
revenue. It is MARGINAL after store commission (15-30% if billed through
IAP): web checkout (~3%) vs store billing is now a first-order packaging
decision, not a detail. Below 100 subs, serverless keeps infra ~$0.50-1.50
per subscriber-month, so the model is cash-positive at every scale.
Consequence: launch-plan.md and infra-scaling-review.md revised same day;
pitch-deck economics need a refresh pass.

---

## 2026-08-10 - Two-gate rule for interviewer clips: script review BEFORE render, transcript audit AFTER

**Status:** accepted (founder rule)

**Context:** clips are generated from fixed text scripts (LLM-drafted question
bank, static in guide-lines.json) → TTS → render. Two silent word-loss bugs
(seam-window drop 2026-08-09, output `-to` pad-chop 2026-08-10) were caught
only by listening; separately, most chapter-question texts had never been
founder-reviewed before GPU-hours were spent on them.

**Decision:** (1) PRE-RENDER: the founder reviews every line text before a
batch renders it (`data/interviewer-script-review.md`; batches stay held until
approval) — the content gate, and it's cheap because text edits cost nothing
before rendering. (2) POST-RENDER: the batch ends with a whisper transcript
audit diffing every final clip against its script (PASS/REVIEW at 0.85,
transcript-report.md) — the integrity gate for TTS/render corruption. Idle
(non-speech) clips skip both (whisper hallucinates on noise). Runtime
exception: personalised journal questions are LLM-generated per day and can't
be pre-reviewed per-line — they are TTS-only (no video render) and carry
their own grounding validation.

---

## 2026-08-09 - App remodel: story-first, interviewer-led

**Status:** accepted (founder direction after the device test of the clip-driven sitting)

**Context:** real twin interaction must stay rationed (a few portrait moments per
day, possibly purchasable extras later) to control GPU cost — so interaction
cannot be the app's core loop. The device test confirmed the interviewer
experience carries the product.

**Decision:** rebuild the app around collecting the user's story. (1) Home =
the list of people whose stories you keep; per person one state (gathering /
ready to weave / woven) and one primary action. (2) The synthetic interviewer
LEADS — she decides the next question; no question menus, no webform feel.
(3) The weave button unlocks on a readiness threshold WITH an honest
assessment of how good the twin will be (strengths/gaps). (4) Returning users
get a daily journal: one insightful question per day, personalised from their
existing memories where possible (bank fallback). (5) Uploads (photos/letters)
are tucked away as a secondary option — note: the pipeline DOES use them today
(letters → persona prompt, photo → portrait), so the UI de-emphasizes without
removing capability. New `user-experience` gate agent created to hold this
line. Alternatives rejected: interaction-centred app (cost + disappointment
risk), data-management app (webform feel is exactly what testing flagged).

**Status:** accepted (pick made: candidate 4, identity scale 0.65)

**Context:** the app guide must face the user directly; the chosen site face
(candidate 6) is three-quarter. img2img strength sweeps (0.45→0.75) kept the
body turned — pose is decided in early denoise steps, which img2img inherits
from the source image. Higher strength would free the pose only by destroying
the identity.

**Decision:** decouple the two: SDXL txt2img composes from a frontal prompt
(owns the pose), IP-Adapter plus-face carries her identity from the reference.
Gotcha worth remembering: the vit-h face adapter needs
`image_encoder_folder="models/image_encoder"` (ViT-H, 1280-d) — the sdxl_models
default encoder is ViT-bigG (1664-d) and fails with a shape mismatch.
Consequence: site and app use different photos of the same woman (acceptable —
that's how real people photograph); every scale (0.5/0.65/0.8) held likeness —
"all recognisably her" — so the pick came down to pose, won at 0.65 (runner-up
at 0.5). Process lesson: the contact sheet had no printed indices, causing a
1-based vs 0-based miscount on the first pick — sheets now number every thumb.

---

## 2026-08-02 - Passwordless login (emailed one-time code) as the sole method — at cloud launch

**Status:** accepted (build at AWS deploy, when an email sender exists)

**Context:** passwords are the wrong tool for this audience: elderly users
forget them, families share them, resets are the #1 support burden — and a
password database is pure liability for data this sensitive. Founder asked
whether OTP-only is better.

**Decision:** at cloud launch, replace passwords with a 6-digit emailed code
(SES once AWS is live; codes in Redis with short TTL + attempt limits).
Two conditions make it work for a grandmother: (1) LONG sessions -
"remember this device" ~90 days with silent refresh, so login is a rare
event done with family help, not a daily hurdle; (2) the family-assisted
first sign-in is part of the sitting-setup ritual. Admin page uses the same
flow. Until the email sender exists, local dev keeps passwords.

**Alternatives considered:** password + email reset (same takeover ceiling -
email - but adds a breachable hash DB and forgetting); social logins
(adding third-party login would trigger Apple's Sign-in-with-Apple
requirement and adds trackers we don't want); SMS codes (costs, SIM-swap,
elderly users change numbers).

**Consequences:** email deliverability becomes an availability dependency
(SES reputation, spam-folder guidance in the invite); account takeover
surface concentrates on the email account - acceptable because password
systems with email reset share that ceiling; no more password hashes stored
anywhere. ADMIN_EMAILS narrowed to sean@vener.ai (2026-08-02).

---

## 2026-07-31 - Pricing: $30 Weave + $10/mo including one portrait moment per day

**Status:** SUPERSEDED 2026-08-10 (up-front charge dropped — see "Launch economics v2")

**Context:** the original Weave fee was large for a product asking hesitant,
grief-adjacent buyers to commit; the goal for launch is break-even early,
profit with volume, affordable entry. Separately, a monthly video pool of 10
could be burned in a couple of hours.

**Decision:** Weave $30 (~£24) one-time (still ~15x its ~$2 compute cost - it
keeps its commitment/positioning function at a kinder price); subscription
$10 (~£8)/month INCLUDING one portrait moment per day. Video metering moves
from monthly(10) to daily(1) buckets in the API - the daily moment is both the
cost guard and better product design (a gentle ritual; corpus and habit grow
daily). Deck ARR recomputed at ~£96/yr (upside 2028: £96M+).

**Alternatives considered:** keep £79-99 (deterrent at pilot); no fee at all
(loses commitment + commissioned-portrait positioning); monthly pool with
burst caps (complex, still binge-able).

**Consequences:** maximal daily users at the serverless GPU rung cost $6-7.5/mo
against $10 - thin until utilisation or dedicated GPUs cut per-moment cost;
acceptable because average usage sits well under cap and phase-A renders cost
~electricity. Watch the moment-usage distribution in the pilot.

---

## 2026-07-31 - Phase-A GPU architecture: no cloud GPUs; the home box pulls render jobs

**Status:** SUPERSEDED 2026-08-10 for LAUNCH (cloud-GPU launch — see "Launch economics v2"); the pull-worker remains the dev/test path and an emergency contingency

**Context:** $1,000 Activate credits ~ 3-6 months of core stack but only ~6
weeks with a dedicated L4. The Weave needs no self-hosted GPU (HF APIs + CPU);
the GPU load is really voice replies (latency-sensitive, small) and portrait
renders (heavy, async-tolerant by product design - metered allowance,
voice-first UX).

**Decision:** production runs no GPUs. Chat/ASR/embeddings/fallback-TTS stay on
HF Inference Providers; voice replies use the Kokoro fallback in cloud for now;
portrait renders travel by SQS to a pull-worker on the founder's GPU box
(fully-resolved job messages, S3 result-object as the completion signal, no
inbound connectivity to the house). Scaling ladder when reliability or volume
demands it: serverless per-second GPU (Modal/RunPod/HF Endpoints, ~$50-75/mo at
pilot volume) -> dedicated L4 (~$450-650/mo) only when utilisation fills it.

**Alternatives considered:** dedicated L4 from day one (burns the credits in
weeks at ~zero utilisation); Tailscale-connecting cloud inference to the box
for synchronous renders (couples request latency to home availability);
on-device inference (models need 16GB-class VRAM - not real).

**Consequences:** cloned-voice replies are NOT in cloud phase A (graceful
fallback voice; the clone still shines in local demos and sittings). The box
becomes light production infrastructure: worker uptime matters, renders queue
while it's dark (45-min TTL then refund). COGS at pilot ~ electricity.

---

## 2026-07-28 - Nan's sitting postponed; founder is the test subject first

**Status:** accepted

**Context:** the revamped sitting screen (hands-free VAD, guide video) has never
run on a device; first impressions with an elderly user are unrepeatable.

**Decision:** refine the whole sitting flow on Sean until it is smooth; nan's
sitting happens when it is earned.

**Consequences:** M4 moves behind M1; device retest on Sean's iPhone is the gate
for all sitting work.

---

## 2026-07-28 - Meet AWS Activate eligibility by upgrading the site, not arguing

**Status:** accepted

**Context:** Spendbase rejected the Activate application: site was
"waitlist-only" — needs product features, company/founding-team info, privacy.

**Decision:** add "The product, today" (honest status), "Who is behind this"
(founder + VENERAI LTD + company no. 17335051), and plain-English terms.html;
re-apply. All claims verifiably true.

**Alternatives considered:** dispute the review (slow, weak); apply direct-only
(does not fix the underlying thinness).

---

## 2026-07-25 - Apple: enroll fastest path now, organization before public release

**Status:** accepted

**Context:** TestFlight for family testing is gated only on enrollment; App
Review 5.1.1(ix) requires organization accounts for sensitive-data apps at
public release. UK Ltd DUNS (234989532) exists automatically.

**Decision:** individual enrollment is acceptable now (same-day-ish) — Apple has
an official individual→organization conversion preserving team and apps; convert
before any public App Store release. One Apple ID can hold multiple teams; no new
account needed.

---

## 2026-07-23 - Skip MongoDB for Startups

**Status:** accepted

**Context:** program offers up to $5k Atlas-only credits.

**Decision:** skip — the stack is deliberately Postgres+pgvector (relational
ownership chains, right-to-erasure cascades, twin-scoped vector search); DB cost
is a rounding error vs GPU COGS. Credit programs are for infrastructure we run,
not a reason to re-architect. Priority queue: AWS Activate, NVIDIA Inception,
Google for Startups (GCP terraform exists), MS Founders Hub, Cloudflare.

---

## 2026-07-16 - Guide avatar = pre-rendered clips (Option B), hands-free by VAD

**Status:** accepted

**Context:** the question bank is static content (45 stable ids); elderly users
struggle with buttons; a live avatar interviewer (Option C) needs reliability
tuning we cannot risk at pilot.

**Decision:** batch-render lip-synced clips of a consistent guide avatar speaking
every question (one resumable offline run; TTS remains the fallback), and add
silence-detection auto start/stop (4 s window, generous for storytellers).
Consent stays a deliberate button press — never auto-recorded. Guide voice: a
rights-clean synthetic female reference (Kokoro→Chatterbox); guide face must be
generated/licensed, not stock, before shipping.

---

## 2026-07-15 - "Ark" moat kept out of pitch materials until pilot data exists

**Status:** accepted

**Context:** deck's credibility rests on "built, not promised"; at zero families
the compounding-corpus moat is prospective.

**Decision:** slide 8 carries trust engineering; the Ark story waits for seed
stage with real corpus-growth metrics (instrument now: hours/twin, sittings,
conversations by twin age, churn by twin age).

---

## 2026-07-15 - Video is a metered allowance; quotas enforced in the API

**Status:** accepted

**Context:** 300 rendered replies/month would cost 3× the subscription; text and
voice cost pennies. Blow-up risk is exclusively video.

**Decision:** replies are voice-first; rendered "portrait moments" are a monthly
allowance (default 10/mo) with per-twin calendar-month Redis quotas counted
upfront and refunded on failure; messages get a 1000/mo anti-abuse fair-use
ceiling. Deck states COGS honestly (~15–20% of revenue early → 3–6% at scale).

---

## 2026-07-15 - Persistent-model render server (load once, render many)

**Status:** accepted

**Context:** the engine CLI reloaded ~15 GB per render — 60–90% of wall time
(992 s one-shot on WSL).

**Decision:** patch the engine (Dockerfile layer, upstream untouched) with an
ENGINE_SERVE stdin/stdout job loop + cached prompt embeddings; app.py manages the
resident process (crash respawn, id-matched jobs, LoRA falls back to one-shot).
Result: 264 s warm on WSL; pure GPU time on L4-class.

---

## 2026-07-14 - Bad-RAM verdict: checksum everything that touched the box

**Status:** accepted (standing rule)

**Context:** months of "storage" flakiness (dockerd deaths, CRC errors, 5
different sha256 of one file) was one defective DIMM (stick B: fixed-region
corruption at JEDEC 2133; stick A clean 3× passes). 3 of 12 model files were
silently corrupt vs HF checksums.

**Decision:** RMA stick B (Corsair lifetime warranty; full kit ships back). Any
artifact that transited the box before 2026-07-14 gets checksum-verified against
its source before trust. Weights on both boot personalities re-verified 12/12.

---

## 2026-07-13 - Single monorepo (+ this PM repo alongside)

**Status:** accepted

**Context:** services/apps/infra/docs share config contracts and move together;
solo founder.

**Decision:** one monorepo at `/Users/sbenson/Documents/VENER.AI` (no remote push
yet — explicit user choice). 2026-07-28: `concordia-venerai/` added inside it as
an independent git repo (gitignored by the monorepo) — PM source of truth that
survives conversation compaction.

---

## 2026-07-12 - Sealed waitlist: server cannot read its own signups

**Status:** accepted

**Context:** waitlist runs on a shared self-hosted box; user required signups
unreadable at rest there.

**Decision:** libsodium sealed-box encryption at capture (public key on server,
private key offline on the Mac) + blind HMAC email index for dedupe. Export =
authenticated fetch + offline decrypt (`scripts/waitlist-check.sh`). Also: zero
cookies/analytics on the site → no consent banner, just a dismissible note.

---

## 2026-07-10 - Voice = Chatterbox (MIT); XTTS-v2 rejected on licence

**Status:** accepted

**Context:** XTTS-v2 quality is fine but Coqui's CPML forbids commercial use and
the company is defunct; licences are a hard filter (registry: monorepo
docs/MODELS.md).

**Decision:** ResembleAI Chatterbox Multilingual V3 (MIT, 23 languages, ~10 s
reference, Perth watermarking helps EU AI Act disclosure). Lip-sync:
EchoMimicV3 (Apache-2.0 weights+code, native LoRA path for premium likeness
tier). Chat gpt-oss-120b, ASR Whisper, embeddings MiniLM (384-d), TTS fallback
Kokoro — all via HF with env-config endpoints.

---

## 2026-07-03 - Driver architecture: every backend behind an env-selected interface

**Status:** accepted

**Context:** cloud target uncertain at build time (GCP → AWS pivot happened;
credit programs may split workloads).

**Decision:** `VENER_ENV=local|gcp|aws` with per-concern overrides
(`MEDIA_BACKEND=fs|gcs|s3`, `QUEUE_BACKEND=local|pubsub|sqs`); Terraform for both
clouds with one `scale_tier` contract. Consequence: multi-cloud credits are
leverage, and local dev is a first-class deployment.

---

## 2026-07-03 - Consent and honesty are platform-enforced, not policy

**Status:** accepted (product-defining)

**Context:** the sector's failure mode is a trust scandal ("deadbots",
hallucinating grief-tech).

**Decision:** no recorded consent → no build (API-enforced); grounded retrieval
with refusal to invent; "what would they say" answers are labelled estimates;
the twin self-identifies as a living memory. Brand rules: "VENER" never without
".AI"; "ISO-aligned controls", never "certified"; pricing honesty (never free).
