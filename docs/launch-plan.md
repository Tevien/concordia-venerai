# VENER.AI — Coordinated Launch Plan v1 (website + Android + iOS)

**Date:** 2026-08-10 · **Author:** launch-specialist · **Status:** proposed, awaiting founder sign-off
**Launch date recommended:** **Tuesday 2026-10-20** — "Founding Access" launch (inside the Oct 2026 window stated to AWS)

**What "launch" means here:** both apps live in their stores, the site switched from
queue-only to invitation-wave mode, and wave 1 of founding families invited from the
priority-access queue. It is **not** open floodgates — onboarding stays
invitation-gated and capacity-bounded (a wave that melts the queue is a failed
launch). Every mechanic below honors the law: **nothing is free, consent-first,
capacity-bounded, pricing as decided 2026-07-31.**

---

## 1. The critical path (what actually gates 2026-10-20)

```
aws configure → AWS core deploy (HTTPS) ─┐
Apple org enrollment approved ───────────┼→ TestFlight build → founder self-sitting
Remodel branch: founder device pass ─────┘        │
                                                  ▼
                              nan's sitting (product validation gate)
                                                  │
Purchase rails (IAP) built ──────────────┐        ▼
User-facing account deletion built ──────┼→ App Review submission (evidence pack)
Continuity promise + import page live ───┘        │
                                                  ▼
                              approvals held → content freeze → wave-1 invites
                                                  ▼
                                          2026-10-20 LAUNCH
```

Two items on this path are **not yet on the project board as build tasks**:

1. **Purchase rails.** Verified 2026-08-10: zero StoreKit/Play Billing/Stripe/
   RevenueCat code anywhere in `apps/` or `services/`. Quotas are built (62 tests);
   the ability to *charge* is not. Weave $30 + $10/mo subscription must exist as
   IAP products on both stores before submission. Recommend RevenueCat (or
   `expo-iap`) to cover both stores with one integration. ~1–2 weeks incl. sandbox
   testing. **Start immediately.**
2. **User-facing account deletion.** Admin has RESET/ERASE (GDPR), but Apple
   5.1.1(v) and Play policy require *user-initiated, in-app* account deletion
   (Play additionally requires a web deletion URL). Reuse the admin erase path
   behind an in-app confirm flow + a page on vener.ai. ~2–3 days.

---

## 2. Dated timeline (owners: F = founder, C = Claude-side)

### Now → Aug 16 — unblock everything unblockable
- [ ] **F:** `aws configure` on the Mac, then run `prompts/aws-deploy.md`
  (confirm-before-apply). Core stack only — Kokoro fallback voice, **no cloud GPU
  tier** (phase-A decision; $1,000 credits ≈ 3–6 months without GPU, ~6 weeks with).
- [ ] **F:** device pass of the remodel branch (`app-remodel-story-first`, Expo Go
  serves it now) → merge. Everything downstream builds from the remodeled app.
- [ ] **F:** ship the Corsair RAM RMA **now**. The box goes dark during turnaround;
  better in August (zero external users) than October (wave 1 live). This is the
  cheapest schedule insurance in the whole plan.
- [ ] **C:** start purchase-rails build (IAP products: Weave consumable $30,
  subscription auto-renewable $10/mo; one twin per family at pilot — multi-twin
  subscription structure deliberately deferred).
- [ ] **C:** continuity & portability promise — FAQ + terms copy ("your archive
  outlives us": full export, open formats, guaranteed wind-down export window).
  Already up-next on the board; it is a **precondition for HereAfter outreach**,
  so it moves to this week. Claims must be true today: the export script
  (pg_dump + media→S3) ships with the AWS deploy, so the promise goes live only
  once export works.
- [ ] **F:** chase Apple: enrollment submitted 2026-08-09; respond to any D&B/
  verification contact same-day. Calendar reminder: if no movement by **Aug 21**,
  phone Apple Developer Support.

### Aug 17 – Aug 31 — first store contact
- [ ] **F:** full self-sitting on the merged remodel → iterate flow annoyances.
- [ ] **C:** Android closed-test track (Play org VERIFIED 2026-08-10 — org accounts
  are exempt from the 12-tester/14-day production-access rule that binds personal
  accounts; verify in console) + Beam Pro APK via EAS.
- [ ] **C:** user-facing account deletion (app + web URL).
- [ ] **C:** "Bring your recordings" import landing page (displaced HereAfter
  families; functionally supported today via artifact uploads — positioning only)
  + continuity promise live on site. Rights-clean media only.
- [ ] **F+C:** TestFlight build #1 to both phones the day Apple approves
  (EAS builds ready to fire; HTTPS backend from AWS deploy; R2 consent clip
  already in the bundle). **Buffer: if Apple hasn't approved by Sep 4, escalate
  daily and pull the contingency in §8 risk 1.**
- [ ] **C:** app screen-recordings S4/S5 for the site (closes the URGENT website row).

### Sep 1 – Sep 14 — the validation gate
- [ ] **F:** **nan's sitting** — on TestFlight over HTTPS, real conditions, only
  after the self-sitting round is smooth (postponement decision 2026-07-28 stands:
  her first impression is unrepeatable). This is the product validation gate.
- [ ] **F:** with her recorded consent, capture the press asset (her sitting /
  her reveal — the founder story UK press will actually run).
- [ ] **C:** App Review evidence pack assembled (§6). Demo account with a
  pre-woven twin for reviewers — they cannot do a 45-minute sitting.
- [ ] **F+C:** closed-test soak both platforms (both phones + Beam Pro), OTA
  update path (EAS Update) exercised once end-to-end.
- [ ] **C:** SES warm-up begins (OTP-only login makes email deliverability an
  availability dependency — DKIM/SPF/DMARC, low-volume ramp).

### Sep 15 – Sep 30 — review gauntlet + queue warm-up
- [ ] **C:** TestFlight **external** beta submission first (lighter Beta App Review
  — a cheap dry run that surfaces objections before the real submission).
- [ ] **F+C:** submit iOS for App Store review with the evidence pack; submit
  Android to production track (staged rollout). **Two-week buffer assumes one
  rejection cycle each.** Release mode: manual/held on approval.
- [ ] **F:** apply to the App Store Small Business Program + Play 15% tier
  (commission 30%→15%; at pilot scale this is the margin).
- [ ] **F:** UK press outreach opens (§5) — embargo for launch week.
- [ ] **C:** queue warm-up email #1 to the sealed waitlist (offline decrypt via
  `scripts/waitlist-check.sh`): honest status, launch month, what an invitation
  carries (cost contribution — per the site's existing FAQ language).

### Oct 1 – Oct 19 — freeze and load
- [ ] **Oct 5: content freeze.** App copy, store listings (screenshots, privacy
  nutrition labels, Play data safety form), site sections, guide clips. Any new
  clip before freeze passes the **two-gate rule** (founder script review
  pre-render; whisper transcript audit post-render — decision 2026-08-10).
- [ ] **C:** wave-1 invitation emails drafted; invite→pay→sit flow dry-run with
  internal accounts on production infra.
- [ ] **F:** approve held store releases; queue warm-up email #2
  ("invitations begin Oct 20").
- [ ] **F:** box readiness check: RMA complete, render worker uptime monitor on
  /admin, serverless valve credentials configured but dormant (§3).

### Tuesday 2026-10-20 — Founding Access launch
- Site flips to invitation-wave mode (queue stays open; framing: "invitations
  are going out").
- Apps public in both stores; onboarding gated by invitation from the queue.
- **Wave 1: 10 households** (§3). Press embargo lifts.

### Nov 2026 — wave 2
- +15 households → the **25-family founding pilot cap** (deck commitment:
  "25 — deliberately capped"), gated on §3 metrics, cadence ≥ 2 weeks.

---

## 3. Wave plan — capacity arithmetic **(PROVISIONAL — supersede with docs/infra-scaling-review.md when it lands)**

No infra-scaling-review exists as of 2026-08-10, so this is computed conservatively
from the board's measured figures; the review's weave-throughput ceiling replaces
these numbers the day it exists.

**Measured inputs (board/decisions):**
- Warm portrait render: **264 s / 125 frames (~5 s video)** on the 5070 Ti
  (persistent-model server, WSL).
- A daily "portrait moment" (short video, ~10 s) ≈ 2 segments ≈ **~9–10 min GPU**
  incl. overhead.
- Per-new-twin one-time renders (portrait + 6–10 ambient idle clips) ≈
  **~45–60 min GPU**.
- Weave build itself is CPU/HF-API (~$1.50–2.50, no GPU) — the GPU ceiling, not
  the pipeline, is the throughput bound.

**Conservative capacity model (phase-A home box):**
- Committed render window **8 h/day** (box is shared + RMA risk) →
  48 moments/day theoretical.
- Apply **50% headroom** (dark days, retries, onboarding renders, guide-clip
  work) → **~24 moments/day budget**.
- Worst case (every subscriber uses their daily moment): **~24 concurrent active
  twins** inside budget. Expected take-rate is well under cap (pricing decision
  2026-07-31), so the 25-family cap fits — tight, with the valve armed.
- Onboarding budget: **≤ 2–3 new weaves/day** (~2 h of window) during a wave.

**Waves:** W0 = internal (founder, nan, family — pre-launch). **W1 = 10** external
households (Oct 20). **W2 = +15** (Nov, to the 25 cap). Nothing beyond 25 in 2026.

**Gates for the next wave (all measured, all on /admin unless noted):**
- p95 moment queue-wait **< 15 min** (against the 45-min TTL-refund bound), zero
  TTL expiries in the window;
- render-worker uptime ≥ 95% over 14 days;
- sitting completion rate ≥ 80% for the previous wave;
- daily-moment take-rate observed (Redis daily buckets) — if sustained > 70%,
  halve the next wave.

**Pressure valve (decided ladder, 2026-07-31):** serverless per-second GPU
(Modal/RunPod-class, ~$50–75/mo at pilot volume). **Trigger:** p95 wait > 20 min
for 3 consecutive days, OR box dark > 24 h with a non-empty queue. Dedicated L4
only when utilisation fills it — not a 2026 decision.

---

## 4. Packaging, pricing, trial design (around the decided $30 + $10/mo)

Pricing is decided (2026-07-31) and not reopened here: **Weave $30 one-time,
$10/mo including one portrait moment per day.** Packaging around it:

**Founding Families tier (waves 1–2, the only 2026 packaging):**
- Invitation acceptance = paying the Weave **$30** ("an invitation comes with a
  cost contribution" — exactly the site's existing FAQ sentence; the never-free
  law is already public copy).
- The Weave **includes the first 30 days** of the daily-moment subscription as
  included value — paid for by the Weave fee, honestly framed ("your Weave
  includes your first month"), never framed as free. Subscription billing starts
  day 31.
- **14-day money-back on the Weave after the reveal** — money-back framing, never
  free framing. (On iOS, refunds route via Apple; honor directly elsewhere and
  state the mechanics plainly in terms.)
- Founding-member permanence: founding families keep launch pricing for life —
  costs nothing today, rewards the risk-takers, and creates urgency without
  discounting.

**Paywall placement vs the reveal (deliberate):** pay at invitation acceptance →
sitting → honest readiness assessment (`GET /twins/{id}/readiness`, strengths AND
gaps) → weave → **reveal arrives with payment already behind it**. The emotional
peak is delivered, not dangled behind a paywall — and the money-back guarantee,
not a $0 trial, carries the risk. The conversion moment this plan optimizes is
**day-31 subscription continuation**, driven by the daily-moment ritual habit
formed in the included window.

**Gift flow ("commission a portrait"):** the buyer is usually not the subject.
Buyer pays the Weave; the subject receives the sitting invitation; consent is
recorded from the **subject** at the sitting (deliberate button press, R1 spoken
sentence — never auto-recorded, never assumed from purchase). **If the subject
declines consent: full refund to the buyer.** The refund is what keeps the
purchase from becoming pressure on someone else's consent — consent-first
survives contact with gifting.

**Arithmetic (all measured figures):**
| Item | Price | COGS (measured) | Multiple |
|---|---|---|---|
| Weave | $30 ($25.50 net of 15% store cut) | ~$1.50–2.50 build | 10–20× |
| Subscription | $10/mo ($8.50 net) | worst-case 30 moments ≈ $4.50–7.50 at serverless rates; ≈ electricity on phase-A box | thin at 100% take-rate; fine at observed usage |
| Extra moments (post-pilot only) | $1 each in 5-packs ($5) | $0.15–0.25/moment | 4–6.7× — floor rule: never below 3× measured COGS |

**Extra moments are NOT sold in 2026:** phase-A capacity is reserved for the
included daily moments; selling extras before the serverless valve is proven
would be selling capacity we can't guarantee. Revisit at wave 3.

**Annual prepay: deferred.** At 25 families it's ~$2.5k of cash against a promise
of 12 months of daily capacity on a home GPU box — the cash is trivial and the
capacity promise is not. Revisit post-pilot when the serverless valve has run.

**IAP structure note (build detail):** Weave = consumable IAP (repeatable per
twin); subscription = one auto-renewable product (one twin per family at pilot;
multi-twin subscription structure is a deliberate deferral, logged when it
arrives). Restore-purchases + terms/privacy links in-app are review requirements.

---

## 5. Filling the queue (it currently holds ~zero external demand: 2 internal signups)

The queue is the launch's fuel and it is empty. Draining mechanics without
filling mechanics is a plan for launching to nobody. Target: **≥ 150 external
queue signups by Oct 20; ≥ 400 by year end** (a 25-family pilot at a plausible
30–50% warm-queue invite→acceptance needs ~50–80 genuinely warm names; the rest
is 2027 H1 runway — the deck's guided-rollout number is 500).

**Positioning spine (one sentence per audience):**
- **Press/social hook:** *"Living portraits — like the paintings in Harry Potter,
  but of your own family, telling their real stories."* Use the comparison
  descriptively in conversation and pitches; never in paid ads or branding
  (trademark) — journalists will write the Harry Potter line themselves, which
  is worth more anyway. The defensible claim under it (standing rule,
  competitor-watch): "the first consumer-priced, end-to-end living portrait:
  guided capture → grounded twin → their voice and face." Never "first AI legacy
  interviewer."
- **Displaced HereAfter families:** HereAfter shut down; users are racing a
  clock to retrieve recordings. Our angle is **rescue, not ambulance-chasing**:
  the "Bring your recordings" page (imports work today via artifact uploads) +
  the continuity promise ("your archive outlives us" — the exact structural
  answer to the abandonment fear HereAfter just taught the whole market).
  Channels: the forums/subreddits where HereAfter users compare notes,
  genealogy communities, a short honest post — "if you're losing your HereAfter
  archive, here is a home for it, and here is our written promise about what
  happens if *we* ever wind down." Tone gate: `product-critic` reviews every
  outreach text.
- **UK press (the founder story):** solo UK founder building it so his
  grandmother's voice is never lost — and her sitting happens in September.
  With her consent, that's the piece. Targets: BBC tech/human-interest, Guardian
  family, i/inews, founder's local press, Saga Magazine, Gransnet/Mumsnet
  (grandparent communities are the buyer — adult children gift the sitting),
  family-history press (WDYTYA-adjacent), grief-literate podcasts. Embargo for
  launch week; press kit = nan asset + guide-clip demo + S4/S5 recordings +
  economics one-pager. Care-sector (hospices, celebrants) stays **out** —
  roadmap rule: pilots before claims.

**Mechanics:** every queue email is honest status + what an invitation carries
(cost contribution — never "free early access"); the sealed waitlist stays
sealed (export offline via `scripts/waitlist-check.sh`); site keeps zero
trackers, so measure fill by weekly export counts, not analytics.

**If the queue is still thin on Oct 1 (< 100):** launch anyway. The launch is
invitation-scarce by design — a 10-household wave 1 needs ~20–30 warm names, and
scarcity framing ("we onboard a handful at a time, personally") is true. The
press push continues post-launch with the launch itself as the news hook. An
empty queue delays wave 2, not launch day.

## 6. App Review risk — what reviewers will probe on a grief-tech app, and the evidence pack

Expected scrutiny: sensitive personal data (voice, face, life stories),
grief-adjacent purpose, subscriptions. Assemble the pack in the App Review notes
field + a reviewer-facing page.

| Probe | Evidence to have ready |
|---|---|
| Org account for sensitive-data app (5.1.1(ix)) | Apple **organization** enrollment (in review since 2026-08-09) — this is why we did not ship on an individual account |
| Consent for the person being recorded | Demo video of the consent step: deliberate button press + the R1 spoken sentence disclosing voice/audience/photos; `asked_prompt` persisted per artifact (R3); API-enforced "no recorded consent → no twin" |
| Data collection & privacy (5.1.1, 5.1.2) | Privacy nutrition labels matching reality (voice recordings, photos, stories; no ads, no data sale, no trackers — site has zero cookies); privacy.html; GDPR erase path; EU region (eu-west-1) residency |
| Account deletion (5.1.1(v)) | **User-facing in-app deletion (build item, §1)** + web URL (Play requires the URL) |
| Subscription clarity (3.1.2) | Price/term/what-renews on the paywall; "one portrait moment per day" defined in plain words; restore purchases; terms + privacy links in-app and in metadata |
| "Does it pretend to be the dead?" (the sector's scandal risk) | The honesty architecture as review notes: grounded retrieval, refusal to invent, estimates labelled, twin self-identifies as a living memory; consent recorded **by the living subject** — we never build twins of the deceased without their recorded consent |
| Reviewer can't do a 45-min sitting | Demo account with a **pre-woven twin** + short walkthrough video of the full sitting flow |
| UGC / abuse (1.2) | Per-twin ownership chain, fair-use message ceiling, admin erase, abuse-report contact |
| Google Play equivalents | Data safety form (matches labels), sensitive permissions (mic/camera) declarations, deletion URL, org account verified 2026-08-10 |

**Dry-run rule:** TestFlight *external* beta review first (§2, Sep 15–30) — it
catches most metadata/consent objections at a fraction of the cost of a
production rejection.

## 7. Metrics that gate the plan (and where each is measured)

| Metric | Where measured | Used for |
|---|---|---|
| Queue size / weekly growth | `scripts/waitlist-check.sh` offline export count | §5 targets, wave sizing |
| Invite→acceptance (Weave purchases / invites sent) | invitation list vs /admin users + store-console sales | wave conversion, queue quality |
| Sitting completion rate | /admin twin states (gathering→ready) | wave gate (≥ 80%) |
| Weave→first-moment time | /admin + render-queue logs | product promise (≤ ~5 min on prod GPU is the roadmap bar; phase-A: within TTL) |
| D7/D30 daily-moment usage | Redis daily quota buckets surfaced on /admin | day-31 conversion predictor, capacity model |
| Churn vs moment usage | store-console subscription reports × usage cohorts | pricing health, pilot report |
| Capacity headroom (p95 queue-wait, worker uptime) | /admin live log + worker heartbeat | §3 wave gates, serverless trigger |
| Corpus growth per twin ("Ark", instrument from day one) | DB counts: hours/twin, sittings, conversations by twin age | seed-stage evidence (kept out of pitch until real) |

## 8. Top risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | **Apple org enrollment stalls** (D&B verification, solo-founder org checks) | med | launch-critical | Same-day responses; phone support from Aug 21; **contingency:** if not approved by **Sep 22**, iOS launches Oct 20 as invitation-only TestFlight "Founding Access" (external TestFlight, up to 10k testers — capacity needs 25) while Android + web go store-live; App Store public release follows approval. Launch date holds; only the iOS distribution channel degrades. |
| 2 | App Review rejection (grief-tech scrutiny, consent, subscriptions) | med | 1–3 wk slip | Evidence pack (§6); external-TestFlight dry run; two-week resubmission buffer already in the timeline; account deletion + IAP compliance built early, not at submission |
| 3 | **Purchase rails unbuilt** (discovered in this plan: zero payment code) | certain (it's a gap, not a maybe) | launch-critical | Start Aug 10 (§1); RevenueCat/expo-iap; sandbox both stores by Sep 14 so review submission isn't blocked |
| 4 | GPU box downtime (RAM RMA + shared box) during live waves | med | broken promise to paying families | RMA ships **now** (Aug), not October; serverless valve armed with explicit trigger (§3); 45-min TTL auto-refunds already built |
| 5 | Nan's sitting exposes flow problems | med (that's its job) | slip or quality risk | Two-week gap between her sitting and content freeze absorbs one iteration round; her sitting is deliberately *after* founder refinement (decision 2026-07-28) |
| 6 | Queue stays thin | med | small wave 1, weak press moment | §5 fallback: launch is invitation-scarce by design; press continues post-launch; wave 2 delays, launch doesn't |
| 7 | Email deliverability (OTP-only login + invites on fresh SES) | med | users literally can't log in | SES warm-up from Sep 1, DKIM/SPF/DMARC, spam-folder guidance inside every invite (decision 2026-08-02 anticipated this) |
| 8 | AWS credit burn | low | runway | Core stack only ($200–300/mo footprint), GPU tier deliberately dormant; $1,000 ≈ 3–6 months — covers the pilot window |
| 9 | Scope creep from the quality-envelope standing directive | med | schedule | Renderer rungs continue **only** on async content (guide clips, pre-renders — polish is free there); content freeze Oct 5 applies to shipped clips; two-gate rule on anything new |

## 9. Explicitly deferred (logged so deferral is a decision, not an accident)

- Extra-moments purchasing (wave 3+, after serverless valve proves out).
- Annual prepay (post-pilot).
- Multi-twin subscription structure (one twin per family at pilot).
- Guide-voice journal questions (device TTS at launch — flagged founder decision;
  don't market the journal as "in her voice" until the fast-follow ships).
- Care-sector partnerships and claims (pilots before claims).
- Cloud GPU tier / dedicated L4 (utilisation-gated, per the phase-A decision).
- Android beyond closed-test polish (Beam Pro is a test device, not a marketing
  segment; Play production ships launch day but iOS + web carry the story).

---
*Board and decision-log updates when this plan is accepted: launch date + wave
plan onto `project_board.md`; the Founding-Families packaging (included 30-day
window, money-back, gift-refund rule) into `decisions.md`. Wave arithmetic in §3
is PROVISIONAL and yields to `docs/infra-scaling-review.md` on arrival.*
