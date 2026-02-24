# F1: Share Extension — Design & Development Guide

**Priority:** P0 (MVP)  
**Phase:** 1  
**Platforms:** iOS (Swift) + Android (Kotlin)  
**Dependencies:** None (foundational feature)  
**Estimated Effort:** 2–3 weeks

---

## 1. Feature Overview

The Share Extension is the primary entry point for all content into Staq. When a user is browsing Instagram, TikTok, or YouTube and taps the native Share button, Staq appears as an option in the system share sheet. Tapping it captures the URL, any available metadata, and immediately begins on-device processing.

This is the most critical feature — if the save experience feels slow, unreliable, or clunky, users abandon the app regardless of how good the library is.

---

## 2. User Stories

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-1.1 | As a user, I want to save a Reel by sharing it to Staq so I don't have to leave Instagram | Share sheet appears within 300ms, Staq icon visible, tap-to-save completes in < 500ms |
| US-1.2 | As a user, I want confirmation that my save worked so I don't worry about losing content | Visual + haptic confirmation within 500ms of tap |
| US-1.3 | As a user, I want to add a quick note when saving so I remember why I saved it | Optional text field in share sheet (not blocking the save flow) |
| US-1.4 | As a user, I want to save from multiple platforms so all my content is in one place | Works with Instagram, TikTok, YouTube, Twitter/X, Pinterest URLs |
| US-1.5 | As a user, I want saving to work even without internet so I never miss a save | Queues save locally, processes when connected |

---

## 3. UX Design

### 3.1 Design Principle: "Faster than Instagram's own Save button"

The share extension must feel instant. The user's mental model is: tap share → tap Staq → done. Any friction (loading spinners, mandatory fields, confirmation screens) will kill adoption.

### 3.2 Share Sheet Flow

```
┌─────────────────────────────────────┐
│         iOS Share Sheet             │
│                                     │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ Copy │ │ Staq │ │ More │       │
│  │ Link │ │  ⬡   │ │ ...  │       │
│  └──────┘ └──────┘ └──────┘       │
│                                     │
│  User taps Staq icon               │
│           │                         │
│           ▼                         │
│  ┌─────────────────────────────┐   │
│  │                             │   │
│  │  ✅ Saved to Staq!         │   │
│  │                             │   │
│  │  📝 Add a note (optional)  │   │
│  │  ┌───────────────────────┐ │   │
│  │  │ Make this for dinner  │ │   │
│  │  └───────────────────────┘ │   │
│  │                             │   │
│  │  ┌─────────────────────┐   │   │
│  │  │     Done ✓          │   │   │
│  │  └─────────────────────┘   │   │
│  │                             │   │
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

### 3.3 UX States

#### State 0: First-Time Save (One-time onboarding moment)

The very first time a user shares to Staq, show a slightly richer confirmation that teaches the value:

```
┌─────────────────────────────────┐
│                                 │
│  🎉 First save!                │
│                                 │
│  [Thumbnail preview]           │
│  ✅ Saved to Staq!             │
│                                 │
│  We're organizing this for you.│
│  Open Staq to see the magic.   │
│                                 │
│  ┌───────────────────────────┐ │
│  │     Open Staq ▸           │ │
│  └───────────────────────────┘ │
│  ┌───────────────────────────┐ │
│  │     Keep Browsing         │ │
│  └───────────────────────────┘ │
│                                 │
└─────────────────────────────────┘
```

"Open Staq" deep-links to the library where they see the card being organized in real-time. This creates the "wow moment." Only shown on the first save — never again.

#### State 1: Instant Save (Default — 90% of interactions)

User taps Staq icon → compact confirmation card slides up showing "✅ Saved to Staq!" with the thumbnail preview. Card auto-dismisses after 2.5 seconds if user doesn't interact. Save is already complete and processing begins in background.

```
┌─────────────────────────────────┐
│  ┌────────┐                     │
│  │[thumb] │ ✅ Saved to Staq!  │
│  │        │ Organizing...       │
│  └────────┘                     │
│                                 │
│  📝 Add a note...    [Done ✓]  │
│                                 │
└─────────────────────────────────┘
```

**Why auto-dismiss at 2.5 seconds (not shorter):** At 1.5 seconds, usability testing shows users often miss the confirmation entirely because the share sheet animation itself takes ~0.5 seconds. At 2.5 seconds, users see it, feel reassured, and can optionally interact before it disappears. Timer pauses if user touches the card.

**Why auto-dismiss (not manual close):** Most saves are impulse actions. User is scrolling, sees something, saves it, wants to keep scrolling. Don't break the flow.

#### State 2: Save + Note (Optional interaction)

If user taps the note field before auto-dismiss, the auto-dismiss timer cancels and the card expands slightly with a single-line text input. Keyboard appears. User types a quick note ("make for dinner", "show to Mom", "Bali day 3"). Taps "Done" to close.

**Why optional and not mandatory:** Adding friction to every save would reduce save frequency by 60%+ based on behavioral patterns from bookmark apps. The note is a power-user feature.

**Note field is a ghost text placeholder** ("Add a note..."), not pre-filled text. It looks tappable but unobtrusive.

#### State 3: Save + Quick Category (Power user — progressive disclosure)

Below the note field, show 3–4 recently used category chips: `🍳 Recipe` `✈️ Travel` `🛍️ Product`. Tapping one assigns the category immediately. If user doesn't tap any, auto-categorization handles it.

**Progressive disclosure rules:**
- **0–9 saves:** Don't show category chips. User doesn't have a mental model for categories yet. Let AI categorize everything.
- **10–49 saves:** Show 3 most-used category chips. User now understands categories and may want control.
- **50+ saves:** Show 4 chips + "More..." to expand full category picker. Power users want speed and precision.

**Why progressive:** Showing categories to a brand-new user is cognitive overload. They don't know what "category" means in Staq's context. Let them experience auto-categorization first, then offer manual control once they understand the system.

#### State 4: Error State — Unsupported Content

If URL parsing fails or the shared content isn't from a supported platform:

```
┌─────────────────────────────────┐
│                                 │
│  📎 Got it!                    │
│                                 │
│  We saved the link. Full AI    │
│  organization works best with  │
│  Instagram, TikTok & YouTube.  │
│                                 │
│  ┌───────────────────────────┐ │
│  │   Save as Bookmark ✓     │ │
│  └───────────────────────────┘ │
│                                 │
│  ┌───────────────────────────┐ │
│  │   Cancel                  │ │
│  └───────────────────────────┘ │
│                                 │
└─────────────────────────────────┘
```

**Why positive framing:** "Can't save this" is a dead-end message that frustrates users. "Got it! We saved the link" acknowledges their action positively while explaining the limitation. "Save as Bookmark" stores the URL without AI extraction — still useful as a fallback. The user never feels rejected.

#### State 5: Duplicate Detection

If the exact URL was already saved (after normalizing query params, trailing slashes, www/non-www, and resolving short URLs):

```
┌─────────────────────────────────┐
│                                 │
│  📌 Already in your library!   │
│  Saved 3 days ago in "Recipes" │
│                                 │
│  ┌────────────┐ ┌────────────┐│
│  │ View Save  │ │  Dismiss   ││
│  └────────────┘ └────────────┘│
│                                 │
└─────────────────────────────────┘
```

Prevents clutter without being annoying. "View Save" deep-links into the Staq app to the existing card. Copy adjusted to "Already in your library!" (positive) instead of "Already saved!" (neutral).

**URL normalization for dedup:** Strip query parameters like `utm_source`, `igshid`, `si=` that don't affect content identity. Resolve `bit.ly`, `vm.tiktok.com`, and other shorteners to canonical URLs.

#### State 6: Offline Save

```
┌─────────────────────────────────┐
│                                 │
│  ✅ Saved!                     │
│  Will organize when you're     │
│  back online.                  │
│                                 │
└─────────────────────────────────┘
```

Save queued locally. URL + any share sheet metadata stored. Processing happens when connectivity returns. No different visual treatment from normal save except the one-line subtitle — user should feel confident the save worked regardless of network state.

#### State 7: Multiple Items Shared

Some platforms (e.g., Pinterest) allow sharing multiple items. If more than one URL is detected:

```
┌─────────────────────────────────┐
│                                 │
│  ✅ Saved 3 items to Staq!    │
│  Organizing...                 │
│                                 │
└─────────────────────────────────┘
```

Save each URL as a separate PendingSave record. Don't ask user to confirm each one individually.

### 3.4 Animation Specifications

| Transition | Duration | Easing | Description |
|-----------|----------|--------|-------------|
| Card slide up | 250ms | ease-out (iOS: `.spring`) | Confirmation card enters from bottom |
| Card auto-dismiss | 200ms | ease-in | Fades out + slides down |
| Card expand (note tapped) | 200ms | ease-in-out | Height animates to reveal note field |
| Thumbnail fade-in | 150ms | ease-out | Thumbnail loads with crossfade |
| Category chips appear | 150ms staggered (50ms each) | ease-out | Chips animate in left-to-right |

**iOS:** Use SwiftUI `.transition(.move(edge: .bottom).combined(with: .opacity))` with `.spring(response: 0.3)`.  
**Android:** Use Compose `AnimatedVisibility` with `slideInVertically() + fadeIn()`.

### 3.5 Haptic & Audio Feedback

- **iOS:** `UIImpactFeedbackGenerator` with `.medium` style on successful save. Single subtle haptic tap.
- **Android:** `HapticFeedbackConstants.CONFIRM` on successful save.
- **No sound** — user is likely in a social setting or has audio playing. Sound would be disruptive.

### 3.6 Accessibility

- **VoiceOver (iOS) / TalkBack (Android):** Announce "Saved to Staq" on successful save. Announce error messages in full.
- **Dynamic Type / Font Scaling:** Confirmation card text must scale with system font size. Test at largest accessibility size — card height should grow, not truncate.
- **Reduce Motion:** If user has "Reduce Motion" enabled, replace slide animations with instant appear/disappear (crossfade only).
- **Minimum tap targets:** All interactive elements ≥ 44×44pt (iOS) / 48×48dp (Android).
- **Color contrast:** All text must meet WCAG AA contrast ratio (4.5:1 for body text, 3:1 for large text) against the card background.

### 3.7 Share Sheet Icon & Branding

- App icon should be distinct and recognizable at share sheet icon size (60×60pt on iOS)
- Use the Staq brand color for the icon background — needs to stand out in a row of grey/blue social icons
- Action label: "Save to Staq" (not "Staq" alone — verb-first helps new users understand what it does)

---

## 4. Data Captured from Share Sheet

| Platform | URL | Caption Text | Thumbnail | Creator Handle |
|----------|-----|-------------|-----------|----------------|
| Instagram Reel | ✅ Always | ⚠️ Sometimes (via Open Graph) | ⚠️ Sometimes | ⚠️ Sometimes |
| TikTok Video | ✅ Always | ⚠️ Sometimes | ⚠️ Sometimes | ⚠️ Sometimes |
| YouTube Short | ✅ Always | ✅ Usually (page title) | ⚠️ Sometimes | ❌ Rarely |
| Twitter/X | ✅ Always | ✅ Usually (tweet text) | ⚠️ Sometimes | ✅ Usually |
| Pinterest Pin | ✅ Always | ✅ Usually | ✅ Usually | ⚠️ Sometimes |

**Key insight:** The URL is the only guaranteed data point. Everything else varies by platform, OS version, and app version. The share extension must gracefully handle having only a URL.

---

## 5. Edge Cases & Error Handling

| Scenario | Handling |
|----------|----------|
| User shares a non-URL (plain text, image) | Show "Save as Note" option or explain Staq works with links |
| URL is from unsupported platform (LinkedIn, Facebook) | Save as basic bookmark with URL, show "We'll add support for this platform soon" |
| Share sheet payload is empty | Show error: "Nothing to save. Try copying the link and pasting in Staq." |
| User shares same URL twice | Duplicate detection — show "Already saved" with link to existing save |
| User is not logged in to Staq | Allow save anyway (store locally), prompt sign-up on next app open |
| App memory/storage pressure | Share extensions have strict memory limits (iOS: ~120MB). Keep processing minimal, queue heavy work for main app. |
| Share extension crashes | Implement crash recovery — on next app open, check for incomplete saves and retry |
| URL redirects (shortened links like bit.ly) | Resolve redirects server-side to get canonical URL for dedup and platform detection |

---

## 6. Technical Implementation

### 6.1 iOS Share Extension (Swift)

```
StaqShareExtension/
├── ShareViewController.swift      // Main extension entry point
├── ShareExtensionView.swift       // SwiftUI view for the share card
├── URLParser.swift                // Extract & validate URL from share payload
├── MetadataExtractor.swift        // Pull caption, thumbnail from share data
├── LocalSaveManager.swift         // Persist to shared App Group container
└── Info.plist                     // Extension configuration
```

**Critical iOS considerations:**
- Share extension and main app communicate via **App Group** shared container (UserDefaults + shared SQLite/Core Data store)
- Extension has ~120MB memory limit — do NOT load AI models here
- Extension should ONLY capture data and persist it. All AI processing happens in the main app on next launch or via background task.
- Use `NSItemProvider` to extract URL and metadata from `extensionContext.inputItems`

**Share Extension activation rules (Info.plist):**
```xml
<key>NSExtensionActivationSupportsWebURLWithMaxCount</key>
<integer>1</integer>
<key>NSExtensionActivationSupportsText</key>
<true/>
```

### 6.2 Android Share Target (Kotlin)

```
app/src/main/
├── ShareReceiverActivity.kt       // Intent filter for shared content
├── ui/ShareConfirmationSheet.kt   // Bottom sheet UI
├── data/PendingSaveRepository.kt  // Room DB for queued saves
└── AndroidManifest.xml            // Intent filter configuration
```

**Critical Android considerations:**
- Register as share target via `<intent-filter>` with `ACTION_SEND` and `text/plain` MIME type
- Extract URL from `Intent.EXTRA_TEXT`
- Use Room database for local persistence of pending saves
- Main app processes queue on next foreground event or via WorkManager background task

### 6.3 Shared Data Model

```
PendingSave {
  id: UUID
  url: String                    // Canonical URL (after redirect resolution)
  platform: Enum(instagram, tiktok, youtube, twitter, pinterest, other)
  captionText: String?           // From share sheet if available
  thumbnailUrl: String?          // From share sheet if available
  creatorHandle: String?         // If available
  userNote: String?              // User's optional note
  userCategory: String?          // User's manual category pick (if any)
  status: Enum(pending, processing, complete, failed)
  createdAt: DateTime
  offlineSave: Boolean           // Was saved without connectivity
}
```

### 6.4 URL Platform Detection

```swift
func detectPlatform(url: String) -> Platform {
    if url.contains("instagram.com") || url.contains("instagr.am") { return .instagram }
    if url.contains("tiktok.com") || url.contains("vm.tiktok.com") { return .tiktok }
    if url.contains("youtube.com") || url.contains("youtu.be") || url.contains("youtube.com/shorts") { return .youtube }
    if url.contains("twitter.com") || url.contains("x.com") { return .twitter }
    if url.contains("pinterest.com") || url.contains("pin.it") { return .pinterest }
    return .other
}
```

---

## 7. Development Plan

### 7.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Set up App Group / shared container | iOS | 2h | — |
| 2 | Build share extension entry point + URL extraction | iOS | 4h | Task 1 |
| 3 | Build SwiftUI share confirmation card | iOS | 4h | Task 2 |
| 4 | Implement PendingSave local persistence | iOS | 3h | Task 1 |
| 5 | Add duplicate detection (URL matching) | iOS | 2h | Task 4 |
| 6 | Handle all edge cases (offline, errors, non-URL) | iOS | 4h | Tasks 2–5 |
| 7 | Register share target + intent filter | Android | 2h | — |
| 8 | Build ShareReceiverActivity + bottom sheet UI | Android | 4h | Task 7 |
| 9 | Implement Room DB for pending saves | Android | 3h | Task 7 |
| 10 | Port duplicate detection to Android | Android | 2h | Task 9 |
| 11 | Handle Android edge cases | Android | 4h | Tasks 7–10 |
| 12 | URL redirect resolution (shared utility) | Backend | 3h | — |
| 13 | Platform detection logic (shared) | Shared/KMP | 2h | — |
| 14 | QA: test across Instagram/TikTok/YouTube on both OS | QA | 8h | All above |
| 15 | QA: test offline, duplicates, memory pressure | QA | 4h | All above |

**Total: ~51 hours (~1.5 weeks with one developer per platform)**

### 7.2 Testing Matrix

| Test | Instagram | TikTok | YouTube | Twitter/X | Pinterest |
|------|-----------|--------|---------|-----------|-----------|
| URL captured correctly | ✓ | ✓ | ✓ | ✓ | ✓ |
| Caption text extracted | ✓ | ✓ | ✓ | ✓ | ✓ |
| Thumbnail available | ✓ | ✓ | ✓ | ✓ | ✓ |
| Duplicate detection works | ✓ | ✓ | ✓ | ✓ | ✓ |
| Offline save works | ✓ | ✓ | ✓ | ✓ | ✓ |

Test on: iOS 17, iOS 18, iOS 26, Android 13, Android 14, Android 15.

### 7.3 Definition of Done

- [ ] Share extension appears in system share sheet for both platforms
- [ ] Saves complete in < 500ms with haptic confirmation
- [ ] URL correctly extracted from all 5 supported platforms
- [ ] Duplicate detection prevents same URL from being saved twice
- [ ] Offline saves queued and processed when connectivity returns
- [ ] Optional note field works without blocking save flow
- [ ] Error states handled gracefully for unsupported URLs
- [ ] Share extension memory usage < 100MB on iOS
- [ ] Save data persisted to shared container accessible by main app
