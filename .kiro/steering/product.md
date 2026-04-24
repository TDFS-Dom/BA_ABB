---
inclusion: always
---
## Nền tảng Ngân hàng — Bối cảnh sản phẩm

### Lĩnh vực kinh doanh
- Hệ thống Core Banking: Quản lý tài khoản, giao dịch, thanh toán, cho vay
- Kênh khách hàng: Web portal (React), Mobile Banking, Internet Banking, Quầy giao dịch
- Cơ quan quản lý: Ngân hàng Nhà nước Việt Nam (NHNN/SBV)
- Tuân thủ: Thông tư 09/2020/TT-NHNN, PCI-DSS v4.0, Nghị định 13/2023/NĐ-CP

### Quy tắc nghiệp vụ
- Mọi tính toán tiền tệ dùng BigDecimal — không bao giờ dùng floating point
- Số tiền giao dịch phải validate: dương, khác 0, trong hạn mức tài khoản
- Mọi thao tác khách hàng phải ghi audit trail
- Lưu trữ dữ liệu: tối thiểu 10 năm cho hồ sơ tài chính (theo Luật Kế toán 2015)
- Đơn vị tiền tệ chính: VND (không có phần thập phân)
- Hỗ trợ đa tiền tệ: VND, USD, EUR, JPY, CNY
- Làm tròn VND: luôn làm tròn đến đồng (0 decimal places)
- Tỷ giá: lấy từ NHNN hoặc tỷ giá niêm yết của ngân hàng

### Các bên liên quan
- Khối Ngân hàng Bán lẻ (khách hàng cá nhân)
- Khối Ngân hàng Doanh nghiệp (SME + Corporate)
- Khối Quản trị Rủi ro
- Khối Tuân thủ & Pháp chế
- Khối Công nghệ Thông tin
- Kiểm toán Nội bộ

### Quy tắc tra cứu trước khi trả lời
- Khi được hỏi về số liệu, quy định, biểu phí → ĐỌC file nguồn trong workspace hoặc search web TRƯỚC, không trả lời từ bộ nhớ
- Khi được hỏi về trạng thái issue, sprint → tra Jira qua MCP TRƯỚC
- Nếu không tìm được nguồn → ghi rõ "[CẦN CONFIRM: ...]"

### Quy tắc chống hallucination
- KHÔNG bịa số liệu (lãi suất, phí, hạn mức, tỷ giá). Nếu không có data → ghi "[CẦN CONFIRM: ...]"
- KHÔNG bịa tên quy định, số thông tư. Nếu không chắc → ghi "[CẦN VERIFY: ...]"
- KHÔNG bịa tên người, chức danh, team. Dùng placeholder [Tên NV], [Team X]
- Mọi số liệu trong output phải trích từ file nguồn hoặc đánh dấu rõ là giả định

### Definition of Done
- Code reviewed bởi ít nhất 1 peer
- Unit test coverage ≥ 80%
- Không có lỗ hổng critical/high từ security scan
- Checklist tuân thủ đã ký duyệt
- Tài liệu API cập nhật trên Confluence
