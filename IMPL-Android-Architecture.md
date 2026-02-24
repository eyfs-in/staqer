# Android Architecture Plan — Staqer
**Version:** 1.0  
**Date:** 2026-02-24  
**Platform:** Native Android (Kotlin, Jetpack Compose, Room, Material 3)  
**Shared Layer:** Kotlin Multiplatform (KMP)  

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [App Architecture](#2-app-architecture)
3. [Share/Intent Architecture](#3-shareintent-architecture)
4. [Data Layer](#4-data-layer)
5. [AI/ML Layer](#5-aiml-layer)
6. [Network Layer](#6-network-layer)
7. [KMP Integration](#7-kmp-integration)
8. [Key Libraries & Dependencies](#8-key-libraries--dependencies)
9. [Testing Strategy](#9-testing-strategy)
10. [Build & CI/CD](#10-build--cicd)
11. [Device Compatibility](#11-device-compatibility)
12. [Sprint-by-Sprint Breakdown](#12-sprint-by-sprint-breakdown)

---

## 1. Project Structure

### 1.1 Multi-Module Gradle Setup

Staqer uses a **feature-modular** architecture for scalability, clean separation of concerns, and faster build times.

```
staqer/
├── app/                                    # Main application module
│   ├── src/main/
│   │   ├── AndroidManifest.xml
│   │   ├── kotlin/com/staqapp/
│   │   │   ├── StaqApplication.kt         # Application class, Hilt setup
│   │   │   ├── MainActivity.kt            # Single Activity + Compose Navigation
│   │   │   └── navigation/                # App-level nav graph
│   │   └── res/
│   └── build.gradle.kts                   # App module build config
│
├── feature/                                # Feature modules
│   ├── share/                             # F01: Share Extension
│   │   ├── src/main/
│   │   │   ├── AndroidManifest.xml        # Intent filters for ACTION_SEND
│   │   │   └── kotlin/com/staqapp/feature/share/
│   │   │       ├── ShareReceiverActivity.kt
│   │   │       ├── ui/ShareConfirmationSheet.kt
│   │   │       └── data/PendingSaveRepository.kt
│   │   └── build.gradle.kts
│   │
│   ├── classification/                    # F02: Auto-Categorization
│   │   ├── src/main/kotlin/com/staqapp/feature/classification/
│   │   │   ├── ui/RecategorizeSheet.kt
│   │   │   ├── domain/
│   │   │   │   ├── Tier1Classifier.kt     # Gemini Nano Prompt API
│   │   │   │   ├── Tier25Classifier.kt    # TFLite bundled model
│   │   │   │   ├── Tier2Classifier.kt     # Rule-based keyword matching
│   │   │   │   └── ClassifierTierSelector.kt
│   │   │   └── data/
│   │   │       ├── CategoryRepository.kt
│   │   │       └── KeywordDictionaries.kt
│   │   └── build.gradle.kts
│   │
│   ├── library/                           # F03: Content Library
│   │   ├── src/main/kotlin/com/staqapp/feature/library/
│   │   │   ├── ui/
│   │   │   │   ├── LibraryScreen.kt       # Main grid/list view
│   │   │   │   ├── LibraryViewModel.kt
│   │   │   │   ├── GridView.kt
│   │   │   │   ├── StaggeredGridView.kt
│   │   │   │   ├── ListView.kt
│   │   │   │   ├── SearchOverlay.kt
│   │   │   │   └── CategoryFilterBar.kt
│   │   │   └── data/
│   │   │       └── LibraryRepository.kt
│   │   └── build.gradle.kts
│   │
│   ├── cards/                             # F04: Structured Data Cards
│   │   ├── src/main/kotlin/com/staqapp/feature/cards/
│   │   │   ├── ui/
│   │   │   │   ├── CardDetailScreen.kt
│   │   │   │   ├── templates/
│   │   │   │   │   ├── RecipeCard.kt
│   │   │   │   │   ├── TravelCard.kt
│   │   │   │   │   ├── ProductCard.kt
│   │   │   │   │   ├── FitnessCard.kt
│   │   │   │   │   ├── EducationCard.kt
│   │   │   │   │   └── GenericCard.kt
│   │   │   │   └── components/
│   │   │   │       ├── IngredientChecklist.kt
│   │   │   │       ├── TimerChip.kt
│   │   │   │       └── WeatherWidget.kt
│   │   │   └── domain/
│   │   │       └── CardTemplateRegistry.kt
│   │   └── build.gradle.kts
│   │
│   ├── deepextract/                       # F05: Deep Extract
│   │   ├── src/main/kotlin/com/staqapp/feature/deepextract/
│   │   │   ├── ui/
│   │   │   │   ├── DeepExtractProgressSheet.kt
│   │   │   │   └── QuotaExhaustedDialog.kt
│   │   │   ├── domain/
│   │   │   │   ├── OnDeviceExtractor.kt
│   │   │   │   ├── TranscriptionEngine.kt  # ML Kit Speech
│   │   │   │   ├── OcrEngine.kt            # ML Kit TextRecognition
│   │   │   │   └── KeyFrameExtractor.kt
│   │   │   ├── data/
│   │   │   │   ├── ExtractRepository.kt
│   │   │   │   └── QuotaRepository.kt
│   │   │   └── worker/
│   │   │       └── DeepExtractWorker.kt     # WorkManager background processing
│   │   └── build.gradle.kts
│   │
│   └── settings/                          # Settings & Account
│       ├── src/main/kotlin/com/staqapp/feature/settings/
│       │   ├── ui/SettingsScreen.kt
│       │   └── data/PreferencesRepository.kt
│       └── build.gradle.kts
│
├── core/                                  # Core/shared modules
│   ├── designsystem/                     # Material 3 theming, common components
│   │   ├── src/main/kotlin/com/staqapp/core/designsystem/
│   │   │   ├── theme/
│   │   │   │   ├── Theme.kt
│   │   │   │   ├── Color.kt
│   │   │   │   └── Typography.kt
│   │   │   └── component/
│   │   │       ├── StaqButton.kt
│   │   │       ├── StaqCard.kt
│   │   │       └── CategoryBadge.kt
│   │   └── build.gradle.kts
│   │
│   ├── data/                              # Room database, data models
│   │   ├── src/main/kotlin/com/staqapp/core/data/
│   │   │   ├── database/
│   │   │   │   ├── StaqDatabase.kt
│   │   │   │   ├── dao/
│   │   │   │   │   ├── SavedContentDao.kt
│   │   │   │   │   └── PendingSaveDao.kt
│   │   │   │   └── entity/
│   │   │   │       ├── SavedContentEntity.kt
│   │   │   │       └── PendingSaveEntity.kt
│   │   │   ├── model/                     # Local data models
│   │   │   │   ├── SavedContent.kt
│   │   │   │   ├── Category.kt
│   │   │   │   └── ExtractedData.kt
│   │   │   └── repository/
│   │   │       └── SavedContentRepository.kt
│   │   └── build.gradle.kts
│   │
│   ├── network/                           # Retrofit/Ktor clients
│   │   ├── src/main/kotlin/com/staqapp/core/network/
│   │   │   ├── api/
│   │   │   │   ├── ExtractApi.kt
│   │   │   │   └── SseClient.kt
│   │   │   ├── model/                     # API response models
│   │   │   │   └── ExtractResponse.kt
│   │   │   └── di/
│   │   │       └── NetworkModule.kt
│   │   └── build.gradle.kts
│   │
│   ├── domain/                            # Use cases, domain models
│   │   ├── src/main/kotlin/com/staqapp/core/domain/
│   │   │   ├── usecase/
│   │   │   │   ├── SaveContentUseCase.kt
│   │   │   │   ├── CategorizeContentUseCase.kt
│   │   │   │   └── SearchContentUseCase.kt
│   │   │   └── model/
│   │   │       └── ... (domain entities)
│   │   └── build.gradle.kts
│   │
│   └── common/                            # Utilities, extensions
│       ├── src/main/kotlin/com/staqapp/core/common/
│       │   ├── util/
│       │   │   ├── UrlNormalizer.kt
│       │   │   ├── PlatformDetector.kt
│       │   │   └── Extensions.kt
│       │   └── analytics/
│       │       └── AnalyticsTracker.kt
│       └── build.gradle.kts
│
├── shared/                                # KMP shared module (consumed from iOS + Android)
│   ├── src/
│   │   ├── commonMain/kotlin/com/staqapp/shared/
│   │   │   ├── network/
│   │   │   │   ├── ApiClient.kt           # Ktor HTTP client
│   │   │   │   └── UrlValidator.kt
│   │   │   ├── model/
│   │   │   │   ├── SavedItem.kt           # Shared data models
│   │   │   │   └── Category.kt
│   │   │   ├── classification/
│   │   │   │   ├── CategoryKeywords.kt    # Multilingual dictionaries
│   │   │   │   └── RuleBasedClassifier.kt
│   │   │   └── parsing/
│   │   │       ├── UrlParser.kt
│   │   │       └── MetadataExtractor.kt
│   │   ├── androidMain/kotlin/            # Android-specific implementations
│   │   └── iosMain/kotlin/                # iOS-specific (Kotlin/Native)
│   └── build.gradle.kts
│
├── buildSrc/                              # Convention plugins
│   ├── src/main/kotlin/
│   │   ├── AndroidFeatureConventionPlugin.kt
│   │   ├── AndroidLibraryConventionPlugin.kt
│   │   └── KotlinMultiplatformConventionPlugin.kt
│   └── build.gradle.kts
│
├── gradle/
│   └── libs.versions.toml                # Centralized dependency versions
│
├── build.gradle.kts                       # Root build config
└── settings.gradle.kts                    # Module includes

```

### 1.2 Convention Plugins

**Why:** Eliminates boilerplate across 10+ Gradle modules. Each module type (feature, library, KMP) inherits common config via convention plugins.

**`buildSrc/src/main/kotlin/AndroidFeatureConventionPlugin.kt`:**
```kotlin
class AndroidFeatureConventionPlugin : Plugin<Project> {
    override fun apply(target: Project) {
        with(target) {
            pluginManager.apply {
                apply("com.android.library")
                apply("org.jetbrains.kotlin.android")
                apply("com.google.devtools.ksp")
                apply("dagger.hilt.android.plugin")
            }

            extensions.configure<LibraryExtension> {
                compileSdk = 35
                defaultConfig {
                    minSdk = 26
                    testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
                }
                buildFeatures {
                    compose = true
                }
                composeOptions {
                    kotlinCompilerExtensionVersion = libs.findVersion("androidx-compose-compiler").get().toString()
                }
            }

            dependencies {
                // All feature modules depend on designsystem, domain, common
                add("implementation", project(":core:designsystem"))
                add("implementation", project(":core:domain"))
                add("implementation", project(":core:common"))

                // Hilt
                add("implementation", libs.findLibrary("hilt-android").get())
                add("ksp", libs.findLibrary("hilt-compiler").get())

                // Compose
                val composeBom = libs.findLibrary("androidx-compose-bom").get()
                add("implementation", platform(composeBom))
                add("implementation", libs.findLibrary("androidx-ui").get())
                add("implementation", libs.findLibrary("androidx-material3").get())
                add("implementation", libs.findLibrary("androidx-ui-tooling-preview").get())
            }
        }
    }
}
```

**Usage:** In `feature/library/build.gradle.kts`:
```kotlin
plugins {
    id("staqer.android.feature")
}

dependencies {
    implementation(project(":core:data"))
    // Feature-specific deps...
}
```

---

## 2. App Architecture

### 2.1 MVVM + MVI Hybrid

Staqer uses **MVVM (Model-View-ViewModel)** for screen-level state management, with **MVI (Model-View-Intent)** principles for unidirectional data flow and state immutability.

**Key Principles:**
- **Single source of truth:** UI state is a single immutable data class (`UiState`)
- **Unidirectional data flow:** User actions → ViewModel → State update → UI recomposition
- **ViewModel doesn't know about View:** No Android framework dependencies in ViewModels
- **Repository pattern:** ViewModels interact with Repositories, not DAOs or APIs directly

**Example: LibraryViewModel (F03)**

```kotlin
// UiState definition
data class LibraryUiState(
    val saves: List<SavedContent> = emptyList(),
    val activeFilter: Category? = null,
    val sortOrder: SortOrder = SortOrder.MOST_RECENT,
    val viewMode: ViewMode = ViewMode.UNIFORM_GRID,
    val isLoading: Boolean = false,
    val error: String? = null
) {
    val filteredSaves: List<SavedContent>
        get() = if (activeFilter != null) {
            saves.filter { it.category == activeFilter }
        } else saves
}

sealed class LibraryAction {
    data class FilterByCategory(val category: Category?) : LibraryAction()
    data class ChangeSort(val order: SortOrder) : LibraryAction()
    data class ToggleViewMode(val mode: ViewMode) : LibraryAction()
    data class DeleteSave(val id: String) : LibraryAction()
    data class Search(val query: String) : LibraryAction()
}

@HiltViewModel
class LibraryViewModel @Inject constructor(
    private val savedContentRepository: SavedContentRepository,
    private val searchUseCase: SearchContentUseCase,
    private val preferencesRepository: PreferencesRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow(LibraryUiState())
    val uiState: StateFlow<LibraryUiState> = _uiState.asStateFlow()

    init {
        // Reactive subscription to Room Flow
        viewModelScope.launch {
            combine(
                savedContentRepository.getAllSaves(),
                preferencesRepository.sortOrder,
                preferencesRepository.viewMode
            ) { saves, sort, viewMode ->
                _uiState.update { it.copy(
                    saves = saves.sortedBy(sort),
                    sortOrder = sort,
                    viewMode = viewMode,
                    isLoading = false
                )}
            }.collect()
        }
    }

    fun handle(action: LibraryAction) {
        when (action) {
            is LibraryAction.FilterByCategory -> {
                _uiState.update { it.copy(activeFilter = action.category) }
            }
            is LibraryAction.ChangeSort -> {
                viewModelScope.launch {
                    preferencesRepository.setSortOrder(action.order)
                }
            }
            is LibraryAction.ToggleViewMode -> {
                viewModelScope.launch {
                    preferencesRepository.setViewMode(action.mode)
                }
            }
            is LibraryAction.DeleteSave -> {
                viewModelScope.launch {
                    savedContentRepository.deleteSave(action.id)
                }
            }
            is LibraryAction.Search -> {
                viewModelScope.launch {
                    _uiState.update { it.copy(isLoading = true) }
                    val results = searchUseCase(action.query)
                    _uiState.update { it.copy(saves = results, isLoading = false) }
                }
            }
        }
    }
}
```

**UI Layer (Composable):**
```kotlin
@Composable
fun LibraryScreen(
    viewModel: LibraryViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()

    Scaffold(
        topBar = { LibraryTopBar() },
        floatingActionButton = { QuickSaveFab() },
        bottomBar = { StaqNavBar() }
    ) { paddingValues ->
        when {
            uiState.isLoading -> LoadingIndicator()
            uiState.error != null -> ErrorState(uiState.error!!)
            else -> {
                Column(Modifier.padding(paddingValues)) {
                    SearchBar(onSearch = { viewModel.handle(LibraryAction.Search(it)) })
                    CategoryFilterBar(
                        activeFilter = uiState.activeFilter,
                        onFilterChange = { viewModel.handle(LibraryAction.FilterByCategory(it)) }
                    )
                    when (uiState.viewMode) {
                        ViewMode.UNIFORM_GRID -> UniformGridView(
                            saves = uiState.filteredSaves,
                            onDelete = { viewModel.handle(LibraryAction.DeleteSave(it)) }
                        )
                        ViewMode.STAGGERED_GRID -> StaggeredGridView(uiState.filteredSaves)
                        ViewMode.LIST -> ListView(uiState.filteredSaves)
                    }
                }
            }
        }
    }
}
```

### 2.2 Dependency Injection (Hilt)

**Hilt** is the recommended DI framework for Android. It generates compile-time bindings and integrates seamlessly with Jetpack (ViewModels, WorkManager, etc.).

**Setup: `app/src/main/kotlin/com/staqapp/StaqApplication.kt`**
```kotlin
@HiltAndroidApp
class StaqApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        if (BuildConfig.DEBUG) {
            Timber.plant(Timber.DebugTree())
        }
    }
}
```

**Repository Module:**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DataModule {
    @Provides
    @Singleton
    fun provideStaqDatabase(@ApplicationContext context: Context): StaqDatabase {
        return Room.databaseBuilder(
            context,
            StaqDatabase::class.java,
            "staq_database"
        )
            .setJournalMode(RoomDatabase.JournalMode.WRITE_AHEAD_LOGGING)
            .addMigrations(/* migrations */)
            .build()
    }

    @Provides
    @Singleton
    fun provideSavedContentDao(db: StaqDatabase): SavedContentDao = db.savedContentDao()

    @Provides
    @Singleton
    fun provideSavedContentRepository(
        dao: SavedContentDao,
        kmpShared: SharedRepository
    ): SavedContentRepository = SavedContentRepositoryImpl(dao, kmpShared)
}
```

**ViewModel Injection:**
```kotlin
@HiltViewModel
class LibraryViewModel @Inject constructor(
    private val repository: SavedContentRepository
) : ViewModel() { /* ... */ }
```

### 2.3 Navigation (Jetpack Compose Navigation)

**Single-Activity architecture** with Compose Navigation. All screens are Composables in a NavHost.

**`app/src/main/kotlin/com/staqapp/navigation/NavGraph.kt`:**
```kotlin
@Composable
fun StaqNavGraph(
    navController: NavHostController = rememberNavController(),
    startDestination: String = Screen.Library.route
) {
    NavHost(
        navController = navController,
        startDestination = startDestination
    ) {
        composable(route = Screen.Library.route) {
            LibraryScreen(
                onCardClick = { cardId ->
                    navController.navigate(Screen.CardDetail.createRoute(cardId))
                }
            )
        }

        composable(
            route = Screen.CardDetail.route,
            arguments = listOf(navArgument("cardId") { type = NavType.StringType })
        ) { backStackEntry ->
            val cardId = backStackEntry.arguments?.getString("cardId")!!
            CardDetailScreen(
                cardId = cardId,
                onBack = { navController.popBackStack() }
            )
        }

        composable(route = Screen.Collections.route) {
            CollectionsScreen()
        }

        composable(route = Screen.Settings.route) {
            SettingsScreen()
        }
    }
}

sealed class Screen(val route: String) {
    object Library : Screen("library")
    object CardDetail : Screen("card/{cardId}") {
        fun createRoute(cardId: String) = "card/$cardId"
    }
    object Collections : Screen("collections")
    object Settings : Screen("settings")
}
```

**Deep Link Support (from widgets, notifications):**
```kotlin
// AndroidManifest.xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:scheme="staq"
            android:host="library" />
        <data
            android:scheme="staq"
            android:host="card"
            android:pathPrefix="/" />
    </intent-filter>
</activity>
```

```kotlin
// MainActivity deep link handler
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent {
        StaqTheme {
            val navController = rememberNavController()
            
            // Handle deep link
            intent?.data?.let { uri ->
                when (uri.host) {
                    "library" -> {
                        val category = uri.getQueryParameter("category")
                        // Navigate to library with filter
                    }
                    "card" -> {
                        val cardId = uri.lastPathSegment
                        navController.navigate(Screen.CardDetail.createRoute(cardId!!))
                    }
                }
            }

            StaqNavGraph(navController)
        }
    }
}
```

---

## 3. Share/Intent Architecture

### 3.1 Share Target (ShareReceiverActivity)

**Implementation: `feature/share/src/main/kotlin/.../ShareReceiverActivity.kt`**

```kotlin
@AndroidEntryPoint
class ShareReceiverActivity : ComponentActivity() {

    @Inject lateinit var pendingSaveRepository: PendingSaveRepository
    @Inject lateinit var urlNormalizer: UrlNormalizer
    @Inject lateinit var platformDetector: PlatformDetector

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Extract URL from intent
        val intentReader = ShareCompat.IntentReader(this)
        if (!intentReader.isShareIntent) {
            finish()
            return
        }

        val sharedText = intentReader.text?.toString()
        val subject = intentReader.subject  // Often contains page title
        val mimeType = intentReader.type

        val url = extractUrlFromText(sharedText)

        if (url == null) {
            // Handle non-URL share (show error state)
            setContent {
                StaqTheme {
                    ShareErrorSheet(
                        message = "That doesn't look like a link. Try sharing from Instagram, TikTok, or YouTube.",
                        onDismiss = { finish() }
                    )
                }
            }
            return
        }

        setContent {
            StaqTheme {
                ShareConfirmationSheet(
                    url = url,
                    platform = platformDetector.detect(url),
                    onConfirm = { note, categoryOverride ->
                        lifecycleScope.launch {
                            val normalizedUrl = urlNormalizer.normalize(url)
                            val pendingSave = PendingSave(
                                id = UUID.randomUUID().toString(),
                                url = normalizedUrl,
                                rawUrl = url,
                                platform = platformDetector.detect(url),
                                captionText = subject,
                                userNote = note,
                                userCategory = categoryOverride,
                                status = SaveStatus.PENDING,
                                createdAt = Clock.System.now(),
                                offlineSave = !isNetworkAvailable()
                            )
                            pendingSaveRepository.insert(pendingSave)

                            // Enqueue WorkManager job
                            enqueuePendingSaveProcessing(applicationContext)

                            // Show toast
                            Toast.makeText(this@ShareReceiverActivity, "Saved to Staq!", Toast.LENGTH_SHORT).show()

                            finish()
                        }
                    },
                    onDismiss = { finish() }
                )
            }
        }
    }

    private fun extractUrlFromText(text: String?): String? {
        if (text.isNullOrBlank()) return null
        val urlPattern = Regex("""https?://[^\s]+""")
        return urlPattern.find(text)?.value
    }
}
```

**ShareConfirmationSheet (Jetpack Compose):**

```kotlin
@Composable
fun ShareConfirmationSheet(
    url: String,
    platform: Platform,
    onConfirm: (note: String?, categoryOverride: Category?) -> Unit,
    onDismiss: () -> Unit
) {
    var note by remember { mutableStateOf("") }
    var selectedCategory by remember { mutableStateOf<Category?>(null) }

    ModalBottomSheet(
        onDismissRequest = onDismiss,
        sheetState = rememberModalBottomSheetState()
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        ) {
            // Thumbnail preview
            AsyncImage(
                model = getThumbnailUrl(url),
                contentDescription = null,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(120.dp)
                    .clip(RoundedCornerShape(12.dp))
            )

            Spacer(modifier = Modifier.height(16.dp))

            Text(
                text = "✅ Saved to Staq!",
                style = MaterialTheme.typography.headlineSmall
            )

            Spacer(modifier = Modifier.height(8.dp))

            // Optional note field
            OutlinedTextField(
                value = note,
                onValueChange = { note = it },
                label = { Text("Add a note...") },
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(modifier = Modifier.height(16.dp))

            // Optional category chips (progressive disclosure: only show after 10 saves)
            // ...

            Spacer(modifier = Modifier.height(16.dp))

            Button(
                onClick = {
                    onConfirm(note.ifBlank { null }, selectedCategory)
                },
                modifier = Modifier.fillMaxWidth()
            ) {
                Text("Done")
            }
        }
    }
}
```

**AndroidManifest.xml:**

```xml
<activity
    android:name=".ShareReceiverActivity"
    android:theme="@android:style/Theme.Translucent.NoTitleBar"
    android:excludeFromRecents="true"
    android:taskAffinity=""
    android:autoRemoveFromRecents="true"
    android:exported="true">
    
    <!-- Text/plain (URLs from Instagram, TikTok, YouTube) -->
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    
    <!-- Text/* (handles text/html from browsers) -->
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/*" />
    </intent-filter>
    
    <!-- Image/* (handles screenshot shares) -->
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```

### 3.2 WorkManager Background Processing

**`feature/share/src/main/kotlin/.../worker/PendingSaveWorker.kt`:**

```kotlin
@HiltWorker
class PendingSaveWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted workerParams: WorkerParameters,
    private val pendingSaveRepository: PendingSaveRepository,
    private val categorizeUseCase: CategorizeContentUseCase,
    private val savedContentRepository: SavedContentRepository
) : CoroutineWorker(appContext, workerParams) {

    override suspend fun doWork(): Result {
        return try {
            val pendingSaves = pendingSaveRepository.getAllPending()

            pendingSaves.forEach { pending ->
                try {
                    // Run classification (Tier 1 → 2.5 → 2 fallback)
                    val classification = categorizeUseCase(
                        caption = pending.captionText ?: "",
                        url = pending.url,
                        platform = pending.platform
                    )

                    // Convert to SavedContent
                    val saved = SavedContent(
                        id = pending.id,
                        url = pending.url,
                        platform = pending.platform,
                        category = classification.category,
                        categoryConfidence = classification.confidence,
                        richnessScore = classification.richnessScore,
                        extractedData = classification.extractedData,
                        userNote = pending.userNote,
                        userCategoryOverride = pending.userCategory,
                        createdAt = pending.createdAt,
                        processedAt = Clock.System.now()
                    )

                    savedContentRepository.insert(saved)
                    pendingSaveRepository.delete(pending.id)

                } catch (e: Exception) {
                    Timber.e(e, "Failed to process pending save: ${pending.id}")
                    // Leave in pending state for retry
                }
            }

            Result.success()
        } catch (e: Exception) {
            Timber.e(e, "PendingSaveWorker failed")
            Result.retry()
        }
    }
}
```

**Enqueue WorkManager Job:**

```kotlin
fun enqueuePendingSaveProcessing(context: Context) {
    val constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()

    val workRequest = OneTimeWorkRequestBuilder<PendingSaveWorker>()
        .setConstraints(constraints)
        .setBackoffCriteria(
            BackoffPolicy.EXPONENTIAL,
            WorkRequest.MIN_BACKOFF_MILLIS,
            TimeUnit.MILLISECONDS
        )
        .build()

    WorkManager.getInstance(context).enqueueUniqueWork(
        "process_pending_saves",
        ExistingWorkPolicy.KEEP,  // Don't duplicate if already enqueued
        workRequest
    )
}
```

---

## 4. Data Layer

### 4.1 Room Database Schema

**`core/data/src/main/kotlin/.../database/StaqDatabase.kt`:**

```kotlin
@Database(
    entities = [
        SavedContentEntity::class,
        PendingSaveEntity::class,
        CategoryEntity::class,
        CollectionEntity::class
    ],
    version = 1,
    exportSchema = true
)
@TypeConverters(Converters::class)
abstract class StaqDatabase : RoomDatabase() {
    abstract fun savedContentDao(): SavedContentDao
    abstract fun pendingSaveDao(): PendingSaveDao
    abstract fun categoryDao(): CategoryDao
    abstract fun collectionDao(): CollectionDao
}
```

**SavedContentEntity:**

```kotlin
@Entity(
    tableName = "saved_content",
    indices = [
        Index(value = ["url"], unique = true),
        Index(value = ["category"]),
        Index(value = ["created_at"]),
        Index(value = ["platform"])
    ]
)
data class SavedContentEntity(
    @PrimaryKey val id: String,
    val url: String,
    val platform: String,  // "instagram", "tiktok", "youtube"
    val captionText: String?,
    val transcriptText: String?,
    val thumbnailLocalPath: String?,
    val thumbnailUrl: String?,
    val thumbnailAspectRatio: Double?,
    val category: String,
    val categoryConfidence: Double,
    val richnessScore: Double,
    val extractedDataJson: String,  // JSON stored as String, parsed on read
    val userNote: String?,
    val userCategoryOverride: String?,
    val collectionIds: String?,  // Comma-separated UUIDs (or use a junction table)
    val creatorHandle: String?,
    val createdAt: Long,  // Unix timestamp (Instant.toEpochMilliseconds())
    val processedAt: Long?,
    val status: String  // "pending", "processing", "complete", "failed"
)
```

**Type Converters:**

```kotlin
class Converters {
    @TypeConverter
    fun fromInstant(value: Instant?): Long? = value?.toEpochMilliseconds()

    @TypeConverter
    fun toInstant(value: Long?): Instant? = value?.let { Instant.fromEpochMilliseconds(it) }

    @TypeConverter
    fun fromExtractedData(value: ExtractedData?): String? =
        value?.let { Json.encodeToString(it) }

    @TypeConverter
    fun toExtractedData(value: String?): ExtractedData? =
        value?.let { Json.decodeFromString(it) }
}
```

### 4.2 FTS5 Full-Text Search

Room doesn't natively support FTS5, so we use a raw SQL approach alongside Room.

**Create FTS5 Virtual Table (run on database creation):**

```kotlin
class FtsSetupCallback : RoomDatabase.Callback() {
    override fun onCreate(db: SupportSQLiteDatabase) {
        super.onCreate(db)
        db.execSQL("""
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
        """.trimIndent())

        // Triggers to keep FTS in sync
        db.execSQL("""
            CREATE TRIGGER saved_content_ai AFTER INSERT ON saved_content BEGIN
                INSERT INTO saved_content_fts(rowid, caption, transcript, extracted_fields, user_note, creator_handle)
                VALUES (new.rowid, new.captionText, new.transcriptText, '', new.userNote, new.creatorHandle);
            END;
        """.trimIndent())

        db.execSQL("""
            CREATE TRIGGER saved_content_au AFTER UPDATE ON saved_content BEGIN
                INSERT INTO saved_content_fts(saved_content_fts, rowid, caption, transcript, extracted_fields, user_note, creator_handle)
                VALUES ('delete', old.rowid, old.captionText, old.transcriptText, '', old.userNote, old.creatorHandle);
                INSERT INTO saved_content_fts(rowid, caption, transcript, extracted_fields, user_note, creator_handle)
                VALUES (new.rowid, new.captionText, new.transcriptText, '', new.userNote, new.creatorHandle);
            END;
        """.trimIndent())

        db.execSQL("""
            CREATE TRIGGER saved_content_ad AFTER DELETE ON saved_content BEGIN
                INSERT INTO saved_content_fts(saved_content_fts, rowid, caption, transcript, extracted_fields, user_note, creator_handle)
                VALUES ('delete', old.rowid, old.captionText, old.transcriptText, '', old.userNote, old.creatorHandle);
            END;
        """.trimIndent())
    }
}
```

**Database Initialization:**

```kotlin
@Provides
@Singleton
fun provideStaqDatabase(@ApplicationContext context: Context): StaqDatabase {
    return Room.databaseBuilder(
        context,
        StaqDatabase::class.java,
        "staq_database"
    )
        .setJournalMode(RoomDatabase.JournalMode.WRITE_AHEAD_LOGGING)
        .addCallback(FtsSetupCallback())
        .build()
}
```

**Search Query (via RawQuery):**

```kotlin
@Dao
interface SavedContentDao {
    @RawQuery(observedEntities = [SavedContentEntity::class])
    fun searchFts(query: SimpleSQLiteQuery): Flow<List<SavedContentEntity>>
}

// In Repository
suspend fun search(query: String): List<SavedContent> {
    val ftsQuery = SimpleSQLiteQuery("""
        SELECT sc.* FROM saved_content sc
        JOIN saved_content_fts fts ON sc.rowid = fts.rowid
        WHERE saved_content_fts MATCH ?
        ORDER BY rank
        LIMIT 50
    """.trimIndent(), arrayOf("$query*"))

    return dao.searchFts(ftsQuery).first().map { it.toDomainModel() }
}
```

### 4.3 Migration Strategy

**Version 1 → 2 Example (adding a column):**

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE saved_content ADD COLUMN is_learned INTEGER NOT NULL DEFAULT 0")
    }
}

@Provides
@Singleton
fun provideStaqDatabase(@ApplicationContext context: Context): StaqDatabase {
    return Room.databaseBuilder(/*...*/)
        .addMigrations(MIGRATION_1_2)
        .build()
}
```

**Auto-migrations (Room 2.4+):** For simple schema changes, use `@AutoMigration`:
```kotlin
@Database(
    version = 2,
    autoMigrations = [
        AutoMigration(from = 1, to = 2)
    ]
)
```

### 4.4 Offline-First Architecture

**Principles:**
- All user-facing features work offline
- Cloud sync is additive (Phase 2 scope)
- Never show "No connection" errors for core library operations

**Data Flow:**

```
User Action → ViewModel → Repository → Room (local DB)
                                     ↓
                              (optionally) Network API
                                     ↓
                              Update local DB on response
```

**Example: Delete Save**
```kotlin
// Repository
suspend fun deleteSave(id: String) {
    // Delete locally first (instant UI update)
    dao.delete(id)

    // Sync to server (if user is logged in and online)
    try {
        api.deleteSave(id)
    } catch (e: Exception) {
        // Queue for retry when online
        syncQueue.enqueue(SyncAction.Delete(id))
    }
}
```

---

## 5. AI/ML Layer

### 5.1 Gemini Nano Integration (Tier 1)

**Early Access API Status:** As of Feb 2024, Gemini Nano Prompt API is in **Alpha/Early Access**. Monitor [Google AI Edge SDK releases](https://ai.google.dev/edge) closely.

**Check Availability:**

```kotlin
suspend fun isGeminiNanoAvailable(context: Context): Boolean {
    return try {
        when (AICore.getAvailability(context)) {
            Availability.AVAILABLE -> true
            Availability.UNAVAILABLE_MODEL_NOT_DOWNLOADED -> {
                // Request background download
                AICore.requestModelDownload(context)
                false
            }
            else -> false
        }
    } catch (e: Exception) {
        Timber.w("Gemini Nano check failed: ${e.message}")
        false
    }
}
```

**Classify Content (Tier 1):**

```kotlin
class Tier1Classifier @Inject constructor(
    @ApplicationContext private val context: Context
) {
    private val model by lazy {
        GenerativeModel(modelName = "gemini-nano")
    }

    suspend fun classify(
        caption: String,
        ocrText: String,
        hashtags: List<String>
    ): ClassificationResult? {
        if (!isGeminiNanoAvailable(context)) return null

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
            val classification = Json.decodeFromString<Classification>(response.text)
            ClassificationResult.Success(classification, tier = Tier.TIER_1)
        } catch (e: Exception) {
            Timber.e(e, "Gemini Nano inference failed")
            null
        }
    }
}
```

**Graceful Fallback Chain:**

```kotlin
class ClassifierTierSelector @Inject constructor(
    private val tier1: Tier1Classifier,
    private val tier25: Tier25Classifier,
    private val tier2: Tier2Classifier
) {
    suspend fun classify(input: ClassificationInput): ClassificationResult {
        // Try Tier 1 (Gemini Nano)
        tier1.classify(input.caption, input.ocrText, input.hashtags)?.let { return it }

        // Try Tier 2.5 (bundled TFLite model)
        tier25.classify(input.caption, input.hashtags)?.let { return it }

        // Fallback to Tier 2 (rule-based)
        return tier2.classify(input.caption, input.hashtags)
    }
}
```

### 5.2 ML Kit OCR Pipeline

**`feature/classification/src/main/kotlin/.../OcrEngine.kt`:**

```kotlin
class OcrEngine @Inject constructor() {
    private val latinRecognizer = TextRecognition.getClient(TextRecognizerOptions.DEFAULT_OPTIONS)
    private val devanagariRecognizer = TextRecognition.getClient(
        DevanagariTextRecognizerOptions.Builder().build()
    )

    suspend fun extractText(bitmap: Bitmap): String {
        // Downscale to 720p for performance
        val resized = bitmap.scaleToFit(maxDimension = 720)
        val inputImage = InputImage.fromBitmap(resized, 0)

        val latinResult = latinRecognizer.process(inputImage).await()
        val devanagariResult = devanagariRecognizer.process(inputImage).await()

        // Use whichever extracted more text (likely the correct script)
        val rawText = if (devanagariResult.text.length > latinResult.text.length) {
            "${devanagariResult.text} ${latinResult.text}".trim()
        } else {
            latinResult.text
        }

        return cleanOCRText(rawText)
    }

    private fun cleanOCRText(raw: String): String {
        var text = raw

        // Remove social media handles, "follow for more", etc.
        val handlePatterns = listOf(
            Regex("""@[\w.]+"""),
            Regex("""(?i)ig:\s*@?[\w.]+"""),
            Regex("""(?i)follow\s+(me\s+)?@[\w.]+"""),
            Regex("""(?i)link\s+in\s+bio""")
        )

        handlePatterns.forEach { pattern ->
            text = text.replace(pattern, " ")
        }

        // Collapse whitespace
        return text.replace(Regex("""\s+"""), " ").trim()
    }
}
```

### 5.3 TFLite Bundled Model (Tier 2.5)

**Model Training:** Fine-tune a DistilBERT-tiny or MobileBERT model on 10K+ labeled captions. Export to TFLite.

**Model Location:** `core/classification/src/main/assets/category_classifier.tflite`

**Inference:**

```kotlin
class Tier25Classifier @Inject constructor(
    @ApplicationContext private val context: Context
) {
    private val interpreter: Interpreter by lazy {
        val modelFile = loadModelFile(context, "category_classifier.tflite")
        Interpreter(modelFile)
    }

    private fun loadModelFile(context: Context, fileName: String): ByteBuffer {
        val assetManager = context.assets
        val fileDescriptor = assetManager.openFd(fileName)
        val inputStream = FileInputStream(fileDescriptor.fileDescriptor)
        val fileChannel = inputStream.channel
        val startOffset = fileDescriptor.startOffset
        val declaredLength = fileDescriptor.declaredLength
        return fileChannel.map(FileChannel.MapMode.READ_ONLY, startOffset, declaredLength)
    }

    fun classify(caption: String, hashtags: List<String>): ClassificationResult? {
        val input = tokenize(caption, hashtags)
        val output = Array(1) { FloatArray(NUM_CATEGORIES) }

        interpreter.run(input, output)

        val probabilities = output[0]
        val maxIndex = probabilities.indices.maxByOrNull { probabilities[it] } ?: 0
        val category = CATEGORY_LABELS[maxIndex]
        val confidence = probabilities[maxIndex].toDouble()

        return if (confidence > 0.5) {
            ClassificationResult.Success(
                Classification(category, confidence, title = extractTitle(caption)),
                tier = Tier.TIER_2_5
            )
        } else null
    }

    private fun tokenize(caption: String, hashtags: List<String>): FloatArray {
        // Implement tokenization matching training pipeline
        // ...
        return FloatArray(128) // Max sequence length
    }

    companion object {
        const val NUM_CATEGORIES = 14
        val CATEGORY_LABELS = arrayOf(
            "recipe", "travel", "product", "tech_gadgets", "fitness", 
            "beauty", "fashion", "diy", "finance", "entertainment", 
            "education", "parenting", "inspiration", "general"
        )
    }
}
```

### 5.4 Tier 2: Rule-Based Classifier (Fallback)

**Multilingual Keyword Dictionaries:**

```kotlin
// Shared via KMP module: shared/src/commonMain/kotlin/.../CategoryKeywords.kt
object CategoryKeywords {
    val recipeKeywords = mapOf(
        // English
        "recipe" to 3, "ingredients" to 3, "cook" to 2, "bake" to 2,
        "tablespoon" to 3, "teaspoon" to 3, "cup" to 2, "preheat" to 3,

        // Hindi transliterated
        "sabzi" to 2, "masala" to 2, "paneer" to 2, "dal" to 2,
        "biryani" to 2, "chutney" to 2, "roti" to 2, "paratha" to 2,

        // Portuguese (Brazil)
        "receita" to 3, "ingredientes" to 3, "cozinhar" to 2,

        // Arabic
        "وصفة" to 3, "مكونات" to 3, "طبخ" to 2,

        // Bahasa Indonesia
        "resep" to 3, "bahan" to 3, "masak" to 2
    )

    // ... (define for all categories)
}
```

**Tier 2 Implementation:**

```kotlin
class Tier2Classifier @Inject constructor() {
    fun classify(caption: String, hashtags: List<String>): ClassificationResult {
        val scores = mutableMapOf<String, Int>()

        // Score hashtags
        hashtags.forEach { tag ->
            CategoryKeywords.allCategories.forEach { (category, keywords) ->
                if (tag.lowercase() in keywords.keys) {
                    scores[category] = (scores[category] ?: 0) + keywords[tag.lowercase()]!!
                }
            }
        }

        // Score caption tokens
        val tokens = caption.lowercase().split(Regex("""\s+"""))
        tokens.forEach { token ->
            CategoryKeywords.allCategories.forEach { (category, keywords) ->
                if (token in keywords.keys) {
                    scores[category] = (scores[category] ?: 0) + keywords[token]!!
                }
            }
        }

        val topCategory = scores.maxByOrNull { it.value }?.key ?: "general"
        val confidence = (scores[topCategory] ?: 0).toDouble() / scores.values.sum().coerceAtLeast(1)

        return ClassificationResult.Success(
            Classification(topCategory, confidence, title = extractTitle(caption)),
            tier = Tier.TIER_2
        )
    }
}
```

### 5.5 ML Kit Speech Recognition (Deep Extract)

**`feature/deepextract/src/main/kotlin/.../TranscriptionEngine.kt`:**

```kotlin
class TranscriptionEngine @Inject constructor() {
    private val recognizer = SpeechRecognition.getClient(
        SpeechRecognizerOptions.Builder()
            .setResultType(SpeechRecognizerOptions.RESULT_TYPE_FINAL)
            .build()
    )

    suspend fun transcribe(audioFile: File): String {
        // For audio > 1 minute, chunk into 55-second segments
        val chunks = chunkAudio(audioFile, chunkLengthMs = 55_000)
        
        return chunks.map { chunk ->
            val uri = Uri.fromFile(chunk)
            recognizer.recognizeAudio(uri).await().text
        }.joinToString(" ")
    }

    private fun chunkAudio(file: File, chunkLengthMs: Long): List<File> {
        // MediaExtractor + MediaMuxer to split audio into chunks
        // ...
        return emptyList() // Placeholder
    }
}
```

---

## 6. Network Layer

### 6.1 Retrofit / OkHttp Setup

**`core/network/src/main/kotlin/.../di/NetworkModule.kt`:**

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = if (BuildConfig.DEBUG) HttpLoggingInterceptor.Level.BODY
                        else HttpLoggingInterceptor.Level.NONE
            })
            .addInterceptor { chain ->
                val request = chain.request().newBuilder()
                    .addHeader("User-Agent", "Staq Android/${BuildConfig.VERSION_NAME}")
                    .build()
                chain.proceed(request)
            }
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.API_BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
            .build()
    }

    @Provides
    @Singleton
    fun provideExtractApi(retrofit: Retrofit): ExtractApi {
        return retrofit.create(ExtractApi::class.java)
    }
}
```

### 6.2 SSE Client (Deep Extract)

**OkHttp SSE Support via `okhttp-sse`:**

```kotlin
dependencies {
    implementation("com.squareup.okhttp3:okhttp-sse:4.12.0")
}
```

**`core/network/src/main/kotlin/.../SseClient.kt`:**

```kotlin
class SseClient @Inject constructor(
    private val okHttpClient: OkHttpClient
) {
    fun streamExtractProgress(
        url: String,
        onProgress: (ExtractStage) -> Unit,
        onComplete: (ExtractResult) -> Unit,
        onError: (Throwable) -> Unit
    ): EventSource {
        val request = Request.Builder()
            .url(url)
            .header("Accept", "text/event-stream")
            .build()

        val eventSource = EventSources.createFactory(okHttpClient)
            .newEventSource(request, object : EventSourceListener() {
                override fun onEvent(
                    eventSource: EventSource,
                    id: String?,
                    type: String?,
                    data: String
                ) {
                    when (type) {
                        "progress" -> {
                            val stage = Json.decodeFromString<ExtractStage>(data)
                            onProgress(stage)
                        }
                        "complete" -> {
                            val result = Json.decodeFromString<ExtractResult>(data)
                            onComplete(result)
                            eventSource.cancel()
                        }
                    }
                }

                override fun onFailure(
                    eventSource: EventSource,
                    t: Throwable?,
                    response: Response?
                ) {
                    onError(t ?: Exception("SSE connection failed"))
                }
            })

        return eventSource
    }
}
```

### 6.3 API Definitions

**`core/network/src/main/kotlin/.../api/ExtractApi.kt`:**

```kotlin
interface ExtractApi {
    @POST("api/v1/extract")
    suspend fun startExtract(
        @Body request: ExtractRequest
    ): ExtractJobResponse

    @GET("api/v1/extract/{jobId}/status")
    suspend fun getExtractStatus(
        @Path("jobId") jobId: String
    ): ExtractStatusResponse
}

@Serializable
data class ExtractRequest(
    val url: String,
    val platform: String,
    val saveId: String,
    val deviceCanTranscribe: Boolean
)

@Serializable
data class ExtractJobResponse(
    val jobId: String,
    val sseUrl: String
)
```

---

## 7. KMP Integration

### 7.1 Consuming KMP Shared Module

The `shared` module is a Kotlin Multiplatform module that compiles to:
- **Android:** JVM bytecode (consumed like any other Gradle module)
- **iOS:** Kotlin/Native framework (consumed as a CocoaPods pod or XCFramework)

**Android Consumption (trivial):**

In `settings.gradle.kts`:
```kotlin
include(":shared")
```

In `app/build.gradle.kts`:
```kotlin
dependencies {
    implementation(project(":shared"))
}
```

**Usage:**
```kotlin
import com.staqapp.shared.network.ApiClient
import com.staqapp.shared.parsing.UrlParser

// Use KMP code directly
val apiClient = ApiClient()
val parsedUrl = UrlParser.parse("https://instagram.com/reel/ABC123")
```

### 7.2 KMP Architecture

**What's in KMP:**
- Networking (Ktor HTTP client)
- Data models (SavedItem, Category, ExtractedData)
- Classification rules (keyword dictionaries, category definitions)
- URL parsing & validation
- Rule-based classifier (Tier 2)

**What's Native (Android):**
- UI (Jetpack Compose)
- On-device AI (ML Kit, Gemini Nano)
- Room database (Android-specific persistence)
- Share Extension (Intent handling)
- WorkManager (background processing)

**shared/build.gradle.kts:**

```kotlin
plugins {
    kotlin("multiplatform")
    kotlin("plugin.serialization")
}

kotlin {
    androidTarget()
    
    iosX64()
    iosArm64()
    iosSimulatorArm64()

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation("io.ktor:ktor-client-core:2.3.7")
                implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.2")
            }
        }

        val androidMain by getting {
            dependencies {
                implementation("io.ktor:ktor-client-okhttp:2.3.7")
            }
        }

        val iosMain by creating {
            dependsOn(commonMain)
            dependencies {
                implementation("io.ktor:ktor-client-darwin:2.3.7")
            }
        }
    }
}
```

---

## 8. Key Libraries & Dependencies

### 8.1 Core Dependencies

**`gradle/libs.versions.toml`:**

```toml
[versions]
kotlin = "1.9.22"
compose-compiler = "1.5.8"
compose-bom = "2024.01.00"
hilt = "2.50"
room = "2.6.1"
retrofit = "2.9.0"
okhttp = "4.12.0"
coil = "2.5.0"
accompanist = "0.34.0"
work = "2.9.0"
mlkit = "17.0.0"

[libraries]
# Compose
androidx-compose-bom = { group = "androidx.compose", name = "compose-bom", version.ref = "compose-bom" }
androidx-ui = { group = "androidx.compose.ui", name = "ui" }
androidx-ui-graphics = { group = "androidx.compose.ui", name = "ui-graphics" }
androidx-ui-tooling-preview = { group = "androidx.compose.ui", name = "ui-tooling-preview" }
androidx-material3 = { group = "androidx.compose.material3", name = "material3" }
androidx-navigation-compose = { group = "androidx.navigation", name = "navigation-compose", version = "2.7.6" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }
hilt-navigation-compose = { group = "androidx.hilt", name = "hilt-navigation-compose", version = "1.1.0" }

# Room
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
androidx-room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }

# Network
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
okhttp = { group = "com.squareup.okhttp3", name = "okhttp", version.ref = "okhttp" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
okhttp-sse = { group = "com.squareup.okhttp3", name = "okhttp-sse", version.ref = "okhttp" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version = "1.6.2" }

# Image loading
coil-compose = { group = "io.coil-kt", name = "coil-compose", version.ref = "coil" }

# WorkManager
androidx-work-runtime-ktx = { group = "androidx.work", name = "work-runtime-ktx", version.ref = "work" }
androidx-hilt-work = { group = "androidx.hilt", name = "hilt-work", version = "1.1.0" }

# ML Kit
mlkit-text-recognition = { group = "com.google.mlkit", name = "text-recognition", version = "16.0.0" }
mlkit-text-recognition-devanagari = { group = "com.google.mlkit", name = "text-recognition-devanagari", version = "16.0.0" }
mlkit-language-id = { group = "com.google.mlkit", name = "language-id", version = "17.0.5" }

# Gemini Nano (Early Access — version subject to change)
google-ai-client = { group = "com.google.ai.client", name = "generativemodel", version = "0.1.0-alpha" }
google-aicore = { group = "com.google.android.aicore", name = "aicore", version = "0.1.0" }

# TensorFlow Lite
tensorflow-lite = { group = "org.tensorflow", name = "tensorflow-lite", version = "2.14.0" }
tensorflow-lite-support = { group = "org.tensorflow", name = "tensorflow-lite-support", version = "0.4.4" }

# Utilities
timber = { group = "com.jakewharton.timber", name = "timber", version = "5.0.1" }
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version = "1.12.0" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version = "2.7.0" }
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version = "2.7.0" }

# Analytics & Monitoring
firebase-bom = { group = "com.google.firebase", name = "firebase-bom", version = "32.7.1" }
firebase-analytics = { group = "com.google.firebase", name = "firebase-analytics" }
firebase-crashlytics = { group = "com.google.firebase", name = "firebase-crashlytics" }
firebase-perf = { group = "com.google.firebase", name = "firebase-perf" }

# Glance (Widgets)
androidx-glance-appwidget = { group = "androidx.glance", name = "glance-appwidget", version = "1.0.0" }
androidx-glance-material3 = { group = "androidx.glance", name = "glance-material3", version = "1.0.0" }

# Testing
junit = { group = "junit", name = "junit", version = "4.13.2" }
androidx-test-ext-junit = { group = "androidx.test.ext", name = "junit", version = "1.1.5" }
androidx-test-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version = "3.5.1" }
androidx-compose-ui-test-junit4 = { group = "androidx.compose.ui", name = "ui-test-junit4" }
androidx-compose-ui-test-manifest = { group = "androidx.compose.ui", name = "ui-test-manifest" }
androidx-room-testing = { group = "androidx.room", name = "room-testing", version.ref = "room" }
turbine = { group = "app.cash.turbine", name = "turbine", version = "1.0.0" }
kotest-assertions = { group = "io.kotest", name = "kotest-assertions-core", version = "5.8.0" }
mockk = { group = "io.mockk", name = "mockk-android", version = "1.13.9" }
robolectric = { group = "org.robolectric", name = "robolectric", version = "4.11.1" }

# LeakCanary
leakcanary = { group = "com.squareup.leakcanary", name = "leakcanary-android", version = "2.12" }

[plugins]
android-application = { id = "com.android.application", version = "8.2.2" }
android-library = { id = "com.android.library", version = "8.2.2" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version = "1.9.22-1.0.17" }
google-services = { id = "com.google.gms.google-services", version = "4.4.0" }
firebase-crashlytics-plugin = { id = "com.google.firebase.crashlytics", version = "2.9.9" }
```

### 8.2 Optional / Phase 2 Libraries

- **Amplitude / Mixpanel:** Analytics
- **Sentry:** Crash reporting (alternative to Crashlytics)
- **Detekt:** Static code analysis
- **Spotless:** Code formatting
- **Compose Destinations:** Type-safe Compose navigation (alternative to manual NavHost)

---

## 9. Testing Strategy

### 9.1 Unit Tests (JUnit + Kotest)

**Test ViewModels:**

```kotlin
@ExperimentalCoroutinesApi
class LibraryViewModelTest {
    private val testDispatcher = UnconfinedTestDispatcher()

    @get:Rule
    val mainDispatcherRule = MainDispatcherRule(testDispatcher)

    private lateinit var viewModel: LibraryViewModel
    private val repository = mockk<SavedContentRepository>()

    @Before
    fun setup() {
        viewModel = LibraryViewModel(repository, /* ... */)
    }

    @Test
    fun `filter by category updates state`() = runTest {
        // Given
        val category = Category.RECIPE
        val mockSaves = listOf(/* ... */)
        coEvery { repository.getAllSaves() } returns flowOf(mockSaves)

        // When
        viewModel.handle(LibraryAction.FilterByCategory(category))

        // Then
        assertEquals(category, viewModel.uiState.value.activeFilter)
    }

    @Test
    fun `delete save calls repository`() = runTest {
        // Given
        val saveId = "test-id"
        coEvery { repository.deleteSave(any()) } just Runs

        // When
        viewModel.handle(LibraryAction.DeleteSave(saveId))

        // Then
        coVerify { repository.deleteSave(saveId) }
    }
}
```

**Test Repositories (with Room in-memory DB):**

```kotlin
@RunWith(AndroidJUnit4::class)
class SavedContentRepositoryTest {
    private lateinit var database: StaqDatabase
    private lateinit var dao: SavedContentDao
    private lateinit var repository: SavedContentRepository

    @Before
    fun setup() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        database = Room.inMemoryDatabaseBuilder(context, StaqDatabase::class.java)
            .allowMainThreadQueries()
            .build()
        dao = database.savedContentDao()
        repository = SavedContentRepositoryImpl(dao, /* ... */)
    }

    @After
    fun tearDown() {
        database.close()
    }

    @Test
    fun `insert and retrieve save`() = runTest {
        // Given
        val save = SavedContent(/* ... */)

        // When
        repository.insert(save)
        val result = repository.getSaveById(save.id)

        // Then
        assertEquals(save, result)
    }
}
```

### 9.2 UI Tests (Espresso + Compose Testing)

**Compose UI Testing:**

```kotlin
@RunWith(AndroidJUnit4::class)
class LibraryScreenTest {
    @get:Rule
    val composeTestRule = createAndroidComposeRule<ComponentActivity>()

    @Test
    fun categoryFilterBar_displaysCategories() {
        composeTestRule.setContent {
            StaqTheme {
                CategoryFilterBar(
                    categories = listOf(Category.RECIPE, Category.TRAVEL),
                    activeFilter = null,
                    onFilterChange = {}
                )
            }
        }

        composeTestRule.onNodeWithText("All").assertIsDisplayed()
        composeTestRule.onNodeWithText("Recipe").assertIsDisplayed()
        composeTestRule.onNodeWithText("Travel").assertIsDisplayed()
    }

    @Test
    fun clickingCategoryChip_callsFilterChange() {
        var selectedCategory: Category? = null

        composeTestRule.setContent {
            StaqTheme {
                CategoryFilterBar(
                    categories = listOf(Category.RECIPE),
                    activeFilter = null,
                    onFilterChange = { selectedCategory = it }
                )
            }
        }

        composeTestRule.onNodeWithText("Recipe").performClick()
        assertEquals(Category.RECIPE, selectedCategory)
    }
}
```

### 9.3 Integration Tests (Robolectric)

Robolectric allows testing Android framework code (like Room, WorkManager) in JVM unit tests without an emulator.

```kotlin
@RunWith(RobolectricTestRunner::class)
@Config(sdk = [33])
class DeepExtractWorkerTest {
    private lateinit var context: Context
    private lateinit var worker: DeepExtractWorker

    @Before
    fun setup() {
        context = ApplicationProvider.getApplicationContext()
        val workerParams = TestListenableWorkerBuilder<DeepExtractWorker>(context).build()
        worker = TestListenableWorkerBuilder<DeepExtractWorker>(context)
            .build()
    }

    @Test
    fun `worker processes pending saves`() = runTest {
        // Given
        val pendingSave = PendingSave(/* ... */)
        // Insert into test DB

        // When
        val result = worker.doWork()

        // Then
        assertEquals(ListenableWorker.Result.success(), result)
        // Verify pending save was processed
    }
}
```

### 9.4 Test Coverage Goals

| Layer | Target Coverage |
|-------|----------------|
| ViewModels | > 80% |
| Use Cases | > 85% |
| Repositories | > 75% |
| Data Sources (DAOs) | > 70% |
| UI (Composables) | > 50% (focus on critical paths) |

---

## 10. Build & CI/CD

### 10.1 Gradle Configuration

**Root `build.gradle.kts`:**

```kotlin
buildscript {
    dependencies {
        classpath(libs.google.services)
        classpath(libs.firebase.crashlytics.plugin)
    }
}

plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.android.library) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.serialization) apply false
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.ksp) apply false
}

tasks.register("clean", Delete::class) {
    delete(rootProject.buildDir)
}
```

**App Module `build.gradle.kts`:**

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.serialization)
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
    alias(libs.plugins.google.services)
    alias(libs.plugins.firebase.crashlytics.plugin)
}

android {
    namespace = "com.staqapp"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.staqapp"
        minSdk = 26
        targetSdk = 35
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"

        buildConfigField("String", "API_BASE_URL", "\"https://api.staqapp.com\"")
    }

    buildTypes {
        debug {
            applicationIdSuffix = ".debug"
            isDebuggable = true
            isMinifyEnabled = false
        }

        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("release")
        }
    }

    signingConfigs {
        create("release") {
            storeFile = file("../keystore/staq-release.jks")
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
        freeCompilerArgs += listOf(
            "-opt-in=kotlin.RequiresOptIn",
            "-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi"
        )
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = libs.versions.compose.compiler.get()
    }

    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    implementation(project(":feature:library"))
    implementation(project(":feature:share"))
    implementation(project(":feature:classification"))
    implementation(project(":feature:cards"))
    implementation(project(":feature:deepextract"))
    implementation(project(":feature:settings"))
    implementation(project(":core:designsystem"))
    implementation(project(":core:data"))
    implementation(project(":core:domain"))
    implementation(project(":core:network"))
    implementation(project(":core:common"))
    implementation(project(":shared"))

    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.ui)
    implementation(libs.androidx.material3)
    implementation(libs.androidx.navigation.compose)
    
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)

    implementation(libs.timber)
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)

    implementation(platform(libs.firebase.bom))
    implementation(libs.firebase.analytics)
    implementation(libs.firebase.crashlytics)

    debugImplementation(libs.leakcanary)

    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.test.ext.junit)
    androidTestImplementation(libs.androidx.test.espresso.core)
}
```

### 10.2 ProGuard / R8 Configuration

**`app/proguard-rules.pro`:**

```pro
# Keep data models for serialization
-keep class com.staqapp.core.data.model.** { *; }
-keep class com.staqapp.core.network.model.** { *; }
-keepclassmembers class * implements kotlinx.serialization.SerialStrategy { *; }

# Retrofit
-keepattributes Signature
-keepattributes Exceptions
-keep class retrofit2.** { *; }

# OkHttp
-dontwarn okhttp3.**
-keep class okhttp3.** { *; }

# Hilt
-keep class dagger.hilt.** { *; }
-keep class javax.inject.** { *; }

# Room
-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-dontwarn androidx.room.paging.**

# ML Kit
-keep class com.google.mlkit.** { *; }
-dontwarn com.google.mlkit.**

# Compose
-keep class androidx.compose.** { *; }
-dontwarn androidx.compose.**
```

### 10.3 GitHub Actions CI/CD

**`.github/workflows/android-ci.yml`:**

```yaml
name: Android CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Grant execute permission for gradlew
      run: chmod +x gradlew

    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: |
          ~/.gradle/caches
          ~/.gradle/wrapper
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
        restore-keys: |
          ${{ runner.os }}-gradle-

    - name: Run Detekt
      run: ./gradlew detekt

    - name: Run unit tests
      run: ./gradlew testDebugUnitTest

    - name: Build debug APK
      run: ./gradlew assembleDebug

    - name: Upload APK artifact
      uses: actions/upload-artifact@v3
      with:
        name: app-debug
        path: app/build/outputs/apk/debug/app-debug.apk

  instrumentation-tests:
    runs-on: macos-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Run instrumented tests
      uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: 33
        target: google_apis
        arch: x86_64
        script: ./gradlew connectedDebugAndroidTest
```

**`.github/workflows/release.yml` (Play Store Deployment):**

```yaml
name: Release to Play Store

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Decode keystore
      run: |
        echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > keystore/staq-release.jks

    - name: Build release AAB
      run: ./gradlew bundleRelease
      env:
        KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
        KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
        KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}

    - name: Upload to Play Store (Internal Testing)
      uses: r0adkll/upload-google-play@v1
      with:
        serviceAccountJsonPlainText: ${{ secrets.PLAY_STORE_SERVICE_ACCOUNT_JSON }}
        packageName: com.staqapp
        releaseFiles: app/build/outputs/bundle/release/app-release.aab
        track: internal
        status: completed
```

---

## 11. Device Compatibility

### 11.1 Min SDK & Target SDK

```kotlin
minSdk = 26  // Android 8.0 Oreo (2017)
targetSdk = 35  // Android 15 (2024)
compileSdk = 35
```

**Rationale:**
- **Min SDK 26:** Covers 95%+ of active Android devices. Enables modern APIs (Notification Channels, Autofill, etc.).
- **Target SDK 35:** Required by Play Store for new app submissions (2024 policy).

### 11.2 Gemini Nano Availability Detection

**Supported Devices (as of Feb 2024):**
- Pixel 8, 8 Pro, 8a
- Galaxy S24, S24+, S24 Ultra
- OnePlus 13
- Xiaomi 15
- Honor Magic 7

**Runtime Check:**

```kotlin
suspend fun checkGeminiNanoSupport(context: Context): GeminiNanoStatus {
    return when (AICore.getAvailability(context)) {
        Availability.AVAILABLE -> GeminiNanoStatus.Available
        
        Availability.UNAVAILABLE_MODEL_NOT_DOWNLOADED -> {
            AICore.requestModelDownload(context)
            GeminiNanoStatus.DownloadPending
        }
        
        Availability.UNAVAILABLE_DEVICE_NOT_SUPPORTED ->
            GeminiNanoStatus.DeviceNotSupported
        
        Availability.UNAVAILABLE_USER_NOT_OPTED_IN ->
            GeminiNanoStatus.UserNotOptedIn
        
        else -> GeminiNanoStatus.Unknown
    }
}
```

### 11.3 Graceful Fallback for Mid-Range Devices

**Mid-range device examples:**
- Redmi Note 12/13 series
- Realme 11/12 series
- Samsung Galaxy A34/A54
- Motorola Moto G series

**Fallback Strategy:**
1. Attempt Tier 1 (Gemini Nano) → Unavailable
2. Attempt Tier 2.5 (bundled TFLite model) → Works on all devices
3. Fallback to Tier 2 (rule-based) → Always available

**Memory Constraints:**
- Target peak memory < 300MB for classification
- Use `onTrimMemory()` callbacks to release caches under pressure

```kotlin
override fun onTrimMemory(level: Int) {
    super.onTrimMemory(level)
    when (level) {
        ComponentCallbacks2.TRIM_MEMORY_RUNNING_LOW,
        ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL -> {
            // Clear image caches
            Coil.imageLoader(this).memoryCache?.clear()
            
            // Unload TFLite model if loaded
            tier25Classifier.unloadModel()
        }
    }
}
```

**Battery Optimization:**
- Respect Doze mode — use WorkManager with appropriate constraints
- Show battery warning when < 20% before Deep Extract

---

## 12. Sprint-by-Sprint Breakdown

### Sprint 0 (Week 1-2): Foundation & Setup
**Goal:** Project structure, build config, core modules

- [ ] Create multi-module Gradle project structure
- [ ] Set up convention plugins (feature, library, KMP)
- [ ] Configure Hilt DI in app module
- [ ] Build core/designsystem: Material 3 theme, common components
- [ ] Build core/data: Room database schema, DAOs, entities
- [ ] Set up GitHub Actions CI pipeline (build + unit tests)
- [ ] Initialize KMP shared module with Ktor client
- [ ] Create placeholder screens (Library, Collections, Settings)

**Deliverable:** Buildable app skeleton with navigation between empty screens

---

### Sprint 1 (Week 3-4): F01 — Share Extension
**Goal:** Share target receives URLs, saves to local DB

**Tasks:**
- [ ] Implement ShareReceiverActivity with Intent filters (text/plain, image/*)
- [ ] Build ShareConfirmationSheet (Compose bottom sheet)
- [ ] Extract URL from Intent.EXTRA_TEXT using ShareCompat
- [ ] Build URL normalization (strip tracking params, resolve short URLs)
- [ ] Build PendingSaveRepository + Room entity
- [ ] Implement WorkManager PendingSaveWorker
- [ ] Add URL deduplication check before save
- [ ] Handle offline saves (queue locally, process when online)
- [ ] Build analytics event logging (local file, flushed by app)
- [ ] QA: Test sharing from Instagram, TikTok, YouTube on 3 devices

**Deliverable:** Share extension saves URLs to local DB and shows confirmation toast

---

### Sprint 2 (Week 5-6): F02 — Auto-Categorization (Tier 2 + 2.5)
**Goal:** Classify saved content into categories (rule-based + TFLite)

**Tasks:**
- [ ] Build Tier 2 rule-based classifier (keyword dictionaries)
- [ ] Add multilingual keyword support (Hindi, Portuguese, Arabic, Bahasa)
- [ ] Build TF-IDF disambiguation scorer
- [ ] Train Tier 2.5 TFLite model (DistilBERT-tiny) on 10K labeled captions
- [ ] Integrate TFLite model inference
- [ ] Build ClassifierTierSelector (Tier 2.5 → Tier 2 fallback)
- [ ] Build ML Kit OCR pipeline (Latin + Devanagari)
- [ ] Implement richness score calculation
- [ ] Build RecategorizeSheet (manual category override UI)
- [ ] Track analytics: categorization_completed, category_overridden
- [ ] QA: Accuracy test on 300-caption dataset (target > 80% Tier 2, > 85% Tier 2.5)

**Deliverable:** Saved content is auto-categorized with 80%+ accuracy

---

### Sprint 3 (Week 7-8): F02 — Gemini Nano (Tier 1)
**Goal:** Integrate Gemini Nano for flagship devices

**Tasks:**
- [ ] Integrate Google AI Edge SDK (Gemini Nano)
- [ ] Implement AICore availability check + model download request
- [ ] Build Tier 1 classifier (Gemini Nano Prompt API)
- [ ] Implement prompt versioning + A/B test infrastructure
- [ ] Update ClassifierTierSelector (Tier 1 → 2.5 → 2 chain)
- [ ] Handle Gemini Nano errors (model not ready, busy, guardrail violation)
- [ ] Compare Tier 1 vs Tier 2.5 accuracy on test dataset
- [ ] Track analytics: tier used, prompt_version, processing_time_ms
- [ ] QA: Test on Pixel 8, Galaxy S24, mid-range device (Redmi)

**Deliverable:** Tier 1 classification works on supported devices (> 90% accuracy)

---

### Sprint 4 (Week 9-10): F03 — Content Library (Grid View + Search)
**Goal:** Visual library with filtering, sorting, search

**Tasks:**
- [ ] Build LibraryScreen with TabRow navigation
- [ ] Build uniform 2-column grid (LazyVerticalGrid)
- [ ] Build staggered grid (LazyVerticalStaggeredGrid)
- [ ] Build list view with swipe-to-delete
- [ ] Implement category filter bar (scrollable chips)
- [ ] Implement sort options (recent, oldest, category, platform, richness)
- [ ] Build SearchOverlay with FTS5 full-text search
- [ ] Implement Coil multi-layer image caching (memory, disk, network)
- [ ] Implement thumbnail prefetching (next 2 pages)
- [ ] Build empty states (first launch, no results, filtered empty)
- [ ] Build Quick Save FAB with URL paste
- [ ] Implement reactive Room Flow → UI updates
- [ ] Track analytics: library_opened, category_filter_applied, search_performed
- [ ] QA: Scroll performance test (1000+ saves, 60fps target)

**Deliverable:** Library displays saves in grid/list with search and filtering

---

### Sprint 5 (Week 11-12): F04 — Structured Data Cards (Part 1)
**Goal:** Recipe, Travel, Product cards with interactive components

**Tasks:**
- [ ] Build CardDetailScreen with collapsing header
- [ ] Build CardTemplateRegistry (auto-selects template by category)
- [ ] Build RecipeCard template
  - [ ] Ingredient checklist with progress counter
  - [ ] Unit conversion toggle (metric ↔ imperial)
  - [ ] Serving size adjuster (1-12 servings)
  - [ ] Tappable timer chips (ML Kit to detect time mentions)
  - [ ] Dietary tag detection (Vegetarian, Vegan, Gluten-Free, etc.)
- [ ] Build TravelCard template
  - [ ] Static map snapshot (Google Maps Static API)
  - [ ] Weather widget (OpenWeatherMap)
  - [ ] Distance calculation from user location
  - [ ] Currency conversion for budget amounts
- [ ] Build ProductCard template
  - [ ] Purchase link + affiliate tag injection
  - [ ] Price staleness warning (> 7 days)
  - [ ] "Similar Saves" cross-reference section
- [ ] Build "Share Card" image generator (3 formats: 9:16, 1:1, 4:5)
- [ ] Implement QR code generation for shared images
- [ ] Track analytics: card_viewed, view_original_tapped, share_card_generated
- [ ] QA: All 3 card types render correctly with varying data completeness

**Deliverable:** Recipe, Travel, Product cards render with structured data

---

### Sprint 6 (Week 13-14): F04 — Structured Data Cards (Part 2)
**Goal:** Fitness, Education, Generic cards + polish

**Tasks:**
- [ ] Build FitnessCard template
- [ ] Build EducationCard template
  - [ ] Difficulty level bar
  - [ ] "Mark as Learned" toggle
  - [ ] Tools & Resources links
- [ ] Build GenericCard with "Get Full Details" CTA
- [ ] Implement inline field editing (tap any field to edit)
- [ ] Build "More" menu (View Original, Recategorize, Share, Delete)
- [ ] Implement hero transition from grid → card detail
- [ ] Implement sticky "View Original" floating CTA on scroll
- [ ] Build WhatsApp text share (low-bandwidth alternative)
- [ ] Implement card theming (accent colors, dark mode)
- [ ] Track analytics: ingredient_checked, unit_conversion_toggled, timer_started
- [ ] QA: Accessibility audit (TalkBack, font scaling, Reduce Motion)

**Deliverable:** All 8 card types render correctly with full feature set

---

### Sprint 7 (Week 15-16): F05 — Deep Extract (Part 1: Server API + Client)
**Goal:** Deep extract triggers, server scrape, client SSE

**Tasks:**
- [ ] Build server Extract API endpoint (POST /api/v1/extract)
- [ ] Integrate Apify Instagram/TikTok scraper
- [ ] Implement scraper fallback chain (Apify → ScrapeCreators → Bright Data)
- [ ] Build circuit breaker pattern for scrapers
- [ ] Build job queue (BullMQ) with priority (paid > free)
- [ ] Implement SSE streaming (progress events)
- [ ] Build SSE client (OkHttp EventSource)
- [ ] Build DeepExtractProgressSheet (client UI)
- [ ] Implement quota management (10/month free, unlimited paid)
- [ ] Build QuotaExhaustedDialog with upgrade CTA
- [ ] Implement URL deduplication cache (Redis, 30-day TTL)
- [ ] Track analytics: deep_extract_initiated, deep_extract_completed
- [ ] QA: Extract completes in < 30 sec for 60-sec Reel

**Deliverable:** "Get Full Details" triggers server scrape, shows progress

---

### Sprint 8 (Week 17-18): F05 — Deep Extract (Part 2: On-Device Pipeline)
**Goal:** On-device transcription, OCR, extraction

**Tasks:**
- [ ] Build video download + audio extraction (MediaExtractor)
- [ ] Integrate ML Kit Speech Recognition (transcription)
- [ ] Build key frame extractor (1fps)
- [ ] Integrate ML Kit TextRecognition (OCR on key frames)
- [ ] Build text deduplication + combination logic
- [ ] Integrate Gemini Nano extraction from combined text
- [ ] Implement memory management (< 300MB peak)
- [ ] Implement battery checks (warn if < 20%)
- [ ] Build background processing (DeepExtractWorker)
- [ ] Implement auto-extract setting (paid users)
- [ ] Build batch extract UI (multi-select → extract)
- [ ] Implement temp file cleanup (< 60 sec lifecycle)
- [ ] Track analytics: deep_extract_stage_reached, processing_path
- [ ] QA: On-device pipeline completes in < 30 sec, cost < $0.003

**Deliverable:** Deep extract runs on-device for Tier 1 devices

---

### Sprint 9 (Week 19-20): Polish, Testing, Launch Prep
**Goal:** Bug fixes, performance optimization, Play Store submission

**Tasks:**
- [ ] Performance profiling (Systrace, memory profiler)
- [ ] Fix all P0/P1 bugs from QA
- [ ] Implement crash reporting (Firebase Crashlytics)
- [ ] Implement analytics (Firebase Analytics or Mixpanel)
- [ ] Build onboarding flow (3-screen carousel + share extension tutorial)
- [ ] Build Settings screen (account, preferences, about, help)
- [ ] Implement ProGuard/R8 optimization
- [ ] Write Play Store listing (title, description, screenshots, video)
- [ ] Submit to Play Store (Internal Testing track)
- [ ] Conduct beta testing with 50 users
- [ ] Fix beta feedback issues
- [ ] Submit to Play Store (Production, staged rollout 10%)
- [ ] Monitor crash-free rate (target > 99.5%)

**Deliverable:** App live on Play Store for 10% of users

---

### Sprint 10 (Week 21-22): Post-Launch Monitoring + Phase 2 Prep
**Goal:** Monitor metrics, iterate on feedback, plan Phase 2

**Tasks:**
- [ ] Monitor analytics dashboard (DAU, retention, categorization accuracy)
- [ ] Monitor server costs (extract API, scraper API)
- [ ] Respond to user reviews and support tickets
- [ ] Fix critical bugs from production
- [ ] Roll out to 50%, then 100% of users
- [ ] Analyze conversion funnel (install → save → Pro upgrade)
- [ ] Begin Phase 2 planning (Collections, Smart Search, Widgets)

**Deliverable:** Stable app at 100% rollout, < 1% crash rate, roadmap for Phase 2

---

## Summary

This architecture plan provides a **complete, production-ready blueprint** for building the Staqer Android app. Key highlights:

- **Multi-module Gradle structure** with convention plugins for scalability
- **MVVM + MVI architecture** with Hilt DI and Jetpack Compose
- **Share Extension** via transparent Activity + WorkManager background processing
- **Room + FTS5** for offline-first, searchable local storage
- **3-tier AI classification**: Gemini Nano (Tier 1), TFLite bundled model (Tier 2.5), rule-based (Tier 2)
- **ML Kit OCR + Speech Recognition** for on-device extraction
- **KMP shared module** for networking, data models, and classification rules
- **Comprehensive testing** (unit, UI, integration) with 75%+ coverage target
- **CI/CD with GitHub Actions** and automated Play Store deployment
- **10-sprint roadmap** mapping features F01-F05 to 2-week sprints

**Next Steps:**
1. Review this plan with the team
2. Set up the initial project structure (Sprint 0)
3. Begin Sprint 1 (Share Extension) implementation
4. Iterate based on user testing and analytics

This document serves as the **single source of truth** for Android architecture decisions. Update it as the project evolves.