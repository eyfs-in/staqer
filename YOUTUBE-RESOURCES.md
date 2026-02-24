# YouTube & Video Resources for Staqer Development

Curated learning resources organized by the key technologies in the Staqer stack.

---

## 1. iOS — Swift & SwiftUI

| Resource | Type | Description |
|----------|------|-------------|
| [100 Days of SwiftUI — Hacking with Swift](https://www.hackingwithswift.com/100/swiftui) | Free course | Comprehensive beginner-to-advanced SwiftUI curriculum with videos, tutorials, and projects |
| [Stanford CS193p](https://cs193p.sites.stanford.edu/) | Free university course | Stanford's iOS development course covering SwiftUI, MVVM, animations, multithreading |
| [Build an iOS App with SwiftUI — Swift.org](https://www.swift.org/getting-started/swiftui/) | Free tutorial | Official Apple-endorsed introductory tutorial |
| [iOS Development Masterclass 2026 — Udemy](https://www.udemy.com/course/swiftui-masterclass-course-ios-development-with-swift/) | Paid course | Covers iOS 26, Liquid Glass design system, SwiftData, and AI integration |

### iOS Share Extension (Critical for F1)

| Resource | Description |
|----------|-------------|
| [Implementing a SwiftUI ShareSheet Extension (agtlucas.com)](https://agtlucas.com/blog/implementing-a-swift-ui-sharesheet-extension/) | Apr 2025 — Step-by-step Share Extension with SwiftUI |
| [iOS Share Extension with SwiftUI and SwiftData (merrell.dev)](https://www.merrell.dev/ios-share-extension-with-swiftui-and-swiftdata/) | Covers App Groups for shared data between main app and extension |
| [Share Extension with Custom UI (Medium — Henri Bredt)](https://medium.com/@henribredtprivat/create-an-ios-share-extension-with-custom-ui-in-swift-and-swiftui-2023-6cf069dc1209) | Workarounds for UIKit-based Share Extension API with SwiftUI views |
| [Custom Share Extension Bottom Sheet (Medium — Yana Sychevska)](https://medium.com/@yanasychevska/creating-a-custom-share-extension-bottom-sheet-for-ios-inspired-by-telegram-fea214cea117) | Telegram-inspired UI with SwiftUI + SwiftData |

**Key pattern:** Replace `SLComposeServiceViewController` with `UIViewController`, embed SwiftUI via `UIHostingController`, use **App Groups** to share data.

---

## 2. Android — Kotlin & Jetpack Compose

| Resource | Type | Description |
|----------|------|-------------|
| [Android Basics with Compose — Google](https://developer.android.com/courses/android-basics-compose/course) | Free course | Official Google course, beginner-friendly |
| [Jetpack Compose for Android Developers — Google](https://developer.android.com/courses/jetpack-compose/course) | Free course | Official intermediate course for existing Android devs |
| [Philipp Lackner — Android & KMP Roadmap 2025 (YouTube)](https://www.youtube.com/@PhilippLackner) | Free video | 20-min roadmap covering Kotlin, Compose, Coroutines, Room, Ktor, MVVM/MVI |
| [Jetpack Compose Tutorials for Beginners (GetStream)](https://getstream.io/blog/compose-tutorials/) | Aggregated list | Curated books, courses, and guides for Compose |
| [Android Jetpack Compose: Comprehensive Bootcamp — Udemy](https://www.udemy.com/course/kotling-android-jetpack-compose-/) | Paid course | Firebase Firestore, Hilt/Dagger, Room DB, ViewModel, Navigation, Clean Architecture |

---

## 3. Kotlin Multiplatform (KMP) — Shared Logic Layer

### Philipp Lackner's "KMP for Beginners" Series (YouTube — Free)

Search for these on [Philipp Lackner's YouTube channel](https://www.youtube.com/@PhilippLackner):

| Video | Topic |
|-------|-------|
| Building Your First Compose Multiplatform Hello World App | Project setup and first steps |
| Expect/Actual in Kotlin Multiplatform | Platform-specific declarations (~23 min) |
| Shared Navigation with Decompose | Cross-platform navigation |
| HTTP Requests with Ktor | Networking in KMP |
| Local Preferences with DataStore | Shared local storage |
| Multi-Module Architecture in KMP | Code organization (~18 min) |

### Other KMP Resources

| Resource | Description |
|----------|-------------|
| [Official Kotlin Multiplatform Learning Resources](https://kotlinlang.org/docs/multiplatform/kmp-learning-resources.html) | 30+ tutorials organized by skill level from JetBrains and Google |
| [Google's KMP Shared Module Template (May 2025)](https://android-developers.googleblog.com/2025/05/kotlin-multiplatform-shared-module-templates.html) | Official Android Studio template for adding KMP to existing projects |
| [CommonMain.dev — Ultimate KMP Guide 2026](https://commonmain.dev/kotlin-multiplatform/) | Comprehensive guide; "push as much logic into commonMain as possible" |
| [Building Industry-Level Multiplatform Apps — PL Coding](https://pl-coding.com/kmp/) | Paid 56-hour course: chat app across Android, iOS, macOS, Windows, Linux |

---

## 4. On-Device AI

### Apple Foundation Models (iOS 26 — Critical for F2 Tier 1)

| Resource | Description |
|----------|-------------|
| [WWDC25: "Meet the Foundation Models Framework" (Apple)](https://developer.apple.com/videos/play/wwdc2025/286/) | Official session covering guided generation, streaming, tool calling, sessions |
| [Foundation Models Documentation (Apple)](https://developer.apple.com/documentation/FoundationModels) | API reference for the on-device LLM |
| [Getting Started with Foundation Models — AppCoda](https://www.appcoda.com/foundation-models/) | Build an "Ask Me Anything" app with on-device AI in SwiftUI |
| [Ultimate Guide to Foundation Models — AzamSharp](https://azamsharp.com/2025/06/18/the-ultimate-guide-to-the-foundation-models-framework.html) | Streaming, @Generable, tools, SwiftData, performance optimization |
| [Exploring the Foundation Models Framework — Create with Swift](https://www.createwithswift.com/exploring-the-foundation-models-framework/) | SystemLanguageModel, LanguageModelSession, guided generation deep dive |

### Google Gemini Nano / ML Kit GenAI (Android — Critical for F2 Tier 1)

| Resource | Description |
|----------|-------------|
| [Google I/O 2025: "Gemini Nano on Android: Building with on-device gen AI"](https://io.google/2025/explore/technical-session-14) | Official session on on-device use cases and the new GenAI APIs |
| [ML Kit GenAI APIs Overview (Google)](https://developers.google.com/ml-kit/genai) | Summarization, proofreading, rewriting, image description via Gemini Nano |
| [ML Kit GenAI Prompt API (Google)](https://developers.google.com/ml-kit/genai/prompt/android) | Custom prompts to Gemini Nano — text and image+text input |
| [Gemini Nano Android Guide 2026 (LocalAIMaster)](https://localaimaster.com/blog/gemini-nano-android-guide) | ML Kit APIs, device support, privacy features, developer integration |
| [On-device GenAI APIs with ML Kit (Android Blog, May 2025)](https://android-developers.googleblog.com/2025/05/on-device-gen-ai-apis-ml-kit-gemini-nano.html) | High-level APIs, no prompt engineering needed |

---

## 5. Backend — Supabase

| Resource | Description |
|----------|-------------|
| [Supabase Full Setup: Easy Mobile Tutorial (Appery.io, Jan 2026)](https://blog.appery.io/2026/01/supabase-full-setup-tutorial/) | YouTube tutorial + blog: building a Notes app with Supabase backend |
| [Supabase for Mobile Apps: Complete Backend Guide (Natively.dev)](https://natively.dev/articles/supabase-mobile-backend) | PostgreSQL, auth, storage, real-time, edge functions for mobile |
| [Edge Functions Quickstart (Supabase Docs)](https://supabase.com/docs/guides/functions/quickstart) | Create, test, and deploy Edge Functions with the CLI |
| [Edge Functions Architecture (Supabase Docs)](https://supabase.com/docs/guides/functions/architecture) | How Edge Functions work under the hood |
| [Getting Started with Supabase Edge Functions (Medium)](https://medium.com/@faizan.pervaz/getting-started-with-supabase-and-edge-functions-a-friendly-guide-67453932adcf) | Authenticated APIs, external API calls, real-time features |

### BullMQ + Redis (Job Queue for F5 Deep Extract)

| Resource | Description |
|----------|-------------|
| [BullMQ Official Docs](https://docs.bullmq.io/) | Fast, robust Redis-based queue with exactly-once delivery |
| [BullMQ Ultimate Guide + Tutorial (DragonflyDB, 2025)](https://www.dragonflydb.io/guides/bullmq) | Getting started guide with code examples |
| [Build a Job Queue with BullMQ and Redis (OneUptime, Jan 2026)](https://oneuptime.com/blog/post/2026-01-06-nodejs-job-queue-bullmq-redis/view) | Delayed jobs, priorities, retries, dead letter queues |
| [Scalable Job Queue with BullMQ (DEV Community, Apr 2025)](https://dev.to/hexshift/building-a-scalable-job-queue-with-bullmq-and-redis-in-nodejs-b36) | Redis Streams, retries, rate limiting, sandboxed workers |
| [Job Scheduling with BullMQ (Better Stack)](https://betterstack.com/community/guides/scaling-nodejs/bullmq-scheduled-tasks/) | Concurrent jobs, horizontal scaling, prioritization |

---

## 6. Scraping & Content Extraction (for F5 Deep Extract)

| Resource | Description |
|----------|-------------|
| [Apify YouTube Scraper](https://apify.com/streamers/youtube-scraper) | Extract video metadata, captions, comments — no API limits |
| [How to Scrape YouTube Data (Apify Blog, Dec 2025)](https://blog.apify.com/how-to-scrape-youtube/) | Step-by-step guide using Apify Store |
| [How to Scrape YouTube Using Python (Crawlee.dev, Jul 2025)](https://crawlee.dev/blog/scrape-youtube-python) | Playwright-based crawling with Crawlee for Python |
| [Ultimate Guide to YouTube Data Scraping 2026](https://use-apify.com/blog/youtube-scraper-tutorial-2026) | Video details, channel analytics, comments, shorts, captions |

---

## Recommended Learning Path

For a team building Staqer, here's a suggested order:

### iOS Developer
1. 100 Days of SwiftUI (or Stanford CS193p if time-constrained)
2. iOS Share Extension tutorials (merrell.dev + agtlucas.com)
3. WWDC25 Foundation Models session + AppCoda tutorial
4. Supabase mobile backend guide

### Android Developer
1. Android Basics with Compose (Google)
2. Philipp Lackner's Android/KMP Roadmap video
3. Google I/O 2025 Gemini Nano session + ML Kit GenAI docs
4. Supabase mobile backend guide

### KMP Developer
1. Philipp Lackner's KMP for Beginners series (all 6 videos)
2. Official Kotlin Multiplatform learning resources
3. Google's KMP Shared Module Template blog post
4. CommonMain.dev Ultimate Guide

### Backend Developer
1. Supabase Full Setup tutorial (Appery.io)
2. Edge Functions quickstart + architecture docs
3. BullMQ Ultimate Guide (DragonflyDB)
4. Apify documentation for scraper integration
