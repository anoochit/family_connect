# PRD — Family Connect (แอปสื่อสารครอบครัว)

| | |
|---|---|
| **Version** | 1.0 (MVP) |
| **Status** | Draft |
| **Date** | 2026-10-06 |
| **Platform** | Android phones (Flutter), iOS later |
| **Backend** | Firebase (Auth, Firestore, Storage, FCM) |
| **Video** | 3rd-party prebuilt call SDK (ZegoCloud recommended) |
| **Timeline** | Solo developer, ~2 months to MVP |
| **Languages** | Thai + English (in-app switch) |

---

## 1. Overview

### 1.1 Problem
Elderly family members and patients living at home struggle with modern chat apps: tiny text, typing required, complex navigation, cluttered notifications. Family members living elsewhere want to stay close — seeing photos, hearing voices, and reaching their elders with **one tap**.

### 1.2 Solution
A deliberately simple family communication app with a **home screen that works like a photo frame**, oversized controls, voice-first interaction, and a family-shared feed of messages, photos, and events.

### 1.3 Product principles
1. **Zero typing required** for the elderly user — voice and taps only.
2. **One tap to reach family** — calling is always the biggest element.
3. **Consumption over creation** — family members create content; elders receive it.
4. **Never overwhelm** — few notifications, big badges, no feeds to scroll.
5. **Works when the network doesn't** — offline queueing for voice and photos.

### 1.4 Non-goals (MVP)
- iOS / web / tablet layouts
- Group video calls (1-to-1 only in MVP)
- Health monitoring, medication tracking logic, wearable integration
- Voice/video calls placed to non-app phone numbers
- In-app purchases / monetization
- Content moderation tooling

---

## 2. Personas

### P1 — Yai Malee (ผู้ป่วย/ผู้สูงอายุ) — *primary user*
72, lives at home alone, children live in Bangkok. Uses Android phone for Line calls only. Presbyopia, taps slowly, avoids typing, checks phone 3–5× a day.
**Goal:** see grandchildren's photos, hear voices, call children without hunting for buttons.
**Frustration:** "the buttons are too small", "I don't know if the message went through".

### P2 — Lek (ลูกหลาน) — *content creator*
38, works in the city, visits home twice a month. Comfortable with apps, uploads photos after family events, sends voice notes while commuting.
**Goal:** stay present in parents' daily life remotely; know parent is okay.
**Frustration:** "I sent a photo but Mom never saw it", "I don't know if she's eaten / is okay today".

### P3 — Pond (หลาน) — *light user*
16, sends voice messages and photos to grandparents.
**Goal:** quick, fun way to talk to grandparents.

### P4 — Multiple elderly users
A household may have **grandpa AND grandma**, each with their own account, own home screen, own check-ins, sharing the same family group. Family members may also be elderly (e.g., a 68-year-old mother using it to talk to her children).

---

## 3. Users, roles & access

### 3.1 Family model
```
Family (familyId)
 ├── Elderly member A  (role: elder)     ← own home screen, own check-in
 ├── Elderly member B  (role: elder)
 ├── Family member X   (role: member)    ← creates content
 └── Family member Y   (role: member)
```
- One family group per household; a user can belong to **one family** in MVP.
- **Members** can: call, send voice, upload photos, add calendar events, chat, view all activity.
- **Elders** can: view home screen, play voice, answer/initiate calls, tap check-in, view calendar read-only, reply with voice.
- Any member can **invite** a new person via QR/6-digit code (role chosen at invite time).

### 3.2 Onboarding — invite code, no password
1. Family member opens **Add member → Create invite** → picks role (`elder` / `member`) → app shows a **QR code + 6-digit code** (valid 24h).
2. New user installs app → **"Join family"** → scans QR (or types 6 digits) → enters display name + optional photo → done.
3. First-time elder onboarding: 3-tap guided setup (choose display name, hear a test voice message, add a "favorite" contact to home screen). Family member can complete this on the elder's phone by handing it over once.
4. Re-entry: session persists indefinitely on device; if logged out, re-join via invite code (no password, no OTP typing burden). Device token refresh is automatic.
5. Sign-out is hidden in Settings (deliberately hard to hit by accident) and requires a confirmation dialog.

**Why no OTP:** typing a 6-digit SMS code is acceptable for members but was judged a barrier for elders; invite-code flow lets family set up the elder's phone for them.

---

## 4. Feature specifications

### F1. Home screen — the photo frame (Elder) ★ core
The app opens directly to this screen; it is also the launcher-style default tab.

| Element | Spec |
|---|---|
| Background | Full-bleed rotating photo slideshow, newest family photos first, 8s per photo, slow crossfade. Subtle dark scrim behind text for legibility. |
| "Call family" button | Single oversized circular button (~40% of screen width), green, phone icon + Thai/EN label. Tapping opens contact list (big avatars) → tap = video call; second tap = voice-only call. |
| Emergency button | Red oversized button below Call: **"เรียกครอบครัว / Call family now"** — instantly starts a call to the member marked as primary contact + sends an urgent in-app badge + push to all members. |
| Daily check-in | One pill button: **"วันนี้สบายดี / I'm okay today"**. Tapping shows a big ✓ + timestamp, resets daily. Optional second state: "ไม่สบาย / Not feeling well" (long-press → two choices). |
| Voice message banner | When a new voice arrives: a large card slides in — sender avatar, name, ▶ Play button (big). Tapping plays inline; no list navigation needed. |
| Unread badges | Large numeric badges on the nav items (Messages, Calendar). No red dots. |
| Slideshow tap | Tapping the photo shows photo meta (who uploaded, when) + "next"/"prev" arrows; auto-advance pauses 10s. |

**Empty state:** friendly illustration + "ยังไม่มีรูป — ให้ครอบครัวส่งรูปมาได้เลย" with the call button still dominant.

---

### F2. Video / voice calling ★ core
- **Provider:** prebuilt SDK UI, customized to our big-button theme. Recommended: **ZegoCloud** (free tier ~10k min/month, Flutter SDK, prebuilt call UI, Thailand/Asia presence). Alternatives evaluated: 100ms, Agora, Stream Video.
- **1-to-1 only** in MVP (multi-party in v2).
- **Incoming call screen:** full-screen with caller photo, giant green **Accept** and red **Decline** buttons, audible ringtone, works when app is backgrounded (FCM data message → SDK CallKit-equivalent on Android).
- **In-call UI:** oversized mute, speaker, end-call; camera flip; video area fills screen. No chat/reactions/effects in MVP.
- **Voice-only call option** from the contact list (data-saver for elders on weak networks).
- **Fallback:** if call fails (no internet / SDK error) → friendly screen: "การเชื่อมต่อขัดข้อง / Connection problem" + big **Retry** + **"ส่งข้อความเสียงแทน / Send a voice message instead"** escape hatch.
- Call history (last 10) shown on contact list with missed-call badges.

**Acceptance:** elder receives a call with app closed → one tap answers; call connects in <5s on 4G.

---

### F3. Voice messages (ข้อความเสียง) ★ core
- **Send (member):** push-to-talk record button in chat; waveform preview, re-record, optional caption text.
- **Send (elder):** any chat screen has a persistent large mic button; hold or tap-to-toggle recording (accessibility: tap-to-toggle default for elders, hold optional in settings).
- **Receive:** appears as a large card with **▶ Play**, duration, sender photo. Auto-download; playback with big play/pause + speed 1× only.
- **Offline:** recording is saved locally immediately and queued; card shows "รอส่ง / Waiting to send". Auto-sync on reconnect. Failed items retry 3× with visible status.
- **Limits:** max 60s, ≤5MB, stored in Firebase Storage under `/families/{familyId}/voice/{msgId}.opus|aac`.
- Playback also announced via TTS notification banner when possible ("ข้อความเสียงจากคุณลูกเล็ก").

---

### F4. Photos — auto-updating family album
- **Upload (member):** chat photo attach or dedicated **Photos → +** ; multi-select up to 10; auto-compress to ≤1600px / ~300KB; upload shows progress; caption optional.
- **Storage:** `Storage /families/{familyId}/photos/{photoId}` + Firestore doc with uploader, timestamp, caption, sha for dedupe.
- **Display:** elder home screen slideshow (see F1) shows photos sorted by recency; a "new photo" badge appears until the elder has *seen* it (view event logged).
- **Offline:** member uploads queue locally; elder's slideshow is served from a **local cache of the last 50 photos** so the home screen never renders blank (placeholder only if cache is empty).
- **V1 cap:** 500 photos per family; oldest photos evicted from *cache* (not from cloud).
- No albums/tags/likes in MVP.

---

### F5. Family calendar — read-only for elders + reminders
- **Create/edit (member):** title, date/time, optional recurrence (weekly/monthly), optional reminder lead time (15m / 1h / 1d), assignee (specific elder or whole family), optional note.
- **Elder view:** a simple **agenda list** ("วันนี้ / พรุ่งนี้ / สัปดาห์นี้") with giant day headers and colored chips — not a dense month grid by default. A month grid exists as an optional toggle showing dots only.
- **Reminders (elder):** in-app big card at the reminder time; OS notification only if reminder is ≤1 hour away (per "minimal notifications" rule) or the event is flagged as important (doctor visit).
- **Suggested event types:** พบหมอ, ญาติมาเยี่ยม, ทานยา, กิจกรรมครอบครัว — one-tap templates for members.
- **Elders cannot edit** in MVP; a "แจ้งครอบครัว / Tell family" button pre-fills a voice message if they can't attend.
- Public Thai calendar holidays (parents' day, mother's/father's day) auto-marked as read-only entries via a bundled Thai holiday dataset (local, offline-capable).

---

### F6. Text chat
- Secondary to voice, present for members who prefer typing.
- **For elders:** read-only-ish experience — messages render large (min 20sp), auto-voice via TTS button per message, and the only input offered prominently is the mic. Text field exists but is collapsed behind an "A" button.
- Standard: image + voice attachments, delivery/read ticks, swipe-reply in v2 (MVP: no reply/forward/edit), 100-message local history, infinite scroll to cloud.
- Conversation structure: **one family group chat** in MVP (no DMs yet) + a **system channel** for calendar/photo notifications.

---

### F7. Important-day alerts (วันสำคัญ)
- Family members maintain a **birthdays & special days** list per family member (birthday, วันพ่อ 6 Dec, วันแม่ 8 Aug, วันครบรอบแต่งงาน, วันเกิดหลาน...).
- **Behavior:** day-of, elder home screen shows a big greeting card ("วันเกิดคุณลุง! 🎉 สุขสันต์วันเกิด") + a one-tap **"ฟังคำอวยพร / Hear the greeting"** (pre-recorded voice from family or TTS) + **"ส่งคำขอบคุณ / Send thanks"** (voice).
- Members get an in-app badge **1 day before** ("พรุ่งนี้วันเกิดคุณแม่ — ส่งคำอวยพร") with a shortcut to record a voice greeting.
- No OS notification unless the user enables it in Settings (default off).
- Data: `familyDays: [{id, type, label, date, recurring, assignee}]`.

---

### F8. Activity status & family dashboard (member view)
- **Last seen / active today** card on the member's family screen: "คุณแม่เปิดแอปวันนี้ 08:14" + green/gray dot. Privacy: elders see this too for each other; toggled off only by explicit setting.
- **Today summary card** for members: check-in status (😊 วันนี้สบายดี / ⚠️ ยังไม่ได้เช็คอิน), unread voice count, upcoming events, new photos.
- No message *content* read receipts beyond what F6 already defines; no location tracking in MVP.

---

### F9. Notifications policy (deliberately quiet)
| Event | Elder | Member |
|---|---|---|
| Incoming call | Full-screen ringing | Full-screen / push |
| Voice message | In-app banner + badge; push only if app closed AND >2h since last open | Push |
| New photo | In-app badge only | Badge |
| Calendar reminder | Card in-app; OS notification only if ≤1h or "important" | Push (1 day before important events) |
| Important day | In-app greeting card | Badge + optional push day-before |
| Emergency call | n/a (initiator) | **Push, high priority, cannot be muted** |
| Daily 8:00 reminder to open app | Optional (default **off**) | — |

Rationale: elderly users disable apps that spam them; badges + banners keep them engaged without notification fatigue.

---

### F10. Accessibility & localization
- **Font scaling:** supports OS font scale up to 200%; app's own "Extra large text" setting in Settings scales base size ×1.3.
- **Color:** high-contrast palette (primary #1B5E20 green, danger #C62828 red, background white/very light), WCAG AA contrast, never color as sole signal.
- **Touch targets:** minimum 64×64dp for elder-facing controls, ≥16dp spacing.
- **Language:** Thai + English, switchable in Settings without restart; persisted per device. All elder-facing strings carry both translations from day one.
- **TTS read-aloud:** every text message and calendar event has a 🔊 button using `flutter_tts` (Thai + English voices).
- **No swipe-only gestures** as the sole path to any action.
- Landscape not required; app is portrait-first.

---

## 5. Technical architecture

### 5.1 Stack
| Layer | Choice |
|---|---|
| App | Flutter 3.x (Dart SDK ^3.13), Material 3 |
| State | Riverpod (or Provider if simpler — decide at kickoff) |
| Auth | Firebase Auth (anonymous + custom token via invite redemption) |
| DB | Cloud Firestore, single `families` collection tree |
| Files | Firebase Storage (photos, voice) |
| Push | FCM + high-priority data messages for call signaling |
| Calls | ZegoCloud Flutter SDK (prebuilt UI kit) |
| Local cache | `drift` for offline queue + photo cache (sqflite optional) |
| Recording | `record` package → AAC/Opus |
| TTS/STT | `flutter_tts` (read-aloud), `speech_to_text` (optional dictation for members) |
| i18n | `flutter_localizations` + ARB files (th, en) |

### 5.2 Data model (Firestore)
```
families/{familyId}
  name, inviteCodeHash, createdAt, settings{langDefault, ...}
  members/{userId}
    displayName, photoUrl, role: 'elder'|'member',
    fcmTokens[], lastSeenAt, checkIns/{yyyy-MM-dd}: {status, at},
    primaryContact: bool
  messages/{msgId}
    type: 'text'|'voice'|'photo'|'system',
    senderId, senderName, text?, mediaUrl?, durationMs?, caption?,
    createdAt, seenBy[], status: 'sending'|'sent'|'failed'|'queued'
  photos/{photoId}
    uploaderId, url, thumbUrl, caption?, sha256, createdAt, seenBy[]
  events/{eventId}
    title, startAt, endAt?, recurrence?, remindLead?, assigneeId?|'all',
    category, note?, createdBy, isHoliday
  days/{dayId}
    type: 'birthday'|'special', label, date, recurring, assigneeId, greetingVoiceUrl?
  invites/{code}
    role, createdBy, expiresAt, usedBy?
```
Security rules: read/write scoped to `request.auth.uid` ∈ `familyId.members`. Elders get write access limited to `messages`, `checkIns`, `seenBy` arrays.

### 5.3 Offline strategy
1. Firestore `persistenceEnabled` → local reads for messages/events/calendar.
2. Voice/photo uploads go through a local queue table; retried with exponential backoff; surfaced as `queued` state.
3. Photo slideshow reads from disk cache first; Firestore metadata updates trigger cache prefetch (last 50).
4. Friendly error surfaces everywhere instead of stack traces (see F2/F3).

### 5.4 Screen map
```
/onboarding: join-scan | join-code | setup-elder
/elder-home  (tab 1, default for role=elder)
/messages    (tab 2) → /chat
/calendar    (tab 3)
/settings    (tab 4 — hidden for elders unless via member login)
member-home  (family dashboard: today summary, members, media)
call: /call/incoming, /call/active, /call/result
```
Elders see a **2-tab bottom bar** (Home, Messages) or no bar at all (home + overlay cards); members see 4 tabs.

---

## 6. Success metrics (MVP, first 8 weeks with 5–10 pilot families)

| Metric | Target |
|---|---|
| Elder D7 retention | ≥ 60% |
| Elder completes first incoming call unaided | ≥ 80% of elders after setup |
| Voice messages sent per family per week | ≥ 10 |
| Home screen opened daily by elder | ≥ 4×/day average |
| Daily check-in completion | ≥ 70% of elders, ≥ 5 days/week |
| Crash-free sessions | ≥ 99% |
| Time-to-first-photo for new family | < 10 min after install |
| Task success: elder places a call | 100% in usability test (5 elders, no help) |

**Qualitative:** 5-user moderated usability test with elders before pilot; success = every elder completes (1) answer call, (2) send voice, (3) tap check-in with zero prompts.

---

## 7. Milestones (solo dev, ~8 weeks)

| Week | Deliverable |
|---|---|
| **W1** | Project scaffold, Firebase project, auth + invite flow, data model + security rules, i18n skeleton (th/en), design tokens (colors, type scale, big-button components) |
| **W2** | Elder home screen: photo slideshow + cache, call button, check-in, badges. Family member dashboard skeleton |
| **W3** | Voice messages: record/playback/offline queue + family chat (text) |
| **W4** | Photo upload flow + album + slideshow integration + seen tracking |
| **W5** | Video/voice calling via ZegoCloud: incoming/outgoing, FCM signaling, call history, fallback errors |
| **W6** | Calendar (member CRUD, elder agenda) + reminders + Thai holidays; important-days feature |
| **W7** | Notifications policy, activity/last-seen, emergency call flow, accessibility pass (font scale, contrast, TTS), offline hardening |
| **W8** | Usability testing with elders → fixes; crash analytics (Crashlytics), pilot build, Play internal track |

**Definition of Done (MVP):** all F1–F10 acceptance criteria met; zero P0/P1 bugs; usable by an elder with no assistance.

### MoSCoW summary
- **Must:** F1 home/slideshow, F2 calls, F3 voice, F4 photos, invite/onboarding, offline queue, emergency button, check-in, th/en, accessibility basics.
- **Should:** F5 calendar + reminders, F6 text chat, F7 important days, F8 activity status, TTS read-aloud.
- **Could:** extra-large text toggle, speech-to-text dictation, voice greeting recordings, digest reminder.
- **Won't (v2):** iOS, tablets, group calls, DMs, multi-family, location, health integrations, monetization.

---

## 8. Risks & open questions

| # | Risk / question | Mitigation |
|---|---|---|
| 1 | ZegoCloud free-tier limits / cost after pilot | Prototype call in W4 spike; keep Agora/100ms as fallback; abstraction layer `CallService` interface |
| 2 | Elder can't set up device themselves | Invite flow designed for member-assisted setup; record setup session in usability test |
| 3 | FCM delayed → missed incoming calls | SDK handles push via data messages with `high` priority + persistent connection; test on Android Doze |
| 4 | Photo storage costs | Client-side compression (≤300KB), 500-photo cap, check Firebase Storage pricing before pilot |
| 5 | Two elders in one household — shared device confusion | Per-account home screen; if shared device, add account switcher (out of MVP: document as known limitation) |
| 6 | Notification policy too quiet → elder misses messages | Pilot instrumentation on open-rate; tune in W7 |
| 7 | Thai TTS voice quality on low-end devices | Test on 2–3 low-end Androids in W7; fallback to bundled audio |
| 8 | Which invite redemption mechanism — QR scan needs camera permission | Always provide 6-digit code fallback |

**Open decisions for kickoff:**
- State management: Riverpod vs Provider.
- Firestore vs Realtime DB for call signaling (likely SDK-managed).
- Whether elders' text input is exposed at all (currently collapsed).

---

## 9. Appendix — Elder UI reference rules
- Primary text ≥ 20sp; call button ≥ 160dp diameter; emergency ≥ 64dp height.
- Max 4 top-level destinations for elders; max 2 actions per screen.
- Every destructive/hard action has a 5-second undo or explicit confirm.
- Error copy in plain Thai/English: what happened + one thing to do next.
- No hamburger menu for elders; no swipe-to-delete as sole path.
