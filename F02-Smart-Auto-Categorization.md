# F2: Smart Auto-Categorization — Design & Development Guide

**Priority:** P0 (MVP)  
**Phase:** 1  
**Platforms:** iOS (Swift) + Android (Kotlin)  
**Dependencies:** F1 (Share Extension)  
**Estimated Effort:** 3–4 weeks

---

## 1. Feature Overview

Smart Auto-Categorization is the AI brain of Staq. When content is saved via the Share Extension, this feature automatically determines what type of content it is (recipe, travel, product, fitness, etc.) and extracts relevant structured data — without the user doing anything.

This feature operates across three processing tiers depending on device capability, making it the most technically complex component of the app.

---

## 2. User Stories

| ID | Story | Acceptance Criteria |
|----|-------|-------------------|
| US-2.1 | As a user, I want my saves to be automatically categorized so I don't have to manually organize | > 85% of saves correctly categorized without user intervention |
| US-2.2 | As a user, I want to override incorrect categories so my library stays accurate | One-tap category change from the card view |
| US-2.3 | As a user, I want to create custom categories for content that doesn't fit defaults | Custom category creation with emoji picker |
| US-2.4 | As a user, I want categorization to work even on my older phone | Tier 2 rule-based classifier works on all devices |
| US-2.5 | As a user, I want to see why Staq chose a category so I trust the system | Subtle "Why this category?" label showing key signals |

---

## 3. Category Taxonomy

### 3.1 Default Categories

| Category | Emoji | Signal Keywords | Content Extracted |
|----------|-------|----------------|-------------------|
| Recipes | 🍳 | recipe, ingredients, cook, bake, tbsp, cup, preheat, oven, prep time | Dish name, ingredients, steps, prep time, cuisine |
| Travel | ✈️ | travel, visit, hotel, flight, beach, trip, itinerary, 📍, destination | Location, tips, best time, budget, things to do |
| Products | 🛍️ | buy, price, amazon, discount, sale, link in bio, code, $, review | Product name, brand, price, link, rating |
| Fitness | 💪 | workout, exercise, reps, sets, cardio, gym, HIIT, yoga, stretch | Exercise names, sets/reps, duration, muscle groups |
| Beauty | 💄 | skincare, makeup, routine, serum, SPF, moisturizer, foundation | Product names, routine steps, skin type |
| Fashion | 👗 | outfit, style, OOTD, haul, try-on, size, wear | Items, brands, sizes, occasion, season |
| DIY & Home | 🏠 | DIY, home, decor, organize, hack, before and after, renovation | Materials, steps, tools, cost |
| Finance | 💰 | invest, stock, mutual fund, SIP, budget, savings, crypto, portfolio | Instruments, strategies, amounts, terms |
| Entertainment | 🎬 | watch, movie, show, series, Netflix, review, trailer, must-watch | Title, platform, genre, rating, review |
| Inspiration | ✨ | quote, motivation, mindset, affirmation, productivity, life | Quote text, author, theme |
| General | 📌 | (default fallback) | Title, summary |

### 3.2 Custom Categories

Users can create up to 20 custom categories (free: 5, paid: 20) with:
- Custom name (max 30 characters)
- Emoji icon (from system emoji picker)
- Optional keyword hints (helps classifier learn user intent)

Example: User creates "Wedding Planning 💒" with keywords: "wedding, venue, dress, flowers, registry"

---

## 4. Three-Tier Processing Architecture

### 4.1 Tier 1: On-Device AI (Flagship Phones)

**Devices:** iPhone 15 Pro+, Pixel 8+, Galaxy S24+, OnePlus 13, Xiaomi 15

```
Input: caption text + OCR text from thumbnail
         │
         ▼
┌─────────────────────────────────────┐
│  Apple Foundation Models (iOS)      │
│  OR                                 │
│  Gemini Nano Prompt API (Android)   │
│                                     │
│  System Prompt:                     │
│  "Classify this social media post   │
│  into one category and extract      │
│  structured data. Categories:       │
│  recipe, travel, product, fitness,  │
│  beauty, fashion, diy, finance,     │
│  entertainment, inspiration,        │
│  general.                           │
│                                     │
│  Return JSON:                       │
│  {                                  │
│    category: string,                │
│    confidence: 0.0-1.0,             │
│    title: string,                   │
│    extracted_data: {...}             │
│  }                                  │
│                                     │
│  Post caption: <caption text>       │
│  On-screen text: <OCR text>         │
│  Hashtags: <parsed hashtags>"       │
└─────────────────────────────────────┘
         │
         ▼
   Parse JSON response
         │
         ▼
   ┌── Confidence ≥ 0.7? ──┐
   │                         │
  YES                       NO
   │                         │
   ▼                         ▼
  Save with              Flag as "uncertain"
  category               Use best guess but
                         show "Recategorize?" hint
```

**Prompt Engineering — iOS Foundation Models:**

```swift
import FoundationModels

let session = LanguageModelSession()
let prompt = """
Classify this social media post. Pick exactly one category from: recipe, travel, product, fitness, beauty, fashion, diy, finance, entertainment, inspiration, general.

Extract relevant structured data based on the category.

Caption: "\(captionText)"
On-screen text: "\(ocrText)"
Hashtags: \(hashtags.joined(separator: ", "))

Respond with JSON only, no other text:
{
  "category": "recipe",
  "confidence": 0.92,
  "title": "15-Minute Garlic Pasta",
  "extracted_data": {
    "dish_name": "Garlic Pasta",
    "ingredients": ["pasta", "garlic", "olive oil", "parmesan", "red pepper flakes"],
    "prep_time": "15 minutes",
    "cuisine": "Italian"
  }
}
"""

let response = try await session.respond(to: prompt)
let classification = try JSONDecoder().decode(Classification.self, from: response.data)
```

**Prompt Engineering — Android Gemini Nano:**

```kotlin
val promptApi = GenAiPromptApi.newBuilder().build()
val request = GenAiPromptRequest.newBuilder()
    .setPrompt(promptText) // Same prompt structure as iOS
    .build()

val response = promptApi.generateContent(request)
val classification = gson.fromJson(response.text, Classification::class.java)
```

### 4.2 Tier 2: Rule-Based Classifier (Mid-Range Phones)

**Devices:** All phones without native AI frameworks

```
Input: caption text + hashtags
         │
         ▼
┌─────────────────────────────────────┐
│  Step 1: Hashtag Scoring            │
│                                     │
│  Parse all #hashtags from caption   │
│  Score each against category        │
│  keyword dictionaries               │
│  #easyrecipe → recipe (+2)          │
│  #travel → travel (+2)              │
│  #homemade → recipe (+1)            │
│  #bali → travel (+2)               │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Step 2: Caption Keyword Matching   │
│                                     │
│  Tokenize caption text              │
│  Match against keyword dictionaries │
│  "2 cups flour" → recipe (+3)       │
│  "preheat oven" → recipe (+3)       │
│  Weight: exact match > partial      │
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  Step 3: Pattern Extraction         │
│                                     │
│  Regex: ingredient patterns         │
│    /\d+\s*(cup|tbsp|tsp|oz|g)/      │
│  Regex: price patterns              │
│    /\$\d+\.?\d*/                     │
│    /₹\d+/                           │
│  Regex: location patterns           │
│    /📍\s*\w+/                       │
│  Regex: exercise patterns           │
│    /\d+\s*(reps|sets|min)/          │
└─────────────────────────────────────┘
         │
         ▼
   Highest-scoring category wins
   Confidence = top score / total score
```

**Keyword Dictionary Structure:**

```swift
let categoryKeywords: [String: [(keyword: String, weight: Int)]] = [
    "recipe": [
        ("recipe", 3), ("ingredients", 3), ("cook", 2), ("bake", 2),
        ("tablespoon", 3), ("teaspoon", 3), ("cup", 2), ("preheat", 3),
        ("oven", 2), ("prep time", 3), ("serving", 2), ("homemade", 1),
        ("cuisine", 2), ("dish", 1), ("simmer", 2), ("sauté", 2),
        // Hindi/bilingual for India market
        ("sabzi", 2), ("masala", 2), ("tadka", 2), ("roti", 2),
        ("paneer", 2), ("dal", 2), ("biryani", 2), ("chutney", 2)
    ],
    "travel": [
        ("travel", 3), ("visit", 2), ("hotel", 2), ("flight", 2),
        ("beach", 2), ("trip", 2), ("itinerary", 3), ("destination", 2),
        ("booking", 2), ("resort", 2), ("explore", 1), ("wanderlust", 1),
        ("backpacking", 2), ("hidden gem", 2), ("must visit", 2)
    ],
    // ... etc for each category
]
```

### 4.3 Tier Selection Logic

```swift
func selectProcessingTier() -> ProcessingTier {
    #if os(iOS)
    if #available(iOS 26, *) {
        // Check if Foundation Models available (iPhone 15 Pro+)
        if FoundationModels.isAvailable {
            return .tier1_onDeviceAI
        }
    }
    #endif
    
    // Android: check for Gemini Nano availability
    // AICore.isAvailable() equivalent
    
    return .tier2_ruleBased
}
```

---

## 5. UX Design

### 5.1 Categorization Happens Invisibly

The user never sees "categorizing..." in normal flow. By the time they open the Staq app, the save is already categorized. The magic is in the invisibility.

**Exception — in-progress state:** If the user opens Staq within seconds of saving (before categorization completes), show a skeleton card with a shimmer animation and a subtle "Organizing..." label. The card transitions to its final state in-place once processing finishes — no page reload, no jump.

```
┌─────────────────────────────┐
│  ┌───────────────────────┐  │
│  │ ░░░░░░░░░░░░░░░░░░░░ │  │
│  │ ░░░ [Shimmer] ░░░░░░ │  │
│  │ ░░░░░░░░░░░░░░░░░░░░ │  │
│  └───────────────────────┘  │
│  ░░░░░░░░░░░░░░             │
│  Organizing...              │
└─────────────────────────────┘
        ↓ (1-2 sec later)
┌─────────────────────────────┐
│  ┌───────────────────────┐  │
│  │     [Thumbnail]       │  │
│  │              🍳 Recipe│  │
│  └───────────────────────┘  │
│  15-Minute Garlic Pasta     │
│  instagram · just now       │
└─────────────────────────────┘
```

### 5.2 Category Badge on Cards

Each saved item shows a small category badge as an overlay on the thumbnail (bottom-right corner):

```
┌─────────────────────────────┐
│  ┌───────────────────────┐  │
│  │                       │  │
│  │     [Thumbnail]       │  │
│  │                       │  │
│  │              🍳 Recipe│  │
│  └───────────────────────┘  │
│  15-Minute Garlic Pasta     │
│  instagram · 2h ago         │
└─────────────────────────────┘
```

Badge uses a frosted-glass/blur background for legibility over any thumbnail color. Badge is always visible — it's the primary organizational signal in the grid view.

### 5.3 Manual Override Flow

**Trigger options (multiple paths for discoverability):**
1. **Tap the category badge** on any card (primary — most discoverable)
2. **Long-press the card → "Recategorize"** from context menu (secondary)
3. **Tap "Not quite right?"** hint on uncertain classifications (contextual)

Tapping the badge opens a bottom sheet:

```
┌─────────────────────────────────────┐
│  Change Category                    │
│                                     │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ 🍳   │ │ ✈️   │ │ 🛍️   │       │
│  │Recipe│ │Travel│ │Shop  │       │
│  └──────┘ └──────┘ └──────┘       │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ 💪   │ │ 💄   │ │ 👗   │       │
│  │Fit   │ │Beauty│ │Style │       │
│  └──────┘ └──────┘ └──────┘       │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ 🏠   │ │ 💰   │ │ 🎬   │       │
│  │ DIY  │ │Money │ │Watch │       │
│  └──────┘ └──────┘ └──────┘       │
│  ┌──────┐ ┌──────┐                 │
│  │ ✨   │ │ 📌   │                 │
│  │Inspo │ │Other │                 │
│  └──────┘ └──────┘                 │
│                                     │
│  ┌─────────────────────────────┐   │
│  │ + Create New Category       │   │
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**Design details:**
- **Current category is visually highlighted** with a colored ring/border and a small checkmark, so the user knows the starting state.
- **One-tap change** — tapping a new category applies immediately and dismisses the sheet with a brief confirmation toast: "Moved to ✈️ Travel".
- **Undo:** Toast includes "Undo" action for 4 seconds in case of misclick. Essential because the action is immediate and there's no confirmation dialog.
- **Recently used categories are shown first** (above the fold) for repeat overriders.

### 5.4 Custom Category Creation

Tapping "+ Create New Category" transitions the bottom sheet:

```
┌─────────────────────────────────────┐
│  New Category                       │
│                                     │
│  ┌──────┐                          │
│  │  📎  │  ← Tap to pick emoji    │
│  └──────┘                          │
│                                     │
│  Name                               │
│  ┌─────────────────────────────┐   │
│  │  Wedding Planning           │   │
│  └─────────────────────────────┘   │
│                                     │
│  Keywords (optional — helps AI)     │
│  ┌─────────────────────────────┐   │
│  │  wedding, venue, dress,     │   │
│  │  flowers, registry          │   │
│  └─────────────────────────────┘   │
│  Separate with commas. These help  │
│  Staq recognize similar content.   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │       Create ✓              │   │
│  └─────────────────────────────┘   │
│                                     │
│  Free: 2 of 5 custom categories    │
│  used. Upgrade for unlimited.      │
│                                     │
└─────────────────────────────────────┘
```

**Keyword interaction with tiers:**
- **Tier 1 (AI):** Custom keywords are appended to the AI prompt as additional categories. E.g., "Also consider these user-defined categories: Wedding Planning (keywords: wedding, venue, dress)."
- **Tier 2 (rule-based):** Keywords are added to the keyword dictionary with weight 3 (high priority — user explicitly defined them).

### 5.5 Uncertain Classification Hint

When confidence < 0.7, show a subtle, non-intrusive hint on the card. The tone should be helpful, not self-deprecating — Staq should never say "I don't know."

```
┌─────────────────────────────┐
│  [Thumbnail]       📌 Other │
│  "omg this is so good 😍"  │
│  ── Not quite right? ─────── │
│  instagram · 1h ago         │
└─────────────────────────────┘
```

**Copy choice:** "Not quite right?" is a gentle question that invites correction without admitting failure. Alternatives tested and rejected:
- ~~"Not sure? Tap to fix"~~ — self-deprecating, undermines trust
- ~~"Miscategorized?"~~ — too technical
- ~~"Help us improve"~~ — feels like unpaid labor

Tapping "Not quite right?" opens the recategorize sheet (same as 5.3).

**Frequency capping:** If a user ignores the "Not quite right?" hint 3 times on the same save, stop showing it. They likely don't care.

### 5.6 Multi-Category Tagging

When AI detects high confidence for two categories (both > 0.5), assign the primary (highest score) as the main category and show the secondary as a small tag:

```
┌─────────────────────────────┐
│  [Thumbnail]       🍳 Recipe│
│  Bali Cooking Class          │
│  ✈️ also Travel              │
│  instagram · 5h ago          │
└─────────────────────────────┘
```

"also Travel" is tappable — opens the save filtered within the Travel category view. This ensures the save is findable from both categories.

### 5.7 Learning from User Corrections

Every manual override is a training signal:

```
UserCorrection {
    originalCategory: "general"
    correctedCategory: "recipe"
    captionText: "Must try this 🔥 link in bio"
    hashtags: ["#foodie", "#viral"]
    platform: "instagram"
}
```

Over time, these corrections improve the keyword dictionary and prompts. For example, if users frequently recategorize posts with "#foodie" from General to Recipe, the system adds "foodie" to the recipe keyword dictionary with a weight of 1.

---

## 6. Richness Score System

Every classified save gets a richness score that determines what the user sees:

| Score | Criteria | UI Treatment |
|-------|----------|-------------|
| **High (0.8–1.0)** | Category identified + 3 or more structured fields extracted (e.g., dish name + ingredients + cook time) | Full structured card, no "Get Full Details" button |
| **Medium (0.5–0.79)** | Category identified + 1–2 structured fields (e.g., dish name but no ingredients) | Partial card with "Get Full Details" button |
| **Low (0.0–0.49)** | Only category guess, no meaningful extraction | Basic card (thumbnail + caption only) with prominent "Get Full Details" button |

**Score calculation:**

```swift
func calculateRichnessScore(classification: Classification) -> Double {
    var score = 0.0
    
    // Category confidence contributes 40%
    score += classification.confidence * 0.4
    
    // Number of extracted fields contributes 60%
    let fieldCount = classification.extractedData.nonNilFieldCount
    let maxExpectedFields = expectedFieldCount(for: classification.category)
    let fieldRatio = min(Double(fieldCount) / Double(maxExpectedFields), 1.0)
    score += fieldRatio * 0.6
    
    return score
}
```

---

## 7. Edge Cases

| Scenario | Handling |
|----------|----------|
| Caption is entirely emojis ("😍🔥💯") | Low confidence. Classify as General. Rely on thumbnail OCR if available. |
| Caption is in non-English language | Tier 1 AI handles multilingual natively. Tier 2: maintain Hindi/Spanish/Portuguese keyword dictionaries for primary markets. |
| Content fits multiple categories (recipe + travel: "I cooked this in Bali") | Assign highest-confidence as primary. Show secondary as "also [Category]" tag if confidence > 0.5. |
| Caption is "link in bio" or generic promotional text | Low confidence, General category. This is the primary trigger for Deep Extract (F5). |
| User creates custom category that overlaps with default | Custom categories take priority in matching. User intent > system defaults. |
| Hashtag spam (30 unrelated hashtags) | Weight hashtags lower when count > 10. Focus on caption text. |
| Shared URL has no caption at all (only URL) | Classify as General with low richness. Eligible for immediate Deep Extract. |
| On-device AI returns malformed JSON | Wrap parsing in try/catch. On parse failure, fall back to Tier 2 rule-based classifier. Log the failure for prompt tuning. Never show an error to the user — just degrade gracefully. |
| On-device AI returns a category not in the taxonomy | Map to nearest valid category by string similarity. If no match, default to General. |
| OCR returns garbage text from a decorative/stylized thumbnail | Score OCR text confidence separately. Discard OCR input if < 50% of characters are recognizable words. |
| Same caption with different URLs (reposts, duplication) | Classify independently — same caption may be shared by different creators with different context. |
| User recategorizes the same save multiple times | Track all corrections (not just the latest). Use the frequency pattern to identify genuinely ambiguous content. |

---

## 8. Accessibility

- **VoiceOver / TalkBack:** Category badge announces as "Category: Recipe" (not just the emoji). Uncertain hint announces "Category may be incorrect. Double tap to change."
- **Dynamic Type:** Category badge text scales with system font. Badge grows vertically, never truncates the label.
- **Color independence:** Categories are differentiated by emoji + text label, not color alone. Color-blind users can distinguish all categories.
- **Recategorize sheet:** All category chips are focusable and navigable via keyboard/switch control. Currently-selected category announces "Recipe, currently selected."
- **Minimum tap targets:** Category badge ≥ 44×44pt, each chip in recategorize sheet ≥ 44×44pt.
- **Reduce Motion:** Shimmer animation on processing state replaced with static "Organizing..." text.

---

## 9. Animation Specifications

| Transition | Duration | Easing | Description |
|-----------|----------|--------|-------------|
| Skeleton → final card | 300ms | ease-out | Crossfade shimmer to thumbnail + badge |
| Category badge change | 200ms | spring (0.6 damping) | Old badge scales down + fades, new badge scales up + fades in |
| Recategorize sheet open | 250ms | ease-out | Standard bottom sheet spring |
| "Not quite right?" appear | 400ms | ease-in | Delayed fade-in (appears 0.5s after card renders, so it doesn't distract from content) |
| Multi-category tag appear | 200ms | ease-out | Slides in from left below primary category |
| Toast ("Moved to Travel") | 250ms in, 200ms out | ease-out / ease-in | Slides up from bottom, auto-dismisses after 4 sec |

---

## 8. Development Plan

### 8.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Define Classification data model + JSON schema | Shared | 3h | — |
| 2 | Build category taxonomy + keyword dictionaries | Shared | 6h | — |
| 3 | Implement Tier 2 rule-based classifier | Shared/KMP | 8h | Task 2 |
| 4 | Implement regex extractors (ingredients, prices, locations) | Shared/KMP | 6h | — |
| 5 | Implement Tier 1 Foundation Models integration | iOS | 6h | Tasks 1, 2 |
| 6 | Implement Tier 1 Gemini Nano integration | Android | 6h | Tasks 1, 2 |
| 7 | Build tier selection logic | Both | 2h | Tasks 3, 5, 6 |
| 8 | Implement richness score calculation | Shared/KMP | 3h | Task 1 |
| 9 | Build OCR pipeline (Vision/ML Kit) | Both | 4h | — |
| 10 | Build manual override UI (recategorize sheet) | Both | 4h | — |
| 11 | Build custom category creation flow | Both | 4h | — |
| 12 | Implement user correction tracking | Both | 3h | Task 10 |
| 13 | Implement confidence-based UI hints | Both | 2h | Task 8 |
| 14 | Connect Share Extension → Categorization pipeline | Both | 4h | F1, Tasks 3–7 |
| 15 | Keyword dictionary tuning + accuracy testing | QA | 8h | Task 3 |
| 16 | AI prompt tuning + accuracy testing | QA | 8h | Tasks 5, 6 |
| 17 | End-to-end testing across tiers | QA | 6h | All above |

**Total: ~83 hours (~3 weeks with one developer per platform)**

### 8.2 Accuracy Benchmarking

Create a test dataset of 200 Instagram/TikTok captions (manually labeled):
- 40 recipes, 40 travel, 30 products, 20 fitness, 20 beauty, 20 DIY, 10 finance, 10 entertainment, 10 inspiration
- Include edge cases: emoji-only, bilingual, multi-category, vague captions

Measure:
- **Tier 1 accuracy target:** > 90% correct category
- **Tier 2 accuracy target:** > 80% correct category
- **False positive rate:** < 10% (content classified into wrong specific category vs. General)

### 8.3 Definition of Done

- [ ] Tier 1 on-device AI classifies content with > 85% accuracy on test dataset
- [ ] Tier 2 rule-based classifier achieves > 75% accuracy on test dataset
- [ ] Tier selection automatically picks best available tier for device
- [ ] Richness score correctly differentiates High/Medium/Low saves
- [ ] Manual category override works with one tap
- [ ] Custom category creation works with emoji picker
- [ ] OCR extracts on-screen text from thumbnails on both platforms
- [ ] User corrections logged for future dictionary improvement
- [ ] Processing completes within 2 seconds on Tier 1, 1 second on Tier 2
- [ ] Bilingual captions (English + Hindi) handled correctly
