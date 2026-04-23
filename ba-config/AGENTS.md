# Nền tảng Ngân hàng Việt Nam — Hướng dẫn Business Analyst

## Quy tắc bắt buộc (Luôn áp dụng)
1. Mọi user story phải theo format: Với vai trò / Tôi muốn / Để
2. Tối thiểu 3 acceptance criteria mỗi story (Cho trước / Khi / Thì)
3. Mọi tính năng liên quan tiền phải chỉ rõ đơn vị tiền tệ và quy tắc làm tròn
4. Tính năng xử lý dữ liệu cá nhân phải đánh giá tác động theo Nghị định 13/2023/NĐ-CP
5. Tính năng thanh toán thẻ phải đánh giá phạm vi PCI-DSS
6. Thay đổi xác thực phải review theo Thông tư 09/2020/TT-NHNN
7. Mọi deadline quy định pháp luật là ưu tiên không thương lượng

## Tuân thủ pháp quy
- Thông tư 09/2020/TT-NHNN (An toàn hệ thống CNTT ngân hàng) — bắt buộc
- Thông tư 35/2016/TT-NHNN (An toàn, bảo mật cho dịch vụ ngân hàng trực tuyến)
- Nghị định 13/2023/NĐ-CP (Bảo vệ dữ liệu cá nhân)
- PCI-DSS v4.0 cho xử lý thẻ thanh toán
- Thông tư 41/2018/TT-NHNN (Hệ thống kiểm soát nội bộ)

## Khi không chắc chắn
- Câu hỏi tuân thủ → check compliance-sbv steering
- Validate requirements → invoke requirements-validator subagent
- Ưu tiên backlog → invoke story-prioritizer subagent
- Đánh giá rủi ro → invoke risk-assessor subagent
