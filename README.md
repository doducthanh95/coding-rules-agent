# coding-rules-agent

Bộ **Coding Standard** & **Engineering Workflow** cho AI Coding Agent (Antigravity, Codex, Claude Code) trên **Flutter, Android, iOS**.

---

## 1. Kiến trúc 3 tầng & Phạm vi phụ thuộc

```mermaid
flowchart LR
    subgraph P["Presentation (UI & Coordinator)"]
        UI["View (4 MARK sections)"]
        State["Coordinator + State<br/>(sealed class / @freezed)"]
        PHelpers["presentation/helpers/<br/>(UI helpers)"]
        PSDK["External SDK Wrappers<br/>(eKYC, Biometric, DeviceInfo, Lottie...)"]
    end

    subgraph D["Domain (Pure Language)"]
        UC["UseCase + ErrorUseCase<br/>(sealed Outcome / Result)"]
        RepoIf["Repository Interfaces"]
        Entity["Entities"]
        DHelpers["helpers/ & package utils/<br/>(Pure stateless helpers)"]
    end

    subgraph DT["Data (Transport & Mapping)"]
        RepoImpl["RepositoryImpl"]
        DTO["Endpoint / DataSource / DTO"]
        DSDK["Data Infra Libs<br/>(HTTP, Storage, Crypto, Freezed)"]
    end

    P -->|"Pure input / Intent"| D
    D -->|"Outcome / Entity"| P
    DT -.->|"Implements"| RepoIf
    PSDK -.->|"Pure values only"| State
```

> **Nguyên tắc cốt lõi:** `Presentation ➔ Domain ⬅ Data`
> - `Domain` thuần ngôn ngữ — không UI framework, không 3rd-party SDK.
> - `Data` chỉ làm transport & map DTO 1-1 — không chứa business rule, không import Feature/Device/UI SDK.
> - **Feature / Device / UI SDK** (`ekyc`, `local_auth`, `device_info_plus`, `camera`, `webview`...) **chỉ import trong `Presentation`** để mỗi feature/plugin tự đóng gói và chạy độc lập.

---

## 2. Luồng xử lý & Phân tách Lỗi (Error / Popup / Toast)

```mermaid
sequenceDiagram
    actor User
    participant V as View (Page / Screen)
    participant SDK as External SDK (in Presentation)
    participant C as Coordinator (Cubit / VM)
    participant U as UseCase + ErrorUseCase
    participant R as Repository (Data)

    User->>V: Tap Action
    opt Cần SDK thiết bị / eKYC / Biometric
        V->>SDK: Run SDK flow
        SDK-->>V: Pure result (token / string)
    end
    V->>C: Forward 1 intent (+ pure value)
    C->>U: execute()
    U->>R: Fetch / Submit
    R-->>U: Entity / Failure
    U-->>C: sealed Outcome (scenario + message)
    C-->>V: emit(UIState)
    V->>V: Flat switch ➔ Popup / Toast / Inline
```

---

## 3. Cấu trúc Feature & Helpers

```mermaid
flowchart TD
    Pkg["package/lib/"] --> Utils["utils/ (Cross-feature pure helpers, >= 2 features)"]
    Pkg --> Feat["feature/<name>/"]
    Feat --> DI["di.* (Wire Data -> Domain -> Presentation)"]
    Feat --> Util["util/ (Feature-local documented constants)"]
    Feat --> Helpers["helpers/ (Feature-local pure helpers, formatters)"]
    Feat --> Data["data/ (endpoint, models/DTO, repositories_impl)"]
    Feat --> Domain["domain/ (entities, repositories, usecases)"]
    Feat --> Pres["presentation/ (cubit/vm, helpers/, ui/, widgets/)"]
```

---

## 4. Giao tiếp & Kết nối giữa các Feature

```mermaid
flowchart LR
    subgraph FA["Feature A (VD: Register Device)"]
        FA_UI["Presentation (View)"]
        FA_UC["Domain (UseCase)"]
    end

    subgraph Shared["Cấp Package / Shared Bridge (Không phụ thuộc ngược Feature)"]
        Router["Router / Navigation<br/>(Pure Params & Result)"]
        SharedUC["Shared Domain / UseCase<br/>(VD: Common OTP, Session)"]
        EventBus["Shared Event / State Stream<br/>(VD: AuthEvent, Logout)"]
    end

    subgraph FB["Feature B (VD: Common OTP / Biometric)"]
        FB_UI["Presentation (View)"]
        FB_UC["Domain (UseCase)"]
    end

    FA_UI -->|"1. Điều hướng + Args"| Router --> FB_UI
    FB_UI -.->|"Trả kết quả thuần"| Router -.-> FA_UI

    FA_UC -->|"2. Tái sử dụng logic"| SharedUC
    FB_UC -->|"2. Tái sử dụng logic"| SharedUC

    FA_UC -->|"3. Phát / Nghe sự kiện"| EventBus
    FB_UC -->|"3. Phát / Nghe sự kiện"| EventBus

    FA -. "CẤM import chéo trực tiếp<br/>(data/cubit/internal)" .-> FB
```

- **Chuyển màn hình:** Đi qua **Router** bằng tham số/kết quả thuần (không mở trực tiếp `Cubit` hoặc `Data` của feature khác).
- **Dùng chung nghiệp vụ:** Đưa logic/contract dùng chung ra **Shared Domain / UseCase** hoặc `utils/` cấp package (tối đa 2 cấp `Orchestrator -> Atomic`).
- **Giao tiếp bất đồng bộ / tách rời:** Dùng **Event / Stream** dùng chung ở cấp package.

---

## 5. Tóm tắt Quy chuẩn

| Quy tắc | Trọng tâm |
|---|---|
| **3-Layer & DI** | `Presentation ➔ Domain ⬅ Data`, nối qua `di.*`. |
| **Hard Boundary** | Logic chạy được khi bỏ UI ➔ `UseCase`. `Coordinator` chỉ giữ state, gọi 1 `UseCase`, `switch(Outcome)`. |
| **Anti-God-Function** | `execute()` ~25–35 dòng như mục lục; tối đa 2 cấp UseCase (`Orchestrator -> Atomic`). |
| **Control Flow** | Lồng `if/else` tối đa **2 cấp**; ưu tiên guard clause & `switch`. |
| **Single-Class View** | 4 section `// MARK: -`: `Properties & Dependencies`, `Lifecycle`, `UI Rendering`, `UI Handlers & Listeners`. |
| **Exhaustive State** | Immutable & exhaustive `switch`: Dart `sealed class`/`@freezed`, Kotlin `sealed interface`, Swift `enum`. |
| **Single Model** | 1 model ở `domain/entities/` cho case 1-1 (`fromJson` chỉ gọi ở `Data`); tách `DTO` ở `data/` khi lệch schema. |
| **External SDK Scope** | Feature/Device/UI SDK chỉ ở `Presentation`; HTTP/Storage/Serialization lib ở `Data`; `Domain` thuần. |
| **Helpers Purity** | Pure helper ở `helpers/` (local) hoặc `utils/` (shared); UI helper chỉ ở `presentation/helpers/`. |
| **No Magic Constants** | Không fix cứng số/ngưỡng inline; gom `<Feature>Constants` vào `util/` (hoặc `utils/`/`App/Util/`) và ghi doc comment cho hằng số quan trọng. |
| **Plain English** | Đặt tên & commit bằng từ tiếng Anh phổ thông (`get`, `load`, `add`, `fix`, `update`...), tránh từ hiếm/học thuật. |

---

## 6. Cấu trúc Repo

```text
coding-rules-agent/
├── plugin.json                              # Metadata plugin cho Antigravity
├── rules/
│   └── AGENTS.md                            # Luật luôn bật (Clean Arch + English/IPA + HoH trigger)
└── skills/
    ├── clean-arch-coding/                   # Template & adapter Flutter / Android / iOS
    ├── opus-coding-workflow/                # Quy trình Spec-first, Refactor, Debug, Verify
    ├── hoh-workflow/                        # Quy trình phân quyền 3 vai trò Planner -> Dev -> QA
    └── english-grammar-mentor/              # Sửa tiếng Anh & IPA viết hoa trọng âm
```

- `rules/AGENTS.md` (Always-on): Nguyên tắc bất biến, luôn nạp vào context.
- `skills/` (On-demand): Template chi tiết & checklist, chỉ đọc khi đúng tác vụ.

---

## 7. Cài đặt & Tích hợp

### A. Antigravity
- **Global Plugin:**
  ```bash
  git clone https://github.com/doducthanh95/coding-rules-agent.git ~/.gemini/config/plugins/coding-rules-agent
  # Cập nhật: git -C ~/.gemini/config/plugins/coding-rules-agent pull
  ```
- **Per-Project (`.agents/`):**
  ```bash
  mkdir -p .agents && cp -r /path/to/coding-rules-agent/{rules,skills} .agents/
  ```

### B. OpenAI Codex
- **Global (`~/.codex/`):**
  ```bash
  git clone https://github.com/doducthanh95/coding-rules-agent.git ~/coding-rules-agent
  mkdir -p ~/.codex/skills
  ln -sf ~/coding-rules-agent/rules/AGENTS.md ~/.codex/AGENTS.md
  ln -sf ~/coding-rules-agent/skills/* ~/.codex/skills/
  ```
- **Per-Project:**
  ```bash
  cp ~/coding-rules-agent/rules/AGENTS.md ./AGENTS.md
  mkdir -p .agents/skills && cp -r ~/coding-rules-agent/skills/* .agents/skills/
  ```

### C. Claude Code (`claude`)
- **Global (`~/.claude/`):**
  ```bash
  git clone https://github.com/doducthanh95/coding-rules-agent.git ~/coding-rules-agent
  mkdir -p ~/.claude/skills
  ln -sf ~/coding-rules-agent/rules/AGENTS.md ~/.claude/CLAUDE.md
  ln -sf ~/coding-rules-agent/skills/* ~/.claude/skills/
  ```
  *(Nếu đã có `~/.claude/CLAUDE.md`, hãy nối thêm nội dung thay vì ghi đè.)*
- **Per-Project:**
  ```bash
  cp ~/coding-rules-agent/rules/AGENTS.md ./CLAUDE.md
  mkdir -p .claude/skills && cp -r ~/coding-rules-agent/skills/* .claude/skills/
  ```
