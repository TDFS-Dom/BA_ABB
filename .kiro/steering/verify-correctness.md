---
inclusion: always
---

# Verify Correctness — Steering
## Capability: Web Search + Spec Review

## Khi nào áp dụng
Khi user yêu cầu review spec, check correctness, hoặc verify spec với thực tế bên ngoài.

## Phương pháp
1. Đọc spec để hiểu domain và context
2. **Search internet** để lấy thông tin liên quan:
   - Quy định pháp lý áp dụng cho domain đó
   - Industry standards và best practices
   - Threat landscape, fraud trends mới nhất
   - Thông tin kỹ thuật (vendor capabilities, protocol specs...)
3. So sánh spec với thông tin tìm được → flag chỗ mơ hồ, sai, hoặc lệch thực tế

## Quy tắc
Với MỖI statement trong spec, hỏi: "Statement này có đúng và đủ rõ so với thực tế bên ngoài không?"

Flag nếu:
- Mâu thuẫn với regulation — spec nói X nhưng quy định yêu cầu Y
- Thiếu con số mà regulation/standard đã define — threshold, timeout, retention period...
- Thiếu behavior mà best practice khuyến nghị
- Thiếu kênh/method cụ thể — "gửi notification" nhưng không nói push/email/SMS
- AC không verifiable — QA không biết test thế nào, pass/fail dựa vào đâu

## Format output
- Dùng ❓ emoji cho mỗi finding
- Mỗi finding: statement gốc → vấn đề → thông tin từ web search → suggest fix
- Cite nguồn khi reference regulation hoặc data từ internet
- Cuối cùng: bảng trước/sau cho mỗi statement cần fix
- KHÔNG tìm scenarios thiếu theo Inversion Method (đó là việc của Completeness)
