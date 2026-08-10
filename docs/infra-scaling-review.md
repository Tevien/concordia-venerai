# Infra scaling review — phase-A architecture

**Date:** 2026-08-10 · **Reviewer:** infra-scaling-review agent · **Scope:** first full
review at 10 / 100 / 1,000 / 10,000 paying twins.

Sources read: `infra/aws/*.tf`, `services/render-worker/worker.py`,
`services/inference/app/render_jobs.py` + `app/main.py`,
`services/common/vener_common/{auth,config}.py`, `services/talking-head/app.py`,
`docs/DEPLOYMENT.md`, `docs/LOCAL-GPU-TESTING.md`, and this repo's
`project_board.md` / `decisions.md`. MEASURED numbers come from those; everything
else is tagged ASSUMED with the estimate stated.

---

## 0. The numbers this review stands on

| Figure | Value | Status |
|---|---|---|
| Warm portrait render, 512², 125 frames (25 fps → 5.0 s video) | 264 s ⇒ 0.47 f/s | MEASURED (board, `e52a69a`) |
| Native 768² render rate | ≈ 0.056 f/s ⇒ 125 f ≈ 37 min | MEASURED (guide-clip batch 2026-08-04; not on the board — record it) |
| Render length clamp (`frames_for`) | 49–201 frames ⇒ 103–425 s per job at 512² | MEASURED (code, talking-head `app.py`) |
| Voice synth (Chatterbox) | ~5 s/sentence, ~1.6 s short phrases | MEASURED (LOCAL-GPU-TESTING.md) |
| Local chat floor (Qwen3-4B, llama.cpp on 3060) | 63+ tok/s | MEASURED (board) |
| Weave external-AI cost | $0 proven end-to-end (2026-08-02 milestone) | MEASURED |
| Weave all-in compute | ~$2 | RECORDED (decision 2026-07-31 pricing) |
| Quota law | 1 portrait moment/twin/day (Redis day bucket, count-upfront, refund-once); 1,000 msgs/mo fair-use | MEASURED (config.py, auth.py, main.py — enforced) |
| Job TTL | 45 min from enqueue → client sees "failed", refund fires | MEASURED (render_jobs.py) |
| AWS credits vs starter footprint | $1,000 ≈ 3–6 months ⇒ starter ≈ $250–350/mo incl. NAT ~$37, ALB ~$25, Fargate ~$180, RDS t4g.medium ~$66, ElastiCache ~$25 | RECORDED range; breakdown ASSUMED |
| Serverless rung at pilot volume | ~$50–75/mo | RECORDED (decision 2026-07-31) |
| Dedicated L4 | ~$450–650/mo | RECORDED (decision 2026-07-31) |
| Per-moment wall time on the box | ≈ 5 min (render 4.4 min + S3 transfers + synth wait; worker is deliberately serial, `MaxNumberOfMessages=1`) | DERIVED from measured parts |
| Daily-quota utilisation | 40–60 % of twins take their moment on a given day | ASSUMED (decision log: "average usage sits well under cap"; pilot must measure) |
| Evening clustering | ~65 % of a day's moments land in an 18:00–22:00 window | ASSUMED (families gather in evenings; uniform-day averages lie) |

---

## 1. Moments/day ceiling on the 5070 Ti (interactive 512 tier)

**Service rate:** 1 moment ≈ 5 min ⇒ **12 moments/hour**, 288/day at an impossible
24/7. Realistic availability (shared box, Windows/WSL, sleep, other projects) is
10–13 h/day ⇒ **~120–160 moments/day realistic ceiling**. At native 768² the same
moment takes ~37 min of GPU — that resolution is a pre-rendered/archival tier only,
never interactive.

**Demand vs ceiling (ASSUMED 40–60 % utilisation, 65 % in the 4-h evening window;
evening capacity = 48 moments):**

| Twins | Moments/day | Evening-window jobs | Peak arrival vs 12/h service | Verdict |
|---|---|---|---|---|
| 10 | 4–6 | 3–4 | trivial | waits < 10 min. Fine. |
| 100 | 40–60 | 26–39 | 55–80 % of window capacity | Works on average; Poisson bursts push waits past 40 min on the worst evenings at the top of the range. **The edge.** |
| 1,000 | 400–600 | 260–390 | λ ≈ 65–98/h vs μ = 12/h | Queue explodes within ~10 min of the evening; even a perfectly uniform day (impossible) exceeds the 288/day theoretical max. **Phase-A is arithmetically dead here.** |
| 10,000 | 4,000–6,000 | 2,600–3,900 | — | Fleet territory (see §4). |

**Plain statement: the home box carries ~110–130 paying twins before evenings start
failing jobs. 1,000 twins breaks phase-A outright — not by margin, by 8–50×.**
The serverless rung must be live well before ~150 twins.

## 2. Queue depth × render time vs the 45-min TTL

Service time 5 min, TTL 45 min ⇒ a job enqueued behind **≥ 8 waiting jobs** is
already condemned (wait > 40 min + own render > TTL). Sustained arrivals above
**12/hour for ~45 minutes** guarantee client-visible failures. That threshold is
crossed on ordinary evenings somewhere around **~60 moments/day demanded
(~110–130 twins at assumed utilisation)**.

Independent failure mode: **any worker outage ≥ 45 min fails 100 % of jobs queued
in that window** (refund-once fires; families see "the studio may be offline").
The box is currently loaned out and the RAM RMA will take it down again — this is
today's reality, not a tail risk.

**Bug-grade finding (cheap fix):** `worker.py` never compares `job["enqueued_at"]`
to the 45-min TTL, and the render queue retains messages for 4 days
(`message_retention_seconds = 345600`). After any outage the returning worker
burns GPU rendering jobs the client was already told failed (and refunded), at
~5 min each, ahead of live jobs. One `if` statement fixes it.

Also note: the worker deletes messages on success *and* failure (by design, to
avoid re-burning GPU), so the renders DLQ only ever catches crash redeliveries —
don't expect job-failure forensics there; the `.failed` S3 markers are the record.

## 3. Weave (onboarding) throughput ceiling — for the wave plan

Weave GPU work runs on the **3060** (local Whisper ASR + chat chain when NIM is
cold) plus CPU stages in cloud. Per weave, fully local: ASR of a ~45-min sitting
at ASSUMED 4–6× realtime ⇒ 8–12 min, LLM extraction/persona at 63 tok/s ⇒
15–25 min ⇒ **25–40 min of 3060 time per weave** (NIM warm: 10–15 min). At ~10 h/day
box availability ⇒ ~20 weaves/day.

**Clean numbers for the launch specialist:**
- **Hard ceiling: ~140 weaves/week** (NIM warm), **~100/week** fully local.
- **Recommended invitation-wave size: ≤ 50 invitations/week.** Even 100 %
  same-week sittings clustered on a weekend (30 of 50 in 2 days = 15/day) stays
  under the worst-case fully-local ceiling with headroom for daily moments on
  the same card. Waves above 50/week require the serverless ASR spillover to be
  live first.
- Sitting *uploads* don't touch the box (phone → ALB → S3), but ASR audio
  transits home broadband both ways — see SPOF-2.

## 4. The serverless per-second GPU rung — priced

Providers (ASSUMED current pricing, 2026-08; verify at signup):
- **Modal**: L4 24 GB ≈ **$0.80/hr** ($0.000222/s), A10G ≈ $1.10/hr; per-second
  billing, warm-pool option.
- **RunPod serverless** (flex workers, 24 GB class L4/A5000): ≈ **$1.10/hr active**
  (~$0.00031/s), cheaper idle-priced warm workers available.

**Per moment:** an L4 is ASSUMED 1.3–1.6× slower than the 5070 Ti ⇒ 450–530 s
active per moment ⇒ **$0.10–0.16 active**, realistically **$0.15–0.25/moment**
with warm-pool amortisation (the persistent-model server matters here: the ~15 GB
cold load must be paid by a warm pool, not per job).

**Per weave:** ASR ~5 min L4 + LLM extraction ⇒ **$0.20–0.50 GPU**; all-in weave
stays ≈ $2 (recorded) ⇒ the $30 fee keeps its ~15× margin. Not a concern.

**Margin impact on $10/mo:**

| Subscriber behaviour | Serverless GPU/mo | Margin before CPU-infra share |
|---|---|---|
| Max usage (30 moments) | $4.50–7.50 (matches recorded $6–7.5) | **$2.50–5.50 — thin** |
| Average (ASSUMED 50 %) | $2.25–3.75 | $6.25–7.75 — healthy |

The rung is affordable *because* of the 1/day quota and sub-cap averages; a
"power-user" cohort at max usage is a margin problem the pilot must measure.
**Trigger to leave the rung:** sustained serverless spend > $450/mo (dedicated-L4
parity, recorded) ≈ **~2,000–2,500 moments/mo**.

**Honesty flag on the deck's "3–6 % COGS at scale":** at measured render speeds,
10,000 twins ⇒ 4,000–6,000 moments/day with an evening peak needing ~30 dedicated
L4s (~$16.5k/mo) + ~$9k/mo serverless burst ⇒ **~$2.50–3/twin/mo GPU = 25–30 % of
revenue**, not 3–6 %. The 3–6 % claim requires the cheaper render path
(LivePortrait-class reenactment / step-distilled or mouth-only second stage — the
quality-envelope roadmap's own rungs) or heavy peak-smoothing. Say so in any
investor conversation before they do the arithmetic.

## 5. Cost per subscriber per tier

| Twins | Cloud CPU stack (ASSUMED) | GPU | Cost/sub/mo | Margin/sub on $10 |
|---|---|---|---|---|
| 10 | starter $250–350 (credits absorb) | home box (~£20–40/mo electricity, ASSUMED) | $27–39 | negative; credits + electricity only. Expected. |
| 100 | starter $250–350 | home box + first serverless spillover $50–75 | $3.00–4.25 | **$5.75–7.00** — the sweet spot, but reliability rides on one house |
| 1,000 | growth tier $900–1,300 (ASSUMED: r6g.large DB multi-AZ, Redis replica, more tasks) | serverless/early-dedicated $2,600–3,800 | $3.50–5.10 | $4.90–6.50 — works, **but only off phase-A** |
| 10,000 | enterprise $3–5k | ~30×L4 + burst ≈ $25k | $2.80–3.00 | ~$7 — needs the cheap-render path to reach deck COGS |

Break-even on cash (credits exhausted, starter stack, home GPU): **~30–35
subscribers** cover the $250–350 fixed cloud. Margin crosses genuinely positive
territory at ~40 twins and stays positive through every tier *if* the GPU rung
climbs on the triggers below.

## 6. SPOF register (ranked by blast radius)

| # | SPOF | Blast radius | Evidence | Cheapest mitigation |
|---|---|---|---|---|
| 1 | **Home GPU box** (5070 Ti + 3060, WSL under Windows) | ALL portrait moments, local ASR (weaves slow to HF-paid), chat floor, cloned voice. Outage ≥ 45 min = every queued moment fails. | Bad-RAM history (stick-B RMA pending — **full kit ships = planned downtime**); failing-NVMe docker deaths; CUDA contexts die on hibernate; **currently loaned out** (capacity today = 0); shared with other projects | Serverless spillover wired NOW (same container on Modal/RunPod, fired by queue-age alarm) — turns box death into a cost blip. Then: UPS, Ubuntu boot over WSL, `run.sh` under systemd/Task-Scheduler auto-start |
| 2 | **Home broadband** | Same as #1 while ISP is down; also per-job transfer tax — worker re-downloads portrait+voice-ref (and future face LoRA, 100–300 MB, **uncached**) every job | worker.py has no local artifact cache; uplink ASSUMED 15–20 Mbps residential | Local content-addressed cache keyed on S3 URI (~20 lines); 4G-dongle failover ≈ £15/mo |
| 3 | **NIM free tier** (primary chat) | Chat latency degrades to the 63 tok/s floor (contending with ASR+voice on the 3060), then paid HF. Free tiers are revocable without notice. | Chain NIM→local→HF proven (board: floor carried a whole weave) — degradation, not outage | Budget a paid second cloud LLM (small-model API, ~$10–40/mo at 1k twins, ASSUMED) before waves start |
| 4 | **Single founder** | Everything: ops, RMA logistics, consent gates, support, and the "archive outlives us" promise itself | solo; no remote push of the monorepo (explicit choice) — **the code's only full copy is one Mac + one house** | Push to a private remote (decision to revisit); document worker runbook; wind-down data-escrow note in terms (see §7) |
| 5 | **Raspberry Pi** (marketing site + sealed waitlist) | Acquisition funnel; an invitation wave landing on a dead site | Behind Cloudflare, but origin is one Pi on home power/net | Static site → Cloudflare Pages ($0); Pi keeps the proxy role only until then |
| 6 | **Redis fail-open quotas** | ElastiCache down ⇒ quota checks fail OPEN (auth.py, deliberate) ⇒ unmetered render enqueues exactly when things are already broken; starter tier runs **0 replicas** | auth.py `quota_exceeded` catches and allows | Accept at pilot (documented trade-off); belt-and-braces: worker-side per-twin daily cap (it has `twin_id` + `enqueued_at` in the message) |
| 7 | **Email deliverability** (OTP-only login at cloud launch) | Nobody can log in; SES reputation is an availability dependency the decision log already names | decision 2026-08-02 | SES + a fallback SMTP (SMTP2GO already proven for /admin) |
| 8 | **Credit cliff** | $1,000 ≈ 3–6 months; after that $250–350/mo cash | recorded | Break-even math above: ~30–35 subs; time waves accordingly |

## 7. Data durability at scale — the "archive outlives us" promise, costed

**GB/twin (ASSUMED from artifact kinds):** sitting answer recordings (audio-first,
17–45 answers) ~0.05–0.2 GB; video sittings up to the 500 MB/file cap and the
planned 30–60 s enrollment video push a video-rich twin to 2–4 GB; portrait
photos + 8-ref voice pack ~0.05 GB; future face LoRA 0.1–0.3 GB; **retained daily
moments ~2–4 MB × 365 ≈ 0.7–1.5 GB/twin/YEAR** (nothing expires them). Blended:
**~1–3 GB/twin in year one, growing ~1 GB/yr**.

| Twins | Media store | S3 standard (ASSUMED $0.024/GB-mo) | With lifecycle (IA/Glacier for moments+masters) |
|---|---|---|---|
| 100 | 0.1–0.3 TB | $3–7/mo | ~$1–3/mo |
| 1,000 | 1–3 TB | $25–70/mo | ~$8–25/mo |
| 10,000 | 10–30 TB | $250–700/mo | ~$80–250/mo |

**Backup posture today (from `data.tf`):** RDS 30-day PITR + deletion protection +
final snapshot — good. S3: KMS, versioned, public-blocked — but **same account,
same region, no lifecycle, no replication**. A compromised AWS root or an
eu-west-1 event takes primaries *and* their versions together. The promise is
currently a policy, not an architecture.

**What the promise costs:** cross-account replication to a locked backup account
with Glacier Deep Archive (ASSUMED $0.00099/GB-mo) ≈ **$1/TB-mo** — $0.30/mo at
pilot, ~$30/mo at 10k twins. Plus the export tooling the board already lists
(open-format per-twin export; wind-down window in terms). Buy it now; it is the
cheapest promise the company will ever keep.

**GDPR erasure flag:** on a **versioned** bucket, plain deletes leave noncurrent
versions readable. The /admin ERASE path and the known delete-order media-leak
backlog item must delete *versions* (or a noncurrent-version-expiry lifecycle
rule must back them up). Right-to-erasure is a consent promise — this is a
correctness gap, not an optimisation. Same for the future backup account:
erasure must propagate.

## 8. Trigger table — watch these on /admin

/admin today shows services/homes, users, twins, storage GB and live logs, but
**none of the queue metrics below — adding three numbers (render-queue depth,
oldest-message age, TTL-failures today) is a prerequisite for this table.**

| Climb | Observable metric | Threshold |
|---|---|---|
| Home box → serverless spillover **(wire now, fire automatically)** | Render-queue oldest-message age | > 15 min while worker healthy, any day |
| | TTL-failed jobs (refund-once markers/day) | ≥ 3 in one evening, twice in 7 days |
| | p95 moment wait (enqueue→object) | > 15 min for 3 consecutive days |
| | Paying twins | > 120, full stop — the arithmetic ceiling |
| | Box dark 18:00–22:00 | > once/week |
| Serverless → dedicated GPU | Serverless GPU spend | > $450/mo sustained 2 months (L4 parity) ≈ >2,000–2,500 moments/mo |
| | Serverless p95 cold-start | > 90 s degrading the "minutes later" promise |
| Weave capacity → serverless ASR | Weave backlog (enqueue→twin ready) | > 24 h, or any planned wave > 50 invitations/week |
| starter → growth tier (Terraform `scale_tier`) | ECS service CPU / DB connections | > 70 % sustained, or > ~500 twins |
| Pi → Cloudflare Pages | First origin outage during a wave, or waitlist wave planned | do it before the next wave regardless — it's free |
| Storage lifecycle + backup account | Media GB on /admin | > 500 GB for lifecycle; replication **now** (promise-critical, ~$1/TB-mo) |

## 9. Top-5 actions (cheapest meaningful first)

1. **Worker TTL check** (~5 lines): skip any job where `now − enqueued_at > 45 min`
   — stops post-outage GPU burn on already-refunded jobs. Add a content-addressed
   local cache for portrait/voice-ref/LoRA while in the file.
2. **Three queue metrics on /admin** (SQS `GetQueueAttributes` + refund-marker
   count): depth, oldest-age, TTL-failures — the entire trigger table is
   unwatchable without them.
3. **Wire the serverless spillover before the first invitation wave** (Modal or
   RunPod, same talking-head container, warm-pool, fired by the queue-age alarm):
   ~$50–75/mo at pilot (recorded), and it converts SPOF-1 from product outage to
   cost line. This is the single change that lets 100 twins be safe and 1,000 be
   possible.
4. **Durability pack**: S3 lifecycle (moments + 768 masters → IA/Glacier),
   cross-account Glacier Deep Archive replication (~$1/TB-mo), version-aware
   erasure in /admin ERASE, and push the monorepo to a private remote. This is
   the "archive outlives us" promise made structural.
5. **Move the marketing site to Cloudflare Pages** and hold **invitation waves at
   ≤ 50/week** until (3) is live and the RMA'd RAM is back and soak-tested.

---

*Honest bottom line: phase-A is a correct pilot architecture with a measured
ceiling of ~110–130 paying twins, and it fails at 1,000 by an order of magnitude —
which is fine, because the serverless rung is priced, thin-but-positive, and one
alarm away. The two things that are NOT fine to defer: the spillover path and the
backup account. Both cost less per month than one family's subscription.*
