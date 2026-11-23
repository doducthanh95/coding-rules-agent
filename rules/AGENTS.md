# End-to-End Spec, Design & Engineering Workflow

Follow this integrated lifecycle for new features and UI/UX work:

```text
speckit-constitution ─▶ speckit-specify ─▶ (speckit-clarify / speckit-converge)
        │
        ▼
[app-design-pipeline]  (Mode A: Dribbble → DESIGN.md → 6 Gates → Stitch | Mode B: Fast Path if DESIGN.md exists)
        │
        ▼
speckit-plan ─▶ speckit-tasks ─▶ speckit-analyze ─▶ speckit-implement (opus-coding-workflow + clean-arch-coding) ─▶ Verify
```

- **Spec Phase (`speckit-*`):** Capture requirements and acceptance criteria in `spec.md` before designing or coding. Use `speckit-clarify` / `speckit-converge` to resolve ambiguities before planning.
- **Design Phase (`app-design-pipeline`):** Always check project root for `DESIGN.md` first:
  - **Mode A (First-Time Setup):** `DESIGN.md` missing → run full 6 steps (Screen Budget → Dribbble Reference → `DESIGN.md` → 6 Quality Gates → Stitch MCP → Code Handoff).
  - **Mode B (Incremental Fast Path):** `DESIGN.md` exists (with locked `stitch_project_id` & `stitch_design_system`) → skip Dribbble/style discovery and go directly from Screen Budget → 6 Quality Gates → Stitch `generate_screen_from_text` → Code Handoff.
  - Combine `ui-ux-pro-max`, `mobile-design`, and `stitch-ui-design` during Gate 4.
- **Engineering Phase (`opus-coding-workflow` + `clean-arch-coding`):** Implement inside-out (`Domain -> Data -> DI -> Presentation`) referencing both `spec.md` (business contract) and `DESIGN.md` + approved Stitch screens (visual contract), then verify with real build/test/device checks.

---

# Clean Architecture & Coding Standard (Core Router)

Mandatory baseline for **Flutter, Android, and iOS**.
To prevent cross-platform terminology leaks, **load ONLY the platform-specific standard matching the active project** from the `clean-arch-coding` skill before designing or writing code:

- **Flutter (Dart):** `skills/clean-arch-coding/references/flutter.md`
- **Android (Kotlin):** `skills/clean-arch-coding/references/android.md`
- **iOS (Swift):** `skills/clean-arch-coding/references/ios.md`

Never mix types, folder names, or idioms from the other two platforms.

## Core Invariants (All Platforms)

1. **Feature-First 3-Layer:** `Presentation -> Domain <- Data` + `DI`. `Data` and `Domain` never depend on UI frameworks. `RepositoryImpl` is pure transport + 1-to-1 mapping only (no business filtering, sorting, or fallback logic).
2. **Hard Boundary (`UseCase` vs `Presentation`):**
   - **Headless Test:** Any logic that must still run if UI is removed (business rules, validation, orchestration, eligibility/cooldown, error mapping) belongs in `UseCase` / `ErrorUseCase`.
   - **Thin State Holder:** Only holds/emits immutable UI state, forwards 1 user intent to 1 `UseCase`, and switches on `Outcome` to emit UI state. Never inspects HTTP status/codes, accesses repositories/storage, or chains multi-step conditional UseCases.
   - **Self-Contained `UseCase`:** Never return raw HTTP responses or ambiguous `bool` to `Presentation`; return an explicit `Outcome` or `Result`. Max 2 `UseCase` call levels (`Orchestrator -> Atomic`), no circular dependencies. Keep the public entry method as a ~25–35 line step-by-step Table of Contents with focused private helpers.
3. **Control Flow:** Max 2 levels of `if/else` nesting. Prefer early returns/guard clauses, flat `switch`/`when` with explicit branch handling or safe fallback, and small helpers.
4. **Pragmatic Single Model:** Default to 1 model in `Domain` (`entities/`), with pure-language serialization (`fromJson`/`toJson` or `Codable`) called **only** by `Data`.
   - **Never create a 1-to-1 `DTO`:** Do NOT create `data/models/*Dto` if its fields mirror `domain/entities/` 1-to-1. Split a separate `DTO` in `Data` **only** when the API schema is deeply nested, aggregates multiple endpoints, or diverges significantly from domain needs.
   - **Refactor rule:** When refactoring existing code that already has redundant 1-to-1 DTOs, delete `data/models/` and collapse them into `domain/entities/`.
5. **Plain & Common English:** Use simple, everyday English words for file/class/function/variable names and git commits (e.g., `get`, `load`, `save`, `check`, `show`, `add`, `fix`, `update`, `remove`).
6. **Tiered Feature Scope & Verification:**
   - *Full Feature (API/DB/Storage):* `Data` + `Domain` + `Presentation` + `DI`.
   - *Logic / Device-SDK Feature (no API/DB):* `Domain` + `Presentation` (+ `DI`); omit `Data`.
   - *Pure UI / Static Screen:* `Presentation` UI only; omit `Domain`, `Data`, `DI`.
   - *Minimum Domain Verification:* Verify `UseCase` business logic with headless unit tests alongside platform lint/build checks.
7. **No Hardcoded Magic Constants (`Util/`):**
   - Never hardcode magic numbers, thresholds, timeouts, quotas, memory/token budgets, or domain rules inline in code.
   - Extract feature-scoped constants into a dedicated constants container (`<Feature>Constants` — `enum` in Swift, `object` in Kotlin, `abstract class` in Dart) inside that feature's `Util/` (`util/` on Android/Flutter) folder; place cross-feature constants in `App/Util/` or shared `utils/` / `core/util/`.
   - **Mandatory documentation:** Every important constant (RAM/context/token budgets, audio/DSP tuning, grading thresholds, algorithm coefficients, timeouts/quotas) MUST include a clear doc comment (`///` or `/** ... */`) explaining its purpose, unit, and rationale/impact.
   - Pure mathematical identities (`0`, `1`), standard protocol ranges (`200..<300`), and one-off local UI layout metrics may remain inline unless shared as design tokens.

---

# English Grammar & IPA Review Rule (IPA with Capitalized Stress)

Mỗi khi người dùng gửi tin nhắn hoặc câu hỏi có chứa tiếng Anh:

## 1. Nguyên tắc cốt lõi: Ngắn gọn - Sửa nhanh - Phiên âm IPA VIẾT HOA trọng âm
- **Không giải thích lý thuyết dông dài**. Chỉ ra trực tiếp từ sai ➔ từ đúng trong 1-2 dòng gạch đầu dòng.
- Đưa ra **1 câu gợi ý tự nhiên nhất**.
- Cung cấp **phiên âm IPA chuẩn gốc**, nhưng **VIẾT HOA âm tiết mang trọng âm** để người dùng nhìn vào là thấy ngay nhịp điệu cần nhấn (ví dụ: *example* ➔ `/ɪɡˈZÆM.pl̩/`, *UseCase* ➔ `/ˈJUːS.keɪs/`).

## 2. Định dạng phản hồi chuẩn:

### 📝 Góp ý tiếng Anh nhanh:
- **Câu gốc:** `[câu tiếng Anh]`
- **Sửa lỗi:** `[từ/cụm sai]` ➔ `[từ/cụm đúng]` (*lý do ngắn*)
- **Gợi ý tự nhiên:**
  > *"..."*
- **Phiên âm IPA (Trọng âm VIẾT HOA):**
  > */.../*

*(Nếu câu gốc đã hoàn toàn chuẩn ngữ pháp: Bỏ qua phần góp ý hoặc chỉ ghi nhận ngắn rồi trả lời ngay).*

## 3. Trả lời nội dung chính
Tiến hành trả lời câu hỏi / yêu cầu kỹ thuật của người dùng ngay bên dưới.

---

# HOH Engineering Workflow & Role Separation Rules (Harness of Harness)

> **ĐIỀU KIỆN KÍCH HOẠT DUY NHẤT:** Chỉ kích hoạt khi người dùng nhắc đến từ khóa **"HoH"**, **"HOH"**, **"Harness of Harness"** hoặc yêu cầu cụ thể áp dụng quy trình Planner -> Dev -> QA. Với mọi tác vụ thông thường khác, trả lời trực tiếp, súc tích và không kích hoạt chu trình 3 vai trò. Khi kích hoạt, đọc skill `hoh-workflow` để thực thi đầy đủ.

1. **Phân quyền 3 vai trò độc lập:**
   - **Project Planner (`read-only`):** Khảo sát dependency, chia Micro-Sprints, thống nhất Contract/Model trước khi sửa code.
   - **Developer (`write/edit`):** Chỉ code đúng phạm vi Micro-Sprint hiện tại; cấm tự tuyên bố xong khi chưa qua QA.
   - **QA Tester & Reviewer (`verify-only`):** Chạy lệnh kiểm thử thật (`flutter analyze`, `flutter test`, `xcodebuild`...), rà soát regression; không tự sửa code sản phẩm.
2. **Evidence-Based Verification:** Zero Trust — chỉ `PASSED` khi có log thực thi với exit code `0`.
3. **Ground Truth & Rollback:** Dựa vào file hệ thống; nếu sửa gây lỗi nghiêm trọng/regression thì dừng chắp vá, rollback về checkpoint ổn định và phân tích lại từ Planner.
