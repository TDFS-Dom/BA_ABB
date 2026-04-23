---
inclusion: auto
name: sbv-compliance
description: Quy định NHNN về an toàn CNTT ngân hàng và bảo vệ dữ liệu cá nhân tại Việt Nam
---
## Quy định NHNN & Pháp luật Việt Nam — Yêu cầu chính

### Thông tư 09/2020/TT-NHNN (An toàn hệ thống CNTT ngân hàng)
- Phân loại hệ thống CNTT theo mức độ quan trọng (Cấp 1-5)
- Core Banking = Cấp 5 (quan trọng nhất)
- Yêu cầu đánh giá rủi ro CNTT định kỳ hàng năm
- Kiểm thử xâm nhập (pentest) tối thiểu 1 lần/năm
- Kế hoạch dự phòng và khôi phục thảm họa bắt buộc

### Thông tư 35/2016/TT-NHNN (Dịch vụ ngân hàng trực tuyến)
- Xác thực tối thiểu 2 yếu tố (2FA) cho giao dịch tài chính
- OTP qua SMS hoặc Soft Token cho giao dịch chuyển tiền
- Giới hạn giao dịch theo cấp xác thực
- Phiên đăng nhập timeout: 5 phút cho mobile banking, 10 phút cho internet banking

### Nghị định 13/2023/NĐ-CP (Bảo vệ dữ liệu cá nhân)
- Dữ liệu cá nhân: họ tên, CCCD/CMND, SĐT, email, địa chỉ, tài khoản ngân hàng
- Dữ liệu nhạy cảm: thông tin tài chính, lịch sử giao dịch, điểm tín dụng
- Phải có sự đồng ý rõ ràng trước khi thu thập/xử lý
- Quyền của chủ thể: truy cập, chỉnh sửa, xóa, rút lại đồng ý
- Đánh giá tác động xử lý dữ liệu cá nhân (DPIA) bắt buộc cho hệ thống mới
- Thông báo vi phạm dữ liệu trong vòng 72 giờ cho Bộ Công an

### Thông tư 41/2018/TT-NHNN (Kiểm soát nội bộ)
- Phân tách nhiệm vụ (Segregation of Duties): người phát triển không được deploy production
- Kiểm soát truy cập theo nguyên tắc quyền tối thiểu
- Rà soát quyền truy cập hệ thống định kỳ hàng quý
- Ghi log mọi thao tác thay đổi trạng thái

### Khả dụng & Khôi phục
- Hệ thống Core Banking: uptime ≥ 99.9%
- RTO (Recovery Time Objective): ≤ 4 giờ cho hệ thống Cấp 5
- RPO (Recovery Point Objective): ≤ 1 giờ cho dữ liệu giao dịch
- Diễn tập DR (Disaster Recovery) tối thiểu 1 lần/năm

### Mã hóa & Bảo mật
- Mã hóa dữ liệu lưu trữ: AES-256
- Mã hóa truyền tải: TLS 1.2 trở lên (khuyến nghị TLS 1.3)
- Quản lý khóa qua HSM hoặc KMS
- Dữ liệu khách hàng phải lưu trữ tại Việt Nam (data residency)

### Quản lý thay đổi
- Mọi thay đổi production cần phê duyệt CAB (Change Advisory Board)
- Thay đổi khẩn cấp: review hậu kiểm trong 48 giờ
- Kế hoạch rollback bắt buộc cho mọi deployment production

### Audit & Logging
- Lưu trữ audit log: tối thiểu 10 năm (theo Luật Kế toán 2015)
- Log không thể sửa đổi (immutable/append-only)
- Cảnh báo real-time cho hoạt động bất thường
- Rà soát log bảo mật hàng tháng
