# F3: Content Library — Design & Development Guide

**Priority:** P0 (MVP)  
**Phase:** 1  
**Platforms:** iOS (Swift/SwiftUI) + Android (Kotlin/Compose)  
**Dependencies:** F1 (Share Extension), F2 (Auto-Categorization)  
**Estimated Effort:** 3–4 weeks

---

## 1. Feature Overview

The Content Library is the home screen of Staq — where users browse, search, and access all their saved content. It must feel faster and more useful than opening Instagram's own Saved tab. The library transforms a chaotic list of bookmarks into an organized, visual, searchable collection.

---

## 2. User Stories

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-3.1 | As a user, I want to see all my saves in a visual grid so I can quickly scan what I've collected | Thumbnail grid loads in < 500ms, smooth 60fps scrolling |
| US-3.2 | As a user, I want to filter by category so I can focus on recipes when I'm cooking | Category filter bar with one-tap filtering |
| US-3.3 | As a user, I want to search my saves by keyword so I can find "that pasta recipe" | Full-text search across captions, transcripts, extracted data |
| US-3.4 | As a user, I want to sort by date, platform, or category | Sort options accessible without deep navigation |
| US-3.5 | As a user, I want quick actions (delete, recategorize) without opening each save | Swipe actions on grid items or long-press context menu |
| US-3.6 | As a user, I want to see new saves appear automatically without pull-to-refresh | Real-time UI update when share extension saves new content |

---

## 3. UX Design

### 3.1 Design Principle: "Pinterest meets Spotlight Search"

The library should feel visual like Pinterest (grid of thumbnails that draw you in) combined with the speed and intelligence of iOS Spotlight (type anything, find instantly). It should NOT feel like a file manager or list of URLs.

### 3.2 Navigation Model

The app uses a 3-tab bottom navigation:

```
┌─────────┐   ┌──────────────┐   ┌──────────┐
│  📚     │   │  📁          │   │  ⚙️      │
│ Library │   │ Collections  │   │ Settings │
└─────────┘   └──────────────┘   └──────────┘
```

- **Library** (this feature) — the primary feed of all saved content with inline search
- **Collections** — user-created themed boards (F6)
- **Settings** — account, preferences, export, developer

**Why no standalone Search tab:** Search is embedded at the top of the Library screen because search is always contextual to the saves the user is looking at. A separate tab adds navigation friction to the most common action. When the user taps the search bar, the full search experience (recent searches, filters, results) takes over the screen as an overlay.

### 3.3 Main Library Screen

```
┌─────────────────────────────────────────┐
│ Library                   ↕️ sort  ▦ ▤  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │  🔍  Search your saves...           │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌────┐┌────┐┌────┐┌────┐┌────┐┌─────┐ │
│ │All ││🍳  ││✈️  ││🛍️  ││💪  ││More▸│ │
│ │    ││ 38 ││ 15 ││ 22 ││ 12 ││     │ │
│ └────┘└────┘└────┘└────┘└────┘└─────┘ │
│                                         │
│ Today                                   │
│ ┌──────────┐ ┌──────────┐              │
│ │ 🔵 NEW   │ │          │              │
│ │ [thumb]  │ │ [thumb]  │              │
│ │          │ │          │              │
│ │   🍳    │ │   ✈️    │              │
│ ├──────────┤ ├──────────┤              │
│ │ Garlic   │ │ Hidden   │              │
│ │ Pasta    │ │ Bali     │              │
│ │ 15 min   │ │ Beaches  │              │
│ │ IG · 2h  │ │ TT · 3h  │              │
│ └──────────┘ └──────────┘              │
│                                         │
│ Yesterday                               │
│ ┌──────────┐ ┌──────────┐              │
│ │          │ │          │              │
│ │ [thumb]  │ │ [thumb]  │              │
│ │          │ │          │              │
│ │   🛍️    │ │   💪    │              │
│ ├──────────┤ ├──────────┤              │
│ │ Amazon   │ │ Full     │              │
│ │ Find $29 │ │ Body     │              │
│ │          │ │ Workout  │              │
│ │ IG · 1d  │ │ YT · 1d  │              │
│ └──────────┘ └──────────┘              │
│                                         │
│ ┌─────┐   ┌─────┐   ┌─────┐          │
│ │  📚 │   │  📁 │   │  ⚙️ │          │
│ │Libry│   │Colls│   │ Set │          │
│ └─────┘   └─────┘   └─────┘          │
└─────────────────────────────────────────┘
```

**Header bar elements:**
- "Library" title (left)
- Sort icon ↕️ (right) — opens sort options
- View toggle ▦ grid / ▤ list (right) — switches layout

**"NEW" badge:** A small blue dot appears on saves added in the current session that haven't been viewed yet. Disappears after the card is tapped. Provides a sense of freshness without being intrusive.

### 3.4 Layout Options

#### Grid View (Default)
- 2-column grid with thumbnails
- Category badge overlay on thumbnail (bottom-right corner, frosted glass background)
- Below thumbnail: title (1 line, max), key detail (1 line), platform icon + relative time
- Grouped by date section headers ("Today", "Yesterday", "This Week", "January 2026")

#### List View (Toggle)

```
┌─────────────────────────────────────────┐
│ Today                                   │
│ ┌───────────────────────────────────┐   │
│ │ ┌────────┐ Garlic Pasta          │   │
│ │ │[thumb] │ 🍳 Recipe · 15 min   │   │
│ │ │        │ "Easy weeknight pasta │   │
│ │ │  🍳   │ with just 5 ingredi..."│   │
│ │ └────────┘ IG · @sara · 2h ago   │   │
│ └───────────────────────────────────┘   │
│ ┌───────────────────────────────────┐   │
│ │ ┌────────┐ Hidden Bali Beaches   │   │
│ │ │[thumb] │ ✈️ Travel · Bali     │   │
│ │ │        │ "Best secret spots   │   │
│ │ │  ✈️   │ you need to visit..." │   │
│ │ └────────┘ TT · @wanderlust · 3h │   │
│ └───────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

- Full-width rows with 60×60pt thumbnail on left, details on right
- Shows 2-line caption preview (truncated with ellipsis)
- Shows creator handle alongside platform + time
- Better for scanning text-heavy saves and finding saves by caption content
- **Swipe actions available:** Swipe left reveals Delete (red) + Recategorize (blue) action buttons

**List view swipe actions:**
```
│ ┌────────┐ Garlic Pasta          ┌──────┐┌──────┐│
│ │[thumb] │ 🍳 Recipe · 15 min   │🏷️    ││🗑️    ││
│ │        │ "Easy weeknight..."   │Recatg││Delete││
│ └────────┘ IG · @sara · 2h ago   └──────┘└──────┘│
```

**Why swipe is list-only:** Swipe gestures conflict with horizontal scrolling in grid layouts and feel unnatural on square grid cards. In list view, horizontal swipe on a full-width row is standard iOS/Android behavior.

### 3.5 Sort Options

Tapping the sort icon ↕️ opens a compact bottom sheet:

```
┌─────────────────────────────────────┐
│ Sort by                             │
│                                     │
│ ● Most recent first (default)      │
│ ○ Oldest first                     │
│ ○ By category (A → Z)             │
│ ○ By platform                      │
│ ○ By richness (most detail first)  │
│                                     │
└─────────────────────────────────────┘
```

- Radio selection — single sort at a time
- "Most recent first" is the default and most common
- "By richness" is useful for finding saves that still need deep extraction
- Sort applies across all views (grid and list) and persists across sessions
- When a sort other than date is active, the date section headers are replaced by the sort key grouping (e.g., "Instagram", "TikTok", "YouTube" when sorted by platform)

### 3.6 Category Filter Bar

Horizontally scrollable pill/chip bar below the search field:

```
┌────┐┌────┐┌────┐┌────┐┌────┐┌────┐┌─────┐
│ All││🍳38││✈️15││🛍️22││💪12││💄8 ││More▸│
└────┘└────┘└────┘└────┘└────┘└────┘└─────┘
```

- "All" is selected by default (filled/highlighted state)
- Each chip shows category emoji + save count
- Tapping a chip filters the grid instantly (no loading state)
- Chips are ordered by frequency (most saves first)
- "More▸" expands a bottom sheet with all categories including custom ones

**Filter behavior — single-select with toggle:**
- Tapping a category chip selects it exclusively (deselects "All" and any other active category)
- Tapping the same chip again deselects it and returns to "All"
- **Why single-select, not multi-select:** Multi-category filters create confusing states ("Am I seeing Recipes AND Travel, or Recipes OR Travel?"). For the MVP, single-filter keeps the mental model simple. Users wanting multi-category can use Search (F7: "recipes or travel from this week").

**UX detail:** When a filter is active, the section header changes from date groups to a summary: "🍳 Recipes · 38 saves" — reinforcing what the user is looking at.

**Active filter chip styling:** Filled background color matching brand palette. Inactive chips have a subtle outline only. High contrast between active and inactive states.

### 3.7 Search Experience

Tapping the search bar transitions to a full-screen search overlay:

**State A: Search focused, empty query**
```
┌─────────────────────────────────────────┐
│ ┌─────────────────────────────────────┐ │
│ │  🔍  |                     Cancel  │ │
│ └─────────────────────────────────────┘ │
│                                         │
│  Recent Searches                        │
│  ┌─────────────────────────────────┐   │
│  │ 🕐 paneer recipe               │   │
│  │ 🕐 bali beaches                │   │
│  │ 🕐 standing desk               │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Suggested Filters                      │
│  ┌──────────┐ ┌──────────┐            │
│  │ 🍳 In    │ │ 📅 This  │            │
│  │ Recipes  │ │ Week     │            │
│  └──────────┘ └──────────┘            │
│                                         │
└─────────────────────────────────────────┘
```

**State B: Typing query (live results)**
```
┌─────────────────────────────────────────┐
│ ┌─────────────────────────────────────┐ │
│ │  🔍  pasta lemon         ✕ Cancel  │ │
│ └─────────────────────────────────────┘ │
│                                         │
│  3 results                              │
│                                         │
│  ┌──────────┐ ┌──────────┐            │
│  │ [thumb]  │ │ [thumb]  │            │
│  │ 🍳       │ │ 🍳       │            │
│  │ Lemon    │ │ Pasta    │            │
│  │ **Pasta**│ │ Aglio e  │            │
│  │          │ │ Olio     │            │
│  └──────────┘ └──────────┘            │
│                                         │
│  ┌──────────┐                          │
│  │ [thumb]  │                          │
│  │ 🍳       │                          │
│  │ **Lemon**│                          │
│  │ Chicken  │                          │
│  └──────────┘                          │
└─────────────────────────────────────────┘
```

**Search scope:** Searches across all of the following for each save:
- Caption text
- Extracted transcript (if deep-extracted)
- Structured data fields (ingredient names, location names, product names)
- User-added notes
- Creator handle / username

**Search behavior:**
- Results appear as-you-type after 2+ characters (debounce 300ms)
- "Cancel" button dismisses search overlay and returns to library
- "✕" in search field clears the query but stays in search mode
- Recent searches persisted (last 10), swipeable to delete individual entries
- Search highlight: matching terms shown in **bold** in card titles
- Empty search results: "No saves match 'sushi'. Try different keywords." + "Clear Filters" if any filter is active.
- Tapping a search result navigates to the full card detail view (F4)

### 3.8 Card Interactions

#### Tap
Opens the full card detail view (see F4: Structured Data Cards). Card thumbnail animates with a hero transition (thumbnail zooms to fill the detail view header) for spatial continuity.

#### Long Press
Shows context menu (iOS: native UIContextMenuInteraction, Android: Material popup menu):

```
┌─────────────────────────┐
│  🔗 View Original       │
│  📁 Add to Collection   │
│  🏷️ Recategorize        │
│  📝 Edit Note           │
│  📤 Share               │
│  🗑️ Delete              │
└─────────────────────────┘
```

Context menu appears alongside a scaled-up preview of the card for visual context (iOS native behavior).

#### Multi-Select Mode

Long-pressing for 0.5 seconds activates multi-select mode (distinct from the instant long-press context menu — long-press shows context menu, but a toolbar button or a two-finger tap activates multi-select):

**Entry:** Toolbar "Select" button (top-right, appears in place of the view toggle icon when entering edit mode), or iOS-style two-finger drag.

```
┌─────────────────────────────────────────┐
│ 3 selected             Done   Select All│
│                                         │
│ ┌──────────┐ ┌──────────┐              │
│ │ ✅       │ │          │              │
│ │ [thumb]  │ │ [thumb]  │              │
│ │   🍳    │ │   ✈️    │              │
│ ├──────────┤ ├──────────┤              │
│ │ Garlic   │ │ Hidden   │              │
│ │ Pasta    │ │ Bali     │              │
│ └──────────┘ └──────────┘              │
│                                         │
│ ┌──────────┐ ┌──────────┐              │
│ │ ✅       │ │ ✅       │              │
│ │ [thumb]  │ │ [thumb]  │              │
│ │   🛍️    │ │   💪    │              │
│ ├──────────┤ ├──────────┤              │
│ │ Amazon   │ │ Full     │              │
│ │ Find     │ │ Body     │              │
│ └──────────┘ └──────────┘              │
│                                         │
│ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐  │
│ │📁 Add│ │🏷️ Cat│ │🔍 Ext│ │🗑️ Del│  │
│ │to Col│ │egorze│ │ract  │ │ete   │  │
│ └──────┘ └──────┘ └──────┘ └──────┘  │
└─────────────────────────────────────────┘
```

**Multi-select toolbar actions:**
- **📁 Add to Collection** — opens collection picker, adds all selected saves (F6)
- **🏷️ Categorize** — opens category picker, recategorizes all selected saves
- **🔍 Extract** — batch deep-extract all selected saves (F5)
- **🗑️ Delete** — deletes all selected with single undo opportunity

**Why multi-select matters:** It's a prerequisite for Collections (F6), Grocery List (F9), and Export (F13). Without it, users are forced to act on saves one at a time — unacceptable for power users managing 100+ saves.

#### Swipe Actions (List View Only)

Quick actions via swipe left on a list row:

```
┌─────────────────────────────────────────────────────┐
│ [thumb] Garlic Pasta...     ┌──────────┐┌──────────┐│
│                             │ 🏷️ Recatg││ 🗑️ Delete││
│                             └──────────┘└──────────┘│
└─────────────────────────────────────────────────────┘
```

Undo available for 5 seconds after delete. Undo toast anchored to bottom of screen, above tab bar.

### 3.9 Empty States

#### First Launch (No Saves)

```
┌─────────────────────────────────────┐
│                                     │
│       ┌──────────────────┐         │
│       │                  │         │
│       │  [Animated GIF   │         │
│       │   showing share  │         │
│       │   sheet flow]    │         │
│       │                  │         │
│       └──────────────────┘         │
│                                     │
│   Save your first Reel             │
│                                     │
│   Tap Share on any Instagram Reel, │
│   TikTok, or YouTube Short and    │
│   choose "Save to Staq."          │
│                                     │
│   We'll do the rest.              │
│                                     │
└─────────────────────────────────────┘
```

**Why animated GIF over video:** Cheaper to produce, auto-loops, no play button needed, loads instantly, accessible (can include alt text), and can be updated easily when the share sheet UI changes across OS versions.

**No CTA button needed** — the user can't save from within Staq. The empty state educates them about the share sheet flow they'll use from other apps.

#### No Search Results

```
┌─────────────────────────────────────┐
│                                     │
│     No saves match "sushi"         │
│                                     │
│  Try different keywords, or check  │
│  if a category filter is active.   │
│                                     │
│  ┌───────────────────────────┐     │
│  │  Clear All Filters        │     │
│  └───────────────────────────┘     │
└─────────────────────────────────────┘
```

"Clear All Filters" is shown regardless of whether filters are active — it's harmless when no filters are on, and saves the user from wondering whether they've accidentally filtered out results.

#### Filtered Category Empty

```
┌─────────────────────────────────────┐
│                                     │
│  No ✈️ Travel saves yet            │
│                                     │
│  Next time you see a travel Reel,  │
│  share it to Staq and it'll show   │
│  up right here.                    │
│                                     │
└─────────────────────────────────────┘
```

#### Last Save Deleted

If the user deletes their only remaining save, animate back to the first-launch empty state (fade transition). Don't show a jarring "0 results" screen.

### 3.10 Pull-to-Refresh

While the library updates automatically when new saves arrive, pull-to-refresh triggers:
- Re-process any pending/failed saves
- Sync with cloud (if multi-device in future)
- Re-run categorization on any "uncertain" saves
- Subtle haptic pulse when refresh starts

### 3.11 Accessibility

- **VoiceOver / TalkBack:** Grid cards announce: "Garlic Pasta. Recipe. Instagram. Saved 2 hours ago." in a single focus group. New-save badge announces "New. Not yet viewed."
- **Dynamic Type:** Card titles and metadata text scale with system font. Grid switches to single-column layout at the two largest accessibility text sizes to prevent truncation.
- **Reduce Motion:** Hero transitions between library and card detail replaced with crossfade. Shimmer skeleton replaced with static loading text.
- **Voice Control (iOS) / Switch Access (Android):** All interactive elements have stable accessibility labels. "Sort button", "Grid view toggle", "Recipe filter, 38 saves".
- **Minimum tap targets:** Filter chips ≥ 44×44pt with 8pt spacing between chips. Grid cards are inherently large enough.
- **High contrast mode:** Category badge overlay maintains 4.5:1 contrast ratio over any thumbnail via semi-opaque dark background.

---

## 4. Data & Performance

### 4.1 Local Storage

All saves are stored locally on device (Core Data on iOS, Room on Android). Cloud sync is future scope.

**Data model:**

```
SavedContent {
    id: UUID
    url: String
    platform: Platform
    captionText: String?
    transcriptText: String?          // If deep-extracted
    thumbnailLocalPath: String?      // Cached thumbnail
    thumbnailUrl: String?
    category: String
    categoryConfidence: Double
    richnessScore: Double
    extractedData: JSON              // Structured extraction
    userNote: String?
    userCategoryOverride: String?
    collectionIds: [UUID]
    creatorHandle: String?
    createdAt: Date
    processedAt: Date?
    status: SaveStatus
}
```

### 4.2 Performance Requirements

| Metric | Target |
|--------|--------|
| Library load time | < 500ms for first 20 items |
| Scroll performance | 60fps smooth, no jank |
| Search latency | < 100ms for local text search |
| Thumbnail loading | Progressive (blur placeholder → sharp) |
| Memory footprint | < 150MB with 1000+ saves loaded |

### 4.3 Pagination & Virtualization

- Use LazyVGrid (SwiftUI) / LazyVerticalGrid (Compose) for virtualized rendering
- Load thumbnails on demand (SDWebImage on iOS, Coil on Android)
- Paginate database queries: load 50 items at a time, fetch more on scroll
- Search uses FTS5 (SQLite full-text search) via Core Data/Room for sub-100ms results

---

## 5. Edge Cases

| Scenario | Handling |
|----------|----------|
| User has 5,000+ saves | Virtualized grid, paginated DB queries (50 at a time), search is FTS-indexed. Show total count in header: "Library · 5,234 saves" |
| Thumbnail fails to load | Show platform-colored gradient background (Instagram purple, TikTok pink, YouTube red) with platform icon centered. Never show a broken-image icon. |
| Save is still processing | Show skeleton card with shimmer animation and "Organizing..." label. Card transitions in-place when complete. |
| Multiple saves arrive simultaneously | Animate new cards sliding into top of grid sequentially with 100ms stagger. Show count badge on Library tab if user is on another tab: "📚③" |
| User switches category filter rapidly | Debounce filter changes (100ms), cancel previous DB queries. Show active filter instantly (optimistic UI). |
| Device low on storage | Purge cached thumbnails (re-download on demand), show subtle "Storage low" banner with option to clear cache in Settings. |
| Caption text is extremely long | Truncate to 1 line in grid, 2 lines in list, show full text in detail view (F4). |
| User deletes their last save | Animate back to first-launch empty state. |
| User rotates device (iPad / large phone) | Grid adapts: 2 columns portrait → 3–4 columns landscape. List view remains single-column with wider content. |
| User has saves but all are in one category | Filter bar still shows all categories but with count of 0. Zero-count chips are slightly dimmed but not hidden (user may want to see what categories exist). |
| Search returns results from many categories | Show category count summary: "3 results: 2 Recipes, 1 Travel" above the grid for scannability. |
| Deep link into library with filter | Support URL scheme: `staq://library?category=recipe` — opens library pre-filtered. Used by widget, notifications, and F1 share extension's "View Save" action. |

---

## 6. Development Plan

### 6.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Design local database schema (Core Data / Room) | Both | 4h | — |
| 2 | Build FTS5 full-text search index | Both | 4h | Task 1 |
| 3 | Build grid view with virtualized rendering | iOS | 6h | Task 1 |
| 4 | Build grid view with virtualized rendering | Android | 6h | Task 1 |
| 5 | Implement category filter bar (scrollable chips, single-select) | Both | 4h | Task 1 |
| 6 | Implement search overlay UI + live results with highlight | Both | 8h | Task 2 |
| 7 | Build date-grouped section headers with smart grouping | Both | 3h | Task 3/4 |
| 8 | Implement thumbnail caching + progressive loading | Both | 4h | Task 3/4 |
| 9 | Build long-press context menu (native per platform) | Both | 3h | Task 3/4 |
| 10 | Build list view with swipe-to-delete + recategorize | Both | 4h | Task 3/4 |
| 11 | Build multi-select mode with batch toolbar | Both | 6h | Task 3/4 |
| 12 | Build empty states (first launch, no results, empty category, last deleted) | Both | 3h | Task 3/4 |
| 13 | Build sort options bottom sheet + persistence | Both | 3h | Task 1 |
| 14 | Build grid/list view toggle with layout animation | Both | 2h | Tasks 3/4, 10 |
| 15 | Build "NEW" badge for unseen saves | Both | 2h | Task 3/4 |
| 16 | Implement hero transition from grid to card detail | Both | 3h | Task 3/4, F4 |
| 17 | Connect Share Extension pipeline → Library auto-refresh | Both | 3h | F1, F2 |
| 18 | Build deep link handler (staq://library?category=X) | Both | 2h | Task 5 |
| 19 | Performance optimization (pagination, memory, FTS) | Both | 4h | All above |
| 20 | Accessibility audit (VoiceOver/TalkBack, Dynamic Type, Reduce Motion) | Both | 4h | All above |
| 21 | QA: scroll performance, search accuracy, edge cases, multi-select | QA | 8h | All above |

**Total: ~86 hours (~3 weeks with one developer per platform)**

### 6.2 Definition of Done

- [ ] Library displays all saved content in a 2-column grid with thumbnails
- [ ] Category filter bar filters content with zero perceived latency (single-select toggle)
- [ ] Search overlay returns results as-you-type across all text fields with bold highlight
- [ ] FTS search works with 1000+ saves in < 100ms
- [ ] Grid scrolls at 60fps with no jank on iPhone 13 / Pixel 7
- [ ] Thumbnail progressive loading works (blur → sharp) with platform-color fallback
- [ ] Long-press context menu works with all 6 actions (native per platform)
- [ ] List view works with swipe-to-delete + undo (5 sec)
- [ ] Multi-select mode works with batch add-to-collection, recategorize, extract, delete
- [ ] Sort options persist across sessions and correctly regroup content
- [ ] Empty states display correctly for all 4 scenarios (new, no results, empty category, last deleted)
- [ ] New saves from Share Extension appear in library without manual refresh with "NEW" badge
- [ ] Grid/list view toggle works with layout transition animation
- [ ] Hero transition from grid card to detail view is smooth
- [ ] Accessibility: VoiceOver/TalkBack navigation works, Dynamic Type scales correctly, Reduce Motion respected
