# Android (Kotlin) Clean Architecture & Coding Standard

Load this file ONLY when working on **Android / Kotlin** projects.

## 1. Feature Folder Structure

```text
feature/<feature_name>/
├── di/                         # Hilt @Module or Koin module (Data -> Domain -> Presentation)
├── util/                       # <Feature>Constants.kt (documented object constants; cross-feature -> core/util/)
├── helpers/                    # Pure Kotlin feature-local algorithms & formatters (cross-feature -> utils/)
├── data/                       # Omit when no API / DB / local storage
│   ├── datasource/             # ApiService / RemoteDataSource / LocalDataSource
│   ├── model/                  # Optional DTOs (@SerializedName / @Serializable) ONLY when API schema diverges
│   └── repository/             # *RepositoryImpl (pure transport + 1-to-1 mapping only)
├── domain/
│   ├── model/                  # Single domain models (pure Kotlin; only data/ calls serialization)
│   ├── repository/             # Repository interfaces
│   └── usecase/                # *UseCase + *ErrorUseCase + sealed interface *Outcome (pure Kotlin ONLY)
└── presentation/
    ├── viewmodel/              # *ViewModel.kt + *UiState.kt (StateFlow / immutable sealed interface)
    ├── navigation/             # NavController / Router ONLY (optional — NEVER put ViewModel or UiState here)
    ├── services/               # Android SDK wrappers, sensor/camera/biometric managers (optional)
    ├── helpers/                # Pure stateless UI formatters & style helpers (optional)
    └── ui/                     # *Screen.kt / *Fragment.kt + components/
```

## 2. Layer Boundaries & Dependencies

- **Domain (`domain/`):** Pure Kotlin only. **NEVER** import `android.*`, `androidx.*`, Compose, Retrofit/Ktor, or Room.
- **Data (`data/`):** May import Retrofit/Ktor, `@Serializable`/`Gson`, Room, DataStore, and crypto libs. Never import Compose/View/Context UI APIs.
- **Presentation (`presentation/`):**
  - **Device / UI SDKs** (BiometricPrompt, CameraX, WebView, ExoPlayer, Lottie, Lifecycle Observers): Import **ONLY in `presentation/`** — wrap in `presentation/services/` or `components/`, never scatter raw SDK calls inside `@Composable` or `onCreateView`. Pass only pure Kotlin values into `ViewModel -> UseCase`.
  - **Thin ViewModel:** Only holds/emits `StateFlow<*UiState>`, forwards 1 user intent to 1 `*UseCase`, and uses an exhaustive `when (outcome)` to emit state. Never inspect HTTP status/error codes, access repositories/DataStore, or chain multi-step conditional UseCases in `ViewModel`.
  - **Self-Contained UseCase:** Orchestrate multi-step flows inside `*UseCase` and return an explicit `sealed interface *Outcome` (never raw HTTP `Response` or ambiguous `Boolean`). Max 2 UseCase call levels (`Orchestrator -> Atomic`), no circular dependencies. Keep `operator fun invoke(...)` ~25–35 lines as a step-by-step Table of Contents.

## 3. Error, Dialog, Snackbar & Inline Convention

1. **Domain (`*UseCase` + `*ErrorUseCase`):** Classify error codes/backend feedback into explicit `sealed interface *Outcome` variants with localized messages/keys.
2. **ViewModel (`*ViewModel.kt`):** `when (outcome)` -> update `MutableStateFlow<*UiState>`.
3. **Screen / Fragment (`*Screen.kt` / `*Fragment.kt`):** Collect `UiState` with a flat `when` to trigger `AlertDialog` (blocking/confirmation), `Snackbar`/`Toast` (transient), or `Inline` error UI. Route dialog actions back to `ViewModel -> UseCase`.

## 4. Single-Class View & `// region` Sections

Keep UI rendering and UI handlers in the same main `*Screen.kt` / `*Fragment.kt` under 4 ordered region markers:
1. `// region Properties & Dependencies`
2. `// region Lifecycle`
3. `// region UI Rendering`
4. `// region UI Handlers & Listeners`

Pure stateless `@Composable` components under `presentation/ui/components/` are exempt when they have no lifecycle or imperative handlers.

## 5. Control Flow, Compose Performance & Secondary Entry Points

- **Control Flow:** Max 2 levels of `if/else` nesting. Prefer early returns, exhaustive `when`, and small private helpers.
- **Jetpack Compose Performance & Structure:**
  - Collect `StateFlow` using `collectAsStateWithLifecycle()` instead of `collectAsState()`.
  - **Never pass `ViewModel`** into reusable child composables (`components/`); pass immutable state and callback lambdas (`onAction: (...) -> Unit`) instead.
  - Keep state properties `val` (consider `@Immutable` / `@Stable` when needed), and use `remember` / `derivedStateOf` for expensive calculations or high-frequency state (e.g., scroll offset).
  - Standard `@Composable` parameter order: `modifier: Modifier = Modifier` -> required state/data -> optional styling -> callbacks (`onClick`, ...) -> trailing `@Composable` content. A composable should either emit UI or return a value, never both.
- **Tiered Scope:**
  - *Full Feature:* `data/` + `domain/` + `presentation/` + `di/`.
  - *Logic / Device-SDK Feature (no API/DB):* `domain/` + `presentation/` (+ `di/`); omit `data/`.
  - *Pure UI / Static Screen:* `presentation/ui/` only; omit `domain/`, `data/`, `di/`.
- **Secondary Entry Points (`GlanceAppWidget`, `AppWidgetProvider`, `Service`, `BroadcastReceiver`):**
  - Treat as `Presentation` entry points under `presentation/widget/` or `presentation/receiver/` (`Actions/`, `UI/`, `Services/`).
  - Keep action callbacks thin: only invoke `DI -> UseCase` or `presentation/services/`, never embed business logic or access `DataSource` directly.

## 6. Verification

Always run before finishing:
- `./gradlew lint` (or module lint task)
- `./gradlew test` (include headless JUnit tests for `*UseCase` business logic)
