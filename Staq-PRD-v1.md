# PRODUCT REQUIREMENTS DOCUMENT
## Staq — Smart Social Content Organizer

**Version:** 1.1
**Date:** February 24, 2026
**Author:** Vishal
**Status:** Draft
**Confidentiality:** Internal

---

## 1. Executive Summary

Staq is a mobile-first application that transforms how people save, organize, and retrieve content from social media platforms. Users save thousands of Instagram Reels, TikToks, and YouTube Shorts but can never find them again. Staq solves this by using on-device AI to automatically extract, categorize, and structure saved content — turning a chaotic bookmark dump into a searchable personal knowledge base.

The core innovation is a **hybrid on-device/server architecture** that processes 60–70% of saves entirely on the user's phone using Apple's Foundation Models (iOS 26), Google's Gemini Nano (Android), and lightweight rule-based classifiers — reducing per-save server costs to near zero and enabling a highly profitable freemium business model.

---

## 2. Problem Statement

### 2.1 The Digital Hoarding Problem

Social media users save an average of 50–200 posts per month across platforms. Instagram's built-in "Saved" feature and TikTok's "Favorites" offer no organization, no search, and no way to extract the actual content (recipe ingredients, travel locations, product names) from the video.

### 2.2 Content Decay & the "Save and Forget" Cycle

Saved content suffers from rapid decay in usefulness. Studies of digital bookmarking behavior show that approximately **60–70% of bookmarked items are never revisited** after being saved. The intent-to-action gap widens over time: a Reel saved today with a clear purpose ("make this for dinner") becomes an unrecognizable thumbnail two weeks later. Within 30 days, the average user cannot recall the context or content of more than half their saves. This is the "content graveyard" effect — users accumulate hundreds of saves that silently rot in a chronological list, delivering zero value.

### 2.3 User Pain Points

- **Saves become a graveyard** — users save content with intent to revisit but can never find it again among hundreds of bookmarks. Research on personal information management suggests a **re-find failure rate of 40–60%** for unstructured digital collections — users who attempt to locate a specific previously saved item fail to find it roughly half the time.
- **Content is locked in video format** — a 60-second recipe Reel contains 12 ingredients and 8 steps, but users must re-watch the entire video to extract them
- **No cross-platform organization** — saves are siloed within each platform (Instagram, TikTok, YouTube, Pinterest) with no unified view
- **Context is lost** — users forget why they saved something weeks later; the original intent ("make this for dinner", "visit this place") disappears
- **Platform algorithms bury saves** — Instagram's saved collections have no search, no tags, and no sorting beyond chronological

### 2.4 Market Validation

- Over 200 billion Reels are played daily across Facebook and Instagram (Meta, 2025)
- Instagram's "Save" button is one of the most-used engagement actions, surpassing shares on many content types
- Reddit threads and Twitter/X posts about "I can never find my saved Reels" consistently get thousands of upvotes, indicating widespread frustration
- Multiple competitors (Trott, Faves, Dewey, PickTok) have launched in 2024–2025, validating demand

---

## 3. Product Vision & Goals

### 3.1 Vision Statement

> *"Every piece of content you save should be instantly findable, beautifully organized, and actionable — without you lifting a finger."*

### 3.2 Product Goals

| Priority | Goal | Metric | Target (6 months) |
|----------|------|--------|--------------------|
| **P0** | Effortless save-to-organized pipeline | Save-to-organized time | < 3 seconds (lightweight) |
| **P0** | Accurate content categorization | Classification accuracy | > 85% correct category |
| **P0** | Cross-platform support | Platforms supported | Instagram, YouTube, TikTok |
| **P1** | User retention through utility | D30 retention | > 25% |
| **P1** | Sustainable unit economics | Server cost per user/month | < $0.05 free / < $0.15 paid |
| **P2** | Paid conversion | Free-to-paid conversion | > 5% |

---

## 4. Target Users

### 4.1 Primary Personas

#### Persona 1: The Recipe Collector ("Priya")
- Age 25–35, based in India/US, saves 5–10 recipe Reels per week
- **Frustration:** Saved 200+ recipes but can never find the "paneer one with the green sauce"
- **Need:** Searchable recipe library with extracted ingredients and steps
- **Willingness to pay:** High — if she can replace her messy Notes app screenshots

#### Persona 2: The Travel Planner ("Alex")
- Age 22–30, saves travel Reels for future trips
- **Frustration:** Saved 50 Bali Reels but they're mixed with recipes, fashion, and memes
- **Need:** Location-tagged travel board that she can pull up when planning a trip
- **Willingness to pay:** Medium — would pay when actively planning a trip

#### Persona 3: The Deal Hunter ("Mike")
- Age 20–40, saves product recommendation and Amazon-find Reels
- **Frustration:** Saw a great product Reel last month, can't find it now, doesn't remember the brand
- **Need:** Product name, price, and purchase link extracted automatically
- **Willingness to pay:** Low-medium — would use free tier, upgrade if heavily invested

#### Persona 4: The Side Hustler ("Aisha")
- Age 22–35, saves business tips, marketing strategies, and entrepreneurship advice Reels
- **Frustration:** Saves dozens of "how to grow on Instagram" and "pricing your freelance work" Reels weekly, but they all blur together. Can never find the specific growth hack or pricing framework when she actually needs it for a client pitch or content strategy session.
- **Need:** A structured library where business/marketing saves are auto-tagged by topic (social media growth, pricing, sales funnels, branding) with key takeaways extracted — not just a thumbnail wall.
- **Willingness to pay:** Medium-high — views it as a business investment. If Staq replaces her habit of screenshotting advice and losing it in Camera Roll, she will pay.

### 4.2 Gen Z Behavior Patterns

Gen Z (ages 14–27) are the heaviest consumers and savers of short-form video content. Key behavior signals relevant to Staq:

- **Short-form video is the default search engine.** A significant share of younger users turn to TikTok and Instagram before Google for discovery — restaurants, recipes, product reviews, travel tips.
- **"Save for later" is the new bookmark.** Gen Z uses the save/favorite button as a lightweight intent signal ("I want this but not right now"), creating massive personal content backlogs.
- **Low tolerance for friction.** If the save-to-retrieve loop takes more than a few taps, they abandon the tool. Staq's share extension must feel instant.
- **Platform-fluid identity.** The same user saves Reels on Instagram, TikToks on TikTok, and Shorts on YouTube. Cross-platform unification is not a nice-to-have — it is table stakes for this audience.

### 4.3 India-Specific User Behavior

India is Instagram's largest user base and Staq's primary launch market. Unique behavior patterns to design around:

- **WhatsApp sharing culture:** Indian users frequently forward Reels and TikToks via WhatsApp rather than using in-app saves. Staq must handle URLs shared from WhatsApp (pasted links, forwarded messages) as a first-class input path — not just direct shares from Instagram/TikTok.
- **Regional language content:** A large and growing share of Indian short-form video content is in Hindi, Tamil, Telugu, Marathi, Bengali, and other regional languages. On-device classification must handle multilingual captions and hashtags. OCR and text extraction should support Devanagari and other Indic scripts. Transcript extraction for regional-language audio is a Phase 2 priority.
- **Price sensitivity and data consciousness:** Indian users are highly price-sensitive (see Section 9 for India pricing) and often on limited mobile data plans. Lightweight on-device processing (no video download required for Tier 1/2) is a significant advantage. Minimize background data usage.
- **Feature phone / mid-range prevalence:** A large segment of Indian users are on mid-range Android devices (Redmi, Realme, Samsung A-series) that will fall into Tier 2 processing. The rule-based classifier must deliver a good experience on these devices.

### 4.4 Target Markets

- **Primary:** India (Instagram's largest user base, high Reels engagement, price-sensitive — optimize for lightweight/free path)
- **Secondary:** United States (higher willingness to pay, strong YouTube Shorts usage)
- **Tertiary:** Southeast Asia, Middle East, Brazil (high Instagram/TikTok penetration)

---

## 5. Competitive Landscape

### 5.1 Direct Competitors

| Feature | Trott | Faves | Dewey | PickTok | **Staq (Ours)** |
|---------|-------|-------|-------|---------|-----------------|
| AI Extraction | Yes (server) | Limited | No | No | **Yes (on-device + server)** |
| On-Device AI | No | No | No | No | **Yes — Key Differentiator** |
| Instagram | Yes | Yes | Yes | Yes | Yes |
| TikTok | Yes | No | Yes | Yes | Yes |
| YouTube | Yes | No | No | No | Yes |
| Transcript Extract | Yes (server) | No | No | No | **Yes (on-device)** |
| Free Tier | Credits (limited) | Limited saves | Yes | Free | **Generous (on-device)** |
| Cost Per Save | $0.02–0.07 | ~$0.01 | ~$0.005 | ~$0.005 | **$0.00–0.003** |
| Pricing | Credit packs | $4.99/mo | $3.99/mo | Free + ads | Free + $4.99/mo |

### 5.2 Indirect Competitors

| Product | Overlap with Staq | Where It Falls Short |
|---------|-------------------|---------------------|
| **Pocket** | General-purpose "save for later" tool, strong brand in read-it-later space | Designed for articles and web pages, not video content. No video transcription, no structured data extraction from Reels. No AI categorization. |
| **Raindrop.io** | Bookmark manager with collections, tags, and visual previews | Power-user oriented, not optimized for short-form video. No AI extraction, no recipe/travel/product cards. Requires manual tagging — opposite of Staq's zero-effort approach. |
| **Pinterest (native save)** | Users already use Pinterest boards to organize visual inspiration | Pinterest is a discovery platform, not a save-and-retrieve tool for external content. Cannot save Instagram Reels or TikToks into Pinterest. No cross-platform unification. |
| **Apple/Google Notes** | Users screenshot Reels and paste into Notes as a manual workaround | Completely manual, unstructured, and unsearchable. This is the behavior Staq replaces. |

### 5.3 Key Competitive Advantage

**On-device AI processing is Staq's unfair advantage.** Competitors run all AI processing server-side, costing $0.02–0.07 per save. This forces them into credit-based pricing or restrictive free tiers. Staq's on-device architecture processes 60–70% of saves at $0.00 server cost, enabling a genuinely generous free tier that drives viral growth while maintaining 97%+ gross margins on paid subscriptions.

**The on-device cost advantage compounds at scale.** At 100,000 free users, a server-dependent competitor would spend $20,000–70,000/month on AI processing alone. Staq spends ~$2,000. This 10–35x cost advantage means Staq can sustain a generous free tier indefinitely while competitors are forced to gate features or raise prices — a structural moat that deepens as the user base grows. It also means Staq reaches profitability earlier and with fewer paid conversions, reducing pressure on aggressive monetization that damages user experience.

---

## 6. Technical Architecture

### 6.1 Architecture Overview

Staq uses a three-tier hybrid processing architecture that maximizes on-device computation and minimizes server dependency. The processing path is selected dynamically based on device capabilities and content complexity.

### 6.2 Processing Pipeline

#### Tier 1: On-Device Flagship AI (Cost: $0.00)

For devices with native AI capabilities (iPhone 15 Pro+, Pixel 8+, Galaxy S24+, and newer flagships):

1. Share Extension captures URL + caption text + thumbnail from share sheet payload
2. Vision Framework / ML Kit OCR extracts on-screen text from thumbnail image
3. Apple Foundation Models / Gemini Nano Prompt API classifies content type and extracts structured data from caption + OCR text
4. Confidence check evaluates extraction completeness
5. If sufficient → save locally. If insufficient → escalate to server (Tier 3)

**Processing time:** < 2 seconds. **Battery impact:** Negligible. **Server cost:** $0.00.

#### Tier 2: On-Device Lightweight Classifier (Cost: $0.00)

For mid-range devices without native AI frameworks:

1. Share Extension captures URL + caption text + thumbnail
2. Native keyword/hashtag classifier identifies content category using pattern matching
3. Regex extraction pulls structured data (ingredient patterns, prices, location mentions)
4. Optional: bundled Core ML model (iOS) or TFLite model (Android) (~2–5 MB) for text classification
5. Save with category + basic metadata. Flag for server enrichment if needed.

**Processing time:** < 1 second. **Battery impact:** None. **Server cost:** $0.00.

#### Tier 3: Server-Side Deep Extraction (Cost: $0.002–0.003)

Triggered on-demand for saves requiring full video analysis:

1. Backend receives URL from client
2. Scraper API (Apify / ScrapeCreators / Bright Data) fetches video file + full metadata ($0.002–0.003)
3. Client downloads video → on-device SpeechAnalyzer or WhisperKit transcribes audio ($0.00)
4. On-device Foundation Models / Gemini Nano extracts structured data from transcript ($0.00)
5. **Fallback:** If device cannot transcribe, server uses Whisper API ($0.003–0.006) + cloud LLM ($0.001–0.002)

**Total cost:** $0.002–0.003 (on-device transcription) or $0.006–0.011 (full server processing).

### 6.3 Processing Flow Diagram

```
User shares Reel URL via Share Extension
         │
         ▼
  ON-DEVICE: Extract metadata from share payload
  (URL + caption text + thumbnail)
         │
         ▼
  ┌─── Check device capabilities ───┐
  │                                  │
  ▼                                  ▼
TIER 1 (Flagship)              TIER 2 (Mid-range)
Foundation Models /            Keyword classifier +
Gemini Nano on                 hashtag parser +
caption + OCR thumbnail        regex extraction
  │                                  │
  ▼                                  ▼
  ├── Confidence HIGH ──────────► SAVE. Done. $0.00
  │
  ├── Confidence MEDIUM ────────► Save partial data.
  │                                Show "Get Full Details" button.
  │
  └── Confidence LOW ───────────► ESCALATE TO TIER 3
                                         │
                                         ▼
                               Server scrapes video ($0.002-0.003)
                                         │
                                         ▼
                               On-device transcription + extraction
                                         │
                                         ▼
                               SAVE with full structured data
```

### 6.4 Device Capability Matrix

| Platform | AI Framework | Speech-to-Text | On-Device LLM | Supported Devices |
|----------|-------------|----------------|---------------|-------------------|
| iOS 26+ | Foundation Models | SpeechAnalyzer | ~3B param model | iPhone 15 Pro+ |
| Android (Gemini Nano) | ML Kit GenAI Prompt API | ML Kit Speech | 1.8–3.25B param | Pixel 8+, Galaxy S24+, OnePlus 13, Xiaomi 15, Honor Magic 7 |
| Older iOS / Android | None (rule-based) | N/A | N/A | All other devices |

### 6.5 Kotlin Multiplatform (KMP) Shared Logic Layer

To reduce duplication across two native codebases, Staq uses Kotlin Multiplatform (KMP) for shared business logic. KMP code compiles to native binaries on both platforms — JVM/ART on Android and native framework via Kotlin/Native on iOS — with no runtime bridging overhead.

#### What Goes Into KMP (Shared)

| Layer | Specifics | KMP Libraries |
|-------|-----------|---------------|
| **Networking** | API client for Supabase REST, scraper trigger endpoints, URL validation | **Ktor** (HTTP client with platform engines: OkHttp on Android, Darwin on iOS) |
| **Data Models** | All shared data classes: SavedItem, StructuredCard, UserProfile, Collection, SyncPayload | **kotlinx.serialization** (JSON encoding/decoding, shared across platforms) |
| **Database / Local Storage** | Local save database schema, queries, migrations | **SQLDelight** (generates type-safe Kotlin APIs from SQL, uses SQLite on both platforms) |
| **Classification Rules** | Rule-based keyword/hashtag classifier (Tier 2), category definitions, regex extraction patterns | Pure Kotlin (no platform dependencies) |
| **URL Parsing** | Platform URL detection (Instagram, TikTok, YouTube), deep link extraction, share payload normalization | Pure Kotlin with regex |
| **Caching Logic** | URL deduplication, thumbnail cache policy, TTL management for extracted data | Pure Kotlin + SQLDelight |
| **Sync Engine** | Conflict resolution, offline queue management, retry logic for cloud sync | Ktor + SQLDelight |

#### What Stays Native (Platform-Specific)

| Layer | iOS | Android |
|-------|-----|---------|
| **UI** | SwiftUI | Jetpack Compose |
| **On-Device AI (Tier 1)** | Foundation Models framework | ML Kit GenAI Prompt API / Gemini Nano |
| **Speech-to-Text** | SpeechAnalyzer | ML Kit Speech |
| **OCR** | Vision framework | ML Kit Text Recognition |
| **Share Extension** | NSExtensionContext (Swift) | IntentFilter + ShareCompat (Kotlin) |
| **Platform Integrations** | WidgetKit, Apple Reminders, Shortcuts | Glance Widgets, Google Keep, Android Intents |
| **Notifications** | UNUserNotificationCenter | Firebase Cloud Messaging / NotificationCompat |

#### KMP Integration Architecture

```
┌──────────────────────────────────┐
│         iOS App (SwiftUI)        │
│  Foundation Models, Vision, etc. │
└──────────────┬───────────────────┘
               │ imports KMP framework
               ▼
┌──────────────────────────────────┐
│     KMP Shared Module (.klib)    │
│  Ktor, SQLDelight, kotlinx.ser  │
│  Classification, URL Parsing,   │
│  Caching, Sync, Data Models     │
└──────────────┬───────────────────┘
               │ compiles to JVM
               ▼
┌──────────────────────────────────┐
│    Android App (Jetpack Compose) │
│  Gemini Nano, ML Kit, etc.      │
└──────────────────────────────────┘
```

This approach yields an estimated 30–40% shared code while preserving full native performance for UI and AI — the two layers where native implementation matters most.

### 6.6 Technology Stack

- **Frontend:** Native iOS (Swift / SwiftUI) + Native Android (Kotlin / Jetpack Compose)
- **Shared Logic (KMP):** Ktor (networking), kotlinx.serialization (data models), SQLDelight (local database), pure Kotlin (classification rules, URL parsing, caching)
- **On-Device AI (iOS):** Foundation Models framework, SpeechAnalyzer, Vision framework — direct native access, no bridging required
- **On-Device AI (Android):** ML Kit GenAI Prompt API, Gemini Nano via AICore, ML Kit Text Recognition — direct native access
- **Backend:** Supabase (PostgreSQL + Auth + Storage + Realtime) with **Supabase Edge Functions** (Deno-based serverless) for lightweight server-side processing
- **Serverless Processing (Supabase Edge Functions):** URL metadata enrichment, scraper API orchestration, webhook handling for async deep extract jobs, push notification dispatch, usage metering and quota enforcement. Edge Functions run on Deno Deploy globally, cold start < 200ms, billed per invocation — no always-on server cost.
- **Scraper APIs:** Apify ($0.0026/result) or ScrapeCreators ($0.002/credit) as primary; Bright Data as fallback
- **Speech-to-Text (server fallback):** OpenAI GPT-4o-mini-transcribe ($0.003/min) or Whisper API ($0.006/min)
- **LLM (server fallback):** Claude Haiku 4.5 ($0.002/call) or GPT-4o-mini ($0.001/call)
- **Caching:** Redis for URL deduplication (avoid reprocessing viral content)

#### Why Native Over Cross-Platform

The decision to build native apps on both platforms (rather than Flutter or React Native) is driven by Staq's core architecture:

- **Direct AI API access:** Foundation Models (iOS) and Gemini Nano / ML Kit GenAI (Android) are first-party native frameworks. Native apps access these with zero bridging overhead, faster adoption of new API features, and no dependency on third-party plugin maintainers.
- **Share Extension performance:** Share extensions are platform-native by definition. Native implementation ensures the fastest possible capture experience (< 500ms acknowledgment).
- **OS-level integration depth:** Calendar/reminder integration, background processing, notification handling, and on-device speech recognition all benefit from native implementation.
- **Trade-off acknowledged:** Two codebases increase development effort. Mitigated by using Kotlin Multiplatform (KMP) for shared business logic (networking, data models, classification rules, API clients) while keeping UI and AI layers fully native. See Section 6.5 for full KMP breakdown.

### 6.7 App Clip (iOS) / Instant App (Android) for Onboarding

To reduce the friction between "hearing about Staq" and "experiencing the first save," evaluate building an **App Clip (iOS)** and **Instant App (Android)** that provides a lightweight, no-install preview of the core save flow.

- **Use case:** User sees a friend share a Staq collection link or taps a Staq QR code / NFC tag. Instead of going to the App Store, the App Clip launches instantly (< 10 MB on iOS) and demonstrates the save-to-organize pipeline with a sample Reel.
- **iOS App Clip:** Invoked via App Clip Code, universal link, QR code, or NFC. Limited to 10 MB download. Can use Sign in with Apple for frictionless auth. Provides a "Get the Full App" prompt after the first successful demo.
- **Android Instant App:** Delivered via Google Play Instant. Accessed through a URL (no install). Same lightweight demo experience. Prompts full install after engagement.
- **Goal:** Convert curious users who encounter Staq in the wild (shared collection links, social media mentions) into active users without the App Store install wall. Measure: App Clip / Instant App → full install conversion rate.
- **Phase:** Evaluate feasibility in Phase 2. Build if onboarding funnel data shows significant drop-off at the install step.

### 6.8 Legal & Compliance

Instagram scraping operates in a legally defensible gray zone following the **Meta v. Bright Data (Jan 2024)** federal ruling, which established that scraping publicly available data while not logged in does not violate Meta's Terms of Service. Meta subsequently dropped the lawsuit entirely.

**Architectural safeguards:**
- Only scrape public content from URLs the user explicitly shares (user-initiated)
- Never store video files — process transiently, retain only extracted text/metadata + thumbnail
- Always link back to original content on the source platform
- Abstract scraper behind a service layer to swap providers if any API becomes unavailable
- Start with YouTube (official transcript APIs), add TikTok, then Instagram

---

## 7. Feature Requirements

### 7.1 MVP Features (Phase 1 — Months 1–3)

#### F1: Share Extension (P0)
Native iOS/Android share extension that appears in the system share sheet. When a user shares content from Instagram, TikTok, or YouTube, Staq captures the URL, caption text, and thumbnail.

- Accept URLs from Instagram (Reels, posts), TikTok (videos), YouTube (Shorts, videos)
- Show immediate confirmation toast: "Saved to Staq!"
- Begin on-device processing in background immediately after save

#### F2: Smart Auto-Categorization (P0)
Automatically classify saved content into categories using the three-tier processing architecture:

- **Categories:** Recipes, Travel, Products/Shopping, Fitness/Health, Beauty/Fashion, DIY/Home, Finance/Investing, Entertainment, Inspiration/Quotes, General
- Tier 1 devices: Use Foundation Models / Gemini Nano for AI-powered classification
- Tier 2 devices: Use keyword/hashtag pattern matching
- Allow users to manually override or add custom categories

#### F3: Content Library (P0)
The primary app interface — a visual, searchable library of all saved content:

- Grid view with thumbnails and category badges
- Filter by category, platform, date saved
- Full-text search across extracted captions, transcripts, and metadata
- "Tap to view original" button that deep-links back to the source platform
- Swipe actions: delete, re-categorize, add to collection

#### F4: Structured Data Cards (P0)
For extracted content, display purpose-built cards:

- **Recipe Card:** dish name, ingredients list, cooking steps, prep time, cuisine type
- **Travel Card:** location name, map pin, tips, best time to visit
- **Product Card:** product name, brand, price, purchase link (if available)
- **Fitness Card:** exercise names, sets/reps, muscle groups
- **Generic Card:** title, summary, key points

#### F5: Deep Extract — On-Demand (P0)
For saves where the lightweight path didn't capture enough detail:

- Show "Get Full Details" button on incomplete cards
- Triggers server-side video scraping + on-device transcription + AI extraction
- Free users: **10 deep extracts per month**
- Paid users: **unlimited deep extracts**
- Show progress indicator during processing (10–30 seconds)

### 7.2 Post-MVP Features (Phase 2 — Months 4–6)

#### F6: Collections & Boards (P1)
- User-created themed collections (e.g., "Bali Trip 2026", "Weeknight Dinners")
- Drag-and-drop to organize within collections
- Shareable collection links (read-only for recipients)

#### F7: Smart Search with Natural Language (P1)
- "Show me that pasta recipe with lemon" — semantic search across all extracted content
- Powered by on-device embeddings for Tier 1 devices, keyword fallback for others
- Recent search history and suggested filters

#### F8: Reminder & Action Integration (P1)
- "Remind me to cook this on Saturday" — integrates with device calendar/reminders
- "Add ingredients to shopping list" — exports recipe ingredients to Apple Reminders / Google Keep

#### F9: Grocery List from Recipes (P1)
- Combine ingredients from multiple saved recipes into a unified shopping list
- Deduplicate common ingredients, aggregate quantities
- Export or share via messaging apps

#### F14: Widget Support (P1)
Home screen widgets for quick access and ambient awareness of saved content:

- **iOS (WidgetKit):** Small widget showing the 2–3 most recent saves with thumbnails and category badges. Medium widget with a "Quick Save" shortcut that opens the share extension flow. Lock Screen widget showing save count or latest save.
- **Android (Glance Widgets):** Equivalent widget set using Jetpack Glance (Compose-based widgets). Recent saves grid, quick-save action, and category summary.
- **Goal:** Keep Staq visible on the home screen to reinforce the save habit and increase weekly active saver rate. Widgets serve as a passive reminder that the user has a growing, organized library.
- **Phase:** Design in Phase 2, ship alongside Collections (F6).

### 7.3 Growth Features (Phase 3 — Months 7–12)

#### F10: Collaborative Collections (P2)
- Shared boards where multiple users can save and organize content together
- Use case: group trip planning, shared recipe books, team inspiration boards

#### F11: Web Clipper / Browser Extension (P2)
- Extend Staq beyond mobile — save from desktop browsers
- Capture blog posts, articles, YouTube videos from Chrome/Safari

#### F12: Trending & Discover (P2)
- Anonymized, aggregated view of what content types are being saved most
- "Trending recipes this week" — editorial layer on top of user behavior

#### F13: Export & Integrations (P2)
- Export saved content to Notion, Google Sheets, Apple Notes
- API for power users to build custom workflows
- Zapier/IFTTT integration

---

## 8. User Flows

### 8.1 First-Time User Experience (FTUX)

The flow from install to first save is critical. Every extra tap before the user experiences the core value ("I saved a Reel and it was instantly organized") increases drop-off.

#### Install → First Save Flow

1. **App opens for the first time.** Splash screen with value prop: "Save Reels. Find them instantly."
2. **Onboarding carousel (3 screens, skippable):**
   - Screen 1: "Share a Reel from Instagram, TikTok, or YouTube" (shows share sheet animation)
   - Screen 2: "Staq organizes it automatically — recipes, travel, products, and more" (shows structured card)
   - Screen 3: "Search and find anything you've saved in seconds" (shows search result)
3. **Account creation (lightweight).** "Sign in with Apple" / "Sign in with Google" — single tap. Email/password as fallback. No mandatory profile setup. No username required.
4. **Share extension activation prompt.** "To save Reels, Staq needs to appear in your Share Sheet." Show a brief visual tutorial: how to find and enable the Staq extension in the iOS/Android share sheet settings. Provide a "Try it now" deep link that opens Instagram (or YouTube) so the user can immediately test sharing a Reel.
5. **Notification permission (deferred).** Do NOT request notification permission during onboarding. Wait until the user has completed their first save and seen their first organized card. Then show an in-context prompt: "Want to know when your deep extract is ready? Enable notifications." This deferred timing dramatically increases opt-in rates compared to a cold ask at launch.
6. **First save experience.** User shares their first Reel from Instagram/TikTok/YouTube. Staq processes it on-device and shows the organized card. Celebration moment: "Your first save is organized! Here's what Staq found." Highlight the structured data (recipe ingredients, travel location, etc.) to demonstrate immediate value.
7. **Empty state (if user hasn't saved yet).** If the user opens the library before saving anything, show a friendly empty state: "Your library is waiting. Share your first Reel to get started." Include a direct "Open Instagram" button.

#### Key FTUX Metrics

- Install → onboarding complete: target > 80%
- Onboarding complete → first save: target > 50% within first 24 hours
- First save → second save: target > 60% within first 7 days
- Notification opt-in rate (deferred prompt): target > 55%

### 8.2 Core Save Flow

1. User is browsing Instagram, sees an interesting Reel
2. User taps **Share → selects Staq** from share sheet
3. Staq share extension captures URL + metadata
4. Immediate toast: "Saved to Staq! Organizing..."
5. On-device processing: classify content, extract data (< 2 sec)
6. User opens Staq later → sees the save already categorized with a structured card
7. If details are incomplete, "Get Full Details" button is visible

### 8.3 Retrieval Flow

1. User opens Staq → sees library grid organized by category
2. Taps "Recipes" category filter
3. Searches "pasta lemon"
4. Sees matching recipe card with ingredients and steps
5. Taps "View Original" to re-watch the Reel, or "Add to Shopping List"

### 8.4 Deep Extract Flow

1. User views a saved Reel card with minimal information (vague caption)
2. Taps "Get Full Details"
3. Free user: checks remaining monthly quota (10 free)
4. Progress spinner: "Analyzing video... extracting details..." (10–30 sec)
5. Card updates with full structured data (ingredients, location details, product info)
6. If quota exceeded: "Upgrade to Staq Pro for unlimited detailed extractions"

### 8.5 Richness Score Decision Logic

| Score | What Was Extracted | Action |
|-------|-------------------|--------|
| **High** | Full recipe with ingredients, or clear location + details | Save as complete |
| **Medium** | Got content type and topic but missing specifics | Save with "Get Full Details" button |
| **Low** | Just URL and vague caption like "omg this" | Paid: auto-escalate. Free: save as bookmark only |

---

## 9. Business Model & Monetization

### 9.1 Pricing Tiers

| Feature | Free | Staq Pro ($4.99/month or $39.99/year) |
|---------|------|---------------------------------------|
| Saves per month | Unlimited | Unlimited |
| Auto-categorization | Yes (on-device) | Yes (on-device) |
| Search & filter | Yes | Yes + natural language search |
| Deep Extracts | 10 per month | Unlimited |
| Collections | 3 collections | Unlimited collections |
| Grocery list export | No | Yes |
| Shared collections | No | Yes |
| Export to Notion/Sheets | No | Yes |

**Annual pricing:** $39.99/year represents a 33% discount versus monthly billing ($59.88/year). Annual plans improve retention (users commit for a year) and improve LTV predictability. Promote annual as the default selection on the paywall screen.

#### India-Specific Pricing

| Tier | Monthly | Annual |
|------|---------|--------|
| **Staq Pro (India)** | $1.99/month (approx. ₹149/month) | $14.99/year (approx. ₹1,199/year) |

India pricing is approximately 60% lower than US pricing, reflecting purchasing power parity and the competitive landscape in the Indian market. Implemented via App Store / Google Play regional pricing tiers. Users are automatically shown the correct price based on their App Store / Play Store region.

#### Staq Pro Trial Flow

All new users are eligible for a **7-day free trial** of Staq Pro:

1. User hits a Pro-gated feature for the first time (e.g., 11th deep extract, 4th collection, grocery list export).
2. Soft paywall: "Try Staq Pro free for 7 days. Cancel anytime."
3. User confirms via App Store / Play Store subscription flow (requires payment method on file, standard iOS/Android trial mechanics).
4. During trial: full Pro access. Badge in app: "Pro Trial — 5 days remaining."
5. Day 5: In-app reminder: "Your trial ends in 2 days. Keep Pro?" with option to manage subscription.
6. Day 7: Trial converts to paid subscription unless cancelled. If cancelled, user reverts to free tier gracefully (collections beyond 3 become read-only, deep extract quota resets).

### 9.2 Unit Economics

| Metric | Free User | Paid User | Power User (Paid) |
|--------|-----------|-----------|-------------------|
| Saves per month | 20 | 50 | 100 |
| Lightweight saves (Tier 1/2) | 14 ($0.00) | 30 ($0.00) | 60 ($0.00) |
| Deep extracts | 6 ($0.018) | 20 ($0.06) | 40 ($0.12) |
| **Total server cost/month** | **$0.02** | **$0.06** | **$0.12** |
| Revenue/month | $0.00 | $4.99 | $4.99 |
| **Gross margin** | N/A (acquisition) | **98.8%** | **97.6%** |

### 9.3 Scaling Projections

| Milestone | Users | Monthly Server Cost | Monthly Revenue |
|-----------|-------|--------------------:|----------------:|
| Launch (Month 1) | 500 free | $10 | $0 |
| Traction (Month 3) | 5,000 free / 250 paid | $115 | $1,248 |
| Growth (Month 6) | 20,000 free / 1,000 paid | $460 | $4,990 |
| Scale (Month 12) | 100,000 free / 5,000 paid | $2,300 | $24,950 |

### 9.4 Why the Economics Work

The on-device architecture inverts the typical SaaS cost structure. Most AI-powered apps bleed money on free users because every action incurs server cost. Staq's free tier costs nearly nothing:

- **10,000 free users × $0.02/month = $200/month** (trivial)
- **Paywall feels natural:** Users see basic saves work great. When they save a Reel with a vague caption and get "Limited details — upgrade for full recipe extraction," they understand the value. Not gating a basic feature — gating expensive AI processing.
- **Paid users profitable from day one:** Even the heaviest power user costs 12 cents on a $4.99 subscription = 97.6% gross margin.

---

## 10. Non-Functional Requirements

### 10.1 Performance

- Share Extension save acknowledgment: < 500ms
- Lightweight on-device processing: < 2 seconds
- Deep extract server processing: < 30 seconds
- App cold start to library view: < 1.5 seconds
- Search results returned: < 300ms

### 10.2 Reliability

- Scraper layer abstracted behind service interface — swap providers within hours if one fails
- Graceful degradation: if on-device AI fails, fall back to rule-based classification; if server scraping fails, save URL with basic metadata only
- Offline capability: all saves stored locally, sync to cloud when connected
- URL deduplication cache to avoid reprocessing identical content

### 10.3 Privacy & Security

- On-device AI processing by default — user data never leaves the device for 60–70% of saves
- Server-side processing uses transient pipelines — no video files stored
- User data encrypted at rest and in transit
- No tracking or ad networks integrated
- Comply with GDPR, CCPA, and India's Digital Personal Data Protection Act

### 10.4 Scalability

- Stateless server architecture — horizontal scaling via containerized workers
- Async job queue (BullMQ / RabbitMQ) for deep extract processing
- CDN for thumbnail caching
- Database: Supabase PostgreSQL with read replicas at scale

---

## 11. Development Roadmap

| Phase | Timeline | Deliverables | Success Criteria |
|-------|----------|-------------|------------------|
| **Phase 0: Foundation** | Weeks 1–4 | iOS (Swift/SwiftUI) and Android (Kotlin/Compose) project setup, KMP shared module scaffold, Supabase backend + Edge Functions, share extension prototypes on both platforms, on-device AI integration | Share extension captures URLs from 3 platforms on both OS |
| **Phase 1: MVP** | Weeks 5–12 | Core save flow, on-device classification (all 3 tiers), content library UI, structured data cards, deep extract pipeline | 100 beta testers, >80% categorization accuracy, <3 sec processing |
| **Phase 2: Polish** | Weeks 13–18 | Collections, natural language search, reminder integration, grocery list, widgets, paywall implementation (incl. trial flow) | App Store launch, 1,000 users, >5% conversion |
| **Phase 3: Growth** | Weeks 19–30 | Collaborative collections, web clipper, trending/discover feed, export integrations, referral program | 10,000 users, $5K MRR, D30 retention >25% |
| **Phase 4: Scale** | Weeks 31–52 | International expansion, Android optimization, API platform, enterprise/team features, Pinterest support | 100,000 users, $25K MRR |

---

## 12. Metrics & Analytics

### 12.1 North Star Metric

**Weekly Active Savers (WAS)** — number of users who save at least one piece of content per week. This measures the core value loop: if users are regularly saving content through Staq, the product is delivering value.

### 12.2 Key Metrics Dashboard

- **Acquisition:** daily installs, share extension activation rate, onboarding completion rate
- **Engagement:** saves per user per week, library opens per user per week, search queries per user per week
- **Quality:** auto-categorization accuracy, deep extract success rate, user override rate (manual re-categorization)
- **Retention:** D1, D7, D30 retention, weekly active savers / monthly active users
- **Monetization:** free-to-paid conversion rate, trial start rate, trial-to-paid conversion rate, churn rate, LTV, ARPU
- **Infrastructure:** cost per save, cost per deep extract, scraper success rate, API latency

---

## 13. Risks & Mitigations

| Severity | Risk | Impact | Mitigation |
|----------|------|--------|------------|
| **High** | Instagram changes anti-scraping measures, breaking video extraction | Deep extract feature stops working for Instagram content | Abstracted scraper layer allows provider swap within hours. Multiple providers contracted (Apify, ScrapeCreators, Bright Data). Caption-first approach still works. |
| **High** | Apple/Google changes on-device AI APIs or deprecates frameworks | Tier 1 processing breaks | Server fallback always available. WhisperKit (open-source) as alternative to SpeechAnalyzer. Rule-based classifier works on all devices. |
| **High** | On-device AI model quality divergence between iOS and Android | Inconsistent user experience across platforms. Apple's Foundation Models (shipping with iOS 26) are expected to be production-grade, while Google's Gemini Nano Prompt API is still in alpha/early access as of early 2026. Classification accuracy and extraction quality may differ significantly — Android users could see noticeably worse results, leading to lower retention and higher negative review rates on the Play Store. | Build a comprehensive cross-platform extraction quality benchmark (500+ test Reels across all categories). Measure accuracy per-platform and per-device. Set a minimum quality bar — if Gemini Nano falls below threshold on a given content type, automatically route to Tier 2 (rule-based) or Tier 3 (server) instead of delivering poor AI results. Monitor Gemini Nano API maturity closely; maintain a "feature flag" to disable on-device AI per-platform if needed. Contribute to Google's feedback channels to improve Nano quality. |
| **Medium** | App Store rejection due to scraping-adjacent functionality | Cannot distribute via iOS App Store | App never downloads or stores videos. User-initiated URL sharing is standard behavior. Multiple similar apps (Trott, PickTok) approved and live. |
| **Medium** | Low free-to-paid conversion rate | Revenue below projections | On-device processing means free tier costs nearly nothing to support. Experiment with paywall placement, trial periods, and annual pricing. Ad-supported tier as fallback. |
| **Medium** | On-device AI quality insufficient for accurate extraction | Users get incorrect categorization, lose trust | Confidence scoring with graceful degradation. "Not sure about this one" messaging. Easy manual override. Server fallback for low-confidence saves. |
| **Medium** | User data migration when switching phones (especially cross-platform: iOS to Android or vice versa) | Users who switch devices lose their entire save library, collections, and organized data — destroying the accumulated value that drives retention and willingness to pay. | Cloud sync via Supabase ensures all saves, collections, and extracted data are backed up server-side. Cross-platform migration: user signs into Staq on new device, full library syncs down. Design data export format (JSON or similar) as a portable backup. Test the iOS-to-Android and Android-to-iOS migration flows explicitly during QA. Edge case: users with large libraries (1000+ saves) need incremental sync, not a full dump on first login. |
| **Low** | Major competitor (Instagram itself) adds native organization features | Core value prop undermined | Cross-platform unification remains valuable. Deeper extraction (recipes, products) unlikely from Instagram. First-mover data moat from user-generated collections. |

---

## 14. Success Criteria

### 14.1 MVP Launch (Month 3)

- 100+ beta users actively saving content weekly
- Auto-categorization accuracy > 80% (measured by user override rate < 20%)
- Share extension works reliably across Instagram, YouTube, TikTok
- Average processing time < 3 seconds for lightweight saves

### 14.2 Product-Market Fit (Month 6)

- D30 retention > 25%
- 40%+ of users save content at least 3 times per week
- NPS score > 40
- Organic growth (word-of-mouth installs) > 30% of new users
- Free-to-paid conversion > 5%

### 14.3 Growth (Month 12)

- 100,000 total users
- 5,000 paid subscribers
- $25,000 monthly recurring revenue
- Server infrastructure costs < 10% of revenue

---

## 15. Open Questions

1. **Brand Name:** Final selection between staqit.com, snagd.com, and keept.com pending. Domain availability confirmed but registration not yet completed.

2. **Android Gemini Nano Prompt API:** Currently in Alpha. Need to validate extraction quality for content classification use case before committing to on-device path for Android.

3. **Shared Backend Logic:** With native apps on both platforms, evaluate strategies for maximizing shared business logic — consider Kotlin Multiplatform (KMP) for shared networking, data models, and classification rules across iOS and Android, while keeping UI and on-device AI layers fully native. (See Section 6.5 for current KMP plan — validate assumptions with a prototype sprint.)

4. **Share Sheet Metadata Variability:** The metadata available from the iOS/Android share sheet when sharing from Instagram varies by OS version and app version. Need extensive testing across device/OS combinations.

5. **India Pricing:** $4.99/month may be too high for the Indian market. Consider a regional pricing tier ($1.99–$2.99/month) or annual discount ($29.99/year). (See Section 9.1 for proposed India pricing at $1.99/month / ₹149/month.)

6. **Pinterest Support:** Pinterest has a different content model (images + links vs. videos). Evaluate whether the same architecture applies or if a separate extraction pipeline is needed.

7. **Analytics SDK Choice:** Evaluate PostHog (open-source, self-hostable, privacy-friendly), Mixpanel (strong funnel and retention analysis, generous free tier), or a custom lightweight analytics layer via Supabase. Key considerations: cost at scale, privacy compliance (GDPR/DPDPA), event volume limits on free tiers, and whether self-hosting (PostHog) aligns with our privacy-first positioning.

8. **Crash Reporting & Stability Monitoring:** Evaluate Sentry (cross-platform, detailed crash reports, performance monitoring) vs. Firebase Crashlytics (free, deep Android integration, but adds Google dependency on iOS). Decision should factor in KMP compatibility — Sentry has better Kotlin Multiplatform support. Consider whether bundling Crashlytics (Android) + Sentry (iOS) is worth the fragmentation.

9. **CI/CD Pipeline:** Plan for Fastlane (build automation, code signing, App Store / Play Store deployment) + GitHub Actions (CI triggers, test runners, linting). Evaluate: shared KMP module testing in CI, separate iOS/Android build lanes, TestFlight / Firebase App Distribution for beta builds, and automated screenshot generation for App Store listings.

---

## 16. Privacy & Data Philosophy

### 16.1 Privacy as a Core Product Principle

Staq is built on the principle that **your saved content is your personal data, and it should stay on your device by default.** This is not just an engineering decision — it is a product and marketing differentiator.

Most AI-powered apps send every piece of user content to a server for processing. Staq inverts this model: 60–70% of all content processing happens entirely on the user's device. The AI reads the user's saves, classifies them, and extracts structured data — all without any data leaving the phone.

### 16.2 What This Means in Practice

- **On-device by default.** The classification engine, keyword parser, OCR, and (on flagship devices) the LLM all run locally. No server round-trip. No data in transit.
- **Server processing is opt-in and transient.** When a user taps "Get Full Details," the server fetches the video URL (not the user's data), processes it, and returns extracted text. No video files are stored server-side. Processing pipelines are transient — data is discarded after extraction.
- **No tracking. No ad networks. No data brokering.** Staq does not integrate any third-party advertising SDK. Analytics are limited to anonymized product usage metrics (feature adoption, retention) — never content-level data.
- **User data is not the product.** Staq makes money from subscriptions, not from selling or monetizing user data. This is a simple, sustainable business model that keeps incentives aligned with the user.

### 16.3 Privacy-First as a Marketing Differentiator

In a market where users are increasingly aware of data harvesting, Staq's on-device architecture enables a genuine, defensible privacy claim:

> *"Your saves stay on your device. No servers reading your content. No ads. No data selling. Just your stuff, organized."*

This messaging should be prominent in:
- App Store / Play Store listing (first or second bullet point)
- Onboarding flow (Screen 2 or 3)
- Website landing page (above the fold)
- Social media and content marketing

The privacy positioning is especially powerful in the Indian market, where data privacy concerns are rising alongside the Digital Personal Data Protection Act (DPDPA) implementation, and in the European market under GDPR.

### 16.4 Compliance Posture

- **GDPR (EU):** Minimal server-side data processing reduces GDPR surface area. Data Processing Agreements (DPAs) in place for Supabase and scraper APIs.
- **CCPA (California):** No sale of personal information. User can request data deletion (account deletion removes all server-side data; local data is under user control).
- **DPDPA (India):** On-device processing aligns with the DPDPA's data minimization principles. Server-side processing uses Indian data center regions where available (Supabase supports Mumbai region).

---

*End of Document — Staq PRD v1.1*
