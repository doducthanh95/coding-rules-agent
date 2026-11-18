# Flutter (Dart) Clean Architecture & Coding Standard

Load this file ONLY when working on **Flutter / Dart** projects.

## 1. Feature Folder Structure

```text
feature/<feature_name>/
├── di.dart                     # <Feature>DI.register() via getIt (Data -> Domain -> Presentation)
├── helpers/                    # Pure Dart feature-local enums, constants, formatters (cross-feature -> utils/)
├── data/                       # Omit when no API / DB / local storage
│   ├── endpoint.dart
│   ├── models/                 # Optional DTOs (@freezed / @JsonSerializable) ONLY when API schema diverges
│   └── *_repository_impl.dart  # Pure transport + 1-to-1 mapping only
├── domain/
│   ├── entities/               # Single domain models (pure Dart; only data/ calls serialization)
│   ├── repositories/           # Abstract repository contracts
│   └── usecases/               # *_use_case.dart (*UseCase, *ErrorUseCase) + abstract class *Outcome (pure Dart ONLY)
└── presentation/
    ├── cubit/                  # *_cubit.dart + *_state.dart (abstract class hierarchy, @freezed, or enum status; immutable)
    ├── coordinator/            # Navigation / GoRouter ONLY (optional — NEVER put Cubit or State here)
    ├── services/               # Device/UI SDK wrappers, observers, hardware engines (optional)
    ├── helpers/                # Pure stateless UI formatters & style helpers (optional)
    └── ui/                     # *_page.dart + widgets/
```

## 2. Naming Convention & Class Naming

### 2.1 General Naming Rules (Plain & Familiar English)
- **Files & Folders:** `snake_case.dart` (e.g., `transfer_page.dart`, `get_account_list_use_case.dart`).
- **Classes / Enums / Mixins / Typedefs:** `PascalCase` (e.g., `TransferCubit`, `GetAccountListUseCase`).
- **Variables / Functions / Parameters:** `camelCase` (e.g., `accountList`, `submitTransfer()`).
- **Booleans:** Always prefix with `is`, `has`, `can`, or `should` (e.g., `isLoading`, `hasError`, `canSubmit`).
- **Private Members:** Prefix with `_` (e.g., `_buildHeader()`, `_onSubmitTap()`).
- **Plain, Everyday Developer English:** Avoid cryptic abbreviations (`UsrMgr`, `TxnRes`, `BalUC`, `InfoCrd`) and overly academic words. Prefer familiar verbs:
  - Read: `get`, `fetch`, `load` *(avoid `retrieve`, `acquire`, `procure`)*
  - Write: `save`, `create`, `update`, `delete`, `remove`, `clear` *(avoid `persist`, `purge`)*
  - Check: `check`, `verify`, `validate` *(avoid `ascertain`)*
  - Execute: `send`, `submit`, `start`, `stop`, `run` *(avoid `dispatch` unless event-bus)*
  - Transform / UI: `map`, `parse`, `format`, `show`, `hide`, `open`, `close`

### 2.2 Class Naming by Layer

| Layer | Component | Class Naming Rule | Example |
|---|---|---|---|
| **Domain** | Entity | `<Noun>` or `<Noun>Entity` | `Account`, `TransferInfo` |
| | Repository Contract | `<Feature>Repository` | `TransferRepository` |
| | UseCase | `<Verb><Noun>UseCase` | `GetAccountListUseCase`, `SubmitTransferUseCase` |
| | Error UseCase | `<Feature>ErrorUseCase` | `TransferErrorUseCase` |
| | Outcome | `<Action>Outcome` (`abstract class`) + `<Action><Result>` | `TransferOutcome`, `TransferSuccess`, `TransferFailure` |
| **Data** | Repository Impl | `<Feature>RepositoryImpl` | `TransferRepositoryImpl` |
| | Endpoint / DataSource | `<Feature>Endpoint`, `<Feature>RemoteDataSource`, `<Feature>LocalDataSource` | `TransferEndpoint`, `TransferRemoteDataSource` |
| | DTO (when needed) | `<Noun>Dto` or `<Noun>Request` / `<Noun>Response` | `TransferRequest`, `TransferResponse` |
| **DI** | Dependency Injection | `<Feature>DI` | `TransferDI` |
| **Presentation** | Cubit | `<Feature>Cubit` | `TransferCubit` |
| | State | `<Feature>State` + `<Feature><Status>` (or single state + `enum <Feature>Status`) | `TransferState`, `TransferInitial`, `TransferLoading`, `TransferLoaded`, `TransferError` |
| | Main Page | `<Feature>Page` | `TransferPage` |
| | Child Widget | `<Noun><Type>` (`Widget`, `Card`, `Item`, `Dialog`, `BottomSheet`) | `AccountCard`, `ConfirmTransferDialog` |
| | Coordinator (Nav) | `<Feature>Coordinator` or `<Feature>Router` | `TransferCoordinator` |
| | Service (SDK Wrapper) | `<Capability>Service` | `BiometricService`, `CameraService` |

## 3. Layer Boundaries & Dependencies

- **Domain (`domain/`):** Pure Dart only. **NEVER** import `flutter/material.dart`, `flutter/widgets.dart`, HTTP clients, or device/UI SDKs.
- **Data (`data/`):** May import HTTP clients, serialization codegen, local/secure storage, and payload crypto. Never import UI frameworks.
- **Presentation (`presentation/`):**
  - **Device / UI SDKs** (eKYC, Biometric, Camera, WebView, Audio, Lottie, Chart, OS Observers): Import **ONLY in `presentation/`** — wrap in `presentation/services/` or `presentation/ui/widgets/`, never scatter raw SDK calls inside `build()`. Pass only pure Dart values into `Cubit -> UseCase`.
  - **Thin Cubit:** Only holds/emits immutable `*_state.dart`, forwards 1 user intent to 1 `*UseCase`, and maps `outcome` to state (`if / else if`, `switch` with fallback, or `@freezed` `.when()`). Never inspect HTTP status/error codes, access repositories/storage, or chain multi-step conditional UseCases in `Cubit`.
  - **Self-Contained UseCase:** Orchestrate multi-step flows inside `*UseCase` and return an explicit `abstract class *Outcome` (or `@freezed` union) — never raw HTTP responses or ambiguous `bool`. Max 2 UseCase call levels (`Orchestrator -> Atomic`), no circular dependencies. Keep the public `call()` / entry method ~25–35 lines as a step-by-step Table of Contents.

## 4. Error, Popup, Toast & Inline Convention

1. **Domain (`*UseCase` + `*ErrorUseCase`):** Classify error codes/backend feedback into explicit `*Outcome` subclasses (e.g. `TransferSuccess`, `TransferFailure`) with localized messages.
2. **Cubit (`*_cubit.dart`):** Map `outcome` -> emit immutable `*_state.dart`.
3. **Page (`*_page.dart`):** Observe state with `BlocConsumer` / `BlocListener` + `BlocBuilder` using a flat `if / else if` or `switch` (with safe fallback) to trigger `Dialog` (blocking/confirmation), `SnackBar` / `Toast` (transient), or `Inline` feedback. Route dialog actions back to `Cubit -> UseCase`.

## 5. Single-Class Page & `// MARK: -` Sections

Keep UI rendering and UI handlers in the same main `*_page.dart` class under 4 ordered section markers:
1. `// MARK: - Properties & Dependencies` (Cubit, controllers, focus nodes)
2. `// MARK: - Lifecycle` (`initState`, `dispose`, observer registration)
3. `// MARK: - UI Rendering` (`build()` composing layout from focused private `_build*` methods)
4. `// MARK: - UI Handlers & Listeners` (user action callbacks, `BlocListener`, navigation, dialog/toast helpers)

Pure stateless child widgets under `presentation/ui/widgets/` (`StatelessWidget`) are exempt when they have no lifecycle or handlers.

## 6. Dart Coding, Performance & Control Flow Rules

- **Control Flow:** Max 2 levels of `if/else` nesting. Prefer early returns, flat `switch` / `if-else if` with safe fallback, and small private helpers.
- **No `sealed` keyword:** Use `abstract class` hierarchies (or `@freezed` / `Equatable` + `enum` status) instead of `sealed class`.
- **UI Performance & Lifecycle:**
  - Use `const` constructors for immutable widgets/layouts wherever possible.
  - Use `ListView.builder` / `ListView.separated` (with `itemCount`) for long or dynamic lists instead of `ListView(children: [...])`.
  - Scope `BlocBuilder` to the smallest widget subtree that needs rebuilds; use `context.read<T>()` only in callbacks/handlers, never inside `build()`.
  - Extract complex/independent UI blocks into `StatelessWidget` classes in `widgets/` instead of oversized `_build*` methods.
  - Always dispose `TextEditingController`, `FocusNode`, `ScrollController`, and cancel `StreamSubscription`s in `dispose()`.
- **Tiered Scope:**
  - *Full Feature:* `data/` + `domain/` + `presentation/` + `di.dart`.
  - *Logic / Device-SDK Feature (no API/DB):* `domain/` + `presentation/` (+ `di.dart`); omit `data/`.
  - *Pure UI / Static Screen:* `presentation/ui/` only; omit `domain/`, `data/`, `di.dart`.
- **Terminology:** In Flutter, `Widget` means UI components inside `presentation/ui/`. If the repo has native Home Screen Widgets/Extensions, organize that native code according to `ios.md` or `android.md`.

## 7. Verification

Always run before finishing:
- `dart format` on edited/new files
- `flutter analyze`
- `flutter test` (include headless unit tests in `test/` for `*UseCase` business logic)
