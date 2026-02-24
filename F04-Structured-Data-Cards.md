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
| US-4.6 | As a user, I want to convert recipe measurements between metric and imperial so I can cook with the units I know | Recipe card unit toggle switches all quantities between metric and imperial |
| US-4.7 | As a user, I want to adjust serving sizes so ingredient quantities scale automatically | Serving adjuster recalculates all ingredient quantities proportionally |
| US-4.8 | As a user, I want to tap a cook time and start a countdown timer without leaving the card | Tappable time values launch an inline countdown timer |
| US-4.9 | As a user, I want to see weather and distance info for saved travel locations so I can plan trips | Travel card shows current weather and distance from my location |
| US-4.10 | As a user, I want to be notified when a saved product drops in price | Product card supports price drop alerts (Phase 2) |
| US-4.11 | As a user, I want to save tutorials and courses in a structured format so I can track my learning | Education card shows topic, takeaways, difficulty, and learning progress |

---

## 3. UX Design

### 3.1 Design Principle: "The content's essence on a single screen"

Each card should answer: "What did I save and why does it matter?" in one glance. Users should rarely need to re-watch the original video.

### 3.2 Navigation & Scroll Behavior

- **Entry:** Tap a card in the Library grid → hero transition animates thumbnail from grid position to full-width header image.
- **Back:** iOS swipe-from-left-edge gesture + "◀" button. Android system back + toolbar back arrow.
- **Scroll:** Full-screen scrollable content. Thumbnail header collapses as user scrolls (parallax effect — image scrolls at 0.5x speed, then pins to a compact 44pt nav bar showing title + ... button). Content sections scroll naturally below.
- **Sticky "View Original" CTA:** The primary "View Original" button pins to the bottom of the screen as a floating bar when the user scrolls past the thumbnail. Always one tap away.

### 3.3 Common Card Header

Every card shares this header, then diverges into category-specific content:

```
+------------------------------------------+
|                                          |
| < Back                        ... More   |
|                                          |
| +--------------------------------------+ |
| |                                      | |
| |          [Video Thumbnail]           | |
| |              (tap to enlarge)        | |
| |                                      | |
| |                        Recipe        | |
| +--------------------------------------+ |
|                                          |
| 15-Minute Garlic Pasta                   |
| @cookingwithsara - Instagram - 2h ago    |
|                                          |
| +--------------------------------------+ |
| | "try tonight"                 [edit]  | |
| +--------------------------------------+ |
|                                          |
| --- Category-Specific Content Below ---  |
|                                          |
+------------------------------------------+
```

**Header details:**
- **Thumbnail:** Tappable to view full-size. If thumbnail failed to load, show platform-colored gradient with large platform icon.
- **Title:** AI-generated or extracted. Large, bold (22pt). Tappable to edit.
- **Creator + Platform + Time:** Secondary text (14pt, grey). Tapping creator handle opens their profile in the source app.
- **User Note:** Shown in a subtle card/bubble below the metadata. "[edit]" link allows inline editing. If no note exists, show ghost text: "+ Add a note" -- tappable to create one.

### 3.4 Recipe Card

```
+------------------------------------------+
| <                              ... More  |
|                                          |
| +--------------------------------------+ |
| |        [Video Thumbnail]             | |
| |                          Recipe      | |
| +--------------------------------------+ |
|                                          |
| 15-Minute Garlic Pasta                   |
| @cookingwithsara - Instagram - 2h ago    |
| "try tonight"                    [edit]  |
|                                          |
| +------------------------------------+   |
| | 15 min  |  2 servings  [-] [+]     |   |
| | Italian |  Easy                    |   |
| +------------------------------------+   |
|                                          |
| [Metric] [Imperial]       Dietary tags:  |
|                     Vegetarian, Gluten   |
|                                          |
| INGREDIENTS                      0 of 6  |
| +------------------------------------+   |
| | [ ]  200g (7oz) spaghetti         |   |
| | [ ]  4 cloves garlic, minced      |   |
| | [ ]  3 tbsp olive oil             |   |
| | [ ]  1/2 cup parmesan, grated     |   |
| | [ ]  Red pepper flakes            |   |
| | [ ]  Fresh parsley                |   |
| |                                    |   |
| | [cart] Add to Shopping List   PRO  |   |
| +------------------------------------+   |
|                                          |
| STEPS                                    |
| +------------------------------------+   |
| | 1. Boil pasta until al dente       |   |
| | 2. Saute garlic in olive oil       |   |
| |    [clock 3 min] <-- tappable      |   |
| | 3. Toss pasta with garlic oil      |   |
| | 4. Top with parmesan & parsley     |   |
| +------------------------------------+   |
|                                          |
| +--------------------------------------+ |
| |    View Original on Instagram        | |
| +--------------------------------------+ |
+------------------------------------------+
```

**Interactive ingredients:**
- Tapping a checkbox strikes through the ingredient (useful while cooking or shopping). State persists across sessions.
- Counter "0 of 6" updates as items are checked: "3 of 6". Provides progress feel when cooking.
- "Uncheck All" link appears when any items are checked.

**"Add to Shopping List" (Pro):** Free users see the button with a small "PRO" badge. Tapping opens a 1-screen upsell: "Add ingredients to your shopping list with Staq Pro. $4.99/month." -- not a full paywall, just a gentle nudge with a dismiss option.

**Unit Conversion Toggle (metric <-> imperial):**
- A segmented control (`[Metric] [Imperial]`) sits above the ingredients list. Default is auto-detected from device locale (US/UK/Liberia/Myanmar → imperial, everywhere else → metric).
- Toggling converts all ingredient quantities inline: "200g" becomes "7oz", "1 liter" becomes "~4 cups". Values round to friendly cooking amounts (not "7.0548oz" — use "7oz").
- Conversion uses a local lookup table for common cooking units. No network call required.
- The toggle state persists per card. Users in the US who save Indian recipe content (a key cross-market scenario) can switch once and it stays.

```swift
// Conversion model
struct IngredientQuantity {
    let value: Double
    let unit: CookingUnit

    func converted(to system: MeasurementSystem) -> IngredientQuantity {
        switch (unit, system) {
        case (.grams, .imperial): return IngredientQuantity(value: value * 0.03527, unit: .ounces)
        case (.ml, .imperial): return IngredientQuantity(value: value * 0.00423, unit: .cups)
        case (.celsius, .imperial): return IngredientQuantity(value: value * 9/5 + 32, unit: .fahrenheit)
        // ... full mapping table
        default: return self
        }
    }

    /// Round to nearest friendly cooking fraction (1/4, 1/3, 1/2, 2/3, 3/4)
    func friendlyDisplay() -> String { ... }
}
```

**Serving Size Adjuster:**
- A `[-]` / `[+]` stepper sits next to the serving count in the metadata chips bar: "Serves 2 [-] [+]".
- Tapping `[+]` or `[-]` recalculates all ingredient quantities proportionally. "200g spaghetti" at 2 servings becomes "400g spaghetti" at 4 servings.
- Minimum: 1 serving. Maximum: 12 servings (beyond that, you're catering — link to the original recipe).
- Scaled quantities use the same friendly-rounding logic as unit conversion. No "133.33g" — round to "135g".
- Original serving count extracted by AI is stored separately from the user-adjusted count, so resetting to the original is always possible via a "Reset" link that appears when servings differ from original.
- The ingredient list smoothly animates quantity changes (fade transition on the numeric portion, not the ingredient name).

**Timer Integration:**
- Any time value mentioned in steps ("cook for 15 minutes", "let rest 5 min", "saute 3 minutes") is auto-detected and rendered as a tappable inline chip: `[clock] 15 min`.
- Tapping the chip starts an inline countdown timer that appears as a sticky bar at the top of the steps section:

```
+------------------------------------+
| [clock] 14:38 remaining     [X]   |
+------------------------------------+
```

- Timer uses local notifications so it works even when the user backgrounds the app or locks their phone.
- Multiple timers can run simultaneously (e.g., "boil pasta 10 min" and "saute garlic 3 min"). Each shows as a stacked timer bar. Maximum 3 concurrent timers per card.
- Timer completion triggers a haptic pulse (iOS: `.notification`, Android: `VibrationEffect.createOneShot(200)`), a local notification, and an optional alarm sound.
- Implementation: `UNUserNotificationCenter` (iOS) / `AlarmManager` (Android) for background timers. In-app display uses a simple `Timer`/`CountDownTimer`.

**Dietary Tags (auto-detected):**
- AI extraction analyzes the ingredient list and flags applicable dietary tags: **Vegetarian**, **Vegan**, **Gluten-Free**, **Dairy-Free**, **Keto**, **Nut-Free**.
- Tags display as small colored pills below the metadata chips. Each tag has a subtle icon and label: `[leaf] Vegetarian`, `[wheat-off] Gluten-Free`.
- Detection logic runs locally after extraction using a keyword matcher against ingredient names (e.g., presence of "chicken" removes Vegetarian/Vegan; absence of any flour/wheat/bread items → Gluten-Free candidate).
- Tags include a confidence indicator. High-confidence tags (all ingredients clearly match) show solid. Low-confidence tags (ambiguous ingredients like "broth" which could be veggie or chicken) show with a dotted border and a "?" icon — tappable to explain: "This recipe might be vegetarian, but 'broth' could be meat-based. Tap to confirm."
- Users can manually add or remove dietary tags. Manual overrides persist and are never overwritten.

```swift
struct DietaryTagDetector {
    static func detect(from ingredients: [Ingredient]) -> [DietaryTag] {
        var tags: [DietaryTag] = []
        let names = ingredients.map { $0.item.lowercased() }

        let meatKeywords = ["chicken", "beef", "pork", "lamb", "fish", "shrimp", "bacon", "sausage"]
        let dairyKeywords = ["milk", "cheese", "butter", "cream", "yogurt", "parmesan"]
        let glutenKeywords = ["flour", "bread", "pasta", "wheat", "soy sauce"]

        if names.allSatisfy({ name in !meatKeywords.contains(where: { name.contains($0) }) }) {
            tags.append(.vegetarian(confidence: .high))
        }
        // ... additional detection rules
        return tags
    }
}
```

### 3.5 Travel Card

```
+------------------------------------------+
| <                              ... More  |
|                                          |
| +--------------------------------------+ |
| |        [Video Thumbnail]             | |
| |                        Travel        | |
| +--------------------------------------+ |
|                                          |
| Hidden Beaches of Bali                   |
| @wanderlust.vida - TikTok - 3h ago       |
|                                          |
| +------------------------------------+   |
| |       +---------------------+      |   |
| |       |   [Map with Pin]    |      |   |
| |       |                     |      |   |
| |       +---------------------+      |   |
| |  Nyang Nyang Beach,                |   |
| |  Bali, Indonesia                   |   |
| |  ~12,450 km away                   |   |
| |                                    |   |
| |  [map] Open in Maps               |   |
| |  [folder] Save to Trip            |   |
| +------------------------------------+   |
|                                          |
| WEATHER                                  |
| +------------------------------------+   |
| | [sun] 29C / 84F  Partly Cloudy    |   |
| | Dry season (May-Sep) - ideal       |   |
| +------------------------------------+   |
|                                          |
| HIGHLIGHTS                               |
| +------------------------------------+   |
| | Best time: May-September           |   |
| | Budget: $30-50/day (~2,500 INR)    |   |
| +------------------------------------+   |
|                                          |
| TIPS                                     |
| +------------------------------------+   |
| | - Go early morning for privacy     |   |
| | - Bring water -- no vendors        |   |
| | - Access via steep cliff path      |   |
| +------------------------------------+   |
|                                          |
| +--------------------------------------+ |
| |     View Original on TikTok          | |
| +--------------------------------------+ |
+------------------------------------------+
```

**Map integration:** Use MapKit (iOS) / Google Maps SDK (Android) for embedded map. Static map snapshot by default (no interactivity in-card — reduces memory). Tapping map or "Open in Maps" launches native maps app with the pin and location name. If coordinates aren't available but a location name is, show the name as a tappable link that searches Maps.

**"Save to Trip" Integration:**
- A "Save to Trip" button appears below "Open in Maps". Tapping it opens a sheet listing the user's existing Trip collections (ties into F6: Collections), plus a "+ New Trip" option at the top.
- Trip collections are a special collection type scoped to travel cards. They can have a date range, cover image (auto-set from the first card saved), and a trip name.
- If the user has no trips yet, the button reads "Start a Trip" and opens a quick-create flow: trip name + optional dates. The current card is auto-added.
- Travel cards already assigned to a trip show the trip name as a pill below the location: `[suitcase] Bali June 2025`. Tapping the pill navigates to the trip collection.
- This is the only card type that surfaces the collection/trip association directly in the card UI. Other card types manage collections from the More menu.

**Weather Widget:**
- A compact weather row appears between the map section and highlights. Shows current temperature and conditions for the saved location.
- Data source: OpenWeatherMap free tier (60 calls/min, sufficient for on-demand loading). Weather is fetched when the card is opened, not at save time — keeps data fresh.
- Display format: `[sun] 29C / 84F  Partly Cloudy`. Temperature shows both Celsius and Fahrenheit side by side (no toggle needed — both are useful at a glance for a cross-market user base).
- Below the current weather, show a seasonal hint derived from the "best time to visit" data: `Dry season (May-Sep) -- ideal` or `Monsoon season -- expect rain`. This is AI-extracted, not from the weather API.
- If the weather API call fails (offline, rate limit), hide the weather row gracefully — don't show an error state for non-critical data.
- Cache weather data for 3 hours per location. Store in memory only (not persisted to disk — weather data goes stale quickly).

```swift
struct WeatherService {
    func fetchWeather(lat: Double, lng: Double) async throws -> WeatherSnapshot {
        // OpenWeatherMap free tier — 60 calls/min
        let url = URL(string: "https://api.openweathermap.org/data/2.5/weather?lat=\(lat)&lon=\(lng)&appid=\(apiKey)&units=metric")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(WeatherSnapshot.self, from: data)
    }
}

struct WeatherSnapshot {
    let tempCelsius: Double
    var tempFahrenheit: Double { tempCelsius * 9/5 + 32 }
    let condition: String  // "Partly Cloudy", "Rain", etc.
    let icon: String       // Maps to SF Symbol / Material icon
}
```

**Distance from User:**
- Below the location name, show approximate distance from the user's current location: "~12,450 km away" or "~7,740 mi away" (unit matches device locale).
- Uses `CLLocationManager` (iOS) / `FusedLocationProviderClient` (Android) for a single low-accuracy location fix. No continuous tracking.
- If location permission is not granted, hide the distance line entirely — don't prompt for permission from the card view. The first prompt happens during onboarding or when the user taps "Open in Maps".
- Distance is calculated as great-circle distance (Haversine formula). No routing — this is a rough "how far is this place" indicator, not a travel time estimate.

**Currency Conversion for Budget Amounts:**
- Budget values extracted by AI (e.g., "$30-50/day") are shown in their original currency, with a converted equivalent in parentheses based on the user's device locale currency.
- Example for a user in India: `$30-50/day (~2,500-4,200 INR/day)`.
- Conversion rates are fetched from a free API (e.g., exchangerate.host) and cached locally for 24 hours.
- If conversion fails or the user's currency matches the original, no parenthetical is shown.
- Rates are approximate and a small disclaimer appears on first use: "Currency amounts are estimates and may vary."

### 3.6 Product Card

```
+------------------------------------------+
| <                              ... More  |
|                                          |
| +--------------------------------------+ |
| |        [Video Thumbnail]             | |
| |                      Product         | |
| +--------------------------------------+ |
|                                          |
| Portable Standing Desk Converter         |
| @techfinds - Instagram - 1d ago          |
|                                          |
| +------------------------------------+   |
| | FlexiSpot E7 Pro                   |   |
| | $299.99                            |   |
| | 4.7/5 (2,340 reviews)             |   |
| | Amazon                             |   |
| |                                    |   |
| | +------------------------------+   |   |
| | |  View on Amazon              |   |   |
| | +------------------------------+   |   |
| |                                    |   |
| | [bell] Notify on Price Drop  P2    |   |
| |                                    |   |
| | Price may have changed since       |   |
| | this was saved.                    |   |
| +------------------------------------+   |
|                                          |
| KEY DETAILS                              |
| +------------------------------------+   |
| | - Height adjustable: 28"-48"       |   |
| | - Weight capacity: 220 lbs         |   |
| | - Desktop size: 55" x 28"         |   |
| | - Electric motor with memory       |   |
| +------------------------------------+   |
|                                          |
| SIMILAR SAVES                            |
| +------------------------------------+   |
| | [thumb] FlexiSpot M7  $199        |   |
| | [thumb] VIVO Stand    $89         |   |
| +------------------------------------+   |
|                                          |
| +--------------------------------------+ |
| |   View Original on Instagram         | |
| +--------------------------------------+ |
+------------------------------------------+
```

**Price staleness warning:** Show a subtle "Price may have changed since this was saved." note below the price if the save is > 7 days old. Honest UX builds trust and prevents frustration when users click through and see a different price.

**Price Tracking / Drop Alerts (Phase 2):**
- A "Notify on Price Drop" button appears below the purchase link, with a small "P2" badge indicating this is a Phase 2 feature.
- Phase 1 behavior: Tapping shows a teaser bottom sheet: "Price tracking is coming soon. We'll notify you when saved products drop in price. Want early access?" with an email signup or "Notify Me" toggle that registers interest.
- Phase 2 behavior (future): Staq periodically checks product URLs for price changes using a lightweight backend scraper. When the price drops below the saved price, a push notification fires: "FlexiSpot E7 Pro dropped from $299.99 to $249.99 -- tap to view." The card updates to show a green "Price dropped!" pill with the delta.
- Price tracking is a strong Pro retention feature. Free users get tracking on up to 3 products; Pro users get unlimited.

**"Similar Saves" Section:**
- Below Key Details, show a horizontal scroll row of other saved products in the same AI-detected category (e.g., "Standing Desks", "Headphones", "Skincare").
- Each similar save shows a small thumbnail, product name, and price. Tapping navigates to that card.
- Only shows products the user has already saved — not external recommendations. This is a personal library cross-reference, not a discovery feed.
- If there are no other saved products in the same category, hide this section entirely.
- Matching logic: compare the `product_category` field from extraction. If no category was extracted, fall back to keyword overlap in product name and key specs.

**Affiliate Link Strategy:**
- When a product card links to a supported retailer (Amazon, Target, Walmart), Staq appends an affiliate tag to the purchase URL before opening it. Example: `https://amazon.com/dp/B08XYZ?tag=staq-20`.
- This is transparent to the user — the product page loads identically. The affiliate tag earns Staq a small commission (typically 1-4%) on qualifying purchases.
- Supported affiliate programs for Phase 1: Amazon Associates (global), Impact (multi-retailer).
- Affiliate tags are appended client-side at link-open time, not stored in the database. This keeps extracted URLs clean and allows tag updates via remote config without re-processing saved content.
- A brief disclosure in Settings > About: "Some product links may include affiliate tags that support Staq at no extra cost to you." Compliance with FTC guidelines and App Store policies.
- Revenue projection: Even modest conversion (2% of product card taps leading to purchase) on a user base saving product content frequently could contribute meaningful revenue alongside Pro subscriptions.

### 3.7 Fitness Card

```
+--------------------------------------+
| Full Body HIIT -- 20 Minutes         |
| @fitcoach.jay - YouTube - 2d ago     |
|                                      |
| +----------------------------------+ |
| | 20 min | ~300 cal | Full Body    | |
| | None (equipment)                 | |
| +----------------------------------+ |
|                                      |
| EXERCISES                            |
| +----------------------------------+ |
| | 1. Jumping Jacks    30 sec      | |
| | 2. Push-ups         12 reps     | |
| | 3. Squats           15 reps     | |
| | 4. Mountain Climbers 30 sec     | |
| | 5. Plank            45 sec      | |
| | 6. Burpees          10 reps     | |
| |                                  | |
| | Repeat 3x with 60 sec rest      | |
| +----------------------------------+ |
+--------------------------------------+
```

### 3.8 Beauty & Fashion Cards

Beauty and Fashion use a shared "Routine/Items" template pattern:

```
+--------------------------------------+
| Morning Skincare Routine             |
| @skincarebysara - Instagram - 1d ago |
|                                      |
| Skin type: Combination               |
|                                      |
| ROUTINE                              |
| +----------------------------------+ |
| | 1. Cleanser: CeraVe Foaming     | |
| | 2. Toner: Paula's Choice BHA    | |
| | 3. Serum: The Ordinary Niacin.  | |
| | 4. Moisturizer: Neutrogena      | |
| | 5. SPF: La Roche-Posay 50+     | |
| +----------------------------------+ |
|                                      |
| PRODUCTS MENTIONED                   |
| +----------------------------------+ |
| | CeraVe Foaming Cleanser         | |
| | Paula's Choice 2% BHA           | |
| | The Ordinary Niacinamide        | |
| +----------------------------------+ |
+--------------------------------------+
```

Fashion card variant shows items, brands, sizes, and occasion tags.

### 3.9 Education / Learning Card

For tutorial, course, how-to, and educational content — coding tutorials, language lessons, DIY guides, skill-building videos.

```
+------------------------------------------+
| <                              ... More  |
|                                          |
| +--------------------------------------+ |
| |        [Video Thumbnail]             | |
| |                    Education         | |
| +--------------------------------------+ |
|                                          |
| Build a REST API with Node.js            |
| @codewithanna - YouTube - 5h ago         |
| "need this for work project"    [edit]   |
|                                          |
| +------------------------------------+   |
| | Topic: Backend Development         |   |
| | Difficulty: [==--] Intermediate    |   |
| | Est. Time: ~45 min                 |   |
| +------------------------------------+   |
|                                          |
| KEY TAKEAWAYS                            |
| +------------------------------------+   |
| | - Express.js handles routing and   |   |
| |   middleware                        |   |
| | - Use async/await for DB queries   |   |
| | - JWT for auth, bcrypt for hashing |   |
| | - Always validate input server-side|   |
| +------------------------------------+   |
|                                          |
| TOOLS & RESOURCES                        |
| +------------------------------------+   |
| | [link] Node.js      nodejs.org     |   |
| | [link] Express.js   expressjs.com  |   |
| | [link] MongoDB      mongodb.com    |   |
| | [link] Postman      postman.com    |   |
| +------------------------------------+   |
|                                          |
| +------------------------------------+   |
| | [checkmark] Mark as Learned        |   |
| +------------------------------------+   |
|                                          |
| +--------------------------------------+ |
| |    View Original on YouTube          | |
| +--------------------------------------+ |
+------------------------------------------+
```

**Card fields:**
- **Topic:** AI-extracted subject area. Broad category like "Backend Development", "Watercolor Painting", "Spanish Language". Tappable to edit.
- **Difficulty Level:** Rendered as a visual bar (`[====]` Beginner, `[==--]` Intermediate, `[=---]` Advanced, `[----]` Expert). AI-inferred from content complexity, jargon density, and any explicit mentions of difficulty. Tappable to override.
- **Estimated Learning Time:** Duration of the source content adjusted for learning (video length + estimated practice time). For a 20-minute coding tutorial, this might show "~45 min" to account for following along. User-editable.
- **Key Takeaways:** Bullet-point summary of the most important lessons. AI-extracted, limited to 5-7 bullet points. Each bullet is a concise, actionable statement — not a transcript summary. User can add, edit, or remove bullets.
- **Tools & Resources:** Named tools, libraries, apps, or resources mentioned in the content. Each rendered as a tappable link that opens a web search for that tool (not a guaranteed URL — AI can't reliably extract URLs from video). If the tool is well-known (Node.js, Figma, etc.), link directly to its official site using a local known-tools lookup table.
- **"Mark as Learned" Toggle:** A prominent toggle at the bottom of the card. When activated, the card gets a subtle visual treatment — a green checkmark overlay on the thumbnail in the library grid, and the card header shows a `[checkmark] Learned` pill.
  - Learned state is filterable in the library: "Show: All | Learned | Not Yet".
  - This creates a lightweight personal learning tracker without building a full LMS. Users can see at a glance what they've absorbed vs. what's still in their queue.
  - Toggle state syncs across devices via the user's data store.

**Extracted data schema addition:**

```json
{
  "education": {
    "topic": "Backend Development",
    "subtopic": "REST API Design",
    "key_takeaways": [
      "Express.js handles routing and middleware",
      "Use async/await for database queries",
      "JWT for authentication, bcrypt for password hashing",
      "Always validate input on the server side"
    ],
    "tools_resources": [
      { "name": "Node.js", "url": "https://nodejs.org" },
      { "name": "Express.js", "url": "https://expressjs.com" },
      { "name": "MongoDB", "url": "https://mongodb.com" }
    ],
    "difficulty": "intermediate",
    "estimated_learning_time": "45 minutes",
    "is_learned": false
  }
}
```

### 3.10 Generic Card (Fallback)

For content that doesn't fit a specific template or has low richness score:

```
+--------------------------------------+
| +----------------------------------+ |
| |        [Video Thumbnail]         | |
| |                           Pin    | |
| +----------------------------------+ |
|                                      |
| "This changed my whole morning       |
|  routine! Must try the cold shower   |
|  + journaling combo..."             |
|                                      |
| @productivityhacks - Instagram - 5d  |
|                                      |
| +----------------------------------+ |
| |  Get Full Details                 | |
| |  Analyzes the video to extract    | |
| |  the important stuff.             | |
| +----------------------------------+ |
|                                      |
| +----------------------------------+ |
| |  View Original                    | |
| +----------------------------------+ |
+--------------------------------------+
```

**"Get Full Details" has explanatory subtext** for first-time users: "Analyzes the video to extract the important stuff." -- teaches the value without jargon. After the user has used Deep Extract 3+ times, hide the subtext (they know what it does).

### 3.11 Inline Field Editing

Any AI-extracted field can be corrected by the user:

```
Tap on "15 minutes" prep time
         |
         v
+----------------------------------+
| Prep time                        |
| +------------------------------+ |
| | 25 minutes                   | |
| +------------------------------+ |
| +----------+ +----------+      |
| |  Cancel  | |   Save   |      |
| +----------+ +----------+      |
+----------------------------------+
```

- Tapping any data field (prep time, location name, product price, ingredient quantity) opens an inline edit popover
- Pre-filled with current AI-extracted value
- "Save" persists the user's edit and overrides AI data. A small "edited" indicator appears on the field.
- User edits are never overwritten by subsequent Deep Extracts — user intent always wins.

### 3.12 "Share Card" — Generated Image

When user taps "Share Card" from the More menu, Staq generates a branded image:

```
+--------------------------------------+
|  +--------------------------------+  |
|  |      [Thumbnail]               |  |
|  +--------------------------------+  |
|                                      |
|  15-Minute Garlic Pasta              |
|                                      |
|  Ingredients:                        |
|  spaghetti - garlic - olive oil -    |
|  parmesan - red pepper flakes -      |
|  parsley                             |
|                                      |
|  15 min - 2 servings                 |
|                                      |
|  ----------------------------        |
|  Saved with Staq - staq.app         |
|                                      |
+--------------------------------------+
```

- Clean, Pinterest-style vertical image (1080x1920px for Stories, 1080x1080px for feed)
- Includes Staq branding at bottom: "Saved with Staq - staq.app" -- subtle viral attribution
- User can choose sharing format before sending: "Share as Image" vs "Share as Link"
- Shared via native share sheet (Messages, WhatsApp, Instagram Stories, etc.)

---

## 4. Card Theming & Visual Polish

### 4.1 Category Color Accents

Each card type has a subtle color accent that reinforces its category identity without overpowering the content. The accent is applied to the category badge, section headers, interactive element highlights, and the thin top border of the card.

| Card Type | Accent Color | Hex (Light) | Hex (Dark) | Usage |
|-----------|-------------|-------------|------------|-------|
| Recipe | Warm Orange | `#E8740C` | `#F4A236` | Category badge bg, ingredient checkbox fill, timer chip |
| Travel | Sky Blue | `#0A84FF` | `#64B5F6` | Map border, weather row bg tint, distance text |
| Product | Emerald Green | `#34A853` | `#81C784` | Price text, "View on" button, price drop pill |
| Fitness | Energetic Red | `#EA4335` | `#EF9A9A` | Exercise list numbers, calorie burn highlight |
| Beauty/Fashion | Soft Rose | `#D4618C` | `#F48FB1` | Product mention pills, routine step numbers |
| Education | Deep Purple | `#7C3AED` | `#B39DDB` | Difficulty bar fill, "Learned" checkmark, takeaway bullets |
| Generic | Neutral Grey | `#6B7280` | `#9CA3AF` | Pin badge, metadata text |

**Application rules:**
- Accent color is used at 10-15% opacity for section background tints (e.g., the ingredients section has a barely-perceptible warm orange wash).
- Interactive elements (checkboxes, toggles, active buttons) use the full accent color.
- Text is never set in the accent color at sizes below 14pt — readability risk on both light and dark backgrounds.
- The category badge in the thumbnail corner uses the accent color at 90% opacity with white text.

### 4.2 Dark Mode

- **Card background:** iOS `Color(.secondarySystemGroupedBackground)` / Android `MaterialTheme.colorScheme.surfaceVariant`. Not pure black — use elevated surface colors to maintain card depth.
- **Thumbnail overlay:** In dark mode, the category badge uses a slightly more saturated accent color (the "Dark" hex values above) for contrast against darker thumbnails.
- **Section dividers:** 1px lines use `Color(.separator)` (iOS) / `MaterialTheme.colorScheme.outlineVariant` (Android). Visible but not harsh.
- **Map snapshots:** Request dark-mode styled map tiles from MapKit (`MKStandardMapConfiguration` with `.darkGray` style) / Google Maps (dark mode JSON styling). This prevents a blinding white map rectangle in an otherwise dark card.
- **Image overlays:** The Staq branding bar on shared card images auto-adapts: dark background + light text when the user's device is in dark mode at share time.

### 4.3 Typography Hierarchy

Consistent type scale across all card types. Uses the system font (SF Pro on iOS, Roboto on Android) for platform-native feel.

| Role | Size | Weight | Line Height | Usage |
|------|------|--------|-------------|-------|
| Card Title | 22pt | Bold (700) | 28pt | Main content title |
| Section Header | 16pt | Semibold (600) | 22pt | "INGREDIENTS", "STEPS", "HIGHLIGHTS" |
| Body / Content | 14pt | Regular (400) | 20pt | Ingredient names, tips, step descriptions |
| Metadata | 12pt | Light (300) | 16pt | Creator handle, platform, timestamp, "edited" indicators |
| Chips / Badges | 11pt | Medium (500) | 14pt | Category badge, dietary tags, difficulty level |

**Scaling:** All sizes are specified in `pt` (iOS) / `sp` (Android) and respect Dynamic Type / system font scaling. At the two largest accessibility sizes, section headers drop their ALL-CAPS treatment (research shows all-caps is harder to read at large sizes for users with low vision).

### 4.4 Platform Design Compliance

- **iOS:** Follows Human Interface Guidelines. Cards use `GroupedInsetListStyle` visual language — rounded corners (12pt radius), grouped sections with inset padding (16pt horizontal). Navigation uses `NavigationStack` with large title collapsing to inline. Bottom sticky CTA uses `safeAreaInset(edge: .bottom)` to avoid home indicator overlap.
- **Android:** Follows Material Design 3. Cards use `ElevatedCard` with `shape = RoundedCornerShape(16.dp)`. Section headers use `LabelLarge` type role. Bottom sticky CTA uses `Scaffold` `bottomBar` with proper `WindowInsets` padding. Color accents are applied via `MaterialTheme` dynamic color where available (Android 12+), falling back to the static palette above on older versions.

---

## 5. Sharing & Virality

### 5.1 Share Card Image Generation

Share images are generated entirely on-device — no server round-trip required. This keeps sharing instant and works offline.

**iOS Implementation:**

```swift
func generateShareImage(for card: SavedContent, format: ShareFormat) -> UIImage {
    let size = format.canvasSize  // e.g., CGSize(width: 1080, height: 1920) for Story
    let renderer = UIGraphicsImageRenderer(size: size)

    return renderer.image { context in
        // 1. Draw background (white or dark based on user's current appearance)
        drawBackground(in: context, size: size)

        // 2. Draw thumbnail (cropped to 16:9 at the top)
        drawThumbnail(card.thumbnail, in: context, rect: thumbnailRect(for: format))

        // 3. Draw category accent bar
        drawAccentBar(category: card.category, in: context)

        // 4. Draw structured content (varies by card type)
        drawCardContent(card, in: context, format: format)

        // 5. Draw QR code / dynamic link
        drawQRCode(for: card.shareURL, in: context, position: qrPosition(for: format))

        // 6. Draw Staq branding footer
        drawBranding(in: context, size: size)
    }
}
```

**Android Implementation:**

```kotlin
fun generateShareImage(card: SavedContent, format: ShareFormat): Bitmap {
    val size = format.canvasSize  // e.g., Size(1080, 1920)
    val bitmap = Bitmap.createBitmap(size.width, size.height, Bitmap.Config.ARGB_8888)
    val canvas = Canvas(bitmap)

    // Same drawing sequence as iOS
    drawBackground(canvas, size)
    drawThumbnail(canvas, card.thumbnail, format)
    drawAccentBar(canvas, card.category)
    drawCardContent(canvas, card, format)
    drawQRCode(canvas, card.shareURL, format)
    drawBranding(canvas, size)

    return bitmap
}
```

### 5.2 Share Format Options

When the user taps "Share Card", a format picker appears before the share sheet:

| Format | Dimensions | Aspect Ratio | Best For |
|--------|-----------|--------------|----------|
| Instagram Story | 1080 x 1920 px | 9:16 | Stories, TikTok, Reels |
| Square | 1080 x 1080 px | 1:1 | Instagram feed, Twitter |
| Standard | 1080 x 1350 px | 4:5 | Instagram feed (max vertical), Pinterest |

- Default selection is "Story" (9:16) since that's the most common sharing context for this demographic.
- Format picker shows a live preview thumbnail of the generated image in each format before the user commits.
- Content layout adapts per format: Story format has more vertical space and shows full ingredient lists; Square format condenses to key highlights only (title, thumbnail, 3-4 key data points, branding).

### 5.3 QR Code / Dynamic Link in Shared Images

Every shared card image includes a small QR code in the bottom-right corner (above the branding bar). Scanning the QR code opens a Staq deep link:

- **For existing users:** Opens the shared card directly in their Staq app.
- **For non-users:** Opens the App Store / Play Store listing with attribution parameters (campaign=share, card_type=recipe, referrer=user_id).
- QR code is generated on-device using `CIFilter.qrCodeGenerator()` (iOS) / `com.google.zxing` (Android). No network call.
- Dynamic links use Firebase Dynamic Links (or Branch.io) for cross-platform deep linking and install attribution.
- QR code size: 80x80px — small enough to be unobtrusive, large enough to scan reliably from a phone screen.

### 5.4 WhatsApp Text Share (Low-Bandwidth Option)

For markets like India where data costs are a concern and image sharing over WhatsApp consumes meaningful bandwidth, offer a "Share as Text" option alongside the image share.

**Format:**

```
15-Minute Garlic Pasta

Ingredients:
- 200g spaghetti
- 4 cloves garlic, minced
- 3 tbsp olive oil
- 1/2 cup parmesan, grated
- Red pepper flakes
- Fresh parsley

15 min | 2 servings | Italian | Easy

Saved with Staq - https://staq.app/s/abc123
```

- The link at the bottom is a short dynamic link that opens the card in Staq or the app store.
- Text share bypasses image generation entirely — just formats the extracted data as clean plain text and opens the system share sheet with the text payload.
- On Android, detect if the share target is WhatsApp and auto-suggest "Share as Text" as the default for users in India (locale check: `en_IN`, `hi_IN`, or device region = IN).

---

## 6. Card Component Architecture

### 6.1 Template Registry Pattern

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
class EducationCardTemplate: CardTemplate { ... }
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

### 6.2 Extracted Data Schema

```json
{
  "recipe": {
    "dish_name": "Garlic Pasta",
    "ingredients": [
      { "item": "spaghetti", "quantity": "200g", "quantity_metric": { "value": 200, "unit": "g" }, "quantity_imperial": { "value": 7, "unit": "oz" } },
      { "item": "garlic", "quantity": "4 cloves", "prep": "minced" }
    ],
    "steps": [
      { "text": "Boil pasta until al dente", "time_seconds": null },
      { "text": "Saute garlic in olive oil for 3 minutes", "time_seconds": 180 }
    ],
    "prep_time": "15 minutes",
    "servings": 2,
    "cuisine": "Italian",
    "difficulty": "Easy",
    "dietary_tags": [
      { "tag": "vegetarian", "confidence": "high" },
      { "tag": "gluten_free", "confidence": "low", "ambiguous_ingredient": "spaghetti" }
    ]
  },
  "travel": {
    "location_name": "Nyang Nyang Beach",
    "coordinates": { "lat": -8.8214, "lng": 115.1162 },
    "country": "Indonesia",
    "region": "Bali",
    "best_time": "May-September",
    "budget_range": "$30-50/day",
    "budget_amount": { "min": 30, "max": 50, "currency": "USD", "period": "day" },
    "tips": ["Go early morning", "Bring water"],
    "trip_id": null
  },
  "product": {
    "product_name": "FlexiSpot E7 Pro",
    "brand": "FlexiSpot",
    "price": 299.99,
    "currency": "USD",
    "store": "Amazon",
    "purchase_url": "https://...",
    "affiliate_url": null,
    "product_category": "Standing Desks",
    "rating": 4.7,
    "key_specs": ["Height adjustable", "Electric motor"],
    "price_tracking_enabled": false
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
  },
  "education": {
    "topic": "Backend Development",
    "subtopic": "REST API Design",
    "key_takeaways": [
      "Express.js handles routing and middleware",
      "Use async/await for database queries"
    ],
    "tools_resources": [
      { "name": "Node.js", "url": "https://nodejs.org" },
      { "name": "Express.js", "url": "https://expressjs.com" }
    ],
    "difficulty": "intermediate",
    "estimated_learning_time": "45 minutes",
    "is_learned": false
  }
}
```

---

## 7. "More" Menu Actions

The ... button in top-right opens a bottom sheet:

```
+--------------------------------------+
|  View Original                        |
|  Add to Collection                    |
|  Recategorize                         |
|  Edit Note                            |
|  Share Card                           |
|  Copy Extracted Text                  |
|  Delete Save                          |
|  ----------------------------         |
|  Cancel                               |
+--------------------------------------+
```

**"Share Card":** Generates a visual card image (recipe card as PNG) shareable via messaging apps. Viral growth mechanism — friends see the card, wonder what app made it, download Staq.

**"Copy Extracted Text":** Copies all structured data as formatted plain text to clipboard. Useful for pasting recipe into Notes, sending ingredients to someone, etc.

---

## 8. Analytics Events

Track user interactions with structured cards to understand which card types drive the most value, where users edit AI-extracted data (signal for extraction quality), and which features drive sharing and retention.

| Event Name | Parameters | Trigger |
|------------|-----------|---------|
| `card_viewed` | `card_type`, `richness_score`, `has_deep_extract` | Card detail screen appears for > 1 second (debounce accidental taps) |
| `card_field_edited` | `card_type`, `field_name` | User saves an inline edit on any extracted field |
| `ingredient_checked` | `recipe_card_id`, `checked_count`, `total_count` | User checks/unchecks an ingredient checkbox |
| `view_original_tapped` | `platform`, `card_type` | User taps "View Original" (sticky or inline) |
| `share_card_generated` | `card_type`, `format` (story/square/standard/text) | Share image or text is generated and share sheet opens |
| `shopping_list_export_tapped` | `ingredient_count`, `is_pro` | User taps "Add to Shopping List" (track Pro conversion) |
| `map_opened` | `location_name`, `platform` (apple_maps/google_maps) | User taps "Open in Maps" or taps the map snapshot |
| `unit_conversion_toggled` | `card_type`, `from_system`, `to_system` | User switches the metric/imperial toggle |
| `serving_size_adjusted` | `card_type`, `original_servings`, `new_servings` | User changes the serving count |
| `timer_started` | `card_type`, `duration_seconds`, `step_index` | User taps a time chip to start a countdown |
| `timer_completed` | `card_type`, `duration_seconds`, `was_backgrounded` | Timer reaches zero |
| `dietary_tag_overridden` | `card_type`, `tag_name`, `action` (added/removed) | User manually adds or removes a dietary tag |
| `save_to_trip_tapped` | `trip_id`, `is_new_trip` | User assigns a travel card to a trip |
| `weather_loaded` | `location_name`, `success` (bool) | Weather widget fetch completes or fails |
| `price_drop_alert_enabled` | `product_name`, `price_at_save`, `store` | User enables price drop notification |
| `similar_save_tapped` | `source_card_type`, `target_card_id` | User taps a similar save to navigate to it |
| `mark_as_learned_toggled` | `card_type`, `new_state` (learned/not_learned) | User toggles the learned state on an education card |
| `qr_code_scanned` | `referrer_user_id`, `card_type` | A shared QR code is scanned (tracked via dynamic link) |

**Implementation notes:**
- Use Firebase Analytics (both platforms) or a lightweight custom event pipeline.
- All events include implicit context: `user_id`, `device_platform`, `app_version`, `timestamp`.
- `richness_score` is a 0-100 value representing how much structured data the card contains (a recipe with ingredients + steps + times + servings scores higher than one with just a title and ingredients). Useful for correlating extraction quality with user engagement.
- Events are batched and sent every 30 seconds or on app background — not on every interaction. Minimizes battery and network impact.

---

## 9. Performance Benchmarks

Card rendering must feel instant. Users are coming from Instagram/TikTok where content loads in milliseconds — any perceptible delay in showing a structured card erodes trust in the app's quality.

| Operation | Target | Measurement Method | Notes |
|-----------|--------|-------------------|-------|
| Card initial render (layout + data bind) | < 100ms | `os_signpost` (iOS) / `Trace.beginSection` (Android) from `onAppear` to first frame drawn | Includes JSON deserialization, template selection, and SwiftUI/Compose layout. Does NOT include image loading (async). |
| Map snapshot load | < 500ms | Time from map request to snapshot callback | Uses `MKMapSnapshotter` (iOS) / Static Maps API (Android). Cache snapshots for previously viewed locations. If > 500ms, show a placeholder with the location name and a subtle loading shimmer. |
| Image generation for share | < 1 second | Time from "Share Card" tap to share sheet appearing | `UIGraphicsImageRenderer` (iOS) / `Canvas` draw (Android). Profile and optimize: thumbnail decode is usually the bottleneck — keep a pre-decoded bitmap in memory when the card is open. |
| Ingredient checklist toggle | < 16ms (single frame) | Frame time during checkbox tap | Must be synchronous state update + re-render in one frame. No async work on the main thread for this interaction. Use `@State` (SwiftUI) / `mutableStateOf` (Compose) for instant reactivity. |
| Unit conversion toggle | < 50ms | Time from toggle tap to all quantities updating | Conversion is a pure local computation on the ingredient array. Pre-compute both unit systems at card load time and swap the display array on toggle — no recalculation needed at tap time. |
| Serving size adjustment | < 50ms | Time from stepper tap to quantities updating | Same pre-computation strategy as unit conversion. For each serving count 1-12, pre-build the scaled ingredient list at card load. Swap on tap. |
| Timer start | < 100ms | Time from chip tap to timer UI appearing | Schedule local notification immediately. Timer display uses a lightweight `Timer`/`CountDownTimer` ticking at 1-second intervals — not every frame. |
| Hero transition (grid -> card) | < 300ms | Total animation duration | Use `matchedGeometryEffect` (SwiftUI) / `SharedElementTransition` (Compose). Thumbnail image is already in memory from the grid cell — no re-decode. |
| Weather fetch | < 2 seconds | Time from card open to weather row populating | Network-dependent. Show a shimmer placeholder during load. If > 2 seconds, show the card without weather and append it when ready (no layout jump — reserve the row height upfront). |

**Profiling cadence:** Run performance benchmarks on low-end devices (iPhone SE 2nd gen, Pixel 4a) during every release cycle. Cards must meet these targets on 3-year-old hardware, not just flagship phones.

---

## 10. Edge Cases

| Scenario | Handling |
|----------|----------|
| AI extracted ingredients but no steps | Show ingredients section, hide steps section. Show "Get Full Details" to retrieve steps. Don't show an empty "Steps" header. |
| Travel location couldn't be geocoded | Show location name as text without map embed. "Open in Maps" searches for the name as a string query. Hide weather widget and distance (no coordinates to query). |
| Product price in different currency than user's | Show original currency as extracted. Show converted amount in parentheses using cached exchange rate. If conversion unavailable, show original only. |
| Extracted data is partially wrong | User can tap any field to edit via inline popover. Edits persist and are marked with "edited". Edits are never overwritten by Deep Extract. |
| Deep extract returns data that changes the card type | Re-render with new template. Animate transition: old sections fade out, new sections fade in. User note and manual edits carry over. |
| Extremely long ingredient list (30+ items) | Collapse after 10 items with "Show all (30)" expander. Expanded state persists. |
| Video thumbnail fails to load | Show platform-colored gradient background (Instagram purple, TikTok black/pink, YouTube red) with centered platform icon and title overlaid. |
| User opens card while Deep Extract is running | Show current partial data. Sections populate live as extraction progresses. Show "Analyzing video..." spinner in sections that are still loading. |
| Product purchase link is broken or 404 | "View on Amazon" button still works (opens the URL). If link was verified broken at extract time, show "Link may not be available." |
| Multiple locations in a travel post | Show primary location as map pin. List secondary locations as text links below. Each tappable to open in Maps. |
| Card content is extremely short (only a title) | Still render the full card layout but center the title. Show "Get Full Details" prominently. Don't show empty section headers. |
| Unit conversion for non-standard quantities | Ingredients without parseable quantities ("a pinch of salt", "to taste") are shown as-is and not converted. Only numeric quantities with recognized units are toggled. |
| Serving adjuster with non-scalable ingredients | Some ingredients don't scale linearly (e.g., "1 egg" doesn't become "1.5 eggs" at 3 servings — it becomes "2 eggs"). Use ceiling rounding for count-based items (items without weight/volume units). |
| Multiple timers running simultaneously | Stack up to 3 timer bars below the steps section header. If a 4th timer is started, show a toast: "Maximum 3 timers active. Stop one to start another." |
| Weather API rate limit or failure | Hide the weather row gracefully. No error state shown — weather is supplementary, not core. Log the failure in analytics for monitoring. |
| Affiliate link for unsupported retailer | Show the original purchase URL without modification. Only append affiliate tags for supported programs (Amazon, Impact network). |
| Education card with no extractable takeaways | Show the topic and difficulty fields. Replace "Key Takeaways" section with a "Get Full Details" prompt to trigger deeper extraction. |
| Dietary tag false positive | Users can tap any tag to remove it. If a tag is removed by the user, it does not reappear on subsequent extractions. Show a brief toast: "Tag removed. It won't come back." |
| Price drop notification for deleted product | If the user deletes the save, cancel any pending price tracking for that product. If the product URL becomes invalid, disable tracking and remove the notification silently. |
| Share image with very long title | Truncate the title to 2 lines in the share image with an ellipsis. Full title is available when the recipient opens the dynamic link. |

---

## 11. Accessibility

- **VoiceOver / TalkBack:** Recipe ingredients announced as "Ingredient 1 of 6: 200g spaghetti. Unchecked. Double tap to check." Steps announced as "Step 1 of 4: Boil pasta until al dente."
- **Dynamic Type:** All card text scales with system font size. At largest sizes, the metadata chips (prep time, servings) stack vertically instead of horizontally.
- **Map embed:** Provides accessible label: "Map showing Nyang Nyang Beach, Bali, Indonesia. Double tap to open in Maps."
- **Inline editing:** Edit popover is focus-trapped. VoiceOver announces "Editing prep time. Current value: 15 minutes."
- **Ingredient checkboxes:** State changes announced: "Checked: 200g spaghetti. 1 of 6 checked."
- **Reduce Motion:** Hero transition replaced with crossfade. Parallax scrolling header replaced with simple collapse.
- **Minimum tap targets:** All interactive elements >= 44x44pt. Ingredient checkbox rows have full-width tap area.
- **Unit conversion toggle:** VoiceOver announces: "Unit system: Metric. Double tap to switch to Imperial." After toggle: "Switched to Imperial. All quantities updated."
- **Serving adjuster:** VoiceOver announces stepper as: "Servings: 2. Adjustable. Swipe up to increase, swipe down to decrease." After adjustment: "Servings changed to 4. Ingredient quantities updated."
- **Timer:** VoiceOver announces tappable time chips as: "Start 3-minute timer for step 2. Double tap to start." Running timer announced as: "Timer: 2 minutes 38 seconds remaining. Double tap to cancel."
- **Dietary tags:** Each tag announced as: "Dietary tag: Vegetarian, high confidence." Low-confidence tags announced as: "Dietary tag: Gluten-Free, uncertain. Double tap for details."
- **Education card "Mark as Learned":** Announced as: "Mark as Learned. Toggle. Currently not learned. Double tap to mark as learned."
- **Weather widget:** Announced as: "Current weather at Nyang Nyang Beach: 29 degrees Celsius, 84 Fahrenheit, Partly Cloudy."

---

## 12. Development Plan

### 12.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Define ExtractedData JSON schema for all categories (including education) | Shared | 5h | F2 |
| 2 | Build CardTemplate protocol + registry pattern (add EducationCardTemplate) | Both | 5h | -- |
| 3 | Build common card header with collapsing scroll + user note | Both | 6h | -- |
| 4 | Build Recipe card template with ingredient checklist + counter | Both | 6h | Tasks 1-3 |
| 5 | Build unit conversion toggle with locale-aware defaults | Both | 4h | Task 4 |
| 6 | Build serving size adjuster with proportional scaling | Both | 4h | Task 4 |
| 7 | Build inline timer system (chip detection, countdown, notifications) | Both | 6h | Task 4 |
| 8 | Build dietary tag detection and display | Both | 4h | Task 4 |
| 9 | Build Travel card template with static map snapshot | Both | 6h | Tasks 1-3 |
| 10 | Build weather widget with OpenWeatherMap integration | Both | 4h | Task 9 |
| 11 | Build distance-from-user calculation | Both | 2h | Task 9 |
| 12 | Build currency conversion for budget amounts | Both | 3h | Task 9 |
| 13 | Build "Save to Trip" integration (ties into F6) | Both | 4h | Task 9, F6 |
| 14 | Build Product card template with purchase link + staleness warning | Both | 4h | Tasks 1-3 |
| 15 | Build affiliate link tag injection (Amazon Associates) | Both | 3h | Task 14 |
| 16 | Build "Similar Saves" cross-reference section | Both | 4h | Task 14 |
| 17 | Build price tracking teaser UI (Phase 2 placeholder) | Both | 2h | Task 14 |
| 18 | Build Fitness card template | Both | 4h | Tasks 1-3 |
| 19 | Build Beauty/Fashion card template | Both | 4h | Tasks 1-3 |
| 20 | Build Education/Learning card template with "Mark as Learned" | Both | 6h | Tasks 1-3 |
| 21 | Build Generic card template with contextual "Get Full Details" | Both | 3h | Tasks 1-3 |
| 22 | Build "More" menu bottom sheet with all actions | Both | 3h | -- |
| 23 | Build inline field editing popover with persistence | Both | 5h | Tasks 4-21 |
| 24 | Implement "Share Card" image generation (3 formats + QR code) | Both | 6h | Tasks 4-21 |
| 25 | Implement WhatsApp text share for low-bandwidth markets | Both | 2h | Task 24 |
| 26 | Implement "Copy Extracted Text" formatting per card type | Both | 2h | Tasks 4-21 |
| 27 | Build "View Original" deep-link handler per platform | Both | 3h | -- |
| 28 | Build sticky floating "View Original" CTA on scroll | Both | 2h | Task 3 |
| 29 | Hero transition from library grid to card detail | Both | 3h | F3 |
| 30 | Implement card theming (accent colors, dark mode, typography) | Both | 4h | Tasks 4-21 |
| 31 | Integrate analytics events across all card interactions | Both | 4h | Tasks 4-21 |
| 32 | Performance profiling + optimization on low-end devices | Both | 4h | All above |
| 33 | Accessibility audit per card type (including new interactions) | Both | 5h | Tasks 4-21 |
| 34 | QA: all card types render correctly with varying data completeness | QA | 10h | All above |

**Total: ~146 hours (~4-5 weeks with one developer per platform)**

### 12.2 Definition of Done

- [ ] All 8 card types render correctly (Recipe, Travel, Product, Fitness, Beauty/Fashion, Education, Generic)
- [ ] Template registry auto-selects correct card type based on category + data
- [ ] Recipe ingredient checklist is interactive with progress counter and strikethrough
- [ ] Unit conversion toggle switches all quantities between metric and imperial with friendly rounding
- [ ] Serving size adjuster scales ingredient quantities proportionally (1-12 servings)
- [ ] Tappable time values in recipe steps launch inline countdown timers with local notifications
- [ ] Dietary tags are auto-detected from ingredients and user-editable
- [ ] Travel card shows static map snapshot with pin, tappable to open in Maps
- [ ] Travel card shows weather widget, distance from user, and converted currency for budget
- [ ] "Save to Trip" assigns travel cards to trip collections (F6 integration)
- [ ] Product card shows purchase link and staleness warning for saves > 7 days old
- [ ] Affiliate tags are appended to supported retailer links at open time
- [ ] "Similar Saves" section shows other saved products in same category
- [ ] Price tracking teaser UI is in place for Phase 2
- [ ] Education card shows topic, takeaways, tools, difficulty, and "Mark as Learned" toggle
- [ ] Generic card shows "Get Full Details" with contextual subtext for new users
- [ ] "View Original" sticky CTA floats at bottom when user scrolls past thumbnail
- [ ] "Share Card" generates clean branded image in three formats (9:16, 1:1, 4:5) with QR code
- [ ] WhatsApp text share option works for low-bandwidth sharing
- [ ] Inline field editing works on all data fields across all card types
- [ ] User edits persist and are never overwritten by subsequent Deep Extracts
- [ ] Collapsing scroll header works with parallax (and falls back to simple collapse with Reduce Motion)
- [ ] User note is editable from card view
- [ ] Card handles partial data gracefully (shows available sections, hides empty ones, never shows empty headers)
- [ ] Card theming: accent colors, dark mode, and typography hierarchy are consistent across all card types
- [ ] All analytics events fire correctly with proper parameters
- [ ] Performance: card render < 100ms, map load < 500ms, share image < 1s, checkbox toggle < 16ms on low-end devices
- [ ] Accessibility: VoiceOver/TalkBack announces all interactive elements correctly, including new interactions (timers, toggles, steppers)
