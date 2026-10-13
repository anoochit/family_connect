# Implementation Plan — Family Connect

> Roadmap for building the MVP described in [`PRD.md`](PRD.md) (Thai/EN elder-first family
> communication app, Android phones). This plan turns the PRD into concrete, ordered work.
> **The PRD is the source of truth for *what* and *why*. This plan is the *how* and *when*.**
> If they ever conflict, follow the PRD and flag the conflict.

- **Scope:** MVP = PRD §7 (8-week solo-dev timeline), features F1–F10.
- **How to use:** Work milestone by milestone (W1→W8). Each milestone lists ordered tasks,
  the files they create, deliverables, and acceptance checks. Check a task off only when its
  acceptance check passes and `flutter analyze` + `flutter test` are clean.

---

## 1. Confirmed technical decisions

Locked by the PRD (§5.1, §8). Do not re-litigate these in code review:

| Area | Decision | PRD ref |
| --- | --- | --- |
| State management | **Riverpod** (no Provider, no Bloc) | §8, §5.1 |
| Backend | Firebase — Auth, Firestore, Storage, FCM | §5.1 |
| Call signaling | **Firestore `calls/{callId}` docs** + FCM high-priority data message as *wake-up only*. No Realtime DB. | §2 F2, §8 |
| Calling SDK | **ZegoCloud** prebuilt Flutter UI kit, wrapped behind a `CallService` interface (Agora/100ms as fallback) | §2 F2, Risk 1 |
| Offline | Firestore offline persistence + a local **outbox** (drift) for uploads + a **photo disk cache** | §5.3 |
| Recording | `record` → AAC/Opus, ≤60s, ≤5MB | §3 F3 |
| i18n | `flutter_localizations` + ARB (th + en), switchable without restart, persisted **per device** | §3.2, F10 |
| Auth model | Anonymous auth on join; **invite-code redemption** (QR or 6-digit) creates the member. No passwords/OTP for elders. | §3.2 |
| Platform | **Android phones only** for MVP. Portrait-first. | §1.4, F10 |

### 1.1 Open decision to resolve in W1 (spike)

**Invite redemption security.** The family's `inviteCodeHash` lives on the family doc, and 6-digit
codes are enumerable. Two options:

- **A (recommended): callable Cloud Function** `redeemInvite(code, displayName, photoUrl)` →
  validates the code server-side, creates the `members/{uid}` doc, marks the invite used, returns
  `familyId`. Prevents enumeration; one small function.
- **B (lighter, riskier): client-side redemption** — anonymous auth, then a client write that joins
  the family after reading a valid, unexpired `invites/{code}` doc. Requires a permissive rules read
  on invites (enumeration risk). Acceptable **only** for a closed pilot.

Decide in W1 spike and document the choice here. Option A is the default recommendation.

---

## 2. Dependencies (add deliberately — justify each against the PRD)

Add with `flutter pub add <pkg>` (resolves latest compatible; pin via `pubspec.lock`). "Phase" = the
milestone that needs it. Every entry maps to a PRD feature.

| Package | Purpose | PRD feature | Phase |
| --- | --- | --- | --- |
| `flutter_riverpod`, `riverpod_annotation` (opt.) | State management | all | W1 |
| `firebase_core` | Firebase bootstrap | §5.1 | W1 |
| `firebase_auth` | Anonymous auth / invite join | §3.2 | W1 |
| `cloud_firestore` | Data model | §5.2 | W1 |
| `firebase_storage` | Photos + voice files | F3, F4 | W1 |
| `firebase_messaging` | FCM wake-up push | F2, F9 | W1 |
| `flutter_localizations`, `intl` | th/en i18n | F10 | W1 |
| `drift`, `drift_flutter`, `path_provider` | Outbox queue + photo cache | §5.3 | W1 |
| `image_picker` | Multi-select photos (≤10) | F4 | W2 |
| `flutter_image_compress` | ≤1600px / ~300KB compression | F4 | W2 |
| `crypto` | `sha256` dedupe for photos | F4 | W2 |
| `record` | Voice recording (AAC/Opus) | F3 | W3 |
| `just_audio` | Voice playback (1× speed) | F3 | W3 |
| `connectivity_plus` | Trigger outbox flush on reconnect | §5.3 | W3 |
| `zego_uikit_prebuilt_call` | 1-to-1 video/voice call UI | F2 | W5 |
| `mobile_scanner` | Scan invite QR | §3.2 | W1 |
| `qr_flutter` | Render invite QR | §3.2 | W1 |
| `permission_handler` | Camera + mic permissions | §3.2, F3 | W1 |
| `shared_preferences` | Device-local settings (lang, extra-large text) | F10 | W1 |
| `flutter_tts` | Read-aloud (messages, events, greetings) | F6, F5, F7 | W6 |
| `speech_to_text` | Optional member dictation (**Could**) | F6 | W7 |
| `flutter_local_notifications` | Calendar reminders + scheduled alerts | F5, F9 | W6 |
| `firebase_analytics` | MVP success metrics (§6) | §6 | W8 |
| `firebase_crashlytics` | Crash-free ≥99% target | §6, W8 | W8 |

> Keep the set lean. If a package isn't tied to a PRD feature above, don't add it. Re-confirm any
> new dependency in `AGENTS.md`'s "Stack & constraints" note.

---

## 3. Repository structure (feature-first)

Follow the layered pattern the `flutter-apply-architecture-best-practices` skill recommends:
each feature owns its `data` / `models` / `providers` / `screens` / `widgets`. Shared code lives in
`core/`. Keep one widget/feature per file; no code comments unless asked (per `AGENTS.md`).

```
lib/
  main.dart                       # bootstrap: Firebase init, ProviderScope, runApp
  app.dart                        # MaterialApp.router, theme, locale, role-based routing
  firebase_options.dart           # flutterfire-generated — UNTRACKED (contains API keys)
  core/
    theme/          app_theme.dart, tokens (colors.dart, typography.dart, spacing.dart)
    l10n/           app_en.arb, app_th.arb   (generated delegates via l10n.yaml)
    routing/        app_router.dart, role_guard.dart
    widgets/        big_button.dart, elder_badge.dart, avatar_circle.dart,
                    voice_player_card.dart, offline_badge.dart, confirm_dialog.dart
    services/       call_service.dart (abstract), notification_service.dart, tts_service.dart
    utils/          date_format_th.dart, compression.dart, permissions.dart
  features/
    auth/           join via invite (scan/code), anonymous sign-in, member bootstrap
    onboarding/     3-tap elder setup (name, test voice, favorite contact)
    home_elder/     F1 photo-frame home (slideshow, call button, check-in, banners)
    dashboard_member/ F8 today-summary + member list + activity
    messages/       F6 family group chat (text + attachments) + system channel
    voice/          F3 record/playback/outbox
    photos/         F4 upload, album, cache, seen-tracking
    calendar/       F5 member CRUD + elder agenda + reminders + Thai holidays
    important_days/ F7 birthdays/special days + greeting cards
    calls/          F2 incoming/outgoing/active/result + call history (Firestore-backed)
    activity/       F8 last-seen + check-in rollups
    settings/       lang switch, extra-large text, sign-out (hidden, confirmed)
  models/           (optional shared value objects; prefer feature-local models)
test/
  unit/  widget/  integration/     # mirror lib/ feature folders
```

- **Firebase config stays untracked** (`.gitignore` `google-services.json`,
  `firebase_options.dart`, `lib/firebase_options.dart`). Regenerate with FlutterFire CLI; never
  commit keys (per `AGENTS.md`).
- Generate l10n into `lib/core/l10n/generated/` (or default `.dart_tool/flutter_gen`); keep both
  locales complete at all times.

---

## 4. Design system (elder-first tokens)

Centralize in `core/theme/tokens/`. Every elder-facing control ≥ 64×64dp, ≥16dp spacing,
primary text ≥ 20sp, survives 200% OS font scale (PRD §3 F10, §9).

**Color (WCAG AA, never color-only):**

| Token | Value | Use |
| --- | --- | --- |
| `primary` | `#1B5E20` | Call button, confirm, primary actions |
| `danger` | `#C62828` | Emergency, decline, destructive |
| `surface` | `#FFFFFF` / very light | Backgrounds |
| `onSurface` | near-black | Text |
| scrim | black @ ~40% | Behind text on photos |

**Type scale:** base body 20sp; elder "Extra large text" multiplies base ×1.3 (a settings toggle).
Always pair color with an icon **and** text label.

**Key reusable components (build once in `core/widgets/`):**

- `BigButton` — ≥64dp, icon + label, ≥2 semantic variants (primary/danger).
- `OversizedCircleButton` — the ~40%-width call button and the emergency button.
- `VoicePlayerCard` — avatar + name + big ▶ + duration (F3 receive side).
- `ElderBadge` — large numeric badge (no red dots) for nav items (F1/F9).
- `OfflineBadge` — "รอส่ง / Waiting to send" chip for queued items (F3/F4).
- `ConfirmDialog` — 5s-undo or explicit confirm for hard actions (§9).

---

## 5. Data layer

### 5.1 Firestore schema (from PRD §5.2)

```
families/{familyId}
  name, inviteCodeHash, createdAt, settings{langDefault,...}
  members/{userId}
    displayName, photoUrl, role:'elder'|'member', fcmTokens[], lastSeenAt,
    primaryContact:bool,  checkIns/{yyyy-MM-dd}:{status,at}
  messages/{msgId}
    type:'text'|'voice'|'photo'|'system', senderId, senderName, text?, mediaUrl?,
    durationMs?, caption?, createdAt, seenBy[], status:'sending'|'sent'|'failed'|'queued'
  photos/{photoId}
    uploaderId, url, thumbUrl, caption?, sha256, createdAt, seenBy[]
  events/{eventId}
    title, startAt, endAt?, recurrence?, remindLead?, assigneeId?|'all',
    category, note?, createdBy, isHoliday
  days/{dayId}
    type:'birthday'|'special', label, date, recurring, assigneeId, greetingVoiceUrl?
  invites/{code}
    role, createdBy, expiresAt, usedBy?
  calls/{callId}
    fromId, toId, type:'video'|'voice',
    status:'ringing'|'accepted'|'declined'|'ended'|'missed',
    startedAt, answeredAt?, endedAt?, durationMs?, sdkCallId?
```

Model each as an immutable Dart class in `features/<f>/models/` with `fromFirestore`/`toFirestore`
(apply the `flutter-implement-json-serialization` skill). **Create composite indexes early** —
messages & photos ordered by `createdAt`, events by `startAt`; add to `firestore.indexes.json`.

### 5.2 Security rules (sketch — harden with the auditor skill)

Scope every rule to `request.auth.uid ∈ familyId.members`. Elders get write access **only** to
`messages` (voice/text), their own `checkIns`, and `seenBy` arrays. Members can CRUD events/days/
photos. Full authoring → `firestore-rules-creation` skill; review →
`firebase-security-rules-auditor` skill. Sketch:

```
function memberOf(familyId) = exists(families/$(familyId)/members/$(request.auth.uid))
function role(familyId)     = get(...members/$(request.auth.uid)).data.role
function isMember(familyId) = memberOf && role == 'member'
function isElder(familyId)  = memberOf && role == 'elder'

families/{familyId}:
  read : memberOf
  write: false                       # family doc managed server-side
  members/{uid}: read=memberOf; create=request.auth.uid==uid; update=self|member(primaryContact)
  messages/{id}: read=memberOf; create=memberOf && senderId==auth.uid
                 && (isMember || (isElder && type in ['voice','text']));
                 update=memberOf && onlyKeys(['seenBy','status']); delete=isMember
  photos/{id}:   read=memberOf; create=isMember; update=onlyKeys(['seenBy']); delete=isMember
  events|days:   read=memberOf; create/update/delete=isMember
  calls/{id}:    read=memberOf; create=memberOf && fromId==auth.uid; update=memberOf
invites/{code}:  validated server-side (callable fn recommended — see §1.1)
```

### 5.3 Offline strategy (drift)

Two drift tables (message/event reads rely on Firestore's own persistence — §5.3.1):

- **`outbox`** — pending uploads: `id, kind('voice'|'photo'), localPath, familyId, payloadJson,
  attempts, nextAttemptAt, status, createdAt`. Retry with exponential backoff, max 3, surface
  `queued`/`failed` in UI (F3/F4). Flush on `connectivity_plus` reconnect.
- **`cached_photos`** — last-50 disk cache for the elder slideshow: `photoId, localPath, url,
  thumbUrl, uploaderId, caption, sha256, createdAt, lastSeenAt`. Evict oldest beyond 50 (cloud keeps
  up to 500; cache eviction ≠ deletion) (F4).

Home screen must **never render blank** — serve slideshow from `cached_photos`; placeholder only if
cache empty (F1, F4).

---

## 6. Routing & role gating (§5.4)

```
/onboarding        join-scan | join-code | setup-elder
/elder-home        default tab for role=elder   (F1)
/messages → /chat  family group chat            (F6)
/calendar          elder agenda / member CRUD    (F5)
/settings          hidden for elders unless member
member-home        dashboard                     (F8)
/call/incoming | /call/active | /call/result     (F2)
```

- Elders: **2-tab** bar (Home, Messages) or home + overlay cards. Members: 4 tabs. Max 2 actions
  per screen; no hamburger; no swipe-only paths (§9).
- A `role_guard` redirects on first frame based on the loaded member doc; cache role locally for
  fast cold start.

---

## 7. Milestones (W1–W8, from PRD §7)

Each milestone = ordered tasks + deliverable + acceptance checks. Run `flutter analyze` + `flutter test`
before marking any milestone done.

### W1 — Foundation
Scaffold, Firebase project, auth + invite, data model + rules, i18n skeleton, design tokens.

1. Firebase project + FlutterFire CLI; add Android app; `flutterfire configure` →
   `firebase_options.dart` (untracked). Enable Auth (Anonymous), Firestore, Storage, FCM.
2. Add core deps (§2 W1 list). Set up `analysis_options.yaml` (keep `flutter_lints`).
3. Design tokens + `app_theme.dart` + `BigButton`/`OversizedCircleButton` (§4).
4. i18n skeleton: `l10n.yaml`, `app_th.arb`, `app_en.arb`, locale switch (no restart), persisted
   per device (`shared_preferences`). Both locales populated.
5. Riverpod `ProviderScope`; `app.dart` with `MaterialApp.router` + role guard skeleton.
6. **Spike:** resolve invite redemption (§1.1, Option A vs B). Build invite create (QR + 6-digit via
   `qr_flutter`) + join (scan via `mobile_scanner` / type code) + anonymous auth.
7. Firestore data model classes + **security rules** (§5.2) + `firestore.indexes.json`.

**Acceptance:** a member can create an invite; a second device joins via QR/6-digit and lands on the
right role home. Rules deployed; `flutter analyze` clean; th/en switch works.

### W2 — Elder home screen + member dashboard skeleton
1. `home_elder`: full-bleed photo slideshow (8s, crossfade, dark scrim), tap → meta + prev/next,
   auto-advance pause 10s. Empty state illustration (F1).
2. Photo disk cache (`cached_photos`) + prefetch last-50; slideshow reads cache-first (F4 prep).
3. `OversizedCircleButton` "Call family" → contact list (big avatars); **Emergency** red button
   (primary-contact call + urgent push to all members) (F1, F8).
4. Daily check-in pill ("วันนี้สบายดี / I'm okay today") → ✓ + timestamp, resets daily; long-press →
   "ไม่สบาย" second state (F1).
5. `ElderBadge` unread counts on nav (F1).
6. `dashboard_member` skeleton: today-summary card + member list placeholder (F8).

**Acceptance:** elder home shows rotating cached photos, dominant call button, working check-in, and
never blanks offline.

### W3 — Voice messages + family text chat
1. `record` → AAC/Opus recorder with tap-to-toggle (elder default) + push-to-talk (member), waveform
   preview, re-record, ≤60s/≤5MB (F3).
2. `just_audio` playback card (`VoicePlayerCard`): big play/pause, 1× speed, auto-download (F3).
3. Outbox (drift): save-locally-first, `queued`/`waiting to send`, auto-flush on reconnect, 3× retry
   with visible status (F3, §5.3).
4. Storage path `/families/{familyId}/voice/{msgId}.opus|aac` (F3).
5. `messages` feature: one family **group chat** + system channel; text min 20sp; delivery/read
   ticks; 100-msg local history + infinite scroll to cloud; large 🔊 TTS button per message (F6).

**Acceptance:** elder records & sends a voice message offline → shows "รอส่ง" → auto-sends on
reconnect; member receives and plays it.

### W4 — Photo upload + album + seen tracking
1. `image_picker` multi-select ≤10; `flutter_image_compress` to ≤1600px/~300KB; upload progress;
   optional caption (F4).
2. Storage `/families/{familyId}/photos/{photoId}` + Firestore doc (uploader, ts, caption, sha256
   dedupe) (F4).
3. Wire uploads through outbox (queue offline) (F4, §5.3).
4. Slideshow integration: newest-first, "new photo" badge until elder **seen** (`seenBy` event) (F4).
5. Cache eviction: keep last 50 on disk; 500-photo cloud cap (F4).

**Acceptance:** member uploads 10 photos; they appear in elder slideshow in order; "new" badge clears
after elder views; offline upload queues and syncs.

### W5 — Calling (ZegoCloud) ★ highest-risk week
1. **Spike (time-box ~1 day):** ZegoCloud prebuilt call on a real Android device — connect time <5s
   on 4G; verify **incoming call with app closed** (FCM high-priority wake-up + full-screen intent).
   Android 14+ `USE_FULL_SCREEN_INTENT` restrictions are the key risk (Risk #1/#3). Validate early;
   keep `CallService` interface so Agora/100ms can swap in.
2. `CallService` abstract interface + ZegoCloud impl (init with per-user token from a Cloud Function
   or secure endpoint — **never ship Zego app-signing secrets in the APK**).
3. Firestore call signaling: create `calls/{callId}` (`ringing`), callee snapshot listener, status
   transitions (`accepted|declined|ended|missed`); FCM data message = wake-up only (F2).
4. Incoming screen (caller photo, giant accept/decline, ringtone); in-call UI (mute, speaker, end,
   camera flip; no chat/reactions) (F2).
5. Voice-only option from contact list; call-history last-10 + missed badges from `calls` (F2).
6. Fallback error screen: "การเชื่อมต่อขัดข้อง / Connection problem" + Retry + "ส่งข้อความเสียงแทน"
   escape hatch (F2).

**Acceptance:** elder receives a call with app closed → one tap answers; connects <5s on 4G; failed
calls show the friendly fallback.

### W6 — Calendar + reminders + Thai holidays + important days
1. Member event CRUD: title, date/time, recurrence, reminder lead (15m/1h/1d), assignee, note,
   one-tap templates (พบหมอ/ญาติมาเยี่ยม/ทานยา/กิจกรรมครอบครัว) (F5).
2. Elder **agenda** view: "วันนี้ / พรุ่งนี้ / สัปดาห์นี้" giant day headers + colored chips; optional
   month-grid toggle (dots only) (F5).
3. Reminders: in-app big card at reminder time; OS notification **only** if ≤1h or flagged important
   (doctor visit) via `flutter_local_notifications` (F5, F9).
4. Elder "แจ้งครอบครัว / Tell family" → pre-fills a voice message (F5). Elders cannot edit.
5. Bundled **Thai holiday** dataset (local, offline) auto-marked read-only (Parents' Day, Mother's/
   Father's Day) (F5).
6. `important_days` (F7): birthdays/special days per member; day-of elder greeting card + one-tap
   "ฟังคำอวยพร" (pre-recorded or TTS) + "ส่งคำขอบคุณ" (voice); member badge 1 day before. OS
   notification off by default (F7, F9).
7. `flutter_tts` read-aloud on messages, events, greetings (Thai + English voices) (F6/F5/F7).

**Acceptance:** member creates a "พบหมอ" event → elder sees it in agenda → reminder card fires at the
right time; birthday shows a greeting card with a playable voice.

### W7 — Notifications policy, activity, emergency, accessibility, offline hardening
1. Implement the **F9 notification matrix** exactly (elder quiet / member push; emergency = high
   priority, unmutable). Wire FCM topics/tokens per role.
2. `activity` (F8): last-seen/active-today card ("คุณแม่เปิดแอปวันนี้ 08:14" + green/gray dot), with
   privacy toggle; member today-summary (check-in status, unread voice, upcoming events, new photos).
3. Emergency flow end-to-end (initiate → high-priority push to all members) (F1, F9).
4. Accessibility pass: 200% font scale, extra-large-text toggle, WCAG AA contrast, ≥64dp targets,
   TTS read-aloud, no swipe-only paths (F10).
5. Offline hardening: kill network mid-record/mid-upload/mid-call; verify outbox + cache + friendly
   errors everywhere (§5.3).

**Acceptance:** notification behavior matches the F9 table on a real device; app survives full offline
usage without blank screens or stack traces.

### W8 — Testing, hardening, pilot
1. **Moderated usability test** with 5 elders: answer call, send voice, tap check-in — zero prompts
   (§6 qualitative). Fix findings.
2. Wire `firebase_analytics` + `firebase_crashlytics`; track §6 metrics (D7 retention, check-in
   completion, voice/week, crash-free ≥99%).
3. Crash/analytics dashboards; Play internal test track build.
4. Final `flutter analyze` + `flutter test` + integration smoke; zero P0/P1 bugs.

**Acceptance:** all F1–F10 acceptance criteria met; crash-free ≥99%; pilot build on Play internal
track; every tested elder completes the 3 core tasks unaided.

---

## 8. Testing strategy

Use the installed skills: `dart-add-unit-test`, `flutter-add-widget-test`,
`flutter-add-integration-test`, `dart-collect-coverage`, `dart-run-static-analysis`,
`dart-generate-test-mocks`.

- **Unit:** repositories, outbox retry/backoff, compression, date/thai-holiday logic, call status
  transitions, JSON (de)serialization. Mock Firebase with `dart-generate-test-mocks` (mockito).
- **Widget:** `BigButton`/`VoicePlayerCard`/badges render at ≥64dp & 200% font scale; elder home
  empty state; check-in toggle; agenda list.
- **Integration:** invite join → first voice send → offline queue → reconnect flush; call signaling
  state machine.
- **Coverage:** `dart-collect-coverage` → LCOV; target meaningful coverage on `data/` + outbox.
- **Usability (manual):** §7 W8 elder test is the real gate — a feature isn't "done" until an elder
  can use it unaided.

---

## 9. Manual QA / accessibility checklist (run before each milestone sign-off)

- [ ] All elder controls ≥64×64dp, ≥16dp spacing; primary text ≥20sp.
- [ ] App renders correctly at 200% OS font scale and with "Extra large text" on.
- [ ] Every action reachable without swipe-only gestures; no hamburger for elders.
- [ ] Color never the sole signal (icon + text always present); contrast ≥ WCAG AA.
- [ ] th + en strings both present; language switch needs no restart.
- [ ] Offline: home never blanks; queued voice/photos show "รอส่ง" and sync on reconnect.
- [ ] Hard/destructive actions have 5s undo or explicit confirm.
- [ ] Errors show plain-language copy (what happened + one next step), never a stack trace.

---

## 10. Firebase provisioning (untracked config)

1. Create Firebase project; enable **Auth (Anonymous)**, **Firestore**, **Storage**, **FCM**.
2. `flutterfire configure` → generates `firebase_options.dart` (**untracked**, contains API keys).
   Keep `google-services.json` untracked too (per `AGENTS.md`).
3. Deploy security rules + `firestore.indexes.json` (`firebase deploy --only firestore`).
4. If using invite Option A (§1.1): deploy the `redeemInvite` callable function + a Zego token
   generator function. **Never ship signing secrets in the APK.**
5. Add a test device for FCM; verify Doze/background delivery (Risk #3).

---

## 11. Risk register (from PRD §8, with plan actions)

| # | Risk | Plan action | When |
| --- | --- | --- | --- |
| 1 | ZegoCloud limits/cost; SDK swap | W5 spike; `CallService` interface; keep Agora/100ms as fallback | W5 |
| 2 | Elder can't self-setup | Member-assisted invite setup; record setup in usability test | W1, W8 |
| 3 | FCM delayed → missed calls | High-priority wake-up + persistent Firestore listener; missed-call card; test on Doze | W5, W7 |
| 4 | Photo storage cost | ≤300KB compression, 500-cap, check Storage pricing before pilot | W4 |
| 5 | Two elders / shared device | Per-account home screen; account switcher is out-of-MVP (document as known limitation) | W2 |
| 6 | Notifications too quiet → missed msgs | Instrument open-rate; tune in W7 | W7 |
| 7 | Thai TTS quality on low-end | Test 2–3 low-end Androids; fallback to bundled audio | W6–W7 |
| 8 | QR needs camera permission | Always provide 6-digit fallback | W1 |

---

## 12. Definition of Done (MVP)

- All F1–F10 acceptance criteria met; zero P0/P1 bugs.
- `flutter analyze` + `flutter test` clean; integration smoke passes.
- An elder completes answer-call, send-voice, and check-in **with no assistance** (usability test).
- Crash-free sessions ≥99%; §6 metrics instrumented.
- th/en complete; accessibility checklist (§9) fully passing.
- Pilot build on Play internal track.

**MoSCoW (PRD §7):** Must = F1 home, F2 calls, F3 voice, F4 photos, invite/onboarding, offline
queue, emergency, check-in, th/en, accessibility basics. Should = F5 calendar, F6 text chat, F7
important days, F8 activity, TTS. Could = extra-large text, STT dictation, voice greetings, digest
reminder. Won't (v2) = iOS, tablets, group calls, DMs, multi-family, location, health, monetization.

---

## 13. Agent tooling available in this repo

Configured for this project — leverage them to move faster and stay consistent:

- **MCP servers** (`opencode.json`): `dart` (analyze, hot reload/restart, widget inspector, LSP) and
  `firebase-mcp-server` (project/config, Firestore, Functions, deploy status). Use the `dart` MCP for
  `analyze_files` + hot reload after code changes.
- **Installed skills** (`.agents/skills/`): `flutter-*` (widget/integration tests, architecture,
  responsive layout, JSON serialization, localization, routing), `dart-*` (unit tests, static
  analysis, coverage, mocks, pattern matching, docs), `firebase-*` (basics, auth, firestore,
  security-rules-auditor, firestore-rules-creation, crashlytics, remote-config, data-connect).
  Load the relevant skill before writing that feature's code.
