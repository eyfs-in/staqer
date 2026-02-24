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
| Travel | ✈️ | travel, visit, hotel, flight, beach, trip, itinerary, destination | Location, tips, best time, budget, things to do |
| Products | 🛍️ | buy, price, amazon, discount, sale, link in bio, code, $, review | Product name, brand, price, link, rating |
| Tech/Gadgets | 📱 | phone, laptop, gadget, specs, benchmark, unboxing, camera test, processor, RAM, review, comparison, best phone | Device name, specs, rating, pros/cons, price range |
| Fitness | 💪 | workout, exercise, reps, sets, cardio, gym, HIIT, yoga, stretch | Exercise names, sets/reps, duration, muscle groups |
| Beauty | 💄 | skincare, makeup, routine, serum, SPF, moisturizer, foundation | Product names, routine steps, skin type |
| Fashion | 👗 | outfit, style, OOTD, haul, try-on, size, wear | Items, brands, sizes, occasion, season |
| DIY & Home | 🏠 | DIY, home, decor, organize, hack, before and after, renovation | Materials, steps, tools, cost |
| Finance | 💰 | invest, stock, mutual fund, SIP, budget, savings, crypto, portfolio | Instruments, strategies, amounts, terms |
| Entertainment | 🎬 | watch, movie, show, series, Netflix, review, trailer, must-watch | Title, platform, genre, rating, review |
| Education/Learning | 📚 | tutorial, course, learn, study, tips, how to, explained, guide, certification, class, lecture, notes | Topic, skill level, platform, duration, key takeaways |
| Parenting/Kids | 👶 | parenting, baby, toddler, kids, mom, dad, school, milestone, activity, craft for kids, motherhood | Age group, activity type, materials, developmental stage |
| Inspiration | ✨ | quote, motivation, mindset, affirmation, productivity, life | Quote text, author, theme |
| General | 📌 | (default fallback) | Title, summary |

**Tech/Gadgets vs. Products distinction:** Tech/Gadgets captures review/comparison intent (someone evaluating a device), while Products captures purchase intent (someone linking to a deal or drop). A post titled "iPhone 16 Pro Camera Review" is Tech/Gadgets; a post saying "iPhone 16 Pro $200 off, link in bio" is Products. When both signals are present, prefer the stronger intent and show the other as a secondary tag.

**Education/Learning rationale:** Huge content vertical on Instagram and YouTube in India (study tips, exam prep, coding tutorials). Distinct from Inspiration — Education content has actionable learning structure, not just motivational quotes.

**Parenting/Kids rationale:** Fast-growing segment on Instagram Reels globally. These posts have distinct extraction patterns (age-appropriate activities, milestone tracking) that don't fit cleanly into other categories.

### 3.2 Sub-Category Support (Phase 2)

In Phase 2, top-level categories will support optional sub-categories to enable finer-grained organization. Sub-categories are user-facing but not required — content is always categorized at the top level first, and sub-categorization is additive.

**Planned sub-category structure:**

| Category | Sub-Categories (Phase 2) |
|----------|--------------------------|
| Recipes | Indian, Italian, Mexican, Chinese, Desserts, Vegan, Quick Meals |
| Travel | Beach, Mountain, City, Budget, Luxury, Road Trip |
| Fitness | Gym, Yoga, Running, Home Workout, Stretching |
| Education/Learning | Coding, Language, Exam Prep, Career, Creative Skills |
| Tech/Gadgets | Phones, Laptops, Audio, Cameras, Smart Home |

**Implementation plan:** Sub-categories will be inferred by the AI tier using the same prompt (extended to return `sub_category` field). For Tier 2, sub-category assignment uses a secondary keyword pass within the winning top-level category. The data model should include an optional `subCategory` field from Phase 1 to avoid schema migrations later — it simply remains `null` until Phase 2 ships.

### 3.3 Custom Categories

Users can create up to 20 custom categories (free: 5, paid: 20) with:
- Custom name (max 30 characters)
- Emoji icon (from system emoji picker)
- Optional keyword hints (helps classifier learn user intent)

Example: User creates "Wedding Planning 💒" with keywords: "wedding, venue, dress, flowers, registry"

---

## 4. Three-Tier Processing Architecture

### 4.1 Tier 1: On-Device AI (Flagship Phones)

**Devices:** iPhone 15 Pro+ (A17 Pro chip required), Pixel 8+, Galaxy S24+, OnePlus 13, Xiaomi 15

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
│  recipe, travel, product,           │
│  tech_gadgets, fitness, beauty,     │
│  fashion, diy, finance,             │
│  entertainment, education,          │
│  parenting, inspiration, general.   │
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

#### 4.1.1 iOS — Apple Foundation Models

**Hardware requirement:** iPhone 15 Pro or later (A17 Pro chip). Foundation Models framework is available starting iOS 26. On devices with the right hardware but where the model has not yet been downloaded, the framework will trigger a background download — but this can take minutes to hours depending on network conditions.

**Structured Output with `@Generable`:**

Starting in iOS 26, Foundation Models supports the `@Generable` macro for type-safe structured output. This eliminates the need for raw JSON parsing and handles schema validation at the framework level:

```swift
import FoundationModels

@Generable
struct ContentClassification {
    @Guide(description: "One of: recipe, travel, product, tech_gadgets, fitness, beauty, fashion, diy, finance, entertainment, education, parenting, inspiration, general")
    var category: String

    @Guide(description: "Confidence from 0.0 to 1.0")
    var confidence: Double

    @Guide(description: "Short descriptive title for the saved content")
    var title: String

    @Guide(description: "Extracted structured data relevant to the category")
    var extractedData: ExtractedData?
}

@Generable
struct ExtractedData {
    var dishName: String?
    var ingredients: [String]?
    var prepTime: String?
    var cuisine: String?
    var location: String?
    var productName: String?
    var price: String?
    var exerciseNames: [String]?
    // ... additional fields per category
}
```

**Session Management and Token Limits:**

On-device models have a limited context window (~4K tokens). Social media captions typically fit well within this, but long captions with extensive OCR text can exceed the limit. Manage this proactively:

```swift
import FoundationModels

func classifyContent(caption: String, ocrText: String, hashtags: [String]) async throws -> ContentClassification {
    let session = LanguageModelSession()

    // Truncate inputs to stay within ~4K token budget
    // Reserve ~500 tokens for system prompt + output
    let truncatedCaption = String(caption.prefix(1500))
    let truncatedOCR = String(ocrText.prefix(500))
    let topHashtags = Array(hashtags.prefix(15)).joined(separator: ", ")

    let prompt = """
    Classify this social media post. Pick exactly one category from: recipe, travel, product, tech_gadgets, fitness, beauty, fashion, diy, finance, entertainment, education, parenting, inspiration, general.

    Extract relevant structured data based on the category.

    Caption: "\(truncatedCaption)"
    On-screen text: "\(truncatedOCR)"
    Hashtags: \(topHashtags)
    """

    // Use @Generable structured output
    let result = try await session.respond(to: prompt, generating: ContentClassification.self)
    return result
}
```

**Error Handling — Model Availability:**

The on-device model may be unavailable for several reasons. Handle each gracefully and fall back to Tier 2:

```swift
func classifyWithTier1(caption: String, ocrText: String, hashtags: [String]) async -> ClassificationResult {
    do {
        guard FoundationModels.isAvailable else {
            // Model not supported on this hardware (pre-A17 Pro)
            return await classifyWithTier2(caption: caption, hashtags: hashtags)
        }

        let session = LanguageModelSession()
        let result = try await session.respond(to: prompt, generating: ContentClassification.self)
        return .success(result, tier: .tier1)

    } catch let error as LanguageModelSession.GenerationError {
        switch error {
        case .modelNotReady:
            // Model is still downloading — fall back silently
            // Log: "foundation_model_not_downloaded"
            return await classifyWithTier2(caption: caption, hashtags: hashtags)
        case .modelBusy:
            // Another process is using the model — retry once after 500ms, then fall back
            try? await Task.sleep(nanoseconds: 500_000_000)
            // retry or fall back
            return await classifyWithTier2(caption: caption, hashtags: hashtags)
        case .guardrailViolation:
            // Model refused the content — classify as General
            return .success(Classification(category: "general", confidence: 0.3), tier: .tier1)
        default:
            return await classifyWithTier2(caption: caption, hashtags: hashtags)
        }
    } catch {
        // JSON decode failure or unexpected error
        return await classifyWithTier2(caption: caption, hashtags: hashtags)
    }
}
```

#### 4.1.2 Android — Gemini Nano via Prompt API

**Current status:** The Gemini Nano Prompt API is in **Early Access** (not GA). API surface and availability may change. Plan for this instability in the architecture — wrap all Gemini Nano calls behind a clean abstraction layer so that API changes require minimal code updates.

**Model quality note:** Gemini Nano is a 1.8B parameter model, while Apple's on-device Foundation Model is estimated at ~3B parameters. For classification tasks specifically, this gap may result in lower accuracy on ambiguous content (multi-category posts, sarcasm, non-English text). Monitor Android vs. iOS accuracy separately in analytics and tune Android prompts independently if divergence exceeds 5%.

**Implementation:**

```kotlin
import com.google.ai.client.generativemodel.GenerativeModel
import com.google.ai.edge.aicore.AICore

suspend fun classifyWithGeminiNano(
    caption: String,
    ocrText: String,
    hashtags: List<String>
): ClassificationResult {
    // Step 1: Check AICore service availability
    val aiCoreStatus = AICore.getAvailability(context)

    when (aiCoreStatus) {
        Availability.AVAILABLE -> { /* proceed */ }
        Availability.UNAVAILABLE_MODEL_NOT_DOWNLOADED -> {
            // User has a supported device but model isn't downloaded yet.
            // Request download in background, fall back to Tier 2 now.
            AICore.requestModelDownload(context)
            return classifyWithTier2(caption, hashtags)
        }
        Availability.UNAVAILABLE_DEVICE_NOT_SUPPORTED -> {
            // Hardware doesn't support Gemini Nano
            return classifyWithTier2(caption, hashtags)
        }
        Availability.UNAVAILABLE_USER_NOT_OPTED_IN -> {
            // Device supports it, but user hasn't enabled on-device AI
            // in Android settings. Don't nag — silently fall back.
            return classifyWithTier2(caption, hashtags)
        }
        else -> return classifyWithTier2(caption, hashtags)
    }

    // Step 2: Create GenerativeModel via Google AI client SDK
    val model = GenerativeModel(
        modelName = "gemini-nano",
        // No API key needed for on-device inference
    )

    val truncatedCaption = caption.take(1500)
    val truncatedOCR = ocrText.take(500)
    val topHashtags = hashtags.take(15).joinToString(", ")

    val prompt = """
        Classify this social media post. Pick exactly one category from: recipe, travel, product, tech_gadgets, fitness, beauty, fashion, diy, finance, entertainment, education, parenting, inspiration, general.

        Extract relevant structured data based on the category.

        Caption: "$truncatedCaption"
        On-screen text: "$truncatedOCR"
        Hashtags: $topHashtags

        Respond with JSON only, no other text:
        {
          "category": "recipe",
          "confidence": 0.92,
          "title": "15-Minute Garlic Pasta",
          "extracted_data": { ... }
        }
    """.trimIndent()

    return try {
        val response = model.generateContent(prompt)
        val classification = gson.fromJson(response.text, Classification::class.java)
        ClassificationResult.Success(classification, tier = Tier.TIER_1)
    } catch (e: Exception) {
        // Gemini Nano failure — fall back to Tier 2
        // Log: "gemini_nano_inference_failed", error = e.message
        classifyWithTier2(caption, hashtags)
    }
}
```

**Android fallback ladder:** On Android, the fallback path has more steps because Gemini Nano availability is not guaranteed even on "supported" hardware:

```
Gemini Nano available? → YES → Use Tier 1
                        → NO  → Is device supported but model not downloaded?
                                  → YES → Request background download, use Tier 2.5/2 now
                                  → NO  → Is user not opted in?
                                           → YES → Use Tier 2.5/2 (don't prompt user)
                                           → NO  → Device unsupported, use Tier 2.5/2
```

#### 4.1.3 Prompt Versioning Strategy

Prompts are living artifacts. Different prompt wordings can meaningfully shift accuracy, especially on smaller on-device models. Treat prompts like code — version them, test them, measure them.

**Versioning scheme:**

```swift
struct PromptVersion {
    let id: String          // e.g., "v2.3-ios", "v2.3-android"
    let platform: Platform  // .iOS or .android (prompts may diverge)
    let promptText: String
    let activeFrom: Date
    let isDefault: Bool
}
```

**A/B testing framework:**

1. **Rollout buckets:** Assign each device a stable bucket ID (hash of device identifier, mod 100). Route bucket 0–89 to the current default prompt, 90–99 to the experimental prompt.
2. **Metrics per version:** Track `categorization_completed` events tagged with `prompt_version`. Compare:
   - Override rate (lower is better — users correcting fewer classifications)
   - Confidence distribution (higher average confidence suggests better calibration)
   - JSON parse failure rate (prompt format issues)
   - Category distribution skew (new prompt shouldn't suddenly classify everything as General)
3. **Promotion criteria:** An experimental prompt graduates to default when:
   - Override rate is equal or lower over 500+ classifications
   - JSON parse failure rate < 2%
   - No single category sees > 20% relative change in volume (avoids category collapse)
4. **Rollback:** If experimental prompt's override rate exceeds default by > 3 percentage points after 200 classifications, auto-disable the experiment and alert.

**Version history is stored locally** as part of the analytics payload. When prompt versions ship via remote config, the device records which version produced each classification. This creates a clean audit trail without requiring a server round-trip during classification.

```swift
// Remote config (fetched periodically, not per-classification)
struct PromptConfig: Codable {
    let defaultPromptVersion: String
    let experimentPromptVersion: String?
    let experimentBucketRange: ClosedRange<Int>  // e.g., 90...99
    let prompts: [String: String]                // version_id → prompt_text
}
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
│  Step 2b: TF-IDF Disambiguation     │
│                                     │
│  When top 2 categories are within   │
│  20% of each other's score,         │
│  apply TF-IDF scoring to break      │
│  the tie using term specificity     │
│  (see Section 4.2.2)               │
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
│    /R\$\s?\d+/  (Brazilian Real)    │
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

#### 4.2.1 Keyword Dictionaries

**Keyword Dictionary Structure — Multilingual:**

Keywords are organized by language to support primary markets (English, Hindi, Portuguese-BR, Arabic, Bahasa Indonesia/Malay). All language variants are checked against the input text simultaneously — no language detection step is needed for keyword matching.

```swift
let categoryKeywords: [String: [(keyword: String, weight: Int)]] = [
    "recipe": [
        // English
        ("recipe", 3), ("ingredients", 3), ("cook", 2), ("bake", 2),
        ("tablespoon", 3), ("teaspoon", 3), ("cup", 2), ("preheat", 3),
        ("oven", 2), ("prep time", 3), ("serving", 2), ("homemade", 1),
        ("cuisine", 2), ("dish", 1), ("simmer", 2), ("saute", 2),
        ("marinate", 2), ("garnish", 2), ("batch cook", 2),

        // Hindi (Devanagari)
        ("रेसिपी", 3), ("मसाला", 2), ("तड़का", 2), ("सब्ज़ी", 2),
        ("पनीर", 2), ("दाल", 2), ("बिरयानी", 2), ("चटनी", 2),
        ("रोटी", 2), ("पराठा", 2), ("खाना", 2), ("पकाना", 2),

        // Hindi transliterated (Latin script — very common on Instagram India)
        ("sabzi", 2), ("masala", 2), ("tadka", 2), ("roti", 2),
        ("paneer", 2), ("dal", 2), ("biryani", 2), ("chutney", 2),
        ("paratha", 2), ("khana", 2), ("pakora", 2), ("samosa", 2),
        ("raita", 2), ("naan", 2), ("korma", 2), ("tikka", 2),
        ("paneer butter masala", 3), ("dal makhani", 3),
        ("chole bhature", 3), ("aloo gobi", 3), ("palak paneer", 3),

        // Portuguese (Brazil)
        ("receita", 3), ("ingredientes", 3), ("cozinhar", 2),
        ("assar", 2), ("forno", 2), ("colher de sopa", 3),
        ("colher de cha", 3), ("preparo", 2), ("modo de fazer", 3),
        ("tempero", 2), ("refogar", 2), ("misturar", 2),
        ("brigadeiro", 2), ("pao de queijo", 2), ("acai", 2),
        ("farofa", 2), ("feijoada", 2), ("tapioca", 2),

        // Arabic
        ("وصفة", 3), ("مكونات", 3), ("طبخ", 2), ("فرن", 2),
        ("ملعقة", 3), ("كوب", 2), ("تحضير", 2), ("طبق", 1),
        ("شوربة", 2), ("كبسة", 2), ("منسف", 2), ("فلافل", 2),
        ("حمص", 2), ("شاورما", 2), ("مقلوبة", 2),

        // Bahasa Indonesia/Malay
        ("resep", 3), ("bahan", 3), ("masak", 2), ("goreng", 2),
        ("rebus", 2), ("sendok makan", 3), ("sendok teh", 3),
        ("oven", 2), ("bumbu", 2), ("tumis", 2), ("kukus", 2),
        ("nasi goreng", 3), ("rendang", 2), ("sate", 2), ("gado-gado", 2),
        ("sambal", 2), ("bakso", 2), ("mie goreng", 3),
    ],
    "travel": [
        // English
        ("travel", 3), ("visit", 2), ("hotel", 2), ("flight", 2),
        ("beach", 2), ("trip", 2), ("itinerary", 3), ("destination", 2),
        ("booking", 2), ("resort", 2), ("explore", 1), ("wanderlust", 1),
        ("backpacking", 2), ("hidden gem", 2), ("must visit", 2),
        ("hostel", 2), ("road trip", 2), ("bucket list", 2),

        // Hindi transliterated
        ("ghumna", 2), ("safar", 2), ("yatra", 2), ("mandir", 1),
        ("pahad", 2), ("darshan", 1), ("ghoomna", 2),

        // Portuguese (Brazil)
        ("viagem", 3), ("viajar", 3), ("hotel", 2), ("voo", 2),
        ("praia", 2), ("passeio", 2), ("roteiro", 3), ("destino", 2),
        ("mochilao", 2), ("hospedagem", 2), ("turismo", 2),

        // Arabic
        ("سفر", 3), ("رحلة", 3), ("فندق", 2), ("شاطئ", 2),
        ("سياحة", 2), ("طيران", 2), ("حجز", 2), ("وجهة", 2),

        // Bahasa Indonesia/Malay
        ("wisata", 3), ("liburan", 3), ("hotel", 2), ("pantai", 2),
        ("jalan-jalan", 2), ("penerbangan", 2), ("destinasi", 2),
        ("backpacker", 2), ("penginapan", 2),
    ],
    "fitness": [
        // English
        ("workout", 3), ("exercise", 3), ("reps", 3), ("sets", 3),
        ("cardio", 2), ("gym", 2), ("HIIT", 3), ("yoga", 2),
        ("stretch", 2), ("gains", 1), ("PR", 1), ("leg day", 2),
        ("abs", 2), ("protein", 1), ("bulk", 1), ("cut", 1),

        // Hindi transliterated
        ("kasrat", 2), ("vyayam", 2), ("yoga", 2), ("dand", 2),

        // Portuguese (Brazil)
        ("treino", 3), ("exercicio", 3), ("academia", 2),
        ("musculacao", 2), ("alongamento", 2), ("abdominais", 2),
        ("corrida", 2), ("supino", 2),

        // Arabic
        ("تمرين", 3), ("رياضة", 2), ("جيم", 2), ("لياقة", 2),
        ("عضلات", 2), ("تمارين", 3),

        // Bahasa Indonesia/Malay
        ("olahraga", 3), ("latihan", 3), ("gym", 2), ("otot", 2),
        ("kardio", 2), ("peregangan", 2),
    ],
    "education": [
        // English
        ("tutorial", 3), ("course", 3), ("learn", 2), ("study", 2),
        ("how to", 2), ("explained", 2), ("guide", 2), ("certification", 3),
        ("lecture", 2), ("notes", 1), ("exam", 2), ("tips", 1),
        ("beginner", 2), ("advanced", 1), ("step by step", 2),

        // Hindi transliterated
        ("padhai", 2), ("padhna", 2), ("sikho", 2), ("exam", 2),
        ("notes", 1), ("class", 1), ("toppers", 2),

        // Portuguese (Brazil)
        ("tutorial", 3), ("curso", 3), ("aprender", 2), ("estudar", 2),
        ("aula", 2), ("dicas", 1), ("concurso", 2), ("vestibular", 2),

        // Arabic
        ("درس", 3), ("تعلم", 2), ("دورة", 3), ("دراسة", 2),
        ("شرح", 2), ("امتحان", 2),

        // Bahasa Indonesia/Malay
        ("tutorial", 3), ("kursus", 3), ("belajar", 2), ("pelajaran", 2),
        ("tips", 1), ("ujian", 2), ("materi", 2),
    ],
    "tech_gadgets": [
        // English
        ("phone", 2), ("laptop", 2), ("gadget", 3), ("specs", 3),
        ("benchmark", 3), ("unboxing", 3), ("camera test", 3),
        ("processor", 2), ("RAM", 2), ("review", 2), ("comparison", 2),
        ("battery life", 3), ("display", 1), ("teardown", 3),
        ("best phone", 2), ("versus", 1), ("flagship", 2),

        // Hindi transliterated
        ("mobile", 2), ("phone", 2), ("camera", 1), ("review", 2),

        // Portuguese (Brazil)
        ("celular", 2), ("notebook", 2), ("analise", 2),
        ("comparativo", 2), ("tecnologia", 2), ("bateria", 2),

        // Arabic
        ("هاتف", 2), ("لابتوب", 2), ("مراجعة", 2), ("تقنية", 2),
        ("كاميرا", 1), ("بطارية", 2),

        // Bahasa Indonesia/Malay
        ("handphone", 2), ("laptop", 2), ("review", 2), ("spesifikasi", 3),
        ("teknologi", 2), ("kamera", 1), ("baterai", 2),
    ],
    "parenting": [
        // English
        ("parenting", 3), ("baby", 2), ("toddler", 2), ("kids", 2),
        ("mom", 2), ("dad", 2), ("school", 1), ("milestone", 3),
        ("newborn", 3), ("motherhood", 2), ("fatherhood", 2),
        ("diaper", 2), ("breastfeeding", 3), ("nursery", 2),
        ("playtime", 2), ("bedtime routine", 3), ("first words", 3),

        // Hindi transliterated
        ("baccha", 2), ("maa", 2), ("papa", 2), ("school", 1),
        ("parvarish", 2),

        // Portuguese (Brazil)
        ("maternidade", 3), ("bebe", 2), ("crianca", 2),
        ("paternidade", 3), ("gestante", 3), ("gravidez", 3),
        ("amamentacao", 3), ("filho", 1),

        // Arabic
        ("أطفال", 2), ("طفل", 2), ("أمومة", 3), ("تربية", 3),
        ("رضيع", 3), ("حامل", 2),

        // Bahasa Indonesia/Malay
        ("anak", 2), ("bayi", 2), ("ibu", 2), ("parenting", 3),
        ("balita", 2), ("hamil", 2), ("menyusui", 3),
    ],
    // ... etc for remaining categories (products, beauty, fashion, diy, finance, entertainment, inspiration)
]
```

**Transliterated text handling:**

A significant portion of Hindi-language Instagram content in India is written in Latin script (transliterated Hindi / "Hinglish"). Examples: "paneer butter masala recipe", "dal makhani banane ka tarika", "ghar pe pizza kaise banaye". This content must be classified correctly by Tier 2.

Strategy:
1. **Transliterated keywords are included directly** in the keyword dictionary alongside native-script and English keywords (shown above).
2. **Compound transliterated phrases** (like "paneer butter masala") get weight 3 because multi-word matches are high-signal and low false-positive.
3. **Common Hinglish recipe verbs** should be recognized: "banane ka tarika" (how to make), "ghar pe" (at home), "asaan" (easy).
4. **No transliteration engine is needed** at Tier 2 — just maintain the keyword list. For Tier 1, the on-device LLM handles transliterated text natively.

#### 4.2.2 TF-IDF Scoring for Disambiguation

Raw keyword matching struggles with ambiguous captions. "Love this place, amazing food, must visit!" scores for both Travel and Recipes. TF-IDF (Term Frequency-Inverse Document Frequency) helps by weighing terms that are distinctive to a category more heavily than terms that appear across many categories.

**When to use:** TF-IDF scoring activates only when the top two category scores from keyword matching are within 20% of each other. For clear-cut classifications, the keyword scorer's result is used directly (no extra computation).

**Pre-computed IDF values:**

IDF values are computed offline from a corpus of ~10K labeled social media captions (the training set, see Tier 2.5 below). They ship as a bundled dictionary alongside keyword weights and update via remote config.

```swift
struct TFIDFScorer {
    // Pre-computed: log(totalDocs / docsContainingTerm) per category
    let idfValues: [String: Double]  // "preheat" → 2.8, "love" → 0.3

    func score(tokens: [String], forCategory category: String, keywords: [(String, Int)]) -> Double {
        var score = 0.0
        let tokenFrequency = Dictionary(tokens.map { ($0.lowercased(), 1) }, uniquingKeysWith: +)

        for (token, count) in tokenFrequency {
            guard let keywordMatch = keywords.first(where: { $0.0 == token }) else { continue }
            let tf = log(1.0 + Double(count))  // Sublinear TF
            let idf = idfValues[token] ?? 1.0
            score += tf * idf * Double(keywordMatch.1)  // TF * IDF * keyword weight
        }
        return score
    }
}
```

**Example disambiguation:**

| Caption | Keyword Score (Recipe) | Keyword Score (Travel) | TF-IDF Winner |
|---------|----------------------|----------------------|---------------|
| "Amazing food in Bali, must visit this place" | 5 | 6 | Travel (Bali has high IDF for travel, "food" has low IDF across categories) |
| "Try this Bali-inspired coconut curry, 2 cups coconut milk" | 8 | 4 | Recipe ("cups" has high IDF for recipe, measurement pattern boosts) |

### 4.2.5 Tier 2.5: Bundled Lightweight Text Classifier

Between the full on-device LLM (Tier 1) and raw keyword matching (Tier 2), there's a middle ground: a small, bundled text classification model that runs on any device. This model is better than keywords at handling ambiguity, sarcasm, and short captions, but doesn't require the hardware or framework dependencies of Tier 1.

**Model specification:**
- **Format:** CoreML (iOS) / TFLite (Android)
- **Architecture:** DistilBERT-tiny or MobileBERT fine-tuned for category classification
- **Size budget:** 2–5 MB (acceptable for app bundle)
- **Input:** Concatenated caption + hashtags, tokenized, max 128 tokens
- **Output:** Probability distribution across all categories
- **Latency target:** < 100ms on any device from 2020+

**Tier selection update with Tier 2.5:**

```swift
func selectProcessingTier() -> ProcessingTier {
    #if os(iOS)
    if #available(iOS 26, *), FoundationModels.isAvailable {
        return .tier1_onDeviceAI
    }
    #endif

    // Android: check for Gemini Nano availability
    // if AICore.isAvailable() → .tier1_onDeviceAI

    // Tier 2.5: bundled model available on all devices
    if BundledClassifier.isModelLoaded {
        return .tier2_5_bundledModel
    }

    // Tier 2: pure keyword/regex (ultimate fallback)
    return .tier2_ruleBased
}
```

**When Tier 2.5 beats Tier 2:**

| Scenario | Tier 2 Result | Tier 2.5 Result |
|----------|---------------|-----------------|
| "omg this is absolutely insane 🔥🔥" (recipe video caption) | General (no keywords match) | Recipe (model learned short captions with fire emoji in food context) |
| "Trust me you need this" (product link) | General | Products (model learned promotional patterns) |
| "Never doing this again 😂" (fitness fail) | General | Fitness (with hashtag context) |

**Training data requirements:**

| Requirement | Detail |
|-------------|--------|
| Minimum labeled examples | 10,000 captions (aim for 20K+) |
| Per-category minimum | 500 examples for top categories, 200 for smaller ones |
| Label sources | (1) Hand-labeled by team, (2) high-confidence Tier 1 outputs, (3) user-corrected classifications |
| Languages represented | English 50%, Hindi/Hinglish 20%, Portuguese-BR 10%, Arabic 10%, Bahasa 10% |
| Refresh cadence | Retrain quarterly with new correction data |
| Validation split | 80% train / 10% validation / 10% holdout test |
| Accuracy threshold to ship | Must beat Tier 2 keyword accuracy by > 5 percentage points on holdout set |

**Data pipeline for model updates:**

1. Collect user corrections and high-confidence Tier 1 classifications (anonymized — caption text only, no user identifiers).
2. Quarterly: retrain model on expanded dataset, evaluate on holdout set.
3. If accuracy improves by > 2% on holdout set, ship updated model in next app release.
4. Model file ships in the app bundle — no runtime download needed.

### 4.3 Tier Selection Logic

```swift
func selectProcessingTier() -> ProcessingTier {
    #if os(iOS)
    if #available(iOS 26, *) {
        // Check if Foundation Models available (iPhone 15 Pro+ / A17 Pro)
        if FoundationModels.isAvailable {
            return .tier1_onDeviceAI
        }
    }
    #endif

    // Android: check for Gemini Nano availability via AICore
    // See Section 4.1.2 for full availability check flow

    // Tier 2.5: bundled lightweight model (all devices)
    if BundledClassifier.isModelLoaded {
        return .tier2_5_bundledModel
    }

    return .tier2_ruleBased
}
```

---

## 5. On-Device OCR Pipeline

OCR extracts visible text from saved thumbnails — overlay text, captions baked into images, ingredient lists, prices shown on-screen. This text is a critical input signal for classification, especially when the caption itself is short or generic ("link in bio", "must try").

### 5.1 iOS — Vision Framework

```swift
import Vision

func extractTextFromThumbnail(image: UIImage) async throws -> String {
    // Downscale to 720p for performance — OCR accuracy doesn't
    // meaningfully improve above this resolution for social media thumbnails
    let resized = image.resizedToFit(maxDimension: 720)

    guard let cgImage = resized.cgImage else { return "" }

    let request = VNRecognizeTextRequest()
    request.recognitionLevel = .accurate  // .fast is 2x faster but misses stylized text
    request.usesLanguageCorrection = true

    // Language hints for multilingual content
    // Priority order matters — put expected languages first
    request.recognitionLanguages = ["en-US", "hi-IN", "pt-BR", "ar", "id"]

    // For Devanagari script support (Hindi)
    request.revision = VNRecognizeTextRequestRevision3  // iOS 16+, adds Devanagari

    let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
    try handler.perform([request])

    guard let observations = request.results else { return "" }

    let rawText = observations
        .filter { $0.confidence > 0.5 }  // Drop low-confidence fragments
        .map { $0.topCandidates(1).first?.string ?? "" }
        .joined(separator: " ")

    return cleanOCRText(rawText)
}
```

### 5.2 Android — ML Kit Text Recognition v2

```kotlin
import com.google.mlkit.vision.text.TextRecognition
import com.google.mlkit.vision.text.TextRecognizerOptions
import com.google.mlkit.vision.text.devanagari.DevanagariTextRecognizerOptions

suspend fun extractTextFromThumbnail(bitmap: Bitmap): String {
    // Downscale to 720p
    val resized = bitmap.scaleToFit(maxDimension = 720)
    val inputImage = InputImage.fromBitmap(resized, 0)

    // Latin script recognizer (English, Portuguese, Bahasa, etc.)
    val latinRecognizer = TextRecognition.getClient(
        TextRecognizerOptions.Builder().build()
    )

    // Devanagari script recognizer (Hindi)
    val devanagariRecognizer = TextRecognition.getClient(
        DevanagariTextRecognizerOptions.Builder().build()
    )

    // Run both recognizers — take the result with more recognized text
    val latinResult = latinRecognizer.process(inputImage).await()
    val devanagariResult = devanagariRecognizer.process(inputImage).await()

    val rawText = if (devanagariResult.text.length > latinResult.text.length) {
        // Devanagari detected more content — likely Hindi text
        "${devanagariResult.text} ${latinResult.text}".trim()
    } else {
        latinResult.text
    }

    return cleanOCRText(rawText)
}
```

### 5.3 OCR Text Cleaning

OCR output from social media thumbnails is noisy. Watermarks, handles, and decorative text dilute the useful signal. Clean aggressively before passing to the classifier.

```swift
func cleanOCRText(_ raw: String) -> String {
    var text = raw

    // Remove social media handles and watermarks
    // Matches: @username, IG: handle, TikTok: @handle, YT: channel
    let handlePatterns = [
        #"@[\w.]+"#,                          // @username
        #"(?i)ig:\s*@?[\w.]+"#,               // IG: handle
        #"(?i)tiktok:\s*@?[\w.]+"#,           // TikTok: @handle
        #"(?i)yt:\s*@?[\w.]+"#,               // YT: channel
        #"(?i)follow\s+(me\s+)?@[\w.]+"#,     // Follow me @handle
        #"(?i)follow\s+for\s+more"#,          // Follow for more
        #"(?i)link\s+in\s+bio"#,              // Link in bio (useful signal elsewhere, noise in OCR)
    ]

    for pattern in handlePatterns {
        text = text.replacingOccurrences(of: pattern, with: " ", options: .regularExpression)
    }

    // Remove common overlay text patterns (app-generated)
    let overlayPatterns = [
        #"(?i)made with \w+"#,                // Made with CapCut / InShot
        #"(?i)shot on \w+"#,                  // Shot on iPhone
        #"(?i)\d+\s*views?"#,                 // 12K views
        #"(?i)\d+\s*likes?"#,                 // 3.2K likes
        #"(?i)swipe\s*(left|right|up)"#,      // Swipe left
        #"(?i)tap\s+to\s+\w+"#,              // Tap to shop
        #"(?i)part\s+\d+\s*(of\s+\d+)?"#,    // Part 2 of 5
    ]

    for pattern in overlayPatterns {
        text = text.replacingOccurrences(of: pattern, with: " ", options: .regularExpression)
    }

    // Collapse whitespace
    text = text.replacingOccurrences(of: #"\s+"#, with: " ", options: .regularExpression).trimmingCharacters(in: .whitespaces)

    return text
}
```

### 5.4 When to Run OCR

OCR is not free — it adds 200–500ms to processing time and consumes battery. Use it strategically:

| Tier | OCR Policy |
|------|-----------|
| **Tier 1 (on-device AI)** | Always run OCR. The AI model benefits significantly from on-screen text and the user's device is powerful enough to handle it. |
| **Tier 2.5 (bundled model)** | Run OCR if caption is short (< 50 characters) or if keyword scoring confidence is below 0.6. |
| **Tier 2 (keyword matching)** | Run OCR only if caption is very short (< 20 characters) or empty. On mid-range devices, prioritize speed over completeness. |

---

## 6. UX Design

### 6.1 Categorization Happens Invisibly

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

### 6.2 Category Badge on Cards

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

### 6.3 Manual Override Flow

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
│  │ 📱   │ │ 💪   │ │ 💄   │       │
│  │ Tech │ │Fit   │ │Beauty│       │
│  └──────┘ └──────┘ └──────┘       │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ 👗   │ │ 🏠   │ │ 💰   │       │
│  │Style │ │ DIY  │ │Money │       │
│  └──────┘ └──────┘ └──────┘       │
│  ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ 🎬   │ │ 📚   │ │ 👶   │       │
│  │Watch │ │Learn │ │Kids  │       │
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

### 6.4 Custom Category Creation

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
- **Tier 2.5 (bundled model):** Custom keywords are matched post-classification — if the bundled model classifies as General but custom keywords match strongly, override to the custom category.
- **Tier 2 (rule-based):** Keywords are added to the keyword dictionary with weight 3 (high priority — user explicitly defined them).

### 6.5 Uncertain Classification Hint

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

Tapping "Not quite right?" opens the recategorize sheet (same as 6.3).

**Frequency capping:** If a user ignores the "Not quite right?" hint 3 times on the same save, stop showing it. They likely don't care.

### 6.6 Multi-Category Tagging

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

### 6.7 Learning from User Corrections

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

## 7. Richness Score System

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

## 8. Analytics Events

Every classification and user interaction with categories emits analytics events. These events power the accuracy monitoring dashboard (Section 10), A/B testing of prompt versions, and product decisions about category taxonomy changes.

### 8.1 Event Definitions

**`categorization_completed`** — Emitted when a save is classified (any tier).

| Field | Type | Description |
|-------|------|-------------|
| `tier` | enum | `tier1_ios`, `tier1_android`, `tier2_5`, `tier2` |
| `category` | string | Assigned category (e.g., "recipe") |
| `confidence` | float | 0.0–1.0 |
| `processing_time_ms` | int | Wall-clock time from input to classification result |
| `prompt_version` | string | Prompt version ID (Tier 1 only, null for other tiers) |
| `had_ocr_input` | bool | Whether OCR text was available as input |
| `caption_length` | int | Character count of input caption |
| `secondary_category` | string? | Secondary category if multi-tag (null otherwise) |
| `platform` | string | Source platform: "instagram", "tiktok", "youtube", etc. |

**`category_overridden`** — Emitted when a user manually changes a category.

| Field | Type | Description |
|-------|------|-------------|
| `original_category` | string | Category assigned by classifier |
| `new_category` | string | Category chosen by user |
| `original_confidence` | float | Confidence of the overridden classification |
| `tier` | enum | Which tier produced the original classification |
| `prompt_version` | string? | Prompt version (if Tier 1) |
| `time_to_override_sec` | int | Seconds between save and override (proxy for salience) |
| `override_trigger` | enum | `badge_tap`, `context_menu`, `not_quite_right_hint` |

**`custom_category_created`** — Emitted when a user creates a new custom category.

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Category name (hashed for privacy if needed) |
| `keyword_count` | int | Number of keywords the user provided |
| `has_keywords` | bool | Whether user provided any keywords |
| `total_custom_categories` | int | User's total custom categories after creation |
| `is_paid_user` | bool | Whether user has premium (custom category limits differ) |

**`richness_score_calculated`** — Emitted alongside `categorization_completed`.

| Field | Type | Description |
|-------|------|-------------|
| `score` | float | 0.0–1.0 richness score |
| `score_band` | enum | `high`, `medium`, `low` |
| `field_count` | int | Number of non-nil extracted fields |
| `category` | string | Assigned category |
| `tier` | enum | Processing tier used |

### 8.2 Event Volume Expectations

At 10 saves/day/user with 100K MAU:
- `categorization_completed`: ~1M events/day
- `category_overridden`: ~50K–150K events/day (5–15% override rate expected)
- `custom_category_created`: ~1K events/day (one-time action per user)
- `richness_score_calculated`: ~1M events/day (1:1 with categorization)

Keep payloads lean. All fields are primitives — no nested objects in analytics events.

---

## 9. Edge Cases

| Scenario | Handling |
|----------|----------|
| Caption is entirely emojis ("😍🔥💯") | Low confidence. Classify as General. Rely on thumbnail OCR if available. |
| Caption is in non-English language | Tier 1 AI handles multilingual natively. Tier 2: maintain Hindi/Spanish/Portuguese/Arabic/Bahasa keyword dictionaries for primary markets. |
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
| On-device model not yet downloaded (first launch on new device) | Silently fall back to Tier 2.5 or Tier 2. Do not prompt user to download model — it happens automatically in the background. |
| Transliterated Hindi caption with no hashtags | Tier 2 matches against transliterated keyword dictionary. Tier 1 handles natively. Tier 2.5 trained on Hinglish examples. |
| Tech review vs. product deal — ambiguous | Apply Tech/Gadgets vs. Products heuristic: presence of specs/benchmark keywords → Tech; presence of price/discount/link → Products. Show secondary tag when both signals are strong. |

---

## 10. Accuracy Monitoring & Feedback Loop

Accuracy in production is measured indirectly — we can't ask every user if the category was correct, but we can observe when they correct it. The override rate is the primary proxy for accuracy.

### 10.1 Core Accuracy Metric

**Override Rate = (category overrides) / (total classifications) per time window**

A lower override rate means higher perceived accuracy. Target override rates:

| Tier | Launch Target | 6-Month Target |
|------|--------------|----------------|
| Tier 1 (iOS) | < 12% | < 8% |
| Tier 1 (Android) | < 15% | < 10% |
| Tier 2.5 | < 20% | < 15% |
| Tier 2 | < 25% | < 18% |

**Caveat:** Override rate is a lower bound on error rate — many users won't bother correcting a wrong category if they don't care enough. Actual error rates are likely 1.5–2x the override rate.

### 10.2 Dashboard Metrics

Build an internal analytics dashboard (Mixpanel/Amplitude/custom) tracking these views:

**Override rate breakdowns:**
- By category (which categories are most confused?)
- By tier (is Tier 1 meaningfully better than Tier 2?)
- By platform (are TikTok captions harder than Instagram?)
- By prompt version (is the new prompt better?)
- By language/market (are Hindi captions underperforming?)
- By time period (weekly trends — are we improving?)

**Confusion matrix (weekly aggregate):**
- For each category pair, track how often category A is overridden to category B.
- Example insight: "Recipe → General override rate is 3%, but General → Recipe is 18%." This means the classifier is under-identifying recipes.

**Confidence calibration:**
- Plot: confidence score (x-axis) vs. actual override rate (y-axis).
- A well-calibrated model has low override rate at high confidence and higher override rate at low confidence. If override rate is flat across confidence levels, the confidence score is not informative and should be recalibrated.

### 10.3 Automated Improvement Pipeline

User corrections are the highest-quality signal for improving classification. Feed them back into the system on a regular cadence.

**Monthly cycle:**

```
Week 1: Collect correction data from previous month
         │
         ▼
Week 2: Analyze correction patterns
         - Top confused category pairs
         - New keywords emerging from corrected captions
         - Categories with rising override rates
         │
         ▼
Week 3: Update classifiers
         - Add new keywords to Tier 2 dictionaries (auto-suggested, human-reviewed)
         - Adjust keyword weights based on correction frequency
         - Draft new Tier 1 prompt version addressing top confusion patterns
         - Add correction data to Tier 2.5 training set
         │
         ▼
Week 4: A/B test updated classifiers
         - Deploy new prompt to 10% of Tier 1 users
         - Ship updated keyword weights to 10% of Tier 2 users
         - Monitor override rate for 1 week
         - Promote if override rate improves or stays flat
         - Rollback if override rate increases > 3 percentage points
```

**Automated keyword weight adjustment:**

```swift
// Run monthly on aggregated correction data (server-side or on-device)
func adjustKeywordWeights(corrections: [UserCorrection]) -> [String: [(String, Int)]] {
    var adjustments: [String: [String: Int]] = [:]  // category → keyword → delta

    for correction in corrections {
        let tokens = tokenize(correction.captionText) + correction.hashtags

        for token in tokens {
            // Decrease weight for original (wrong) category
            adjustments[correction.originalCategory, default: [:]][token, default: 0] -= 1

            // Increase weight for corrected (right) category
            adjustments[correction.correctedCategory, default: [:]][token, default: 0] += 1
        }
    }

    // Apply adjustments with bounds: weights stay in range [0, 5]
    // Only apply if adjustment magnitude > 3 (enough signal)
    // Human review required for weight changes > 2 on existing keywords
    // ...

    return updatedDictionary
}
```

### 10.4 A/B Testing Framework for Classifier Versions

Beyond prompt versioning (Section 4.1.3), apply the same A/B framework to:
- **Keyword dictionary versions** (Tier 2)
- **Bundled model versions** (Tier 2.5)
- **Confidence thresholds** (e.g., test 0.6 vs. 0.7 as the "uncertain" cutoff)
- **OCR policies** (e.g., test always-on OCR vs. conditional OCR on Tier 2)

Each experiment runs for a minimum of 1 week or 500 classifications per variant — whichever comes later. Use override rate as the primary success metric and confidence distribution as a guardrail metric.

---

## 11. Accessibility

- **VoiceOver / TalkBack:** Category badge announces as "Category: Recipe" (not just the emoji). Uncertain hint announces "Category may be incorrect. Double tap to change."
- **Dynamic Type:** Category badge text scales with system font. Badge grows vertically, never truncates the label.
- **Color independence:** Categories are differentiated by emoji + text label, not color alone. Color-blind users can distinguish all categories.
- **Recategorize sheet:** All category chips are focusable and navigable via keyboard/switch control. Currently-selected category announces "Recipe, currently selected."
- **Minimum tap targets:** Category badge >= 44x44pt, each chip in recategorize sheet >= 44x44pt.
- **Reduce Motion:** Shimmer animation on processing state replaced with static "Organizing..." text.

---

## 12. Animation Specifications

| Transition | Duration | Easing | Description |
|-----------|----------|--------|-------------|
| Skeleton → final card | 300ms | ease-out | Crossfade shimmer to thumbnail + badge |
| Category badge change | 200ms | spring (0.6 damping) | Old badge scales down + fades, new badge scales up + fades in |
| Recategorize sheet open | 250ms | ease-out | Standard bottom sheet spring |
| "Not quite right?" appear | 400ms | ease-in | Delayed fade-in (appears 0.5s after card renders, so it doesn't distract from content) |
| Multi-category tag appear | 200ms | ease-out | Slides in from left below primary category |
| Toast ("Moved to Travel") | 250ms in, 200ms out | ease-out / ease-in | Slides up from bottom, auto-dismisses after 4 sec |

---

## 13. Development Plan

### 13.1 Tasks & Estimates

| # | Task | Platform | Effort | Dependencies |
|---|------|----------|--------|-------------|
| 1 | Define Classification data model + JSON schema | Shared | 3h | — |
| 2 | Build category taxonomy + keyword dictionaries (multilingual) | Shared | 8h | — |
| 3 | Implement Tier 2 rule-based classifier + TF-IDF scorer | Shared/KMP | 10h | Task 2 |
| 4 | Implement regex extractors (ingredients, prices, locations) | Shared/KMP | 6h | — |
| 5 | Implement Tier 1 Foundation Models integration (with @Generable) | iOS | 8h | Tasks 1, 2 |
| 6 | Implement Tier 1 Gemini Nano integration (with AICore checks) | Android | 8h | Tasks 1, 2 |
| 7 | Build tier selection logic (Tier 1 → 2.5 → 2 fallback chain) | Both | 3h | Tasks 3, 5, 6 |
| 8 | Implement richness score calculation | Shared/KMP | 3h | Task 1 |
| 9 | Build OCR pipeline (Vision/ML Kit) with text cleaning | Both | 6h | — |
| 10 | Build manual override UI (recategorize sheet) | Both | 4h | — |
| 11 | Build custom category creation flow | Both | 4h | — |
| 12 | Implement user correction tracking | Both | 3h | Task 10 |
| 13 | Implement confidence-based UI hints | Both | 2h | Task 8 |
| 14 | Connect Share Extension → Categorization pipeline | Both | 4h | F1, Tasks 3–7 |
| 15 | Implement analytics events (Section 8) | Both | 4h | Tasks 3–7 |
| 16 | Build prompt versioning + A/B test infrastructure | Both | 6h | Task 5, 6 |
| 17 | Integrate Tier 2.5 bundled model (CoreML/TFLite) | Both | 6h | Task 1, 2 |
| 18 | Keyword dictionary tuning + accuracy testing (multilingual) | QA | 10h | Task 3 |
| 19 | AI prompt tuning + accuracy testing | QA | 8h | Tasks 5, 6 |
| 20 | End-to-end testing across tiers | QA | 8h | All above |

**Total: ~116 hours (~4 weeks with one developer per platform)**

### 13.2 Accuracy Benchmarking

Create a test dataset of 300 Instagram/TikTok captions (manually labeled):
- 50 recipes (including 15 Hindi/Hinglish, 10 Portuguese), 40 travel, 30 products, 25 tech/gadgets, 25 fitness, 20 beauty, 20 fashion, 20 DIY, 15 education, 15 parenting, 10 finance, 10 entertainment, 10 inspiration, 10 general
- Include edge cases: emoji-only, bilingual, multi-category, vague captions, transliterated Hindi, Arabic captions
- Include 30 captions specifically designed to test disambiguation (captions that could plausibly belong to 2+ categories)

Measure:
- **Tier 1 accuracy target:** > 90% correct category
- **Tier 2.5 accuracy target:** > 85% correct category
- **Tier 2 accuracy target:** > 80% correct category
- **False positive rate:** < 10% (content classified into wrong specific category vs. General)
- **Per-language accuracy:** No language should be more than 10 percentage points below the overall accuracy

### 13.3 Definition of Done

- [ ] Tier 1 on-device AI classifies content with > 85% accuracy on test dataset
- [ ] Tier 2.5 bundled model classifies content with > 80% accuracy on test dataset
- [ ] Tier 2 rule-based classifier achieves > 75% accuracy on test dataset
- [ ] Tier selection automatically picks best available tier for device
- [ ] Tier 1 gracefully handles model-not-downloaded, model-busy, and guardrail errors
- [ ] Android Tier 1 handles all AICore availability states (not downloaded, not opted in, not supported)
- [ ] Richness score correctly differentiates High/Medium/Low saves
- [ ] Manual category override works with one tap
- [ ] Custom category creation works with emoji picker
- [ ] OCR extracts on-screen text from thumbnails on both platforms (Latin + Devanagari)
- [ ] OCR text cleaning removes watermarks and social media handles
- [ ] User corrections logged for future dictionary improvement
- [ ] Processing completes within 2 seconds on Tier 1, 500ms on Tier 2.5, 200ms on Tier 2
- [ ] Bilingual captions (English + Hindi) handled correctly
- [ ] Transliterated Hindi captions classified correctly by Tier 2
- [ ] All 4 analytics events emit correctly with complete payloads
- [ ] Prompt versioning infrastructure deployed and tested with 2 prompt versions
- [ ] New categories (Education, Parenting, Tech/Gadgets) included in all tiers
