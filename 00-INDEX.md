# Staq — Feature Design Documents Index

**13 Feature Specs · 517 estimated development hours · 4 phases**

---

## MVP Features (Phase 1 — Months 1–3)

| Doc | Feature | Priority | Effort | Dependencies |
|-----|---------|----------|--------|-------------|
| [F01](./F01-Share-Extension.md) | **Share Extension** — System share sheet integration for one-tap saving from Instagram, TikTok, YouTube | P0 | ~51h (1.5 wk) | None |
| [F02](./F02-Smart-Auto-Categorization.md) | **Smart Auto-Categorization** — Three-tier AI classification (on-device AI → rule-based → server) with keyword dictionaries and richness scoring | P0 | ~83h (3 wk) | F1 |
| [F03](./F03-Content-Library.md) | **Content Library** — Visual grid/list library with category filtering, full-text search, swipe actions | P0 | ~66h (2.5 wk) | F1, F2 |
| [F04](./F04-Structured-Data-Cards.md) | **Structured Data Cards** — Recipe, Travel, Product, Fitness, and Generic card templates with interactive elements | P0 | ~59h (2 wk) | F2, F3 |
| [F05](./F05-Deep-Extract.md) | **Deep Extract** — On-demand server scraping + on-device transcription pipeline with quota management and paywall | P0 | ~85h (3.5 wk) | F1, F2, F4 |

**Phase 1 Total: ~344 hours (~12 weeks with parallel iOS + Android development)**

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

1. **Feature Overview** — What it does and why it matters
2. **User Stories** — Acceptance criteria for each story
3. **UX Design** — ASCII wireframes, interaction flows, state diagrams, design principles
4. **Edge Cases** — Comprehensive error handling and boundary conditions
5. **Technical Implementation** — Architecture, data models, code patterns
6. **Development Plan** — Task-level breakdown with per-task effort estimates
7. **Definition of Done** — Checkboxes for QA sign-off

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
| Phase 1: MVP | 344h | ~12 weeks | F1–F5 |
| Phase 2: Polish | 143h | ~7 weeks | F6–F9 |
| Phase 3: Growth | 165h | ~9.5 weeks | F10–F13 |
| **Total** | **652h** | **~28.5 weeks** | **13 features** |

*Estimates assume 1 iOS + 1 Android developer working in parallel, with shared backend work.*
