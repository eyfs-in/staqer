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

### 4.1 Platform Metadata Matrix

| Platform | URL | Caption Text | Thumbnail | Creator Handle |
|----------|-----|-------------|-----------|----------------|
| Instagram Reel | ✅ Always | ⚠️ Sometimes (via Open Graph) | ⚠️ Sometimes | ⚠️ Sometimes |
| TikTok Video | ✅ Always | ⚠️ Sometimes | ⚠️ Sometimes | ⚠️ Sometimes |
| YouTube Short | ✅ Always | ✅ Usually (page title) | ⚠️ Sometimes | ❌ Rarely |
| Twitter/X | ✅ Always | ✅ Usually (tweet text) | ⚠️ Sometimes | ✅ Usually |
| Pinterest Pin | ✅ Always | ✅ Usually | ✅ Usually | ⚠️ Sometimes |
| Threads (Meta) | ✅ Always | ⚠️ Sometimes | ❌ Rarely | ⚠️ Sometimes |

**Key insight:** The URL is the only guaranteed data point. Everything else varies by platform, OS version, and app version. The share extension must gracefully handle having only a URL.

### 4.2 iOS Share Sheet Data (NSItemProvider)

Each platform delivers data differently through `NSItemProvider` attached to the `NSExtensionItem` in `extensionContext.inputItems`. Here is what to expect:

| Platform | What iOS sends | NSItemProvider type identifier | Notes |
|----------|---------------|-------------------------------|-------|
| Instagram (Reels/Posts) | URL only | `public.url` | Instagram on iOS sends just a URL via `NSItemProvider`. No caption, no thumbnail, no creator handle. The URL is the canonical `instagram.com/reel/...` or `instagram.com/p/...` form. The `NSExtensionItem.attributedContentText` is typically `nil`. |
| Instagram (Stories link share) | URL only | `public.url` | Story links are ephemeral — the URL may route to a story viewer that expires after 24 hours. |
| Instagram (In-app browser) | URL of the page being viewed | `public.url` | When the user shares from Instagram's in-app browser (e.g., a link they tapped in a bio), the URL format is the destination page URL, NOT an Instagram URL. Platform detection should check the resolved URL, not assume Instagram. |
| TikTok | URL (usually shortened `vm.tiktok.com/...`) | `public.url` | TikTok almost always sends a shortened URL. Must be resolved to canonical form for dedup. Caption text occasionally included as `public.plain-text`. |
| YouTube / YouTube Shorts | URL + page title | `public.url` + `public.plain-text` | YouTube includes the video title as plain text alongside the URL. The URL is typically the full `youtube.com/watch?v=` or `youtube.com/shorts/` form. |
| Twitter/X | URL + tweet text | `public.url` + `public.plain-text` | Twitter/X often sends the tweet text as `attributedContentText` on the `NSExtensionItem`, and the tweet URL as a `public.url` item. |
| Pinterest | URL + optional description | `public.url` + `public.plain-text` | Pinterest sometimes includes pin description. Image URL not included in share payload — must be fetched via Open Graph. |
| Threads (Meta) | URL only | `public.url` | Behaves identically to Instagram's share payload. Sends a `threads.net` URL with no additional metadata. |
| Safari / Browsers | URL + page title | `public.url` + `public.plain-text` | Browsers are the most generous — they send the page URL and usually the page title as text. |

**iOS 17 vs iOS 18 share sheet behavior differences:**
- **iOS 17:** Share sheet presents extensions in a scrollable row of app icons. `NSItemProvider` `loadItem(forTypeIdentifier:)` callbacks fire on the extension's main thread. The `attributedContentText` property on `NSExtensionItem` is more reliably populated by third-party apps.
- **iOS 18:** Apple redesigned the share sheet with a new grouped layout. Extensions using the `UIActivityItemSource` protocol may receive data differently — the `activityViewController(_:itemForActivityType:)` callback may now be called lazily. Additionally, iOS 18 introduced stricter sandboxing for `NSItemProvider` loading: type identifiers must be explicitly declared in the extension's activation rules or the load will silently fail. Always test share payloads on both iOS 17 and iOS 18 devices during QA. Some apps (including Instagram) updated their share behavior in iOS 18 — thumbnail data that was occasionally available on iOS 17 may no longer be present.

### 4.3 Android Share Sheet Data (Intent)

On Android, data arrives through `Intent` extras in the `ShareReceiverActivity`. The key fields are:

| Platform | What Android sends | Intent extras used | Notes |
|----------|-------------------|-------------------|-------|
| Instagram (Reels/Posts) | URL as text | `Intent.EXTRA_TEXT` contains the URL string | Instagram on Android sends the URL via `Intent.EXTRA_TEXT`. No `EXTRA_SUBJECT`, no `EXTRA_STREAM`. The URL is typically the canonical `instagram.com/reel/...` form. |
| TikTok | URL as text (shortened) | `Intent.EXTRA_TEXT` contains `vm.tiktok.com/...` | Same shortened URL behavior as iOS. Occasionally, `EXTRA_SUBJECT` contains the video description, but this is inconsistent across TikTok app versions. |
| YouTube / YouTube Shorts | URL + title | `Intent.EXTRA_TEXT` (URL) + `Intent.EXTRA_SUBJECT` (video title) | YouTube on Android reliably sends the video title as `EXTRA_SUBJECT`. This is the best metadata source for YouTube saves. |
| Twitter/X | URL + tweet text | `Intent.EXTRA_TEXT` contains tweet text followed by the URL | Twitter/X on Android concatenates the tweet text and URL into a single `EXTRA_TEXT` string. URL must be parsed out of the text (typically the last URL in the string). |
| Pinterest | URL + description | `Intent.EXTRA_TEXT` (URL) + `Intent.EXTRA_SUBJECT` (pin title) | Pinterest on Android is relatively generous with metadata. |
| Threads (Meta) | URL as text | `Intent.EXTRA_TEXT` contains the URL string | Sends a `threads.net` URL. No additional metadata in the intent. Same minimal behavior as Instagram. |
| Chrome / Browsers | URL + page title | `Intent.EXTRA_TEXT` (URL) + `Intent.EXTRA_SUBJECT` (page title) | Browsers on Android reliably provide both URL and title. |

**Android version notes:**
- On Android 14+, the predictive back gesture may interfere with the share sheet dismiss animation if the `ShareReceiverActivity` finishes too quickly. Add a brief delay or use `finishAfterTransition()`.
- Android 13 introduced per-app language preferences — ensure the share extension UI respects `AppCompatDelegate.getApplicationLocales()` for any localized strings.

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
| Instagram Story links (ephemeral content) | Save the URL but attach a `mayExpire: true` flag and display a subtle "This link may expire" label on the card in the library. Stories expire after 24 hours. If the user opens the card after expiration, show: "This Story has expired. We saved the link, but the content is no longer available." Do not attempt to cache Story media — Instagram ToS prohibits it. |
| User shares a screenshot instead of a link | Detect `public.image` (iOS) or `image/*` MIME type (Android). Show: "Looks like an image! Would you like to save it as a photo note?" Offer optional OCR text extraction (using `VNRecognizeTextRequest` on iOS, ML Kit on Android) to pull any visible URL or text from the screenshot. If a URL is found in the OCR result, offer to save that URL instead. |
| Share from Instagram in-app browser | Instagram's in-app browser shares the destination page URL, not an Instagram URL. Platform detection should run against the actual URL domain, not assume the source app. The referrer may still indicate Instagram — do not use this for platform classification. |
| iOS share extension 60-second timeout | iOS enforces a hard 60-second time limit on share extensions via `NSExtensionRequestHandling`. If the extension does not call `completeRequest(returningItems:completionHandler:)` within 60 seconds, the system terminates it. All work must complete within this window. Never perform network requests synchronously in the extension — write to local storage and return immediately. Target completing the entire extension lifecycle in under 2 seconds. |
| Deep links vs. universal links | Some platforms send deep links (e.g., `instagram://reel/ABC123`) rather than universal links (`https://instagram.com/reel/ABC123`). Deep links cannot be opened outside the originating app and are useless for metadata extraction. If a deep link scheme is detected (no `http`/`https` prefix), attempt to convert it to the equivalent universal link. Maintain a mapping table: `instagram://reel/{id}` -> `https://instagram.com/reel/{id}`, `vnd.youtube://watch/{id}` -> `https://youtube.com/watch?v={id}`, etc. If no mapping exists, show the unsupported content error state. |
| Rate limiting: rapid saves (20+ items in 1 minute) | Do not block or throttle the user — accept all saves immediately and persist each to the local queue. However, implement a batching strategy for background processing: if more than 10 `PendingSave` records are created within 60 seconds, batch them into a single background processing job rather than spawning individual tasks for each. Display a consolidated confirmation: "Saved 12 items to Staq! Organizing..." On the backend, apply a per-user rate limit of 60 saves per minute and 500 per hour. If the limit is hit server-side, queue the excess items locally and retry with exponential backoff. |

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

**SwiftUI in share extensions (iOS 16+):** Since iOS 16, share extensions can use SwiftUI directly via a hosting approach. Instead of subclassing `SLComposeServiceViewController`, create a `ShareViewController` that subclasses `UIViewController` and hosts a SwiftUI view:

```swift
class ShareViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let contentView = ShareExtensionView(
            extensionContext: extensionContext,
            onDismiss: { [weak self] in
                self?.extensionContext?.completeRequest(returningItems: nil)
            }
        )
        let hostingController = UIHostingController(rootView: contentView)
        addChild(hostingController)
        view.addSubview(hostingController.view)
        hostingController.view.frame = view.bounds
        hostingController.view.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        hostingController.didMove(toParent: self)
    }
}
```

This gives us full access to SwiftUI animations, state management, and the modern declarative UI toolkit. The share card view, note input, and category chips can all be built with standard SwiftUI components.

**Share Extension activation rules (Info.plist):**

Basic plist-key activation (simple but limited):
```xml
<key>NSExtensionActivationSupportsWebURLWithMaxCount</key>
<integer>1</integer>
<key>NSExtensionActivationSupportsText</key>
<true/>
```

**Predicate-based activation rules (recommended):** For more granular control over what content activates the share extension, use an `NSExtensionActivationRule` predicate string instead of the simple plist keys. This allows filtering by type identifier count, combinations, and more:

```xml
<key>NSExtensionAttributes</key>
<dict>
    <key>NSExtensionActivationRule</key>
    <string>
    SUBQUERY (
        extensionItems,
        $extensionItem,
        SUBQUERY (
            $extensionItem.attachments,
            $attachment,
            ANY $attachment.registeredTypeIdentifiers UTI-CONFORMS-TO "public.url"
            || ANY $attachment.registeredTypeIdentifiers UTI-CONFORMS-TO "public.plain-text"
            || ANY $attachment.registeredTypeIdentifiers UTI-CONFORMS-TO "public.image"
        ).@count >= 1
    ).@count >= 1
    </string>
</dict>
```

This predicate activates the extension when the share payload contains at least one URL, text, or image item. The predicate approach is more flexible than the simple boolean/count keys — it supports logical operators, `UTI-CONFORMS-TO` for hierarchical type matching, and `SUBQUERY` for complex attachment inspection. Apple recommends predicates for production apps (the simple keys are primarily for development convenience). Note: if both a predicate string and simple keys are present, the predicate takes precedence.

**App Group container naming convention:**

Use the format `group.{bundle-id-prefix}.staq.shared` — for example:
```
group.com.staqapp.staq.shared
```

Both the main app target and the share extension target must include this App Group in their entitlements. The shared container is used for:
- **UserDefaults:** `UserDefaults(suiteName: "group.com.staqapp.staq.shared")` — for lightweight flags (e.g., `isFirstSave`, save count, last used categories).
- **Shared SQLite/Core Data store:** Place the persistent store file in the App Group container directory obtained via `FileManager.default.containerURL(forSecurityApplicationGroupIdentifier:)`. Both the extension and the main app read/write to the same database. Use WAL journal mode to allow concurrent reads.
- **File-based staging:** If the extension captures image data (screenshots), write to a staging directory within the App Group container. The main app picks up files from this directory on launch.

**Background processing with BGAppRefreshTask:**

The share extension writes `PendingSave` records to the shared container and exits immediately. The main app is responsible for processing the queue. Register a `BGAppRefreshTask` to process queued saves even if the user does not actively open the app:

```swift
// In main app's AppDelegate or @main App init:
BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "com.staqapp.staq.processPendingSaves",
    using: nil
) { task in
    self.handlePendingSaveProcessing(task: task as! BGAppRefreshTask)
}

// Schedule after each share extension save (called from main app on foreground):
func schedulePendingSaveProcessing() {
    let request = BGAppRefreshTaskRequest(identifier: "com.staqapp.staq.processPendingSaves")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 5 * 60) // 5 min minimum
    try? BGTaskScheduler.shared.submit(request)
}
```

This ensures that saves are processed in the background even if the user never opens the main app after sharing. The `BGAppRefreshTask` has ~30 seconds of execution time — sufficient for URL resolution and metadata extraction for a batch of queued saves. For longer processing (thumbnail download, AI categorization), use `BGProcessingTask` instead.

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

**ShareReceiverActivity as a transparent activity:**

The `ShareReceiverActivity` should be a transparent, non-visible activity that immediately presents a bottom sheet over the source app. This avoids the jarring flash of a blank activity window:

```xml
<!-- AndroidManifest.xml -->
<activity
    android:name=".ShareReceiverActivity"
    android:theme="@android:style/Theme.Translucent.NoTitleBar"
    android:excludeFromRecents="true"
    android:taskAffinity=""
    android:autoRemoveFromRecents="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/*" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```

Key attributes:
- `Theme.Translucent.NoTitleBar` makes the activity background transparent so the user sees the source app behind the Staq bottom sheet. This creates the feeling of an overlay rather than a full app switch.
- `excludeFromRecents="true"` + `autoRemoveFromRecents="true"` prevent Staq from appearing in the recents list every time the user shares.
- `taskAffinity=""` launches the share activity in its own task stack, preventing it from interfering with the main Staq app task.
- The `text/*` MIME type filter catches `text/plain` as well as `text/html` which some browsers send. The `image/*` filter allows receiving screenshot shares.

**AndroidX ShareCompat usage:**

Use `ShareCompat.IntentReader` for cleaner, safer extraction of share intent data:

```kotlin
class ShareReceiverActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val intentReader = ShareCompat.IntentReader(this)
        if (intentReader.isShareIntent) {
            val sharedText = intentReader.text?.toString()
            val subject = intentReader.subject  // Often contains the page title
            val mimeType = intentReader.type    // "text/plain", "image/*", etc.
            val streamUri = intentReader.stream  // Non-null for image shares

            val url = extractUrlFromText(sharedText)
            // ... proceed with save flow
        }

        // Present Compose bottom sheet
        setContent {
            ShareConfirmationSheet(/* ... */)
        }
    }
}
```

`ShareCompat.IntentReader` handles null checks, type coercion, and edge cases (like multi-stream intents) more gracefully than manually reading `intent.extras`.

**WorkManager with ExistingWorkPolicy.KEEP for background processing:**

Use WorkManager for processing the pending save queue. `ExistingWorkPolicy.KEEP` ensures that if a processing job is already enqueued or running, duplicate triggers (e.g., rapid successive shares) do not spawn redundant workers:

```kotlin
fun enqueuePendingSaveProcessing(context: Context) {
    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()

    val workRequest = OneTimeWorkRequestBuilder<PendingSaveWorker>()
        .setConstraints(constraints)
        .setBackoffCriteria(
            BackoffPolicy.EXPONENTIAL,
            WorkRequest.MIN_BACKOFF_MILLIS,
            TimeUnit.MILLISECONDS
        )
        .build()

    WorkManager.getInstance(context).enqueueUniqueWork(
        "process_pending_saves",
        ExistingWorkPolicy.KEEP,  // Don't duplicate if already enqueued
        workRequest
    )
}
```

`ExistingWorkPolicy.KEEP` means: if a job named `"process_pending_saves"` is already pending or running, the new request is silently dropped. This is the correct policy for a queue processor — a single worker will drain the entire queue regardless of when it was triggered. Use `ExistingWorkPolicy.REPLACE` only if the new job needs to restart with different parameters (not the case here).

### 6.3 Shared Data Model

```
PendingSave {
  id: UUID
  url: String                    // Canonical URL (after redirect resolution)
  rawUrl: String                 // Original URL as received from share sheet (pre-normalization)
  platform: Enum(instagram, tiktok, youtube, twitter, pinterest, threads, other)
  captionText: String?           // From share sheet if available
  thumbnailUrl: String?          // From share sheet if available
  creatorHandle: String?         // If available
  userNote: String?              // User's optional note
  userCategory: String?          // User's manual category pick (if any)
  status: Enum(pending, processing, complete, failed)
  createdAt: DateTime
  offlineSave: Boolean           // Was saved without connectivity
  mayExpire: Boolean             // True for ephemeral content (Stories, etc.)
  sourceType: Enum(url, image, text)  // What the user actually shared
}
```

### 6.4 URL Platform Detection

```swift
func detectPlatform(url: String) -> Platform {
    if url.contains("instagram.com") || url.contains("instagr.am") { return .instagram }
    if url.contains("threads.net") { return .threads }
    if url.contains("tiktok.com") || url.contains("vm.tiktok.com") { return .tiktok }
    if url.contains("youtube.com") || url.contains("youtu.be") || url.contains("youtube.com/shorts") { return .youtube }
    if url.contains("twitter.com") || url.contains("x.com") { return .twitter }
    if url.contains("pinterest.com") || url.contains("pin.it") { return .pinterest }
    return .other
}
```

### 6.5 URL Normalization

All incoming URLs must be normalized before persistence and duplicate detection. Normalization has three stages: tracking parameter removal, short URL resolution, and canonical form standardization.

#### Stage 1: Strip Tracking Parameters

Social platforms and ad networks append tracking parameters to URLs that do not affect content identity. Strip these before storage:

**Parameters to strip:**
```
utm_source, utm_medium, utm_campaign, utm_term, utm_content  // Google Analytics
igshid                          // Instagram share tracking
si=                             // YouTube share tracking (si param)
fbclid                          // Facebook click ID
gclid                           // Google Ads click ID
twclid                          // Twitter click ID
tt_from                         // TikTok referral tracking
ref, ref_src, ref_url           // Generic referral tracking
feature                         // YouTube feature tracking (e.g., feature=share)
_ga, _gl                        // Google Analytics cross-domain
mc_cid, mc_eid                  // Mailchimp tracking
s_cid                           // Adobe Analytics
msclkid                         // Microsoft Ads click ID
```

**Regex pattern for stripping (applied to URL query string):**

```swift
// Swift
func stripTrackingParams(from url: URL) -> URL {
    guard var components = URLComponents(url: url, resolvingAgainstBaseURL: true) else {
        return url
    }

    let trackingPatterns: Set<String> = [
        "utm_source", "utm_medium", "utm_campaign", "utm_term", "utm_content",
        "igshid", "si", "fbclid", "gclid", "twclid", "tt_from",
        "ref", "ref_src", "ref_url", "feature",
        "_ga", "_gl", "mc_cid", "mc_eid", "s_cid", "msclkid"
    ]

    components.queryItems = components.queryItems?.filter { item in
        !trackingPatterns.contains(item.name)
    }

    // Remove empty query string if all params were stripped
    if components.queryItems?.isEmpty == true {
        components.queryItems = nil
    }

    return components.url ?? url
}
```

```kotlin
// Kotlin
fun stripTrackingParams(url: String): String {
    val uri = Uri.parse(url)
    val trackingParams = setOf(
        "utm_source", "utm_medium", "utm_campaign", "utm_term", "utm_content",
        "igshid", "si", "fbclid", "gclid", "twclid", "tt_from",
        "ref", "ref_src", "ref_url", "feature",
        "_ga", "_gl", "mc_cid", "mc_eid", "s_cid", "msclkid"
    )

    val builder = uri.buildUpon().clearQuery()
    uri.queryParameterNames.forEach { param ->
        if (param !in trackingParams) {
            builder.appendQueryParameter(param, uri.getQueryParameter(param))
        }
    }
    return builder.build().toString()
}
```

#### Stage 2: Resolve Short URLs

Short URLs must be resolved to their canonical destination for both platform detection and dedup. These are resolved via HTTP HEAD requests (following redirects) server-side:

**Known short URL domains to resolve:**
```
vm.tiktok.com       // TikTok shortened share links
youtu.be            // YouTube short links
bit.ly              // Generic shortener
t.co               // Twitter/X link shortener
pin.it              // Pinterest short links
instagr.am          // Instagram legacy short domain
goo.gl              // Google shortener (legacy, still in circulation)
tinyurl.com         // Generic shortener
ow.ly               // Hootsuite shortener
buff.ly             // Buffer shortener
redd.it             // Reddit short links
```

**Resolution implementation:**
```swift
// Server-side URL resolver (Swift, using async/await)
func resolveShortURL(_ url: URL) async throws -> URL {
    let shortDomains: Set<String> = [
        "vm.tiktok.com", "youtu.be", "bit.ly", "t.co", "pin.it",
        "instagr.am", "goo.gl", "tinyurl.com", "ow.ly", "buff.ly", "redd.it"
    ]

    guard let host = url.host, shortDomains.contains(host) else {
        return url  // Not a short URL — return as-is
    }

    var request = URLRequest(url: url)
    request.httpMethod = "HEAD"
    request.timeoutInterval = 5  // Don't block for more than 5s

    let (_, response) = try await URLSession.shared.data(for: request)
    if let httpResponse = response as? HTTPURLResponse,
       let resolvedURL = httpResponse.url {
        return resolvedURL
    }
    return url  // Fallback to original if resolution fails
}
```

**Important:** Short URL resolution should happen server-side or in the main app's background processing — never in the share extension itself (network requests in the extension risk hitting the 60-second timeout and consume precious memory). The extension stores the `rawUrl` as received and the main app resolves and updates the `url` field asynchronously.

#### Stage 3: Canonical Form Standardization

After stripping tracking params and resolving short URLs, apply these normalization rules:

```
1. Lowercase the scheme and host:  HTTPS://WWW.Instagram.COM → https://www.instagram.com
2. Remove trailing slashes:        https://instagram.com/reel/ABC123/ → https://instagram.com/reel/ABC123
3. Remove www prefix:              https://www.youtube.com/watch → https://youtube.com/watch
4. Remove fragment/hash:           https://youtube.com/watch?v=X#t=30 → https://youtube.com/watch?v=X
   (Exception: keep fragments that are content-significant, e.g., SoundCloud track timestamps)
5. Sort remaining query params:    https://youtube.com/watch?v=X&list=Y → https://youtube.com/watch?list=Y&v=X
6. Remove empty query params:      https://site.com/page?foo=&bar=1 → https://site.com/page?bar=1
```

**Platform-specific canonical forms:**

| Platform | Raw URL | Canonical URL |
|----------|---------|--------------|
| YouTube | `https://youtu.be/dQw4w9WgXcQ?si=abc123` | `https://youtube.com/watch?v=dQw4w9WgXcQ` |
| YouTube Short | `https://youtube.com/shorts/dQw4w9WgXcQ?feature=share` | `https://youtube.com/shorts/dQw4w9WgXcQ` |
| TikTok | `https://vm.tiktok.com/ZMrABC123/` | `https://tiktok.com/@user/video/7123456789` (after redirect resolution) |
| Instagram | `https://www.instagram.com/reel/ABC123/?igshid=xyz` | `https://instagram.com/reel/ABC123` |
| Twitter/X | `https://x.com/user/status/123?s=20&t=abc` | `https://x.com/user/status/123` |
| Pinterest | `https://pin.it/abc123` | `https://pinterest.com/pin/123456789/` (after redirect resolution) |
| Threads | `https://www.threads.net/@user/post/ABC123?igshid=xyz` | `https://threads.net/@user/post/ABC123` |

---

## 7. Onboarding Flow for Share Extension

### 7.1 Enabling the Share Extension

**iOS:** The share extension is automatically registered with the system after the user installs Staq from the App Store. No explicit "enable" action is required. The extension appears in the share sheet the next time the user opens it from any app. However, the extension's position in the share sheet is determined by iOS based on usage frequency — a brand-new install will not be prominently placed.

**Android:** The share extension is automatically available via the intent filter declared in `AndroidManifest.xml`. After install, Staq appears as a share target the next time the user taps "Share" in any app. On Android 12+, the system share sheet uses app usage data to rank share targets — Staq may appear lower in the list until the user shares to it a few times.

### 7.2 First-Time Positioning in the Share Sheet

**iOS — helping the user find and pin Staq:**

On first install, Staq will likely not appear in the top row of the share sheet. The user may need to:
1. Scroll horizontally past the suggested contacts and frequently-used apps
2. Tap "More" at the end of the app row to see the full list
3. Find "Save to Staq" in the alphabetical list
4. Optionally, tap "Edit" (top right of the full list) and drag Staq into the favorites section, or toggle it on as a favorite

Because this is a real friction point (users who cannot find the extension will never use it), the app should proactively guide the user during onboarding.

**iOS — Suggested Actions row (iOS 16+):** On iOS 16+, if the extension declares support for specific activity types, iOS may surface it as a Suggested Action in the lower section of the share sheet. This is less visible than the top icon row but provides an additional discovery path.

### 7.3 In-App Tutorial

On first app launch (before the user has made any saves), display an interactive walkthrough that teaches the share extension:

**Step 1: The hook**
```
┌─────────────────────────────────┐
│                                 │
│  Save anything in 2 taps.      │
│                                 │
│  [Animated demo showing        │
│   Instagram → Share → Staq]    │
│                                 │
│  ┌───────────────────────────┐ │
│  │   Show me how ▸           │ │
│  └───────────────────────────┘ │
│                                 │
│  ┌───────────────────────────┐ │
│  │   I already know          │ │
│  └───────────────────────────┘ │
│                                 │
└─────────────────────────────────┘
```

**Step 2: Platform-specific instructions**

Show a short animated sequence (Lottie or recorded screen capture) demonstrating:
1. Opening Instagram (or TikTok/YouTube — use the platform the user signed up from, if known)
2. Tapping the Share button on a post
3. Finding "Save to Staq" in the share sheet
4. The Staq confirmation card appearing

**Step 3 (iOS only): Pin Staq in the share sheet**

```
┌─────────────────────────────────┐
│                                 │
│  Pro tip: Pin Staq for faster  │
│  saving.                       │
│                                 │
│  [Screenshot showing the       │
│   share sheet "More" → "Edit"  │
│   → drag Staq to favorites]   │
│                                 │
│  ┌───────────────────────────┐ │
│  │   Got it ✓                │ │
│  └───────────────────────────┘ │
│                                 │
└─────────────────────────────────┘
```

**Revisiting the tutorial:** Add a "How to save" entry in the app's Settings/Help section so users can replay the walkthrough at any time.

---

## 8. Analytics Events

### 8.1 Event Definitions

Track the following events to measure share extension adoption, reliability, and usage patterns:

| Event Name | Trigger | Key Properties |
|------------|---------|----------------|
| `share_extension_opened` | Extension UI becomes visible to the user | `platform` (source app), `os_version`, `content_type` (url/image/text), `is_first_save` (boolean) |
| `save_initiated` | User taps save / extension begins processing the share | `platform`, `has_caption` (boolean), `has_thumbnail` (boolean), `has_user_note` (boolean), `has_user_category` (boolean), `is_offline` (boolean) |
| `save_completed` | PendingSave record successfully written to local DB | `platform`, `save_duration_ms` (time from extension open to DB write), `content_type`, `is_offline` |
| `save_failed` | Save could not be persisted (DB error, memory pressure, etc.) | `platform`, `error_type` (enum: db_write_failed, memory_pressure, timeout, unknown), `error_message` |
| `note_added` | User typed and submitted a note | `note_length_chars`, `platform` |
| `category_selected` | User manually selected a category chip | `category_name`, `platform`, `chip_position` (1-indexed, which chip they tapped) |
| `duplicate_detected` | URL matched an existing save after normalization | `platform`, `original_save_age_days`, `user_action` (view_save/dismiss) |
| `offline_save_queued` | Save occurred without network connectivity | `platform`, `content_type` |
| `extension_dismissed` | User dismissed the extension without saving (tapped Cancel or swiped away) | `platform`, `time_open_ms` (how long the extension was visible before dismiss) |
| `extension_timeout` | iOS 60-second timeout triggered before completion | `platform`, `time_elapsed_ms` |
| `short_url_resolved` | A shortened URL was successfully resolved to its canonical form | `short_domain` (e.g., vm.tiktok.com), `resolution_duration_ms`, `resolved_platform` |
| `screenshot_shared` | User shared an image instead of a URL | `ocr_attempted` (boolean), `url_found_in_ocr` (boolean) |
| `onboarding_tutorial_viewed` | User viewed the in-app share extension tutorial | `step_reached` (1/2/3), `completed` (boolean) |

### 8.2 Event Properties (Common)

Every event includes these base properties:

```
{
  event_name: String,
  timestamp: ISO8601,
  user_id: String?,          // Null if not logged in
  device_id: String,         // Anonymous device ID for pre-login tracking
  app_version: String,
  os: "ios" | "android",
  os_version: String,
  extension_version: String, // Share extension build number
  session_id: String         // Unique per share extension invocation
}
```

### 8.3 Implementation Notes

- **iOS:** Share extensions have limited ability to make network requests. Write analytics events to a local file in the App Group container. The main app reads and flushes the event queue on foreground. Use a lightweight JSON-lines format (one event per line) to avoid serialization overhead.
- **Android:** Similarly, write events to a local file or Room table. Flush via WorkManager when the main app processes the pending save queue.
- **Do not use Firebase Analytics or heavyweight SDKs in the share extension.** They add memory overhead and initialization time that violates our performance budgets. Use a minimal custom event writer.

---

## 9. Performance Metrics

### 9.1 Benchmarks

The share extension must feel instantaneous. These are the target benchmarks, measured on a baseline device (iPhone 12 / Pixel 6):

| Metric | Target | Hard Limit | How to Measure |
|--------|--------|------------|----------------|
| Extension launch to UI visible | < 200ms | < 400ms | Time from `viewDidLoad` (iOS) / `onCreate` (Android) to first frame rendered. Measure with `os_signpost` (iOS) or `Trace.beginSection` (Android). |
| URL extraction from NSItemProvider / Intent | < 100ms | < 250ms | Time from `loadItem(forTypeIdentifier:)` call to completion handler firing (iOS). Time from `intent.getStringExtra()` to parsed URL (Android — effectively instant). |
| Local DB write (PendingSave insert) | < 50ms | < 100ms | Time to insert one `PendingSave` record into the shared SQLite/Room DB. Measure with instrumented DB wrapper. |
| Total time from user tap to confirmation | < 500ms | < 1000ms | End-to-end: from `extensionContext` delivery to "Saved to Staq!" visible on screen. This is the metric users feel. |
| Peak memory usage (iOS extension) | < 50MB | < 100MB | Measure with Instruments (Allocations). The hard ceiling is ~120MB (system kills the extension above this), so stay well below. |
| Extension cold launch (process spawn) | < 300ms | < 500ms | Time for the system to spawn the extension process. This is mostly out of our control but can be improved by minimizing linked frameworks and avoiding static initializers. |

### 9.2 Performance Optimization Strategies

- **Minimize linked frameworks:** Every framework linked to the share extension target increases launch time. Link only the essentials: SwiftUI, Foundation, and your shared data layer. Do NOT link networking libraries, analytics SDKs, or image processing frameworks in the extension target.
- **Lazy load everything:** The confirmation card should render with placeholder content immediately. Thumbnail loading, category chip fetching, and duplicate checking happen asynchronously after the initial render.
- **Pre-warm the shared container:** On main app launch, ensure the shared SQLite database is initialized and WAL-checkpointed. A cold extension launch that has to create the database schema adds 100–200ms.
- **Avoid `NSItemProvider.loadItem` on the main thread:** On iOS, `loadItem(forTypeIdentifier:)` is asynchronous but its completion handler defaults to the caller's queue. Dispatch to a background queue, then hop back to main for UI updates.
- **Profile on low-end devices:** Benchmark on iPhone SE (3rd gen) and Pixel 4a, not just flagship devices. The share extension performance budget is tighter on devices with less RAM and slower storage.

---

## 10. Development Plan

### 10.1 Tasks & Estimates

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
| 13 | URL normalization + tracking param stripping | Shared | 3h | — |
| 14 | Platform detection logic (shared) | Shared/KMP | 2h | — |
| 15 | BGAppRefreshTask / WorkManager integration | Both | 4h | Tasks 4, 9 |
| 16 | Analytics event logging (local writer) | Both | 3h | Tasks 2, 8 |
| 17 | In-app onboarding tutorial | Both | 4h | — |
| 18 | QA: test across Instagram/TikTok/YouTube on both OS | QA | 8h | All above |
| 19 | QA: test offline, duplicates, memory pressure | QA | 4h | All above |
| 20 | QA: performance profiling against benchmarks | QA | 4h | All above |

**Total: ~64 hours (~2 weeks with one developer per platform)**

### 10.2 Testing Matrix

| Test | Instagram | TikTok | YouTube | Twitter/X | Pinterest | Threads |
|------|-----------|--------|---------|-----------|-----------|---------|
| URL captured correctly | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Caption text extracted | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Thumbnail available | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Duplicate detection works | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Offline save works | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Tracking params stripped | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Short URLs resolved | — | ✓ | ✓ | ✓ | ✓ | — |

Test on: iOS 17, iOS 18, iOS 26, Android 13, Android 14, Android 15.

### 10.3 Definition of Done

- [ ] Share extension appears in system share sheet for both platforms
- [ ] Saves complete in < 500ms with haptic confirmation
- [ ] URL correctly extracted from all 6 supported platforms (including Threads)
- [ ] Duplicate detection prevents same URL from being saved twice
- [ ] URL normalization strips tracking params and resolves short URLs
- [ ] Offline saves queued and processed when connectivity returns
- [ ] Optional note field works without blocking save flow
- [ ] Error states handled gracefully for unsupported URLs
- [ ] Share extension memory usage < 100MB on iOS
- [ ] Save data persisted to shared container accessible by main app
- [ ] BGAppRefreshTask (iOS) / WorkManager (Android) processes pending saves in background
- [ ] Analytics events logged locally and flushed by main app
- [ ] In-app onboarding tutorial guides user to share extension
- [ ] Performance benchmarks met on baseline devices (iPhone 12 / Pixel 6)
