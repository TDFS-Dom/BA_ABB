# Ví dụ User Story — Ngân hàng Việt Nam

## Quản lý Tài khoản
### BANK-101: Xem số dư tài khoản
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** xem số dư tài khoản hiện tại trên dashboard,
**Để** theo dõi tài chính của mình theo thời gian thực.

#### Acceptance Criteria
- **Cho trước** tôi đã đăng nhập, **Khi** tôi vào trang dashboard, **Thì** tôi thấy tất cả tài khoản với số dư hiện tại bằng VND
- **Cho trước** tôi có tài khoản ngoại tệ, **Khi** tôi xem số dư, **Thì** mỗi tài khoản hiển thị đơn vị tiền gốc và quy đổi VND theo tỷ giá ngân hàng
- **Cho trước** dịch vụ số dư không khả dụng, **Khi** tôi xem dashboard, **Thì** tôi thấy số dư gần nhất với nhãn "Cập nhật lúc [thời gian]"

## Chuyển tiền
### BANK-201: Chuyển khoản nội bộ
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** chuyển tiền giữa các tài khoản của mình,
**Để** quản lý tiền giữa các tài khoản.

#### Acceptance Criteria
- **Cho trước** tôi có đủ số dư, **Khi** tôi gửi lệnh chuyển, **Thì** số tiền được trừ từ tài khoản nguồn và cộng vào tài khoản đích nguyên tử
- **Cho trước** tôi gửi lệnh chuyển trùng (cùng idempotency key), **Khi** xử lý, **Thì** chỉ thực hiện 1 giao dịch
- **Cho trước** số tiền chuyển bằng 0 hoặc âm, **Khi** tôi gửi, **Thì** tôi nhận lỗi validation
- **Cho trước** số tiền vượt hạn mức ngày, **Khi** tôi gửi, **Thì** tôi nhận thông báo vượt hạn mức kèm hạn mức còn lại
- **Cho trước** giao dịch ≥ 500 triệu VND, **Khi** tôi xác nhận, **Thì** hệ thống yêu cầu xác thực OTP bổ sung

#### Yêu cầu phi chức năng
- Latency: P95 < 200ms
- Idempotency: phát hiện giao dịch trùng trong 24 giờ
- Audit: ghi đầy đủ audit trail với số dư trước/sau
- Đơn vị: VND, làm tròn đến đồng

### BANK-202: Chuyển khoản liên ngân hàng (Napas)
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** chuyển tiền đến tài khoản ngân hàng khác qua Napas,
**Để** thanh toán cho người nhận ở ngân hàng khác.

#### Acceptance Criteria
- **Cho trước** tôi nhập đúng số tài khoản và ngân hàng đích, **Khi** tôi gửi lệnh, **Thì** hệ thống hiển thị tên người nhận để xác nhận (tra cứu Napas)
- **Cho trước** giao dịch thành công, **Khi** hoàn tất, **Thì** tôi nhận biên lai điện tử với mã giao dịch Napas
- **Cho trước** Napas không khả dụng, **Khi** tôi gửi lệnh, **Thì** hệ thống thông báo lỗi và đề xuất thử lại sau
- **Cho trước** giao dịch ngoài giờ làm việc Napas, **Khi** tôi gửi, **Thì** hệ thống thông báo giao dịch sẽ xử lý vào ngày làm việc tiếp theo

#### Yêu cầu phi chức năng
- Tuân thủ quy định Napas về format message
- Phí chuyển khoản theo biểu phí ngân hàng
- Hạn mức theo Thông tư 35/2016/TT-NHNN
