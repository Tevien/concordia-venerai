# VENER.AI PM - decision log

Lightweight record of non-obvious choices (ADR style), so the "why" survives.
Newest first. One entry per decision; keep it short.

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

**Status:** accepted (implemented 5d70bc8; supersedes the £79-99 + £9-14 hypothesis)

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

**Status:** accepted (built same day, monorepo 13c34a2)

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
