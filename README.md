# Family Connect (แอปสื่อสารครอบครัว)

A Flutter Android app that makes it easy for elderly family members and patients at home to stay connected with remote family — big buttons, voice-first input, and an auto-updating photo frame home screen.

> Full specs: [PRD.md](PRD.md) (English) · [PRD-TH.md](PRD-TH.md) (ไทย) · raw concept: [idea.md](idea.md)

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
| Backend | Firebase (Auth, Firestore, Storage, FCM) |
| Calls | ZegoCloud Flutter SDK |
| Local cache | hive / drift (offline queue + photo cache) |

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
flutterfire configure   # generates android/app/google-services.json
```

> Note: `google-services.json` is not committed — each developer configures their own Firebase project.

## Project structure

```
lib/
  main.dart            # app entry point
PRD.md                 # product requirements (English)
PRD-TH.md              # product requirements (Thai)
idea.md                # original concept
```

Target structure as the app grows:

```
lib/
  app/                 # app shell, routing, theme, l10n
  features/
    home/              # elder home screen (photo frame, call, check-in)
    calls/             # incoming/outgoing call UI + CallService
    messages/          # voice + text chat
    photos/            # upload, album, slideshow cache
    calendar/          # events, reminders, holidays
    onboarding/        # invite QR/code join flow
    settings/
  data/                # Firestore repositories, models, security rules glue
  core/                # widgets, utils, offline queue, constants
```

## Localization

Thai and English ship together (ARB files under `lib/l10n/`). All elder-facing strings must exist in both.

## Testing & quality

```bash
flutter analyze        # must be clean before commit
flutter test           # widget/unit tests
```

## License

MIT — see [LICENSE](LICENSE).
