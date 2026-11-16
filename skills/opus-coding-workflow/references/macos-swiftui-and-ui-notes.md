# macOS SwiftUI & Multi-Surface UI Notes (Reference)

Load this file only when working on macOS native SwiftUI apps or multi-surface UI projects (e.g. Hermes Desktop/WebUI/CLI).

## macOS SwiftUI Pitfalls
- `swift build` passing only proves compilation; verify the actual `.app` bundle or Xcode target launch visibly.
- Keep app launch non-invasive: avoid spawning background CLIs/shells on `onAppear`/init unless requested; inspect `Process`, `open`, `osascript`, or CLI wrappers with `pgrep`/`ps` if extra Terminal windows appear.
- `@main attribute cannot be used in a module that contains top-level code`: check for accidental top-level executable statements.
- Prefer explicit `Binding(..., set:)` closures over complex inline key-paths; use multiline `"""` literals for prompt/template blocks.

## UI Polish & Multi-Surface Discipline
- Lock the exact target surface first (Desktop native vs WebUI vs TUI/CLI) by checking running process, codebase path, entrypoint, and existing event/focus buses before patching.
- Keep active/selected states visually distinct; paginate/chunk long lists with quick filters; centralize SwiftUI colors/fonts/gradients in shared design tokens.
- When asked to "run the app after finishing", launch the built app and confirm the process stays running.
