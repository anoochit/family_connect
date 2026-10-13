# Tasks — Family Connect MVP

> Actionable checklist generated from [`PLAN.md`](PLAN.md). The PRD is the source of truth for *what*
> and *why*; `PLAN.md` for *how/when*; this file is the **tick-list** for actually doing the work.
>
> **How to use**
> - Work top-to-bottom, milestone by milestone (W1→W8). A milestone is done when all its boxes are
>   ticked **and** `flutter analyze` + `flutter test` are clean.
> - Tick a task only when its acceptance check passes — not when the code is merely written.
> - Each task references a PRD feature (F1–F10) and/or a `PLAN.md` section (§n) for detail.
> - Load the relevant installed skill before implementing (see §0.3).

## Progress tracker

| Milestone | Scope | Status |
| --- | --- | --- |
| 0. Setup | Firebase, deps, repo hygiene | ⬜ not started |
| W1 — Foundation | Auth + invite, data model + rules, i18n, tokens | ⬜ not started |
| W2 — Elder home | Slideshow, call button, check-in, dashboard skeleton | ⬜ not started |
| W3 — Voice + chat | Record/playback/outbox + family text chat | ⬜ not started |
| W4 — Photos | Upload, album, seen-tracking, cache eviction | ⬜ not started |
| W5 — Calling | ZegoCloud + Firestore signaling + FCM wake-up | ⬜ not started |
| W6 — Calendar + days | Events, reminders, Thai holidays, important days | ⬜ not started |
| W7 — Polish | Notifications policy, activity, emergency, a11y, offline | ⬜ not started |
| W8 — Pilot | Usability test, analytics/crash, Play build | ⬜ not started |
| DoD | Definition-of-Done gate | ⬜ not started |

---

## 0. Setup & provisioning

### 0.1 Firebase provisioning (PLAN §10) — manual, outside the repo
- [ ] Create Firebase project; enable **Auth (Anonymous)**, **Firestore**, **Storage**, **FCM**
- [ ] Run `flutterfire configure` → generates `lib/firebase_options.dart`
- [ ] Confirm `.gitignore` covers `google-services.json` + `firebase_options.dart` (untracked)
- [ ] Create Firestore in the correct region; note the location
- [ ] Add a physical Android test device; verify FCM token registers
- [ ] Verify background/Doze FCM delivery on the test device (Risk #3)

### 0.2 Dependencies (PLAN §2) — add deliberately, justify each
- [ ] W1 batch: `flutter_riverpod`, `firebase_core`, `firebase_auth`, `cloud_firestore`,
      `firebase_storage`, `firebase_messaging`, `flutter_localizations`, `intl`, `drift`,
      `drift_flutter`, `path_provider`, `mobile_scanner`, `qr_flutter`, `permission_handler`,
      `shared_preferences`
- [ ] W2 batch: `image_picker`, `flutter_image_compress`, `crypto`
- [ ] W3 batch: `record`, `just_audio`, `connectivity_plus`
- [ ] W5 batch: `zego_uikit_prebuilt_call`
- [ ] W6 batch: `flutter_tts`, `flutter_local_notifications`
- [ ] W7 batch: `speech_to_text` (optional / Could)
- [ ] W8 batch: `firebase_analytics`, `firebase_crashlytics`
- [ ] Confirm every added package maps to a PRD feature; remove any that don't (PLAN §2 note)

### 0.3 Repo hygiene & tooling
- [ ] `flutter_lints` active in `analysis_options.yaml`; `flutter analyze` runs clean on scaffold
- [ ] Feature-first folder skeleton created under `lib/` per PLAN §3 (`core/` + `features/*`)
- [ ] `test/unit`, `test/widget`, `test/integration` folders created (mirror `lib/`)
- [ ] Note in `AGENTS.md` that tooling (dart/firebase MCP, flutter-*/dart-*/firebase-* skills) is ready

---

## W1 — Foundation (PLAN §7 W1)

**Goal:** invite-based auth works end-to-end, data model + security rules deployed, i18n + design
tokens + Riverpod wiring in place.

### W1.1 Project bootstrap
- [ ] `main.dart`: Firebase init + `ProviderScope` + `runApp`
- [ ] `app.dart`: `MaterialApp.router` with theme + locale + role-guard hook
- [ ] Riverpod `ProviderScope` wired; no Provider/Bloc anywhere (PLAN §1)

### W1.2 Design tokens & core components (PLAN §4)
- [ ] `core/theme/tokens/colors.dart` — primary `#1B5E20`, danger `#C62828`, surface, onSurface, scrim
- [ ] `core/theme/tokens/typography.dart` — base 20sp; extra-large ×1.3 toggle hook
- [ ] `core/theme/tokens/spacing.dart` — ≥16dp spacing scale
- [ ] `app_theme.dart` — Material 3 theme built from tokens
- [ ] `core/widgets/big_button.dart` — ≥64dp, icon+label, primary/danger variants
- [ ] `core/widgets/oversized_circle_button.dart` — ~40% width call + emergency buttons
- [ ] Widget tests: buttons render ≥64dp and survive 200% font scale

### W1.3 i18n skeleton (PLAN §3, F10)
- [ ] `l10n.yaml` configured; gen output path decided
- [ ] `app_en.arb` + `app_th.arb` created; **both** locales populated for all W1 strings
- [ ] Locale switch in Settings without restart; persisted per device (`shared_preferences`)
- [ ] Add string for every new UI element as it's built (no bare literals — AGENTS.md)

### W1.4 Routing & role gating (PLAN §6)
- [ ] `core/routing/app_router.dart` — routes: onboarding, elder-home, messages, calendar, settings, member-home, call/*
- [ ] `core/routing/role_guard.dart` — redirect by role on first frame; cache role locally
- [ ] Elder = 2-tab (or home+overlay); member = 4 tabs

### W1.5 Auth & invite flow (PLAN §1.1, §7 W1.6) — SPIKE first
- [ ] **Spike:** decide invite redemption Option A (callable fn) vs B (client-side); document in PLAN §1.1
- [ ] Anonymous sign-in on join
- [ ] `features/auth/`: invite create — pick role, generate 6-digit code + QR (`qr_flutter`), 24h expiry
- [ ] `features/auth/`: join via QR scan (`mobile_scanner`) **and** via typed 6-digit (camera-permission fallback, Risk #8)
- [ ] Display name + optional photo capture on join
- [ ] If Option A: implement/deploy `redeemInvite` callable Cloud Function
- [ ] Elder first-run 3-tap setup skeleton (`features/onboarding/`): name, test voice, favorite contact

### W1.6 Data model & security rules (PLAN §5.1, §5.2)
- [ ] Immutable Dart model classes for all collections (`fromFirestore`/`toFirestore`) — use `flutter-implement-json-serialization` skill: families, members (+checkIns), messages, photos, events, days, invites, calls
- [ ] `firestore.rules` per PLAN §5.2 — scope to `auth.uid ∈ family members`; elders limited to messages/checkIns/seenBy
- [ ] `firestore.indexes.json` — composite indexes: messages & photos by `createdAt`, events by `startAt`
- [ ] Deploy rules + indexes (`firebase deploy --only firestore`)
- [ ] Run `firebase-security-rules-auditor` skill against the rules; fix findings

**W1 acceptance:** a member creates an invite; a 2nd device joins via QR/6-digit and lands on the correct role home. Rules deployed; `flutter analyze` clean; th/en switch works.

---

## W2 — Elder home screen + member dashboard skeleton (PLAN §7 W2)

**Goal:** the photo-frame home renders from cache, dominant call button + emergency + check-in work, never blanks offline.

- [ ] `features/home_elder/`: full-bleed photo slideshow — 8s/photo, slow crossfade, dark scrim behind text (F1)
- [ ] Slideshow tap → photo meta (uploader, when) + prev/next arrows; auto-advance pauses 10s (F1)
- [ ] Empty-state illustration + "ยังไม่มีรูป" copy; call button still dominant (F1)
- [ ] `core/widgets/` photo disk cache (`cached_photos` drift table) + prefetch last-50; slideshow reads cache-first (PLAN §5.3, F4 prep)
- [ ] "Call family" `OversizedCircleButton` → contact list with big avatars (F1)
- [ ] Emergency red button: primary-contact call + urgent in-app badge + push to all members (F1, F8)
- [ ] Daily check-in pill "วันนี้สบายดี / I'm okay today" → ✓ + timestamp, resets daily (F1)
- [ ] Check-in long-press → "ไม่สบาย / Not feeling well" second state (F1)
- [ ] `ElderBadge` unread counts on nav (Messages, Calendar) — numeric, no red dots (F1)
- [ ] `features/dashboard_member/` skeleton: today-summary card + member list placeholder (F8)
- [ ] Widget tests: home renders at 200% font scale; empty state; check-in toggle

**W2 acceptance:** elder home shows rotating cached photos, dominant call button, working check-in; never blanks offline.

---

## W3 — Voice messages + family text chat (PLAN §7 W3)

**Goal:** elder records & sends voice offline → auto-syncs on reconnect; member receives & plays it; group text chat present.

- [ ] `features/voice/`: recorder (`record`) → AAC/Opus; tap-to-toggle (elder default) + push-to-talk (member option) (F3)
- [ ] Waveform preview + re-record; enforce ≤60s and ≤5MB (F3)
- [ ] `VoicePlayerCard` (`just_audio`): big play/pause, 1× speed, auto-download, avatar+name+duration (F3)
- [ ] `core/widgets/offline_badge.dart` — "รอส่ง / Waiting to send" chip (PLAN §4)
- [ ] Outbox (drift `outbox` table): save-locally-first, `queued` status, auto-flush on reconnect, 3× retry with visible status (PLAN §5.3, F3)
- [ ] Storage path `/families/{familyId}/voice/{msgId}.opus|aac` (F3)
- [ ] `connectivity_plus` triggers outbox flush on reconnect (PLAN §2)
- [ ] `features/messages/`: one family **group chat** + system channel (F6)
- [ ] Text messages render ≥20sp; delivery/read ticks; 100-msg local history + infinite scroll to cloud (F6)
- [ ] Large 🔊 TTS read-aloud button per message (`flutter_tts` — defer full th/en voices to W6 if needed) (F6)
- [ ] TTS notification banner "ข้อความเสียงจากคุณลูกเล็ก" when possible (F3)
- [ ] Unit tests: outbox retry/backoff; model (de)serialization

**W3 acceptance:** elder records & sends voice offline → shows "รอส่ง" → auto-sends on reconnect; member receives and plays it.

---

## W4 — Photo upload + album + seen tracking (PLAN §7 W4)

**Goal:** member uploads ≤10 photos → appear in elder slideshow in order → "new" badge clears when elder views; offline upload queues.

- [ ] `image_picker` multi-select up to 10 (F4)
- [ ] `flutter_image_compress` → ≤1600px / ~300KB; show upload progress (F4)
- [ ] Optional caption; upload through outbox (queue offline) (F4, PLAN §5.3)
- [ ] Storage `/families/{familyId}/photos/{photoId}` + Firestore doc (uploader, ts, caption, `sha256` dedupe via `crypto`) (F4)
- [ ] Slideshow integration: newest-first; "new photo" badge until elder **seen** (`seenBy` event logged) (F4)
- [ ] Cache eviction: keep last 50 on disk (oldest evicted); cloud cap 500 (F4)
- [ ] Unit tests: compression, sha256 dedupe, cache eviction logic

**W4 acceptance:** member uploads 10 photos → elder slideshow shows them in order → "new" badge clears after view → offline upload queues and syncs.

---

## W5 — Calling via ZegoCloud ★ highest-risk (PLAN §7 W5)

**Goal:** 1-to-1 video/voice calls with Firestore signaling + FCM wake-up; elder answers with app closed; connects <5s on 4G.

- [ ] **Spike (time-box ~1 day):** ZegoCloud prebuilt call on a real Android device — measure connect time <5s on 4G
- [ ] Spike: verify **incoming call with app closed** — FCM high-priority wake-up + full-screen intent; test Android 14+ `USE_FULL_SCREEN_INTENT` restrictions (Risk #1/#3)
- [ ] Document spike result + fallback viability (Agora/100ms) in PLAN §11
- [ ] `core/services/call_service.dart` — abstract `CallService` interface (provider-swappable) (PLAN §1)
- [ ] ZegoCloud implementation of `CallService`; per-user token from a Cloud Function/secure endpoint — **never ship signing secrets in APK** (Risk #1)
- [ ] Firestore call signaling: create `calls/{callId}` (`ringing`), callee snapshot listener, transitions `accepted|declined|ended|missed`; FCM data message = wake-up only (F2)
- [ ] `features/calls/incoming`: full-screen caller photo, giant Accept/Decline, audible ringtone (F2)
- [ ] `features/calls/active`: oversized mute, speaker, end, camera flip; video fills screen; no chat/reactions (F2)
- [ ] Voice-only call option from contact list (data-saver) (F2)
- [ ] Call history last-10 + missed-call badges derived from `calls` docs (F2)
- [ ] `features/calls/result`: fallback error screen "การเชื่อมต่อขัดข้อง / Connection problem" + Retry + "ส่งข้อความเสียงแทน" escape hatch (F2)
- [ ] Unit tests: call status state machine

**W5 acceptance:** elder receives a call with app closed → one tap answers; connects <5s on 4G; failed calls show friendly fallback.

---

## W6 — Calendar + reminders + Thai holidays + important days (PLAN §7 W6)

**Goal:** member creates a "พบหมอ" event → elder sees it in agenda → reminder fires; birthday shows a greeting card with playable voice.

- [ ] `features/calendar/` member event CRUD: title, date/time, recurrence, reminder lead (15m/1h/1d), assignee, note (F5)
- [ ] One-tap event templates: พบหมอ / ญาติมาเยี่ยม / ทานยา / กิจกรรมครอบครัว (F5)
- [ ] Elder **agenda** view: "วันนี้ / พรุ่งนี้ / สัปดาห์นี้" giant day headers + colored chips (F5)
- [ ] Optional month-grid toggle (dots only) (F5)
- [ ] Reminders: in-app big card at reminder time; OS notification **only** if ≤1h or flagged important via `flutter_local_notifications` (F5, F9)
- [ ] Elder "แจ้งครอบครัว / Tell family" → pre-fills a voice message; elders cannot edit events (F5)
- [ ] Bundled **Thai holiday** dataset (local, offline) auto-marked read-only — Parents' Day, Mother's/Father's Day (F5)
- [ ] `features/important_days/` (F7): birthdays/special days per member (`days` collection)
- [ ] Day-of elder greeting card + one-tap "ฟังคำอวยพร" (pre-recorded or TTS) + "ส่งคำขอบคุณ" (voice) (F7)
- [ ] Member badge 1 day before ("พรุ่งนี้วันเกิดคุณแม่"); OS notification off by default (F7, F9)
- [ ] `flutter_tts` read-aloud on messages, events, greetings — Thai + English voices (F6/F5/F7)
- [ ] Test Thai TTS on 2–3 low-end Androids; fallback to bundled audio if poor (Risk #7)
- [ ] Unit tests: recurrence logic, thai-holiday lookup, reminder scheduling

**W6 acceptance:** member creates "พบหมอ" → elder sees it in agenda → reminder card fires at the right time; birthday shows a greeting card with a playable voice.

---

## W7 — Notifications, activity, emergency, accessibility, offline hardening (PLAN §7 W7)

**Goal:** notification behavior matches the F9 matrix on a real device; app survives full offline use with no blank screens or stack traces.

- [ ] Implement **F9 notification matrix** exactly (elder quiet / member push; emergency = high-priority, unmutable); wire FCM topics/tokens per role (F9)
- [ ] `features/activity/` (F8): last-seen / active-today card ("คุณแม่เปิดแอปวันนี้ 08:14" + green/gray dot) with privacy toggle
- [ ] Member today-summary: check-in status, unread voice count, upcoming events, new photos (F8)
- [ ] Emergency flow end-to-end: initiate → high-priority push to all members (F1, F9)
- [ ] Accessibility pass (F10): 200% font scale, extra-large-text toggle, WCAG AA contrast, ≥64dp targets, no swipe-only paths, TTS read-aloud
- [ ] Offline hardening (PLAN §5.3): kill network mid-record / mid-upload / mid-call; verify outbox + cache + friendly errors everywhere
- [ ] Optional (Could): `speech_to_text` dictation for members (F6)
- [ ] Instrument open-rate to validate notification policy isn't too quiet (Risk #6)

**W7 acceptance:** notification behavior matches F9 on a real device; app survives full offline usage without blank screens or stack traces.

---

## W8 — Testing, hardening, pilot (PLAN §7 W8)

**Goal:** all F1–F10 acceptance criteria met; crash-free ≥99%; pilot build on Play internal track; every tested elder completes core tasks unaided.

- [ ] **Moderated usability test** with 5 elders: answer call, send voice, tap check-in — zero prompts (PLAN §8, PRD §6)
- [ ] Fix all usability findings; re-test until unaided success
- [ ] Wire `firebase_analytics` + `firebase_crashlytics` (PLAN §2)
- [ ] Track §6 metrics: D7 retention, check-in completion, voice/week, crash-free ≥99%, time-to-first-photo
- [ ] Set up crash/analytics dashboards
- [ ] Build Play **internal test track** release APK/AAB
- [ ] Final `flutter analyze` + `flutter test` + integration smoke; zero P0/P1 bugs

**W8 acceptance:** all F1–F10 criteria met; crash-free ≥99%; pilot build on Play internal track; every tested elder completes the 3 core tasks unaided.

---

## Testing detail (PLAN §8)

Run alongside implementation; a feature isn't done until its tests pass.

- [ ] Unit: repositories, outbox retry/backoff, compression, date/thai-holiday logic, call-status transitions, JSON (de)serialization — mock Firebase via `dart-generate-test-mocks` (mockito)
- [ ] Widget: `BigButton`/`VoicePlayerCard`/badges render at ≥64dp & 200% font scale; elder home empty state; check-in toggle; agenda list
- [ ] Integration: invite join → first voice send → offline queue → reconnect flush; call-signaling state machine
- [ ] Coverage: `dart-collect-coverage` → LCOV; meaningful coverage on `data/` + outbox
- [ ] Load skills before writing tests: `dart-add-unit-test`, `flutter-add-widget-test`, `flutter-add-integration-test`, `dart-collect-coverage`, `dart-generate-test-mocks`

## Manual QA / accessibility checklist (PLAN §9) — run before each milestone sign-off

- [ ] All elder controls ≥64×64dp, ≥16dp spacing; primary text ≥20sp
- [ ] App renders correctly at 200% OS font scale and with "Extra large text" on
- [ ] Every action reachable without swipe-only gestures; no hamburger for elders
- [ ] Color never the sole signal (icon + text always present); contrast ≥ WCAG AA
- [ ] th + en strings both present; language switch needs no restart
- [ ] Offline: home never blanks; queued voice/photos show "รอส่ง" and sync on reconnect
- [ ] Hard/destructive actions have 5s undo or explicit confirm
- [ ] Errors show plain-language copy (what happened + one next step), never a stack trace

---

## Definition of Done — MVP gate (PLAN §12)

- [ ] All F1–F10 acceptance criteria met; zero P0/P1 bugs
- [ ] `flutter analyze` + `flutter test` clean; integration smoke passes
- [ ] An elder completes answer-call, send-voice, and check-in **with no assistance** (usability test)
- [ ] Crash-free sessions ≥99%; §6 metrics instrumented
- [ ] th/en complete; accessibility checklist (§9) fully passing
- [ ] Pilot build on Play internal track

**MoSCoW reference (PRD §7):**
- **Must:** F1 home, F2 calls, F3 voice, F4 photos, invite/onboarding, offline queue, emergency, check-in, th/en, accessibility basics
- **Should:** F5 calendar, F6 text chat, F7 important days, F8 activity, TTS
- **Could:** extra-large text, STT dictation, voice greetings, digest reminder
- **Won't (v2):** iOS, tablets, group calls, DMs, multi-family, location, health, monetization

---

## Risk watchlist (PLAN §11) — keep visible while building

| # | Risk | Watch for | When |
| --- | --- | --- | --- |
| 1 | ZegoCloud limits/cost; SDK swap | Spike result; keep `CallService` swappable | W5 |
| 2 | Elder can't self-setup | Member-assisted setup; record in usability test | W1, W8 |
| 3 | FCM delayed → missed calls | Doze/background delivery on real device | W5, W7 |
| 4 | Photo storage cost | ≤300KB compression, 500-cap, check Storage pricing | W4 |
| 5 | Two elders / shared device | Per-account home; account switcher is out-of-MVP | W2 |
| 6 | Notifications too quiet | Instrument open-rate; tune | W7 |
| 7 | Thai TTS quality on low-end | Test 2–3 low-end Androids; bundled-audio fallback | W6–W7 |
| 8 | QR needs camera permission | Always provide 6-digit fallback | W1 |
