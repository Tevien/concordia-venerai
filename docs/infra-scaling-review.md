# Infra scaling review — cloud-GPU launch baseline (rev 2)

**Date:** 2026-08-10 · **Reviewer:** infra-scaling-review agent · **Scope:** full
review at 10 / 100 / 1,000 / 10,000 paying twins.

**Rev 2 supersedes the same-day phase-A review** after the founder decision
"Launch economics v2" (decisions.md 2026-08-10): **launch compute is AWS GPU —
serverless per-second from subscriber #1, a dedicated full-time L4 VM at scale;
pricing is $10/mo subscription only (no $30 weave fee); local GPUs are dev/test
plus an emergency contingency valve.** The home box's ceilings, previously the
launch constraint, are retained below only as the contingency valve's rating.

Sources: `infra/aws/*.tf`, `services/render-worker/worker.py`,
`services/inference/app/{render_jobs,main}.py`,
`services/common/vener_common/{auth,config}.py`, `services/talking-head/app.py`,
`docs/DEPLOYMENT.md`, `docs/LOCAL-GPU-TESTING.md`, this repo's
`project_board.md` / `decisions.md`. MEASURED numbers from those; everything
else tagged ASSUMED.

---

## 0. The numbers this review stands on

| Figure | Value | Status |
|---|---|---|
| Warm render, 512², 125 f (25 fps → 5.0 s video), **RTX 5070 Ti** | 264 s ⇒ 0.47 f/s | MEASURED (board, `e52a69a`) |
| Render clamp | 49–201 frames ⇒ 103–425 s on the 5070 Ti | MEASURED (talking-head `app.py`) |
| **L4 slowdown factor vs 5070 Ti** | **2.0× ASSUMED (range 1.8–2.5×)** — L4 is Ada, ~300 GB/s memory bandwidth vs ~900 on the 5070 Ti; video diffusion is bandwidth-bound. All L4 numbers below carry this assumption; **measure it in week one on serverless and re-run this review if it falls outside 1.8–2.5** | ASSUMED |
| Per-moment on L4-class | render ~530 s (475–660) + synth ~30 s + S3 I/O ⇒ **~9.5 min wall (8.5–11.5)** ⇒ ~6.3 moments/hr | DERIVED |
| Voice synth (Chatterbox) | ~5 s/sentence, ~1.6 s short | MEASURED |
| Weave compute, serverless | ~$2–3 all-in (ASR ~5 min L4 + LLM + misc APIs) | RECORDED (Launch economics v2) |
| Serverless GPU pricing (ASSUMED 2026-08): Modal L4 $0.80/hr, A10G $1.10/hr; RunPod serverless 24 GB flex ~$1.10/hr active | ⇒ **$0.15–0.25/moment** incl. warm-pool/cold-start amortisation | ASSUMED, matches recorded $0.15–0.25 |
| Dedicated L4 VM (g6.xlarge-class, eu-west-1) | ~$580–660/mo on-demand; **~$350–400/mo 1-yr savings plan** | RECORDED (decision arithmetic) |
| Core CPU stack | ~$150–250/mo starter (credits ≈ 3–6 months) | RECORDED |
| Quota law | 1 moment/twin/day, count-upfront, refund-once; 1,000 msgs/mo | MEASURED (enforced in code) |
| Job TTL | 45 min enqueue→result, then client-visible failure + refund | MEASURED (render_jobs.py) |
| Daily-quota utilisation | 40–60 % of twins take their moment on a given day (central: 50 %) | ASSUMED — the pilot's single most important measurement |
| Evening clustering | ~65 % of a day's moments inside 18:00–22:00 | ASSUMED (families gather in evenings; uniform-day averages lie) |
| Contingency valve (home 5070 Ti) | ~5 min/moment ⇒ 12/hr; evening window ≈ 48 moments | MEASURED-derived (rev 1) |

---

## 1. Serverless as PRIMARY: cold starts vs the moment experience

The product promise is async: voice plays first, "the video arrives minutes
later". What "minutes" means under serverless:

- **Warm path:** ~9.5 min render wall on L4-class. That is the floor families
  experience regardless of architecture.
- **Cold start, engineered** (weights baked into the image/volume, snapshot or
  flashboot restore, warmup render on spawn — the board's "synthetic warmup"
  item is now launch-critical): **+60–180 s** ⇒ p95 moment latency **~11–13 min**.
  Acceptable against the stated product shape.
- **Cold start, naive** (image pulls ~15 GB of weights from HF at spin-up):
  +8–15 min ⇒ p95 approaches 20–25 min and, with one retry, threatens the
  45-min TTL. **This is the difference between a working launch and a flaky
  one, and it is pure engineering done before subscriber #1.**
- Optional: a warm-pooled worker 18:00–22:00 costs ~$3.2/day ≈ **$96/mo**
  (Modal L4). Only buy it if measured p95 says families notice — at a 9.5-min
  floor, 2 extra cold-start minutes are likely invisible.

**Queueing disappears under serverless**: each moment gets its own worker
(scale-out per job), so the evening peak becomes a *concurrency* number, not a
queue. The TTL almost never fires from load — see §5.

## 2. Capacity per scenario (serverless-primary)

| Twins | Moments/day (40–60 % util) | Evening jobs (4 h) | Peak concurrent GPU workers | Pure-serverless GPU $/mo (@ $0.20) | Verdict |
|---|---|---|---|---|---|
| 10 | 4–6 | 3–4 | 1–2 | $25–35 | Pure serverless. Trivial. |
| 100 | 40–60 | 26–39 | 3–5 | $240–360 | Pure serverless works; the L4 decision point is near (§3) |
| 1,000 | 400–600 | 260–390 | 15–25 | $2,400–3,600 | Hybrid: ~5 SP L4s + serverless spill ≈ $3.1k/mo — roughly cost-neutral vs pure serverless, better latency; provider quota must be raised either way |
| 10,000 | 4,000–6,000 | 2,600–3,900 | **150–250** | fleet: ~30 SP L4s + ~$9k spill ≈ $25k/mo | Fleet + spill. **Provider GPU-concurrency quota is the new ceiling** — default serverless quotas are 10–100 workers (ASSUMED); raises and a second provider are prerequisites, not options |

The binding constraint has moved: from "one home GPU's evening" to
(a) cold-start engineering, (b) provider concurrency quotas, (c) unit cost.

## 3. The crossover: serverless → dedicated L4 VM

L4 hybrid cost = L4 + the serverless spill it still needs at peak (a single VM
serializes at ~6.3/hr; the evening always overflows it — spill is a permanent
feature, not a failure).

**Crossover arithmetic** (serverless @ $0.20/moment central):

- vs **on-demand L4** ($580–660/mo): 2,900–3,300 moments/mo ≈ **97–110/day**
  ≈ 195–220 subs at 50 % util.
- vs **1-yr savings-plan L4** ($350–400/mo) **+ spill (~$30–80/mo at that
  scale)**: ~1,900–2,400 moments/mo ≈ **65–80 moments/day ≈ 130–160 subs at
  50 % utilisation** — my central answer.
- The founder's "~100 subs" holds only at the aggressive corner: savings plan
  AND (utilisation ≥ 60 % OR serverless at the $0.25 top of range). Not wrong
  — slightly early. There is also a soft reason to adopt near 100: resident
  models delete cold-start latency for the majority of moments.

**Recommendation:** trigger on the observable, not the sub count — **adopt the
SP L4 when serverless GPU spend exceeds ~$400/mo for 2 consecutive months**
(that is the ~65–80 moments/day line). Expect that between 100 and 160 subs.

## 4. Can one L4 actually carry 100–150 subs? (the founder's implicit claim)

At the ASSUMED 2.0× slowdown: one L4 does **~150 moments/day** flat-out 24/7
(121–168 across the 1.8–2.5× range), and weaves are noise on top (5–10/week at
that scale ≈ minutes of ASR, run off-peak).

- **Volume: YES.** 100–150 subs demand 50–90 moments/day — comfortably under
  ~150/day capacity.
- **Evening peak: NO, not alone.** 100 subs ⇒ ~32 jobs in the 4-h window =
  ~8.1/hr arrivals vs ~6.3/hr service ⇒ backlog ~7 jobs by 22:00 ⇒ late-window
  waits ~70 min ⇒ **TTL breaches on ordinary evenings**. At 150 subs it is
  worse (≈12/hr vs 6.3/hr).
- **With serverless spill: YES, comfortably.** Spill absorbs ~5–10 moments per
  evening at 100 subs (~$30–60/mo), ~20–30 at 150 subs (~$120–180/mo). The
  L4+spill pair carries to **~150–200 subs**; buy L4 #2 when spill spend itself
  sustains > ~$400/mo. Beyond that, N×L4 baseline + spill for the peak,
  ~+1 L4 per ~100–130 subs (at 50 % util).

**Plain statement: "one L4 at ~100 subs" is true for volume and false for the
evening peak — the correct unit of adoption is "L4 + serverless spill", and it
works. Never deploy the VM and turn the serverless path off.**

## 5. The 45-min TTL and refund logic under serverless

- Warm/engineered-cold worst path: cold 3 min + render 11.5 min ≈ 15 min ≪ 45.
  With per-job workers there is no queue wait, so **under normal operation the
  TTL never fires** — it becomes what it should be: an outage detector
  (provider down, quota exhausted, pathological cold start).
- The `job_status` refund-once logic (Redis marker, S3-object-existence
  protocol) carries over to serverless workers **unchanged** — the worker
  contract is "write mp4 or .failed marker to S3"; nothing in it assumes the
  home box. Keep SQS as the dispatch bus (provider autoscalers can consume it,
  and it keeps the contingency valve plug-compatible).
- Keep TTL at 45 min. Add the metric that makes it diagnosable:
  **enqueue→worker-start p95** (cold-start health) alongside enqueue→ready.
- **Contingency-valve bug stays open:** `worker.py` never checks
  `job["enqueued_at"]` against the TTL and the queue retains messages 4 days —
  a valve activated after an outage would burn GPU on already-refunded jobs.
  Five-line fix; do it even though the box is no longer the launch path.

## 6. Onboarding (weave) throughput — no longer an infrastructure ceiling

Serverless parallelism removes the GPU cap on weaves: each is ~5 min of L4 ASR
+ LLM calls, ~$2–3 all-in (recorded), and any number can run concurrently
within provider quota. **The wave-size constraint moves from GPU to founder
support bandwidth and cash exposure**: a 100-invitation wave costs ~$200–300
of weave compute against ~$1,000 collected at signup (billing is
month-in-advance) — cash-positive by design; worst-case churn exposure is the
recorded ~$2–3 per month-one canceller.

**Clean number for the launch specialist: waves are no longer GPU-capped;
size them to support capacity — ASSUMED ≤100 invitations/week for a solo
founder — and to the cold-start/quota engineering being proven (§1, §8).**

## 7. Cost per subscriber and margin — $10/mo only, commission sensitivity

Costs per active sub/mo: GPU moments at 50 % util ≈ $3.00 serverless
($2.25–3.75), similar under L4-hybrid at its adoption scale; core-stack share
falls with scale; weave ~$2–3 once, in month one.

| Scale | Cost/sub/mo (GPU + infra share) | Margin @ 0 % (net $10) | @ 15 % ($8.50) | @ 30 % ($7.00) |
|---|---|---|---|---|
| 10 | $17–28 (fixed $150–250 dominates) | negative — credits absorb; expected | negative | negative |
| 100 | $4.50–5.50 | **+$4.50–5.50** | +$3.00–4.00 | +$1.50–2.50 |
| 1,000 | $3.90–4.40 | +$5.60–6.10 | +$4.10–4.60 | +$2.60–3.10 |
| 10,000 | $2.90–3.40 | +$6.60–7.10 | +$5.10–5.60 | +$3.60–4.10 |

- **Max-usage subscriber** (all 30 moments): GPU $4.50–7.50 ⇒ at 30 %
  commission the margin is **−$0.50 to +$2.50 — an IAP-billed power user can be
  margin-negative.** The decision log already names web checkout (~3 %) vs
  store billing as first-order packaging; this table is the arithmetic behind
  it. Hand to launch-specialist + compliance-review (IAP rules decide what's
  permitted per jurisdiction).
- **Month one** per new sub at 30 % commission: $7.00 − weave $2.50 − moments
  ~$3 − infra ≈ break-even; positive from month two. Fine — but it means
  month-one churn is the entire acquisition cost, as the decision records.
- **Cash break-even** (covering the $150–250 core stack, post-credits):
  ~**22–38 subs** at 0 % commission, ~28–47 at 15 %, ~38–63 at 30 %.
- Honesty flag retained from rev 1: at measured render speeds, 10k-scale GPU
  is ~$2.5–3/sub = 25–30 % of gross — the deck's "3–6 % at scale" needs the
  cheap-render path (LivePortrait-class reenactment / distillation) or does
  not survive arithmetic.

## 8. SPOF register (re-ranked for the cloud-GPU launch)

| # | SPOF | Blast radius | Evidence | Cheapest mitigation |
|---|---|---|---|---|
| 1 | **Single founder** | Everything: ops, billing, consent gates, support, the "archive outlives us" promise | solo; bus factor 1 | Runbooks (exist), scheduled-export automation, wind-down escrow note in terms |
| 2 | **AWS eu-west-1 single region** | The entire product — API, DB, S3, SQS, and now the GPU dispatch — until restore | all of `infra/aws` is single-region; backups same-account | Cross-account + cross-region backup replication (the §9 durability pack); a written region-recovery runbook. Multi-region active is NOT warranted yet |
| 3 | **Serverless GPU provider** | All portrait moments and weave ASR while it's down or quota-throttled; free/low quotas are revocable and raise-gated | new primary compute; single account today | **Same container live on TWO providers (Modal + RunPod), env-var switch, drilled monthly**; quota raises requested before each wave |
| 4 | **Un-pushed monorepo** | Irrecoverable loss of the codebase that now IS the product and its infra — one Mac, one house | explicit no-remote choice (2026-07-13) — made under a different risk profile | Push to a private remote today; cost: zero |
| 5 | **NIM free tier** (primary chat) | Chat degrades to fallbacks; the home-box local floor is no longer a production tier | chain NIM→local→HF; local leg now dev/test | Budget a paid LLM API second leg (~$10–40/mo at 1k subs, ASSUMED) |
| 6 | **Redis fail-open quotas** | ElastiCache down ⇒ unmetered enqueues ⇒ **unbounded serverless spend** (this got WORSE: the flood now costs real dollars, not home electricity) | auth.py deliberate fail-open; starter = 0 replicas | Provider-side concurrency cap + daily spend alarm as the backstop |
| 7 | **Email deliverability** (OTP-only login) | Nobody logs in | decision 2026-08-02 | SES + proven SMTP2GO fallback |
| 8 | **Raspberry Pi** (site + waitlist) | Acquisition funnel during waves | one Pi behind Cloudflare | Static site → Cloudflare Pages ($0) |
| 9 | **Credit cliff** | $1,000 ≈ 3–6 mo of core stack; GPU now bills in real dollars from day 1 | recorded | Break-even at ~22–63 subs (§7); time waves to it |

**Contingency section (demoted from launch path):** the home box — bad-RAM RMA
pending (full kit ships = downtime), WSL/hibernate kills CUDA, failing-NVMe
history, shared/loaned (capacity today: zero), home broadband. As a valve its
rating is **12 moments/hr, ~48 per evening window** — enough to cover ~100
subs' evening demand in a provider outage, IF it is kept drilled (§10) and the
worker TTL bug is fixed. A valve that is never exercised is fiction.

## 9. Data durability — unchanged in substance, promise-critical

(Unchanged from rev 1; the launch-compute change does not touch it.)
~1–3 GB/twin year one (+~1 GB/yr retained moments); S3 versioned but
same-account, same-region, **no lifecycle, no replication** — the "archive
outlives us" promise is policy, not architecture. Cross-account Glacier Deep
Archive replication ≈ **$1/TB-mo** ($0.30/mo at pilot). **GDPR-erasure gap:
versioned buckets keep noncurrent versions readable after plain deletes —
/admin ERASE and the delete-order backlog item must delete versions.** RDS
30-day PITR + deletion protection is good.

## 10. Trigger table — watch on /admin (+ the two new numbers)

/admin needs: moments/day, **GPU $/day**, enqueue→start p95 (cold start),
enqueue→ready p95, TTL failures/day, provider quota headroom. Without these
the table below is unwatchable.

| Climb | Observable metric | Threshold |
|---|---|---|
| Launch gate (before subscriber #1) | cold-start p95 on the baked serverless image | < 3 min proven, warmup render on spawn verified |
| Serverless → +1 SP L4 (keep spill forever) | serverless GPU spend | **> ~$400/mo for 2 consecutive months** (≈ 65–80 moments/day ≈ 130–160 subs @ 50 % util); adopt as early as ~100 subs only if measured util ≥ 60 % or cold-start p95 > 3 min persists |
| +L4 #N | spill spend | > ~$400/mo sustained, per additional L4 (~every 100–130 subs) |
| Provider quota raise / activate 2nd provider | peak concurrent workers | > 50 % of granted quota |
| Contingency valve health | monthly drill: 1 job end-to-end through the home box | any failure ⇒ fix or formally retire the valve claim |
| Spend backstop | GPU $/day | > 2× trailing-week average ⇒ page founder (quota fail-open guard) |
| starter → growth tier | ECS CPU / DB connections | > 70 % sustained, or > ~500 subs |
| Pi → Cloudflare Pages | before the next wave | free — just do it |
| Durability | media GB | lifecycle at 500 GB; cross-account replication NOW |

## 11. Top-5 actions under the new baseline (cheapest meaningful first)

1. **Push the monorepo to a private remote** (minutes, $0): the un-pushed repo
   is now the cheapest catastrophic risk on the register.
2. **Engineer the serverless image properly before subscriber #1**: weights
   baked into image/volume, snapshot/flashboot, synthetic warmup render on
   spawn, cold-start p95 measured < 3 min — and **measure the real L4
   slowdown factor** (this review assumes 2.0×; outside 1.8–2.5× the
   crossover and capacity numbers must be re-run).
3. **Two-provider portability from day one** (Modal + RunPod, same container,
   env-var switch, monthly drill) + provider concurrency-quota raises ahead
   of each wave.
4. **Metrics + spend backstop on /admin**: the six numbers in §10 plus a
   GPU-$/day alarm — quotas fail open, and the flood now costs real money.
5. **Durability pack + valve hygiene**: S3 lifecycle, cross-account Glacier
   replication, version-aware erasure; fix the worker.py TTL check (5 lines)
   and drill the contingency valve monthly.

## 12. Elasticity audit at 10k–100k: what scales by itself vs by hand

The question: at 10k or 100k subscribers, does the stack load-balance, spin up
compute, and absorb database load *automatically*? Verified against
`infra/aws/ecs.tf` — the answer is three different answers.

**Load model (ASSUMED):** 10k subs ⇒ ~2–6 rps API average, 10–25 rps evening
peak, ~300–800 concurrent chat WebSockets; 100k ⇒ 20–60 rps avg, 100–250 rps
peak, ~3k–8k concurrent WebSockets, ~1–2k DB queries/s at peak.

**A. Scales automatically today (within tier caps):**
- **ALB**: managed, effectively unlimited at these rates. Yes.
- **ECS services** (web/builder/inference): target-tracking autoscaling at
  60 % CPU is wired (`ecs.tf`), Fargate tasks come up in ~60–120 s.
  **But the caps are the tier**: starter maxes at 2 tasks — saturated long
  before 10k; enterprise allows 200 svc / 100 worker tasks, which covers even
  100k's ~10–30-task need with room. **Crossing tiers is a deliberate manual
  `terraform apply`** (a solo-founder cost guard, not an oversight) — the §10
  trigger (>70 % CPU sustained) is what fires it.
- **Builder-worker**: autoscales on SQS queue depth (target 2 queued builds
  per task, 60 s scale-out). Yes.
- **SQS, S3**: effectively infinite managed scaling. Non-issues.

**B. Scales automatically only if engineered (GPU):**
- Serverless scales **per job** — that's its whole appeal — but inside
  provider concurrency quotas. 10k needs 150–250 peak workers: quota raises +
  the second provider, already §11 actions. **100k needs ~1,300 L4-equivalents
  at the evening peak** (32.5k moments in 4 h ÷ 6.3/hr/L4) or ~330 at uniform
  load — that is no longer "serverless with a quota raise", it is a real GPU
  fleet operation (capacity reservations, own scheduler, likely the
  cheap-render path first). GPU spend at 100k ≈ $150–250k/mo against $1M MRR
  — margins hold, but nothing about it is automatic. 100k is a funded-company
  problem; what matters now is that nothing in today's design *precludes* it.

**C. Does NOT scale automatically — the database is the weakest link:**
- **RDS storage** autoscales 50→500 GB (`max_allocated_storage`). 100k twins
  ≈ 125 GB ASSUMED (500 memories × ~2.5 KB text+vector each) — fits.
- **RDS compute does not**: single-writer instance, manual class change per
  tier, **no read replicas anywhere in the Terraform, no RDS Proxy/pgbouncer**.
  Connection arithmetic: 20 conns/task (pool 5 + overflow 15, config.py) ×
  200 enterprise tasks = 4,000 potential vs ~5,000 `max_connections` on
  r6g.2xlarge — config.py itself warns to keep the product under the cap.
  **Add RDS Proxy (or pgbouncer) + at least one read replica before any
  large task-count tier** — this is the missing piece, cheap (~$11/mo/proxy
  + replica instance cost).
- One structural mercy: every hot query is **twin-scoped** (pgvector search
  runs over ONE twin's few hundred memories, never a global index), so vector
  search stays flat-cost at any subscriber count. The DB problem is
  connections and writer throughput, not ANN.
- **No Alembic migrations** (schema is create-only — known backlog): at 10k+
  every schema change becomes a hand-run risk. Promote it from backlog before
  the growth tier.
- **ElastiCache** is fixed-size per tier (starter: one t4g.small, zero
  replicas) — manual climbs; sessions/quotas at 100k need the enterprise
  r6g.xlarge + replicas.
- **Single NAT gateway** (explicit cost/HA trade-off in network.tf): one AZ's
  egress is a SPOF for HF/NIM calls; add the second NAT at growth tier.

**Verdict:** 10k works on today's Terraform with the enterprise tier, GPU
quota raises, and RDS Proxy added — mostly-automatic, human-gated at the
tier boundaries, which is correct for a solo founder. **100k does not work on
today's stack as-is**: it needs read replicas + connection pooling, a second
NAT, migrations discipline, and a GPU fleet that no serverless marketplace
hands you — but it violates no architectural decision; it's additive. The
honest framing: the stack is built to be *scaled deliberately*, not to scale
by itself — and the trigger table is the deliberation schedule.

---

*Honest bottom line: the cloud-GPU baseline is sounder than phase-A — cost now
tracks usage, the evening peak is bought instead of queued, and onboarding is
uncapped. The arithmetic mostly supports the founder: crossover to a
savings-plan L4 lands at ~130–160 subs on my numbers (~100 only at the
aggressive corner), and "one L4 per ~100 subs" is true ONLY as "L4 + permanent
serverless spill" — a lone VM fails ordinary evenings on TTL. The margin story
survives every commission row except one: an IAP-billed max-usage subscriber
at 30 % is roughly margin-zero. Web checkout is not a packaging detail; it is
the margin.*
