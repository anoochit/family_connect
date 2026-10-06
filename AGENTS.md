# AGENTS.md — Family Connect

Instructions for AI agents (and new contributors) working in this repository.

## What this project is

A Flutter Android app for elderly/patient users at home to communicate with remote family. MVP scope, architecture, and acceptance criteria are defined in:

- `PRD.md` — product requirements (English, source of truth)
- `PRD-TH.md` — Thai translation of the PRD
- `idea.md` — original concept (Thai)

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
- Backend: Firebase (Auth, Firestore, Storage, FCM) — not yet wired up in `lib/`.
- Video/voice calls: 3rd-party prebuilt SDK, ZegoCloud recommended — wrap behind a `CallService` interface so the provider is swappable.
- Offline: voice messages and photo slideshow must queue/cache locally and sync on reconnect. Never break the home screen when offline.
- Target platform for MVP: **Android phones only**. Do not add iOS/web/tablet-specific code paths unless asked.

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

Fresh Flutter scaffold as of 2026-10: only `lib/main.dart` and default `test/widget_test.dart` exist. Dependencies beyond `flutter`/`cupertino_icons` are not yet added — add packages deliberately and justify them against the PRD stack table.
