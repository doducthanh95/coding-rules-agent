# coding-rules-agent

Antigravity plugin bundle containing the team's **Multi-Platform Clean Architecture Coding Standard** (Flutter, Android, iOS) and **Universal Engineering Workflow**.

## Contents

- `rules/AGENTS.md` — Always-on Core Invariants (3-layer Clean Architecture, UseCase vs Presentation boundary, Anti-God-Function `UseCase` structure, Error/Popup/Toast split, max 2-level nesting, Single-Class View with `// MARK:` sections).
- `skills/clean-arch-coding/` — Feature folder templates, Platform Adapters (Flutter, Android, iOS), and 4-step implementation checklist.
- `skills/opus-coding-workflow/` — Universal spec-first, preview-gate, refactor, debug, and verification workflow with lazy-loaded stack references (`references/`).

## Installation for Team Members

### Option A: Install as a Global Plugin (Recommended for multi-repo)
Clone directly into your Antigravity plugins directory:

```bash
git clone https://github.com/doducthanh95/coding-rules-agent.git ~/.gemini/config/plugins/coding-rules-agent
```

To update later:
```bash
git -C ~/.gemini/config/plugins/coding-rules-agent pull
```

### Option B: Workspace-scoped (`.agents/`)
Copy `rules/` and `skills/` into `.agents/` at the root of a project repository and commit to share automatically with anyone working in that repo.
