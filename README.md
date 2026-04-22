# BA Workspace — Spec Verification Toolkit

Workspace dành cho Business Analyst, tập trung vào việc viết và **kiểm chứng chất lượng spec** trước khi chuyển sang development.

## Cấu trúc

```
BA_ABB/
└── ekyc-banking-analysis.md    # Phân tích eKYC cho ngân hàng

.kiro/steering/
├── verify-correctness.md       # Rule: kiểm tra tính đúng đắn
├── verify-completeness.md      # Rule: kiểm tra tính đầy đủ
└── no-spec-workflow.md         # Rule: không tự chạy spec workflow
```

## Hai phương pháp kiểm chứng Spec

Workspace cung cấp 2 steering rules hoạt động bổ trợ cho nhau. Mỗi method có scope riêng, không chồng chéo.

### 1. Verify Correctness — "Spec có đúng không?"

Kiểm tra từng statement trong spec có khớp với thực tế bên ngoài hay không.

**Cách hoạt động:**
- Đọc spec → search internet lấy thông tin regulation, industry standards, threat data, vendor specs
- So sánh từng statement với nguồn bên ngoài → flag chỗ sai hoặc mơ hồ

**Flag khi:**
- Mâu thuẫn với regulation (spec nói X, luật yêu cầu Y)
- Thiếu con số cụ thể mà regulation/standard đã define (threshold, timeout, retention period...)
- Thiếu behavior mà best practice khuyến nghị
- Acceptance criteria không verifiable — QA không biết test thế nào

**Output:** Mỗi finding dùng ❓, kèm statement gốc → vấn đề → nguồn tham khảo → suggest fix. Kết thúc bằng bảng trước/sau.

**Trigger:** Nói với Kiro: *"verify correctness"*, *"check correctness"*, hoặc *"review spec"*.

---

### 2. Verify Completeness — "Spec có đủ không?"

Tìm scenarios bị thiếu bằng **Inversion Method** — lật ngược mọi bước trong happy path để hỏi "nếu sai thì sao?".

**Cách hoạt động:**
- Với MỖI step trong golden path, hỏi 6 câu:
  1. Input sai? → Validation, error message, recovery
  2. Service/API fail? → Fallback, retry, error state
  3. Chậm/timeout? → Loading state, timeout threshold
  4. Không có quyền? → Auth check, redirect
  5. Data rỗng? → Empty state, first-time experience
  6. Xung đột? → Concurrent edit, duplicate submit, stale data

- Sau đó check cross-cutting concerns: data retention, audit trail, compliance, app crash/mất mạng, rate limiting.

- Nếu workspace có internal knowledge files (checklist, gap-log, lessons-learned) → cross-check để flag gaps đã từng xảy ra ở dự án trước.

**Output:** Mỗi gap dùng ❌, group theo step, kèm severity. Kết thúc bằng Gap Report tổng hợp.

**Trigger:** Nói với Kiro: *"verify completeness"*, *"tìm gaps"*, hoặc *"chạy Inversion Method"*.

---

### Khi nào dùng method nào?

| Mục tiêu | Method | Ví dụ |
|---|---|---|
| Spec có khớp luật/regulation không? | Correctness | "QĐ 2345 yêu cầu gì mà spec chưa cover?" |
| Spec có thiếu edge case không? | Completeness | "Nếu NFC fail thì user thấy gì?" |
| Review toàn diện trước khi dev | Cả hai | Chạy Correctness trước, Completeness sau |

## Lưu ý

- Hai method **không chồng chéo**: Correctness không tìm scenario thiếu, Completeness không flag statement mơ hồ.
- Steering rules tự động active — chỉ cần nói đúng trigger phrase trong chat.
- Workspace không tự chạy Kiro spec workflow khi mở file trong `.kiro/specs/` (xem `no-spec-workflow.md`).
