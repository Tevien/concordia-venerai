# VENER.AI PM - roadmap

The product vision and where it is going. Keep it honest: what is committed, what is
a bet, what is parked.

## Vision

VENER.AI builds **living portraits** of the people you love — their voice, stories,
humour and way of seeing the world — so families can keep talking with them.
An avatar-guided sitting that an elderly person can complete alone becomes a
grounded, generative twin: cloned voice, gently moving portrait, answers grounded
in what they actually recorded, refusal to invent. Consent-first, priced for
families (Weave $30 + $10/mo incl. a portrait moment a day), never advertising or data sales. "Good" looks
like: a grandmother completes a sitting unassisted, her family talks with her
portrait for years, and no one is ever misled by it.

**USP:** affordable realism (end-to-end capture→twin at consumer price — the gap
between $8/mo clip-playback apps and $50k studio avatars).
**Moat (later, with data):** the "Ark" — each family's compounding, irreplaceable
corpus. Deliberately out of pitch materials until pilot data exists.

## Now / next / later

**Now** (in active build)
- [ ] Sitting refinement with Sean as test subject (device retest → self-sitting → iterate)
- [ ] Guide avatar clips: voice/face/quality verdicts, then the resumable batch render
- [ ] AWS Activate re-application (site made eligible 2026-07-28)
- [ ] Apple Developer enrollment (user; DUNS in hand)

**Next** (committed, not started)
- [ ] TestFlight internal: Sean's phone + nan's phone (auto-updating; EAS Update OTA wired at first build)
- [ ] AWS deploy on credits (Terraform ready) + local→cloud data export script
- [ ] Nan's real sitting (only after refinement passes)
- [ ] Warmup render at talking-head deploy; expo-audio runtime re-test
- [ ] Hardening: Alembic, delete-order media leak, resumable uploads, retrieval eval set

**Later** (bets, not yet committed)
- [ ] Ambient living portraits: always-alive idle reel per twin + approach transitions (app) and the guide sprite roaming the website
- [ ] Founding-families pilot: 25 families, 2026 H2, cost-covering contribution
- [ ] Persona/face LoRA training on real CUDA time; persona-LoRA serving (vLLM)
- [ ] Conversational interviewer (Option C: dynamic follow-ups mapped back to question-bank coverage)
- [ ] Android + Play Console
- [ ] Family sharing with per-person permissions; memorial-archive mode (family-consented, reduced capability — consent research first)
- [ ] Care-sector partnerships (hospices, celebrants) — pilots before claims

## Milestones

| Milestone | Goal | Target | Status |
|-----------|------|--------|--------|
| M1 | Sitting refined on the founder (hands-free flow smooth end-to-end) | Aug 2026 | in progress |
| M2 | TestFlight build on both family phones, auto-updating | Apple + backend gated | ready to fire |
| M3 | AWS production deploy + data migration path | credits gated | terraform ready |
| M4 | Nan's real sitting → her living portrait | after M1 | postponed by design |
| M5 | Founding pilot: 25 families onboarded | 2026 H2 | planned |

## Out of scope (for now)

- Twins of the deceased without their recorded consent (memorial-archive mode is a
  research question, not a product promise).
- Data network effects / cross-family learning claims — the Ark is per-family.
- 90%-SaaS-margin storytelling — media inference has real COGS; we say so.
- GCP deploy (Terraform exists, deliberately parked; AWS first).

## Success metrics

- A completed unassisted sitting by an elderly tester (M1/M4 proof).
- Time-to-first-reply (voice) ≤ ~2 s warm; portrait moment ≤ ~5 min on prod GPU.
- Pilot: 25 families onboarded, zero trust incidents, churn ≈ 0 in first 6 months.
- Corpus growth per twin over time (the Ark metric — instrument from day one).
