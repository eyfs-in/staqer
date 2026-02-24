# Staqer — Feature Design Documents Index

**13+ Feature Specs · ~890 estimated development hours · 4 phases**

*Native iOS (Swift/SwiftUI) + Native Android (Kotlin/Jetpack Compose) · KMP shared logic layer*

---

## MVP Features (Phase 1 — Months 1–4)

| Doc | Feature | Priority | Effort | Dependencies |
|-----|---------|----------|--------|-------------|
| [F01](./F01-Share-Extension.md) | **Share Extension** — System share sheet integration for one-tap saving from Instagram, TikTok, YouTube, Threads + URL normalization pipeline, onboarding tutorial, analytics | P0 | ~64h (2 wk) | None |
| [F02](./F02-Smart-Auto-Categorization.md) | **Smart Auto-Categorization** — Four-tier AI classification (on-device AI → bundled model → TF-IDF keywords → regex) with multilingual dictionaries (5 languages), OCR pipeline, accuracy monitoring | P0 | ~116h (4 wk) | F1 |
| [F03](./F03-Content-Library.md) | **Content Library** — Visual grid/staggered/list library with category filtering, FTS5 search, Quick Save FAB, multi-layer thumbnail caching, offline-first architecture | P0 | ~130h (4 wk) | F1, F2 |
| [F04](./F04-Structured-Data-Cards.md) | **Structured Data Cards** — 8 card types (Recipe, Travel, Product, Fitness, Beauty, Education, Fashion, Generic) with unit conversion, timers, dietary tags, weather, sharing/virality, theming | P0 | ~146h (4-5 wk) | F2, F3 |
| [F05](./F05-Deep-Extract.md) | **Deep Extract** — Server scraping with circuit breaker + on-device transcription/OCR pipeline, job queue, cost optimization, security hardening, quality assurance | P0 | ~127h (5 wk) | F1, F2, F4 |

**Phase 1 Total: ~583 hours (~16 weeks with parallel iOS + Android development)**

---

## Post-MVP Features (Phase 2 — Months 4–6)

| Doc | Feature | Priority | Effort | Dependencies |
|-----|---------|----------|--------|-------------|
| [F06](./F06-Collections-Boards.md) | **Collections & Boards** — User-created themed boards with shareable links | P1 | ~40h (2 wk) | F3 |
| [F07](./F07-Smart-Search.md) | **Smart Search** — Natural language query parsing, filter inference, on-device semantic search | P1 | ~42h (2 wk) | F3, F2 |
| [F08](./F08-Reminder-Action-Integration.md) | **Reminder & Action Integration** — Set reminders on saves, export ingredients to shopping lists | P1 | ~30h (1.5 wk) | F4 |
| [F09](./F09-Grocery-List.md) | **Grocery List from Recipes** — Multi-recipe ingredient aggregation with dedup and aisle categorization | P1 | ~31h (1.5 wk) | F4, F8 |

**Phase 2 Total: ~143 hours (~7 weeks)**

---

## Growth Features (Phase 3 — Months 7–12)

| Doc | Feature | Priority | Effort | Dependencies |
|-----|---------|----------|--------|-------------|
| [F10](./F10-Collaborative-Collections.md) | **Collaborative Collections** — Shared boards with invites, permissions, comments, real-time sync | P2 | ~44h (3 wk) | F6 |
| [F11](./F11-Web-Clipper.md) | **Web Clipper** — Chrome + Safari browser extensions with YouTube transcript and recipe schema extraction | P2 | ~38h (2 wk) | F3, F2 |
| [F12](./F12-Trending-Discover.md) | **Trending & Discover** — Anonymized aggregated trends feed, personalized by user interests | P2 | ~33h (2 wk) | F2, Backend |
| [F13](./F13-Export-Integrations.md) | **Export & Integrations** — Notion, Google Sheets, Apple Notes export + REST API + Zapier/webhooks | P2 | ~50h (2.5 wk) | F3, F4 |

**Phase 3 Total: ~165 hours (~9.5 weeks)**

---

## What Each Doc Contains

Every feature document follows a consistent structure:

1. **Feature Overview** — What it does, why it matters, competitive context
2. **User Stories** — Acceptance criteria for each story
3. **UX Design** — ASCII wireframes, interaction flows, state diagrams, design principles
4. **Edge Cases** — Comprehensive error handling and boundary conditions
5. **Technical Implementation** — Architecture, data models, code patterns (Swift/Kotlin)
6. **Platform-Specific Details** — iOS (SwiftUI, Core Data, Foundation Models) and Android (Jetpack Compose, Room, Gemini Nano) native implementations
7. **Analytics & Events** — Key metrics, tracking events, success criteria
8. **Performance Benchmarks** — Target thresholds for latency, memory, battery
9. **Development Plan** — Task-level breakdown with per-task effort estimates
10. **Definition of Done** — Checkboxes for QA sign-off

---

## Development Dependency Graph

```
F1 Share Extension (foundation)
 │
 ├── F2 Auto-Categorization
 │    │
 │    ├── F3 Content Library
 │    │    │
 │    │    ├── F6 Collections
 │    │    │    └── F10 Collaborative Collections
 │    │    │
 │    │    ├── F7 Smart Search
 │    │    │
 │    │    ├── F11 Web Clipper
 │    │    │
 │    │    ├── F12 Trending & Discover
 │    │    │
 │    │    └── F13 Export & Integrations
 │    │
 │    └── F4 Structured Data Cards
 │         │
 │         ├── F5 Deep Extract
 │         │
 │         ├── F8 Reminder Integration
 │         │    └── F9 Grocery List
 │         │
 │         └── F13 Export & Integrations
 │
 └── (all features depend on F1)
```

---

## Effort Summary

| Phase | Hours | Weeks (parallel dev) | Features |
|-------|-------|---------------------|----------|
| Phase 1: MVP | 583h | ~16 weeks | F1–F5 |
| Phase 2: Polish | 143h | ~7 weeks | F6–F9 |
| Phase 3: Growth | 165h | ~9.5 weeks | F10–F13 |
| **Total** | **~891h** | **~32.5 weeks** | **13 features** |

*Estimates assume 1 iOS + 1 Android developer working in parallel, with shared KMP logic layer and dedicated backend work.*
