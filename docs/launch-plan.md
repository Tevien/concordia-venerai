# VENER.AI — Coordinated Launch Plan v1.1 (website + Android + iOS)

**Date:** 2026-08-10 · **Author:** launch-specialist · **Status:** proposed, awaiting founder sign-off
**Revised same day for "Launch economics v2" (decisions.md 2026-08-10): subscription-only pricing, cloud-GPU launch.**
**Launch date recommended:** **Tuesday 2026-10-20** — "Founding Access" launch (inside the Oct 2026 window stated to AWS)

**What "launch" means here:** both apps live in their stores, the site switched from
queue-only to invitation-wave mode, and wave 1 of founding families invited from the
priority-access queue. It is **not** open floodgates — onboarding stays
invitation-gated and capacity-bounded. Every mechanic below honors the law:
**nothing is free (the first charge lands at signup, before the sitting — no $0
window), consent-first, capacity-bounded, pricing as decided 2026-08-10
($10/mo subscription only; the $30 Weave fee is gone).**

---

## 1. The critical path (what actually gates 2026-10-20)

```
aws configure → AWS core deploy (HTTPS) ─────────┐
Serverless render path live (Modal/RunPod-class) ┼→ TestFlight build → founder self-sitting
Apple org enrollment approved ───────────────────┤        │
Remodel branch: founder device pass ─────────────┘        ▼
                                    nan's sitting (product validation gate,
                                        on the production render path)
                                                          │
Stripe checkout + entitlement + iOS IAP built ───┐        ▼
User-facing account deletion built ──────────────┼→ App Review submission (evidence pack)
Continuity promise (made structural) + import ───┘        │
                                                          ▼
                                      approvals held → content freeze → wave-1 invites
                                                          ▼
                                                  2026-10-20 LAUNCH
```

Launch-blocking gaps **not yet on the project board as build tasks**:

1. **Purchase rails.** Verified 2026-08-10: zero payment code anywhere in `apps/`
   or `services/`. Under economics v2 the build is **web-checkout-primary**
   (position argued in §4a): Stripe Billing + hosted Checkout on the invite page
   (charge at signup, invitation-token-gated), webhook → backend **entitlement
   API** the apps read, Stripe customer portal for cancel/refund; **iOS adds the
   $10/mo auto-renewable IAP** (StoreKit 2 via RevenueCat) as the
   compliance-and-capture path; **Play Billing is deferred** — Android launches
   consumption-only (sign-in to an existing subscription, no in-app purchase, no
   in-app steering to web). ~1–2 weeks incl. sandbox/webhook testing. **Start
   immediately.**
2. **Serverless render path.** Launch compute is cloud from subscriber #1
   (economics v2); the SQS pull-worker is demoted to emergency valve. The
   talking-head container (persistent-model server) must run on Modal/RunPod-class
   per-second GPU with a **warm pool** (the ~15 GB model load must never be paid
   per job — infra-scaling-review §4). Same job contract as the queue today.
   Must be live before nan's sitting (real conditions) and hard-gates wave 1.
3. **User-facing account deletion.** Admin has RESET/ERASE, but Apple 5.1.1(v)
   and Play policy require *user-initiated, in-app* deletion (+ web URL for
   Play). Fold in the infra-review's GDPR flag: erasure on versioned S3 must
   delete **versions**, or it isn't erasure. ~3–4 days.
4. **Continuity promise made structural** (precondition for HereAfter outreach,
   §5): export tooling + the infra-review's durability pack (S3 lifecycle,
   cross-account Glacier Deep Archive replication, ~$1/TB-mo). The promise page
   goes live only when the architecture behind it exists.

---

## 2. Dated timeline (owners: F = founder, C = Claude-side)

### Now → Aug 16 — unblock everything unblockable
- [ ] **F:** `aws configure` on the Mac, then run `prompts/aws-deploy.md`
  (confirm-before-apply). Core stack (Kokoro fallback voice) **plus** the
  serverless GPU integration work starts now — cloud renders from subscriber #1
  supersede the phase-A no-cloud-GPU rule. Credits cover the core stack
  ($1,000 ≈ 3–6 months at $250–350/mo); serverless GPU bills separately and
  tracks usage (~$0.50–1.50/subscriber-month below 100 subs — decision entry).
- [ ] **F:** device pass of the remodel branch (`app-remodel-story-first`, Expo Go
  serves it now) → merge. Everything downstream builds from the remodeled app.
- [ ] **C:** start purchase rails (§1 item 1): Stripe products + invite-gated
  checkout page + entitlement API + customer portal. iOS IAP product created the
  day Apple enrollment lands.
- [ ] **C:** start the serverless render worker (§1 item 2): package the existing
  talking-head container for Modal/RunPod, warm-pool config, queue-age alarm
  wiring. Include the infra-review's two cheap fixes while in the code: worker
  TTL check (~5 lines) and the content-addressed artifact cache.
- [ ] **F:** ship the Corsair RAM RMA. Downgraded from schedule-critical to
  dev/test hygiene (local GPUs are dev/test + emergency valve only now), but
  August downtime is still cheaper than October downtime for guide-clip work.
- [ ] **F:** chase Apple: enrollment submitted 2026-08-09; respond to any D&B/
  verification contact same-day. Calendar reminder: if no movement by **Aug 21**,
  phone Apple Developer Support.

### Aug 17 – Aug 31 — first store contact + the promise made structural
- [ ] **F:** full self-sitting on the merged remodel → iterate flow annoyances.
- [ ] **C:** Android closed-test track (Play org VERIFIED 2026-08-10 — org
  accounts are exempt from the 12-tester/14-day production-access rule that
  binds personal accounts; verify in console) + Beam Pro APK via EAS.
- [ ] **C:** user-facing account deletion (app + web URL), **version-aware**
  erasure per infra-review §7.
- [ ] **C:** durability pack (infra-review action 4): S3 lifecycle, cross-account
  Glacier Deep Archive replication, per-twin open-format export. Then the
  **continuity & portability promise** goes live on FAQ + terms ("your archive
  outlives us": full export, open formats, guaranteed wind-down window) — now a
  claim that is true today, not aspirational.
- [ ] **C:** move the marketing site to Cloudflare Pages (infra-review action 5 —
  the Pi is a SPOF exactly when an invitation wave lands) and build the invite →
  Stripe Checkout page on it.
- [ ] **C:** "Bring your recordings" import landing page (displaced HereAfter
  families; artifact uploads support it today — positioning only). Rights-clean
  media only.
- [ ] **C:** /admin gets the three queue metrics (depth, oldest-message age,
  TTL-failures today) — the §3 wave gates are unwatchable without them
  (infra-review action 2).
- [ ] **F+C:** serverless render path E2E test: enqueue → Modal/RunPod render →
  S3 object → app playback, cold and warm.
- [ ] **F+C:** TestFlight build #1 to both phones the day Apple approves
  (EAS builds ready to fire; HTTPS backend from AWS deploy; R2 consent clip
  already in the bundle). **Buffer: if Apple hasn't approved by Sep 4, escalate
  daily and pull the contingency in §8 risk 1.**
- [ ] **C:** app screen-recordings S4/S5 for the site (closes the URGENT website row).

### Sep 1 – Sep 14 — the validation gate
- [ ] **F:** **nan's sitting** — on TestFlight over HTTPS, renders on the
  production serverless path, only after the self-sitting round is smooth
  (postponement decision 2026-07-28 stands: her first impression is
  unrepeatable). This is the product validation gate.
- [ ] **F:** with her recorded consent, capture the press asset (her sitting /
  her reveal — the founder story UK press will actually run).
- [ ] **C:** App Review evidence pack assembled (§6). Demo account with a
  pre-woven twin for reviewers — they cannot do a 45-minute sitting.
- [ ] **F+C:** closed-test soak both platforms (both phones + Beam Pro), OTA
  update path (EAS Update) exercised once end-to-end. Stripe + IAP sandbox
  flows exercised: signup-charge, month-1 refund, gift-decline refund, cancel.
- [ ] **C:** SES warm-up begins (OTP-only login makes email deliverability an
  availability dependency — DKIM/SPF/DMARC, low-volume ramp; SMTP2GO stays as
  proven fallback).

### Sep 15 – Sep 30 — review gauntlet + queue warm-up
- [ ] **C:** TestFlight **external** beta submission first (lighter Beta App Review
  — a cheap dry run that surfaces objections before the real submission).
- [ ] **F+C:** submit iOS for App Store review with the evidence pack; submit
  Android to production track (staged rollout). **Two-week buffer assumes one
  rejection cycle each.** Release mode: manual/held on approval.
- [ ] **F:** apply to the App Store Small Business Program + Play 15% tier —
  commission 30%→15% on whatever volume the IAP path ever carries.
- [ ] **F:** UK press outreach opens (§5) — embargo for launch week.
- [ ] **C:** queue warm-up email #1 to the sealed waitlist (offline decrypt via
  `scripts/waitlist-check.sh`): honest status, launch month, what an invitation
  carries ($10/mo from signup — the cost contribution the site's FAQ already
  promises; never "free early access").

### Oct 1 – Oct 19 — freeze and load
- [ ] **Oct 5: content freeze.** App copy, store listings (screenshots, privacy
  nutrition labels, Play data safety form), site sections, guide clips. Any new
  clip before freeze passes the **two-gate rule** (founder script review
  pre-render; whisper transcript audit post-render — decision 2026-08-10).
- [ ] **C:** wave-1 invitation emails drafted; **invite → web checkout → sitting
  dry-run** with internal accounts on production infra, including a forced
  serverless cold-start.
- [ ] **F:** approve held store releases; queue warm-up email #2
  ("invitations begin Oct 20").
- [ ] **F:** go/no-go checklist: serverless warm pool configured for the evening
  window, /admin queue metrics green, Stripe webhooks verified in prod, refund
  paths tested, pull-worker valve documented as the emergency runbook.

### Tuesday 2026-10-20 — Founding Access launch
- Site flips to invitation-wave mode (queue stays open; framing: "invitations
  are going out").
- Apps public in both stores; onboarding gated by invitation from the queue;
  signup (and the first $10 charge) happens on the web invite page before the
  sitting.
- **Wave 1: 10 households** (§3). Press embargo lifts.

### Nov 2026 — wave 2
- +15 households → the **25-family founding pilot cap** (deck commitment:
  "25 — deliberately capped"), gated on §3 metrics, cadence ≥ 2 weeks.

---

## 3. Wave plan — regated for cloud compute (numbers: infra-scaling-review.md 2026-08-10 + Launch economics v2)

Economics v2 lifts the home-box ceiling that originally bounded waves (the
infra-review's ~110–130-twin box ceiling and ≤50-invites/week weave cap were
computed for phase-A local compute, now dev/test only). On serverless GPU,
compute scales with spend: **the binding constraint is no longer the GPU — it is
one founder** handling onboarding, support, and consent questions, plus the
serverless spend ramp staying inside its modeled band.

**Cost figures this plan stands on (infra-review §4, decision entry):**
- Portrait moment: **$0.15–0.25** with warm-pool amortisation (L4-class,
  450–530 s active; the ~15 GB model load is paid by the warm pool, never per job).
- Weave GPU: **$0.20–0.50**; all-in weave ≈ **$2–3** — absorbed by the first month.
- Serverless spend below 100 subs: **~$0.50–1.50 per subscriber-month** at
  observed-average usage; **$4.50–7.50** at max usage (30 moments) — thin, and
  the 1/day quota is the cap that keeps it bounded.
- Dedicated L4 adoption point: **~100 active subscriptions**, or serverless spend
  > $450/mo sustained 2 months (≈ 2,000–2,500 moments/mo).

**Human-capacity model (solo founder):** founding onboarding is personal (the
site promises "onboarded personally"): invite call/setup + consent questions +
first-week support ≈ **2–4 h per household**. A 10-household wave is roughly one
founder-week of people-work on top of ops. That, not GPU, sizes the waves.

**Waves:** W0 = internal (founder, nan, family — pre-launch). **W1 = 10** external
households (Oct 20). **W2 = +15** (Nov, to the 25 cap). Nothing beyond 25 in
2026 — the cap survives economics v2 for support and quality reasons, not compute.

**Gates for the next wave (all measured; /admin unless noted):**
- **Serverless spend ≤ $1.50/subscriber-month** (provider billing dashboard ÷
  active subs) — the new capacity metric. Sustained > $2 = power-user cohort;
  investigate before inviting more.
- p95 moment latency (enqueue → object) **≤ 5 min**, serverless **cold-start p95
  < 90 s** (infra-review trigger), **zero TTL-failures** in the window.
- **Founder load green:** all support/onboarding/consent replies < 24 h sustained
  through the wave, onboarding backlog cleared before the next wave opens.
- Sitting completion rate ≥ 80% for the previous wave.
- Month-1 refund rate < 20% (money-back requests are a product signal, not
  just a cost).

**Emergency valve (inverted from v1):** the home-box SQS pull-worker is the
fallback if the serverless provider fails — documented runbook, not a launch
dependency. **L4 VM climb** happens at the trigger above, not before.

---

## 4. Packaging, pricing, trial design (around the decided $10/mo, charged at signup)

Pricing is decided (2026-08-10, "Launch economics v2") and not reopened here:
**$10/mo subscription only, first charge at signup — before the sitting. No
up-front fee.** Never-free is preserved structurally: there is no $0 window
anywhere in the funnel. Packaging around it:

**Founding Families tier (waves 1–2, the only 2026 packaging):**
- Invitation acceptance = **signup + the first $10 charge** ("an invitation
  comes with a cost contribution" — the site's existing FAQ sentence still
  holds; the first month *is* the contribution).
- **"Your first month includes the weave."** The ~$2–3 serverless build cost is
  absorbed by month 1 (decision arithmetic) — framed as included value, never
  as free.
- **Money-back: full refund of the first $10 month**, claimable within 14 days
  of the reveal, if the family isn't moved by it. Money-back framing, never
  free framing. (Stripe refunds are self-serve for us; on the rare IAP path,
  refunds route via Apple — say so plainly in terms.)
- Founding-member permanence: founding families keep $10/mo for life — costs
  nothing today, rewards the risk-takers, creates urgency without discounting.

**Paywall placement vs the reveal (deliberate):** charge at signup → sitting →
honest readiness assessment (`GET /twins/{id}/readiness`, strengths AND gaps) →
weave → **reveal arrives with payment already behind it**. The emotional peak is
delivered, not dangled — and the money-back guarantee, not a $0 trial, carries
the risk. The conversion moment this plan optimizes is the **first renewal
(day ~30)**, driven by the daily-moment ritual habit formed in month one.
Worst-case churn exposure per month-one canceller: **~$2–3** (decision entry) —
an acceptable acquisition cost, and the D7/D30 usage metric predicts it.

**Gift flow ("commission a portrait"):** the buyer is usually not the subject.
Buyer signs up and pays the first month; the subject receives the sitting
invitation; consent is recorded from the **subject** at the sitting (deliberate
button press, R1 spoken sentence — never auto-recorded, never assumed from
purchase). **If the subject declines consent: the $10 is refunded and the
subscription cancelled.** The refund keeps the purchase from becoming pressure
on someone else's consent — consent-first survives contact with gifting.

### 4a. Commission position (first-order under subscription-only revenue): **web-checkout-primary hybrid**

With ALL revenue now subscription, the store cut is the difference between a
working model and a marginal one (decision arithmetic: 100 subs gross $1,000 vs
L4 ≈ $580–660 on-demand — marginal after 15–30% IAP commission; fine at ~3–6%
web checkout).

**Recommendation: web-checkout-primary, with iOS IAP offered and Android
consumption-only.**

- **Primary path (all invitations):** invite email → vener.ai invite page →
  Stripe Checkout → signup charge → app sign-in by email OTP reads the
  entitlement. At launch, *every* customer arrives this way (the queue is the
  only acquisition channel), so ~all revenue lands at Stripe's ~4–6% on a $10
  charge (net ≈ $9.40–9.60) instead of 15–30% (net $7.00–8.50).
- **iOS:** the $10/mo auto-renewable IAP is **offered in-app** as well. This is
  the compliance anchor: VENER.AI is not plausibly a 3.1.3(a) "reader app"
  (that carve-out is for magazines/books/audio/video libraries), so the safe
  posture is 3.1.3(b) **multiplatform services** — users may access
  subscriptions acquired on the web *provided the item is also available as
  IAP*. Offering IAP removes the 3.1.1 fight entirely; the app contains **no
  links, mentions, or steering to web pricing** (steering happens in owned
  channels — email and the website — which Apple does not police). At pilot
  volume the IAP path carries ≈ 0 customers, so its thinner margin is noise;
  it exists for compliance and store-first arrivals.
- **Android:** launch **consumption-only** — sign-in to an existing
  subscription, no in-app purchase of any kind, no in-app steering. Play policy
  permits consuming externally-purchased subscriptions when the app neither
  sells digital goods in-app nor directs users to an outside purchase flow
  (the Netflix posture). Store-first browsers see "sign in / invitation
  required". Play Billing is added later only if store-first Android
  acquisition materializes.
- **Deferred optimizations (do NOT spend review-risk on them at launch):** US
  external-purchase-link entitlement and EU DMA link-outs can recover margin on
  store-first users post-launch; a grief-tech app under first review should not
  pioneer steering mechanics.

**App Review risk of this position, assessed:** iOS with IAP offered is the
*lowest*-risk configuration available (nothing for 3.1.1 to object to);
the residual risk is a reviewer probing how accounts are created — answered in
review notes: "invitation-only founding cohort; subscriptions available in-app
via IAP; web signup exists but is never referenced in-app." Android
consumption-only is established practice; the fallback if Play pushes back is
enabling Play Billing (User Choice Billing's ~4% discount is not worth its
complexity at pilot scale).

**Arithmetic (measured/recorded figures):**
| Path | Net of channel on $10 | Month-1 COGS (weave $2–3 + avg moments $2.25–3.75) | Month-1 margin |
|---|---|---|---|
| Web (Stripe ~4–6%) | $9.40–9.60 | $4.25–6.75 | **+$2.65–5.35** |
| IAP @15% (Small Business) | $8.50 | $4.25–6.75 | +$1.75–4.25 |
| IAP @30% | $7.00 | $4.25–6.75 | +$0.25–2.75 (max-usage month-1 goes negative — why IAP stays the exception) |

**Extra moments are NOT sold in 2026:** when they arrive (wave 3+), floor
$0.25/moment serverless cost → price $1 each in 5-packs ($5), never below 3×
measured COGS.

**Annual prepay: deferred.** ~$2.5k cash at pilot scale against a 12-month
service promise — revisit post-pilot when the serverless path has months of
history.

## 5. Filling the queue (it currently holds ~zero external demand: 2 internal signups)

The queue is the launch's fuel and it is empty. Draining mechanics without
filling mechanics is a plan for launching to nobody. Target: **≥ 150 external
queue signups by Oct 20; ≥ 400 by year end** (a 25-family pilot at a plausible
30–50% warm-queue invite→acceptance needs ~50–80 genuinely warm names; the rest
is 2027 H1 runway — the deck's guided-rollout number is 500). The $10-at-signup
entry (no $30 gate) should measurably lift invite→acceptance — watch it as the
first pricing-v2 datapoint.

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
  the continuity promise ("your archive outlives us") — which by §2 is
  **structural before outreach begins** (export tooling + cross-account backup,
  not just words). Channels: the forums/subreddits where HereAfter users
  compare notes, genealogy communities, a short honest post. Tone gate:
  `product-critic` reviews every outreach text.
- **UK press (the founder story):** solo UK founder building it so his
  grandmother's voice is never lost — and her sitting happens in September.
  With her consent, that's the piece. Targets: BBC tech/human-interest, Guardian
  family, i/inews, founder's local press, Saga Magazine, Gransnet/Mumsnet
  (grandparent communities are the buyer — adult children gift the sitting at
  $10/mo, a much easier gift than $30+$10 was), family-history press
  (WDYTYA-adjacent), grief-literate podcasts. Embargo for launch week; press
  kit = nan asset + guide-clip demo + S4/S5 recordings + economics one-pager.
  Care-sector (hospices, celebrants) stays **out** — roadmap rule: pilots
  before claims.

**Mechanics:** every queue email is honest status + what an invitation carries
($10/mo from signup — never "free early access"); the sealed waitlist stays
sealed (export offline via `scripts/waitlist-check.sh`); site keeps zero
trackers, so measure fill by weekly export counts, not analytics. The site
itself moves to Cloudflare Pages before wave 1 (§2) so a press spike or an
invitation wave never lands on a dead Pi.

**If the queue is still thin on Oct 1 (< 100):** launch anyway. The launch is
invitation-scarce by design — a 10-household wave 1 needs ~20–30 warm names, and
scarcity framing ("we onboard a handful at a time, personally") is true. The
press push continues post-launch with the launch itself as the news hook. An
empty queue delays wave 2, not launch day.

## 6. App Review risk — what reviewers will probe on a grief-tech app, and the evidence pack

Expected scrutiny: sensitive personal data (voice, face, life stories),
grief-adjacent purpose, subscriptions, and now the web/IAP split. Assemble the
pack in the App Review notes field + a reviewer-facing page.

| Probe | Evidence to have ready |
|---|---|
| Org account for sensitive-data app (5.1.1(ix)) | Apple **organization** enrollment (in review since 2026-08-09) — this is why we did not ship on an individual account |
| Consent for the person being recorded | Demo video of the consent step: deliberate button press + the R1 spoken sentence disclosing voice/audience/photos; `asked_prompt` persisted per artifact (R3); API-enforced "no recorded consent → no twin" |
| Data collection & privacy (5.1.1, 5.1.2) | Privacy nutrition labels matching reality (voice recordings, photos, stories; no ads, no data sale, no trackers — site has zero cookies); privacy.html; version-aware GDPR erase path; EU region (eu-west-1) residency + cross-account backup |
| Account deletion (5.1.1(v)) | **User-facing in-app deletion (build item, §1)** + web URL (Play requires the URL); deletion removes S3 *versions*, not just current objects |
| Purchases & the web/IAP split (3.1.1 / 3.1.3(b)) | The $10/mo subscription **is offered as IAP in-app**; no in-app link, mention, or price reference to web signup; review note: "invitation-only founding cohort; subscription also purchasable in-app" |
| Subscription clarity (3.1.2) | Price/term/what-renews on the IAP paywall; "one portrait moment per day" defined in plain words; restore purchases; terms + privacy links in-app and in metadata; money-back policy stated without the word "free" |
| "Does it pretend to be the dead?" (the sector's scandal risk) | The honesty architecture as review notes: grounded retrieval, refusal to invent, estimates labelled, twin self-identifies as a living memory; consent recorded **by the living subject** — we never build twins of the deceased without their recorded consent |
| Reviewer can't do a 45-min sitting | Demo account with a **pre-woven twin** + short walkthrough video of the full sitting flow |
| UGC / abuse (1.2) | Per-twin ownership chain, fair-use message ceiling, admin erase, abuse-report contact |
| Google Play equivalents | Data safety form (matches labels), sensitive permissions (mic/camera) declarations, deletion URL, org account verified 2026-08-10; consumption-only posture: no digital-goods purchase in-app, no external-purchase steering in-app |

**Dry-run rule:** TestFlight *external* beta review first (§2, Sep 15–30) — it
catches most metadata/consent/purchase objections at a fraction of the cost of a
production rejection.

## 7. Metrics that gate the plan (and where each is measured)

| Metric | Where measured | Used for |
|---|---|---|
| Queue size / weekly growth | `scripts/waitlist-check.sh` offline export count | §5 targets, wave sizing |
| Invite→acceptance (signups / invites sent) | invitation list vs Stripe dashboard + /admin users | wave conversion, first pricing-v2 datapoint |
| **Serverless spend per subscriber-month** | provider billing dashboard ÷ active subs (mirror the number on /admin) | **the wave-gating capacity metric** (≤ $1.50; investigate > $2) |
| p95 moment latency + cold-start p95 | /admin queue metrics (depth, oldest-age, TTL-failures — §2 build item) + provider logs | wave gates (≤ 5 min; cold-start < 90 s); L4-climb trigger |
| Founder load (reply latency, onboarding backlog) | support inbox + a one-line /admin note per wave | the human capacity gate (§3) |
| Sitting completion rate | /admin twin states (gathering→ready) | wave gate (≥ 80%) |
| Weave→first-moment time | /admin + render-queue logs | product promise (≤ ~5 min on the serverless path) |
| D7/D30 daily-moment usage | Redis daily quota buckets surfaced on /admin | first-renewal predictor, spend model |
| Month-1 refund rate + churn vs usage | Stripe dashboard (+ App Store Connect for the IAP path) × usage cohorts | money-back health (< 20%), pricing-v2 validation |
| Corpus growth per twin ("Ark", instrument from day one) | DB counts: hours/twin, sittings, conversations by twin age | seed-stage evidence (kept out of pitch until real) |

## 8. Top risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | **Apple org enrollment stalls** (D&B verification, solo-founder org checks) | med | launch-critical | Same-day responses; phone support from Aug 21; **contingency:** if not approved by **Sep 22**, iOS launches Oct 20 as invitation-only TestFlight "Founding Access" (external TestFlight, up to 10k testers — capacity needs 25) while Android + web go store-live; App Store public release follows approval. Launch date holds; only the iOS distribution channel degrades. |
| 2 | App Review rejection (grief-tech scrutiny, consent, purchases) | med | 1–3 wk slip | Evidence pack (§6); the §4a position is the lowest-risk purchase configuration (IAP offered, zero in-app steering); external-TestFlight dry run; two-week resubmission buffer; deletion + IAP compliance built early |
| 3 | **Purchase rails unbuilt** (zero payment code as of 2026-08-10) | certain (it's a gap, not a maybe) | launch-critical | Start Aug 10 (§1): Stripe primary + entitlement API + iOS IAP via RevenueCat; Play Billing deliberately out of scope; sandbox both paths by Sep 14 |
| 4 | **Serverless render path fails to hit the bar** (cold starts on a ~15 GB model, provider variance) | med | product promise ("minutes later") broken for paying families | Warm pool through the 18:00–22:00 window (infra-review: evening clustering ~65%); persistent-model container already built; E2E test Aug, forced-cold-start test in the Oct dry-run; home pull-worker documented as emergency valve; cold-start p95 > 90 s is a stop-the-wave trigger |
| 5 | Nan's sitting exposes flow problems | med (that's its job) | slip or quality risk | Two-week gap between her sitting and content freeze absorbs one iteration round; her sitting deliberately *after* founder refinement (2026-07-28) and *on* the production render path |
| 6 | **Power-user cohort margin** (max usage = $4.50–7.50 GPU vs $10 sub) | med | thin-to-negative unit margin | The 1/day quota is the structural cap; watch spend/sub-month and the usage distribution (infra-review flags this as the pilot's key measurement); extra-moments pricing (wave 3+) monetizes the heavy tail instead of subsidizing it |
| 7 | Queue stays thin | med | small wave 1, weak press moment | §5 fallback: launch is invitation-scarce by design; press continues post-launch; wave 2 delays, launch doesn't |
| 8 | Email deliverability (OTP-only login + invites on fresh SES) | med | users literally can't log in | SES warm-up from Sep 1, DKIM/SPF/DMARC, spam-folder guidance inside every invite; SMTP2GO fallback already proven on /admin |
| 9 | **Cash: subscription-only removes the up-front cushion** | low at pilot | runway | Credits absorb the $250–350/mo core stack for 3–6 months; break-even ≈ 30–35 subs (infra-review §5) — beyond the 25-family 2026 cap, so 2026 runs modestly cash-negative **by design**; wave 3 (2027) crosses it; serverless GPU is self-funding at ~$0.50–1.50/sub-month vs $9.40 net |
| 10 | Scope creep from the quality-envelope standing directive | med | schedule | Renderer rungs continue **only** on async content (guide clips, pre-renders — polish is free there); content freeze Oct 5 applies to shipped clips; two-gate rule on anything new |

## 9. Explicitly deferred (logged so deferral is a decision, not an accident)

- Play Billing (Android launches consumption-only; add only if store-first
  Android acquisition materializes).
- US external-purchase-link entitlement / EU DMA link-outs (margin optimization
  on store-first users — post-launch, not during first review).
- Extra-moments purchasing (wave 3+; floor $0.25 serverless cost, never below
  3× COGS).
- Annual prepay (post-pilot, once the serverless path has months of history).
- Multi-twin subscription structure (one twin per family at pilot).
- Dedicated L4 VM (~100 active subs, or serverless spend > $450/mo sustained —
  triggers in §3, not calendar-driven).
- Guide-voice journal questions (device TTS at launch — flagged founder
  decision; don't market the journal as "in her voice" until the fast-follow).
- Care-sector partnerships and claims (pilots before claims).

---
*Board and decision-log updates when this plan is accepted: launch date + wave
plan onto `project_board.md`; the Founding-Families packaging (month-1-includes-
the-weave, money-back on month 1, gift-decline refund, web-checkout-primary
position) into `decisions.md`. Compute and cost figures defer to
`docs/infra-scaling-review.md` (2026-08-10), whose home-box ceiling sections are
themselves superseded by Launch economics v2 — local GPUs are dev/test and
emergency valve only. The pitch deck still carries the $30 Weave and 40–50%
pilot-margin framing; economics v2 requires a deck refresh pass (flagged in the
decision entry).*
