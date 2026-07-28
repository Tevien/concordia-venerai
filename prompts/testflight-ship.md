# Brief: ship the iOS app to TestFlight (Sean + nan as internal testers)

You are a Claude session with the VENER.AI monorepo checked out. Run this when
(a) Apple Developer enrollment is active and (b) an HTTPS backend exists
(AWS via prompts/aws-deploy.md, or the Tailscale Serve interim — both phones
must then run Tailscale).

## Before anything else

Read `CLAUDE.md` here and in the monorepo root. EAS account: logged in as
sean.venerai; project already linked (id 4722e47c-b6d2-4394-b970-78a289061489,
slug `venerai`). Node ≥ 20 required: `export PATH=/opt/homebrew/opt/node@22/bin:$PATH`.
Detailed runbook: monorepo `docs/IOS-TESTFLIGHT.md`.

## Task

1. Confirm `expo.extra.serverUrl` in `apps/mobile/app.json` is the HTTPS backend.
2. Wire OTA updates while you are here: `npx expo install expo-updates` +
   `eas update:configure` (decided: silent JS-level updates for family phones).
3. `cd apps/mobile && npx eas-cli@latest build --platform ios --profile production`
   — first run creates the App Store Connect app + signing; Sean answers the
   Apple prompts. Seller shows the enrollment's name (individual now is fine;
   organization conversion happens before public release — decision log
   2026-07-25).
4. `npx eas-cli@latest submit --platform ios --latest`.
5. Sean in App Store Connect → TestFlight → Internal Testing: add his Apple ID
   and nan's; both phones install the TestFlight app and accept the invites.
6. Verify on Sean's phone: login, sitting screen (hands-free VAD + guide video
   paths), upload, chat. Only after that, nan's phone.
7. Update this repo's board (TestFlight rows → 🟢) and note the 90-day
   TestFlight build expiry: schedule a rebuild reminder ~day 80.

## Definition of done

- Build in TestFlight, both testers invited, app verified working on Sean's
  device against the production backend; boards updated.

## Report back

Build number, what was verified on-device, anything failing, expiry date of the
build, and what remains before nan's first real sitting (see the board: sitting
refinement gates it).
