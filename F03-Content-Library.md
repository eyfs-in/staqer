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
| US-3.7 | As a user, I want to paste a URL directly into the app without using the share sheet | Quick Save FAB with URL paste support |

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

**Platform tab bar implementation:**

- **iOS:** `TabView` with `.tabViewStyle(.tabBarOnly)` using `@State` selection binding. Tab items use SF Symbols with `.environment(\.symbolVariants, .fill)` for the active state and `.none` for inactive. Tab bar uses the system `UITabBar` appearance with a thin top separator and standard material blur background.
- **Android:** Material 3 `NavigationBar` composable inside a `Scaffold` `bottomBar` slot. Each tab is a `NavigationBarItem` with filled icon for selected state and outlined for unselected. Follow M3 elevation and tonal color defaults.

**Haptic feedback on tab switch:** Each tab selection fires a subtle haptic — `UIImpactFeedbackGenerator(style: .light)` on iOS, `HapticFeedbackType.LongPress` via `LocalHapticFeedback` on Android (Compose). This is a micro-interaction that confirms the tap without being distracting. Disable when the system "Reduce Haptics" / "Touch vibration" accessibility setting is off.

**Why no standalone Search tab:** Search is embedded at the top of the Library screen because search is always contextual to the saves the user is looking at. A separate tab adds navigation friction to the most common action. When the user taps the search bar, the full search experience (recent searches, filters, results) takes over the screen as an overlay.

#### Quick Save FAB (Floating Action Button)

A floating action button sits in the bottom-right corner of the Library screen, above the tab bar, providing a direct "paste a URL" entry point.

```
┌─────────────────────────────────────────┐
│                                         │
│            [ Library content ]          │
│                                         │
│                                         │
│                                ┌──────┐ │
│                                │  +   │ │
│                                │ Paste │ │
│                                └──────┘ │
│ ┌─────┐   ┌─────┐   ┌─────┐          │
│ │  📚 │   │  📁 │   │  ⚙️ │          │
│ │Libry│   │Colls│   │ Set │          │
│ └─────┘   └─────┘   └─────┘          │
└─────────────────────────────────────────┘
```

**Why a Quick Save FAB:**
- Share sheet doesn't always work reliably, particularly in some browsers, Twitter/X, and Reddit where the share extension may not appear.
- Users frequently copy a URL to their clipboard and want to save it without leaving the app.
- Reduces friction for the "I copied a link, now I want to save it" flow — one tap instead of switching apps.

**Behavior:**
1. **Tap:** Opens a compact bottom sheet with a text field pre-populated from the clipboard if a valid URL is detected. Shows a "Save" primary button.
2. **Clipboard detection:** On the bottom sheet appearing, check `UIPasteboard.general` (iOS) / `ClipboardManager` (Android) for a URL. If found, pre-fill the field and show the domain name as a subtitle confirmation ("instagram.com" / "tiktok.com"). If no URL is on the clipboard, show the empty field with placeholder text "Paste or type a URL."
3. **On save:** Dismiss the sheet, trigger the same ingestion pipeline as the share extension (F1 + F2), and show an inline toast: "Saving... 🍳 Garlic Pasta" once categorization completes.
4. **Validation:** Reject non-URL input with inline error: "That doesn't look like a link. Try pasting a URL from Instagram, TikTok, or YouTube."

**iOS clipboard privacy:** iOS 16+ shows a system paste permission prompt on clipboard access. To avoid this firing on every FAB tap, only read the clipboard when the user explicitly taps a "Paste from Clipboard" chip inside the bottom sheet (uses `UIPasteboard.general.detectPatterns(for:)` to check for URLs without triggering the prompt, then reads the value only on user action). On Android, no special permission is needed.

**Platform implementation:**
- **iOS:** Rendered as a `.overlay` on the `ZStack` containing the library content. Positioned with `.padding(.bottom, 80)` to sit above the tab bar. Uses a circular shape with a plus icon, drop shadow, and brand accent fill. On scroll-down, the FAB shrinks to a mini-FAB (icon only) with a spring animation to reduce visual clutter.
- **Android:** Standard Material 3 `FloatingActionButton` placed in the `Scaffold`'s `floatingActionButton` slot. Uses `FloatingActionButtonDefaults.elevation()` and the `containerColor` from the app theme. Use `AnimatedVisibility` with `slideInVertically` to auto-hide when the user is scrolling down quickly (via `LazyListState.isScrollingDown()`), and reappear on scroll-up or idle.

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
│                                ┌──────┐ │
│                                │  +   │ │
│                                └──────┘ │
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
- 2-column uniform grid with thumbnails (default), with option to switch to staggered/masonry layout (see below)
- Category badge overlay on thumbnail (bottom-right corner, frosted glass background)
- Below thumbnail: title (1 line, max), key detail (1 line), platform icon + relative time
- Grouped by date section headers ("Today", "Yesterday", "This Week", "January 2026")

**Thumbnail aspect ratio handling:**

Content saved from different platforms arrives with wildly different native aspect ratios:
- Instagram posts: 1:1 (square) or 4:5 (portrait)
- Instagram Reels / TikTok: 9:16 (tall portrait)
- YouTube thumbnails: 16:9 (landscape)
- Twitter/X link cards: ~2:1 (wide landscape)

In the **uniform grid** (default), all cells are the same height. We use **crop-to-fill** with smart center-crop:

- Each cell targets a **4:5 aspect ratio** (portrait-biased, which fits the majority of mobile content — Instagram posts, Reels, and TikTok all look natural here).
- The thumbnail is scaled to fill the cell frame and center-cropped. This means YouTube 16:9 thumbnails lose the left/right edges, and 9:16 content loses the top/bottom edges — but the visual center (where the subject usually is) remains visible.
- On iOS: `Image(uiImage:).resizable().scaledToFill().frame(...).clipped()` with `.contentMode(.scaleAspectFill)`.
- On Android: `AsyncImage` with `ContentScale.Crop` inside a `Modifier.aspectRatio(4f/5f).clip(shape)`.
- **Why crop-to-fill over fit-with-padding:** Fit-with-padding (letterboxing) creates visible bars around thumbnails and makes the grid look inconsistent and unfinished — especially when mixing landscape YouTube thumbnails next to portrait Reels. Crop-to-fill keeps the grid visually dense and uniform, which is the expectation users have from Pinterest, Instagram Explore, and Apple Photos.

**Staggered / Masonry grid (toggle):**

In addition to the uniform grid, users can switch to a staggered (Pinterest/masonry) layout via a third option in the view toggle (▦ uniform / ⊞ staggered / ▤ list):

```
┌──────────┐ ┌──────────┐
│          │ │          │
│ [thumb]  │ │ [thumb]  │
│ 16:9     │ │          │
│          │ │  9:16    │
├──────────┤ │          │
│ Garlic   │ │          │
│ Pasta    │ │          │
│ IG · 2h  │ ├──────────┤
└──────────┘ │ Hidden   │
┌──────────┐ │ Bali     │
│          │ │ TT · 3h  │
│ [thumb]  │ └──────────┘
│ 4:5      │ ┌──────────┐
│          │ │          │
│          │ │ [thumb]  │
├──────────┤ │ 1:1      │
│ Standing │ │          │
│ Desk     │ ├──────────┤
│ YT · 1d  │ │ Full     │
└──────────┘ │ Body     │
             │ YT · 1d  │
             └──────────┘
```

- In staggered mode, each thumbnail renders at its **native aspect ratio** (fit, no cropping), so the user sees the full image exactly as it appeared on the source platform. Cells in each column have different heights, creating the waterfall/masonry effect.
- This feels more premium and content-forward — it respects the original composition of each piece of content. Users who care about visual fidelity (photographers, designers) will prefer this.
- Column widths are equal; only heights vary based on image aspect ratio.
- Date section headers span the full width across both columns, interrupting the stagger — this keeps temporal grouping readable.

**Platform implementation for grid layouts:**

- **iOS (Uniform grid):** `LazyVGrid` with `columns: [GridItem(.flexible(), spacing: 8), GridItem(.flexible(), spacing: 8)]`. Each cell uses a fixed aspect ratio frame.
- **iOS (Staggered grid):** SwiftUI does not have a native masonry layout. Use a custom `Layout` protocol implementation (iOS 16+) or a two-column `HStack` of `LazyVStack`s with items distributed by cumulative height. Alternatively, wrap a `UICollectionView` with `UICollectionViewCompositionalLayout` using estimated item heights via `UIViewControllerRepresentable`.
- **Android (Uniform grid):** `LazyVerticalGrid(columns = GridCells.Fixed(2))` with each item in a `Modifier.aspectRatio(4f/5f)`.
- **Android (Staggered grid):** `LazyVerticalStaggeredGrid(columns = StaggeredGridCells.Fixed(2))` from `foundation-layout`. Each item uses `Modifier.aspectRatio(item.thumbnailAspectRatio)` to respect native dimensions. This composable handles uneven cell heights natively and is the official Jetpack Compose solution.

**View preference persistence:** The selected layout mode (uniform grid, staggered grid, or list) is stored in `UserDefaults` (iOS) / `DataStore` (Android) and restored on next launch.

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
│   — or —                           │
│                                     │
│   ┌─────────────────────────────┐  │
│   │  📋 Paste a URL to save     │  │
│   └─────────────────────────────┘  │
│                                     │
└─────────────────────────────────────┘
```

**Why animated GIF over video:** Cheaper to produce, auto-loops, no play button needed, loads instantly, accessible (can include alt text), and can be updated easily when the share sheet UI changes across OS versions.

**CTA note:** The primary empty state educates about the share sheet flow. A secondary "Paste a URL" action is provided for users who copied a link before installing — it triggers the same bottom sheet as the Quick Save FAB.

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

All saves are stored locally on device (Core Data on iOS, Room on Android). Cloud sync is future scope (see Section 7: Offline-First Architecture).

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
    thumbnailAspectRatio: Double?    // Width / Height (e.g., 0.5625 for 9:16)
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
| Thumbnail cache hit rate | > 95% for recently viewed items |

### 4.3 Pagination & Virtualization

- Use LazyVGrid / LazyVerticalStaggeredGrid (SwiftUI/Compose) for virtualized rendering
- Load thumbnails on demand (see 4.4 Thumbnail Caching Strategy)
- Paginate database queries: load 50 items at a time, fetch more on scroll
- Search uses FTS5 (SQLite full-text search) via Core Data/Room for sub-100ms results (see 4.5 Full-Text Search Setup)

**Reactive data binding:**

The library view subscribes to the underlying data store reactively, so that new saves from the share extension, deletions, and re-categorizations are reflected in the UI immediately without manual refresh logic.

- **iOS (Core Data):** Use `@FetchRequest` with a `SortDescriptor` and optional `NSPredicate` for the active category filter. SwiftUI re-renders the grid automatically when the managed object context changes. For more complex queries (FTS, joins), wrap a `NSFetchedResultsController` in an `ObservableObject` and publish changes via `@Published`.
- **Android (Room):** Expose queries as `Flow<List<SavedContent>>` from the DAO. Collect the flow in the ViewModel with `stateIn(scope, SharingStarted.WhileSubscribed(5000), emptyList())` and observe it in Compose via `collectAsStateWithLifecycle()`. Room automatically emits new values when the underlying table changes.

**Prefetching strategy:**

When the user scrolls, proactively fetch thumbnails for upcoming content to eliminate visible loading:

- **Prefetch window:** Always keep the next 2 pages (~100 thumbnails) of content queued for loading. As the user scrolls and consumes one page, enqueue the next page ahead.
- **iOS:** Implement `UICollectionViewDataSourcePrefetching` (if using UIKit bridge) or monitor `LazyVGrid` visible item indices via a `GeometryReader` or `onAppear` on sentinel items placed at page boundaries. When a sentinel appears, trigger the next page fetch from Core Data and enqueue thumbnail downloads.
- **Android:** Use `LazyListState` to observe `firstVisibleItemIndex`. When it crosses a page boundary threshold (e.g., within 20 items of the last loaded item), trigger the next page load from Room and enqueue thumbnail prefetches via Coil's `ImageRequest.Builder.memoryCachePolicy(CachePolicy.ENABLED)`.
- **Priority:** Prefetched thumbnails load at a lower priority than on-screen thumbnails. If the user scrolls fast (fling), cancel prefetch requests for items that have already scrolled out of the prefetch window.

### 4.4 Thumbnail Caching Strategy

Thumbnails are the single most important performance-sensitive asset in the library. A multi-layer caching strategy ensures that scrolling feels instant even with thousands of saves.

**Cache hierarchy (checked in order):**

```
┌─────────────────────────────────────────────────────┐
│ Layer 1: Memory LRU Cache                           │
│ - Hot thumbnails for visible + recently scrolled    │
│ - ~50MB budget (approx 200-300 thumbnails)          │
│ - Eviction: LRU, cleared on memory warning          │
│ - Access time: < 1ms                                │
├─────────────────────────────────────────────────────┤
│ Layer 2: Disk Cache                                 │
│ - All previously loaded thumbnails                  │
│ - ~500MB budget, configurable in Settings           │
│ - Stored in app Caches directory (OS can purge)     │
│ - Access time: 5-15ms (SSD read)                    │
├─────────────────────────────────────────────────────┤
│ Layer 3: Local File (thumbnailLocalPath)            │
│ - Thumbnails downloaded during initial save/ingest  │
│ - Stored in app Documents directory (persistent)    │
│ - Access time: 5-15ms                               │
├─────────────────────────────────────────────────────┤
│ Layer 4: Network Fetch (thumbnailUrl)               │
│ - Re-download from source if all caches miss        │
│ - Access time: 100-2000ms (network dependent)       │
│ - May fail if original content was deleted           │
└─────────────────────────────────────────────────────┘
```

**Platform libraries:**

- **iOS:** Use **Kingfisher** (`KFImage` in SwiftUI) for the full caching pipeline. Configure `ImageCache.default.memoryStorage.config.totalCostLimit` for the memory budget and `ImageCache.default.diskStorage.config.sizeLimit` for disk. Kingfisher handles LRU eviction, disk serialization, and progressive JPEG decoding out of the box. Alternative: **SDWebImage** via `SDWebImageSwiftUI` — equivalent capability, slightly larger API surface.
- **Android:** Use **Coil** (`AsyncImage` or `rememberAsyncImagePainter` in Compose). Configure `ImageLoader.Builder` with `memoryCache { MemoryCache.Builder(context).maxSizePercent(0.25).build() }` and `diskCache { DiskCache.Builder().directory(cacheDir).maxSizeBytes(500L * 1024 * 1024).build() }`. Coil is Kotlin-first, coroutine-native, and the recommended image loader for Compose.

**Thumbnail sizing:**

To avoid decoding full-resolution images for small grid cells, request downsampled thumbnails:
- Grid cell target size: ~180×225pt (at 3x = 540×675px). Request thumbnails at this resolution or the next step up.
- Kingfisher: use `.processor(DownsamplingImageProcessor(size: targetSize))`.
- Coil: use `size(540, 675)` in the `ImageRequest.Builder`.
- This reduces memory per thumbnail from ~2MB (full 1080p) to ~0.5MB, making the 50MB memory budget stretch to 100+ thumbnails.

### 4.5 Full-Text Search (FTS5) Setup

FTS5 is SQLite's built-in full-text search engine. It creates an inverted index that enables sub-100ms keyword search across all text content, even with tens of thousands of saves.

**Virtual table definition:**

```sql
CREATE VIRTUAL TABLE saved_content_fts USING fts5(
    caption,
    transcript,
    extracted_fields,
    user_note,
    creator_handle,
    content='saved_content',
    content_rowid='rowid',
    tokenize='unicode61 remove_diacritics 2'
);
```

**Column breakdown:**
- `caption` — the caption text from the original post (Instagram caption, TikTok description, YouTube title + description)
- `transcript` — speech-to-text transcript from video/audio, populated by deep extraction (F5)
- `extracted_fields` — flattened text from structured extraction: ingredient names, location names, product names, exercise names, etc. Stored as a single concatenated string for FTS indexing (e.g., "garlic lemon pasta parmesan olive oil")
- `user_note` — any notes the user has manually added to a save
- `creator_handle` — the username/handle of the content creator (e.g., "@halfbakedharvest")

**Tokenizer choice (`unicode61 remove_diacritics 2`):**
- `unicode61` — Unicode-aware tokenizer that handles non-ASCII characters correctly. Essential for international content (accented characters, CJK, etc.).
- `remove_diacritics 2` — normalizes accented characters so that searching "creme brulee" also matches "creme brulee" (and vice versa). Mode `2` removes diacritics only for ASCII-equivalent characters, preserving non-Latin scripts.

**Keeping the FTS index in sync:**

Use SQLite triggers to keep the FTS index updated whenever the main table changes:

```sql
-- Insert trigger
CREATE TRIGGER saved_content_ai AFTER INSERT ON saved_content BEGIN
    INSERT INTO saved_content_fts(rowid, caption, transcript, extracted_fields, user_note, creator_handle)
    VALUES (new.rowid, new.caption_text, new.transcript_text, new.extracted_fields_text, new.user_note, new.creator_handle);
END;

-- Update trigger
CREATE TRIGGER saved_content_au AFTER UPDATE ON saved_content BEGIN
    INSERT INTO saved_content_fts(saved_content_fts, rowid, caption, transcript, extracted_fields, user_note, creator_handle)
    VALUES ('delete', old.rowid, old.caption_text, old.transcript_text, old.extracted_fields_text, old.user_note, old.creator_handle);
    INSERT INTO saved_content_fts(rowid, caption, transcript, extracted_fields, user_note, creator_handle)
    VALUES (new.rowid, new.caption_text, new.transcript_text, new.extracted_fields_text, new.user_note, new.creator_handle);
END;

-- Delete trigger
CREATE TRIGGER saved_content_ad AFTER DELETE ON saved_content BEGIN
    INSERT INTO saved_content_fts(saved_content_fts, rowid, caption, transcript, extracted_fields, user_note, creator_handle)
    VALUES ('delete', old.rowid, old.caption_text, old.transcript_text, old.extracted_fields_text, old.user_note, old.creator_handle);
END;
```

**Platform integration:**
- **iOS (Core Data):** Core Data does not natively expose FTS5. Execute the `CREATE VIRTUAL TABLE` and trigger SQL directly on the underlying SQLite store via `NSPersistentStoreCoordinator`'s SQLite connection during the Core Data stack setup (in a migration step or on first launch). Query the FTS table with raw SQL through a `NSFetchRequest` with `NSFetchRequestResultType.dictionaryResultType` or by opening a direct SQLite handle alongside Core Data.
- **Android (Room):** Room supports FTS4 via `@Fts4` annotation but not FTS5 natively. Use a `RoomDatabase.Callback` in `onCreate`/`onOpen` to execute the raw FTS5 `CREATE VIRTUAL TABLE` SQL. Query via `@RawQuery` or a `@Query` annotation with the FTS `MATCH` syntax: `SELECT * FROM saved_content_fts WHERE saved_content_fts MATCH :query`.

**Query syntax for the search bar:**

```sql
-- User types "pasta lemon"
SELECT sc.* FROM saved_content sc
JOIN saved_content_fts fts ON sc.rowid = fts.rowid
WHERE saved_content_fts MATCH 'pasta* lemon*'
ORDER BY rank
LIMIT 50;
```

The `*` suffix enables prefix matching so partial words work ("pas" matches "pasta"). The `rank` column is FTS5's built-in BM25 relevance score — results are ordered by relevance by default.

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
| User switches between light/dark mode | Thumbnail category badge overlays and the "NEW" dot use semantic colors (`label` / `systemBackground` on iOS, `MaterialTheme.colorScheme` on Android) that adapt automatically. The frosted glass badge background must maintain its 4.5:1 contrast ratio in both modes — test with both light thumbnails (white food on white plate) and dark thumbnails (night cityscape). Platform-colored fallback gradients (see "Thumbnail fails to load") should have light and dark variants defined in the asset catalog / theme resources. |
| User has 0 saves in a custom category | Custom categories created by the user (via recategorize or "More" sheet) should persist even when empty. Show the custom category chip in the filter bar (dimmed, count 0), and when tapped, display a tailored empty state: "No saves in [Custom Category] yet. Save or recategorize content to add it here." Include a CTA: "Browse All Saves" that returns to the "All" filter. This prevents confusion when users set up categories before filling them. |
| iPad / tablet layout | On devices with horizontal size class `.regular` (iPad, large Android tablets): expand to a **3-column grid** in portrait and **4-column grid** in landscape. Consider a sidebar navigation option (iOS `NavigationSplitView` / Android `NavigationRail`) where the left sidebar replaces the bottom tab bar and shows Library, Collections, and Settings as a vertical list. The category filter bar moves into the sidebar below the navigation items when sidebar is visible. Content area fills the remaining width. On Android foldables (e.g., Pixel Fold), respect `WindowSizeClass` from `material3-window-size-class` and adapt column count dynamically based on unfolded width. |
| Landscape mode on phone | On phones in landscape: grid expands to 3 columns (uniform or staggered) to use horizontal space. The category filter bar remains horizontally scrollable below the search field. The tab bar and Quick Save FAB remain anchored to the bottom. On iOS, respect safe area insets for the Dynamic Island and home indicator. On Android, handle edge-to-edge with `WindowInsets` to avoid content behind the navigation bar. The search overlay should expand to use the full landscape width rather than constraining to a portrait-width column. |

---

## 6. Offline-First Architecture

### 6.1 Core Principle

Staq is local-first. Every feature works fully offline with zero latency. The app never shows a "No connection" error or a loading spinner waiting on a network call for core library functionality. Cloud sync is additive and optional — it enhances the experience (multi-device) but is never required.

### 6.2 Local Data Stack

**iOS — Core Data with SQLite:**
- `NSPersistentContainer` with a single SQLite store in the app's Application Support directory.
- Schema defined with `.xcdatamodeld` model file, versioned with lightweight migrations for schema evolution.
- Write context: a private-queue `NSManagedObjectContext` for all writes (share extension ingestion, recategorization, deletion). Merges to the view context automatically via `automaticallyMergesChangesFromParent = true`.
- Read context: the `viewContext` (main queue) for UI binding via `@FetchRequest`.
- Future CloudKit sync: `NSPersistentCloudKitContainer` is a drop-in replacement for `NSPersistentContainer`. When enabled, it mirrors the local store to the user's private CloudKit database transparently. No application-level sync code needed for the basic case.

**Android — Room with SQLite:**
- Room database with `@Entity` classes mapping to the `SavedContent` schema.
- DAOs expose `Flow<List<SavedContent>>` for reactive reads and `suspend fun` for writes.
- Database file stored in the app's internal storage (`context.getDatabasePath()`).
- Future Supabase sync: implement a `SyncManager` that observes Room's `InvalidationTracker` for table-level change notifications and pushes deltas to Supabase Realtime. Pull changes via Supabase's Postgres Changes subscription. This is future scope and does not affect the Phase 1 architecture.

### 6.3 SQLite WAL Mode

Both platforms enable **Write-Ahead Logging (WAL)** mode on the SQLite database for concurrent read/write performance:

```sql
PRAGMA journal_mode=WAL;
```

**Why WAL matters for Staq:**
- The share extension writes new saves to the database while the library UI is reading from it. Without WAL, these operations would block each other (SQLite's default journal mode uses exclusive locks for writes).
- WAL allows readers and a single writer to operate concurrently without blocking. The UI thread reads from a consistent snapshot while the share extension writes in the background.
- WAL also improves write performance (sequential writes to the WAL file instead of random writes to the main DB).

**Platform setup:**
- **iOS:** Core Data enables WAL by default on modern iOS. No additional configuration needed. Verify with `PRAGMA journal_mode;` query during development.
- **Android:** Room enables WAL by default on API 16+. Alternatively, force it explicitly with `RoomDatabase.Builder.setJournalMode(RoomDatabase.JournalMode.WRITE_AHEAD_LOGGING)`.

### 6.4 Conflict Resolution (Future Multi-Device Sync)

When cloud sync is enabled in a future phase, conflicts will arise when the same save is edited on two devices before syncing. The resolution strategy:

**Last-write-wins with user-edit priority:**
- Each record carries a `lastModifiedAt` timestamp (wall clock) and a `lastModifiedDevice` identifier.
- For **system-generated fields** (category confidence, richness score, extracted data): last-write-wins by timestamp. These fields are deterministic and re-derivable, so losing one version is not destructive.
- For **user-edited fields** (user note, user category override, collection membership): user edits always win over system-generated changes. If both devices have user edits, last-write-wins by timestamp, but the "losing" edit is preserved in a `conflictHistory` array on the record so the user can recover it manually if needed.
- **Deletes:** A deleted record is soft-deleted (tombstoned) with a `deletedAt` timestamp. Tombstones are synced to other devices. A delete always wins over a non-user edit, but if Device A deletes a save and Device B adds a user note to it before syncing, the user is prompted: "You deleted this save on another device, but you also added a note. Keep or discard?"

This strategy avoids silent data loss for the content users care most about (their own notes and organization) while keeping system metadata conflict resolution simple and automatic.

---

## 7. Widget Support (Phase 2 Preview)

> **Note:** Widget support is Phase 2 scope. This section documents the design intent so that Phase 1 architecture decisions (data model, deep link scheme, thumbnail caching) remain forward-compatible.

### 7.1 iOS — WidgetKit

**Small Widget (2x2): "Recent Saves"**

```
┌──────────────────────┐
│ ┌────┐ ┌────┐       │
│ │ 🍳 │ │ ✈️ │  Staq │
│ │    │ │    │       │
│ ├────┤ ├────┤       │
│ │ 🛍️ │ │ 💪 │       │
│ │    │ │    │       │
│ └────┘ └────┘       │
└──────────────────────┘
```

- Shows the last 4 saves as a 2x2 thumbnail grid with category emoji overlay.
- Tapping any thumbnail deep links to that specific card: `staq://card/{id}`.
- Tapping the "Staq" label opens the Library at the top.
- Data provided via a `TimelineProvider` that reads from the shared Core Data store (accessed via an App Group container).
- Timeline refresh: `.atEnd` policy — refresh when the user adds a new save. Also request a timeline reload from the share extension via `WidgetCenter.shared.reloadAllTimelines()` after each new save.

**Medium Widget (4x2): "Recent + Category Breakdown"**

```
┌──────────────────────────────────────┐
│ ┌────┐ ┌────┐ ┌────┐   📊 Staq    │
│ │    │ │    │ │    │              │
│ │ 🍳 │ │ ✈️ │ │ 🛍️ │  🍳 38  ▓▓▓░│
│ │    │ │    │ │    │  ✈️ 15  ▓░░░│
│ └────┘ └────┘ └────┘  🛍️ 22  ▓▓░░│
│  Garlic   Hidden  Amazon         │
│  Pasta    Bali    Find           │
└──────────────────────────────────────┘
```

- Left half: last 3 saves as thumbnails with title below.
- Right half: top 3 categories with save count and a tiny inline bar chart showing relative proportion.
- Tapping a thumbnail deep links to that card. Tapping a category bar deep links to the library filtered by that category: `staq://library?category=recipe`.

**Implementation notes:**
- Widget views are built with SwiftUI (required by WidgetKit) — reuse the same thumbnail caching and card model from the main app.
- Shared Core Data store via App Group: the widget extension reads from the same persistent store as the main app. Use `NSPersistentContainer(name:)` with the shared container URL.
- Keep widget rendering lightweight: pre-cache 4 downsampled thumbnails (120x150px) in the App Group container whenever a new save is added, so the widget never needs to decode full-size images.

### 7.2 Android — Jetpack Glance

**Small Widget: "Recent Saves"**

Equivalent to the iOS small widget — 2x2 thumbnail grid with category overlays.

- Built with `GlanceAppWidget` and `GlanceAppWidgetReceiver`.
- Layout uses `LazyVerticalGrid` (Glance variant) or a `Row`/`Column` composition for the 2x2 grid.
- Thumbnails loaded as `Bitmap` from the disk cache (Coil's disk cache directory) and displayed via `Image(provider = ImageProvider(bitmap))`.
- Tapping a cell uses `actionStartActivity` with an `Intent` containing the deep link URI: `staq://card/{id}`.

**Medium Widget: "Recent + Category Breakdown"**

Equivalent to the iOS medium widget — recent saves + category bar chart.

- Same Glance architecture. Category data queried from Room via a `CoroutineWorker` or `GlanceAppWidget.provideGlance` suspend function.
- Widget updates triggered by Room's `InvalidationTracker` — when the `saved_content` table changes, schedule a Glance widget update via `GlanceAppWidgetManager.requestPinAppWidget()` or `AppWidgetManager.notifyAppWidgetViewDataChanged()`.

### 7.3 Deep Link Handling

Widgets rely on the deep link scheme already defined in Section 5:

- `staq://library?category={category}` — opens Library pre-filtered
- `staq://card/{id}` — opens the full card detail view for a specific save (F4)

The deep link handler in the main app must support cold launch (app not running) and warm resume (app in background). On iOS, handle via `onOpenURL` in the root `App` struct. On Android, handle via `NavDeepLink` in the Jetpack Navigation graph or `intent-filter` in the `AndroidManifest.xml`.

---

## 8. Analytics Events

All analytics events are fired locally and queued for batch upload. They use a lightweight event bus (iOS: `NotificationCenter` or a dedicated `AnalyticsService` singleton; Android: a `SharedFlow` collected by an `AnalyticsRepository`). No third-party SDK dependency in Phase 1 — events are logged to a local SQLite table and can be forwarded to Amplitude, Mixpanel, or a custom backend in Phase 2.

| Event Name | Properties | Trigger |
|------------|------------|---------|
| `library_opened` | `entry_point` (app_launch / tab_switch / deep_link / widget), `save_count` (total saves in library) | Library screen appears |
| `category_filter_applied` | `category` (category name), `result_count` (saves matching filter) | User taps a category chip |
| `search_performed` | `query_length` (character count), `result_count` (matches found), `had_filters` (bool — was a category filter active during search) | User types 2+ chars and results are displayed (debounced, fire once per settled query) |
| `card_tapped` | `category` (card's category), `richness_score` (0.0–1.0), `position_in_list` (0-indexed position in current view) | User taps a card to open detail view |
| `view_toggled` | `from_view` (uniform_grid / staggered_grid / list), `to_view` (uniform_grid / staggered_grid / list) | User changes layout mode |
| `sort_changed` | `sort_type` (recent / oldest / category / platform / richness) | User selects a sort option |
| `multi_select_action` | `action_type` (add_to_collection / recategorize / extract / delete), `item_count` (number of items selected) | User performs a batch action in multi-select mode |
| `swipe_action` | `action_type` (delete / recategorize) | User completes a swipe action in list view |
| `quick_save_initiated` | `had_clipboard_url` (bool — was a URL auto-detected on clipboard), `source_domain` (domain of pasted URL, e.g., "instagram.com") | User taps the Quick Save FAB and the bottom sheet appears |
| `quick_save_completed` | `source_domain`, `category_assigned` (auto-categorization result) | URL is accepted and ingestion pipeline starts |

**Privacy considerations:**
- Never log the full URL, caption text, or any user-generated content in analytics. Only log structural metadata (counts, categories, positions, domains).
- All events respect the user's analytics opt-out toggle in Settings.
- Events are purged from the local queue after 30 days if not uploaded.

---

## 9. Development Plan

### 9.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Design local database schema (Core Data / Room) | Both | 4h | — |
| 2 | Build FTS5 full-text search index with triggers | Both | 6h | Task 1 |
| 3 | Build uniform grid view with virtualized rendering | iOS | 6h | Task 1 |
| 4 | Build uniform grid view with virtualized rendering | Android | 6h | Task 1 |
| 5 | Build staggered/masonry grid layout | iOS | 5h | Task 3 |
| 6 | Build staggered/masonry grid layout | Android | 3h | Task 4 |
| 7 | Implement category filter bar (scrollable chips, single-select) | Both | 4h | Task 1 |
| 8 | Implement search overlay UI + live results with highlight | Both | 8h | Task 2 |
| 9 | Build date-grouped section headers with smart grouping | Both | 3h | Task 3/4 |
| 10 | Implement multi-layer thumbnail caching (memory/disk/network) | Both | 5h | Task 3/4 |
| 11 | Implement thumbnail prefetching (next 2 pages) | Both | 3h | Task 10 |
| 12 | Build long-press context menu (native per platform) | Both | 3h | Task 3/4 |
| 13 | Build list view with swipe-to-delete + recategorize | Both | 4h | Task 3/4 |
| 14 | Build multi-select mode with batch toolbar | Both | 6h | Task 3/4 |
| 15 | Build empty states (first launch, no results, empty category, last deleted, empty custom category) | Both | 4h | Task 3/4 |
| 16 | Build sort options bottom sheet + persistence | Both | 3h | Task 1 |
| 17 | Build view toggle (uniform/staggered/list) with layout animation | Both | 3h | Tasks 3–6, 13 |
| 18 | Build "NEW" badge for unseen saves | Both | 2h | Task 3/4 |
| 19 | Implement hero transition from grid to card detail | Both | 3h | Task 3/4, F4 |
| 20 | Connect Share Extension pipeline → Library auto-refresh | Both | 3h | F1, F2 |
| 21 | Build Quick Save FAB + URL paste bottom sheet | Both | 4h | F1, F2 |
| 22 | Build deep link handler (staq://library, staq://card) | Both | 3h | Task 7 |
| 23 | Implement analytics event logging | Both | 3h | All above |
| 24 | Enable WAL mode + verify concurrent read/write | Both | 1h | Task 1 |
| 25 | iPad/tablet adaptive layout (3-4 column grid, sidebar nav) | Both | 5h | Tasks 3-6 |
| 26 | Landscape mode handling | Both | 2h | Tasks 3-6, 25 |
| 27 | Light/dark mode contrast audit for badges and overlays | Both | 2h | Tasks 3/4, 18 |
| 28 | Performance optimization (pagination, memory, FTS, prefetching) | Both | 4h | All above |
| 29 | Accessibility audit (VoiceOver/TalkBack, Dynamic Type, Reduce Motion) | Both | 4h | All above |
| 30 | QA: scroll performance, search accuracy, edge cases, multi-select, landscape, dark mode | QA | 10h | All above |

**Total: ~130 hours (~4 weeks with one developer per platform)**

### 9.2 Definition of Done

- [ ] Library displays all saved content in a 2-column grid with thumbnails
- [ ] Staggered/masonry grid layout available as alternate view option
- [ ] Thumbnails render correctly for all platform aspect ratios (1:1, 4:5, 9:16, 16:9) in both uniform and staggered grid
- [ ] Category filter bar filters content with zero perceived latency (single-select toggle)
- [ ] Search overlay returns results as-you-type across all text fields with bold highlight
- [ ] FTS5 search works with 1000+ saves in < 100ms, tokenizer handles unicode and diacritics
- [ ] Grid scrolls at 60fps with no jank on iPhone 13 / Pixel 7
- [ ] Multi-layer thumbnail caching works (memory LRU → disk → network) with > 95% cache hit rate
- [ ] Thumbnail prefetching loads next 2 pages ahead of scroll position
- [ ] Thumbnail progressive loading works (blur → sharp) with platform-color fallback
- [ ] Long-press context menu works with all 6 actions (native per platform)
- [ ] List view works with swipe-to-delete + undo (5 sec)
- [ ] Multi-select mode works with batch add-to-collection, recategorize, extract, delete
- [ ] Sort options persist across sessions and correctly regroup content
- [ ] Empty states display correctly for all 5 scenarios (new, no results, empty category, empty custom category, last deleted)
- [ ] New saves from Share Extension appear in library without manual refresh with "NEW" badge
- [ ] Quick Save FAB works: clipboard URL detection, paste, validation, ingestion pipeline
- [ ] View toggle works across all three modes (uniform/staggered/list) with layout transition animation
- [ ] Hero transition from grid card to detail view is smooth
- [ ] Reactive data binding works: Core Data `@FetchRequest` / Room `Flow` updates UI automatically on data changes
- [ ] SQLite WAL mode enabled and verified for concurrent share extension writes + UI reads
- [ ] Haptic feedback fires on tab switch (respects system accessibility settings)
- [ ] Light/dark mode: all badges, overlays, and fallback gradients maintain contrast in both modes
- [ ] iPad/tablet: 3-4 column grid renders correctly, sidebar navigation works on large screens
- [ ] Landscape mode: grid expands to 3 columns, safe areas respected, search overlay uses full width
- [ ] Analytics events fire correctly for all 10 event types with correct properties
- [ ] Accessibility: VoiceOver/TalkBack navigation works, Dynamic Type scales correctly, Reduce Motion respected
- [ ] Deep links work from cold launch and warm resume for both `staq://library` and `staq://card/{id}`
