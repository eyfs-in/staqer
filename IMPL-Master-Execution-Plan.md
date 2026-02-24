# Staqer Master Execution Plan
**Social Content Organizer with AI-Powered Extraction**

---

**Version:** 1.0  
**Date:** February 24, 2026  
**Status:** Ready for Execution  
**Target:** MVP Launch (Phase 1) in 16 weeks

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Team Structure & Roles](#2-team-structure--roles)
3. [Development Philosophy](#3-development-philosophy)
4. [Phase 1 Master Timeline](#4-phase-1-master-timeline)
5. [Sprint Plan (8 x 2-week sprints)](#5-sprint-plan-8-x-2-week-sprints)
6. [Critical Path Analysis](#6-critical-path-analysis)
7. [Technical Risk Register](#7-technical-risk-register)
8. [Launch Checklist](#8-launch-checklist)
9. [Post-MVP Roadmap](#9-post-mvp-roadmap)
10. [Metrics & Success Criteria](#10-metrics--success-criteria)
11. [Budget Estimate](#11-budget-estimate)
12. [Decision Log](#12-decision-log)

---

## 1. Executive Summary

**Staqer** is a native mobile application (iOS + Android) that transforms social media bookmarks into an organized, searchable knowledge base through on-device AI extraction. Users save content from Instagram, TikTok, and YouTube, and Staqer automatically categorizes and structures it—turning vague Reel captions into recipe cards with ingredients, travel locations with maps, and product recommendations with prices.

### Key Innovation
**Hybrid on-device/server architecture:** 60-70% of content is processed entirely on the user's device using Apple Foundation Models (iOS 26) and Google Gemini Nano (Android), with rule-based fallbacks. This reduces server costs to near-zero per save and enables a profitable freemium model.

### MVP Scope (Phase 1)
- **5 Core Features:** F01-F05
- **Estimated Effort:** 583 hours (~16 weeks parallel dev)
- **Team:** 1 iOS dev, 1 Android dev, 1 Backend dev, 1 KMP dev (can overlap with iOS/Android), 1 QA
- **Launch Markets:** India (primary), US (secondary)
- **Monetization:** Freemium—10 free deep extracts/month, Pro at $4.99/mo US ($1.99/mo India)

### Success Definition
- 100+ beta users with >80% categorization accuracy
- D30 retention >25%, free-to-paid conversion >5%
- Server costs <$0.05/user/month for free tier
- App Store and Play Store approval within 16 weeks

---

## 2. Team Structure & Roles

### Recommended Team Composition

| Role | Responsibilities | Overlap Areas | FTE |
|------|-----------------|---------------|-----|
| **iOS Developer** | Native Swift/SwiftUI development, Foundation Models integration, share extension, WidgetKit, Core Data, Vision framework OCR, SpeechAnalyzer transcription | KMP shared module contributions (networking, data models) | 1.0 |
| **Android Developer** | Native Kotlin/Compose development, Gemini Nano integration, share target, Glance widgets, Room database, ML Kit (Text Recognition, Speech) | KMP shared module contributions (networking, data models, classification logic) | 1.0 |
| **Backend Developer** | Supabase setup (PostgreSQL, Auth, Storage, Realtime), Edge Functions (Deno), scraper API integration (Apify/ScrapeCreators), job queue (BullMQ), Redis cache, rate limiting, quota management | API design coordination with iOS/Android teams | 1.0 |
| **KMP/Shared Module Developer** | Kotlin Multiplatform shared logic (Ktor, SQLDelight, kotlinx.serialization), URL parsing, classification rules, sync engine, data models | Can be fulfilled by iOS or Android dev (0.3 FTE overlap) | 0.5 |
| **QA Engineer** | Manual testing across platforms, test case design, regression testing, analytics validation, performance profiling on low-end devices | Cross-functional—works with all developers | 1.0 |

**Total effective FTE: 4.5** (if KMP role is absorbed by iOS/Android devs)

### Responsibility Matrix (RACI)

| Task Area | iOS Dev | Android Dev | Backend Dev | KMP Dev | QA |
|-----------|---------|-------------|-------------|---------|-----|
| Share Extension/Target | R/A | R/A | C | C | R |
| On-Device AI Integration | R/A | R/A | I | C | R |
| Classification Logic (Rule-Based) | C | C | I | R/A | R |
| KMP Shared Module | C | C | I | R/A | R |
| Backend API & Job Queue | I | I | R/A | C | R |
| Scraper Integration | I | I | R/A | I | R |
| Library UI (Native) | R/A | R/A | I | I | R |
| Structured Cards (Native) | R/A | R/A | I | C | R |
| Deep Extract Pipeline | R | R | R/A | C | R |
| Analytics & Monitoring | R | R | R | I | A |

*(R=Responsible, A=Accountable, C=Consulted, I=Informed)*

### Communication & Sync

- **Daily Standup:** 15 min async (Slack/Teams thread), sync call only when blockers
- **Sprint Planning:** Every 2 weeks, ~2 hours
- **Demo/Retrospective:** End of each sprint, 1 hour
- **Tech Sync:** Mid-sprint (weekly), 30 min—focused on integration points (iOS ↔ Android ↔ Backend)
- **Office Hours:** Backend dev holds 2x weekly 30-min slots for iOS/Android to discuss API contracts

---

## 3. Development Philosophy

### Core Principles

#### 1. **Offline-First, Local-First**
- Every feature works fully offline with zero latency
- Local SQLite/Core Data/Room as the source of truth
- Cloud sync is additive, not required
- No "No connection" errors for core functionality

#### 2. **Native-First UI, Shared Business Logic**
- **Native UI:** SwiftUI (iOS) + Jetpack Compose (Android)—no Flutter, no React Native
- **Why:** On-device AI frameworks (Foundation Models, Gemini Nano) are first-party native APIs. Direct access = zero bridging overhead, faster API adoption, no third-party plugin dependency
- **Shared Logic (KMP):** Networking (Ktor), data models (kotlinx.serialization), database (SQLDelight), classification rules, URL parsing—30-40% code reuse
- **Trade-off acknowledged:** Two native codebases increase dev effort, but the cost is justified by the performance and API access gains. KMP mitigates the duplication.

#### 3. **Progressive Enhancement: Tier 1 → Tier 2.5 → Tier 2**
- **Tier 1 (Flagship devices):** Full on-device AI classification + extraction (Foundation Models / Gemini Nano)
- **Tier 2.5 (Mid-range):** Bundled lightweight ML model (CoreML/TFLite, 2-5MB)
- **Tier 2 (All devices):** Rule-based keyword/hashtag classifier + regex extraction
- Automatic fallback chain—user never sees a "not supported on your device" error

#### 4. **Cost-Conscious by Design**
- On-device processing for 60-70% of saves = $0.00 server cost
- Server-side processing only when necessary (Deep Extract)
- URL deduplication cache—viral content processed once, reused 1000x
- Target: <$0.05/user/month server cost for free tier

#### 5. **Quality Over Speed**
- Categorization accuracy >85% at launch (>90% target at 6 months)
- Failed extracts don't count against free user quota
- User corrections feed back into model improvement pipeline
- Weekly human evaluation of 100 random extracts

---

## 4. Phase 1 Master Timeline

**Duration:** 16 weeks (4 months)  
**Start:** Sprint 1, Week 1  
**MVP Launch:** Sprint 8, Week 16  

### High-Level Gantt Chart

```
Week    1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16
       ───────────────────────────────────────────────────────────────
Sprint  │ S1  │ S2  │ S3  │ S4  │ S5  │ S6  │ S7  │ S8  │
       ───────────────────────────────────────────────────────────────
iOS    ████████████████████████████████████████████████████████████
       F1──┐ F2──┐ F3────┐ F4────┐ F5────┐ Polsh│ Bugs │ Store│
           │     │       │       │       │      │      │      │
Android ████████████████████████████████████████████████████████████
        F1──┘ F2──┘ F3────┘ F4────┘ F5────┘ Polsh│ Bugs │ Store│
                                                 │      │      │
Backend ████████████████████████████████████████████████████████████
        Setup─┐ Auth─┐ Scrpr─┐ Queue─┐ Cache│ Optim│ Load │ Prod │
              │      │       │       │      │      │ Test │ Depoy│
                                                         │      │
KMP     ████████████████████████████████████████████████████████████
        Data──┐ Rules┐ Netwk─┐ Sync──┐ Cache│ Refin│ Test │ Final│
              │      │       │       │      │      │      │      │
QA      ░░░░░░██████████████████████████████████████████████████████
        Ramp   │ F1│ F2 │ F3  │ F4  │ F5  │Regres│Stress│Final │
                                                   │  UAT │ Sign │
```

**Legend:**
- `█` = Active development
- `░` = Ramp-up / light testing
- `F1-F5` = Feature implementation windows
- Vertical bars separate sprints

### Parallel Workstream Coordination

| Week Range | iOS Focus | Android Focus | Backend Focus | KMP Focus | QA Focus |
|------------|-----------|---------------|---------------|-----------|----------|
| **1-2** | F1 Share Extension | F1 Share Target | Supabase + Auth setup | Data models + URL parsing | Setup test devices, test plan |
| **3-4** | F2 On-device AI (Foundation Models) | F2 On-device AI (Gemini Nano) | Scraper integration (Apify) | Classification rules (Tier 2) | F1 comprehensive test pass |
| **5-6** | F3 Library grid, FTS5 | F3 Library grid, FTS5 | Job queue + Redis cache | SQLDelight schema, sync engine | F2 accuracy benchmarking |
| **7-8** | F4 Recipe + Travel cards | F4 Recipe + Travel cards | Deep extract pipeline | Networking (Ktor), shared models | F3 performance profiling |
| **9-10** | F5 On-device transcription | F5 On-device transcription | Quota management + rate limit | Cache policy, retry logic | F4 card rendering tests |
| **11-12** | Polish: animations, empty states | Polish: animations, empty states | Cost optimization, monitoring | Shared utility refactoring | F5 end-to-end extract tests |
| **13-14** | Bug fixes, accessibility audit | Bug fixes, accessibility audit | Load testing, circuit breakers | Final integration cleanup | Regression suite, stress test |
| **15-16** | App Store submission prep | Play Store submission prep | Production deployment | Final KMP module release | UAT, final sign-off |

---

## 5. Sprint Plan (8 x 2-week sprints)

### Sprint 1 (Weeks 1-2): **Foundation & Scaffolding**

**Sprint Goal:** Both platforms can capture a URL from the share sheet and persist it locally. Backend is stood up with auth.

#### iOS Tasks
- [ ] Project setup: Xcode project with SwiftUI, Core Data, App Group
- [ ] Share Extension: `NSItemProvider` URL extraction from Instagram/TikTok
- [ ] Local persistence: Core Data schema for `SavedContent`
- [ ] Share Extension confirmation toast + haptic feedback

#### Android Tasks
- [ ] Project setup: Android Studio, Jetpack Compose, Room
- [ ] Share Target: `Intent.EXTRA_TEXT` URL extraction from Instagram/TikTok
- [ ] Local persistence: Room database schema for `SavedContent`
- [ ] Share target bottom sheet + Material 3 theming

#### Backend Tasks
- [ ] Supabase project provisioning (PostgreSQL, Auth, Storage)
- [ ] Database schema: `users`, `saved_content`, `user_quota`
- [ ] Auth endpoints: email/password + OAuth (Google, Apple)
- [ ] Health check endpoint + monitoring setup

#### KMP Tasks
- [ ] KMP module scaffold: `shared` module with Ktor, kotlinx.serialization
- [ ] Data model definitions: `SavedContent`, `ExtractedData`, `UserProfile`
- [ ] URL normalization utilities (strip tracking params, resolve short URLs)
- [ ] Platform detection logic (Instagram, TikTok, YouTube)

#### QA Tasks
- [ ] Device matrix setup: iOS 17/18/26, Android 13/14/15 test devices
- [ ] Share extension test plan: 50 test URLs across platforms
- [ ] Initial smoke test: share sheet appears, URL is captured

**Sprint Deliverable:** Demo saving a Reel URL via share sheet on both platforms, persisted to local DB.

---

### Sprint 2 (Weeks 3-4): **Smart Categorization (On-Device + Rule-Based)**

**Sprint Goal:** Saved content is automatically categorized using the 3-tier classification architecture.

#### iOS Tasks
- [ ] Foundation Models integration (iOS 26+, A17 Pro check)
- [ ] Tier 1: `@Generable` structured output for classification
- [ ] OCR pipeline: Vision framework on thumbnail, text cleaning
- [ ] Tier selection logic: check device capability → Tier 1 or fallback
- [ ] Category badge overlay on thumbnails

#### Android Tasks
- [ ] Gemini Nano integration (AICore availability checks)
- [ ] Tier 1: ML Kit GenAI Prompt API for classification
- [ ] OCR pipeline: ML Kit Text Recognition (Latin + Devanagari)
- [ ] Tier selection logic: Gemini Nano → Tier 2.5 → Tier 2
- [ ] Category badge rendering in Compose

#### Backend Tasks
- [ ] Scraper API integration: Apify Instagram + TikTok scrapers
- [ ] Scraper abstraction layer (ScraperProvider interface)
- [ ] Circuit breaker implementation for scraper health
- [ ] Edge Function: URL metadata enrichment endpoint

#### KMP Tasks
- [ ] Tier 2 rule-based classifier: keyword dictionaries (multilingual)
- [ ] Hashtag scoring + caption keyword matching
- [ ] TF-IDF scorer for ambiguous cases
- [ ] Regex extractors: ingredients, prices, locations
- [ ] Tier 2.5: bundled TFLite/CoreML model integration (2-5MB)

#### QA Tasks
- [ ] Classification accuracy benchmark: 300 labeled test captions
- [ ] Tier fallback testing: simulate Tier 1 unavailable → Tier 2
- [ ] OCR quality tests: multilingual text (Hindi, Arabic, Portuguese)
- [ ] Category accuracy target: >80% correct on test set

**Sprint Deliverable:** Demo a saved Reel being auto-categorized (Recipe, Travel, Product) on screen within 2 seconds.

---

### Sprint 3 (Weeks 5-6): **Content Library UI**

**Sprint Goal:** Users can browse all saves in a visual grid, filter by category, and search with FTS5.

#### iOS Tasks
- [ ] Library screen: `LazyVGrid` with 2-column uniform grid
- [ ] Staggered grid layout option (custom `Layout` protocol)
- [ ] Category filter bar: horizontally scrollable chips, single-select
- [ ] Search overlay: FTS5 query, live results with bold highlight
- [ ] Date section headers with smart grouping

#### Android Tasks
- [ ] Library screen: `LazyVerticalGrid` with 2-column uniform grid
- [ ] Staggered grid: `LazyVerticalStaggeredGrid` (native Compose)
- [ ] Category filter bar: `LazyRow` with chip single-select
- [ ] Search overlay: FTS5 query via Room, live results
- [ ] Date section headers with lazy loading

#### Backend Tasks
- [ ] BullMQ job queue setup with Redis
- [ ] Job priority: paid users > free users
- [ ] DLQ (dead letter queue) for failed jobs
- [ ] Job timeout: 60s hard limit, retry policy (2 retries, 5s/15s delay)

#### KMP Tasks
- [ ] SQLDelight schema: FTS5 virtual table, triggers for sync
- [ ] FTS5 query builder: `MATCH` syntax with prefix support
- [ ] Thumbnail cache policy: memory LRU (50MB) → disk (500MB)
- [ ] Pagination logic: 50 items per page, prefetch next 2 pages

#### QA Tasks
- [ ] Scroll performance: 60fps target on iPhone 13 / Pixel 7
- [ ] FTS5 search latency: <100ms for 1000+ saves
- [ ] Thumbnail cache hit rate: measure >95% on recent items
- [ ] Empty states: test all 5 scenarios (new user, no results, etc.)

**Sprint Deliverable:** Demo browsing 100+ saved Reels in grid, filtering by "Recipes", searching "pasta lemon" with instant results.

---

### Sprint 4 (Weeks 7-8): **Structured Data Cards**

**Sprint Goal:** Tapping a card shows a purpose-built detail view (Recipe card with ingredients, Travel card with map, etc.).

#### iOS Tasks
- [ ] CardTemplate protocol + registry pattern
- [ ] Recipe card: ingredient checklist, unit conversion toggle, serving adjuster
- [ ] Travel card: MapKit snapshot, weather widget (OpenWeatherMap)
- [ ] Product card: purchase link, affiliate tag injection, price staleness
- [ ] Generic card: "Get Full Details" button, fallback template
- [ ] Hero transition: grid thumbnail → card header

#### Android Tasks
- [ ] CardTemplate abstraction + registry pattern
- [ ] Recipe card: ingredient checklist, unit conversion, serving adjuster
- [ ] Travel card: Google Maps static snapshot, weather widget
- [ ] Product card: purchase link, affiliate tag injection, price staleness
- [ ] Generic card: "Get Full Details" button, fallback template
- [ ] SharedElementTransition: grid → card detail

#### Backend Tasks
- [ ] Server-side LLM extraction endpoint (Claude Haiku 4.5 / GPT-4o-mini)
- [ ] Extraction prompt versioning system (semantic versions, rollback)
- [ ] OpenWeatherMap integration for travel cards (60 calls/min free tier)
- [ ] Currency conversion API (exchangerate.host, 24h cache)

#### KMP Tasks
- [ ] ExtractedData JSON schema: recipe, travel, product, fitness, etc.
- [ ] Unit conversion tables: metric ↔ imperial (cooking units)
- [ ] Ingredient quantity parser + proportional scaling logic
- [ ] Dietary tag detection: keyword matcher for vegetarian/vegan/gluten-free

#### QA Tasks
- [ ] Card rendering: <100ms target from tap to first frame
- [ ] Unit conversion accuracy: validate all cooking unit conversions
- [ ] Map snapshot load: <500ms on 4G, fallback to location name
- [ ] Inline field editing: all extracted fields are user-editable

**Sprint Deliverable:** Demo opening a Recipe card with full ingredients, toggling metric/imperial, adjusting servings from 2 to 4.

---

### Sprint 5 (Weeks 9-10): **Deep Extract Pipeline**

**Sprint Goal:** Users can trigger on-demand video analysis for vague saves. Free users get 10/month, paid unlimited.

#### iOS Tasks
- [ ] SpeechAnalyzer transcription (iOS 26+)
- [ ] SFSpeechRecognizer fallback (iOS 17-25, on-device mode)
- [ ] AVFoundation: video download, audio track extraction
- [ ] Key frame extraction: 1fps, Vision OCR per frame, text deduplication
- [ ] On-device Foundation Models extraction from combined text
- [ ] Progress UI: SSE client, stage-based progress bar
- [ ] Memory management: <300MB peak, autoreleasepool

#### Android Tasks
- [ ] ML Kit Speech Recognition transcription
- [ ] MediaExtractor: video download, audio track extraction
- [ ] Key frame extraction: 1fps, ML Kit OCR per frame, deduplication
- [ ] On-device Gemini Nano extraction from combined text
- [ ] Progress UI: OkHttp SSE client, stage-based progress bar
- [ ] Memory management: <300MB peak, explicit bitmap.recycle()

#### Backend Tasks
- [ ] Deep extract API: `POST /api/v1/extract`, SSE streaming
- [ ] Server-side Whisper transcription (fallback for Tier 2 devices)
- [ ] Job queue integration: extract jobs with 60s timeout
- [ ] URL deduplication cache: Redis, 30-day TTL, viral content reuse
- [ ] Quota management: enforce 10/month for free, MAX_INT for paid
- [ ] Rate limiting: 3/min free, 10/min paid, 5 concurrent max

#### KMP Tasks
- [ ] Deep extract state machine: pending → fetching → transcribing → extracting → complete
- [ ] SSE event parser: shared logic for progress updates
- [ ] Retry logic: 2 retries, exponential backoff
- [ ] Temp file lifecycle management: delete within 60s

#### QA Tasks
- [ ] End-to-end extract: trigger → scrape → transcribe → OCR → extract → card update (<30s)
- [ ] Quota enforcement: verify free user blocked at 11th extract
- [ ] Failed extracts don't count against quota
- [ ] Background processing: close app mid-extract, verify completion
- [ ] Memory pressure: test on low-end devices (iPhone SE 2, Pixel 4a)

**Sprint Deliverable:** Demo triggering "Get Full Details" on a vague save, watching SSE progress, card morphs to Recipe with full ingredients extracted from video audio.

---

### Sprint 6 (Weeks 11-12): **Polish & Optimization**

**Sprint Goal:** App feels fast and polished. Animations, empty states, error handling, accessibility audit.

#### iOS Tasks
- [ ] Animation polish: hero transitions, card morphing, shimmer loading
- [ ] Empty states: first launch, no search results, empty categories
- [ ] Error states: scraper failed, partial results, quota exhausted
- [ ] Accessibility: VoiceOver labels, Dynamic Type, Reduce Motion
- [ ] Dark mode audit: all card accents, overlays, map snapshots

#### Android Tasks
- [ ] Animation polish: shared element transitions, skeleton screens
- [ ] Empty states: first launch, no search results, empty categories
- [ ] Error states: scraper failed, partial results, quota exhausted
- [ ] Accessibility: TalkBack labels, font scaling, Reduce Motion
- [ ] Dark mode audit: Material 3 color scheme, map dark mode

#### Backend Tasks
- [ ] Cost optimization: monitor processing path distribution (Tier 1 vs 2 vs 3)
- [ ] Cache hit rate tuning: measure viral content reuse, adjust TTL
- [ ] Circuit breaker tuning: failure thresholds, recovery times
- [ ] Monitoring dashboard: Grafana + Prometheus (scraper health, job queue depth, LLM latency)
- [ ] Alert rules: DLQ >50 jobs/hour, all scrapers degraded, LLM error rate >5%

#### KMP Tasks
- [ ] Shared utility refactoring: consolidate duplicate logic
- [ ] Cache policy refinement: eviction rules, prefetch strategy
- [ ] Network retry logic: exponential backoff, jitter
- [ ] Logging framework: structured logs for debugging

#### QA Tasks
- [ ] Regression suite: execute full test plan on all features
- [ ] Performance profiling: Instruments (iOS) / Profiler (Android) on low-end devices
- [ ] Stress test: 1000+ saves, rapid scrolling, batch extract 10 items
- [ ] Accessibility audit: full VoiceOver/TalkBack navigation

**Sprint Deliverable:** Demo polished app with smooth animations, graceful error handling, and full accessibility support.

---

### Sprint 7 (Weeks 13-14): **Bug Bash & Hardening**

**Sprint Goal:** Fix all P0/P1 bugs. Stabilize for production launch.

#### iOS Tasks
- [ ] Bug triage: prioritize P0 (blocks launch) → P1 (degrades UX) → P2 (nice-to-fix)
- [ ] Crash fixes: Sentry/Crashlytics top crashes
- [ ] Memory leaks: Instruments Leaks tool, fix retain cycles
- [ ] Edge case handling: offline mode, low battery, low storage
- [ ] TestFlight beta: 50 external testers, collect feedback

#### Android Tasks
- [ ] Bug triage: same prioritization as iOS
- [ ] Crash fixes: Sentry/Crashlytics top crashes
- [ ] Memory leaks: LeakCanary, fix bitmap leaks
- [ ] Edge case handling: offline mode, low battery, low storage
- [ ] Firebase App Distribution beta: 50 external testers, feedback

#### Backend Tasks
- [ ] Load testing: simulate 1000 concurrent users, 100 extracts/min
- [ ] Database optimization: index tuning, query performance
- [ ] Security audit: SSRF prevention, rate limiting, quota enforcement
- [ ] Backup strategy: automated daily DB backups to S3
- [ ] Disaster recovery plan: restore from backup, failover procedures

#### KMP Tasks
- [ ] Final integration testing: ensure all platforms use KMP shared logic correctly
- [ ] KMP module versioning: semantic versioning for future updates
- [ ] Documentation: inline KMP code comments, architecture diagrams

#### QA Tasks
- [ ] Regression: full test suite on release candidate builds
- [ ] UAT (User Acceptance Testing): 20 beta users, structured feedback
- [ ] Analytics validation: verify all events fire with correct properties
- [ ] Performance validation: meet all benchmarks (Section 10)

**Sprint Deliverable:** Release candidate build with zero P0 bugs, <10 P1 bugs, ready for store submission.

---

### Sprint 8 (Weeks 15-16): **Launch Preparation & Store Approval**

**Sprint Goal:** App Store and Play Store submission, production deployment, launch readiness.

#### iOS Tasks
- [ ] App Store assets: screenshots (6.7", 6.5", 5.5"), app preview video
- [ ] App Store metadata: title, subtitle, description, keywords (ASO)
- [ ] Privacy manifest: declare data collection, required reasons API
- [ ] App Review submission: 3-5 day review window
- [ ] Monitor App Review status, respond to rejections within 24h

#### Android Tasks
- [ ] Play Store assets: screenshots, feature graphic, promo video
- [ ] Play Store metadata: short/long description, keywords (ASO)
- [ ] Play Console setup: closed testing → open testing → production
- [ ] App Review submission: 1-3 day review window (faster than iOS)
- [ ] Monitor Play Console, respond to pre-launch report issues

#### Backend Tasks
- [ ] Production deployment: Supabase production project, separate from staging
- [ ] Environment variables: API keys (scraper, OpenWeatherMap, affiliate)
- [ ] Database migration: run all migrations on production DB
- [ ] Edge Functions deployment: deploy all serverless functions
- [ ] Monitoring: wire production metrics to Grafana dashboard

#### KMP Tasks
- [ ] Final KMP module release: tag v1.0.0, publish to internal repo
- [ ] iOS framework build: compile KMP to .xcframework for iOS app
- [ ] Android artifact: publish KMP .aar to local Maven repo

#### QA Tasks
- [ ] Final smoke test: install from TestFlight/App Distribution, verify core flows
- [ ] Production readiness checklist: confirm all 50 launch criteria (Section 8)
- [ ] Launch day monitoring: stand by for crash reports, user feedback
- [ ] Post-launch hotfix plan: rollback procedure, emergency patch process

**Sprint Deliverable:** App live on App Store and Play Store, production backend deployed, monitoring active.

---

## 6. Critical Path Analysis

The **critical path** is the sequence of dependent tasks that determines the minimum project duration. Any delay on the critical path delays the entire project.

### Critical Path: Share Extension → Categorization → Library → Cards → Deep Extract

```
F1: Share Extension (Weeks 1-2)
  ↓ (URL capture required for all subsequent features)
F2: Auto-Categorization (Weeks 3-4)
  ↓ (Category data required for library filtering and card templates)
F3: Content Library (Weeks 5-6)
  ↓ (Library UI required to access cards)
F4: Structured Cards (Weeks 7-8)
  ↓ (Card templates required to display deep extract results)
F5: Deep Extract (Weeks 9-10)
  ↓ (Feature-complete milestone)
Polish + Bug Bash (Weeks 11-14)
  ↓
Launch Prep (Weeks 15-16)
```

### Parallelizable Work (Not on Critical Path)

- **Backend Supabase setup** can happen in parallel with iOS/Android F1 work
- **KMP module scaffolding** can start in Sprint 1 alongside native app setup
- **FTS5 schema design** can be finalized while F2 classification is being built
- **Card template design** can be mocked in Sprint 3, implemented in Sprint 4

### Blockers & Dependencies

| Feature | Blocks | Blocked By | Risk Level |
|---------|--------|------------|------------|
| **F1 Share Extension** | F2, F3, F4, F5 (everything) | None | **HIGH** (on critical path) |
| **F2 Categorization** | F3 (filter), F4 (card selection) | F1 | **HIGH** (on critical path) |
| **F3 Library** | F5 (where to trigger extract) | F1, F2 | **MEDIUM** (can dogfood without F3) |
| **F4 Structured Cards** | F5 (where to display results) | F2, F3 | **MEDIUM** |
| **F5 Deep Extract** | Launch | F1, F2, F4 | **MEDIUM** (can launch with Tier 1/2 only) |
| **Backend Scraper** | F5 | None (parallel track) | **HIGH** (scraper fragility) |
| **On-Device AI** | F2 quality | None | **MEDIUM** (Tier 2 fallback) |

### Mitigation Strategies

1. **F1 Risk:** Share extension is the foundation. Allocate best developer to this. Prototype in Sprint 0 (pre-kickoff) if possible.
2. **Scraper Risk:** Integrate Apify by end of Sprint 2. Test with 100 URLs daily. Have fallback provider (ScrapeCreators) ready by Sprint 4.
3. **On-Device AI Risk:** If Foundation Models/Gemini Nano are unavailable or low-quality, fall back to Tier 2.5 (bundled model). Don't block on Tier 1.
4. **Timeline Buffer:** Sprints 6-7 are deliberate buffer zones. If any feature slips, compress polish work to recover.

---

## 7. Technical Risk Register

### Top 10 Risks with Mitigation

| # | Risk | Probability | Impact | Mitigation | Owner |
|---|------|-------------|--------|------------|-------|
| **1** | **Instagram changes anti-scraping measures, breaking video extraction** | High (40%) | Critical | Abstract scraper layer. Contract 3 providers (Apify, ScrapeCreators, Bright Data). Circuit breaker auto-switches. Caption-first approach still works. Monthly scraper health checks. | Backend Dev |
| **2** | **Apple/Google changes on-device AI APIs mid-development** | Medium (25%) | High | Tier 2 rule-based fallback always available. WhisperKit (open-source) as alternative to SpeechAnalyzer. Server-side processing as ultimate fallback. Lock iOS SDK version until stable. | iOS/Android Dev |
| **3** | **On-device AI quality divergence (Gemini Nano underperforms vs Foundation Models)** | Medium (30%) | High | Build cross-platform accuracy benchmark (500 test cases). If Gemini Nano <threshold, route to Tier 2.5 or server. Monitor per-platform override rates. Contribute feedback to Google. | iOS/Android Dev |
| **4** | **Low free-to-paid conversion (<5%)** | Medium (35%) | Medium | On-device processing means free tier costs ~$0. Can sustain generous free tier. Experiment with paywall placement, 7-day trial, annual pricing. Ad-supported tier as fallback. | Product/Backend |
| **5** | **App Store rejection (scraping-related)** | Low (15%) | High | App never stores videos. User-initiated sharing is standard. Similar apps (Trott, PickTok) approved. Emphasize "personal library" in review notes. | iOS Dev |
| **6** | **Backend job queue overwhelmed (viral spike)** | Low (20%) | Medium | Auto-scaling workers (max 10 containers). Rate limit: 100 jobs/min global. Priority queue (paid > free). Redis memory: 4GB → 16GB upgrade path. Alert at 80% queue depth. | Backend Dev |
| **7** | **KMP integration complexity (iOS ↔ Kotlin/Native bridging issues)** | Medium (30%) | Medium | Start KMP in Sprint 1. Incremental adoption—network layer first, then data models. iOS dev shadows KMP dev in Sprints 1-2. Fallback: keep more logic native, less shared. | KMP/iOS Dev |
| **8** | **Database schema migration breaks existing data** | Low (10%) | High | Lightweight Core Data/Room migrations only. Test migrations on 10K record dataset before production. Backup before every migration. Rollback script ready. | iOS/Android Dev |
| **9** | **Third-party API rate limits (OpenWeatherMap, Whisper, LLM)** | Medium (25%) | Low | Cache aggressively (weather: 3h, LLM results: 30d). Upgrade tiers preemptively (OpenWeatherMap: $40/mo for 1M calls). Graceful degradation—hide weather if unavailable. | Backend Dev |
| **10** | **Cross-device sync data loss (user switches phones)** | Low (15%) | Medium | Cloud sync via Supabase ensures backup. Test iOS→Android and Android→iOS migration flows in QA. Data export format (JSON) as portable backup. Incremental sync for large libraries. | Backend/QA |

### Risk Monitoring Cadence

- **Weekly:** Review top 3 risks in tech sync meeting. Update probability/mitigation.
- **Sprint Retrospective:** Did any risk materialize? Add new risks identified.
- **Launch -2 weeks:** Final risk assessment. Go/no-go decision based on open P0 risks.

---

## 8. Launch Checklist

### Pre-Launch Criteria (All Must Be TRUE)

#### Functional Requirements

- [ ] **F1:** Share extension captures URLs from Instagram, TikTok, YouTube on both platforms
- [ ] **F1:** Offline saves queue locally, process when connectivity returns
- [ ] **F1:** Duplicate detection prevents saving same URL twice
- [ ] **F2:** Auto-categorization accuracy >80% on 300-item test set
- [ ] **F2:** Tier 1 on-device AI works on iPhone 15 Pro+ / Pixel 8+
- [ ] **F2:** Tier 2 rule-based classifier works on all devices
- [ ] **F2:** Manual category override works with one tap
- [ ] **F3:** Library grid loads in <500ms, scrolls at 60fps
- [ ] **F3:** FTS5 search returns results in <100ms for 1000+ saves
- [ ] **F3:** Category filter applies instantly (no loading state)
- [ ] **F3:** Thumbnail cache hit rate >95% on recent items
- [ ] **F4:** All 8 card types render correctly (Recipe, Travel, Product, Fitness, Beauty, Education, Generic)
- [ ] **F4:** Recipe card: ingredient checklist, unit conversion, serving adjuster, timers work
- [ ] **F4:** Travel card: map snapshot, weather widget, distance from user work
- [ ] **F4:** Product card: purchase link, affiliate tags, price staleness warning work
- [ ] **F5:** Deep extract completes in <30s for 60-second Reel
- [ ] **F5:** Free user quota enforced (10/month), paid unlimited
- [ ] **F5:** Failed extracts don't count against quota
- [ ] **F5:** Background processing completes when app is closed

#### Quality & Performance

- [ ] Crash-free rate >99.5% (measured over 7 days in beta)
- [ ] Memory usage <150MB for library with 1000+ saves
- [ ] Battery drain <5% per hour during active use (measured on iPhone 13 / Pixel 7)
- [ ] All P0 bugs resolved, <5 P1 bugs open
- [ ] Accessibility: full VoiceOver/TalkBack navigation tested
- [ ] Dark mode: all screens audited, no contrast issues
- [ ] Analytics: all 50+ events firing correctly with complete payloads

#### Security & Compliance

- [ ] SSRF prevention: URL validation rejects private IPs and non-allowlisted domains
- [ ] Rate limiting: per-user and global limits enforced
- [ ] Quota enforcement: server-side checks, not client-side trust
- [ ] Temp file cleanup: all processing files deleted within 60s
- [ ] Privacy manifest (iOS): data collection declared
- [ ] GDPR/CCPA compliance: user data deletion flow works
- [ ] Scraper legal review: public-only content, user-initiated, no video storage

#### App Store / Play Store

- [ ] App Store: screenshots, app preview video, metadata complete
- [ ] App Store: Privacy Nutrition Label filled out
- [ ] App Store: Age rating set (4+)
- [ ] Play Store: screenshots, feature graphic, metadata complete
- [ ] Play Store: Data safety section filled out
- [ ] Play Store: Content rating (PEGI 3, ESRB E)
- [ ] Both stores: Support URL, Privacy Policy, Terms of Service live

#### Backend & Infrastructure

- [ ] Production Supabase project deployed, separate from staging
- [ ] Database backups: automated daily to S3
- [ ] Monitoring: Grafana dashboard live, alerts configured
- [ ] Circuit breakers: scraper health checks passing
- [ ] Job queue: DLQ alerts configured (<50 jobs/hour threshold)
- [ ] Redis cache: 4GB memory, <80% utilization
- [ ] Edge Functions: all deployed to production, health checks green
- [ ] API keys: all production keys rotated, not shared with staging

#### Business & Operations

- [ ] Pricing tiers configured: $4.99 US, $1.99 India (App Store / Play Store)
- [ ] Free trial: 7-day trial mechanics tested end-to-end
- [ ] Payment processing: test purchases complete successfully
- [ ] Support email: `support@staq.app` monitored
- [ ] FAQ / Help Center: basic troubleshooting articles live
- [ ] Launch announcement: blog post, social media posts scheduled

### Launch Day Checklist

**T-24 hours:**
- [ ] Final smoke test on production build
- [ ] Monitoring dashboard review: all systems green
- [ ] On-call rotation: 24/7 coverage for first 72 hours
- [ ] Rollback plan reviewed: steps documented, credentials ready

**T-0 (Launch):**
- [ ] App Store: submit for release (manual release, not automatic)
- [ ] Play Store: publish to 100% production track
- [ ] Backend: flip feature flags to enable all production features
- [ ] Monitoring: watch crash rate, API error rate, job queue depth
- [ ] Social media: publish launch announcement

**T+4 hours:**
- [ ] Check metrics: installs, crash rate, save success rate
- [ ] Monitor support email for early user issues
- [ ] Triage any P0 bugs, hotfix if needed

**T+24 hours:**
- [ ] Review first-day metrics (Section 10)
- [ ] Hotfix decision: patch if crash rate >1% or critical bug found
- [ ] Post-launch retrospective: what went well, what to improve

---

## 9. Post-MVP Roadmap

### Phase 2: Polish & Retention (Months 4-6, Weeks 17-28)

**Goal:** Increase D30 retention from >25% to >35%, drive paid conversion to >8%.

| Feature | Effort | Priority | Rationale |
|---------|--------|----------|-----------|
| **F6: Collections & Boards** | 40h (2 weeks) | P1 | User-created themed boards with shareable links. Drives re-engagement (users return to organize). |
| **F7: Smart Search (NLP)** | 42h (2 weeks) | P1 | Natural language queries ("pasta with lemon"). Increases findability, reduces "can't find my save" frustration. |
| **F8: Reminder & Action Integration** | 30h (1.5 weeks) | P1 | "Remind me to cook this Saturday." Integrates with device calendar. Creates behavioral habit loop. |
| **F9: Grocery List from Recipes** | 31h (1.5 weeks) | P1 | Multi-recipe ingredient aggregation. Strong retention driver for recipe savers (core persona). |
| **F14: Widget Support** | 35h (2 weeks) | P1 | Home screen widgets (recent saves, quick save). Ambient awareness, increases weekly active savers. |
| **Paywall Optimization** | 20h (1 week) | P1 | A/B test paywall placement, trial length (7d vs 14d), annual pricing prominence. |
| **Onboarding Improvements** | 15h (1 week) | P1 | Reduce FTUX friction. Goal: install → first save >60% within 24h. |

**Phase 2 Total: ~213 hours (~10 weeks)**

### Phase 3: Growth & Monetization (Months 7-12, Weeks 29-52)

**Goal:** Reach 100K users, $25K MRR, build network effects.

| Feature | Effort | Priority | Rationale |
|---------|--------|----------|-----------|
| **F10: Collaborative Collections** | 44h (3 weeks) | P2 | Shared boards with invites, real-time sync. Viral growth mechanism (invite loop). |
| **F11: Web Clipper (Browser Extension)** | 38h (2 weeks) | P2 | Chrome/Safari extensions. Expands TAM beyond mobile-first users. |
| **F12: Trending & Discover** | 33h (2 weeks) | P2 | Aggregated trends feed, personalized by interests. Discovery layer, reduces cold-start problem. |
| **F13: Export & Integrations** | 50h (2.5 weeks) | P2 | Notion, Google Sheets, Apple Notes export + API. Power user retention, enterprise signal. |
| **Price Drop Alerts (Product Cards)** | 30h (1.5 weeks) | P2 | Push notification when saved products drop in price. Strong retention + re-engagement. |
| **Multi-Device Sync (Cross-Platform)** | 40h (2 weeks) | P2 | Full cloud sync via Supabase. Retention boost (users never lose data on device switch). |
| **Referral Program** | 25h (1.5 weeks) | P2 | "Give 1 month Pro, get 1 month Pro" invite loop. Organic growth amplifier. |

**Phase 3 Total: ~260 hours (~12 weeks)**

### Phase 4: Scale & Enterprise (Year 2)

- International expansion (Southeast Asia, Middle East, Brazil)
- Team features (shared workspaces, admin controls)
- Enterprise pricing tier ($49/user/month for teams)
- API platform (developer ecosystem)
- Pinterest support (image-first content)
- YouTube full-length video support (educational content)

---

## 10. Metrics & Success Criteria

### North Star Metric
**Weekly Active Savers (WAS):** Number of users who save at least one piece of content per week.

Target trajectory:
- **Month 1:** 200 WAS (beta cohort)
- **Month 3:** 1,500 WAS (post-launch growth)
- **Month 6:** 8,000 WAS (viral curve begins)
- **Month 12:** 40,000 WAS (product-market fit)

### Launch Day Metrics (Day 1)

| Metric | Target | Data Source |
|--------|--------|-------------|
| **Installs** | 500 (soft launch to beta list) | App Store Connect / Play Console |
| **Onboarding complete** | >80% of installs | Analytics: `onboarding_completed` |
| **First save within 24h** | >50% of onboarding completers | Analytics: `save_initiated` |
| **Crash-free rate** | >99% | Sentry / Crashlytics |
| **Share extension activation** | >70% of users | Analytics: `share_extension_opened` |

### Week 1 Metrics (D1-D7)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **D1 Retention** | >60% | Users who return on Day 1 after install |
| **D7 Retention** | >35% | Users who return on Day 7 after install |
| **Saves per user** | Avg 5 saves in first week | Analytics: `save_completed` count per user |
| **Categorization accuracy** | >80% correct (measured by override rate <20%) | Analytics: `category_overridden` / `categorization_completed` |
| **Save success rate** | >95% (saves complete without error) | Analytics: `save_completed` / `save_initiated` |
| **Search usage** | >30% of users search within first week | Analytics: `search_performed` |

### Month 1 Metrics (D1-D30)

| Metric | Target | Measurement |
|--------|--------|-------------|
| **D30 Retention** | >25% | Users still active on Day 30 |
| **Weekly Active Savers** | 200 (beta cohort) | Users saving ≥1 item per week |
| **Deep Extract usage** | Avg 3 extracts per user | Analytics: `deep_extract_completed` count |
| **Deep Extract completion rate** | >90% (not cancelled or failed) | `deep_extract_completed` / `deep_extract_initiated` |
| **Free-to-paid conversion** | >3% (early signal) | Users who start trial or subscribe |
| **NPS (Net Promoter Score)** | >30 (in-app survey) | "How likely are you to recommend Staq?" (0-10) |
| **Server cost per user** | <$0.05/month (free tier) | AWS/Supabase bill / MAU |

### Month 3 Metrics (Product-Market Fit Indicators)

| Metric | Target | Signal |
|--------|--------|--------|
| **D30 Retention** | >30% | Users finding long-term value |
| **Weekly Active Savers** | 1,500 | Growing engaged user base |
| **Saves per user per week** | Avg 8 | Habitual usage forming |
| **Organic growth** | >40% of new users from word-of-mouth | Attribution: organic vs paid |
| **Free-to-paid conversion** | >5% | Monetization validated |
| **NPS** | >40 | Strong product-market fit signal |
| **Category accuracy** | >85% (measured by override rate <15%) | AI quality improving via feedback loop |

### Month 6 Metrics (Growth Phase)

| Metric | Target | Signal |
|--------|--------|--------|
| **Total Users** | 20,000 | Viral growth trajectory |
| **Paid Subscribers** | 1,000 (5% conversion) | $4,990 MRR |
| **D30 Retention** | >35% | Retention improving with Phase 2 features |
| **Server costs** | <$500/month ($0.025/user) | On-device processing scaling well |
| **Gross margin** | >95% | Unit economics validated |

### Month 12 Metrics (Scale Milestone)

| Metric | Target | Signal |
|--------|--------|--------|
| **Total Users** | 100,000 | Product-market fit achieved |
| **Paid Subscribers** | 5,000 (5% conversion) | $24,950 MRR (~$300K ARR) |
| **D30 Retention** | >40% | Best-in-class for utility apps |
| **Server costs** | <$2,500/month | 10% of revenue, sustainable |
| **Gross margin** | >97% | Near-perfect SaaS economics |

---

## 11. Budget Estimate

### 6-Month Cost Projection (Launch + Growth)

#### Infrastructure & Services

| Service | Month 1 | Month 3 | Month 6 | Notes |
|---------|---------|---------|---------|-------|
| **Supabase** | $25 (Pro plan) | $25 | $100 (Team plan) | PostgreSQL, Auth, Storage, Edge Functions. Team plan at 10K+ users. |
| **Redis (Upstash)** | $10 (1GB) | $20 (2GB) | $50 (4GB) | URL dedup cache, rate limiting. |
| **Scraper APIs** | $30 | $150 | $500 | Apify/ScrapeCreators. Avg $0.003/extract × usage. |
| **LLM APIs** | $20 | $80 | $200 | Claude Haiku / GPT-4o-mini for server-side extraction. Declining with on-device adoption. |
| **OpenWeatherMap** | $0 (free tier) | $0 | $40 (paid tier) | 60 calls/min free → $40/mo for 1M calls. |
| **Sentry (Crash Reporting)** | $0 (free tier) | $26 (Team plan) | $80 (Business plan) | 5K errors/mo free, then tiered. |
| **CDN (Cloudflare)** | $0 (free tier) | $0 | $20 (Pro plan) | Thumbnail caching, static assets. |
| **Domain & SSL** | $15/year | — | — | staq.app domain, Cloudflare SSL free. |
| **Total Infrastructure** | **$100** | **$301** | **$990** | |

#### Developer Tools

| Tool | Cost | Frequency | Notes |
|------|------|-----------|-------|
| **Apple Developer Program** | $99 | Annual | Required for App Store distribution. |
| **Google Play Developer** | $25 | One-time | Required for Play Store distribution. |
| **GitHub Team** | $4/user/month | Monthly | 5 users × $4 = $20/month. |
| **Figma Professional** | $12/user/month | Monthly | 2 designers × $12 = $24/month. |
| **Postman Team** | $12/user/month | Monthly | 3 devs × $12 = $36/month (API testing). |
| **Total Dev Tools** | **~$100/month** | Monthly | |

#### Third-Party Affiliates & Revenue Share

| Service | Cost Model | Notes |
|---------|------------|-------|
| **Amazon Associates** | Revenue share (1-4% commission) | Affiliate tags on product links. Net revenue to Staq. |
| **Firebase Dynamic Links** | Free tier: 25K links/month | Upgrade at $5/mo for 100K links if QR code usage scales. |

#### Total 6-Month Budget Estimate

| Category | Cumulative (Months 1-6) |
|----------|-------------------------|
| Infrastructure | $2,400 (avg $400/mo) |
| Dev Tools | $600 (avg $100/mo) |
| One-Time Fees | $124 (Apple + Google + domain) |
| **Total 6-Month Spend** | **~$3,100** |

**Revenue Projection (6 months):**
- Month 1: $0 (beta, no paid users)
- Month 2: $500 (100 paid users @ $4.99)
- Month 3: $1,250 (250 paid users)
- Month 4: $2,500 (500 paid users)
- Month 5: $3,750 (750 paid users)
- Month 6: $5,000 (1,000 paid users @ 5% conversion from 20K users)

**Cumulative 6-Month Revenue: ~$13,000**

**Net: +$9,900** (profitable after Month 3)

### Scaling to 100K Users (Year 1)

At 100K users (5% paid = 5,000 subs):
- **Revenue:** $24,950/month (~$300K/year)
- **Infrastructure:** ~$2,500/month (server costs scale linearly with usage)
- **Net Margin:** ~$22,450/month (~$270K/year, 90% margin)

**Conclusion:** Unit economics are exceptional due to on-device processing. Break-even occurs around 500 users. Profitability scales rapidly beyond that.

---

## 12. Decision Log

Key architectural and strategic decisions with rationale.

### Decision 1: Native iOS + Android (Not Flutter or React Native)

**Decision:** Build two fully native apps—Swift/SwiftUI (iOS) and Kotlin/Jetpack Compose (Android)—instead of a cross-platform framework.

**Rationale:**
- **On-device AI frameworks are native:** Foundation Models (iOS 26), Gemini Nano (Android) are first-party APIs with zero bridging overhead.
- **Share Extension performance:** Native share extensions are faster (<500ms target) and more reliable than cross-platform plugin wrappers.
- **Deep OS integration:** Calendar/reminder integration, background processing, native notifications all benefit from native APIs.
- **Trade-off acknowledged:** Two codebases increase dev effort by ~30%. Mitigated by KMP shared logic layer.

**Alternatives considered:**
- Flutter: Popular, but on-device AI support requires platform channels (slow). Share extension as native plugin still required.
- React Native: Similar bridging issues. Community plugins for share extensions are unmaintained.

**Decision Date:** Sprint 0 (pre-kickoff)  
**Decision Maker:** CTO/Founder  
**Status:** Locked

---

### Decision 2: Kotlin Multiplatform (KMP) for Shared Logic

**Decision:** Use KMP to share 30-40% of business logic (networking, data models, classification rules, sync engine) while keeping UI and on-device AI fully native.

**Rationale:**
- **Reduce duplication:** URL parsing, API client, classification keywords, caching policy—identical on both platforms, should be written once.
- **Type safety:** Ktor, kotlinx.serialization, SQLDelight generate type-safe Kotlin code shared across iOS (via Kotlin/Native) and Android (via JVM).
- **No runtime bridge:** KMP compiles to native binaries on both platforms. Zero performance overhead.
- **Proven at scale:** Used by Netflix, VMware, Cash App for production cross-platform logic.

**Scope of KMP:**
- ✅ Networking (Ktor HTTP client)
- ✅ Data models (kotlinx.serialization)
- ✅ Database schema (SQLDelight)
- ✅ Classification rules (keyword dictionaries, regex extractors)
- ✅ URL parsing and normalization
- ✅ Caching logic
- ❌ UI (SwiftUI / Compose stay native)
- ❌ On-device AI (Foundation Models / Gemini Nano are native frameworks)

**Alternatives considered:**
- Pure native duplication: 2x the code, 2x the bugs, 2x the maintenance. KMP reduces this significantly.
- GraphQL/REST-only sharing: Doesn't help with client-side logic (classification, caching, URL parsing).

**Decision Date:** Sprint 0  
**Decision Maker:** Tech Lead / CTO  
**Status:** Locked

---

### Decision 3: Four-Tier Classification Strategy

**Decision:** Implement a four-tier classification system:
1. **Tier 1:** On-device AI (Foundation Models / Gemini Nano) on flagship devices
2. **Tier 2.5:** Bundled lightweight ML model (CoreML/TFLite, 2-5MB) on mid-range devices
3. **Tier 2:** Rule-based keyword/hashtag classifier (all devices)
4. **Tier 3:** Server-side LLM extraction (Deep Extract, on-demand)

**Rationale:**
- **Cost optimization:** Tier 1 processes 60-70% of saves at $0.00 server cost.
- **Universal compatibility:** Tier 2 rule-based fallback ensures every device gets a functional experience.
- **Quality gradient:** Tier 1 is the best, Tier 2.5 is good, Tier 2 is acceptable. Automatic degradation maintains UX.
- **Progressive enhancement:** Users on older devices aren't locked out. They get basic classification and can use Deep Extract for high-quality results.

**Alternatives considered:**
- Server-only classification: Simplest dev, but costs $0.02-0.07/save, making freemium economics unviable.
- On-device only: Excludes 40% of users without flagship devices. Not acceptable.

**Decision Date:** Sprint 0  
**Decision Maker:** CTO/Founder  
**Status:** Locked, but Tier 2.5 optional (can ship MVP with Tier 1 + 2)

---

### Decision 4: Scraper Service Strategy (Multi-Provider with Circuit Breaker)

**Decision:** Abstract scraper behind a provider interface with three contracted providers (Apify, ScrapeCreators, Bright Data) and circuit breaker logic for automatic failover.

**Rationale:**
- **Scraper fragility:** Instagram/TikTok change anti-bot measures frequently. Single-provider dependency is a business risk.
- **Failover speed:** Circuit breaker detects 3 failures in 5 minutes → auto-switches to fallback provider within seconds. No manual intervention.
- **Cost optimization:** Apify ($0.0026) is cheapest and most reliable. Use it 95% of the time. Fallbacks are more expensive but available when needed.
- **Legal defensibility:** Meta v. Bright Data ruling (Jan 2024) established that public scraping is legal. Multi-provider reduces platform risk.

**Alternatives considered:**
- Single provider (Apify): Cheaper upfront, but catastrophic if Apify is blocked or goes down.
- Build in-house scraper: Massive engineering effort (~400 hours), constant maintenance burden, likely less reliable than specialized providers.

**Decision Date:** Sprint 1  
**Decision Maker:** Backend Lead / CTO  
**Status:** Locked

---

### Decision 5: Deep Extract Quota (10 Free, Unlimited Pro)

**Decision:** Free users get 10 deep extracts per month. Paid users ($4.99/mo) get unlimited.

**Rationale:**
- **Align costs with value:** Deep extract is the only server-intensive operation. Free users get generous on-device processing (unlimited), pay for expensive server processing.
- **Quota is understandable:** "10 free detailed analyses per month" is easier to explain than abstract feature gates.
- **Conversion funnel:** Users hit quota → see value → upgrade. Measured in analytics: `quota_exhausted_shown` → `paywall_upgrade_tapped` → subscription.
- **Quota doesn't hurt retention:** 80% of users use <5 deep extracts/month. 10 is generous for most users.

**Alternatives considered:**
- Unlimited free: Server costs balloon. Not sustainable at scale.
- Credit-based (like Trott): More complex UX. Users don't understand "credits".
- Time-based trial (7 days free, then pay): High churn. Users save sporadically, not daily.

**Decision Date:** Sprint 0  
**Decision Maker:** Product / CTO  
**Status:** Locked

---

### Decision 6: India as Primary Launch Market

**Decision:** Launch in India first (English + Hindi content), then US secondary.

**Rationale:**
- **Largest Instagram user base:** India has more Reels creators and consumers than any other country.
- **WhatsApp sharing culture:** Indians forward Reels via WhatsApp → Staq's "paste URL" flow is natural.
- **Price sensitivity → on-device value:** Indian users are data-conscious. On-device processing (no data upload) is a selling point.
- **Regional pricing validated:** $1.99/mo (₹149) in India vs $4.99/mo in US. Play Store supports regional pricing automatically.

**Alternatives considered:**
- US-first: Higher ARPU, but smaller TAM for Instagram Reels organization use case. India is where the pain is acute.

**Decision Date:** Sprint 0  
**Decision Maker:** Founder  
**Status:** Locked

---

### Decision 7: Freemium Over Ad-Supported

**Decision:** No ads in the free tier. Monetization is 100% subscription-based.

**Rationale:**
- **Ad SDKs bloat the app:** Firebase Ads, AdMob add 5-10MB, slow app startup, violate privacy positioning.
- **Privacy-first brand:** Staq's positioning is "your data stays on your device." Ads contradict this.
- **Ad revenue < subscription revenue:** Even at 20K MAU, ad revenue would be ~$500/month. 5% paid conversion = $5,000/month.
- **Freemium works because on-device costs $0:** Can afford a generous free tier. Ads aren't needed to sustain free users.

**Alternatives considered:**
- Ad-supported free tier: More revenue, but worse UX, privacy concerns, slower app.
- Pay-up-front (no free tier): Dead on arrival. Users won't pay before trying. Freemium is table stakes for social utility apps.

**Decision Date:** Sprint 0  
**Decision Maker:** Founder  
**Status:** Locked

---

### Decision 8: WAL Mode for SQLite (Concurrent Read/Write)

**Decision:** Enable Write-Ahead Logging (WAL) mode on SQLite databases (Core Data, Room).

**Rationale:**
- **Concurrency:** Share extension writes new saves while the library UI reads. WAL allows this without blocking.
- **Performance:** Sequential WAL writes are faster than random DB updates.
- **Standard practice:** Core Data and Room enable WAL by default on modern OS versions. Explicitly verify it's on.

**Alternatives considered:**
- Default journal mode: Exclusive locks for writes → UI freezes when share extension saves → bad UX.

**Decision Date:** Sprint 1 (database setup)  
**Decision Maker:** iOS/Android Leads  
**Status:** Locked

---

### Decision 9: FTS5 for Full-Text Search (Not Client-Side Fuzzy Matching)

**Decision:** Use SQLite FTS5 (Full-Text Search) for search, not client-side fuzzy matching libraries.

**Rationale:**
- **Performance:** FTS5 indexed search is <100ms for 10K records. Client-side fuzzy matching is O(n) per query → slow.
- **Scalability:** FTS5 scales to 100K+ records. Fuzzy matching doesn't.
- **Built-in:** SQLite FTS5 is included in iOS/Android. No extra dependencies.
- **Quality:** FTS5 BM25 ranking is better than naive substring matching.

**Alternatives considered:**
- Fuse.js (JavaScript fuzzy): Not available natively in Swift/Kotlin. Requires bridging, slow.
- Manual LIKE queries: `WHERE caption LIKE '%pasta%'` doesn't scale, no ranking, no prefix matching.

**Decision Date:** Sprint 3 (library search)  
**Decision Maker:** iOS/Android Leads  
**Status:** Locked

---

### Decision 10: Supabase Over Custom Backend

**Decision:** Use Supabase (PostgreSQL + Auth + Edge Functions) instead of building a custom Node.js/Python backend.

**Rationale:**
- **Time to market:** Supabase provides auth, database, and serverless functions out of the box. Saves 4-6 weeks vs custom backend.
- **Auto-scaling:** Supabase scales automatically. No DevOps burden for Weeks 1-16.
- **Cost-effective:** Free tier → $25/mo Pro → $100/mo Team. Cheaper than AWS EC2 + RDS for low-volume MVP.
- **Real-time subscriptions:** Supabase Realtime for future multi-device sync. Built-in, no custom WebSocket server.

**Alternatives considered:**
- Custom Node.js (Express) + PostgreSQL: More flexible, but requires Docker, CI/CD, monitoring setup. Overkill for MVP.
- Firebase: Good for mobile-first, but PostgreSQL (Supabase) is better for complex queries (FTS5, analytics).

**Decision Date:** Sprint 0  
**Decision Maker:** Backend Lead / CTO  
**Status:** Locked

---

## Execution Summary

This Master Execution Plan provides a **complete roadmap** for building Staqer from Sprint 1 to App Store/Play Store launch in **16 weeks**. The plan is grounded in:

1. **Realistic effort estimates** (583 hours total, parallelized across 4.5 FTE)
2. **Clear dependencies** (critical path: F1 → F2 → F3 → F4 → F5)
3. **Risk mitigation** (top 10 risks identified, circuit breakers in place)
4. **Measurable success criteria** (>80% categorization accuracy, >25% D30 retention, >5% conversion)
5. **Sustainable economics** (on-device processing → <$0.05/user/month server cost)

**Next Steps:**
1. **Sprint 0 (Pre-Kickoff, 1 week):** Finalize team, set up dev environments, prototype share extension
2. **Sprint 1 Kickoff (Week 1):** Execute Sprint 1 tasks as outlined in Section 5
3. **Weekly Tech Sync:** Monitor critical path, adjust timeline if blockers arise
4. **Mid-Sprint 4 (Week 8):** Dogfood internally—team uses Staq daily to find bugs
5. **Sprint 7 (Week 14):** Beta launch to 50 external testers, collect feedback
6. **Sprint 8 (Week 16):** Public launch on App Store + Play Store

**This document is the single source of truth for Staqer's execution.** All stakeholders (engineering, product, QA, marketing) should refer to this plan for alignment.

---

**Document Owner:** CTO/Founder  
**Last Updated:** February 24, 2026  
**Next Review:** End of Sprint 2 (Week 4)
