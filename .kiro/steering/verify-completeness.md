---
inclusion: always
---

# Verify Completeness — Steering
## Capability: Internal Files + Inversion Method

## Khi nào áp dụng
Khi user yêu cầu tìm gaps, chạy Inversion Method, hoặc check completeness của spec.

## Phương pháp
1. **Tìm và đọc internal knowledge files** trong workspace:
   - Tìm files có tên chứa: checklist, gap-log, lessons-learned, known-issues
   - Tìm trong các folder: internal-knowledge/, docs/, knowledge-base/
   - Nếu không tìm thấy → chạy Inversion Method thuần, không cross-check
2. Chạy Inversion Method trên spec
3. Cross-check kết quả với internal files (nếu có) → flag gaps đã từng xảy ra

## Quy tắc: The Inversion Method
Với MỖI step trong golden path, hỏi 6 câu:
1. Nếu input sai? → Validation, error message, recovery
2. Nếu service/API fail? → Fallback, retry, error state
3. Nếu chậm/timeout? → Loading state, timeout threshold, user feedback
4. Nếu không có quyền? → Auth check, redirect, message
5. Nếu data rỗng? → Empty state, first-time experience
6. Nếu xung đột? → Concurrent edit, duplicate submit, stale data

Sau 6 câu per-step, check cross-cutting concerns:
- Data retention / lưu trữ
- Audit trail / logging
- Compliance / regulatory
- App interrupted (kill, crash, mất mạng)
- Rate limiting / abuse prevention

## Format output
- Dùng ❌ emoji cho mỗi gap
- Group theo step trong golden path
- Mỗi gap: scenario → spec nói gì → severity
- Nếu gap match với internal knowledge → thêm "⚠️ Dự án trước đã gặp lỗi này: [mô tả]"
- Nếu gap match với checklist → thêm "📋 Checklist: [item]"
- Cuối cùng: Gap Report bảng tổng hợp, xếp theo severity
- KHÔNG flag statements mơ hồ (đó là việc của Correctness)
