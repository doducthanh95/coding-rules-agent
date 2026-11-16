# Clean Architecture & Coding Standard (Multi-Platform)

Mandatory baseline for all coding tasks across **Flutter, Android, and iOS**.
For full folder templates, platform adapters, and the 4-step checklist when implementing or refactoring a feature, read the `clean-arch-coding` skill.

## Core Invariants

1. **Feature-First 3-Layer Architecture:** `data/`, `domain/`, `presentation/`, plus `di.*`. Dependency rule: `Presentation -> Domain <- Data`. `Data` and `Domain` must never depend on UI frameworks. `Data / RepositoryImpl` is pure transport and 1-to-1 DTO mapping—never put business filtering, sorting, fallback, or source-selection rules there.
2. **Hard Boundary — UseCase vs Presentation (3 Rules):**
   - **Headless Test:** Any logic that must still run if the UI is removed (business rules, eligibility/cooldown, orchestration, validation, status checks, error mapping) belongs in `UseCase` / `ErrorUseCase`. Only UI/OS-lifecycle concerns (loading, focus, animations, layout, navigation, showing Popup/Toast) belong in `Presentation`.
   - **Thin Coordinator:** `Cubit` / `ViewModel` / `TimelineProvider` does only 3 things: hold/emit UI state, forward 1 user intent to 1 `UseCase`, and `switch` on `Outcome` to emit UI state. Never check HTTP status/codes, access repositories/storage, or chain multi-step conditional UseCases here.
   - **Self-Contained UseCase:** Never return raw HTTP responses or ambiguous `bool` to Presentation. Orchestrate multi-step flows inside the UseCase and return an explicit `sealed Outcome` or `Result<Data, Error>`.
   - **UseCase Composition & Anti-God-Function:** A UseCase may call another UseCase only to reuse shared domain logic (`*ErrorUseCase` or logic shared across >=2 flows); keep single-use logic as private methods. Max 2 UseCase call levels (`Orchestrator -> Atomic`), no circular dependencies. Keep `execute()` as a ~25–35 line step-by-step Table of Contents, extract each step into focused private helpers under `// MARK: -` (`Public Entry`, `Business Steps`, `Builders & Helpers`), and scope each UseCase class to 1 cohesive capability.
3. **Error / Popup / Toast Split:** Domain (`*UseCase` + `*ErrorUseCase`) classifies the error/scenario and message; Coordinator maps `Outcome -> UIState`; View (`Page`/`Screen`/`ViewController`) chooses the UI mechanism (`Popup`, `Toast`, `Inline`).
4. **Control Flow:** Max 2 levels of `if/else` nesting. Prefer early returns/guard clauses, `switch`/`when` or map lookups, and small focused helpers.
5. **Single-Class View + Section Markers:** Keep UI rendering and UI handlers in the same View class, separated into dedicated methods under 4 ordered `// MARK: -` (or `// region`) sections: `Properties & Dependencies`, `Lifecycle`, `UI Rendering`, `UI Handlers & Listeners`. Extract child widgets/components only when reused or large.
