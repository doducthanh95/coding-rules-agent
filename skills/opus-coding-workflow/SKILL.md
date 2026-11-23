---
name: opus-coding-workflow
description: Universal engineering workflow for coding, refactoring, debugging, code review, and verification across all tech stacks (Flutter, Android, iOS, Web, Backend, CLI). Emphasizes spec-first inventory, contract/preview approval gates, root-cause fixes, and real command verification.
---

# Opus Coding Workflow (Universal)

Applies to all coding tasks across any language or framework: new features, refactoring, debugging, code review, and build fixes.

## 1. Core Principles
1. Read specs/docs first, code second.
2. Inventory existing files, symbols, and patterns before editing.
3. Stop at contract boundaries; never guess architectural decisions.
4. Break large tasks into small, independently verifiable phases.
5. Preserve working public APIs, UI, and behavior unless explicitly approved to change.
6. Fix root causes, not symptoms.
7. Review grep-first, assume nothing; read function bodies rather than trusting comments.
8. Verify with real build/test/runtime commands—never stop at "looks right".
9. Prefer existing repository patterns over inventing new abstractions.

## 2. Task Start & Approval Gate
- Before writing code, summarize: **scope, goals, hard rules, phases, stopping conditions, and reference paths**.
- **Preview before write (when appropriate):** Present a preview (file paths, scope, and content/diff) and wait for explicit user approval before editing when changing architecture/public contracts, deleting or renaming multiple files, adding new dependencies, or when requirements are ambiguous. Proceed directly without blocking on small bug fixes, lint/format fixes, localized refactors, or explicit user-directed edits. Never auto-apply after asking for confirmation.
- **Respect user-pinned constraints:** Keep user-specified config locations, method signatures, and file-scope limits intact; report cross-file dependencies as risks rather than silently expanding scope.
- **Repo architecture precedence:** Follow existing repository patterns (and local `AGENTS.md` if present) when a repo uses another architecture; apply `clean_arch_coding_standard.md` only for Clean Architecture repos or new projects.

## 3. Refactor Workflow
1. Inventory current files, symbols, dependencies, and global/singleton state.
2. Pick the smallest safe scope; scaffold infrastructure before feature logic.
3. Change internals first while preserving public contracts; do not bundle unrelated refactors.
4. **Quarantine before delete:** Comment in-place or move legacy code first; delete only after the new flow is verified end-to-end.

## 4. Debug Workflow
1. Capture and read the full error output (do not pipe-filter too early and lose context).
2. Reproduce the failure with an actual command.
3. Trace the data flow to the root cause and compare against a working pattern in the codebase.
4. Apply the minimal fix and re-run build/tests immediately. If repeated fixes fail, stop and re-evaluate the architecture.
5. **Non-deterministic transforms/crypto:** Inspect the real implementation (keys, runtime timestamps, random IV/nonce) and state clearly when outputs vary per run.

## 5. Review & Repo Analysis Workflow
- Grep across the whole workspace/monorepo (including indirect/runtime/build-graph references) before declaring any symbol unused or renaming shared contracts.
- Evaluate at 3 levels: **micro** (in-file correctness), **meso** (dependency/layering flow), **macro** (architectural consistency).
- When asked to read a whole folder/repo, report quantitative coverage (files/lines read vs intentionally skipped) and redact secrets/tokens as `[REDACTED]`.
- For non-git folders with graph tools, confirm with the user before initializing local index metadata, or offer direct manual file review.

## 6. Verification & Stopping Conditions
- **Always verify** with the stack's real commands (compile, lint/analyze, unit tests, app/simulator launch, or artifact inspection). Never rely on a piped `| tail` exit code without `pipefail` or a full build check.
- **Stop and ask the user** when:
  - Contract-level info is missing.
  - Multiple architectural approaches exist.
  - A change impacts unapproved public APIs, UI, or behavior.
  - A fix touches more layers than anticipated.

## 7. Stack-Specific References (Lazy-Load Only When Needed)
- `references/ios-xcode-and-networking-pitfalls.md` — Xcode `.pbxproj` target wiring, Swift `Decodable`/access-control, monorepo type disambiguation, iOS networking contract drift & token-refresh queues.
- `references/flutter-and-cross-platform-notes.md` — Flutter networking domain/Dio registration, native-bridge endpoint discovery, iOS `.xcarchive` triage.
- `references/macos-swiftui-and-ui-notes.md` — macOS SwiftUI launch/CLI pitfalls, multi-surface UI locking, design tokens.
