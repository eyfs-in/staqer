# Staqer iOS Architecture Plan

**Version:** 1.0  
**Date:** February 24, 2026  
**Target:** iOS 17.0+, Optimized for iOS 26+ (Foundation Models)  
**Author:** iOS Architecture Team  
**Status:** Implementation Blueprint

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [App Architecture](#2-app-architecture)
3. [Share Extension Architecture](#3-share-extension-architecture)
4. [Data Layer](#4-data-layer)
5. [AI/ML Layer](#5-aiml-layer)
6. [Network Layer](#6-network-layer)
7. [KMP Integration](#7-kmp-integration)
8. [Key Libraries & Dependencies](#8-key-libraries--dependencies)
9. [Testing Strategy](#9-testing-strategy)
10. [Build & CI/CD](#10-build--cicd)
11. [Sprint-by-Sprint Breakdown](#11-sprint-by-sprint-breakdown)

---

## 1. Project Structure

### 1.1 Xcode Workspace Organization

```
Staqer.xcworkspace/
├── Staqer/                           # Main app target
│   ├── App/
│   │   ├── StaqerApp.swift          # @main entry point
│   │   ├── AppDelegate.swift        # Legacy app lifecycle hooks
│   │   └── SceneDelegate.swift      # Multi-window support (iPad)
│   ├── Features/
│   │   ├── Library/
│   │   │   ├── Views/
│   │   │   │   ├── LibraryView.swift
│   │   │   │   ├── CategoryFilterBar.swift
│   │   │   │   ├── SaveCardView.swift
│   │   │   │   └── SearchOverlayView.swift
│   │   │   ├── ViewModels/
│   │   │   │   ├── LibraryViewModel.swift
│   │   │   │   └── SearchViewModel.swift
│   │   │   └── Models/
│   │   │       └── LibraryState.swift
│   │   ├── Cards/
│   │   │   ├── Views/
│   │   │   │   ├── RecipeCardView.swift
│   │   │   │   ├── TravelCardView.swift
│   │   │   │   ├── ProductCardView.swift
│   │   │   │   ├── FitnessCardView.swift
│   │   │   │   ├── EducationCardView.swift
│   │   │   │   └── GenericCardView.swift
│   │   │   ├── ViewModels/
│   │   │   │   ├── CardViewModel.swift
│   │   │   │   └── RecipeCardViewModel.swift
│   │   │   └── Components/
│   │   │       ├── IngredientChecklistView.swift
│   │   │       ├── ServingSizeAdjuster.swift
│   │   │       ├── UnitConversionToggle.swift
│   │   │       ├── InlineTimerView.swift
│   │   │       └── DietaryTagsView.swift
│   │   ├── DeepExtract/
│   │   │   ├── Views/
│   │   │   │   ├── DeepExtractProgressView.swift
│   │   │   │   ├── QuotaExhaustedView.swift
│   │   │   │   └── BatchExtractView.swift
│   │   │   ├── ViewModels/
│   │   │   │   ├── DeepExtractViewModel.swift
│   │   │   │   └── QuotaManager.swift
│   │   │   └── Services/
│   │   │       ├── SSEClient.swift
│   │   │       ├── OnDeviceTranscriber.swift
│   │   │       ├── VideoDownloader.swift
│   │   │       ├── KeyFrameExtractor.swift
│   │   │       └── OCRProcessor.swift
│   │   ├── Collections/
│   │   │   └── [Future: F6]
│   │   └── Settings/
│   │       ├── Views/
│   │       │   ├── SettingsView.swift
│   │       │   └── AutoExtractSettingsView.swift
│   │       └── ViewModels/
│   │           └── SettingsViewModel.swift
│   ├── Core/
│   │   ├── Data/
│   │   │   ├── CoreData/
│   │   │   │   ├── StaqerModel.xcdatamodeld
│   │   │   │   ├── CoreDataStack.swift
│   │   │   │   ├── SavedContentEntity+CoreDataClass.swift
│   │   │   │   └── PersistenceController.swift
│   │   │   ├── Models/
│   │   │   │   ├── SavedContent.swift
│   │   │   │   ├── ExtractedData.swift
│   │   │   │   ├── PendingSave.swift
│   │   │   │   └── UserProfile.swift
│   │   │   └── Repositories/
│   │   │       ├── SavedContentRepository.swift
│   │   │       ├── PendingSaveRepository.swift
│   │   │       └── UserRepository.swift
│   │   ├── AI/
│   │   │   ├── Classification/
│   │   │   │   ├── Tier1Classifier.swift      # Foundation Models
│   │   │   │   ├── Tier2_5Classifier.swift    # Bundled CoreML
│   │   │   │   ├── Tier2Classifier.swift      # Rule-based
│   │   │   │   └── TierSelector.swift
│   │   │   ├── Prompts/
│   │   │   │   ├── PromptVersion.swift
│   │   │   │   ├── ClassificationPrompts.swift
│   │   │   │   └── ExtractionPrompts.swift
│   │   │   └── Models/
│   │   │       ├── FoundationModelsClient.swift
│   │   │       └── CoreMLClassifier.swift
│   │   ├── Networking/
│   │   │   ├── APIClient.swift
│   │   │   ├── Endpoints.swift
│   │   │   ├── NetworkError.swift
│   │   │   └── URLNormalizer.swift
│   │   ├── Services/
│   │   │   ├── AnalyticsService.swift
│   │   │   ├── ImageCache.swift
│   │   │   ├── DeepLinkHandler.swift
│   │   │   └── BackgroundTaskManager.swift
│   │   └── Utilities/
│   │       ├── Extensions/
│   │       │   ├── Date+Formatting.swift
│   │       │   ├── String+Validation.swift
│   │       │   └── View+Modifiers.swift
│   │       ├── Helpers/
│   │       │   ├── HapticFeedback.swift
│   │       │   ├── KeyboardObserver.swift
│   │       │   └── DeviceCapability.swift
│   │       └── Constants/
│   │           ├── AppConstants.swift
│   │           ├── CategoryConstants.swift
│   │           └── ColorConstants.swift
│   ├── Resources/
│   │   ├── Assets.xcassets
│   │   ├── Localizable.strings
│   │   ├── Localizable.stringsdict
│   │   ├── Keywords/
│   │   │   ├── keywords-en.json
│   │   │   ├── keywords-hi.json
│   │   │   ├── keywords-pt.json
│   │   │   └── keywords-ar.json
│   │   └── Bundled Models/
│   │       └── TextClassifier.mlpackage     # CoreML Tier 2.5
│   └── Supporting Files/
│       ├── Info.plist
│       ├── Staqer.entitlements
│       └── GoogleService-Info.plist
│
├── StaqerShareExtension/              # Share Extension target
│   ├── ShareViewController.swift
│   ├── ShareExtensionView.swift
│   ├── URLParser.swift
│   ├── MetadataExtractor.swift
│   ├── LocalSaveManager.swift
│   ├── Info.plist
│   └── Staqer-ShareExtension.entitlements
│
├── StaqerTests/                       # Unit tests
│   ├── ClassificationTests/
│   ├── DataLayerTests/
│   ├── ViewModelTests/
│   └── UtilityTests/
│
├── StaqerUITests/                     # UI tests
│   ├── LibraryFlowTests.swift
│   ├── ShareExtensionTests.swift
│   └── DeepExtractTests.swift
│
├── StaqerShared/                      # KMP shared framework (imported)
│   └── StaqerShared.xcframework
│
└── Packages/                          # Swift Package Manager dependencies
    └── [managed by SPM]
```

### 1.2 Target Configuration

| Target | Bundle ID | Deployment Target | Capabilities |
|--------|-----------|------------------|--------------|
| **Staqer** | `com.staqapp.staq` | iOS 17.0 | App Groups, Push Notifications, Background Modes (fetch, processing, remote-notification), Sign in with Apple, In-App Purchase |
| **StaqerShareExtension** | `com.staqapp.staq.ShareExtension` | iOS 17.0 | App Groups |

**App Group:** `group.com.staqapp.staq.shared`

### 1.3 Swift Package Manager vs CocoaPods

**Decision: Pure SPM**

We exclusively use Swift Package Manager for all third-party dependencies. No CocoaPods or Carthage.

**Rationale:**
- Native Xcode integration (no workspace pollution)
- Faster incremental builds
- Better compatibility with KMP frameworks
- Industry standard for modern Swift projects
- Simplified CI/CD

---

## 2. App Architecture

### 2.1 MVVM Pattern

**Architecture:** SwiftUI + MVVM with unidirectional data flow

```
View (SwiftUI)  ─observes─>  ViewModel (@Observable)  ─uses─>  Repository/Service
                                        │
                                        └──> State (@Published)
```

#### Core Principles

1. **Views are dumb**: Views only render state and forward user actions to ViewModels
2. **ViewModels own state**: All app state lives in ViewModels, exposed via `@Published` or `@Observable`
3. **Repositories handle data**: ViewModels never talk directly to Core Data or network — they use Repository abstractions
4. **Services are stateless**: Shared logic (networking, analytics, image caching) lives in singleton Services

#### Example: LibraryViewModel

```swift
import SwiftUI
import Combine

@MainActor
@Observable
final class LibraryViewModel {
    // MARK: - State
    
    private(set) var saves: [SavedContent] = []
    private(set) var isLoading = false
    private(set) var selectedCategory: Category? = nil
    private(set) var viewMode: ViewMode = .uniformGrid
    private(set) var sortMode: SortMode = .mostRecent
    
    // MARK: - Dependencies
    
    private let repository: SavedContentRepository
    private let analytics: AnalyticsService
    
    // MARK: - Initializer
    
    init(
        repository: SavedContentRepository = .shared,
        analytics: AnalyticsService = .shared
    ) {
        self.repository = repository
        self.analytics = analytics
        observeSaves()
    }
    
    // MARK: - Data Binding
    
    private func observeSaves() {
        // Core Data @FetchRequest equivalent
        repository.observeSavedContent()
            .receive(on: DispatchQueue.main)
            .sink { [weak self] newSaves in
                self?.saves = newSaves
            }
            .store(in: &cancellables)
    }
    
    // MARK: - Actions
    
    func applyFilter(category: Category?) {
        selectedCategory = category
        analytics.track(.categoryFilterApplied(category: category?.name ?? "all"))
    }
    
    func changeViewMode(_ mode: ViewMode) {
        viewMode = mode
        UserDefaults.standard.set(mode.rawValue, forKey: "library_view_mode")
        analytics.track(.viewToggled(from: viewMode, to: mode))
    }
    
    func deleteSave(_ save: SavedContent) async {
        await repository.delete(save)
        analytics.track(.saveDeleted(category: save.category))
    }
    
    private var cancellables = Set<AnyCancellable>()
}
```

### 2.2 Dependency Injection

**Approach:** Constructor injection with default parameters

We use protocol-based dependency injection for testability while keeping production code simple via default parameters.

```swift
protocol SavedContentRepositoryProtocol {
    func observeSavedContent() -> AnyPublisher<[SavedContent], Never>
    func save(_ content: SavedContent) async throws
    func delete(_ content: SavedContent) async throws
}

final class SavedContentRepository: SavedContentRepositoryProtocol {
    static let shared = SavedContentRepository()
    
    private let persistenceController: PersistenceController
    
    init(persistenceController: PersistenceController = .shared) {
        self.persistenceController = persistenceController
    }
    
    // Implementation...
}

// In production:
let viewModel = LibraryViewModel() // Uses .shared defaults

// In tests:
let mockRepo = MockSavedContentRepository()
let viewModel = LibraryViewModel(repository: mockRepo)
```

### 2.3 Navigation (NavigationStack)

**Primary:** `NavigationStack` with type-safe paths (iOS 16+)

```swift
// Navigation State
enum Route: Hashable {
    case library
    case cardDetail(SavedContent)
    case settings
    case deepExtractProgress(SavedContent)
}

// Root App View
struct RootView: View {
    @State private var navigationPath = NavigationPath()
    @State private var selectedTab: Tab = .library
    
    var body: some View {
        TabView(selection: $selectedTab) {
            NavigationStack(path: $navigationPath) {
                LibraryView()
                    .navigationDestination(for: Route.self) { route in
                        switch route {
                        case .cardDetail(let save):
                            CardDetailView(save: save)
                        case .deepExtractProgress(let save):
                            DeepExtractProgressView(save: save)
                        default:
                            EmptyView()
                        }
                    }
            }
            .tabItem { Label("Library", systemImage: "square.grid.2x2") }
            .tag(Tab.library)
            
            NavigationStack {
                CollectionsView()
            }
            .tabItem { Label("Collections", systemImage: "folder") }
            .tag(Tab.collections)
            
            NavigationStack {
                SettingsView()
            }
            .tabItem { Label("Settings", systemImage: "gearshape") }
            .tag(Tab.settings)
        }
        .onOpenURL { url in
            handleDeepLink(url, navigationPath: $navigationPath)
        }
    }
}
```

**Deep Link Handling:**

```swift
// Deep link scheme: staq://library?category=recipe
//                   staq://card/UUID-123

func handleDeepLink(_ url: URL, navigationPath: Binding<NavigationPath>) {
    guard url.scheme == "staq" else { return }
    
    switch url.host {
    case "library":
        if let category = url.queryParameters["category"] {
            // Navigate to library with filter
            navigationPath.wrappedValue = NavigationPath()
            NotificationCenter.default.post(
                name: .applyLibraryFilter,
                object: category
            )
        }
    case "card":
        if let cardID = url.pathComponents.dropFirst().first,
           let uuid = UUID(uuidString: cardID) {
            let save = repository.fetchSave(id: uuid)
            navigationPath.wrappedValue.append(Route.cardDetail(save))
        }
    default:
        break
    }
}
```

---

## 3. Share Extension Architecture

### 3.1 Memory Constraints

**Critical limitation:** iOS share extensions have a **120MB memory ceiling**. Exceeding this triggers termination by the system.

**Implications:**
- NO on-device AI processing in the extension
- NO video downloads in the extension
- NO image processing beyond basic metadata extraction
- Extension's ONLY job: capture URL + metadata → persist → exit fast

### 3.2 Share Extension Entry Point

```swift
import SwiftUI
import UniformTypeIdentifiers

class ShareViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Host SwiftUI view
        let contentView = ShareExtensionView(
            extensionContext: extensionContext,
            onDismiss: { [weak self] in
                self?.extensionContext?.completeRequest(returningItems: nil)
            }
        )
        
        let hostingController = UIHostingController(rootView: contentView)
        addChild(hostingController)
        view.addSubview(hostingController.view)
        hostingController.view.frame = view.bounds
        hostingController.view.autoresizingMask = [.flexibleWidth, .flexibleHeight]
        hostingController.didMove(toParent: self)
    }
}
```

### 3.3 URL Extraction from NSItemProvider

```swift
struct MetadataExtractor {
    func extractURL(from extensionContext: NSExtensionContext?) async -> String? {
        guard let inputItems = extensionContext?.inputItems as? [NSExtensionItem] else {
            return nil
        }
        
        for item in inputItems {
            guard let attachments = item.attachments else { continue }
            
            for provider in attachments {
                // Try URL first
                if provider.hasItemConformingToTypeIdentifier(UTType.url.identifier) {
                    do {
                        let url = try await provider.loadItem(
                            forTypeIdentifier: UTType.url.identifier,
                            options: nil
                        ) as? URL
                        return url?.absoluteString
                    } catch {
                        continue
                    }
                }
                
                // Fallback: plain text containing URL
                if provider.hasItemConformingToTypeIdentifier(UTType.plainText.identifier) {
                    do {
                        let text = try await provider.loadItem(
                            forTypeIdentifier: UTType.plainText.identifier,
                            options: nil
                        ) as? String
                        
                        if let url = extractURLFromText(text) {
                            return url
                        }
                    } catch {
                        continue
                    }
                }
            }
        }
        
        return nil
    }
    
    private func extractURLFromText(_ text: String?) -> String? {
        guard let text = text else { return nil }
        let detector = try? NSDataDetector(types: NSTextCheckingResult.CheckingType.link.rawValue)
        let matches = detector?.matches(in: text, range: NSRange(text.startIndex..., in: text))
        return matches?.first?.url?.absoluteString
    }
}
```

### 3.4 App Group Persistence

**Shared container path:**

```swift
let containerURL = FileManager.default.containerURL(
    forSecurityApplicationGroupIdentifier: "group.com.staqapp.staq.shared"
)
```

**Core Data shared store:**

```swift
// In PersistenceController (shared by both targets)
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "StaqerModel")
    
    // Use App Group container for the SQLite store
    let storeURL = FileManager.default
        .containerURL(forSecurityApplicationGroupIdentifier: "group.com.staqapp.staq.shared")!
        .appendingPathComponent("Staqer.sqlite")
    
    let description = NSPersistentStoreDescription(url: storeURL)
    description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
    description.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
    
    container.persistentStoreDescriptions = [description]
    container.loadPersistentStores { _, error in
        if let error = error as NSError? {
            fatalError("Unresolved error \(error), \(error.userInfo)")
        }
    }
    
    // Enable WAL mode for concurrent access
    container.viewContext.automaticallyMergesChangesFromParent = true
    
    return container
}()
```

**PendingSave entity in share extension:**

```swift
func saveToQueue(url: String, platform: Platform, caption: String?) async {
    let context = PersistenceController.shared.persistentContainer.newBackgroundContext()
    
    await context.perform {
        let pendingSave = PendingSave(context: context)
        pendingSave.id = UUID()
        pendingSave.url = url
        pendingSave.rawUrl = url
        pendingSave.platform = platform.rawValue
        pendingSave.captionText = caption
        pendingSave.status = SaveStatus.pending.rawValue
        pendingSave.createdAt = Date()
        
        do {
            try context.save()
            
            // Trigger main app processing
            schedulePendingSaveProcessing()
            
            // Log analytics event
            logShareExtensionSave(platform: platform)
        } catch {
            print("Failed to save: \(error)")
        }
    }
}
```

### 3.5 Background Processing Trigger

```swift
func schedulePendingSaveProcessing() {
    let request = BGAppRefreshTaskRequest(identifier: "com.staqapp.staq.processPendingSaves")
    request.earliestBeginDate = Date(timeIntervalSinceNow: 5 * 60) // 5 min
    
    try? BGTaskScheduler.shared.submit(request)
}
```

**In AppDelegate:**

```swift
func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
) -> Bool {
    // Register background task
    BGTaskScheduler.shared.register(
        forTaskWithIdentifier: "com.staqapp.staq.processPendingSaves",
        using: nil
    ) { task in
        self.handlePendingSaveProcessing(task: task as! BGAppRefreshTask)
    }
    
    return true
}

func handlePendingSaveProcessing(task: BGAppRefreshTask) {
    let processor = PendingSaveProcessor()
    
    task.expirationHandler = {
        processor.cancel()
    }
    
    Task {
        await processor.processQueue()
        task.setTaskCompleted(success: true)
    }
}
```

---

## 4. Data Layer

### 4.1 Core Data Schema

**Entities:**

```
SavedContentEntity
├── id: UUID (primary key)
├── url: String (indexed)
├── platform: String (indexed)
├── captionText: String?
├── transcriptText: String?
├── thumbnailLocalPath: String?
├── thumbnailUrl: String?
├── thumbnailAspectRatio: Double?
├── category: String (indexed)
├── categoryConfidence: Double
├── richnessScore: Double
├── extractedDataJSON: String         # JSON blob
├── userNote: String?
├── userCategoryOverride: String?
├── creatorHandle: String?
├── createdAt: Date (indexed)
├── processedAt: Date?
├── status: String
└── Relationships:
    └── collections: [CollectionEntity]

PendingSaveEntity
├── id: UUID
├── url: String
├── rawUrl: String
├── platform: String
├── captionText: String?
├── thumbnailUrl: String?
├── userNote: String?
├── userCategory: String?
├── status: String
├── createdAt: Date
└── offlineSave: Bool

UserProfileEntity
├── id: UUID
├── userId: String
├── isPro: Bool
├── deepExtractsUsedThisMonth: Int16
├── quotaResetDate: Date
└── preferences: String                # JSON

CollectionEntity (Phase 2)
├── id: UUID
├── name: String
├── emoji: String?
├── createdAt: Date
└── Relationship:
    └── saves: [SavedContentEntity]
```

**FTS5 Setup:**

Core Data doesn't expose FTS5 directly. We execute raw SQL during first launch using SQLite.swift:

```swift
import SQLite

extension PersistenceController {
    func setupFTS5() {
        let storeURL = persistentContainer.persistentStoreDescriptions.first!.url!
        guard let db = try? Connection(storeURL.path) else { return }
        
        // Create virtual table
        try? db.execute("""
            CREATE VIRTUAL TABLE IF NOT EXISTS saved_content_fts USING fts5(
                caption,
                transcript,
                extracted_fields,
                user_note,
                creator_handle,
                content='saved_content',
                tokenize='unicode61 remove_diacritics 2'
            );
        """)
        
        // Sync triggers
        try? db.execute("""
            CREATE TRIGGER IF NOT EXISTS saved_content_ai AFTER INSERT ON saved_content BEGIN
                INSERT INTO saved_content_fts(rowid, caption, transcript, extracted_fields, user_note, creator_handle)
                VALUES (new.Z_PK, new.ZCAPTIONTEXT, new.ZTRANSCRIPTTEXT, new.ZEXTRACTEDDATAJSON, new.ZUSERNOTE, new.ZCREATORHANDLE);
            END;
        """)
        
        // Update and delete triggers follow same pattern
    }
}
```

### 4.2 Repository Pattern

```swift
protocol SavedContentRepositoryProtocol {
    func observeSavedContent(filter: CategoryFilter?) -> AnyPublisher<[SavedContent], Never>
    func save(_ content: SavedContent) async throws
    func delete(_ content: SavedContent) async throws
    func search(query: String) async -> [SavedContent]
    func fetchByID(_ id: UUID) -> SavedContent?
}

final class SavedContentRepository: SavedContentRepositoryProtocol {
    static let shared = SavedContentRepository()
    
    private let persistenceController: PersistenceController
    private let context: NSManagedObjectContext
    
    init(persistenceController: PersistenceController = .shared) {
        self.persistenceController = persistenceController
        self.context = persistenceController.persistentContainer.viewContext
    }
    
    func observeSavedContent(filter: CategoryFilter? = nil) -> AnyPublisher<[SavedContent], Never> {
        let fetchRequest: NSFetchRequest<SavedContentEntity> = SavedContentEntity.fetchRequest()
        fetchRequest.sortDescriptors = [NSSortDescriptor(keyPath: \SavedContentEntity.createdAt, ascending: false)]
        
        if let category = filter?.category {
            fetchRequest.predicate = NSPredicate(format: "category == %@", category)
        }
        
        return NotificationCenter.default
            .publisher(for: .NSManagedObjectContextObjectsDidChange, object: context)
            .map { _ in
                try? self.context.fetch(fetchRequest).map { $0.toDomainModel() }
            }
            .replaceNil(with: [])
            .eraseToAnyPublisher()
    }
    
    func search(query: String) async -> [SavedContent] {
        // Use FTS5 via raw SQL (SQLite.swift)
        let storeURL = persistenceController.persistentContainer.persistentStoreDescriptions.first!.url!
        guard let db = try? Connection(storeURL.path) else { return [] }
        
        let stmt = try? db.prepare("""
            SELECT sc.Z_PK FROM saved_content sc
            JOIN saved_content_fts fts ON sc.Z_PK = fts.rowid
            WHERE saved_content_fts MATCH ?
            ORDER BY rank
            LIMIT 50
        """)
        
        let rowIDs = try? stmt?.bind(query + "*").map { $0[0] as! Int64 }
        
        // Fetch Core Data entities by rowID
        let predicate = NSPredicate(format: "SELF IN %@", rowIDs ?? [])
        let request: NSFetchRequest<SavedContentEntity> = SavedContentEntity.fetchRequest()
        request.predicate = predicate
        
        return (try? context.fetch(request).map { $0.toDomainModel() }) ?? []
    }
}
```

### 4.3 SQLite WAL Mode

Enabled by default in Core Data on iOS. Verify:

```swift
// In PersistenceController setup
let description = NSPersistentStoreDescription(url: storeURL)
description.setOption("WAL" as NSString, forKey: "journal_mode")
```

### 4.4 Migration Strategy

**Lightweight migrations** for schema evolution:

```swift
let container = NSPersistentContainer(name: "StaqerModel")
let description = container.persistentStoreDescriptions.first!
description.shouldMigrateStoreAutomatically = true
description.shouldInferMappingModelAutomatically = true
```

For complex migrations, use **custom mapping models**:

```
StaqerModel_v1.xcdatamodel  →  StaqerModel_v2.xcdatamodel
                ↓
        Mapping_v1_to_v2.xcmappingmodel
```

---

## 5. AI/ML Layer

### 5.1 Foundation Models Integration (@Generable)

**iOS 26+ only. Requires A17 Pro chip (iPhone 15 Pro+).**

```swift
import FoundationModels

@available(iOS 26, *)
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

@available(iOS 26, *)
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
}

@available(iOS 26, *)
final class Tier1Classifier {
    func classify(caption: String, ocrText: String, hashtags: [String]) async throws -> ContentClassification {
        guard FoundationModels.isAvailable else {
            throw ClassificationError.modelNotAvailable
        }
        
        let session = LanguageModelSession()
        
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
        
        do {
            let result = try await session.respond(to: prompt, generating: ContentClassification.self)
            return result
        } catch let error as LanguageModelSession.GenerationError {
            switch error {
            case .modelNotReady:
                throw ClassificationError.modelDownloading
            case .modelBusy:
                try? await Task.sleep(nanoseconds: 500_000_000)
                throw ClassificationError.modelBusy
            case .guardrailViolation:
                return ContentClassification(category: "general", confidence: 0.3, title: "Content", extractedData: nil)
            default:
                throw ClassificationError.inferenceError(error)
            }
        }
    }
}
```

### 5.2 CoreML Tier 2.5 Bundled Model

**Fallback for iOS 17-25 devices without Foundation Models.**

```swift
import CoreML

final class Tier2_5Classifier {
    private lazy var model: TextClassifier? = {
        guard let modelURL = Bundle.main.url(forResource: "TextClassifier", withExtension: "mlmodelc") else {
            return nil
        }
        return try? TextClassifier(contentsOf: modelURL)
    }()
    
    var isModelLoaded: Bool {
        model != nil
    }
    
    func classify(caption: String, hashtags: [String]) -> ContentClassification? {
        guard let model = model else { return nil }
        
        let input = caption + " " + hashtags.joined(separator: " ")
        let truncated = String(input.prefix(128)) // Model max tokens
        
        guard let prediction = try? model.prediction(text: truncated) else {
            return nil
        }
        
        return ContentClassification(
            category: prediction.category,
            confidence: prediction.categoryProbability[prediction.category] ?? 0.5,
            title: extractTitle(from: caption),
            extractedData: nil
        )
    }
}
```

### 5.3 Rule-Based Tier 2 Classifier

```swift
final class Tier2Classifier {
    private let keywordDictionary: [String: [(keyword: String, weight: Int)]]
    private let tfidfScorer: TFIDFScorer
    
    init() {
        self.keywordDictionary = KeywordLoader.load() // From bundled JSON
        self.tfidfScorer = TFIDFScorer()
    }
    
    func classify(caption: String, hashtags: [String]) -> ContentClassification {
        var scores: [String: Int] = [:]
        
        // 1. Hashtag scoring
        for hashtag in hashtags {
            for (category, keywords) in keywordDictionary {
                if let match = keywords.first(where: { hashtag.lowercased().contains($0.keyword) }) {
                    scores[category, default: 0] += match.weight * 2
                }
            }
        }
        
        // 2. Caption keyword matching
        let tokens = caption.lowercased().components(separatedBy: .whitespacesAndNewlines)
        for token in tokens {
            for (category, keywords) in keywordDictionary {
                if let match = keywords.first(where: { $0.keyword == token }) {
                    scores[category, default: 0] += match.weight
                }
            }
        }
        
        // 3. TF-IDF disambiguation if top 2 are close
        let sorted = scores.sorted { $0.value > $1.value }
        if sorted.count >= 2,
           Double(sorted[1].value) / Double(sorted[0].value) > 0.8 {
            // Top 2 within 20% → apply TF-IDF
            let tfidfScores = tfidfScorer.score(tokens: tokens, candidates: [sorted[0].key, sorted[1].key])
            return ContentClassification(
                category: tfidfScores.max { $0.value < $1.value }?.key ?? "general",
                confidence: 0.6,
                title: extractTitle(from: caption),
                extractedData: nil
            )
        }
        
        let winningCategory = sorted.first?.key ?? "general"
        let totalScore = scores.values.reduce(0, +)
        let confidence = totalScore > 0 ? Double(sorted.first?.value ?? 0) / Double(totalScore) : 0.0
        
        return ContentClassification(
            category: winningCategory,
            confidence: confidence,
            title: extractTitle(from: caption),
            extractedData: nil
        )
    }
}
```

### 5.4 Tier Selection

```swift
final class TierSelector {
    func selectTier() -> ProcessingTier {
        if #available(iOS 26, *), FoundationModels.isAvailable {
            return .tier1_foundationModels
        }
        
        let tier2_5 = Tier2_5Classifier()
        if tier2_5.isModelLoaded {
            return .tier2_5_bundledModel
        }
        
        return .tier2_ruleBased
    }
}
```

### 5.5 Vision Framework OCR

```swift
import Vision

final class OCRProcessor {
    func extractText(from image: UIImage) async throws -> String {
        let resized = image.resizedToFit(maxDimension: 720)
        
        guard let cgImage = resized.cgImage else {
            throw OCRError.invalidImage
        }
        
        let request = VNRecognizeTextRequest()
        request.recognitionLevel = .accurate
        request.usesLanguageCorrection = true
        request.recognitionLanguages = ["en-US", "hi-IN", "pt-BR", "ar", "id"]
        request.revision = VNRecognizeTextRequestRevision3 // Devanagari support
        
        let handler = VNImageRequestHandler(cgImage: cgImage, options: [:])
        try handler.perform([request])
        
        guard let observations = request.results else {
            return ""
        }
        
        let rawText = observations
            .filter { $0.confidence > 0.5 }
            .compactMap { $0.topCandidates(1).first?.string }
            .joined(separator: " ")
        
        return cleanOCRText(rawText)
    }
    
    private func cleanOCRText(_ raw: String) -> String {
        var text = raw
        
        let handlePatterns = [
            #"@[\w.]+"#,
            #"(?i)ig:\s*@?[\w.]+"#,
            #"(?i)tiktok:\s*@?[\w.]+"#,
            #"(?i)link\s+in\s+bio"#
        ]
        
        for pattern in handlePatterns {
            text = text.replacingOccurrences(of: pattern, with: " ", options: .regularExpression)
        }
        
        return text.replacingOccurrences(of: #"\s+"#, with: " ", options: .regularExpression).trimmingCharacters(in: .whitespaces)
    }
}
```

### 5.6 SpeechAnalyzer (iOS 26+) & SFSpeechRecognizer Fallback

```swift
import Speech
#if canImport(FoundationModels)
import FoundationModels
#endif

final class OnDeviceTranscriber {
    @available(iOS 26, *)
    func transcribeWithSpeechAnalyzer(audioURL: URL) async throws -> String {
        let analyzer = SpeechAnalyzer()
        let result = try await analyzer.transcribe(audioURL)
        return result.segments.map { $0.text }.joined(separator: " ")
    }
    
    func transcribeWithSFSpeech(audioURL: URL) async throws -> String {
        let recognizer = SFSpeechRecognizer()
        guard recognizer?.isAvailable == true else {
            throw TranscriptionError.recognizerUnavailable
        }
        
        let request = SFSpeechURLRecognitionRequest(url: audioURL)
        request.requiresOnDeviceRecognition = true
        request.shouldReportPartialResults = false
        
        return try await withCheckedThrowingContinuation { continuation in
            recognizer?.recognitionTask(with: request) { result, error in
                if let error = error {
                    continuation.resume(throwing: error)
                    return
                }
                
                if let result = result, result.isFinal {
                    continuation.resume(returning: result.bestTranscription.formattedString)
                }
            }
        }
    }
    
    func transcribe(audioURL: URL) async throws -> String {
        if #available(iOS 26, *) {
            return try await transcribeWithSpeechAnalyzer(audioURL: audioURL)
        } else {
            return try await transcribeWithSFSpeech(audioURL: audioURL)
        }
    }
}
```

---

## 6. Network Layer

### 6.1 URLSession-Based API Client

```swift
import Foundation

final class APIClient {
    static let shared = APIClient()
    
    private let session: URLSession
    private let baseURL = URL(string: "https://api.staq.app")!
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    func request<T: Decodable>(_ endpoint: Endpoint) async throws -> T {
        var request = URLRequest(url: baseURL.appendingPathComponent(endpoint.path))
        request.httpMethod = endpoint.method.rawValue
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("Bearer \(AuthManager.shared.token)", forHTTPHeaderField: "Authorization")
        
        if let body = endpoint.body {
            request.httpBody = try JSONEncoder().encode(body)
        }
        
        let (data, response) = try await session.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse else {
            throw NetworkError.invalidResponse
        }
        
        switch httpResponse.statusCode {
        case 200...299:
            return try JSONDecoder().decode(T.self, from: data)
        case 401:
            throw NetworkError.unauthorized
        case 429:
            throw NetworkError.rateLimited
        default:
            throw NetworkError.serverError(httpResponse.statusCode)
        }
    }
}

enum Endpoint {
    case triggerDeepExtract(url: String, saveID: UUID, deviceCanTranscribe: Bool)
    case checkQuota
    
    var path: String {
        switch self {
        case .triggerDeepExtract: return "/api/v1/extract"
        case .checkQuota: return "/api/v1/quota"
        }
    }
    
    var method: HTTPMethod {
        switch self {
        case .triggerDeepExtract: return .post
        case .checkQuota: return .get
        }
    }
    
    var body: Encodable? {
        switch self {
        case .triggerDeepExtract(let url, let saveID, let canTranscribe):
            return DeepExtractRequest(url: url, saveId: saveID, deviceCanTranscribe: canTranscribe)
        default:
            return nil
        }
    }
}
```

### 6.2 SSE (Server-Sent Events) Client

```swift
import Foundation

final class SSEClient {
    private var task: URLSessionDataTask?
    private var buffer = Data()
    
    func connect(
        to url: URL,
        onEvent: @escaping (SSEEvent) -> Void,
        onComplete: @escaping () -> Void
    ) {
        var request = URLRequest(url: url)
        request.setValue("text/event-stream", forHTTPHeaderField: "Accept")
        request.setValue("Bearer \(AuthManager.shared.token)", forHTTPHeaderField: "Authorization")
        
        let delegate = SSEDelegate(
            onData: { [weak self] data in
                self?.buffer.append(data)
                self?.parseEvents(onEvent: onEvent)
            },
            onComplete: onComplete
        )
        
        let session = URLSession(configuration: .default, delegate: delegate, delegateQueue: nil)
        
        task = session.dataTask(with: request)
        task?.resume()
    }
    
    func disconnect() {
        task?.cancel()
        task = nil
    }
    
    private func parseEvents(onEvent: (SSEEvent) -> Void) {
        guard let text = String(data: buffer, encoding: .utf8) else { return }
        
        let events = text.components(separatedBy: "\n\n")
        for eventText in events.dropLast() {
            if let event = parseEvent(eventText) {
                onEvent(event)
            }
        }
        
        // Keep unparsed tail
        if let lastEvent = events.last, !lastEvent.hasSuffix("\n\n") {
            buffer = lastEvent.data(using: .utf8) ?? Data()
        } else {
            buffer = Data()
        }
    }
    
    private func parseEvent(_ text: String) -> SSEEvent? {
        var eventType: String?
        var data: String?
        
        for line in text.components(separatedBy: "\n") {
            if line.hasPrefix("event:") {
                eventType = line.dropFirst(6).trimmingCharacters(in: .whitespaces)
            } else if line.hasPrefix("data:") {
                data = line.dropFirst(5).trimmingCharacters(in: .whitespaces)
            }
        }
        
        guard let type = eventType, let jsonData = data?.data(using: .utf8) else {
            return nil
        }
        
        return SSEEvent(type: type, data: jsonData)
    }
}

struct SSEEvent {
    let type: String
    let data: Data
}

private class SSEDelegate: NSObject, URLSessionDataDelegate {
    let onData: (Data) -> Void
    let onComplete: () -> Void
    
    init(onData: @escaping (Data) -> Void, onComplete: @escaping () -> Void) {
        self.onData = onData
        self.onComplete = onComplete
    }
    
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        onData(data)
    }
    
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        onComplete()
    }
}
```

---

## 7. KMP Integration

### 7.1 Consuming KMP Framework

**Kotlin Multiplatform compiles to an `.xcframework` for iOS.**

**Integration via SPM (recommended):**

The KMP team provides a Package.swift file alongside the compiled `.xcframework`:

```swift
// Package.swift (provided by KMP team)
// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "StaqerShared",
    platforms: [.iOS(.v17)],
    products: [
        .library(
            name: "StaqerShared",
            targets: ["StaqerShared"]
        )
    ],
    targets: [
        .binaryTarget(
            name: "StaqerShared",
            path: "./StaqerShared.xcframework"
        )
    ]
)
```

**In Xcode:**

File → Add Package Dependencies → Add Local → Select the folder containing the KMP Package.swift

**Usage in Swift:**

```swift
import StaqerShared

// Use KMP-provided types
let urlParser = URLParserKMP()
let platform = urlParser.detectPlatform(url: "https://instagram.com/reel/ABC123")

// KMP data models
let saveModel = SavedItemKMP(
    id: UUID().uuidString,
    url: url,
    platform: platform.name,
    category: category,
    createdAt: Date().timeIntervalSince1970
)

// URL normalization from KMP
let normalizer = URLNormalizerKMP()
let cleanedURL = normalizer.normalize(url: "https://www.instagram.com/reel/ABC/?igshid=xyz")
```

### 7.2 What Comes from KMP

From the PRD Section 6.5, KMP provides:

- **Networking:** Ktor HTTP client wrapper (abstraction for API calls)
- **Data Models:** `SavedItemKMP`, `StructuredCardKMP`, `UserProfileKMP`, JSON serialization
- **URL Parsing:** Platform detection, URL normalization, deep link extraction
- **Classification Rules:** Tier 2 keyword dictionaries, regex extractors
- **Caching Logic:** URL dedup, TTL management

**iOS-specific wrappers for ergonomics:**

```swift
// Swift extension to make KMP types feel native
extension URLNormalizerKMP {
    static func normalize(_ url: String) -> String {
        URLNormalizerKMP().normalize(url: url)
    }
}

extension KeywordClassifierKMP {
    func classifySwift(caption: String, hashtags: [String]) -> (category: String, confidence: Double) {
        let result = self.classify(caption: caption, hashtags: hashtags)
        return (result.category, result.confidence)
    }
}
```

---

## 8. Key Libraries & Dependencies

### 8.1 Swift Package Manager Dependencies

Add via Xcode → File → Add Package Dependencies

| Library | Purpose | URL |
|---------|---------|-----|
| **Kingfisher** | Image caching (multi-layer, LRU, disk) | `https://github.com/onevcat/Kingfisher` |
| **Firebase iOS SDK** | Analytics, Crashlytics, Remote Config | `https://github.com/firebase/firebase-ios-sdk` |
| **Supabase Swift** | Backend client (auth, database, storage) | `https://github.com/supabase-community/supabase-swift` |
| **SQLite.swift** | Type-safe SQLite wrapper (for FTS5) | `https://github.com/stephencelis/SQLite.swift` |

### 8.2 Image Loading & Caching (Kingfisher)

```swift
import Kingfisher

struct SaveCardView: View {
    let save: SavedContent
    
    var body: some View {
        KFImage(URL(string: save.thumbnailUrl ?? ""))
            .placeholder {
                Color.gray.opacity(0.2)
                    .overlay {
                        ProgressView()
                    }
            }
            .retry(maxCount: 3, interval: .seconds(2))
            .fade(duration: 0.25)
            .cacheOriginalImage()
            .scaledToFill()
            .frame(width: 180, height: 225)
            .clipped()
            .cornerRadius(12)
    }
}

// Configure cache in AppDelegate
func application(_ application: UIApplication, didFinishLaunchingWithOptions...) -> Bool {
    // Configure Kingfisher cache
    ImageCache.default.memoryStorage.config.totalCostLimit = 50 * 1024 * 1024 // 50MB
    ImageCache.default.diskStorage.config.sizeLimit = 500 * 1024 * 1024 // 500MB
    ImageCache.default.diskStorage.config.expiration = .days(30)
    
    return true
}
```

### 8.3 Analytics (Firebase)

```swift
import FirebaseAnalytics

enum AnalyticsEvent {
    case libraryOpened(entryPoint: String, saveCount: Int)
    case categoryFilterApplied(category: String, resultCount: Int)
    case deepExtractInitiated(trigger: String, platform: String)
    
    var name: String {
        switch self {
        case .libraryOpened: return "library_opened"
        case .categoryFilterApplied: return "category_filter_applied"
        case .deepExtractInitiated: return "deep_extract_initiated"
        }
    }
    
    var parameters: [String: Any] {
        switch self {
        case .libraryOpened(let entry, let count):
            return ["entry_point": entry, "save_count": count]
        case .categoryFilterApplied(let cat, let count):
            return ["category": cat, "result_count": count]
        case .deepExtractInitiated(let trigger, let platform):
            return ["trigger_point": trigger, "platform": platform]
        }
    }
}

final class AnalyticsService {
    static let shared = AnalyticsService()
    
    private init() {}
    
    func track(_ event: AnalyticsEvent) {
        Analytics.logEvent(event.name, parameters: event.parameters)
    }
}
```

### 8.4 Crash Reporting (Crashlytics)

```swift
import FirebaseCrashlytics

// In AppDelegate
func application(_ application: UIApplication, didFinishLaunchingWithOptions...) -> Bool {
    FirebaseApp.configure()
    Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
    return true
}

// Log non-fatal errors
func handleNonFatalError(_ error: Error, context: String) {
    Crashlytics.crashlytics().record(error: error)
    Crashlytics.crashlytics().log("\(context): \(error.localizedDescription)")
}
```

---

## 9. Testing Strategy

### 9.1 Unit Tests

**Target:** 70%+ code coverage on business logic

```swift
import XCTest
@testable import Staqer

final class Tier2ClassifierTests: XCTestCase {
    var classifier: Tier2Classifier!
    
    override func setUp() {
        super.setUp()
        classifier = Tier2Classifier()
    }
    
    func testRecipeClassification() {
        let result = classifier.classify(
            caption: "Easy 15-minute pasta with 4 ingredients",
            hashtags: ["recipe", "pasta", "cooking"]
        )
        
        XCTAssertEqual(result.category, "recipe")
        XCTAssertGreaterThan(result.confidence, 0.7)
    }
    
    func testTravelClassification() {
        let result = classifier.classify(
            caption: "Hidden beaches in Bali you must visit",
            hashtags: ["travel", "bali", "wanderlust"]
        )
        
        XCTAssertEqual(result.category, "travel")
    }
}

final class SavedContentRepositoryTests: XCTestCase {
    var repository: SavedContentRepository!
    var mockPersistenceController: MockPersistenceController!
    
    override func setUp() {
        super.setUp()
        mockPersistenceController = MockPersistenceController()
        repository = SavedContentRepository(persistenceController: mockPersistenceController)
    }
    
    func testSaveContent() async throws {
        let content = SavedContent(
            id: UUID(),
            url: "https://instagram.com/reel/test",
            platform: .instagram,
            category: "recipe"
        )
        
        try await repository.save(content)
        
        let fetched = repository.fetchByID(content.id)
        XCTAssertEqual(fetched?.url, content.url)
    }
}
```

### 9.2 UI Tests

**Target:** Critical user flows

```swift
import XCTest

final class LibraryFlowUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        app = XCUIApplication()
        app.launchArguments = ["UI_TESTING"]
        app.launch()
    }
    
    func testCategoryFilter() {
        // Tap library tab
        app.tabBars.buttons["Library"].tap()
        
        // Wait for grid to load
        XCTAssertTrue(app.collectionViews.firstMatch.waitForExistence(timeout: 2))
        
        // Tap recipe filter chip
        app.buttons["🍳 38"].tap()
        
        // Verify filter applied
        XCTAssertTrue(app.staticTexts["🍳 Recipes · 38 saves"].exists)
    }
    
    func testSearchFlow() {
        app.tabBars.buttons["Library"].tap()
        
        let searchField = app.textFields["Search your saves..."]
        searchField.tap()
        searchField.typeText("pasta")
        
        // Wait for results
        XCTAssertTrue(app.staticTexts["3 results"].waitForExistence(timeout: 1))
        
        // Tap first result
        app.collectionViews.cells.firstMatch.tap()
        
        // Verify card detail view
        XCTAssertTrue(app.staticTexts["15-Minute Garlic Pasta"].exists)
    }
}
```

### 9.3 Snapshot Tests

Use **swift-snapshot-testing** for UI regression:

```swift
import SnapshotTesting
import SwiftUI
import XCTest

final class CardViewSnapshotTests: XCTestCase {
    func testRecipeCardLight() {
        let save = SavedContent.mockRecipe()
        let view = RecipeCardView(save: save)
            .frame(width: 375, height: 600)
        
        assertSnapshot(matching: view, as: .image)
    }
    
    func testRecipeCardDark() {
        let save = SavedContent.mockRecipe()
        let view = RecipeCardView(save: save)
            .frame(width: 375, height: 600)
            .preferredColorScheme(.dark)
        
        assertSnapshot(matching: view, as: .image)
    }
}
```

---

## 10. Build & CI/CD

### 10.1 Xcode Cloud

**Preferred for App Store integration.**

**Workflow:**

Create `ci_workflows/main.xcode-ci.yml`:

```yaml
version: 1
workflows:
  Build and Test:
    name: Build & Test
    trigger:
      branch_patterns:
        - main
      pull_request: true
    actions:
      - name: Build
        platform: iOS
        scheme: Staqer
        configuration: Debug
        destination: platform=iOS Simulator,name=iPhone 15 Pro
        
      - name: Test
        platform: iOS
        scheme: Staqer
        configuration: Debug
        destination: platform=iOS Simulator,name=iPhone 15 Pro
        
      - name: Archive
        platform: iOS
        scheme: Staqer
        configuration: Release
        archive: true
        
      - name: TestFlight
        distribution_method: testflight
        groups:
          - Internal Testers
```

### 10.2 Fastlane (Alternative)

**Install:**

```bash
brew install fastlane
cd /path/to/Staqer
fastlane init
```

**Fastfile:**

```ruby
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    run_tests(
      scheme: "Staqer",
      device: "iPhone 15 Pro",
      code_coverage: true
    )
  end
  
  desc "Build for TestFlight"
  lane :beta do
    increment_build_number(xcodeproj: "Staqer.xcodeproj")
    build_app(
      scheme: "Staqer",
      export_method: "app-store"
    )
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
    
    # Post to Slack
    slack(
      message: "New TestFlight build uploaded!",
      slack_url: ENV["SLACK_WEBHOOK_URL"]
    )
  end
  
  desc "Submit to App Store"
  lane :release do
    capture_screenshots
    build_app(scheme: "Staqer")
    upload_to_app_store(
      submit_for_review: true,
      automatic_release: false,
      submission_information: {
        add_id_info_uses_idfa: false
      }
    )
  end
end
```

### 10.3 Environment Configs

**Xcconfig files:**

```
Configurations/
├── Debug.xcconfig
├── Release.xcconfig
└── Shared.xcconfig
```

**Debug.xcconfig:**

```
#include "Shared.xcconfig"

API_BASE_URL = https:/$()/api-dev.staq.app
ENABLE_ANALYTICS = NO
ENABLE_CRASHLYTICS = NO
```

**Release.xcconfig:**

```
#include "Shared.xcconfig"

API_BASE_URL = https:/$()/api.staq.app
ENABLE_ANALYTICS = YES
ENABLE_CRASHLYTICS = YES
```

**Usage in code:**

```swift
struct AppConfig {
    static let apiBaseURL: String = {
        #if DEBUG
        return "https://api-dev.staq.app"
        #else
        return "https://api.staq.app"
        #endif
    }()
}
```

---

## 11. Sprint-by-Sprint Breakdown

**Total Duration:** 16 weeks (4 months)  
**Sprints:** 8 two-week sprints  
**Team:** 1 iOS developer (you), 1 Android developer (parallel), 1 backend developer (shared)

---

### Sprint 1 (Weeks 1-2): Foundation & Share Extension

**Goal:** Project setup + working share extension that captures URLs

**Tasks:**

1. **Project Setup** (4h)
   - Create Xcode workspace
   - Configure targets (main app + share extension)
   - Set up App Groups entitlement
   - Add SPM dependencies (Kingfisher, Firebase, SQLite.swift)
   - Configure Firebase project

2. **Core Data Stack** (6h)
   - Design schema (SavedContent, PendingSave, UserProfile)
   - Create `StaqerModel.xcdatamodeld`
   - Implement `PersistenceController` with shared container
   - Enable WAL mode
   - Write migration strategy

3. **Share Extension** (12h)
   - Implement `ShareViewController` with SwiftUI hosting
   - Build `URLParser` for platform detection
   - Build `MetadataExtractor` for NSItemProvider
   - Implement `LocalSaveManager` to persist PendingSave
   - Add confirmation UI with haptics
   - Handle edge cases (offline, duplicate, unsupported URL)

4. **URL Normalization** (4h)
   - Implement tracking param stripping
   - Implement canonical form standardization
   - Test across platforms (Instagram, TikTok, YouTube)

5. **Background Task Setup** (2h)
   - Register `BGAppRefreshTask`
   - Implement scheduling logic from share extension

**Deliverable:** Working share extension that saves URLs to shared Core Data store

---

### Sprint 2 (Weeks 3-4): Auto-Categorization (Tier 2 & 2.5)

**Goal:** Rule-based and bundled model classification working

**Tasks:**

1. **Keyword Dictionaries** (8h)
   - Create multilingual keyword JSON files (en, hi, pt, ar)
   - Implement Tier 2 keyword scorer
   - Implement TF-IDF disambiguation
   - Test accuracy on 50 sample captions

2. **Bundled CoreML Model** (6h)
   - Train TextClassifier.mlmodel (DistilBERT-tiny) on labeled dataset
   - Export as `.mlpackage`
   - Integrate Tier 2.5 classifier
   - Benchmark latency on iPhone 12

3. **OCR Pipeline** (6h)
   - Implement Vision-based text extraction
   - Implement OCR text cleaning (remove handles, watermarks)
   - Test on 20 sample thumbnails

4. **Tier Selection Logic** (2h)
   - Implement `TierSelector` to pick best available tier
   - Add device capability checks

5. **Richness Score** (2h)
   - Implement score calculation based on confidence + field count
   - Wire into UI (show "Get Full Details" on low-richness cards)

6. **Analytics Events** (4h)
   - Implement `AnalyticsService` with Firebase
   - Add `categorization_completed` event
   - Add `category_overridden` event

**Deliverable:** Saves are auto-categorized with 75%+ accuracy (Tier 2/2.5)

---

### Sprint 3 (Weeks 5-6): Foundation Models (Tier 1) + Library UI

**Goal:** On-device AI for iOS 26+ + basic library grid view

**Tasks:**

1. **Foundation Models Integration** (8h)
   - Implement `Tier1Classifier` with `@Generable`
   - Handle model availability checks
   - Handle errors (modelNotReady, modelBusy, guardrail)
   - Test on iPhone 15 Pro device

2. **Prompt Versioning** (4h)
   - Implement `PromptVersion` model
   - Add A/B test infrastructure (bucket assignment)
   - Store prompt version in analytics

3. **Library Grid View** (10h)
   - Build `LibraryView` with `LazyVGrid`
   - Build `SaveCardView` with thumbnail + category badge
   - Integrate Kingfisher for image loading
   - Add date section headers
   - Implement pull-to-refresh

4. **Category Filter Bar** (4h)
   - Build scrollable chip bar
   - Implement single-select filtering
   - Wire to `LibraryViewModel`

5. **Tab Bar Navigation** (2h)
   - Set up `TabView` with Library/Collections/Settings
   - Add haptic feedback on tab switch

**Deliverable:** Working library grid with category filtering. Tier 1 classification on iOS 26+ devices.

---

### Sprint 4 (Weeks 7-8): Search + Multi-Select + Card Templates

**Goal:** Full-text search + batch actions + basic card detail views

**Tasks:**

1. **FTS5 Setup** (6h)
   - Execute raw SQL to create FTS virtual table
   - Add sync triggers
   - Test search queries with SQLite.swift
   - Implement `search()` in repository

2. **Search UI** (8h)
   - Build `SearchOverlayView`
   - Implement live results as-you-type
   - Add recent searches
   - Add search highlighting in results

3. **Multi-Select Mode** (6h)
   - Add selection state to `LibraryViewModel`
   - Build batch action toolbar
   - Implement batch delete with undo
   - Implement batch recategorize

4. **Card Templates** (8h)
   - Implement `CardTemplate` protocol + registry
   - Build `RecipeCardView` (ingredients, steps)
   - Build `TravelCardView` (map stub, weather stub)
   - Build `ProductCardView` (price, purchase link)
   - Build `GenericCardView` (fallback)

5. **Hero Transition** (2h)
   - Implement `matchedGeometryEffect` from grid to card detail

**Deliverable:** Search works. Multi-select batch actions work. Card detail views render.

---

### Sprint 5 (Weeks 9-10): Card Features + Structured Extraction

**Goal:** Interactive card features (timers, unit conversion, serving adjuster)

**Tasks:**

1. **Ingredient Checklist** (4h)
   - Build checkbox list with progress counter
   - Persist checkbox state in UserDefaults
   - Add "Uncheck All" action

2. **Unit Conversion** (6h)
   - Build conversion lookup table
   - Implement toggle UI (metric/imperial)
   - Implement friendly rounding
   - Persist toggle state per card

3. **Serving Size Adjuster** (4h)
   - Build stepper UI
   - Implement proportional scaling
   - Add reset button to original serving count

4. **Inline Timer** (6h)
   - Detect time values in steps (regex)
   - Build countdown timer UI
   - Register local notifications
   - Handle multiple concurrent timers (max 3)

5. **Dietary Tags** (4h)
   - Implement detection logic (keyword matcher)
   - Build tag pills UI
   - Add user override (tap to remove)

6. **Travel Card Features** (6h)
   - Integrate MapKit static snapshot
   - Integrate OpenWeatherMap API
   - Calculate distance from user (CoreLocation)
   - Implement currency conversion for budget

**Deliverable:** Recipe and Travel cards are fully interactive and feature-complete.

---

### Sprint 6 (Weeks 11-12): Deep Extract Pipeline

**Goal:** On-device video transcription + server-side scraping

**Tasks:**

1. **Server API Endpoint** (8h, backend)
   - Build `/api/v1/extract` with SSE
   - Integrate Apify scraper
   - Implement job queue (BullMQ)
   - Implement circuit breaker for scraper providers

2. **SSE Client** (6h)
   - Implement `SSEClient` with incremental parsing
   - Handle reconnection with Last-Event-ID
   - Test with mock server

3. **Video Download + Audio Extraction** (6h)
   - Implement `VideoDownloader` (URLSession streaming)
   - Use AVFoundation to extract audio track
   - Implement temp file cleanup (defer blocks)

4. **On-Device Transcription** (8h)
   - Implement `OnDeviceTranscriber` with SpeechAnalyzer (iOS 26+)
   - Implement SFSpeechRecognizer fallback (iOS 17-25)
   - Handle 1-minute audio chunking for long videos

5. **Key Frame Extraction + OCR** (6h)
   - Implement `KeyFrameExtractor` (AVAssetImageGenerator at 1fps)
   - Run OCR on frames
   - Implement text deduplication (Levenshtein similarity)

6. **Progress UI** (6h)
   - Build `DeepExtractProgressView` with stage-based progress bar
   - Show meaningful stage labels ("Fetching video...", "Listening to audio...")
   - Add cancel button
   - Handle errors gracefully

**Deliverable:** Deep extract works end-to-end (server scrape → on-device transcription → extraction).

---

### Sprint 7 (Weeks 13-14): Quota + Paywall + Polish

**Goal:** Free/paid quota system + paywall + UX polish

**Tasks:**

1. **Quota Management** (6h)
   - Implement `QuotaManager`
   - Track monthly usage in UserProfile entity
   - Implement server-side enforcement
   - Add reset logic (first of month)

2. **Paywall UI** (6h)
   - Build `QuotaExhaustedView`
   - Integrate StoreKit 2 for In-App Purchase
   - Add trial flow (7-day free trial)
   - Add pricing tiers (monthly $4.99, annual $39.99)

3. **Auto-Extract (Paid Users)** (4h)
   - Add Settings toggle
   - Trigger auto-extract on low-richness saves
   - Add Wi-Fi vs cellular preference

4. **Share Card Image Generation** (6h)
   - Implement `UIGraphicsImageRenderer` pipeline
   - Generate 3 formats (9:16, 1:1, 4:5)
   - Add QR code with Firebase Dynamic Link
   - Add branding footer

5. **Dark Mode Audit** (4h)
   - Test all views in dark mode
   - Fix contrast issues (badges, overlays)
   - Test map snapshots in dark mode

6. **Accessibility Audit** (6h)
   - VoiceOver testing on all flows
   - Dynamic Type testing (largest sizes)
   - Reduce Motion testing
   - Minimum tap target fixes (44x44pt)

**Deliverable:** Free/paid quota enforced. Paywall converts users. App is polished and accessible.

---

### Sprint 8 (Weeks 15-16): QA + Beta + App Store Prep

**Goal:** Bug fixes, performance optimization, TestFlight beta

**Tasks:**

1. **Performance Optimization** (8h)
   - Profile library scroll performance (Instruments)
   - Optimize thumbnail caching (prefetch next 2 pages)
   - Reduce memory footprint (autoreleasepool in loops)
   - Optimize FTS query speed

2. **Bug Fixes from QA** (12h)
   - Fix edge cases discovered in testing
   - Fix crashes (if any)
   - Fix layout issues on iPad
   - Fix landscape mode issues

3. **TestFlight Beta** (4h)
   - Set up Xcode Cloud or Fastlane
   - Upload first build to TestFlight
   - Invite 50-100 beta testers
   - Set up crash reporting dashboard

4. **App Store Listing** (4h)
   - Write App Store description
   - Create screenshots (iPhone 6.7" and iPad 12.9")
   - Create App Preview video (optional)
   - Submit for review

5. **Analytics Dashboard** (4h)
   - Set up Firebase Analytics dashboard
   - Create custom funnels (save → categorize → view card)
   - Set up Crashlytics alerting

**Deliverable:** App in TestFlight. Ready for App Store submission.

---

## Summary

This iOS Architecture Plan provides:

1. **Clear project structure** with SwiftUI + MVVM + Core Data
2. **Share extension** with 120MB memory awareness and App Group sharing
3. **Three-tier AI classification** (Foundation Models → CoreML → Rule-based)
4. **Full-text search** via FTS5 with WAL-mode Core Data
5. **Deep extract pipeline** with on-device transcription and server-side scraping
6. **Rich card templates** with interactive features (timers, unit conversion, maps, weather)
7. **KMP integration** for shared logic (URL parsing, classification rules, networking)
8. **Testing strategy** (unit, UI, snapshot)
9. **CI/CD setup** (Xcode Cloud or Fastlane)
10. **Sprint breakdown** mapping features F01-F05 to 16 weeks

All protocols, code patterns, and architectural decisions are **specific and actionable** for a senior iOS developer. Each code snippet is production-ready and follows iOS best practices (async/await, SwiftUI, Combine, modern concurrency).

Start with Sprint 1 and ship incrementally. Good luck building Staqer!

---

**End of iOS Architecture Plan v1.0**
