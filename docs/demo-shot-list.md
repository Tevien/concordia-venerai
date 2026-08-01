# VENER.AI — demo content shot-list

Production plan for the website demos (board: URGENT visual upgrade) and the
guide-avatar assets. Everything renders on the test setup — zero AWS spend.
Decisions locked: founder is the demo subject (2026-08-01); guide face =
generated synthetic person (candidates in `~/venerai/guide-faces/` on the box);
guide voice = synthetic female clone (rights-clean chain: Kokoro → Chatterbox).

**Honesty rule on every demo:** captioned as real pipeline output, never a
mock-up. Standard caption: *"Real output from our pipeline. The subject is our
founder — every real twin requires the person's recorded consent."*

## Inputs still needed

- [ ] Founder portrait photo: front-facing, soft even light, shoulders-up,
      plain warm background, no glasses glare — phone photo is fine (the render
      works at 512², detail is not the constraint). A 10s neutral-expression
      video also works; we extract the frame.
- [ ] Guide face pick from the contact sheet (candidate number).
- [ ] Founder voice reference: existing `voice_ref.wav` on the box (already
      proven) — nothing new needed unless you want a fresher take.

## Shots

### S1 — Hero: the living portrait (site hero, muted autoplay loop)
- **What:** founder's portrait speaking, ~12 s, gold-frame crop, loops cleanly
  (start/end on closed-mouth rest).
- **Line (founder's cloned voice):** "Some stories deserve more than a photo
  album. Ask me about the summer I turned ten — I remember all of it."
- **Render:** 16 steps, 512², audio-guidance 4.5. Poster = first frame.

### S2 — The guide asks (app asset + site "guided sitting" demo)
- **What:** guide avatar asking the real `intro` question, ~14 s.
- **Line (guide voice, verbatim from the bank):** "Introduce yourself as if to
  a great-grandchild who will never meet you. Your name, where you're from,
  and what you'd want them to know first."
- **Render:** same settings; this clip doubles as the first bundled app clip.

### S3 — Ask anything: a grounded answer (site "why" section)
- **What:** founder portrait answering a question with a REAL answer from his
  built twin (pull the actual reply text from a local chat session — grounded,
  not scripted), ~15 s.
- **Suggested question:** "What matters most to you about family?" (or
  whichever real twin answer reads best).

### S4 — The conversation (screen recording, Mac, no GPU needed)
- **What:** web chat with the founder twin: question typed, grounded reply
  streams token-by-token, voice reply plays. ~25 s, cursor visible, no dead
  air. Record at 2x display scale for crisp text; export ≤1080p.

### S5 — The sitting (screen recording, iPhone via Expo Go)
- **What:** guide asks aloud → hands-free listening indicator → answer spoken →
  "That's safely kept" → progress ticks. ~30 s. **This session IS the device
  retest** (board item) — one stone, two birds.

### S6 — Phone → TV (optional, if time)
- **What:** tap "Send to TV", code entered in a browser "TV", the portrait
  appears there mid-conversation. ~15 s. The "walks between frames" wow.

## Post & delivery

- h264 mp4, ≤1 MB per clip where possible (crf ~28, 512² or 720p for screen
  recordings), faststart flag for instant web play; poster JPG per clip.
- Files → `apps/site/assets/demo/` → new visual sections on the site (task 9)
  → deploy to the Pi. No AWS involvement anywhere.
- S2 also lands in `apps/mobile/assets/guide/` via
  `scripts/gen_guide_clips_module.py` once the full clip batch is approved.

## Session runbook (when box + founder photo are ready)

1. Guide face pick → `guide-portrait.jpg` on the box.
2. Synthesize S1–S3 audio lines (founder clone / guide voice) — voice on the
   3060, no parking needed.
3. Render S1–S3 on the 5070 Ti (persistent engine; ~5–7 min each warm).
4. Screen-record S4 (Mac) and S5 (iPhone) against the local stack.
5. ffmpeg pass (trim, crf, faststart, posters) → `apps/site/assets/demo/`.
6. Site sections + deploy (separate commit; task 9).
