# F4: Structured Data Cards — Design & Development Guide

**Priority:** P0 (MVP)  
**Phase:** 1  
**Platforms:** iOS (Swift/SwiftUI) + Android (Kotlin/Compose)  
**Dependencies:** F2 (Auto-Categorization), F3 (Content Library)  
**Estimated Effort:** 2–3 weeks

---

## 1. Feature Overview

Structured Data Cards are the detail view for each saved piece of content. Instead of just showing a thumbnail and caption, Staq presents extracted information in purpose-built card templates — a recipe card shows ingredients and steps, a travel card shows a location map, a product card shows price and purchase link.

This is where the AI extraction becomes tangible value to the user.

---

## 2. User Stories

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-4.1 | As a user, I want to see recipe ingredients as a checklist so I can shop from saved Reels | Recipe card renders ingredients as tappable checklist items |
| US-4.2 | As a user, I want to see the location on a map for travel saves | Travel card shows embedded map pin |
| US-4.3 | As a user, I want to tap through to buy a product I saved | Product card shows "Buy" link to source |
| US-4.4 | As a user, I want to re-watch the original video without leaving Staq | Inline "View Original" button deep-links to source app |
| US-4.5 | As a user, I want to see basic info even when AI couldn't extract structured data | Generic card always shows thumbnail + caption + metadata |

---

## 3. UX Design

### 3.1 Design Principle: "The content's essence on a single screen"

Each card should answer: "What did I save and why does it matter?" in one glance. Users should rarely need to re-watch the original video.

### 3.2 Navigation & Scroll Behavior

- **Entry:** Tap a card in the Library grid → hero transition animates thumbnail from grid position to full-width header image.
- **Back:** iOS swipe-from-left-edge gesture + "◀" button. Android system back + toolbar back arrow.
- **Scroll:** Full-screen scrollable content. Thumbnail header collapses as user scrolls (parallax effect — image scrolls at 0.5× speed, then pins to a compact 44pt nav bar showing title + ⋯ button). Content sections scroll naturally below.
- **Sticky "View Original" CTA:** The primary "View Original" button pins to the bottom of the screen as a floating bar when the user scrolls past the thumbnail. Always one tap away.

### 3.3 Common Card Header

Every card shares this header, then diverges into category-specific content:

```
┌─────────────────────────────────────────┐
│                                         │
│ ◀ Back                        ⋯ More   │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │                                     │ │
│ │          [Video Thumbnail]          │ │
│ │              (tap to enlarge)       │ │
│ │                                     │ │
│ │                        🍳 Recipe    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ 15-Minute Garlic Pasta                  │
│ @cookingwithsara · Instagram · 2h ago   │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📝 "try tonight"            [edit] │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ─── Category-Specific Content Below ─── │
│                                         │
└─────────────────────────────────────────┘
```

**Header details:**
- **Thumbnail:** Tappable to view full-size. If thumbnail failed to load, show platform-colored gradient with large platform icon.
- **Title:** AI-generated or extracted. Large, bold (22pt). Tappable to edit.
- **Creator + Platform + Time:** Secondary text (14pt, grey). Tapping creator handle opens their profile in the source app.
- **User Note:** Shown in a subtle card/bubble below the metadata. "[edit]" link allows inline editing. If no note exists, show ghost text: "+ Add a note" — tappable to create one.

### 3.4 Recipe Card

```
┌─────────────────────────────────────────┐
│ ◀                              ⋯ More  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │        [Video Thumbnail]            │ │
│ │                          🍳 Recipe  │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ 15-Minute Garlic Pasta                  │
│ @cookingwithsara · Instagram · 2h ago   │
│ 📝 "try tonight"                [edit] │
│                                         │
│ ┌───────────────────────────────────┐   │
│ │ ⏱️ 15 min  │  🍽️ 2 servings     │   │
│ │ 🌍 Italian │  📊 Easy            │   │
│ └───────────────────────────────────┘   │
│                                         │
│ INGREDIENTS                      0 of 6 │
│ ┌───────────────────────────────────┐   │
│ │ ☐  200g spaghetti                │   │
│ │ ☐  4 cloves garlic, minced       │   │
│ │ ☐  3 tbsp olive oil              │   │
│ │ ☐  1/2 cup parmesan, grated      │   │
│ │ ☐  Red pepper flakes             │   │
│ │ ☐  Fresh parsley                 │   │
│ │                                   │   │
│ │ 🛒 Add to Shopping List     PRO  │   │
│ └───────────────────────────────────┘   │
│                                         │
│ STEPS                                   │
│ ┌───────────────────────────────────┐   │
│ │ 1. Boil pasta until al dente     │   │
│ │ 2. Sauté garlic in olive oil     │   │
│ │ 3. Toss pasta with garlic oil    │   │
│ │ 4. Top with parmesan & parsley   │   │
│ └───────────────────────────────────┘   │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │    🔗 View Original on Instagram    │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

**Interactive ingredients:**
- Tapping a checkbox strikes through the ingredient (useful while cooking or shopping). State persists across sessions.
- Counter "0 of 6" updates as items are checked: "3 of 6". Provides progress feel when cooking.
- "Uncheck All" link appears when any items are checked.

**"Add to Shopping List" (Pro):** Free users see the button with a small "PRO" badge. Tapping opens a 1-screen upsell: "Add ingredients to your shopping list with Staq Pro. $4.99/month." — not a full paywall, just a gentle nudge with a dismiss option.

### 3.5 Travel Card

```
┌─────────────────────────────────────────┐
│ ◀                              ⋯ More  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │        [Video Thumbnail]            │ │
│ │                        ✈️ Travel    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Hidden Beaches of Bali                  │
│ @wanderlust.vida · TikTok · 3h ago      │
│                                         │
│ ┌───────────────────────────────────┐   │
│ │       ┌───────────────────┐       │   │
│ │       │   [Map with Pin]  │       │   │
│ │       │   📍              │       │   │
│ │       └───────────────────┘       │   │
│ │  Nyang Nyang Beach,               │   │
│ │  Bali, Indonesia                  │   │
│ │                                   │   │
│ │  🗺️ Open in Maps                 │   │
│ └───────────────────────────────────┘   │
│                                         │
│ HIGHLIGHTS                              │
│ ┌───────────────────────────────────┐   │
│ │ 🕐 Best time: May–September      │   │
│ │ 💰 Budget: $30–50/day            │   │
│ └───────────────────────────────────┘   │
│                                         │
│ TIPS                                    │
│ ┌───────────────────────────────────┐   │
│ │ • Go early morning for privacy   │   │
│ │ • Bring water — no vendors       │   │
│ │ • Access via steep cliff path    │   │
│ └───────────────────────────────────┘   │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │     🔗 View Original on TikTok     │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

**Map integration:** Use MapKit (iOS) / Google Maps SDK (Android) for embedded map. Static map snapshot by default (no interactivity in-card — reduces memory). Tapping map or "Open in Maps" launches native maps app with the pin and location name. If coordinates aren't available but a location name is, show the name as a tappable link that searches Maps.

### 3.6 Product Card

```
┌─────────────────────────────────────────┐
│ ◀                              ⋯ More  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │        [Video Thumbnail]            │ │
│ │                      🛍️ Product    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Portable Standing Desk Converter        │
│ @techfinds · Instagram · 1d ago         │
│                                         │
│ ┌───────────────────────────────────┐   │
│ │ 🏷️ FlexiSpot E7 Pro              │   │
│ │ 💲 $299.99                        │   │
│ │ ⭐ 4.7/5 (2,340 reviews)         │   │
│ │ 🏪 Amazon                         │   │
│ │                                   │   │
│ │ ┌─────────────────────────────┐   │   │
│ │ │  🔗 View on Amazon          │   │   │
│ │ └─────────────────────────────┘   │   │
│ │                                   │   │
│ │ ⚠️ Price may have changed since  │   │
│ │ this was saved.                   │   │
│ └───────────────────────────────────┘   │
│                                         │
│ KEY DETAILS                             │
│ ┌───────────────────────────────────┐   │
│ │ • Height adjustable: 28"–48"     │   │
│ │ • Weight capacity: 220 lbs       │   │
│ │ • Desktop size: 55" × 28"       │   │
│ │ • Electric motor with memory     │   │
│ └───────────────────────────────────┘   │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │   🔗 View Original on Instagram     │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

**Price staleness warning:** Show a subtle "⚠️ Price may have changed since this was saved." note below the price if the save is > 7 days old. Honest UX builds trust and prevents frustration when users click through and see a different price.

### 3.7 Fitness Card

```
┌───────────────────────────────────────┐
│ Full Body HIIT — 20 Minutes          │
│ @fitcoach.jay · YouTube · 2d ago      │
│                                       │
│ ┌───────────────────────────────────┐ │
│ │ ⏱️ 20 min │ 🔥 ~300 cal │ 💪 Full│ │
│ │ 🏋️ None   │ (equipment)          │ │
│ └───────────────────────────────────┘ │
│                                       │
│ EXERCISES                             │
│ ┌───────────────────────────────────┐ │
│ │ 1. Jumping Jacks    30 sec       │ │
│ │ 2. Push-ups         12 reps      │ │
│ │ 3. Squats           15 reps      │ │
│ │ 4. Mountain Climbers 30 sec      │ │
│ │ 5. Plank            45 sec       │ │
│ │ 6. Burpees          10 reps      │ │
│ │                                   │ │
│ │ Repeat 3x with 60 sec rest       │ │
│ └───────────────────────────────────┘ │
└───────────────────────────────────────┘
```

### 3.8 Beauty & Fashion Cards

Beauty and Fashion use a shared "Routine/Items" template pattern:

```
┌───────────────────────────────────────┐
│ Morning Skincare Routine              │
│ @skincarebysara · Instagram · 1d ago  │
│                                       │
│ 👤 Skin type: Combination            │
│                                       │
│ ROUTINE                               │
│ ┌───────────────────────────────────┐ │
│ │ 1. Cleanser: CeraVe Foaming      │ │
│ │ 2. Toner: Paula's Choice BHA     │ │
│ │ 3. Serum: The Ordinary Niacin.   │ │
│ │ 4. Moisturizer: Neutrogena       │ │
│ │ 5. SPF: La Roche-Posay 50+      │ │
│ └───────────────────────────────────┘ │
│                                       │
│ PRODUCTS MENTIONED                    │
│ ┌───────────────────────────────────┐ │
│ │ 🏷️ CeraVe Foaming Cleanser      │ │
│ │ 🏷️ Paula's Choice 2% BHA        │ │
│ │ 🏷️ The Ordinary Niacinamide     │ │
│ └───────────────────────────────────┘ │
└───────────────────────────────────────┘
```

Fashion card variant shows items, brands, sizes, and occasion tags.

### 3.9 Generic Card (Fallback)

For content that doesn't fit a specific template or has low richness score:

```
┌───────────────────────────────────────┐
│ ┌─────────────────────────────────┐   │
│ │        [Video Thumbnail]        │   │
│ │                          📌     │   │
│ └─────────────────────────────────┘   │
│                                       │
│ "This changed my whole morning        │
│  routine! Must try the cold shower    │
│  + journaling combo..."              │
│                                       │
│ @productivityhacks · Instagram · 5d   │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │  🔍 Get Full Details            │   │
│ │  Analyzes the video to extract  │   │
│ │  the important stuff.           │   │
│ └─────────────────────────────────┘   │
│                                       │
│ ┌─────────────────────────────────┐   │
│ │  🔗 View Original               │   │
│ └─────────────────────────────────┘   │
└───────────────────────────────────────┘
```

**"Get Full Details" has explanatory subtext** for first-time users: "Analyzes the video to extract the important stuff." — teaches the value without jargon. After the user has used Deep Extract 3+ times, hide the subtext (they know what it does).

### 3.10 Inline Field Editing

Any AI-extracted field can be corrected by the user:

```
Tap on "15 minutes" prep time
         │
         ▼
┌───────────────────────────────────┐
│ ⏱️ Prep time                     │
│ ┌─────────────────────────────┐   │
│ │ 25 minutes                  │   │
│ └─────────────────────────────┘   │
│ ┌──────────┐ ┌──────────┐       │
│ │  Cancel  │ │   Save   │       │
│ └──────────┘ └──────────┘       │
└───────────────────────────────────┘
```

- Tapping any data field (prep time, location name, product price, ingredient quantity) opens an inline edit popover
- Pre-filled with current AI-extracted value
- "Save" persists the user's edit and overrides AI data. A small "✏️ edited" indicator appears on the field.
- User edits are never overwritten by subsequent Deep Extracts — user intent always wins.

### 3.11 "Share Card" — Generated Image

When user taps "Share Card" from the More menu, Staq generates a branded image:

```
┌─────────────────────────────────────┐
│  ┌─────────────────────────────┐   │
│  │      [Thumbnail]            │   │
│  └─────────────────────────────┘   │
│                                     │
│  🍳 15-Minute Garlic Pasta         │
│                                     │
│  Ingredients:                       │
│  spaghetti · garlic · olive oil ·  │
│  parmesan · red pepper flakes ·    │
│  parsley                           │
│                                     │
│  ⏱️ 15 min · 🍽️ 2 servings       │
│                                     │
│  ──────────────────────────         │
│  Saved with Staq · staq.app        │
│                                     │
└─────────────────────────────────────┘
```

- Clean, Pinterest-style vertical image (1080×1920px for Stories, 1080×1080px for feed)
- Includes Staq branding at bottom: "Saved with Staq · staq.app" — subtle viral attribution
- User can choose sharing format before sending: "Share as Image" vs "Share as Link"
- Shared via native share sheet (Messages, WhatsApp, Instagram Stories, etc.)

---

## 4. Card Component Architecture

### 4.1 Template Registry Pattern

```swift
protocol CardTemplate {
    var category: String { get }
    func canRender(data: ExtractedData) -> Bool
    func build(save: SavedContent) -> some View
}

class RecipeCardTemplate: CardTemplate { ... }
class TravelCardTemplate: CardTemplate { ... }
class ProductCardTemplate: CardTemplate { ... }
class FitnessCardTemplate: CardTemplate { ... }
class GenericCardTemplate: CardTemplate { ... }  // Always available as fallback

class CardTemplateRegistry {
    let templates: [CardTemplate]
    
    func templateFor(save: SavedContent) -> CardTemplate {
        // Find best matching template
        for template in templates {
            if template.category == save.category && template.canRender(data: save.extractedData) {
                return template
            }
        }
        return GenericCardTemplate() // Fallback
    }
}
```

This makes it easy to add new card types later without modifying existing code.

### 4.2 Extracted Data Schema

```json
{
  "recipe": {
    "dish_name": "Garlic Pasta",
    "ingredients": [
      { "item": "spaghetti", "quantity": "200g" },
      { "item": "garlic", "quantity": "4 cloves", "prep": "minced" }
    ],
    "steps": ["Boil pasta until al dente", "Sauté garlic in olive oil"],
    "prep_time": "15 minutes",
    "servings": 2,
    "cuisine": "Italian",
    "difficulty": "Easy"
  },
  "travel": {
    "location_name": "Nyang Nyang Beach",
    "coordinates": { "lat": -8.8214, "lng": 115.1162 },
    "country": "Indonesia",
    "region": "Bali",
    "best_time": "May–September",
    "budget_range": "$30–50/day",
    "tips": ["Go early morning", "Bring water"]
  },
  "product": {
    "product_name": "FlexiSpot E7 Pro",
    "brand": "FlexiSpot",
    "price": 299.99,
    "currency": "USD",
    "store": "Amazon",
    "purchase_url": "https://...",
    "rating": 4.7,
    "key_specs": ["Height adjustable", "Electric motor"]
  },
  "fitness": {
    "workout_name": "Full Body HIIT",
    "duration": "20 minutes",
    "calories": "~300",
    "target_area": "Full Body",
    "exercises": [
      { "name": "Jumping Jacks", "duration": "30 sec" },
      { "name": "Push-ups", "reps": 12 }
    ],
    "equipment": "None"
  }
}
```

---

## 5. "More" Menu Actions

The ⋯ button in top-right opens a bottom sheet:

```
┌─────────────────────────────────────┐
│  🔗 View Original                   │
│  📁 Add to Collection               │
│  🏷️ Recategorize                    │
│  📝 Edit Note                       │
│  📤 Share Card                       │
│  📋 Copy Extracted Text             │
│  🗑️ Delete Save                     │
│  ──────────────────                 │
│  Cancel                             │
└─────────────────────────────────────┘
```

**"Share Card":** Generates a visual card image (recipe card as PNG) shareable via messaging apps. Viral growth mechanism — friends see the card, wonder what app made it, download Staq.

**"Copy Extracted Text":** Copies all structured data as formatted plain text to clipboard. Useful for pasting recipe into Notes, sending ingredients to someone, etc.

---

## 6. Edge Cases

| Scenario | Handling |
|----------|----------|
| AI extracted ingredients but no steps | Show ingredients section, hide steps section. Show "Get Full Details" to retrieve steps. Don't show an empty "Steps" header. |
| Travel location couldn't be geocoded | Show location name as text without map embed. "Open in Maps" searches for the name as a string query. |
| Product price in different currency than user's | Show original currency as extracted. Don't convert (inaccurate over time). |
| Extracted data is partially wrong | User can tap any field to edit via inline popover. Edits persist and are marked with "✏️ edited". Edits are never overwritten by Deep Extract. |
| Deep extract returns data that changes the card type | Re-render with new template. Animate transition: old sections fade out, new sections fade in. User note and manual edits carry over. |
| Extremely long ingredient list (30+ items) | Collapse after 10 items with "Show all (30)" expander. Expanded state persists. |
| Video thumbnail fails to load | Show platform-colored gradient background (Instagram purple, TikTok black/pink, YouTube red) with centered platform icon and title overlaid. |
| User opens card while Deep Extract is running | Show current partial data. Sections populate live as extraction progresses. Show "Analyzing video..." spinner in sections that are still loading. |
| Product purchase link is broken or 404 | "View on Amazon" button still works (opens the URL). If link was verified broken at extract time, show "⚠️ Link may not be available." |
| Multiple locations in a travel post | Show primary location as map pin. List secondary locations as text links below. Each tappable to open in Maps. |
| Card content is extremely short (only a title) | Still render the full card layout but center the title. Show "Get Full Details" prominently. Don't show empty section headers. |

---

## 7. Accessibility

- **VoiceOver / TalkBack:** Recipe ingredients announced as "Ingredient 1 of 6: 200g spaghetti. Unchecked. Double tap to check." Steps announced as "Step 1 of 4: Boil pasta until al dente."
- **Dynamic Type:** All card text scales with system font size. At largest sizes, the metadata chips (prep time, servings) stack vertically instead of horizontally.
- **Map embed:** Provides accessible label: "Map showing Nyang Nyang Beach, Bali, Indonesia. Double tap to open in Maps."
- **Inline editing:** Edit popover is focus-trapped. VoiceOver announces "Editing prep time. Current value: 15 minutes."
- **Ingredient checkboxes:** State changes announced: "Checked: 200g spaghetti. 1 of 6 checked."
- **Reduce Motion:** Hero transition replaced with crossfade. Parallax scrolling header replaced with simple collapse.
- **Minimum tap targets:** All interactive elements ≥ 44×44pt. Ingredient checkbox rows have full-width tap area.

---

## 8. Development Plan

### 8.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Define ExtractedData JSON schema for all categories | Shared | 4h | F2 |
| 2 | Build CardTemplate protocol + registry pattern | Both | 4h | — |
| 3 | Build common card header with collapsing scroll + user note | Both | 6h | — |
| 4 | Build Recipe card template with ingredient checklist + counter | Both | 6h | Tasks 1–3 |
| 5 | Build Travel card template with static map snapshot | Both | 6h | Tasks 1–3 |
| 6 | Build Product card template with purchase link + staleness warning | Both | 4h | Tasks 1–3 |
| 7 | Build Fitness card template | Both | 4h | Tasks 1–3 |
| 8 | Build Beauty/Fashion card template | Both | 4h | Tasks 1–3 |
| 9 | Build Generic card template with contextual "Get Full Details" | Both | 3h | Tasks 1–3 |
| 10 | Build "More" menu bottom sheet with all actions | Both | 3h | — |
| 11 | Build inline field editing popover with persistence | Both | 5h | Tasks 4–9 |
| 12 | Implement "Share Card" image generation (1080×1920 + 1080×1080) | Both | 5h | Tasks 4–9 |
| 13 | Implement "Copy Extracted Text" formatting per card type | Both | 2h | Tasks 4–9 |
| 14 | Build "View Original" deep-link handler per platform | Both | 3h | — |
| 15 | Build sticky floating "View Original" CTA on scroll | Both | 2h | Task 3 |
| 16 | Hero transition from library grid to card detail | Both | 3h | F3 |
| 17 | Accessibility audit per card type | Both | 4h | Tasks 4–9 |
| 18 | QA: all card types render correctly with varying data completeness | QA | 8h | All above |

**Total: ~76 hours (~3 weeks with one developer per platform)**

### 8.2 Definition of Done

- [ ] All 7 card types render correctly (Recipe, Travel, Product, Fitness, Beauty/Fashion, Generic)
- [ ] Template registry auto-selects correct card type based on category + data
- [ ] Recipe ingredient checklist is interactive with progress counter and strikethrough
- [ ] Travel card shows static map snapshot with pin, tappable to open in Maps
- [ ] Product card shows purchase link and staleness warning for saves > 7 days old
- [ ] Generic card shows "Get Full Details" with contextual subtext for new users
- [ ] "View Original" sticky CTA floats at bottom when user scrolls past thumbnail
- [ ] "Share Card" generates clean branded image in two sizes
- [ ] Inline field editing works on all data fields across all card types
- [ ] User edits persist and are never overwritten by subsequent Deep Extracts
- [ ] Collapsing scroll header works with parallax (and falls back to simple collapse with Reduce Motion)
- [ ] User note is editable from card view
- [ ] Card handles partial data gracefully (shows available sections, hides empty ones, never shows empty headers)
- [ ] Accessibility: VoiceOver/TalkBack announces all interactive elements correctly
