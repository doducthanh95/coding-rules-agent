---
name: app-design-pipeline
description: >-
  End-to-end 6-step App UI/UX Design Pipeline (Spec -> Dribbble Reference Selection ->
  Vision-extracted DESIGN.md -> 6 Design Quality Gates -> Stitch MCP Generation -> Code Handoff).
  Use when designing or redesigning mobile/web app UI from functional specs, extracting styles
  from Dribbble shots/videos, or generating consistent multi-screen designs in Google Stitch.
---

# App Design Pipeline (Spec → Dribbble → DESIGN.md → Stitch → Code)

A repeatable workflow that transforms dense functional specifications into distinctive, trendy, zero-clutter UI designs without keeping legacy boxy layouts.

## Execution Modes (Check Project Root First!)
Before starting, check whether `DESIGN.md` already exists in the project root:
- **Mode A — First-Time Project Setup (Full 6 Steps):** `DESIGN.md` does **not** exist yet. Run Steps 1 → 2 → 3 → 4 → 5 → 6 to let the user pick a Dribbble reference and lock the project's visual identity.
- **Mode B — Incremental Feature Fast Path (Skip Steps 2 & 3):** `DESIGN.md` **already exists** (with locked `stitch_project_id` and `stitch_design_system`). **NEVER ask the user to pick a Dribbble link or design style again** (unless they explicitly ask to redesign). Go straight from **Step 1 (Screen Budget)** → **Step 4 (6 Quality Gates)** → **Step 5 (prompt `generate_screen_from_text` directly onto the existing `stitch_project_id` + `stitch_design_system`)** → **Step 6 (Code Handoff)**.

---

## Step 1: Distill Functional Spec (`Spec → Screen Budget`)
- **Input:** Functional spec (`spec.md`, `UI_SPEC_REQUIREMENTS.md`, or user feature description).
- **Actions:**
  1. Classify screens into **Primary Tab Screens** (max 4–5 tabs) vs **Immersive / Modal Rooms** (hide bottom tab bar).
  2. For each screen, define **1 Hero Focal Block** (35–45% viewport) and move secondary breakdowns (multi-part explanations, long lists, settings) into **1-line summary chips** that open a Bottom Sheet (*Progressive Disclosure*).
  3. Replace raw technical specs with **short, realistic sample data** before prompting any UI tool.
- **Output:** `Screen Budget Sheet`.

## Step 2: Dribbble Trend Discovery & User Selection *(Human-in-the-loop)*
- **Actions:**
  1. Search `site:dribbble.com` for current UI/UX trends matching the app domain (`search_web`).
  2. Resolve redirect links (`curl -sI`) and present **3–5 direct Dribbble shot URLs** with brief style descriptions.
  3. **Wait for the user** to pick 1 *North Star Reference* (link, screenshot, or `.mp4` motion clip).
- **Output:** 1 approved reference shot/video.

## Step 3: Extract Visual DNA & Layout Architecture → `DESIGN.md`
- **Actions:**
  1. Inspect the reference image directly (`read_file`), or for `.mp4` videos download to scratch and extract 6 key frames with `ffmpeg` (`fps=6/$DUR`) then read each frame.
  2. Extract two layers into `DESIGN.md` in the project root (and include `stitch_project_id` + `stitch_design_system` metadata once created in Step 5 so future features trigger **Mode B Fast Path** automatically):
     - **Visual Tokens:** Background (pastel mesh / light / dark), card surface & opacity, primary/category accents, typography (`BE_VIETNAM_PRO` + `JETBRAINS_MONO` for Stitch preview → `SF Pro` + `SF Mono` on iOS), corner radius, shadow vs hairline.
     - **Layout Architecture:** Spatial pattern (Open Breathing Stage, Bento Grid, Chat + Voice Wave, Stacked Deck, Center-FAB Tab Bar).
     - **Motion Identity (`motion-design`):** Select 1 Motion Personality (`Playful` / `Premium` / `Corporate` / `Energetic`), signature easing, and 3-tier duration palette (quick / standard / slow) — especially when the Dribbble reference is an `.mp4` motion clip.
- **Output:** Project `DESIGN.md`.

## Step 4: Enforce 6 Design Quality Gate Rules
Combine `ui-ux-pro-max` (query `--domain style` and `--domain color` individually; avoid web-centric `--design-system` defaults), `mobile-design`, and `stitch-ui-design` with these **6 non-negotiable rules**:
1. **Layout Refactor Rule:** Refactor the entire spatial layout to match the Dribbble reference — NEVER keep old stacked card frames and only recolor them.
2. **1 Hero / 1 Screen + Progressive Disclosure:** Collapse dense details into compact chips/sheets.
3. **Text Budget:** Card titles `<= 4` words; secondary lines `<= 1` line (`<= 45` chars).
4. **Single-Line & Zero-Truncation Buttons:** Every button label MUST be `1–3` words on **1 single line** (`white-space: nowrap`, `lineLimit(1)`), zero `...`. If 3 actions share a `390pt` row, side buttons MUST be `44×44pt` icon-only circles.
5. **Native Tab Bar:** Max 4–5 single-line labels (or Center-FAB 4-tab bar); hide inside immersive task rooms.
6. **Touch & Contrast:** `>= 44×44pt` hit area, `>= 4.5:1` text contrast.

## Step 5: Lock Design System & Generate via Stitch MCP
1. Create a clean Stitch project (`create_project`).
2. Lock tokens with `create_design_system` (`colorMode`, `customColor`, `headlineFont: "BE_VIETNAM_PRO"`, `bodyFont: "BE_VIETNAM_PRO"`, `labelFont: "JETBRAINS_MONO"`, `roundness: "ROUND_TWELVE"`, `designMd`).
3. Generate screens **sequentially** (1 screen at a time to avoid HTTP2 timeouts) via `generate_screen_from_text(projectId, prompt, designSystem: "assets/<id>", deviceType: "MOBILE", modelId: "GEMINI_3_5_FLASH_LITE")`.
4. Download rendered PNGs via `download_file(name: screenshot.name)` and visually inspect every PNG with `read_file` against the 6 Quality Gate rules. Fix any text wrapping or overlap with `edit_screens` (and re-apply `apply_design_system` without the `assets/` prefix if style drifts).

## Step 6: Engineering Handoff (UI-Only Refactor)
- **Actions:** Implement approved Stitch screens + `DESIGN.md` tokens into the target UI framework (SwiftUI / Flutter / Web) while keeping `Domain/`, `Data/`, and `ViewModel/` logic untouched. Apply `motion-design` (timing, easing, 1/3 rules, micro-interactions, loading/success/error states) for all UI transitions and animations, then build and verify on a real device.

---

## Integration with Spec-Driven Development (`speckit`)
- **Placement:** Insert `app-design-pipeline` right between `review-spec` and `plan`:
  `specify → review-spec → [app-design-pipeline: DESIGN.md + Stitch Screens] → plan → tasks → implement`
- **Why:** `plan` and `tasks` then reference both `spec.md` (business contract) and `DESIGN.md` + approved Stitch screen IDs (visual contract), preventing UI rework during implementation.
- **Bundled Safety:** Do not mutate bundled `Spec/.specify/workflows/speckit/workflow.yml`; invoke this skill or add a separate custom `speckit.design` command.
