# Family Connect (แอปสื่อสารครอบครัว)

A Flutter Android app that makes it easy for elderly family members and patients at home to stay connected with remote family — big buttons, voice-first input, and an auto-updating photo frame home screen.

> Full specs: [PRD.md](PRD.md) (English) · [PRD-TH.md](PRD-TH.md) (ไทย) · raw concept: [idea.md](idea.md)
>
> **Plan & tracking:** [PLAN.md](PLAN.md) (implementation plan) · [TASKS.md](TASKS.md) (checklist) · [AGENTS.md](AGENTS.md) (contributor/agent guide)

## Features (MVP)

- **Photo-frame home screen** — rotating family photos, oversized call button, emergency call, daily "I'm okay" check-in
- **Video / voice calling** — 1-to-1, prebuilt SDK (ZegoCloud), full-screen incoming calls
- **Voice messages** — push-to-talk, offline queue, one-tap playback
- **Photo sharing** — family uploads, auto-slideshow on the elder's home screen, local cache
- **Family calendar** — read-only agenda for elders + reminders, CRUD for family members
- **Text chat** — one family group chat, TTS read-aloud for elders
- **Important-day alerts** — birthdays & Thai holidays with greeting cards
- **Quiet notifications** — badges and in-app banners, minimal OS push

Design principles: zero typing for elders, one tap to reach family, works offline, Thai + English.

## Stack

| Layer | Choice |
|---|---|
| App | Flutter 3.x / Dart ^3.13, Material 3 |
| State | Riverpod |
| Backend | Firebase (Auth, Firestore, Storage, FCM) |
| Calls | ZegoCloud Flutter SDK (behind a swappable `CallService` interface) |
| Signaling | Firestore `calls/{callId}` docs + FCM wake-up (no Realtime DB) |
| Local cache | drift (offline outbox + photo disk cache) |

See [PLAN.md](PLAN.md) §1–§2 for the full tech decisions and dependency manifest.

## Getting started

Prerequisites: [Flutter 3.47+](https://docs.flutter.dev/get-started/install), a Firebase project, and (for calls) a ZegoCloud account.

```bash
flutter pub get        # install dependencies
flutter analyze        # lint
flutter test           # run tests
flutter run            # run on a connected Android device/emulator
```

Firebase setup (once):

```bash
dart pub global activate flutterfire_cli
flutterfire configure   # generates lib/firebase_options.dart + android/app/google-services.json
```

> Note: `google-services.json` and `firebase_options.dart` are not committed — each developer configures their own Firebase project.

## Development roadmap

Work is planned milestone-by-milestone (W1→W8) in [PLAN.md](PLAN.md) §7 and tracked as a checklist in [TASKS.md](TASKS.md). A milestone is done when its tasks are ticked and `flutter analyze` + `flutter test` are clean. Start with [TASKS.md](TASKS.md) §0 (Firebase provisioning + core dependencies).

## Project structure

Currently a fresh scaffold:

```
lib/
  main.dart            # app entry point
test/
  widget_test.dart
```

Target feature-first structure as the app grows (see [PLAN.md](PLAN.md) §3):

```
lib/
  main.dart            # bootstrap: Firebase init, ProviderScope, runApp
  app.dart             # MaterialApp.router, theme, locale, role-based routing
  core/
    theme/             # design tokens (colors, type scale, spacing) + app_theme
    l10n/              # app_en.arb, app_th.arb
    routing/           # app_router, role_guard
    widgets/           # BigButton, VoicePlayerCard, badges, dialogs...
    services/          # call_service (abstract), notification_service, tts_service
    utils/
  features/
    auth/              # invite create/join (QR or 6-digit), anonymous sign-in
    onboarding/        # 3-tap elder setup
    home_elder/        # F1 photo-frame home (slideshow, call button, check-in)
    dashboard_member/  # F8 today summary + member list + activity
    messages/          # F6 family group chat (text + attachments)
    voice/             # F3 record/playback/outbox
    photos/            # F4 upload, album, cache, seen-tracking
    calendar/          # F5 member CRUD + elder agenda + reminders + Thai holidays
    important_days/    # F7 birthdays/special days + greeting cards
    calls/             # F2 incoming/outgoing/active/result + call history
    activity/          # F8 last-seen + check-in rollups
    settings/          # language, extra-large text, sign-out
```

Each feature owns its `data` / `models` / `providers` / `screens` / `widgets`; shared code lives in `core/`. Firebase config (`google-services.json`, `firebase_options.dart`) stays untracked.

## Localization

Thai and English ship together (ARB files under `lib/core/l10n/`, per [PLAN.md](PLAN.md) §3). All elder-facing strings must exist in both.

## Testing & quality

```bash
flutter analyze        # must be clean before commit
flutter test           # widget/unit tests
```

## License

MIT — see [LICENSE](LICENSE).
