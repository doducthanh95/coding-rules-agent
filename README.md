# coding-rules-agent

Bộ **Quy chuẩn Lập trình (Coding Standard)** & **Quy trình Kỹ thuật (Engineering Workflow)** dành cho AI Coding Agent (Antigravity, Codex, Claude Code) trên đa nền tảng: **Flutter, Android, iOS**.

---

## 1. Mindset cốt lõi (Giải thích cho người Non-Tech cũng hiểu)

Hãy tưởng tượng bạn thuê một **AI viết code** giống như tuyển một **nhân viên mới làm việc cực nhanh nhưng rất hay "tiện tay đâu làm đó"**.

Nếu không có quy định rõ ràng, AI thường mắc 3 lỗi kinh điển:
1. **Trộn lẫn mọi thứ vào một chỗ:** Lấy dữ liệu mạng, kiểm tra logic nghiệp vụ và vẽ giao diện nút bấm chung trong 1 file. Lúc mới chạy thì rất nhanh, nhưng 1 tháng sau sửa nút bấm có thể làm hỏng luôn logic tính tiền.
2. **Làm theo kiểu "đoán mò":** Thấy chỗ nào cũng sửa, không đọc code cũ trước, xong việc thì bảo *"chắc là chạy được"* mà không chạy kiểm tra thật.
3. **Không thể kiểm thử tự động (Unit Test):** Vì nghiệp vụ bị dính chặt vào màn hình điện thoại, muốn kiểm tra logic đúng hay sai bắt buộc phải bật cả app lên bấm tay.

### Ví dụ Nhà hàng để hiểu kiến trúc 3 tầng

Bộ rule này ép AI tổ chức code giống như cách vận hành một **nhà hàng chuyên nghiệp**, chia làm 3 bộ phận tách biệt:

* **`Presentation` (Mặt tiền & Nhân viên phục vụ):**
  * Chỉ làm nhiệm vụ **hiển thị đẹp, nhận yêu cầu từ khách hàng và báo lại kết quả** (bật loading, hiện popup, toast, chuyển màn hình).
  * *Không tự vào bếp nấu ăn, không tự đi chợ.*
* **`Domain` (`UseCase` & `ErrorUseCase` — Bếp trưởng & Quy trình chế biến):**
  * Nắm toàn bộ **luật chơi và quy trình nghiệp vụ**: kiểm tra điều kiện, phối hợp các bước, phân loại lỗi.
  * *Nguyên tắc quan trọng:* Dù tháo bỏ toàn bộ màn hình điện thoại đi, "bếp trưởng" vẫn chạy đúng và kiểm tra được bằng test tự động (**Headless Test**).
* **`Data` (`RepositoryImpl` & `DTO` — Người đi chợ / Giao vận):**
  * Chỉ lo **lấy dữ liệu từ Server/Database** về đúng định dạng rồi giao cho bếp trưởng.
  * *Không tự ý quyết định nghiệp vụ hay lọc/chế biến dữ liệu theo cảm tính.*

> **Tóm lại:**
> `Presentation` (Mặt tiền) ➔ `Domain` (Nghiệp vụ) ⬅ `Data` (Lấy dữ liệu)
>
> Nhờ tách bạch như vậy: đổi giao diện không làm hỏng nghiệp vụ, đổi API không làm vỡ màn hình, và toàn bộ logic quan trọng đều test tự động được.

---

## 2. Tại sao lại chia `rules/` và `skills/`? Và lý do có từng Rule

### Tại sao tách `rules/` và `skills/`?
AI có giới hạn bộ nhớ trong mỗi cuộc trò chuyện (context window). Nếu nhồi tất cả hướng dẫn dài hàng nghìn dòng vào mọi lúc, AI sẽ bị chậm, tốn chi phí và dễ quên luật quan trọng. Vì vậy repo chia làm 2 phần:

* **`rules/AGENTS.md` (Luật luôn bật — Always-on):** Chứa những nguyên tắc **bất di bất dịch**, ngắn gọn, AI luôn phải nhớ ở mọi tác vụ.
* **`skills/` (Sổ tay chuyên môn — On-demand):** Chứa template thư mục, hướng dẫn chi tiết theo từng công việc. AI chỉ mở ra đọc khi gặp đúng tình huống cần dùng.

### Ý nghĩa của từng phần trong bộ Rule

| Thành phần | Mục đích & Lý do tồn tại |
| :--- | :--- |
| **1. Kiến trúc 3 tầng (`data/`, `domain/`, `presentation/`, `di.*`)** | Mỗi tính năng nằm gọn trong 1 thư mục độc lập, dễ tìm, dễ xóa, không đan chéo lung tung giữa các màn hình. |
| **2. Ranh giới `UseCase` vs `Presentation`** | • **Headless Test:** Logic chạy được khi bỏ UI thì bắt buộc để ở `Domain`.<br>• **Thin Coordinator:** `Cubit`/`ViewModel` chỉ làm 3 việc: giữ state UI, chuyển 1 yêu cầu sang 1 `UseCase`, và `switch` kết quả để cập nhật UI.<br>• **Self-Contained UseCase:** Không trả dữ liệu HTTP thô hay `true/false` mơ hồ cho UI; phải trả về kết quả phân loại rõ ràng (`Outcome`/`Result`).<br>• **Anti-God-Function:** Hàm chính (`execute()`) chỉ dài ~25–35 dòng như một **mục lục** từng bước; tránh đẻ ra hàm khổng lồ hàng trăm dòng không ai dám sửa. |
| **3. Tách `Error / Popup / Toast`** | `Domain` phân loại **lỗi gì & thông điệp gì**; `View` tự quyết định **hiển thị bằng cách nào** (Popup chặn, Toast thoáng qua, hay chữ đỏ dưới ô nhập). |
| **4. Control Flow tối đa 2 cấp `if/else`** | Tránh code lồng 4–5 tầng `if` rối rắm dễ sót trường hợp lỗi. Dùng `return` sớm (guard clause) và `switch` để đọc thẳng từ trên xuống dưới. |
| **5. Single-Class View + `// MARK:`** | Gom code giao diện của 1 màn hình vào 1 class nhưng chia làm 4 ngăn cố định (`Properties`, `Lifecycle`, `UI Rendering`, `UI Handlers & Listeners`), vừa gọn vừa dễ tìm. |
| **`skills/clean-arch-coding/`** | Cung cấp cấu trúc thư mục mẫu, bảng quy đổi cho **Flutter / Android / iOS** và checklist 4 bước khi làm tính năng mới hoặc refactor. |
| **`skills/opus-coding-workflow/`** | Ép AI làm việc có kỷ luật: **đọc code cũ trước khi sửa → báo trước phạm vi thay đổi → sửa tận gốc → chạy lệnh build/test thật để kiểm chứng** (kèm tài liệu lỗi thường gặp của iOS/Flutter/macOS). |
| **`skills/english-grammar-mentor/`** | Khi bạn nhắn bằng tiếng Anh, AI tự động sửa nhanh 1–2 dòng, gợi ý câu tự nhiên và kèm phiên âm IPA **VIẾT HOA trọng âm** trước khi trả lời kỹ thuật. |

---

## 3. Cấu trúc Repo

```text
coding-rules-agent/
├── plugin.json                              # Metadata plugin cho Antigravity
├── rules/
│   └── AGENTS.md                            # Luật luôn bật (Clean Arch + English/IPA review)
└── skills/
    ├── clean-arch-coding/                   # Template & adapter Flutter / Android / iOS
    ├── opus-coding-workflow/                # Quy trình Spec-first, Refactor, Debug, Verify
    └── english-grammar-mentor/              # Sửa tiếng Anh & IPA viết hoa trọng âm
```

---

## 4. Hướng dẫn Cài đặt & Tích hợp

### A. Antigravity

#### Cách 1: Cài Global Plugin (Khuyên dùng cho mọi repo)
```bash
git clone https://github.com/doducthanh95/coding-rules-agent.git ~/.gemini/config/plugins/coding-rules-agent
```
Cập nhật khi có bản mới:
```bash
git -C ~/.gemini/config/plugins/coding-rules-agent pull
```

#### Cách 2: Cài riêng cho từng Project (`.agents/`)
Copy thư mục `rules/` và `skills/` vào `.agents/` ở thư mục gốc của project rồi commit lên git để cả team tự động dùng chung:
```bash
mkdir -p .agents
cp -r /path/to/coding-rules-agent/rules .agents/
cp -r /path/to/coding-rules-agent/skills .agents/
```

---

### B. OpenAI Codex (CLI / Desktop App)

Codex tự động đọc file chỉ dẫn `AGENTS.md` và thư mục `skills/`.

#### Cách 1: Tích hợp Global (Áp dụng cho mọi project trên máy)
1. Clone repo về máy (nếu chưa có):
   ```bash
   git clone https://github.com/doducthanh95/coding-rules-agent.git ~/coding-rules-agent
   ```
2. Liên kết (symlink) `AGENTS.md` và các `skills` vào thư mục `~/.codex/`:
   ```bash
   mkdir -p ~/.codex/skills
   ln -sf ~/coding-rules-agent/rules/AGENTS.md ~/.codex/AGENTS.md
   ln -sf ~/coding-rules-agent/skills/clean-arch-coding ~/.codex/skills/clean-arch-coding
   ln -sf ~/coding-rules-agent/skills/opus-coding-workflow ~/.codex/skills/opus-coding-workflow
   ln -sf ~/coding-rules-agent/skills/english-grammar-mentor ~/.codex/skills/english-grammar-mentor
   ```
   *(Nhờ dùng `ln -sf`, sau này chỉ cần `git -C ~/coding-rules-agent pull` là Codex tự nhận bản mới nhất).*

#### Cách 2: Tích hợp riêng cho từng Project
Copy `rules/AGENTS.md` ra thư mục gốc project thành `./AGENTS.md` và copy `skills/*` vào `./.agents/skills/` (hoặc `./.codex/skills/`):
```bash
cp ~/coding-rules-agent/rules/AGENTS.md ./AGENTS.md
mkdir -p .agents/skills
cp -r ~/coding-rules-agent/skills/* .agents/skills/
```

---

### C. Claude Code (`claude`)

Claude Code đọc file chỉ dẫn `CLAUDE.md` và thư mục `skills/`.

#### Cách 1: Tích hợp Global (Áp dụng cho mọi project trên máy)
1. Clone repo về máy (nếu chưa có):
   ```bash
   git clone https://github.com/doducthanh95/coding-rules-agent.git ~/coding-rules-agent
   ```
2. Liên kết (symlink) `AGENTS.md` thành `~/.claude/CLAUDE.md` và các `skills` vào `~/.claude/skills/`:
   ```bash
   mkdir -p ~/.claude/skills
   ln -sf ~/coding-rules-agent/rules/AGENTS.md ~/.claude/CLAUDE.md
   ln -sf ~/coding-rules-agent/skills/clean-arch-coding ~/.claude/skills/clean-arch-coding
   ln -sf ~/coding-rules-agent/skills/opus-coding-workflow ~/.claude/skills/opus-coding-workflow
   ln -sf ~/coding-rules-agent/skills/english-grammar-mentor ~/.claude/skills/english-grammar-mentor
   ```
   *(Nếu bạn đã có sẵn `~/.claude/CLAUDE.md` trước đó, hãy gộp thêm nội dung từ `rules/AGENTS.md` vào cuối file thay vì ghi đè bằng `ln -sf`).*

#### Cách 2: Tích hợp riêng cho từng Project
Copy `rules/AGENTS.md` thành `./CLAUDE.md` ở gốc project và copy `skills/*` vào `./.claude/skills/`:
```bash
cp ~/coding-rules-agent/rules/AGENTS.md ./CLAUDE.md
mkdir -p .claude/skills
cp -r ~/coding-rules-agent/skills/* .claude/skills/
```
