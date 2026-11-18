# iOS (Swift / SwiftUI / UIKit) Clean Architecture & Coding Standard

Load this file ONLY when working on **iOS / macOS / watchOS Swift** projects.

## 1. Feature Folder Structure

```text
Features/<FeatureName>/
├── DI/                         # <FeatureName>DI.swift (enum factory/container: Data -> Domain -> Presentation)
├── Helpers/                    # Pure Swift feature-local enums, constants, formatters (cross-feature -> Shared/Helpers/)
├── Data/                       # Omit when no API / DB / local storage
│   ├── DataSources/
│   ├── Models/                 # Optional Codable DTOs ONLY when API schema diverges
│   └── Repositories/           # *RepositoryImpl.swift (pure transport + 1-to-1 mapping only)
├── Domain/
│   ├── Entities/               # Single domain models (pure Foundation; only Data/ calls Codable serialization)
│   ├── Repositories/           # Repository protocols
│   └── UseCases/               # *UseCase.swift + *ErrorUseCase.swift + enum *Outcome (pure Foundation ONLY)
└── Presentation/
    ├── ViewModel/              # *ViewModel.swift + *UIState.swift (immutable enum, exhaustive switch)
    ├── Coordinator/            # Navigation / Router ONLY (optional — NEVER put ViewModel or UIState here)
    ├── Services/               # Device/OS SDK wrappers, AVAudio/Biometric/Camera observers (optional)
    ├── Helpers/                # Pure stateless UI formatters & style helpers (optional)
    └── UI/                     # *View.swift / *ViewController.swift + Components/
```

## 2. Naming Convention & Class Naming

### 2.1 General Naming Rules (Plain & Familiar English)
- **Files & Folders:** `PascalCase.swift` (e.g., `TransferView.swift`, `GetAccountListUseCase.swift`).
- **Types (Class / Struct / Enum / Protocol):** `PascalCase` (e.g., `TransferViewModel`, `GetAccountListUseCase`).
- **Properties / Functions / Parameters:** `camelCase` (e.g., `accountList`, `submitTransfer()`).
- **Booleans:** Always prefix with `is`, `has`, `can`, or `should` (e.g., `isLoading`, `hasError`, `canSubmit`).
- **Access Control:** Mark internal UI/helper methods `private` (e.g., `private func handleSubmitTap()`).
- **Plain, Everyday Developer English:** Avoid cryptic abbreviations (`UsrMgr`, `TxnRes`, `BalUC`, `InfoCrd`) and overly academic words. Prefer familiar verbs:
  - Read: `get`, `fetch`, `load` *(avoid `retrieve`, `acquire`, `procure`)*
  - Write: `save`, `create`, `update`, `delete`, `remove`, `clear` *(avoid `persist`, `purge`)*
  - Check: `check`, `verify`, `validate` *(avoid `ascertain`)*
  - Execute: `send`, `submit`, `start`, `stop`, `run` *(avoid `dispatch` unless event-bus)*
  - Transform / UI: `map`, `parse`, `format`, `show`, `hide`, `open`, `close`

### 2.2 Type Naming by Layer

| Layer | Component | Type Naming Rule | Example |
|---|---|---|---|
| **Domain** | Entity | `<Noun>` or `<Noun>Entity` | `Account`, `TransferInfo` |
| | Repository Protocol | `<Feature>Repository` | `TransferRepository` |
| | UseCase | `<Verb><Noun>UseCase` | `GetAccountListUseCase`, `SubmitTransferUseCase` |
| | Error UseCase | `<Feature>ErrorUseCase` | `TransferErrorUseCase` |
| | Outcome | `<Action>Outcome` (`enum`) | `TransferOutcome` (`case success(...)`, `case failure(...)`) |
| **Data** | Repository Impl | `<Feature>RepositoryImpl` | `TransferRepositoryImpl` |
| | DataSource | `<Feature>RemoteDataSource`, `<Feature>LocalDataSource` | `TransferRemoteDataSource` |
| | DTO (when needed) | `<Noun>DTO` or `<Noun>Request` / `<Noun>Response` | `TransferRequest`, `TransferResponse` |
| **DI** | Dependency Container | `<Feature>DI` (`enum`) | `TransferDI` |
| **Presentation** | ViewModel | `<Feature>ViewModel` | `TransferViewModel` |
| | UIState | `<Feature>UIState` (`enum`) | `TransferUIState` (`case initial`, `loading`, `loaded`, `error`) |
| | Main View / VC | `<Feature>View` / `<Feature>ViewController` | `TransferView`, `TransferViewController` |
| | Child Sub-View | `<Noun><Type>` (`View`, `Card`, `Row`, `Sheet`, `Alert`) | `AccountCard`, `ConfirmTransferSheet` |
| | Coordinator (Nav) | `<Feature>Coordinator` | `TransferCoordinator` |
| | Service (SDK Wrapper) | `<Capability>Service` | `BiometricService`, `CameraService` |

## 3. Layer Boundaries & Dependencies

- **Domain (`Domain/`):** Pure `Foundation` only. **NEVER** import `SwiftUI`, `UIKit`, `WidgetKit`, `AppIntents`, `AVFoundation`, or networking/storage frameworks.
- **Data (`Data/`):** May import `URLSession`/networking, `Codable`, `SwiftData`/`CoreData`, `Keychain`, and crypto libs. Never import `SwiftUI` or `UIKit`.
- **Presentation (`Presentation/`):**
  - **Device / OS SDKs** (LocalAuthentication, AVFoundation, PhotosUI, WebKit, CoreLocation, OS Observers): Import **ONLY in `Presentation/`** — wrap in `Presentation/Services/` or `Components/`, never scatter raw SDK calls inside SwiftUI `body`. Pass only pure Swift values into `ViewModel -> UseCase`.
  - **Thin ViewModel:** Only holds/publishes immutable `*UIState`, forwards 1 user intent to 1 `*UseCase`, and uses an exhaustive `switch outcome` to emit state. Never inspect HTTP status/error codes, access repositories/Keychain/UserDefaults, or chain multi-step conditional UseCases in `ViewModel`.
  - **Self-Contained UseCase:** Orchestrate multi-step flows inside `*UseCase` and return an explicit `enum *Outcome` (never raw `HTTPURLResponse` or ambiguous `Bool`). Max 2 UseCase call levels (`Orchestrator -> Atomic`), no circular dependencies. Keep the public `execute(...)` method ~25–35 lines as a step-by-step Table of Contents.

## 4. Error, Alert, Toast & Inline Convention

1. **Domain (`*UseCase` + `*ErrorUseCase`):** Classify error codes/backend feedback into explicit `enum *Outcome` cases with localized messages.
2. **ViewModel (`*ViewModel.swift`):** `switch outcome` -> update `*UIState`.
3. **View (`*View.swift` / `*ViewController.swift`):** Observe `UIState` with a flat `switch` to trigger `.alert` (blocking/confirmation), Toast/Banner (transient), or `Inline` feedback. Route alert actions back to `ViewModel -> UseCase`.

## 5. Single-Class View & `// MARK: -` Sections

Keep UI rendering and UI handlers in the same main `*View.swift`, `*ViewController.swift`, or `*WidgetEntryView.swift` under 4 ordered section markers:
1. `// MARK: - Properties & Dependencies`
2. `// MARK: - Lifecycle`
3. `// MARK: - UI Rendering`
4. `// MARK: - UI Handlers & Listeners`

Pure stateless sub-views (`Components/*Card.swift`) and Widget family layouts (`Small*View`, `Medium*View`) are exempt when they have no lifecycle or imperative handlers.

## 6. Control Flow, SwiftUI Performance, Tiered Scope & Extensions (`WidgetKit`, `AppIntents`, `Watch`)

- **Control Flow:** Max 2 levels of `if/else` nesting. Prefer `guard` early returns, exhaustive `switch`, and small private helpers.
- **SwiftUI Performance, Concurrency & Accessibility:**
  - Use `LazyVStack` / `LazyHStack` for large or dynamic lists, and `VStack` / `HStack` for small static groups.
  - Always use `Identifiable` or stable `id: \.id` in `ForEach` (avoid `id: \.self` for mutable collections).
  - Isolate `ViewModel` and UI state updates to `@MainActor`.
  - Add `.accessibilityLabel(...)` to icon-only buttons/custom controls and support Dynamic Type (avoid fixed non-scaling font sizes where text needs to scale).
- **Tiered Scope:**
  - *Full Feature:* `Data/` + `Domain/` + `Presentation/` + `DI/`.
  - *Logic / Device-SDK Feature (no API/DB):* `Domain/` + `Presentation/` (+ `DI/`); omit `Data/`.
  - *Pure UI / Static Screen:* `Presentation/UI/` only; omit `Domain/`, `Data/`, `DI/`.
- **Extension & Secondary Targets (`Widget/`, `Extensions/`, `Watch/`):**
  - Treat as `Presentation` entry points. Keep only `*Bundle.swift`, `Info.plist`, and `*.entitlements` at the target root.
  - Organize all Swift code inside `Presentation/`:
    - `Presentation/Provider/` — `TimelineProvider` (separate from Views)
    - `Presentation/Intents/` — `AppIntent`, `AppShortcutsProvider`
    - `Presentation/UI/` — `WidgetEntryView`, family views, `Widget` configuration
    - `Presentation/Services/` — Device/OS SDK wrappers & observers (if needed)
    - `Presentation/Helpers/` — Pure stateless UI helpers/formatters (if needed)
  - Keep `AppIntent` handlers thin: only call `DI -> UseCase` or `Presentation/Services/`, never embed business logic or access `DataSource` directly.

## 7. Verification

Always run before finishing:
- `swiftlint` (if configured)
- `xcodebuild` build & test (include headless `XCTest` unit tests for `*UseCase` business logic)
