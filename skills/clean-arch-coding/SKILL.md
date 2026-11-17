---
name: clean-arch-coding
description: Multi-platform Clean Architecture 3-layer workflow, templates, and platform adapters for Flutter, Android, and iOS. Activate when creating, extending, or refactoring features under the Clean Architecture coding standard.
---

# Clean Architecture Coding Workflow

## 1. Feature Folder Structure
```text
feature/<feature_name>/
├── di.*                    # Dependency wiring: Data -> Domain -> Presentation
├── helpers/                # Pure feature-local enums, constants, formatters (cross-feature -> package utils/)
├── data/                   # Endpoint/DataSource, RepositoryImpl, optional DTOs (only when schema differs from Domain)
├── domain/
│   ├── entities/           # Single domain models (pure serialization annotations allowed; only data/ calls fromJson/toJson)
│   ├── repositories/       # Abstract/Protocol repository contracts
│   └── usecases/           # *UseCase + *ErrorUseCase (core logic lives here)
└── presentation/
    ├── <coordinator>/      # Cubit/State, ViewModel/UiState, TimelineProvider
    ├── helpers/            # UI-only helpers (BuildContext, styles, input formatters) (optional)
    └── ui/                 # Page/Screen/ViewController + widgets/ (also wraps external UI/device/feature SDKs)
```

## 2. Error, Popup, Toast & Inline Convention
- **Domain (`*UseCase` + `*ErrorUseCase`):** Resolve error codes/backend feedback types into explicit domain `Outcome` variants with localized messages. Never import UI APIs.
- **Coordinator (`Cubit` / `ViewModel`):** `switch (outcome)` -> `emit(UIState)`. Never inspect `error.code` or `httpStatus`.
- **View (`Page` / `Screen` / `ViewController`):** Observe `UIState` with a flat `switch` to trigger `Popup/Dialog` (blocking/confirmation), `Toast/Snackbar` (transient), or `Inline` feedback. Route popup actions back to Coordinator -> UseCase.

## 3. Single-Class View Organization (`// MARK: -`)
Keep UI rendering and UI handling in the same View class with 4 ordered sections (no inline event/navigation/listener logic inside layout builders):
1. `// MARK: - Properties & Dependencies` — Coordinator, scroll/focus/text controllers, bindings.
2. `// MARK: - Lifecycle` — Init, observer registration, resource disposal.
3. `// MARK: - UI Rendering` — Main `build`/`body`/`setupUI` composes layout from dedicated section methods (`_buildHeaderSection`, `_buildFormSection`, etc.).
4. `// MARK: - UI Handlers & Listeners` — User action callbacks, state listener (`_handleStateChanged`), navigation, and popup/toast helpers.

## 4. External Libs & Helpers Rules
- **External Libs / SDKs:**
  - *Feature / Device / UI SDKs* (`ekyc`, `biometrics`, `local_auth`, `device_info_plus`, `camera`, `webview`, `lottie`, etc.): Import **only in `presentation/`** (wrap in a dedicated widget/helper/service under `presentation/`, never scatter raw SDK calls in `build()`). **Never import in `domain/` or `data/`**; pass only pure values into `Coordinator -> UseCase`.
  - *Data Infrastructure Libs* (HTTP client, `freezed`/`json_serializable`, local/secure storage, payload crypto/signing): Allowed in `data/` for transport/serialization/persistence; never import in `domain/` or `presentation/`.
  - *`domain/`*: Pure language code only (no UI frameworks or 3rd-party SDKs).
- **Helpers & Shared Utilities:**
  - *Scope:* Feature-local pure helpers go in `feature/<name>/helpers/`; cross-feature (`>= 2` features) helpers go in package/app-level `utils/` or `shared/helpers/`.
  - *Layer Purity:* Pure stateless helpers may be used by `domain/` and `presentation/`. UI helpers (importing UI frameworks, `BuildContext`, styles) belong only in `presentation/helpers/` or `shared/presentation/`. Never put API/SDK calls or mutable state in `helpers/`.

## 5. Platform Adapters

| Layer | Flutter / Dart *(ref: `authen_v2/feature/manager_devices`)* | Android *(Kotlin)* | iOS *(Swift)* |
|---|---|---|---|
| **Coordinator & State** | `*_cubit.dart` + `*_state.dart` (`sealed class` or `@freezed`, immutable; match feature's existing style) | `*ViewModel` + `StateFlow` / `sealed interface *UiState` | `*ViewModel` / `TimelineProvider` + `enum *State` |
| **View** | `ui/*_page.dart` + `ui/widgets/` | `ui/*Screen.kt` / `Fragment` + `components/` | `Views/*View.swift` / `*ViewController.swift` + `Components/` |
| **Domain** | `*Usecase` + `*ErrorUsecase` | `*UseCase` + `*ErrorUseCase` | `*UseCase` + `*ErrorUseCase` |
| **Data** | `endpoint.dart` + `RepositoryImpl` + optional `DTO` (`freezed`/`json_serializable` when schema diverges) | API Service + `RepositoryImpl` + optional `DTO` | Endpoint/DataSource + `RepositoryImpl` + optional `Codable DTO` |
| **DI** | `di.dart` (`<Feature>DI.register()` via `getIt`) | Hilt `@Module` / Koin `module` | `*DI` container / Factory |
| **Verify** | `dart format`, `flutter analyze`, `flutter test` | `./gradlew lint`, unit tests | `xcodebuild` / `swiftlint` / XCTest |

## 6. Execution Workflow & Self-Review
1. **Inventory:** Read existing feature/sibling patterns (`di`, `usecase`, `state`) before editing.
2. **Contract First:** Define `UseCase` methods, `Outcome`/`Result`, `Repository` interface, and `UIState`.
3. **Implement Inside-Out:** `Domain (Entity, Repo, ErrorUseCase, UseCase)` -> `Data` -> `DI` -> `Presentation`.
4. **Self-Review & Verify:**
   - Zero non-UI logic in `Presentation`; zero `if/else` nesting > 2 levels.
   - UseCase reads like the complete business spec of the feature.
   - Run the platform's static analysis/build/test command before finishing.
