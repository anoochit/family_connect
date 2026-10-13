# AGENTS.md — Family Connect

Instructions for AI agents (and new contributors) working in this repository.

## What this project is

A Flutter Android app for elderly/patient users at home to communicate with remote family. MVP scope, architecture, and acceptance criteria are defined in:

- `PRD.md` — product requirements (English, source of truth)
- `PRD-TH.md` — Thai translation of the PRD
- `idea.md` — original concept (Thai)
- `PLAN.md` — detailed implementation plan (the *how/when*: tech decisions, dependency manifest, feature-first structure, W1–W8 milestones, testing & QA). Read it before starting a milestone.
- `TASKS.md` — the working checklist derived from `PLAN.md`. Tick items off as you complete them; a milestone is done when all its boxes are ticked and `flutter analyze` + `flutter test` are clean.

**Before adding or changing a feature, check it against the PRD.** If a request conflicts with the PRD, ask the user rather than silently diverging.

## Commands

```bash
flutter pub get        # install dependencies
flutter analyze        # lint — MUST be clean before finishing any task
flutter test           # tests
flutter run            # run on connected Android device/emulator
flutter build apk      # release APK
```

Run `flutter analyze` and `flutter test` after any code change and fix all errors/warnings introduced by your changes.

## Stack & constraints

- Flutter 3.47.x / Dart ^3.13, Material 3, portrait-first.
- State management: **Riverpod** (no Provider/Bloc) — decided in PRD §8.
- Backend: Firebase (Auth, Firestore, Storage, FCM) — not yet wired up in `lib/`.
- Call signaling: **Firestore `calls/{callId}` docs** + FCM high-priority data message as wake-up only (no Realtime DB).
- Video/voice calls: 3rd-party prebuilt SDK, ZegoCloud recommended — wrap behind a `CallService` interface so the provider is swappable.
- Offline: voice messages and photo slideshow must queue/cache locally and sync on reconnect. Never break the home screen when offline.
- Target platform for MVP: **Android phones only**. Do not add iOS/web/tablet-specific code paths unless asked.
- Add dependencies deliberately; every new package must justify itself against the PRD stack table (PRD §5.1) and the `PLAN.md` dependency manifest.

## Code conventions

- Follow existing style in `lib/`; run `flutter analyze` (uses `flutter_lints` via `analysis_options.yaml`).
- No comments in code unless explicitly requested.
- Keep files focused: one widget/feature per file; prefer `features/<name>/` folders as the app grows (see README structure).
- Never hardcode secrets, API keys, or `google-services.json` values. Firebase config files stay untracked.
- All user-facing strings go through l10n (Thai + English, ARB files) — no bare string literals in widgets. Both locales must be present.

## Design rules (elder-first)

These are hard requirements from the PRD, not preferences:

- Touch targets ≥ 64×64dp for elder-facing controls, ≥ 16dp spacing.
- Primary text ≥ 20sp; app must survive OS font scaling up to 200%.
- Max 4 top-level destinations for elders; max 2 actions per screen.
- Call button is the dominant element on the home screen; emergency button always reachable.
- No hamburger menu for elders; no swipe-only paths to any action.
- Color is never the only signal (pair with icon/text). Contrast ≥ WCAG AA.
- Notification behavior must follow the PRD §F9 matrix — do not add push notifications beyond it.

## Roles & data model reminders

- One family group; roles `elder` and `member`. Elders consume content and reply with voice; members create content.
- Elder accounts cannot create/edit calendar events or delete content.
- Join flow is invite QR/6-digit code only — no passwords, no OTP typing for elders.
- Multiple elderly users may share a household; each has their own home screen and check-ins.

## Working on tasks

1. Read the relevant PRD section first (e.g. F3 for voice messages).
2. Search the codebase for existing patterns/helpers before writing new ones.
3. Implement, then `flutter analyze` + `flutter test`.
4. If Firebase/ZegoCloud setup is required but missing, note it instead of committing credentials.
5. Do not commit, push, or open PRs unless explicitly asked.
6. Keep responses and diffs concise; no drive-by refactors outside the task.

## Current state

- **Planning complete.** `PLAN.md` holds the full implementation roadmap (tech decisions, dependency manifest, feature-first `lib/` structure, W1–W8 milestones, testing strategy, QA checklist, risk register). Start work by picking the next unfinished milestone in `PLAN.md` §7.
- **Code not started.** Only `lib/main.dart` and default `test/widget_test.dart` exist. No Firebase/ZegoCloud dependencies added yet — add packages deliberately and justify them against `PLAN.md` §2.
- **Firebase not provisioned.** No project configured; `google-services.json` / `firebase_options.dart` are untracked by design. Provision per `PLAN.md` §10 before wiring any Firebase feature.
- **Tooling ready.** MCP servers `dart` (analyze, hot reload, widget inspector) and `firebase-mcp-server` are configured in `opencode.json`. Agent skills are installed under `.agents/skills/` (`flutter-*`, `dart-*`, `firebase-*`). Load the relevant skill before implementing a feature (e.g. `firestore-rules-creation` + `firebase-security-rules-auditor` for rules, `flutter-add-widget-test` for tests).

## Roadmap summary (see PLAN.md §7 for detail)

| Week | Deliverable |
| --- | --- |
| W1 | Scaffold, Firebase, auth + invite, data model + rules, i18n skeleton, design tokens |
| W2 | Elder home (slideshow, call button, check-in, badges) + member dashboard skeleton |
| W3 | Voice messages (record/playback/offline queue) + family text chat |
| W4 | Photo upload + album + slideshow integration + seen tracking |
| W5 | Video/voice calling (ZegoCloud) + Firestore signaling + FCM wake-up + history + fallback |
| W6 | Calendar + reminders + Thai holidays + important-days |
| W7 | Notifications policy, activity/last-seen, emergency flow, accessibility pass, offline hardening |
| W8 | Elder usability testing → fixes, Crashlytics/analytics, pilot build |
