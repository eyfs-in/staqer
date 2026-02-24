# Kotlin Multiplatform Shared Module Architecture Plan
## Staqer App

**Version:** 1.0  
**Last Updated:** 2026-02-24  
**Author:** Lead Developer Architecture Guide  
**Status:** Implementation Ready

---

## Table of Contents

1. [Project Setup](#1-project-setup)
2. [What to Share vs What Stays Native](#2-what-to-share-vs-what-stays-native)
3. [Module Structure](#3-module-structure)
4. [Data Models](#4-data-models)
5. [URL Normalization Module](#5-url-normalization-module)
6. [Classification Engine](#6-classification-engine)
7. [Network Module](#7-network-module)
8. [Platform-Specific Expect/Actual](#8-platform-specific-expectactual)
9. [iOS Integration](#9-ios-integration)
10. [Android Integration](#10-android-integration)
11. [Testing](#11-testing)
12. [Versioning & Publishing](#12-versioning--publishing)
13. [Migration Path](#13-migration-path)

---

## 1. Project Setup

### 1.1 Gradle Project Structure

```
staqer/
├── shared/                                # KMP shared module
│   ├── build.gradle.kts                  # Multiplatform configuration
│   ├── src/
│   │   ├── commonMain/                   # Shared code
│   │   │   ├── kotlin/
│   │   │   │   └── com.staqer.shared/
│   │   │   │       ├── core/             # Core utilities
│   │   │   │       ├── models/           # Data models
│   │   │   │       ├── network/          # API client
│   │   │   │       ├── classification/   # Classification engine
│   │   │   │       ├── url/              # URL normalization
│   │   │   │       ├── analytics/        # Analytics definitions
│   │   │   │       └── subscription/     # Quota/subscription logic
│   │   │   └── resources/                # Multilingual dictionaries
│   │   ├── commonTest/                   # Shared tests
│   │   ├── iosMain/                      # iOS-specific implementations
│   │   ├── iosTest/
│   │   ├── androidMain/                  # Android-specific implementations
│   │   └── androidTest/
│   └── staqer-shared.podspec             # CocoaPods spec for iOS
├── iosApp/                                # Native iOS app
├── androidApp/                            # Native Android app
├── gradle/
│   └── libs.versions.toml                # Version catalog
├── build.gradle.kts                      # Root build file
└── settings.gradle.kts
```

### 1.2 build.gradle.kts (shared module)

```kotlin
plugins {
    alias(libs.plugins.kotlinMultiplatform)
    alias(libs.plugins.androidLibrary)
    alias(libs.plugins.kotlinSerialization)
    alias(libs.plugins.sqldelight)
}

kotlin {
    androidTarget {
        compilations.all {
            kotlinOptions {
                jvmTarget = "17"
            }
        }
    }
    
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach { iosTarget ->
        iosTarget.binaries.framework {
            baseName = "StaqerShared"
            isStatic = true
            
            // Export main dependencies
            export(libs.kotlinx.datetime)
            export(libs.kotlinx.coroutines.core)
        }
    }

    sourceSets {
        commonMain.dependencies {
            // Coroutines
            implementation(libs.kotlinx.coroutines.core)
            
            // Serialization
            implementation(libs.kotlinx.serialization.json)
            
            // Date/Time
            implementation(libs.kotlinx.datetime)
            api(libs.kotlinx.datetime)
            
            // Networking
            implementation(libs.ktor.client.core)
            implementation(libs.ktor.client.contentNegotiation)
            implementation(libs.ktor.serialization.json)
            implementation(libs.ktor.client.logging)
            
            // Database
            implementation(libs.sqldelight.runtime)
            implementation(libs.sqldelight.coroutinesExtensions)
        }
        
        androidMain.dependencies {
            implementation(libs.ktor.client.okhttp)
            implementation(libs.sqldelight.androidDriver)
        }
        
        iosMain.dependencies {
            implementation(libs.ktor.client.darwin)
            implementation(libs.sqldelight.nativeDriver)
        }
        
        commonTest.dependencies {
            implementation(kotlin("test"))
            implementation(libs.kotlinx.coroutines.test)
            implementation(libs.ktor.client.mock)
        }
    }
}

android {
    namespace = "com.staqer.shared"
    compileSdk = 34
    
    defaultConfig {
        minSdk = 26
    }
    
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
}

sqldelight {
    databases {
        create("StaqerDatabase") {
            packageName.set("com.staqer.shared.db")
            schemaOutputDirectory.set(file("build/dbs"))
            verifyMigrations.set(true)
        }
    }
}
```

### 1.3 gradle/libs.versions.toml

```toml
[versions]
kotlin = "1.9.22"
kotlinxCoroutines = "1.7.3"
kotlinxSerialization = "1.6.2"
kotlinxDateTime = "0.5.0"
ktor = "2.3.7"
sqldelight = "2.0.1"

[libraries]
kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "kotlinxCoroutines" }
kotlinx-coroutines-test = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-test", version.ref = "kotlinxCoroutines" }
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinxSerialization" }
kotlinx-datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version.ref = "kotlinxDateTime" }

ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
ktor-client-contentNegotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
ktor-client-logging = { module = "io.ktor:ktor-client-logging", version.ref = "ktor" }
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor" }
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }
ktor-client-mock = { module = "io.ktor:ktor-client-mock", version.ref = "ktor" }
ktor-serialization-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }

sqldelight-runtime = { module = "app.cash.sqldelight:runtime", version.ref = "sqldelight" }
sqldelight-coroutinesExtensions = { module = "app.cash.sqldelight:coroutines-extensions", version.ref = "sqldelight" }
sqldelight-androidDriver = { module = "app.cash.sqldelight:android-driver", version.ref = "sqldelight" }
sqldelight-nativeDriver = { module = "app.cash.sqldelight:native-driver", version.ref = "sqldelight" }

[plugins]
kotlinMultiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
kotlinSerialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
androidLibrary = { id = "com.android.library", version = "8.2.2" }
sqldelight = { id = "app.cash.sqldelight", version.ref = "sqldelight" }
```

---

## 2. What to Share vs What Stays Native

### 2.1 Shared in KMP (30-40% of codebase)

| Layer | Components | Rationale |
|-------|-----------|-----------|
| **Data Models** | SavedContent, Category, ExtractionResult, UserProfile, Subscription, Collection, PendingSave | Identical structure across platforms. Single source of truth. |
| **Network** | Ktor HTTP client, API definitions, JSON serialization, retry logic | Platform-agnostic networking. Ktor compiles to OkHttp (Android) and Darwin (iOS). |
| **URL Normalization** | Tracking param removal, short URL patterns, canonical form logic | Pure string manipulation. No platform APIs needed. |
| **Classification Rules** | Keyword dictionaries (5 languages), TF-IDF scoring, regex patterns, scoring logic | Rule-based classification is deterministic and platform-independent. |
| **Analytics Definitions** | Event schemas, property validation, event names | Consistent analytics across platforms. Native SDKs fire the events. |
| **Quota Logic** | Quota checking, reset dates, subscription state, Pro feature gates | Business logic shared. Platform billing systems provide state. |
| **Utility Functions** | Date formatting, currency conversion, unit conversion (cooking), text cleaning | Pure functions. No platform dependencies. |

### 2.2 Stays Native (60-70% of codebase)

| Layer | iOS (Swift) | Android (Kotlin) | Rationale |
|-------|-------------|-----------------|-----------|
| **UI** | SwiftUI | Jetpack Compose | Native UI frameworks. KMP UI libs not mature enough for production-quality animations and platform feel. |
| **On-Device AI** | Foundation Models (iOS 26) | Gemini Nano / ML Kit GenAI Prompt API | First-party native frameworks. No KMP bridge available. Direct access = zero overhead. |
| **Speech-to-Text** | SpeechAnalyzer, SFSpeechRecognizer | ML Kit Speech | Native frameworks. High performance requirements. |
| **OCR** | Vision framework | ML Kit Text Recognition | Native frameworks. |
| **Share Extension** | NSExtensionContext | IntentFilter + ShareCompat | Platform-specific APIs by definition. Share sheets are OS-level. |
| **Database Access** | Core Data (with SQLite backend) | Room (with SQLite backend) | Platform-specific ORMs. SQL schema and queries can be shared via SQLDelight for common queries. |
| **Secure Storage** | Keychain | EncryptedSharedPreferences | Platform-specific secure storage. |
| **Background Tasks** | BGAppRefreshTask, BGProcessingTask | WorkManager | Platform-specific background execution models. |
| **Notifications** | UNUserNotificationCenter | NotificationCompat / FCM | Platform-specific notification systems. |
| **Platform Integrations** | WidgetKit, Shortcuts, Apple Maps, Reminders | Glance Widgets, Google Maps, Google Keep | Deep platform integrations. |

### 2.3 Decision Matrix: When to Share Code

**Share if:**
- Pure business logic (no UI, no platform APIs)
- Data structures used by both platforms
- Network communication (REST APIs, WebSockets)
- Text processing, regex, string manipulation
- Simple calculations and algorithms
- Configuration and constants

**Keep Native if:**
- Involves UI rendering or animations
- Requires direct platform API access (Camera, ML, Bluetooth, etc.)
- Performance-critical (on-device AI, image processing)
- Deeply integrated with OS features (share sheets, widgets, notifications)
- Simpler to implement natively than to bridge

---

## 3. Module Structure

### 3.1 Package Organization

```
com.staqer.shared/
├── core/                                 # Core utilities and base classes
│   ├── Result.kt                        # Result wrapper for operations
│   ├── Logger.kt                        # Expect/actual logging
│   ├── Platform.kt                      # Platform detection
│   └── Constants.kt                     # Shared constants
│
├── models/                               # Data models
│   ├── SavedContent.kt                  # Core saved item model
│   ├── Category.kt                      # Category definitions
│   ├── ExtractionResult.kt              # AI extraction results
│   ├── CardType.kt                      # Structured card types
│   ├── UserProfile.kt                   # User account data
│   ├── Subscription.kt                  # Pro subscription state
│   ├── Collection.kt                    # Collections/boards
│   ├── PendingSave.kt                   # Share extension queue
│   ├── ClassificationResult.kt          # Classification outputs
│   └── Platform.kt                      # Platform enum (Instagram, TikTok, etc.)
│
├── network/                              # Network layer
│   ├── StaqerApiClient.kt               # Main API client
│   ├── requests/                        # Request DTOs
│   │   ├── DeepExtractRequest.kt
│   │   ├── SaveContentRequest.kt
│   │   └── SyncRequest.kt
│   ├── responses/                       # Response DTOs
│   │   ├── DeepExtractResponse.kt
│   │   ├── MetadataResponse.kt
│   │   └── UserProfileResponse.kt
│   ├── NetworkResult.kt                 # Network-specific Result wrapper
│   └── HttpClientFactory.kt             # Expect/actual Ktor client creation
│
├── classification/                       # Classification engine (Tier 2 rule-based)
│   ├── ClassificationEngine.kt          # Main classifier interface
│   ├── KeywordClassifier.kt             # Keyword/hashtag matching
│   ├── TFIDFScorer.kt                   # TF-IDF disambiguation
│   ├── RegexExtractor.kt                # Pattern extraction (ingredients, prices, etc.)
│   ├── DietaryTagDetector.kt            # Recipe dietary tags
│   ├── RichnessScoreCalculator.kt       # Content richness scoring
│   ├── dictionaries/                     # Multilingual keyword data
│   │   ├── KeywordDictionary.kt
│   │   ├── en/                          # English keywords
│   │   │   ├── RecipeKeywords.kt
│   │   │   ├── TravelKeywords.kt
│   │   │   ├── ProductKeywords.kt
│   │   │   └── ...
│   │   ├── hi/                          # Hindi keywords (Devanagari + transliterated)
│   │   ├── pt/                          # Portuguese (Brazil)
│   │   ├── ar/                          # Arabic
│   │   └── id/                          # Bahasa Indonesia
│   └── patterns/                         # Regex patterns
│       ├── IngredientPatterns.kt
│       ├── PricePatterns.kt
│       ├── LocationPatterns.kt
│       └── TimePatterns.kt
│
├── url/                                  # URL normalization & platform detection
│   ├── UrlNormalizer.kt                 # Main normalization logic
│   ├── TrackingParamRemover.kt          # Strip utm_*, igshid, etc.
│   ├── ShortUrlResolver.kt              # Expand bit.ly, vm.tiktok.com, etc. (uses network)
│   ├── PlatformDetector.kt              # Identify Instagram, TikTok, YouTube from URL
│   └── UrlValidator.kt                  # Validation & SSRF prevention
│
├── analytics/                            # Analytics event definitions
│   ├── AnalyticsEvent.kt                # Base event interface
│   ├── events/                           # Event definitions
│   │   ├── ShareExtensionEvents.kt
│   │   ├── CategorizationEvents.kt
│   │   ├── LibraryEvents.kt
│   │   ├── CardEvents.kt
│   │   └── DeepExtractEvents.kt
│   └── EventLogger.kt                   # Expect/actual event dispatch
│
├── subscription/                         # Quota & subscription state
│   ├── QuotaManager.kt                  # Quota checks, resets
│   ├── SubscriptionState.kt             # Pro vs Free state
│   └── FeatureGate.kt                   # Feature gating logic
│
└── utils/                                # Utility functions
    ├── DateTimeUtils.kt                 # kotlinx-datetime helpers
    ├── TextCleaning.kt                  # OCR text cleaning, normalization
    ├── UnitConversion.kt                # Metric <-> Imperial (cooking)
    ├── CurrencyConverter.kt             # Basic currency conversion
    └── StringExtensions.kt              # Shared string utilities
```

---

## 4. Data Models

All shared data models use `@Serializable` from kotlinx.serialization for JSON encoding/decoding and type-safe serialization across platforms.

### 4.1 SavedContent.kt

```kotlin
package com.staqer.shared.models

import kotlinx.datetime.Instant
import kotlinx.serialization.Serializable

@Serializable
data class SavedContent(
    val id: String,                          // UUID
    val url: String,                         // Canonical normalized URL
    val rawUrl: String,                      // Original URL from share sheet
    val platform: Platform,
    val captionText: String? = null,
    val transcriptText: String? = null,      // From deep extract
    val thumbnailUrl: String? = null,
    val thumbnailLocalPath: String? = null,  // Platform-specific path
    val thumbnailAspectRatio: Double? = null, // Width / Height (for staggered grid)
    val category: String,                    // "recipe", "travel", etc.
    val categoryConfidence: Double,          // 0.0-1.0
    val richnessScore: Double,               // 0.0-1.0 (how complete the extraction is)
    val extractedData: ExtractedData? = null, // Structured extraction
    val userNote: String? = null,
    val userCategoryOverride: String? = null, // User manual override
    val collectionIds: List<String> = emptyList(),
    val creatorHandle: String? = null,
    val createdAt: Instant,
    val processedAt: Instant? = null,
    val status: SaveStatus,
    val mayExpire: Boolean = false,          // True for ephemeral content (Stories)
    val secondaryCategory: String? = null,   // Multi-category tagging
    val customCategoryId: String? = null     // User-created category
)

@Serializable
enum class Platform {
    INSTAGRAM,
    TIKTOK,
    YOUTUBE,
    TWITTER,
    PINTEREST,
    THREADS,
    OTHER
}

@Serializable
enum class SaveStatus {
    PENDING,           // Awaiting processing
    PROCESSING,        // Classification in progress
    COMPLETE,          // Fully processed
    FAILED,            // Processing failed
    QUEUED_FOR_EXTRACT // Pending deep extract
}
```

### 4.2 ExtractedData.kt

```kotlin
package com.staqer.shared.models

import kotlinx.serialization.Serializable

@Serializable
data class ExtractedData(
    val recipe: RecipeData? = null,
    val travel: TravelData? = null,
    val product: ProductData? = null,
    val fitness: FitnessData? = null,
    val beauty: BeautyData? = null,
    val fashion: FashionData? = null,
    val education: EducationData? = null,
    val techGadgets: TechGadgetsData? = null,
    val parenting: ParentingData? = null
)

@Serializable
data class RecipeData(
    val dishName: String,
    val ingredients: List<Ingredient>,
    val steps: List<RecipeStep>,
    val prepTime: String? = null,           // "15 minutes"
    val servings: Int? = null,
    val originalServings: Int? = null,      // For scaling
    val cuisine: String? = null,            // "Italian"
    val difficulty: String? = null,         // "Easy"
    val dietaryTags: List<DietaryTag> = emptyList()
)

@Serializable
data class Ingredient(
    val item: String,                        // "spaghetti"
    val quantity: String? = null,            // "200g"
    val quantityMetric: QuantityValue? = null,
    val quantityImperial: QuantityValue? = null,
    val prep: String? = null,                // "minced"
    val isChecked: Boolean = false           // UI state (persisted)
)

@Serializable
data class QuantityValue(
    val value: Double,
    val unit: CookingUnit
)

@Serializable
enum class CookingUnit {
    GRAMS, KILOGRAMS, MILLILITERS, LITERS, CELSIUS,
    OUNCES, POUNDS, CUPS, TABLESPOONS, TEASPOONS, FAHRENHEIT,
    CLOVES, PINCH, TO_TASTE, WHOLE
}

@Serializable
data class RecipeStep(
    val text: String,
    val timeSeconds: Int? = null             // Extracted timer duration
)

@Serializable
data class DietaryTag(
    val tag: String,                         // "vegetarian", "gluten_free", etc.
    val confidence: Confidence,
    val ambiguousIngredient: String? = null  // If low confidence, which ingredient?
)

@Serializable
enum class Confidence {
    HIGH, MEDIUM, LOW
}

@Serializable
data class TravelData(
    val locationName: String,
    val coordinates: Coordinates? = null,
    val country: String? = null,
    val region: String? = null,              // "Bali"
    val bestTime: String? = null,            // "May-September"
    val budgetRange: String? = null,         // "$30-50/day"
    val budgetAmount: BudgetAmount? = null,
    val tips: List<String> = emptyList(),
    val tripId: String? = null               // Assigned trip collection
)

@Serializable
data class Coordinates(
    val lat: Double,
    val lng: Double
)

@Serializable
data class BudgetAmount(
    val min: Double,
    val max: Double,
    val currency: String,                    // "USD"
    val period: String                       // "day"
)

@Serializable
data class ProductData(
    val productName: String,
    val brand: String? = null,
    val price: Double? = null,
    val currency: String? = null,            // "USD"
    val store: String? = null,               // "Amazon"
    val purchaseUrl: String? = null,
    val affiliateUrl: String? = null,        // Populated at link-open time
    val productCategory: String? = null,     // "Standing Desks"
    val rating: Double? = null,              // 4.7
    val keySpecs: List<String> = emptyList(),
    val priceTrackingEnabled: Boolean = false
)

@Serializable
data class FitnessData(
    val workoutName: String,
    val duration: String? = null,            // "20 minutes"
    val calories: String? = null,            // "~300"
    val targetArea: String? = null,          // "Full Body"
    val exercises: List<Exercise> = emptyList(),
    val equipment: String? = null            // "None"
)

@Serializable
data class Exercise(
    val name: String,                        // "Push-ups"
    val duration: String? = null,            // "30 sec"
    val reps: Int? = null                    // 12
)

@Serializable
data class BeautyData(
    val routineName: String,
    val skinType: String? = null,            // "Combination"
    val routineSteps: List<RoutineStep> = emptyList(),
    val productsMentioned: List<String> = emptyList()
)

@Serializable
data class RoutineStep(
    val stepNumber: Int,
    val type: String,                        // "Cleanser", "Toner", etc.
    val productName: String
)

@Serializable
data class FashionData(
    val outfitName: String,
    val items: List<String> = emptyList(),   // ["Black blazer", "White tee", "Jeans"]
    val brands: List<String> = emptyList(),
    val sizes: List<String> = emptyList(),
    val occasion: String? = null,            // "Office", "Casual"
    val season: String? = null               // "Winter"
)

@Serializable
data class EducationData(
    val topic: String,                       // "Backend Development"
    val subtopic: String? = null,            // "REST API Design"
    val keyTakeaways: List<String> = emptyList(),
    val toolsResources: List<ToolResource> = emptyList(),
    val difficulty: String? = null,          // "intermediate"
    val estimatedLearningTime: String? = null, // "45 minutes"
    val isLearned: Boolean = false           // UI state (persisted)
)

@Serializable
data class ToolResource(
    val name: String,                        // "Node.js"
    val url: String? = null                  // "https://nodejs.org"
)

@Serializable
data class TechGadgetsData(
    val deviceName: String,
    val brand: String? = null,
    val specs: List<String> = emptyList(),   // ["6.7\" display", "8GB RAM"]
    val rating: Double? = null,
    val prosAndCons: ProsAndCons? = null,
    val priceRange: String? = null           // "$700-$900"
)

@Serializable
data class ProsAndCons(
    val pros: List<String> = emptyList(),
    val cons: List<String> = emptyList()
)

@Serializable
data class ParentingData(
    val topicName: String,
    val ageGroup: String? = null,            // "Toddler", "Newborn"
    val activityType: String? = null,        // "Craft", "Learning Activity"
    val materials: List<String> = emptyList(),
    val developmentalStage: String? = null   // "Milestone: First Words"
)
```

### 4.3 Category.kt

```kotlin
package com.staqer.shared.models

import kotlinx.serialization.Serializable

@Serializable
data class Category(
    val id: String,
    val name: String,                        // "Recipes"
    val emoji: String,                       // "🍳"
    val type: CategoryType,
    val keywords: List<String> = emptyList(), // User-provided hints for custom categories
    val saveCount: Int = 0                   // Denormalized count for UI
)

@Serializable
enum class CategoryType {
    DEFAULT,           // Built-in category
    CUSTOM             // User-created
}

object DefaultCategories {
    val ALL = Category("all", "All", "📚", CategoryType.DEFAULT)
    val RECIPES = Category("recipe", "Recipes", "🍳", CategoryType.DEFAULT)
    val TRAVEL = Category("travel", "Travel", "✈️", CategoryType.DEFAULT)
    val PRODUCTS = Category("product", "Products", "🛍️", CategoryType.DEFAULT)
    val TECH_GADGETS = Category("tech_gadgets", "Tech & Gadgets", "📱", CategoryType.DEFAULT)
    val FITNESS = Category("fitness", "Fitness", "💪", CategoryType.DEFAULT)
    val BEAUTY = Category("beauty", "Beauty", "💄", CategoryType.DEFAULT)
    val FASHION = Category("fashion", "Fashion", "👗", CategoryType.DEFAULT)
    val DIY = Category("diy", "DIY & Home", "🏠", CategoryType.DEFAULT)
    val FINANCE = Category("finance", "Finance", "💰", CategoryType.DEFAULT)
    val ENTERTAINMENT = Category("entertainment", "Entertainment", "🎬", CategoryType.DEFAULT)
    val EDUCATION = Category("education", "Education", "📚", CategoryType.DEFAULT)
    val PARENTING = Category("parenting", "Parenting & Kids", "👶", CategoryType.DEFAULT)
    val INSPIRATION = Category("inspiration", "Inspiration", "✨", CategoryType.DEFAULT)
    val GENERAL = Category("general", "General", "📌", CategoryType.DEFAULT)
    
    fun all() = listOf(
        RECIPES, TRAVEL, PRODUCTS, TECH_GADGETS, FITNESS, BEAUTY, FASHION,
        DIY, FINANCE, ENTERTAINMENT, EDUCATION, PARENTING, INSPIRATION, GENERAL
    )
}
```

### 4.4 UserProfile.kt & Subscription.kt

```kotlin
package com.staqer.shared.models

import kotlinx.datetime.Instant
import kotlinx.serialization.Serializable

@Serializable
data class UserProfile(
    val userId: String,
    val email: String? = null,
    val displayName: String? = null,
    val subscriptionState: SubscriptionState,
    val quota: QuotaState,
    val preferences: UserPreferences,
    val createdAt: Instant,
    val lastSyncedAt: Instant? = null
)

@Serializable
data class SubscriptionState(
    val isPro: Boolean,
    val tier: SubscriptionTier,
    val expiresAt: Instant? = null,          // Null if free or lifetime
    val isTrialing: Boolean = false,
    val trialEndsAt: Instant? = null,
    val platform: BillingPlatform? = null    // iOS App Store, Google Play
)

@Serializable
enum class SubscriptionTier {
    FREE,
    PRO_MONTHLY,
    PRO_YEARLY
}

@Serializable
enum class BillingPlatform {
    APP_STORE,
    PLAY_STORE
}

@Serializable
data class QuotaState(
    val monthYear: String,                   // "2026-02"
    val deepExtractsUsed: Int,
    val deepExtractsLimit: Int,              // 10 for free, Int.MAX_VALUE for pro
    val resetDate: Instant
)

@Serializable
data class UserPreferences(
    val autoExtractEnabled: Boolean = false,
    val autoExtractOnWifiOnly: Boolean = true,
    val defaultMeasurementSystem: MeasurementSystem = MeasurementSystem.METRIC,
    val enableNotifications: Boolean = true
)

@Serializable
enum class MeasurementSystem {
    METRIC,
    IMPERIAL
}
```

### 4.5 PendingSave.kt

```kotlin
package com.staqer.shared.models

import kotlinx.datetime.Instant
import kotlinx.serialization.Serializable

/**
 * Represents a save captured by the share extension that hasn't been fully processed yet.
 * This model bridges the share extension and the main app.
 */
@Serializable
data class PendingSave(
    val id: String,                          // UUID
    val url: String,                         // Canonical normalized URL
    val rawUrl: String,                      // Original URL from share
    val platform: Platform,
    val captionText: String? = null,
    val thumbnailUrl: String? = null,
    val creatorHandle: String? = null,
    val userNote: String? = null,
    val userCategory: String? = null,        // Manual category from share sheet
    val status: PendingSaveStatus,
    val createdAt: Instant,
    val offlineSave: Boolean = false,
    val mayExpire: Boolean = false,
    val sourceType: SourceType = SourceType.URL
)

@Serializable
enum class PendingSaveStatus {
    PENDING,           // Not yet processed
    PROCESSING,        // Classification running
    COMPLETE,          // Converted to SavedContent
    FAILED             // Processing failed
}

@Serializable
enum class SourceType {
    URL,               // Shared URL
    IMAGE,             // Shared image (screenshot with OCR)
    TEXT               // Pasted text
}
```

### 4.6 Collection.kt

```kotlin
package com.staqer.shared.models

import kotlinx.datetime.Instant
import kotlinx.serialization.Serializable

@Serializable
data class Collection(
    val id: String,
    val name: String,
    val emoji: String? = null,
    val type: CollectionType,
    val saveIds: List<String> = emptyList(),
    val coverImageUrl: String? = null,       // Auto-set from first save
    val createdAt: Instant,
    val updatedAt: Instant,
    val isShared: Boolean = false,
    val shareLink: String? = null,           // Read-only share link
    // Trip-specific fields (only for TRIP type)
    val tripDateRange: DateRange? = null
)

@Serializable
enum class CollectionType {
    COLLECTION,        // Standard collection
    TRIP               // Travel-specific collection with date range
}

@Serializable
data class DateRange(
    val startDate: Instant,
    val endDate: Instant
)
```

---

## 5. URL Normalization Module

The URL normalization module is pure Kotlin with no platform dependencies. It handles tracking parameter removal, short URL resolution, and canonical form standardization.

### 5.1 UrlNormalizer.kt

```kotlin
package com.staqer.shared.url

class UrlNormalizer(
    private val trackingParamRemover: TrackingParamRemover = TrackingParamRemover(),
    private val shortUrlResolver: ShortUrlResolver = ShortUrlResolver()
) {
    /**
     * Full normalization pipeline:
     * 1. Strip tracking parameters
     * 2. Resolve short URLs (network call)
     * 3. Canonicalize (lowercase, remove trailing slash, etc.)
     */
    suspend fun normalize(rawUrl: String): UrlNormalizationResult {
        return try {
            // Step 1: Strip tracking params
            val withoutTracking = trackingParamRemover.removeTrackingParams(rawUrl)
            
            // Step 2: Resolve short URLs (if applicable)
            val resolved = shortUrlResolver.resolve(withoutTracking)
            
            // Step 3: Canonicalize
            val canonical = canonicalize(resolved)
            
            UrlNormalizationResult.Success(
                canonicalUrl = canonical,
                rawUrl = rawUrl,
                hadTrackingParams = withoutTracking != rawUrl,
                wasShortUrl = resolved != withoutTracking
            )
        } catch (e: Exception) {
            UrlNormalizationResult.Failure(rawUrl, e)
        }
    }
    
    /**
     * Synchronous normalization without short URL resolution.
     * Use for offline or when network resolution isn't needed.
     */
    fun normalizeLocal(rawUrl: String): String {
        val withoutTracking = trackingParamRemover.removeTrackingParams(rawUrl)
        return canonicalize(withoutTracking)
    }
    
    private fun canonicalize(url: String): String {
        try {
            val parsed = parseUrl(url)
            
            return buildString {
                // Scheme: lowercase
                append(parsed.scheme.lowercase())
                append("://")
                
                // Host: lowercase, remove www
                val host = parsed.host.lowercase().removePrefix("www.")
                append(host)
                
                // Port: omit default ports (80, 443)
                if (parsed.port != null && parsed.port != 80 && parsed.port != 443) {
                    append(":${parsed.port}")
                }
                
                // Path: remove trailing slash (unless root)
                val path = parsed.path.removeSuffix("/").ifEmpty { "/" }
                append(path)
                
                // Query: sort params, remove empty values
                if (parsed.queryParams.isNotEmpty()) {
                    val sortedParams = parsed.queryParams
                        .filter { it.value.isNotEmpty() }
                        .sortedBy { it.key }
                        .joinToString("&") { "${it.key}=${it.value}" }
                    append("?$sortedParams")
                }
                
                // Fragment: omit (except for content-significant ones like SoundCloud)
                // For Staqer's use case, we omit all fragments
            }
        } catch (e: Exception) {
            // If parsing fails, return the input
            return url
        }
    }
}

sealed class UrlNormalizationResult {
    data class Success(
        val canonicalUrl: String,
        val rawUrl: String,
        val hadTrackingParams: Boolean,
        val wasShortUrl: Boolean
    ) : UrlNormalizationResult()
    
    data class Failure(
        val rawUrl: String,
        val error: Throwable
    ) : UrlNormalizationResult()
}

// Simple URL parser (using stdlib, no external deps)
private data class ParsedUrl(
    val scheme: String,
    val host: String,
    val port: Int?,
    val path: String,
    val queryParams: Map<String, String>,
    val fragment: String?
)

private fun parseUrl(url: String): ParsedUrl {
    // Simplified URL parsing (in production, use a robust parser or kotlinx-io)
    val schemeEndIndex = url.indexOf("://")
    require(schemeEndIndex > 0) { "Invalid URL: missing scheme" }
    
    val scheme = url.substring(0, schemeEndIndex)
    val afterScheme = url.substring(schemeEndIndex + 3)
    
    val pathStartIndex = afterScheme.indexOf('/')
    val hostPort = if (pathStartIndex > 0) {
        afterScheme.substring(0, pathStartIndex)
    } else {
        afterScheme
    }
    
    val (host, port) = if (':' in hostPort) {
        val parts = hostPort.split(':')
        parts[0] to parts[1].toIntOrNull()
    } else {
        hostPort to null
    }
    
    val pathAndQuery = if (pathStartIndex > 0) {
        afterScheme.substring(pathStartIndex)
    } else {
        "/"
    }
    
    val queryStartIndex = pathAndQuery.indexOf('?')
    val fragmentStartIndex = pathAndQuery.indexOf('#')
    
    val path = when {
        queryStartIndex > 0 -> pathAndQuery.substring(0, queryStartIndex)
        fragmentStartIndex > 0 -> pathAndQuery.substring(0, fragmentStartIndex)
        else -> pathAndQuery
    }
    
    val queryString = if (queryStartIndex > 0) {
        val end = if (fragmentStartIndex > queryStartIndex) fragmentStartIndex else pathAndQuery.length
        pathAndQuery.substring(queryStartIndex + 1, end)
    } else {
        ""
    }
    
    val queryParams = if (queryString.isNotEmpty()) {
        queryString.split('&').associate {
            val kv = it.split('=')
            kv[0] to (kv.getOrNull(1) ?: "")
        }
    } else {
        emptyMap()
    }
    
    val fragment = if (fragmentStartIndex > 0) {
        pathAndQuery.substring(fragmentStartIndex + 1)
    } else {
        null
    }
    
    return ParsedUrl(scheme, host, port, path, queryParams, fragment)
}
```

### 5.2 TrackingParamRemover.kt

```kotlin
package com.staqer.shared.url

class TrackingParamRemover {
    private val trackingParams = setOf(
        // Google Analytics
        "utm_source", "utm_medium", "utm_campaign", "utm_term", "utm_content",
        // Social platforms
        "igshid",        // Instagram share tracking
        "si",            // YouTube share tracking
        "fbclid",        // Facebook click ID
        "twclid",        // Twitter click ID
        "tt_from",       // TikTok referral
        // Generic
        "ref", "ref_src", "ref_url", "feature",
        // Google cross-domain
        "_ga", "_gl",
        // Ad networks
        "gclid", "msclkid",
        // Email marketing
        "mc_cid", "mc_eid", "s_cid"
    )
    
    fun removeTrackingParams(url: String): String {
        if ('?' !in url) return url
        
        val parts = url.split('?', limit = 2)
        if (parts.size != 2) return url
        
        val baseUrl = parts[0]
        val queryString = parts[1]
        
        val cleanParams = queryString.split('&')
            .filter { param ->
                val key = param.substringBefore('=')
                key !in trackingParams
            }
        
        return if (cleanParams.isEmpty()) {
            baseUrl
        } else {
            "$baseUrl?${cleanParams.joinToString("&")}"
        }
    }
}
```

### 5.3 ShortUrlResolver.kt

```kotlin
package com.staqer.shared.url

import io.ktor.client.*
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*

class ShortUrlResolver(
    private val httpClient: HttpClient
) {
    private val shortDomains = setOf(
        "vm.tiktok.com", "youtu.be", "bit.ly", "t.co", "pin.it",
        "instagr.am", "goo.gl", "tinyurl.com", "ow.ly", "buff.ly", "redd.it"
    )
    
    /**
     * Resolve a short URL by following redirects.
     * Returns the final destination URL.
     */
    suspend fun resolve(url: String): String {
        val host = extractHost(url) ?: return url
        
        if (host !in shortDomains) {
            return url  // Not a short URL
        }
        
        return try {
            val response = httpClient.request(url) {
                method = HttpMethod.Head
                timeout {
                    requestTimeoutMillis = 5000  // 5 second timeout
                }
            }
            response.request.url.toString()
        } catch (e: Exception) {
            // If resolution fails, return original URL
            url
        }
    }
    
    private fun extractHost(url: String): String? {
        val hostStart = url.indexOf("://")
        if (hostStart < 0) return null
        
        val afterScheme = url.substring(hostStart + 3)
        val hostEnd = afterScheme.indexOfAny(charArrayOf('/', '?', '#'))
        
        return if (hostEnd > 0) {
            afterScheme.substring(0, hostEnd)
        } else {
            afterScheme
        }.removePrefix("www.").lowercase()
    }
}
```

### 5.4 PlatformDetector.kt

```kotlin
package com.staqer.shared.url

import com.staqer.shared.models.Platform

object PlatformDetector {
    /**
     * Detect the source platform from a URL.
     */
    fun detectPlatform(url: String): Platform {
        val lowercaseUrl = url.lowercase()
        
        return when {
            "instagram.com" in lowercaseUrl || "instagr.am" in lowercaseUrl -> Platform.INSTAGRAM
            "threads.net" in lowercaseUrl -> Platform.THREADS
            "tiktok.com" in lowercaseUrl || "vm.tiktok.com" in lowercaseUrl -> Platform.TIKTOK
            "youtube.com" in lowercaseUrl || "youtu.be" in lowercaseUrl -> Platform.YOUTUBE
            "twitter.com" in lowercaseUrl || "x.com" in lowercaseUrl -> Platform.TWITTER
            "pinterest.com" in lowercaseUrl || "pin.it" in lowercaseUrl -> Platform.PINTEREST
            else -> Platform.OTHER
        }
    }
    
    /**
     * Platform-specific canonical URL forms.
     */
    fun toCanonicalForm(url: String, platform: Platform): String {
        return when (platform) {
            Platform.YOUTUBE -> {
                // Convert youtu.be/ID to youtube.com/watch?v=ID
                if ("youtu.be/" in url) {
                    val videoId = url.substringAfter("youtu.be/").substringBefore("?")
                    "https://youtube.com/watch?v=$videoId"
                } else {
                    url
                }
            }
            else -> url
        }
    }
}
```

---

## 6. Classification Engine

The classification engine implements Tier 2 rule-based classification: keyword matching, TF-IDF scoring, and regex extraction. Tier 1 (on-device AI) stays native.

### 6.1 ClassificationEngine.kt

```kotlin
package com.staqer.shared.classification

import com.staqer.shared.models.ClassificationResult
import com.staqer.shared.models.ExtractedData

interface ClassificationEngine {
    /**
     * Classify content using available signals:
     * - caption text
     * - hashtags
     * - OCR text (if available)
     */
    suspend fun classify(
        captionText: String?,
        hashtags: List<String>,
        ocrText: String? = null
    ): ClassificationResult
}

data class ClassificationResult(
    val category: String,
    val confidence: Double,              // 0.0-1.0
    val secondaryCategory: String? = null,
    val extractedData: ExtractedData? = null,
    val tier: ProcessingTier
)

enum class ProcessingTier {
    TIER_1_ON_DEVICE_AI,
    TIER_2_5_BUNDLED_MODEL,
    TIER_2_RULE_BASED,
    TIER_3_SERVER
}
```

### 6.2 KeywordClassifier.kt

```kotlin
package com.staqer.shared.classification

import com.staqer.shared.classification.dictionaries.KeywordDictionary

class KeywordClassifier(
    private val dictionary: KeywordDictionary,
    private val tfidfScorer: TFIDFScorer
) {
    /**
     * Classify based on keyword/hashtag matching.
     * Returns a map of category -> score.
     */
    fun score(
        captionText: String?,
        hashtags: List<String>,
        ocrText: String? = null
    ): Map<String, Double> {
        val allText = buildString {
            captionText?.let { append(it.lowercase()).append(" ") }
            hashtags.joinToString(" ") { it.removePrefix("#").lowercase() }.let { append(it).append(" ") }
            ocrText?.let { append(it.lowercase()) }
        }
        
        if (allText.isBlank()) {
            return mapOf("general" to 0.3)
        }
        
        val tokens = tokenize(allText)
        val categoryScores = mutableMapOf<String, Double>()
        
        // Score each category
        for (category in dictionary.allCategories()) {
            val keywords = dictionary.getKeywords(category)
            var score = 0.0
            
            for ((keyword, weight) in keywords) {
                if (keyword in allText) {
                    score += weight.toDouble()
                }
            }
            
            // Hashtag boost (hashtags are high-signal)
            for (hashtag in hashtags) {
                val cleanTag = hashtag.removePrefix("#").lowercase()
                for ((keyword, weight) in keywords) {
                    if (keyword == cleanTag) {
                        score += weight * 2.0  // Double weight for hashtag matches
                    }
                }
            }
            
            categoryScores[category] = score
        }
        
        // If top two are within 20%, use TF-IDF to disambiguate
        val sorted = categoryScores.entries.sortedByDescending { it.value }
        if (sorted.size >= 2) {
            val top = sorted[0]
            val second = sorted[1]
            
            if (second.value / top.value > 0.8) {  // Within 20%
                // Apply TF-IDF
                val tfidfScores = mapOf(
                    top.key to tfidfScorer.score(tokens, top.key, dictionary.getKeywords(top.key)),
                    second.key to tfidfScorer.score(tokens, second.key, dictionary.getKeywords(second.key))
                )
                
                // Combine keyword score + TF-IDF
                categoryScores[top.key] = top.value + tfidfScores[top.key]!!
                categoryScores[second.key] = second.value + tfidfScores[second.key]!!
            }
        }
        
        // Normalize to confidence (0-1)
        val maxScore = categoryScores.values.maxOrNull() ?: 0.0
        val totalScore = categoryScores.values.sum()
        
        return if (totalScore > 0) {
            categoryScores.mapValues { it.value / totalScore }
        } else {
            mapOf("general" to 1.0)
        }
    }
    
    private fun tokenize(text: String): List<String> {
        return text.lowercase()
            .split(Regex("\\W+"))
            .filter { it.length > 2 }
    }
}
```

### 6.3 TFIDFScorer.kt

```kotlin
package com.staqer.shared.classification

import kotlin.math.ln
import kotlin.math.log

class TFIDFScorer {
    // Pre-computed IDF values (loaded from resources or bundled)
    private val idfValues: Map<String, Double> = loadIdfValues()
    
    /**
     * Score a set of tokens for a given category using TF-IDF.
     */
    fun score(
        tokens: List<String>,
        category: String,
        keywords: List<Pair<String, Int>>
    ): Double {
        val tokenFrequency = tokens.groupingBy { it }.eachCount()
        var score = 0.0
        
        for ((token, count) in tokenFrequency) {
            val keywordMatch = keywords.find { it.first == token } ?: continue
            
            // Sublinear TF
            val tf = ln(1.0 + count)
            
            // IDF
            val idf = idfValues[token] ?: 1.0
            
            // Keyword weight
            val weight = keywordMatch.second
            
            score += tf * idf * weight
        }
        
        return score
    }
    
    private fun loadIdfValues(): Map<String, Double> {
        // In production, load from resources or embed precomputed values
        // For now, return a simple mock
        return mapOf(
            "preheat" to 2.8,
            "recipe" to 1.5,
            "love" to 0.3,
            "travel" to 2.1,
            "beach" to 1.9,
            "product" to 1.7,
            "price" to 2.4
            // ... full dictionary of ~5000 terms
        )
    }
}
```

### 6.4 RegexExtractor.kt

```kotlin
package com.staqer.shared.classification

import com.staqer.shared.classification.patterns.*

class RegexExtractor(
    private val ingredientPatterns: IngredientPatterns = IngredientPatterns(),
    private val pricePatterns: PricePatterns = PricePatterns(),
    private val locationPatterns: LocationPatterns = LocationPatterns(),
    private val timePatterns: TimePatterns = TimePatterns()
) {
    /**
     * Extract structured data from text using regex patterns.
     */
    fun extract(
        text: String,
        category: String
    ): Map<String, Any> {
        return when (category) {
            "recipe" -> extractRecipeData(text)
            "product" -> extractProductData(text)
            "travel" -> extractTravelData(text)
            "fitness" -> extractFitnessData(text)
            else -> emptyMap()
        }
    }
    
    private fun extractRecipeData(text: String): Map<String, Any> {
        val ingredients = ingredientPatterns.findIngredients(text)
        val times = timePatterns.findTimes(text)
        
        return buildMap {
            if (ingredients.isNotEmpty()) {
                put("ingredients", ingredients)
            }
            if (times.isNotEmpty()) {
                put("prep_time", times.first())
            }
        }
    }
    
    private fun extractProductData(text: String): Map<String, Any> {
        val prices = pricePatterns.findPrices(text)
        
        return buildMap {
            if (prices.isNotEmpty()) {
                put("price", prices.first())
            }
        }
    }
    
    private fun extractTravelData(text: String): Map<String, Any> {
        val locations = locationPatterns.findLocations(text)
        
        return buildMap {
            if (locations.isNotEmpty()) {
                put("location_name", locations.first())
            }
        }
    }
    
    private fun extractFitnessData(text: String): Map<String, Any> {
        val exercises = timePatterns.findExerciseDurations(text)
        
        return buildMap {
            if (exercises.isNotEmpty()) {
                put("exercises", exercises)
            }
        }
    }
}
```

### 6.5 Multilingual Keyword Dictionaries

Keyword dictionaries are organized by language and stored as Kotlin code (not JSON) for type safety and compile-time validation. They're bundled into the KMP module.

**Example: RecipeKeywords.kt (English)**

```kotlin
package com.staqer.shared.classification.dictionaries.en

object RecipeKeywords {
    val keywords = listOf(
        "recipe" to 3,
        "ingredients" to 3,
        "cook" to 2,
        "bake" to 2,
        "tablespoon" to 3,
        "teaspoon" to 3,
        "cup" to 2,
        "preheat" to 3,
        "oven" to 2,
        "prep time" to 3,
        "serving" to 2,
        "homemade" to 1,
        "cuisine" to 2,
        "dish" to 1,
        "simmer" to 2,
        "saute" to 2,
        "marinate" to 2,
        "garnish" to 2,
        "batch cook" to 2
    )
}
```

**Example: RecipeKeywords.kt (Hindi Transliterated)**

```kotlin
package com.staqer.shared.classification.dictionaries.hi

object RecipeKeywords {
    val keywords = listOf(
        // Devanagari
        "रेसिपी" to 3,
        "मसाला" to 2,
        "तड़का" to 2,
        "सब्ज़ी" to 2,
        "पनीर" to 2,
        
        // Transliterated (Latin script)
        "sabzi" to 2,
        "masala" to 2,
        "tadka" to 2,
        "roti" to 2,
        "paneer" to 2,
        "dal" to 2,
        "biryani" to 2,
        "chutney" to 2,
        "paratha" to 2,
        "khana" to 2,
        "pakora" to 2,
        "samosa" to 2,
        "paneer butter masala" to 3,
        "dal makhani" to 3,
        "chole bhature" to 3,
        "aloo gobi" to 3,
        "palak paneer" to 3
    )
}
```

**KeywordDictionary.kt (aggregator)**

```kotlin
package com.staqer.shared.classification.dictionaries

import com.staqer.shared.classification.dictionaries.en.*
import com.staqer.shared.classification.dictionaries.hi.*
import com.staqer.shared.classification.dictionaries.pt.*
import com.staqer.shared.classification.dictionaries.ar.*
import com.staqer.shared.classification.dictionaries.id.*

class KeywordDictionary {
    private val allKeywords: Map<String, List<Pair<String, Int>>> = mapOf(
        "recipe" to (RecipeKeywords_EN.keywords + RecipeKeywords_HI.keywords + RecipeKeywords_PT.keywords),
        "travel" to (TravelKeywords_EN.keywords + TravelKeywords_HI.keywords + TravelKeywords_PT.keywords),
        "product" to (ProductKeywords_EN.keywords + ProductKeywords_PT.keywords),
        "fitness" to (FitnessKeywords_EN.keywords + FitnessKeywords_PT.keywords),
        "education" to (EducationKeywords_EN.keywords + EducationKeywords_HI.keywords),
        "tech_gadgets" to (TechGadgetsKeywords_EN.keywords),
        "parenting" to (ParentingKeywords_EN.keywords + ParentingKeywords_PT.keywords),
        // ... all categories
    )
    
    fun getKeywords(category: String): List<Pair<String, Int>> {
        return allKeywords[category] ?: emptyList()
    }
    
    fun allCategories(): Set<String> = allKeywords.keys
}
```

---

## 7. Network Module

Ktor is used for networking. It compiles to OkHttp on Android and Darwin (URLSession) on iOS with zero bridging overhead.

### 7.1 HttpClientFactory.kt (Expect/Actual)

**commonMain:**

```kotlin
package com.staqer.shared.network

import io.ktor.client.*

expect class HttpClientFactory {
    fun create(): HttpClient
}
```

**androidMain:**

```kotlin
package com.staqer.shared.network

import io.ktor.client.*
import io.ktor.client.engine.okhttp.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.plugins.logging.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

actual class HttpClientFactory {
    actual fun create(): HttpClient {
        return HttpClient(OkHttp) {
            install(ContentNegotiation) {
                json(Json {
                    ignoreUnknownKeys = true
                    isLenient = true
                    encodeDefaults = false
                })
            }
            
            install(Logging) {
                logger = Logger.DEFAULT
                level = LogLevel.INFO
            }
            
            engine {
                config {
                    followRedirects(true)
                    connectTimeout(15, java.util.concurrent.TimeUnit.SECONDS)
                    readTimeout(30, java.util.concurrent.TimeUnit.SECONDS)
                }
            }
        }
    }
}
```

**iosMain:**

```kotlin
package com.staqer.shared.network

import io.ktor.client.*
import io.ktor.client.engine.darwin.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.client.plugins.logging.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

actual class HttpClientFactory {
    actual fun create(): HttpClient {
        return HttpClient(Darwin) {
            install(ContentNegotiation) {
                json(Json {
                    ignoreUnknownKeys = true
                    isLenient = true
                    encodeDefaults = false
                })
            }
            
            install(Logging) {
                logger = Logger.DEFAULT
                level = LogLevel.INFO
            }
            
            engine {
                configureRequest {
                    setAllowsCellularAccess(true)
                }
            }
        }
    }
}
```

### 7.2 StaqerApiClient.kt

```kotlin
package com.staqer.shared.network

import com.staqer.shared.network.requests.*
import com.staqer.shared.network.responses.*
import io.ktor.client.*
import io.ktor.client.call.*
import io.ktor.client.request.*
import io.ktor.http.*

class StaqerApiClient(
    private val httpClient: HttpClient,
    private val baseUrl: String = "https://api.staqer.app/v1"
) {
    /**
     * Trigger deep extract on the server.
     */
    suspend fun requestDeepExtract(request: DeepExtractRequest): NetworkResult<DeepExtractResponse> {
        return try {
            val response = httpClient.post("$baseUrl/extract") {
                contentType(ContentType.Application.Json)
                setBody(request)
            }
            NetworkResult.Success(response.body())
        } catch (e: Exception) {
            NetworkResult.Failure(e)
        }
    }
    
    /**
     * Save content metadata to backend.
     */
    suspend fun saveContent(request: SaveContentRequest): NetworkResult<Unit> {
        return try {
            httpClient.post("$baseUrl/saves") {
                contentType(ContentType.Application.Json)
                setBody(request)
            }
            NetworkResult.Success(Unit)
        } catch (e: Exception) {
            NetworkResult.Failure(e)
        }
    }
    
    /**
     * Fetch user profile and subscription state.
     */
    suspend fun getUserProfile(userId: String): NetworkResult<UserProfileResponse> {
        return try {
            val response = httpClient.get("$baseUrl/users/$userId/profile")
            NetworkResult.Success(response.body())
        } catch (e: Exception) {
            NetworkResult.Failure(e)
        }
    }
    
    /**
     * Check and update quota state.
     */
    suspend fun getQuotaState(userId: String): NetworkResult<QuotaStateResponse> {
        return try {
            val response = httpClient.get("$baseUrl/users/$userId/quota")
            NetworkResult.Success(response.body())
        } catch (e: Exception) {
            NetworkResult.Failure(e)
        }
    }
}

sealed class NetworkResult<out T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Failure(val error: Throwable) : NetworkResult<Nothing>()
}
```

### 7.3 Request/Response DTOs

**DeepExtractRequest.kt:**

```kotlin
package com.staqer.shared.network.requests

import kotlinx.serialization.Serializable

@Serializable
data class DeepExtractRequest(
    val url: String,
    val platform: String,
    val saveId: String,
    val deviceCanTranscribe: Boolean
)
```

**DeepExtractResponse.kt:**

```kotlin
package com.staqer.shared.network.responses

import com.staqer.shared.models.ExtractedData
import kotlinx.serialization.Serializable

@Serializable
data class DeepExtractResponse(
    val videoUrl: String? = null,
    val fullCaption: String? = null,
    val creator: String? = null,
    val hashtags: List<String> = emptyList(),
    val thumbnailUrl: String? = null,
    val transcript: String? = null,
    val category: String,
    val extractedData: ExtractedData? = null,
    val richnessScore: Double
)
```

---

## 8. Platform-Specific Expect/Actual

For functionality that has platform-specific implementations but needs a common interface, use KMP's `expect`/`actual` mechanism.

### 8.1 Logger.kt

**commonMain:**

```kotlin
package com.staqer.shared.core

expect object Logger {
    fun d(tag: String, message: String)
    fun w(tag: String, message: String)
    fun e(tag: String, message: String, throwable: Throwable? = null)
}
```

**androidMain:**

```kotlin
package com.staqer.shared.core

import android.util.Log

actual object Logger {
    actual fun d(tag: String, message: String) {
        Log.d(tag, message)
    }
    
    actual fun w(tag: String, message: String) {
        Log.w(tag, message)
    }
    
    actual fun e(tag: String, message: String, throwable: Throwable?) {
        Log.e(tag, message, throwable)
    }
}
```

**iosMain:**

```kotlin
package com.staqer.shared.core

import platform.Foundation.NSLog

actual object Logger {
    actual fun d(tag: String, message: String) {
        NSLog("[$tag] $message")
    }
    
    actual fun w(tag: String, message: String) {
        NSLog("[WARNING][$tag] $message")
    }
    
    actual fun e(tag: String, message: String, throwable: Throwable?) {
        val errorMsg = throwable?.let { " - Error: ${it.message}" } ?: ""
        NSLog("[ERROR][$tag] $message$errorMsg")
    }
}
```

### 8.2 SecureStorage.kt

**commonMain:**

```kotlin
package com.staqer.shared.core

expect class SecureStorage {
    suspend fun save(key: String, value: String)
    suspend fun get(key: String): String?
    suspend fun delete(key: String)
}
```

**androidMain:**

```kotlin
package com.staqer.shared.core

import android.content.Context
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

actual class SecureStorage(private val context: Context) {
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()
    
    private val prefs = EncryptedSharedPreferences.create(
        context,
        "staqer_secure_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    actual suspend fun save(key: String, value: String) {
        prefs.edit().putString(key, value).apply()
    }
    
    actual suspend fun get(key: String): String? {
        return prefs.getString(key, null)
    }
    
    actual suspend fun delete(key: String) {
        prefs.edit().remove(key).apply()
    }
}
```

**iosMain:**

```kotlin
package com.staqer.shared.core

import platform.Foundation.*
import platform.Security.*

actual class SecureStorage {
    actual suspend fun save(key: String, value: String) {
        val query = mapOf(
            kSecClass to kSecClassGenericPassword,
            kSecAttrAccount to key,
            kSecValueData to value.encodeToByteArray().toNSData()
        )
        
        SecItemDelete(query as CFDictionaryRef)
        SecItemAdd(query as CFDictionaryRef, null)
    }
    
    actual suspend fun get(key: String): String? {
        val query = mapOf(
            kSecClass to kSecClassGenericPassword,
            kSecAttrAccount to key,
            kSecReturnData to kCFBooleanTrue,
            kSecMatchLimit to kSecMatchLimitOne
        )
        
        val result: CFTypeRefVar = nativeHeap.alloc()
        val status = SecItemCopyMatching(query as CFDictionaryRef, result.ptr)
        
        return if (status == errSecSuccess) {
            val data = result.value as NSData
            data.toByteArray().decodeToString()
        } else {
            null
        }
    }
    
    actual suspend fun delete(key: String) {
        val query = mapOf(
            kSecClass to kSecClassGenericPassword,
            kSecAttrAccount to key
        )
        
        SecItemDelete(query as CFDictionaryRef)
    }
}

// Helper extension
private fun ByteArray.toNSData(): NSData {
    return NSData.create(bytes = this.refTo(0), length = this.size.toULong())
}

private fun NSData.toByteArray(): ByteArray {
    return ByteArray(length.toInt()).apply {
        usePinned {
            memcpy(it.addressOf(0), bytes, length)
        }
    }
}
```

### 8.3 Platform-Specific AI Invocation

The on-device AI (Foundation Models, Gemini Nano) stays entirely native. The KMP module provides the interface and data models, but the native apps implement the actual AI calls.

**From KMP perspective:**

```kotlin
// commonMain
package com.staqer.shared.classification

/**
 * Interface for Tier 1 on-device AI classification.
 * Implemented natively on each platform.
 */
expect class OnDeviceAIClassifier {
    suspend fun classify(
        captionText: String?,
        ocrText: String?,
        hashtags: List<String>
    ): ClassificationResult?
}
```

**Native implementations:**

iOS (Swift):
```swift
// Implements the OnDeviceAIClassifier interface
class OnDeviceAIClassifierImpl {
    func classify(captionText: String?, ocrText: String?, hashtags: [String]) async -> ClassificationResult? {
        // Use Foundation Models here
        let session = LanguageModelSession()
        // ... classification logic
        return classificationResult
    }
}
```

Android (Kotlin):
```kotlin
// Implements the OnDeviceAIClassifier interface
class OnDeviceAIClassifierImpl {
    suspend fun classify(captionText: String?, ocrText: String?, hashtags: List<String>): ClassificationResult? {
        // Use Gemini Nano here
        val model = GenerativeModel(modelName = "gemini-nano")
        // ... classification logic
        return classificationResult
    }
}
```

---

## 9. iOS Integration

### 9.1 CocoaPods Export

The KMP shared module is exported as a CocoaPods framework for iOS consumption.

**shared/staqer-shared.podspec:**

```ruby
Pod::Spec.new do |spec|
    spec.name                     = 'StaqerShared'
    spec.version                  = '1.0.0'
    spec.homepage                 = 'https://staqer.app'
    spec.source                   = { :http=> ''}
    spec.authors                  = ''
    spec.license                  = ''
    spec.summary                  = 'Staqer Kotlin Multiplatform shared module'
    spec.vendored_frameworks      = 'build/cocoapods/framework/StaqerShared.framework'
    spec.libraries                = 'c++'
    spec.ios.deployment_target = '15.0'
                
    spec.pod_target_xcconfig = {
        'KOTLIN_PROJECT_PATH' => ':shared',
        'PRODUCT_MODULE_NAME' => 'StaqerShared',
    }
                
    spec.script_phases = [
        {
            :name => 'Build StaqerShared',
            :execution_position => :before_compile,
            :shell_path => '/bin/sh',
            :script => <<-SCRIPT
                if [ "YES" = "$OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED" ]; then
                  echo "Skipping Gradle build task invocation due to OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED environment variable set to \"YES\""
                  exit 0
                fi
                set -ev
                REPO_ROOT="$PODS_TARGET_SRCROOT"
                "$REPO_ROOT/../gradlew" -p "$REPO_ROOT" $KOTLIN_PROJECT_PATH:syncFramework \
                    -Pkotlin.native.cocoapods.platform=$PLATFORM_NAME \
                    -Pkotlin.native.cocoapods.archs="$ARCHS" \
                    -Pkotlin.native.cocoapods.configuration="$CONFIGURATION"
            SCRIPT
        }
    ]
end
```

**iOS Podfile:**

```ruby
platform :ios, '15.0'
use_frameworks!

target 'StaqerApp' do
  pod 'StaqerShared', :path => '../shared'
end
```

### 9.2 Swift-Friendly API Design

KMP generates Swift-compatible APIs automatically, but some patterns need attention:

**Use `@ObjCName` for clear Swift APIs:**

```kotlin
// commonMain
@ObjCName("StaqerURLNormalizer")
class UrlNormalizer {
    @ObjCName("normalizeURL")
    suspend fun normalize(rawUrl: String): UrlNormalizationResult {
        // ...
    }
}
```

In Swift:
```swift
let normalizer = StaqerURLNormalizer()
let result = try await normalizer.normalizeURL(rawUrl: "https://instagram.com/...")
```

**Avoid Kotlin-specific types in public APIs:**

- Use `List<T>` instead of `Collection<T>` (Swift sees `KotlinArray`)
- Use nullable `T?` explicitly (Swift Optional)
- Avoid `sealed class` in public APIs (Swift doesn't model them well) — use enums or abstract classes
- Prefer `suspend fun` over `Flow` for iOS (async/await is more natural than Combine)

**Coroutines to async/await:**

KMP suspend functions are automatically bridged to Swift async/await when targeting iOS 13+:

```kotlin
suspend fun fetchData(): Result<Data>
```

Becomes:
```swift
func fetchData() async throws -> Result<Data>
```

### 9.3 iOS Build Configuration

**shared/build.gradle.kts (iOS-specific):**

```kotlin
kotlin {
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64()
    ).forEach { iosTarget ->
        iosTarget.binaries.framework {
            baseName = "StaqerShared"
            isStatic = true
            
            // Export transitive dependencies for Swift visibility
            export(libs.kotlinx.datetime)
            export(libs.kotlinx.coroutines.core)
            
            // Embed bitcode for distribution (if needed)
            embedBitcode = Framework.BitcodeEmbeddingMode.DISABLE
        }
    }
}
```

### 9.4 Swift Usage Example

```swift
import StaqerShared

class ContentProcessor {
    let apiClient: StaqerApiClient
    let urlNormalizer: StaqerURLNormalizer
    let classifier: KeywordClassifier
    
    init() {
        let httpClient = HttpClientFactory().create()
        self.apiClient = StaqerApiClient(httpClient: httpClient)
        self.urlNormalizer = StaqerURLNormalizer(
            trackingParamRemover: TrackingParamRemover(),
            shortUrlResolver: ShortUrlResolver(httpClient: httpClient)
        )
        self.classifier = KeywordClassifier(
            dictionary: KeywordDictionary(),
            tfidfScorer: TFIDFScorer()
        )
    }
    
    func processSave(rawUrl: String, caption: String?, hashtags: [String]) async throws -> SavedContent {
        // 1. Normalize URL
        let normalizationResult = try await urlNormalizer.normalize(rawUrl: rawUrl)
        guard let normalizedUrl = (normalizationResult as? UrlNormalizationResultSuccess)?.canonicalUrl else {
            throw ProcessingError.urlNormalizationFailed
        }
        
        // 2. Detect platform
        let platform = PlatformDetector.shared.detectPlatform(url: normalizedUrl)
        
        // 3. Classify (Tier 2 rule-based)
        let classificationResult = try await classifier.classify(
            captionText: caption,
            hashtags: hashtags,
            ocrText: nil
        )
        
        // 4. Create SavedContent model
        let savedContent = SavedContent(
            id: UUID().uuidString,
            url: normalizedUrl,
            rawUrl: rawUrl,
            platform: platform,
            captionText: caption,
            transcriptText: nil,
            thumbnailUrl: nil,
            thumbnailLocalPath: nil,
            thumbnailAspectRatio: nil,
            category: classificationResult.category,
            categoryConfidence: classificationResult.confidence,
            richnessScore: 0.5,
            extractedData: classificationResult.extractedData,
            userNote: nil,
            userCategoryOverride: nil,
            collectionIds: [],
            creatorHandle: nil,
            createdAt: Kotlinx_datetimeInstant.companion.fromEpochMilliseconds(epochMilliseconds: Int64(Date().timeIntervalSince1970 * 1000)),
            processedAt: nil,
            status: .complete,
            mayExpire: false,
            secondaryCategory: nil,
            customCategoryId: nil
        )
        
        return savedContent
    }
}
```

---

## 10. Android Integration

### 10.1 Gradle Dependency

Android consumes the KMP shared module as a direct Gradle dependency (no framework export needed).

**androidApp/build.gradle.kts:**

```kotlin
plugins {
    alias(libs.plugins.androidApplication)
    alias(libs.plugins.kotlinAndroid)
}

android {
    namespace = "com.staqer.android"
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.staqer.android"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }
}

dependencies {
    implementation(project(":shared"))
    
    // Android-specific dependencies
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.material3)
    
    // ... other Android deps
}
```

### 10.2 Kotlin Consumption

Android code consumes the shared module directly as Kotlin. No bridging layer needed.

**Example: ContentRepository.kt (Android)**

```kotlin
package com.staqer.android.data

import com.staqer.shared.classification.KeywordClassifier
import com.staqer.shared.classification.dictionaries.KeywordDictionary
import com.staqer.shared.classification.TFIDFScorer
import com.staqer.shared.models.SavedContent
import com.staqer.shared.models.Platform
import com.staqer.shared.models.SaveStatus
import com.staqer.shared.network.StaqerApiClient
import com.staqer.shared.url.UrlNormalizer
import com.staqer.shared.url.PlatformDetector
import kotlinx.datetime.Clock
import java.util.UUID

class ContentRepository(
    private val apiClient: StaqerApiClient,
    private val urlNormalizer: UrlNormalizer,
    private val classifier: KeywordClassifier
) {
    suspend fun processSave(
        rawUrl: String,
        caption: String?,
        hashtags: List<String>
    ): Result<SavedContent> {
        return try {
            // 1. Normalize URL
            val normalizationResult = urlNormalizer.normalize(rawUrl)
            val normalizedUrl = when (normalizationResult) {
                is UrlNormalizationResult.Success -> normalizationResult.canonicalUrl
                is UrlNormalizationResult.Failure -> return Result.failure(normalizationResult.error)
            }
            
            // 2. Detect platform
            val platform = PlatformDetector.detectPlatform(normalizedUrl)
            
            // 3. Classify
            val classificationResult = classifier.score(
                captionText = caption,
                hashtags = hashtags,
                ocrText = null
            )
            
            val topCategory = classificationResult.maxByOrNull { it.value }
            
            // 4. Create SavedContent
            val savedContent = SavedContent(
                id = UUID.randomUUID().toString(),
                url = normalizedUrl,
                rawUrl = rawUrl,
                platform = platform,
                captionText = caption,
                transcriptText = null,
                thumbnailUrl = null,
                thumbnailLocalPath = null,
                thumbnailAspectRatio = null,
                category = topCategory?.key ?: "general",
                categoryConfidence = topCategory?.value ?: 0.0,
                richnessScore = 0.5,
                extractedData = null,
                userNote = null,
                userCategoryOverride = null,
                collectionIds = emptyList(),
                creatorHandle = null,
                createdAt = Clock.System.now(),
                processedAt = null,
                status = SaveStatus.COMPLETE,
                mayExpire = false,
                secondaryCategory = null,
                customCategoryId = null
            )
            
            Result.success(savedContent)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 10.3 Compose Integration

Shared models work seamlessly with Jetpack Compose:

```kotlin
@Composable
fun SavedContentCard(content: SavedContent) {
    Card(
        modifier = Modifier.fillMaxWidth()
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = content.extractedData?.recipe?.dishName ?: "Untitled",
                style = MaterialTheme.typography.titleLarge
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            Text(
                text = "${content.category} · ${content.platform.name}",
                style = MaterialTheme.typography.bodySmall
            )
            
            content.captionText?.let { caption ->
                Text(
                    text = caption,
                    style = MaterialTheme.typography.bodyMedium,
                    maxLines = 2,
                    overflow = TextOverflow.Ellipsis
                )
            }
        }
    }
}
```

---

## 11. Testing

### 11.1 commonTest Structure

```
src/commonTest/kotlin/com/staqer/shared/
├── url/
│   ├── UrlNormalizerTest.kt
│   ├── TrackingParamRemoverTest.kt
│   └── PlatformDetectorTest.kt
├── classification/
│   ├── KeywordClassifierTest.kt
│   ├── TFIDFScorerTest.kt
│   └── RegexExtractorTest.kt
├── models/
│   └── SavedContentSerializationTest.kt
└── network/
    └── StaqerApiClientTest.kt
```

### 11.2 Example Test: UrlNormalizerTest.kt

```kotlin
package com.staqer.shared.url

import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue

class UrlNormalizerTest {
    private val normalizer = UrlNormalizer()
    
    @Test
    fun `strips tracking parameters`() = runTest {
        val rawUrl = "https://instagram.com/reel/ABC123/?utm_source=twitter&igshid=xyz123"
        val normalized = normalizer.normalizeLocal(rawUrl)
        
        assertEquals("https://instagram.com/reel/ABC123", normalized)
    }
    
    @Test
    fun `removes www prefix`() = runTest {
        val rawUrl = "https://www.tiktok.com/@user/video/123"
        val normalized = normalizer.normalizeLocal(rawUrl)
        
        assertTrue(normalized.startsWith("https://tiktok.com/"))
    }
    
    @Test
    fun `removes trailing slash`() = runTest {
        val rawUrl = "https://youtube.com/watch?v=ABC123/"
        val normalized = normalizer.normalizeLocal(rawUrl)
        
        assertEquals("https://youtube.com/watch?v=ABC123", normalized)
    }
    
    @Test
    fun `sorts query parameters`() = runTest {
        val rawUrl = "https://youtube.com/watch?v=ABC&list=XYZ&t=30"
        val normalized = normalizer.normalizeLocal(rawUrl)
        
        assertEquals("https://youtube.com/watch?list=XYZ&t=30&v=ABC", normalized)
    }
}
```

### 11.3 Mock Network Tests

```kotlin
package com.staqer.shared.network

import io.ktor.client.*
import io.ktor.client.engine.mock.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.http.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue

class StaqerApiClientTest {
    @Test
    fun `deep extract request returns success`() = runTest {
        val mockEngine = MockEngine { request ->
            respond(
                content = """{"category":"recipe","richnessScore":0.92}""",
                status = HttpStatusCode.OK,
                headers = headersOf(HttpHeaders.ContentType, ContentType.Application.Json.toString())
            )
        }
        
        val mockClient = HttpClient(mockEngine) {
            install(ContentNegotiation) {
                json()
            }
        }
        
        val apiClient = StaqerApiClient(mockClient, baseUrl = "https://test.api")
        val request = DeepExtractRequest(
            url = "https://instagram.com/reel/test",
            platform = "instagram",
            saveId = "test-id",
            deviceCanTranscribe = true
        )
        
        val result = apiClient.requestDeepExtract(request)
        
        assertTrue(result is NetworkResult.Success)
        assertEquals("recipe", (result as NetworkResult.Success).data.category)
        assertEquals(0.92, result.data.richnessScore)
    }
}
```

### 11.4 Platform-Specific Tests

**androidTest:**

```kotlin
package com.staqer.shared.core

import android.content.Context
import androidx.test.core.app.ApplicationProvider
import kotlinx.coroutines.test.runTest
import kotlin.test.Test
import kotlin.test.assertEquals

class SecureStorageTest {
    @Test
    fun saveAndRetrieve() = runTest {
        val context = ApplicationProvider.getApplicationContext<Context>()
        val storage = SecureStorage(context)
        
        storage.save("test_key", "test_value")
        val retrieved = storage.get("test_key")
        
        assertEquals("test_value", retrieved)
    }
}
```

**iosTest:**

Similar pattern using XCTest framework (run via `./gradlew iosX64Test`).

### 11.5 Test Fixtures

Create shared test fixtures for reusable test data:

```kotlin
package com.staqer.shared.fixtures

import com.staqer.shared.models.*
import kotlinx.datetime.Clock

object TestFixtures {
    fun savedContent(
        category: String = "recipe",
        platform: Platform = Platform.INSTAGRAM
    ) = SavedContent(
        id = "test-id",
        url = "https://instagram.com/reel/test",
        rawUrl = "https://instagram.com/reel/test",
        platform = platform,
        captionText = "Test caption",
        transcriptText = null,
        thumbnailUrl = null,
        thumbnailLocalPath = null,
        thumbnailAspectRatio = 1.0,
        category = category,
        categoryConfidence = 0.9,
        richnessScore = 0.8,
        extractedData = null,
        userNote = null,
        userCategoryOverride = null,
        collectionIds = emptyList(),
        creatorHandle = "@testuser",
        createdAt = Clock.System.now(),
        processedAt = null,
        status = SaveStatus.COMPLETE,
        mayExpire = false,
        secondaryCategory = null,
        customCategoryId = null
    )
    
    fun recipeData() = RecipeData(
        dishName = "Test Recipe",
        ingredients = listOf(
            Ingredient(item = "flour", quantity = "2 cups"),
            Ingredient(item = "sugar", quantity = "1 cup")
        ),
        steps = listOf(
            RecipeStep(text = "Mix ingredients", timeSeconds = null),
            RecipeStep(text = "Bake for 30 minutes", timeSeconds = 1800)
        ),
        prepTime = "45 minutes",
        servings = 4,
        originalServings = 4,
        cuisine = "Italian",
        difficulty = "Easy",
        dietaryTags = emptyList()
    )
}
```

---

## 12. Versioning & Publishing

### 12.1 Semantic Versioning

Follow semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR**: Breaking API changes (e.g., rename a data class field)
- **MINOR**: New features, backward-compatible (e.g., add a new API method)
- **PATCH**: Bug fixes, no API changes

**Version defined in shared/build.gradle.kts:**

```kotlin
version = "1.0.0"
```

### 12.2 Publishing to Maven Local (for development)

```bash
./gradlew :shared:publishToMavenLocal
```

This publishes the KMP artifact to your local Maven repository (`~/.m2/repository`) for testing before release.

### 12.3 CI/CD Pipeline

**GitHub Actions example:**

```yaml
name: Build Shared Module

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: macos-latest  # macOS required for iOS builds
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-
    
    - name: Build shared module
      run: ./gradlew :shared:build
    
    - name: Run tests (common)
      run: ./gradlew :shared:allTests
    
    - name: Publish to Maven Local
      run: ./gradlew :shared:publishToMavenLocal
```

### 12.4 Release Process

1. **Update version** in `shared/build.gradle.kts`
2. **Tag the release:**
   ```bash
   git tag -a shared-v1.0.0 -m "Shared module v1.0.0"
   git push origin shared-v1.0.0
   ```
3. **Build and publish artifacts:**
   ```bash
   ./gradlew :shared:build
   ./gradlew :shared:publishToMavenLocal
   ```
4. **Update iOS Podfile** to point to the new version
5. **Update Android dependency** version in `libs.versions.toml`
6. **Test integration** on both platforms
7. **Document breaking changes** in CHANGELOG.md

### 12.5 Dependency Updates

Monitor dependencies for security and feature updates:

```bash
./gradlew dependencyUpdates
```

Update `gradle/libs.versions.toml` as needed. Test thoroughly after each dependency bump.

---

## 13. Migration Path

### 13.1 Phase 1: Foundation (Week 1-2)

**Goal:** Establish the KMP project structure and migrate core data models.

**Tasks:**
1. Create `shared/` module with basic Gradle setup
2. Migrate data models to `commonMain`:
   - `SavedContent.kt`
   - `Category.kt`
   - `ExtractionResult.kt`
   - `Platform.kt` enum
3. Add kotlinx.serialization annotations
4. Write basic unit tests for model serialization
5. Build iOS framework and integrate via CocoaPods
6. Add Android dependency in `androidApp/build.gradle.kts`
7. Verify models are accessible from both native apps

**Success Criteria:**
- Native apps can create and serialize `SavedContent` instances
- No compilation errors on iOS or Android
- Unit tests pass on all platforms

### 13.2 Phase 2: URL Normalization (Week 3)

**Goal:** Share URL processing logic across platforms.

**Tasks:**
1. Implement `UrlNormalizer.kt` in `commonMain`
2. Implement `TrackingParamRemover.kt`
3. Implement `PlatformDetector.kt`
4. Add `ShortUrlResolver.kt` (requires Ktor HttpClient)
5. Set up Ktor HttpClient factory (expect/actual)
6. Write comprehensive tests for URL normalization
7. Replace native URL normalization code with KMP calls

**Success Criteria:**
- Share extension on both platforms uses `UrlNormalizer`
- All URL normalization tests pass
- Native URL code is removed

### 13.3 Phase 3: Classification Engine (Week 4-5)

**Goal:** Share Tier 2 rule-based classification logic.

**Tasks:**
1. Migrate keyword dictionaries to `commonMain/resources`
2. Implement `KeywordClassifier.kt`
3. Implement `TFIDFScorer.kt` with bundled IDF values
4. Implement `RegexExtractor.kt` with pattern classes
5. Add multilingual keyword support (Hindi, Portuguese, Arabic, Bahasa)
6. Write classification accuracy tests
7. Integrate into native classification pipelines (fallback from Tier 1)

**Success Criteria:**
- Tier 2 classification runs from shared code
- Native apps call `KeywordClassifier.classify()` when Tier 1 is unavailable
- Accuracy matches or exceeds existing native implementations

### 13.4 Phase 4: Network Module (Week 6)

**Goal:** Share API client and network logic.

**Tasks:**
1. Set up Ktor client configuration (expect/actual)
2. Implement `StaqerApiClient.kt` with REST endpoints
3. Create request/response DTOs
4. Add retry logic and error handling
5. Write mock network tests
6. Integrate into native data layers

**Success Criteria:**
- Deep extract requests go through shared `StaqerApiClient`
- Network errors are handled consistently
- All API integration tests pass

### 13.5 Phase 5: Analytics & Subscription (Week 7)

**Goal:** Share business logic for analytics and quota management.

**Tasks:**
1. Define analytics event schemas in `commonMain`
2. Implement `EventLogger` (expect/actual)
3. Implement `QuotaManager.kt` with reset logic
4. Implement `SubscriptionState.kt` and `FeatureGate.kt`
5. Wire up native analytics SDKs to shared event definitions
6. Wire up native billing systems to shared subscription state

**Success Criteria:**
- Analytics events are defined once, fired consistently
- Quota logic is shared and tested
- Pro feature gating works on both platforms

### 13.6 Phase 6: Optimization & Polish (Week 8+)

**Goal:** Refine shared code, optimize performance, and document.

**Tasks:**
1. Profile shared module performance (Kotlin/Native optimization)
2. Add caching layers where needed
3. Document all public APIs with KDoc
4. Create developer onboarding guide
5. Set up CI/CD for automated builds and tests
6. Monitor shared code coverage (aim for 80%+)

**Success Criteria:**
- Shared module has comprehensive API documentation
- CI/CD pipeline runs on every commit
- Code coverage ≥ 80%
- No performance regressions from migration

### 13.7 Gradual Migration Strategy

**Do NOT migrate everything at once.** Use this incremental approach:

1. **Start with pure business logic** (no platform dependencies)
2. **Migrate one feature at a time** (URL normalization, then classification, then network)
3. **Keep both implementations running in parallel** for 1-2 weeks per feature
4. **A/B test shared vs native** where possible (e.g., classification accuracy)
5. **Remove native code only after** the shared implementation is proven stable
6. **Document every migration** in a migration log

**Migration checklist per feature:**
- [ ] Shared implementation complete
- [ ] Unit tests passing (≥ 90% coverage)
- [ ] iOS integration tested
- [ ] Android integration tested
- [ ] Performance benchmarked (no regressions)
- [ ] Native code deprecated (marked with `@Deprecated`)
- [ ] Native code removed after 2-week grace period
- [ ] Documentation updated

---

## Conclusion

This KMP Shared Module Architecture Plan provides a complete blueprint for sharing 30-40% of Staqer's codebase across iOS and Android while preserving native performance and platform integration quality for the remaining 60-70%.

**Key Takeaways:**

1. **Clear Boundaries**: UI and on-device AI stay native. Business logic, networking, and data models are shared.
2. **Production-Ready Stack**: Ktor (networking), kotlinx.serialization (data models), SQLDelight (database), kotlinx-datetime (time handling).
3. **Swift-Friendly**: Exported framework works seamlessly with Swift async/await and SwiftUI.
4. **Android-Native**: Direct Kotlin consumption with zero bridging overhead.
5. **Testable**: Comprehensive testing strategy with commonTest, platform-specific tests, and fixtures.
6. **Incremental Migration**: Gradual rollout minimizes risk and validates each component before removing native code.

**Next Steps:**

1. Review this plan with the team
2. Set up the initial KMP module (Phase 1)
3. Migrate data models first (lowest risk, highest value)
4. Follow the 8-week migration path outlined in Section 13
5. Iterate based on real-world integration feedback

This architecture will reduce duplication, speed up feature development, and maintain the high-quality native experience that Staqer users expect.

---

**Document Version:** 1.0  
**Last Updated:** 2026-02-24  
**Maintained By:** Lead Developer