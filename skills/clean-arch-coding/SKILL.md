---
name: clean-arch-coding
description: Multi-platform Clean Architecture 3-layer workflow, templates, and platform adapters for Flutter, Android, and iOS. Activate when creating, extending, or refactoring features under the Clean Architecture coding standard.
---

# Clean Architecture Coding Workflow

## 0. Platform Adapter Selection (Mandatory First Step)
Before designing or writing code, **read ONLY the reference file matching the active project's stack** (never load or mix the other two):
- **Flutter (Dart):** `references/flutter.md`
- **Android (Kotlin):** `references/android.md`
- **iOS (Swift):** `references/ios.md`

## 1. Execution Workflow
1. **Inventory:** Read the matching `references/<platform>.md` and inspect 1 sibling feature in the codebase (`DI`, `UseCase`, `State`) before editing.
2. **Contract First:** Define `UseCase` methods, `Outcome` variants, `Repository` interface (if needed), single models in `domain/entities/` (justify any `data/models/` DTO explicitly against the 3 divergence criteria before creating it), and immutable `UIState`.
3. **Implement Inside-Out:** `Domain` -> `Data` (if needed) -> `DI` -> `Presentation` (and secondary entry points if any).
4. **Self-Review & Verify:**
   - **Single-Model Check:** Zero 1-to-1 duplicate `DTO` <-> `Entity` pairs (if `data/models/` mirrors `domain/entities/`, collapse into `domain/entities/` and delete `data/models/`).
   - **Constants Check:** Zero hardcoded magic numbers in logic/config; important constants live in each feature's `Util/` (`util/`) constants file with clear doc comments explaining meaning, unit, and rationale.
   - Zero non-UI logic in `Presentation`; zero `if/else` nesting > 2 levels.
   - Zero cross-platform terminology leaks.
   - Run the platform's static analysis, build, and headless `UseCase` unit tests before finishing.
