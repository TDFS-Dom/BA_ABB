# Business Analyst — Kiro Configuration (Ngân hàng Việt Nam)

Bộ config Kiro tách riêng cho role **Business Analyst** trong nền tảng ngân hàng Việt Nam.

---

## Tổng quan Elements

| Element | Files | Mục đích |
|---------|-------|----------|
| Steering (3) | product.md, backlog-standards.md, compliance-sbv.md | Bối cảnh nghiệp vụ + backlog + tuân thủ NHNN |
| Agents (3) | requirements-validator, story-prioritizer, risk-assessor | Validate, ưu tiên, đánh giá rủi ro |
| Skills (3) | requirements-gathering, backlog-management, sprint-planning | User stories, grooming, sprint planning |
| Hooks (1) | post-batch-sync | Check TODO/FIXME + uncommitted changes |
| MCP (2) | Jira, Confluence | Issue tracker + wiki |

---

## 1. Steering — Ngữ cảnh tự động (bạn không cần làm gì)

Steering files được Kiro **tự động đọc** khi bạn chat. Bạn không cần gọi, không cần nhớ — chúng luôn ở đó.

### 📄 product.md — Luôn nạp (always)

Kiro luôn biết bạn đang làm việc trong domain ngân hàng VN.

**Ví dụ — bạn chỉ cần gõ:**
```
Viết API spec cho tính năng xem số dư tài khoản
```

**Kiro tự động biết (nhờ product.md):**
- Đơn vị tiền: VND, không có phần thập phân
- Phải ghi audit trail cho mọi thao tác
- Stakeholders: Khối Bán lẻ, Khối CNTT
- Definition of Done: code review + 80% test coverage + security scan

**Nếu KHÔNG có product.md**, bạn phải gõ:
```
Viết API spec cho tính năng xem số dư tài khoản.
Lưu ý: đơn vị VND không decimal, phải có audit trail,
hệ thống core banking, stakeholders gồm Khối Bán lẻ và CNTT,
DoD cần code review, 80% coverage, security scan...
```

---

### 📄 backlog-standards.md — Tự nạp khi nhắc đến backlog/story

**Ví dụ — bạn gõ:**
```
Viết user story cho tính năng đổi PIN thẻ tín dụng
```

**Kiro tự nạp backlog-standards.md** (nhận diện "user story") và viết đúng format:

```markdown
### BANK-456 Đổi PIN thẻ tín dụng

**Với vai trò** chủ thẻ tín dụng,
**Tôi muốn** đổi PIN thẻ qua mobile banking,
**Để** tăng bảo mật mà không cần đến quầy.

#### Acceptance Criteria
- **Cho trước** khách hàng đã đăng nhập và có thẻ tín dụng active,
  **Khi** chọn "Đổi PIN" và nhập PIN cũ đúng + PIN mới (6 số),
  **Thì** hệ thống cập nhật PIN thành công và gửi SMS xác nhận.

- **Cho trước** khách hàng nhập sai PIN cũ 3 lần,
  **Khi** thử lần thứ 4,
  **Thì** hệ thống khóa tính năng đổi PIN trong 24 giờ
  và gửi cảnh báo qua SMS + push notification.

- **Cho trước** PIN mới trùng với 3 PIN gần nhất,
  **Khi** submit,
  **Thì** hiển thị lỗi "PIN mới không được trùng với PIN đã sử dụng gần đây".

#### Ưu tiên: P2 | Story Points: 5
```

**Nếu KHÔNG có backlog-standards.md**, Kiro sẽ viết story format tùy ý, không có ưu tiên P0-P3, không có thang Fibonacci, không có Definition of Ready.

---

### 📄 compliance-sbv.md — Tự nạp khi nhắc đến tuân thủ/bảo mật

**Ví dụ — bạn gõ:**
```
Tính năng chuyển khoản liên ngân hàng cần lưu ý gì về compliance?
```

**Kiro tự nạp compliance-sbv.md** và trả lời với quy định cụ thể:

```
Chuyển khoản liên ngân hàng cần tuân thủ:

1. Thông tư 35/2016/TT-NHNN:
   - Bắt buộc 2FA (OTP SMS hoặc Soft Token)
   - Giới hạn giao dịch theo cấp xác thực
   - Session timeout: 5 phút (mobile), 10 phút (web)

2. Nghị định 13/2023/NĐ-CP:
   - Thông tin người nhận là dữ liệu cá nhân → cần consent
   - Lịch sử giao dịch là dữ liệu nhạy cảm → mã hóa AES-256

3. Thông tư 09/2020/TT-NHNN:
   - Ghi audit log mọi giao dịch (immutable, lưu 10 năm)
   - Cảnh báo real-time cho giao dịch bất thường
```

**Nếu KHÔNG có compliance-sbv.md**, Kiro chỉ trả lời chung chung "cần bảo mật, cần 2FA" mà không cite được quy định cụ thể.

---

## 2. Skills — Quy trình theo yêu cầu (bạn gõ lệnh)

Skills là quy trình có cấu trúc. Bạn gọi bằng cách gõ tên skill hoặc mô tả yêu cầu.

### 🛠 /requirements-gathering — Viết user stories từ đầu

**Ví dụ — bạn gõ:**
```
/requirements-gathering

Tính năng: Khách hàng mở tài khoản tiết kiệm online
```

**Kiro chạy quy trình 6 bước:**

```
Bước 1 — Thu thập: Kiro hỏi bạn
  "Loại tiết kiệm nào? (có kỳ hạn / không kỳ hạn / cả hai?)
   Kỳ hạn tối thiểu? Lãi suất cố định hay thả nổi?
   Số tiền gửi tối thiểu? Nguồn tiền từ tài khoản thanh toán?"

Bước 2 — Tài liệu hóa: Viết stories theo template
  → Story 1: Mở sổ tiết kiệm có kỳ hạn
  → Story 2: Mở sổ tiết kiệm không kỳ hạn
  → Story 3: Tất toán sổ tiết kiệm trước hạn

Bước 3 — Acceptance Criteria: Tạo Cho trước/Khi/Thì
  → Mỗi story ≥ 3 AC, bao gồm happy path + error cases

Bước 4 — Kiểm tra pháp quy:
  ⚠️ "Lãi suất tiết kiệm phải tuân thủ trần lãi suất NHNN"
  ⚠️ "Mở tài khoản online cần eKYC theo TT 16/2020/TT-NHNN"
  ⚠️ "Dữ liệu khách hàng cần DPIA theo NĐ 13/2023"

Bước 5 — Phụ thuộc:
  → Core Banking API (tạo sổ tiết kiệm)
  → eKYC service (nếu khách hàng mới)
  → SMS Gateway (gửi OTP xác nhận)

Bước 6 — Validate: Gọi requirements-validator agent kiểm tra
```

---

### 🛠 /backlog-management — Grooming và phân loại

**Ví dụ — bạn gõ:**
```
/backlog-management

Tôi có 8 issues cần triage cho grooming chiều nay:
1. BANK-101: Lỗi hiển thị số dư sai sau chuyển khoản
2. BANK-102: Thêm tính năng scan QR thanh toán
3. BANK-103: Cập nhật báo cáo NHNN quý 2 (deadline 15/07)
4. BANK-104: Refactor module xác thực
5. BANK-105: Lỗi timeout khi chuyển khoản Napas giờ cao điểm
6. BANK-106: Thêm dark mode cho mobile app
7. BANK-107: Vá lỗ hổng CVE-2024-xxxx trong thư viện JWT
8. BANK-108: Cải thiện tốc độ load danh sách giao dịch
```

**Kiro phân loại theo framework ngân hàng VN:**

```
📋 Kết quả Triage:

P0 — Khẩn cấp (trong ngày):
  BANK-107: Vá CVE JWT → Lỗ hổng bảo mật, TT 09/2020 yêu cầu patch ngay
  BANK-103: Báo cáo NHNN Q2 → Deadline 15/07 không thương lượng

P1 — Cao (trong sprint):
  BANK-101: Lỗi số dư sai → Bug chặn khách hàng, ảnh hưởng giao dịch
  BANK-105: Timeout Napas → Ảnh hưởng chuyển khoản liên ngân hàng

P2 — Bình thường (2 sprint tới):
  BANK-102: QR thanh toán → Tính năng doanh thu mới
  BANK-104: Refactor xác thực → Nợ kỹ thuật, cải thiện velocity
  BANK-108: Tốc độ load → Cải thiện UX

P3 — Thấp (backlog):
  BANK-106: Dark mode → Nice-to-have, không ảnh hưởng nghiệp vụ
```

---

### 🛠 /sprint-planning — Lập kế hoạch sprint

**Ví dụ — bạn gõ:**
```
/sprint-planning

Sprint 47, team 6 người, 2 tuần.
Velocity 3 sprint gần nhất: 34, 38, 36 points.
Lưu ý: 1 dev nghỉ phép 3 ngày, tuần sau có kiểm toán nội bộ.
```

**Kiro lập kế hoạch:**

```
📊 Sprint 47 Planning:

Velocity trung bình: 36 points
Capacity điều chỉnh:
  - 1 dev nghỉ 3/10 ngày → giảm ~5 points
  - Kiểm toán nội bộ → dự trữ 10% (~3 points) cho hỗ trợ
  - Buffer production 20% → ~6 points
  → Cam kết đề xuất: 22-24 points

Đề xuất kéo stories:
  ✅ BANK-107 (P0, 2pts) — Vá CVE JWT
  ✅ BANK-103 (P0, 5pts) — Báo cáo NHNN Q2
  ✅ BANK-101 (P1, 3pts) — Fix lỗi số dư
  ✅ BANK-105 (P1, 5pts) — Fix timeout Napas
  ✅ BANK-102 (P2, 8pts) — QR thanh toán
  → Tổng: 23 points ✅ trong capacity

Rủi ro sprint:
  | Rủi ro | Khả năng | Tác động | Giảm thiểu |
  |--------|----------|----------|------------|
  | Kiểm toán yêu cầu thêm data | Có thể | Trung bình | Dự trữ 3pts |
  | CVE patch gây regression | Ít có thể | Cao | Regression test trước merge |
```

---

## 3. Agents — Chuyên gia AI (Kiro tự gọi khi cần)

Agents là AI chuyên biệt. Bạn không cần nhớ tên — chỉ cần mô tả yêu cầu, Kiro tự chọn agent phù hợp.

### 🤖 requirements-validator — Validate requirements

**Ví dụ — bạn gõ:**
```
Validate requirements trong file sprint-47-stories.md
```

**Kiro gọi requirements-validator, agent trả về:**

```
📋 Báo cáo Validate Requirements

BANK-102: QR Thanh toán
  ❌ KHÔNG ĐẠT — Rủi ro CAO
  - Thiếu AC cho trường hợp QR hết hạn (QR thường expire sau 5 phút)
  - Thiếu đơn vị tiền tệ trong AC "số tiền hiển thị đúng"
    → Sửa: "số tiền hiển thị đúng theo format VND (không decimal, có dấu chấm phân cách hàng nghìn)"
  - ⚠️ Thanh toán QR liên quan PCI-DSS → cần đánh giá phạm vi
  - ⚠️ Cần 2FA theo TT 35/2016 nếu giao dịch > hạn mức

BANK-101: Fix lỗi số dư
  ✅ ĐẠT — Rủi ro THẤP
  - 3 AC đầy đủ, có error case
  - Đã ghi rõ VND, quy tắc làm tròn

BANK-105: Fix timeout Napas
  ⚠️ ĐẠT CÓ ĐIỀU KIỆN — Rủi ro TRUNG BÌNH
  - AC thiếu timeout threshold cụ thể (bao nhiêu giây là timeout?)
  - Thiếu AC cho retry logic (retry mấy lần? interval bao lâu?)
  - Đề xuất thêm: "Cho trước giao dịch Napas timeout sau 30 giây,
    Khi hệ thống retry lần 2, Thì giao dịch thành công
    và không bị trừ tiền 2 lần"
```

---

### 🤖 story-prioritizer — Ưu tiên backlog

**Ví dụ — bạn gõ:**
```
Ưu tiên 5 items backlog này cho sprint tới:
1. Thêm tính năng chuyển khoản theo lịch (scheduled transfer)
2. Fix lỗi font tiếng Việt trên sao kê PDF
3. Cập nhật API theo chuẩn Napas mới (deadline 01/09)
4. Thêm biểu đồ chi tiêu theo danh mục
5. Nâng cấp thư viện mã hóa (OpenSSL 3.x)
```

**Kiro gọi story-prioritizer, agent trả về:**

```
📊 Backlog đã ưu tiên:

| # | Item | Ưu tiên | Sprint | Lý do |
|---|------|---------|--------|-------|
| 1 | API Napas mới | P1 | Hiện tại | Deadline 01/09 từ Napas — không thương lượng |
| 2 | Nâng cấp OpenSSL | P1 | Hiện tại | OpenSSL 1.x EOL, TT 09/2020 yêu cầu patch |
| 3 | Fix font tiếng Việt | P2 | Tiếp theo | Ảnh hưởng UX khách hàng, fix nhanh (2pts) |
| 4 | Chuyển khoản theo lịch | P2 | Tiếp theo | Tính năng doanh thu, cần design + API |
| 5 | Biểu đồ chi tiêu | P3 | Backlog | Nice-to-have, không ảnh hưởng nghiệp vụ core |

Phụ thuộc:
  - API Napas mới → cần spec từ Napas (đã nhận chưa?)
  - Chuyển khoản theo lịch → phụ thuộc scheduler service
```

---

### 🤖 risk-assessor — Đánh giá rủi ro

**Ví dụ — bạn gõ:**
```
Đánh giá rủi ro cho dự án triển khai eKYC trong Q3
```

**Kiro gọi risk-assessor, agent trả về:**

```
📋 Sổ đăng ký rủi ro — Dự án eKYC Q3

| Mã | Rủi ro | Loại | Khả năng | Tác động | Mức | Giảm thiểu |
|----|--------|------|----------|----------|-----|------------|
| R1 | Vendor eKYC chưa hỗ trợ NFC chip CCCD VN | Kỹ thuật | Có thể | Nghiêm trọng | 🔴 Nghiêm trọng | POC với 2 vendors song song, deadline chọn vendor: tuần 2 |
| R2 | Deadline QĐ 2345 (01/07) trùng go-live | Pháp quy | Rất có thể | Cao | 🔴 Nghiêm trọng | Triển khai MVP trước deadline, bổ sung features sau |
| R3 | API C06 (CSDL dân cư) không ổn định | Kỹ thuật | Có thể | Cao | 🟠 Cao | Thiết kế fallback flow (video call KYC) |
| R4 | Team chưa có kinh nghiệm NFC/biometric | Nguồn lực | Có thể | Trung bình | 🟡 Trung bình | Training 2 tuần đầu sprint, pair với vendor |
| R5 | Deepfake bypass liveness detection | Vận hành | Ít có thể | Nghiêm trọng | 🟠 Cao | Yêu cầu vendor đạt ISO 30107-3, iBeta Level 2 |

Khuyến nghị:
  - R1 + R2 cần escalate lên Steering Committee tuần này
  - R3 nên có fallback flow trong MVP scope
  - Tổng rủi ro dự án: CAO — cần theo dõi hàng tuần
```

---

## 4. Hooks — Tự động hóa (chạy ngầm, bạn không cần làm gì)

### ⚡ post-batch-sync — Nhắc nhở sau mỗi phiên chat

Mỗi khi Kiro xử lý xong 1 lượt, hook tự động chạy script kiểm tra.

**Ví dụ — bạn vừa chat xong với Kiro, output hiện ra:**

```
Syncing issue statuses...
WARNING:    2 TODO/FIXME marker(s) found:
  sprint-47-stories.md:45: TODO: confirm lãi suất với Product
  sprint-47-stories.md:78: FIXME: thiếu AC cho timeout case
INFO:       3 uncommitted file(s) detected.
Remember to commit and push before marking sprint items as Done.
Status sync complete.
```

**Ý nghĩa:**
- `WARNING TODO/FIXME` → Bạn còn 2 chỗ chưa hoàn thiện, review trước khi đánh Done trên Jira
- `INFO uncommitted` → Bạn có 3 files đã sửa nhưng chưa commit, nhớ commit + push

**Nếu KHÔNG có hook này**, bạn có thể quên commit hoặc bỏ sót TODO, đánh Done trên Jira trong khi stories chưa thực sự hoàn thiện.

---

## 5. MCP — Kết nối Jira & Confluence (chat trực tiếp)

MCP cho phép Kiro gọi API Jira/Confluence ngay trong chat, không cần mở tab khác.

### 🔗 Jira — Tìm issues, xem sprint

**Ví dụ 1 — Tìm issues:**
```
Tìm tất cả bugs P0-P1 chưa resolve trong sprint hiện tại
```

**Kiro gọi Jira API (auto-approved, không hỏi bạn):**
```
Tìm thấy 3 issues:

| Key | Summary | Priority | Assignee | Status |
|-----|---------|----------|----------|--------|
| BANK-101 | Lỗi số dư sai sau chuyển khoản | P1 | Minh | In Progress |
| BANK-105 | Timeout Napas giờ cao điểm | P1 | Hùng | Ready |
| BANK-107 | CVE JWT library | P0 | Chưa assign | Backlog |

⚠️ BANK-107 (P0) chưa có assignee — cần assign ngay.
```

**Ví dụ 2 — Xem chi tiết issue:**
```
Xem chi tiết BANK-105
```

**Kiro trả về:**
```
BANK-105: Timeout chuyển khoản Napas giờ cao điểm

Status: Ready | Priority: P1 | Points: 5
Assignee: Hùng | Sprint: Sprint 46
Created: 2026-04-15 | Updated: 2026-04-20

Description:
  Khách hàng báo chuyển khoản liên ngân hàng qua Napas
  bị timeout trong khung 11:00-13:00. Tỷ lệ fail ~15%.

Acceptance Criteria:
  - Timeout rate < 2% trong giờ cao điểm
  - Retry tự động tối đa 2 lần
  - Thông báo rõ ràng cho khách hàng khi fail

Comments (2):
  - Hùng: "Nghi do connection pool Napas gateway không đủ"
  - Minh: "Cần check log Napas từ 11:00-13:00 ngày 14/04"
```

**Ví dụ 3 — Tạo issue (cần xác nhận):**
```
Tạo bug mới: Sao kê PDF hiển thị sai ký tự tiếng Việt có dấu
```

**Kiro hỏi xác nhận trước khi tạo:**
```
Tạo issue mới trên Jira:
  Project: BANK
  Type: Bug
  Summary: Sao kê PDF hiển thị sai ký tự tiếng Việt có dấu
  Priority: P2
  Labels: [pdf, i18n, ux]

Bạn xác nhận tạo? (Kiro sẽ hỏi vì đây không phải thao tác auto-approved)
```

### 🔗 Confluence — Tìm và đọc wiki (disabled mặc định)

Bật khi cần bằng cách sửa `disabled: false` trong `.kiro/settings/mcp.json`.

**Ví dụ khi đã bật:**
```
Tìm tài liệu API chuyển khoản Napas trên Confluence
```

**Kiro tìm và trả về:**
```
Tìm thấy 2 trang liên quan:
1. "API Specification — Napas Interbank Transfer v2.1" (cập nhật 2026-03-15)
2. "Napas Integration Troubleshooting Guide" (cập nhật 2026-01-20)

Trang 1 chứa:
  - Endpoint: POST /api/v2/napas/transfer
  - Timeout: 30 giây
  - Retry policy: max 2 lần, interval 5 giây
  - Error codes: N001 (timeout), N002 (insufficient funds)...
```

---

## Cài đặt

### Copy config vào project
```bash
# Copy .kiro/ và AGENTS.md vào project của bạn
cp -r .kiro/ /path/to/your-project/
cp AGENTS.md /path/to/your-project/
chmod +x /path/to/your-project/.kiro/hooks/scripts/sync-issue-status.sh
```

### Cấu hình Jira credentials
Thêm env vars vào Kiro IDE settings → "Mcp Approved Env Vars":
- `JIRA_API_TOKEN` — API token từ https://id.atlassian.com/manage-profile/security/api-tokens
- `JIRA_EMAIL` — Email Atlassian của bạn
- `JIRA_DOMAIN` — Domain Jira (ví dụ: `mybank.atlassian.net`)

### Bật Confluence (optional)
Sửa `.kiro/settings/mcp.json` → `confluence` → `"disabled": false`
Thêm env vars: `CONFLUENCE_API_TOKEN`, `CONFLUENCE_EMAIL`, `CONFLUENCE_DOMAIN`

---

## Quy định pháp luật áp dụng

| Quy định | Phạm vi |
|----------|---------|
| Thông tư 09/2020/TT-NHNN | An toàn hệ thống CNTT ngân hàng |
| Thông tư 35/2016/TT-NHNN | Dịch vụ ngân hàng trực tuyến (2FA, session timeout) |
| Nghị định 13/2023/NĐ-CP | Bảo vệ dữ liệu cá nhân (DPIA, consent, 72h breach notification) |
| Thông tư 41/2018/TT-NHNN | Kiểm soát nội bộ (SoD, access review) |
| PCI-DSS v4.0 | Xử lý thẻ thanh toán |
| Luật Kế toán 2015 | Lưu trữ hồ sơ tài chính tối thiểu 10 năm |
