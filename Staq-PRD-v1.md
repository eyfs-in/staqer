# PRODUCT REQUIREMENTS DOCUMENT
## Staq — Smart Social Content Organizer

**Version:** 1.0  
**Date:** February 23, 2026  
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

### 2.2 User Pain Points

- **Saves become a graveyard** — users save content with intent to revisit but can never find it again among hundreds of bookmarks
- **Content is locked in video format** — a 60-second recipe Reel contains 12 ingredients and 8 steps, but users must re-watch the entire video to extract them
- **No cross-platform organization** — saves are siloed within each platform (Instagram, TikTok, YouTube, Pinterest) with no unified view
- **Context is lost** — users forget why they saved something weeks later; the original intent ("make this for dinner", "visit this place") disappears
- **Platform algorithms bury saves** — Instagram's saved collections have no search, no tags, and no sorting beyond chronological

### 2.3 Market Validation

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

### 4.2 Target Markets

- **Primary:** India (Instagram's largest user base, high Reels engagement, price-sensitive — optimize for lightweight/free path)
- **Secondary:** United States (higher willingness to pay, strong YouTube Shorts usage)
- **Tertiary:** Southeast Asia, Middle East, Brazil (high Instagram/TikTok penetration)

---

## 5. Competitive Landscape

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

### 5.1 Key Competitive Advantage

**On-device AI processing is Staq's unfair advantage.** Competitors run all AI processing server-side, costing $0.02–0.07 per save. This forces them into credit-based pricing or restrictive free tiers. Staq's on-device architecture processes 60–70% of saves at $0.00 server cost, enabling a genuinely generous free tier that drives viral growth while maintaining 97%+ gross margins on paid subscriptions.

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

### 6.5 Technology Stack

- **Frontend:** Native iOS (Swift / SwiftUI) + Native Android (Kotlin / Jetpack Compose)
- **On-Device AI (iOS):** Foundation Models framework, SpeechAnalyzer, Vision framework — direct native access, no bridging required
- **On-Device AI (Android):** ML Kit GenAI Prompt API, Gemini Nano via AICore, ML Kit Text Recognition — direct native access
- **Backend:** Node.js or Python (lightweight API server)
- **Database:** Supabase (PostgreSQL + Auth + Storage + Realtime)
- **Scraper APIs:** Apify ($0.0026/result) or ScrapeCreators ($0.002/credit) as primary; Bright Data as fallback
- **Speech-to-Text (server fallback):** OpenAI GPT-4o-mini-transcribe ($0.003/min) or Whisper API ($0.006/min)
- **LLM (server fallback):** Claude Haiku 4.5 ($0.002/call) or GPT-4o-mini ($0.001/call)
- **Caching:** Redis for URL deduplication (avoid reprocessing viral content)

#### Why Native Over Cross-Platform

The decision to build native apps on both platforms (rather than Flutter or React Native) is driven by Staq's core architecture:

- **Direct AI API access:** Foundation Models (iOS) and Gemini Nano / ML Kit GenAI (Android) are first-party native frameworks. Native apps access these with zero bridging overhead, faster adoption of new API features, and no dependency on third-party plugin maintainers.
- **Share Extension performance:** Share extensions are platform-native by definition. Native implementation ensures the fastest possible capture experience (< 500ms acknowledgment).
- **OS-level integration depth:** Calendar/reminder integration, background processing, notification handling, and on-device speech recognition all benefit from native implementation.
- **Trade-off acknowledged:** Two codebases increase development effort. Mitigated by using Kotlin Multiplatform (KMP) for shared business logic (networking, data models, classification rules, API clients) while keeping UI and AI layers fully native.

### 6.6 Legal & Compliance

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

### 8.1 Core Save Flow

1. User is browsing Instagram, sees an interesting Reel
2. User taps **Share → selects Staq** from share sheet
3. Staq share extension captures URL + metadata
4. Immediate toast: "✅ Saved to Staq! Organizing..."
5. On-device processing: classify content, extract data (< 2 sec)
6. User opens Staq later → sees the save already categorized with a structured card
7. If details are incomplete, "Get Full Details" button is visible

### 8.2 Retrieval Flow

1. User opens Staq → sees library grid organized by category
2. Taps "Recipes" category filter
3. Searches "pasta lemon"
4. Sees matching recipe card with ingredients and steps
5. Taps "View Original" to re-watch the Reel, or "Add to Shopping List"

### 8.3 Deep Extract Flow

1. User views a saved Reel card with minimal information (vague caption)
2. Taps "Get Full Details"
3. Free user: checks remaining monthly quota (10 free)
4. Progress spinner: "Analyzing video... extracting details..." (10–30 sec)
5. Card updates with full structured data (ingredients, location details, product info)
6. If quota exceeded: "Upgrade to Staq Pro for unlimited detailed extractions"

### 8.4 Richness Score Decision Logic

| Score | What Was Extracted | Action |
|-------|-------------------|--------|
| **High** | Full recipe with ingredients, or clear location + details | Save as complete |
| **Medium** | Got content type and topic but missing specifics | Save with "Get Full Details" button |
| **Low** | Just URL and vague caption like "omg this 😂" | Paid: auto-escalate. Free: save as bookmark only |

---

## 9. Business Model & Monetization

### 9.1 Pricing Tiers

| Feature | Free | Staq Pro ($4.99/month) |
|---------|------|------------------------|
| Saves per month | Unlimited | Unlimited |
| Auto-categorization | Yes (on-device) | Yes (on-device) |
| Search & filter | Yes | Yes + natural language search |
| Deep Extracts | 10 per month | Unlimited |
| Collections | 3 collections | Unlimited collections |
| Grocery list export | No | Yes |
| Shared collections | No | Yes |
| Export to Notion/Sheets | No | Yes |

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
| **Phase 0: Foundation** | Weeks 1–4 | iOS (Swift/SwiftUI) and Android (Kotlin/Compose) project setup, Supabase backend, share extension prototypes on both platforms, on-device AI integration | Share extension captures URLs from 3 platforms on both OS |
| **Phase 1: MVP** | Weeks 5–12 | Core save flow, on-device classification (all 3 tiers), content library UI, structured data cards, deep extract pipeline | 100 beta testers, >80% categorization accuracy, <3 sec processing |
| **Phase 2: Polish** | Weeks 13–18 | Collections, natural language search, reminder integration, grocery list, paywall implementation | App Store launch, 1,000 users, >5% conversion |
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
- **Monetization:** free-to-paid conversion rate, trial start rate, churn rate, LTV, ARPU
- **Infrastructure:** cost per save, cost per deep extract, scraper success rate, API latency

---

## 13. Risks & Mitigations

| Severity | Risk | Impact | Mitigation |
|----------|------|--------|------------|
| **High** | Instagram changes anti-scraping measures, breaking video extraction | Deep extract feature stops working for Instagram content | Abstracted scraper layer allows provider swap within hours. Multiple providers contracted (Apify, ScrapeCreators, Bright Data). Caption-first approach still works. |
| **High** | Apple/Google changes on-device AI APIs or deprecates frameworks | Tier 1 processing breaks | Server fallback always available. WhisperKit (open-source) as alternative to SpeechAnalyzer. Rule-based classifier works on all devices. |
| **Medium** | App Store rejection due to scraping-adjacent functionality | Cannot distribute via iOS App Store | App never downloads or stores videos. User-initiated URL sharing is standard behavior. Multiple similar apps (Trott, PickTok) approved and live. |
| **Medium** | Low free-to-paid conversion rate | Revenue below projections | On-device processing means free tier costs nearly nothing to support. Experiment with paywall placement, trial periods, and annual pricing. Ad-supported tier as fallback. |
| **Medium** | On-device AI quality insufficient for accurate extraction | Users get incorrect categorization, lose trust | Confidence scoring with graceful degradation. "Not sure about this one" messaging. Easy manual override. Server fallback for low-confidence saves. |
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

3. **Shared Backend Logic:** With native apps on both platforms, evaluate strategies for maximizing shared business logic — consider Kotlin Multiplatform (KMP) for shared networking, data models, and classification rules across iOS and Android, while keeping UI and on-device AI layers fully native.

4. **Share Sheet Metadata Variability:** The metadata available from the iOS/Android share sheet when sharing from Instagram varies by OS version and app version. Need extensive testing across device/OS combinations.

5. **India Pricing:** $4.99/month may be too high for the Indian market. Consider a regional pricing tier ($1.99–$2.99/month) or annual discount ($29.99/year).

6. **Pinterest Support:** Pinterest has a different content model (images + links vs. videos). Evaluate whether the same architecture applies or if a separate extraction pipeline is needed.

---

*End of Document — Staq PRD v1.0*
