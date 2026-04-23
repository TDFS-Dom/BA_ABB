# Sprint 47 — Stories

> Sprint Goal: Đảm bảo an toàn hệ thống (patch CVE, fix bugs tài chính) và hoàn thành báo cáo NHNN Q2 đúng deadline pháp quy.
>
> Duration: 23/04 — 07/05/2026 | Team: 6 người | Capacity: 26 pts | Committed: 23 pts

---

## BANK-107 — Vá lỗ hổng CVE-2024-21319 trong thư viện JWT

**Ưu tiên:** P0 | **Story Points:** 2 | **Assignee:** Hùng | **Status:** Ready

**Với vai trò** kỹ sư bảo mật,
**Tôi muốn** cập nhật thư viện JWT lên phiên bản đã vá CVE-2024-21319,
**Để** ngăn chặn tấn công giả mạo token và đảm bảo tuân thủ TT 09/2020/TT-NHNN.

### Acceptance Criteria

- **Cho trước** hệ thống đang dùng jsonwebtoken@8.5.1 (vulnerable),
  **Khi** nâng cấp lên jsonwebtoken@9.0.2+,
  **Thì** không còn CVE critical/high trong dependency scan (Snyk/Trivy pass).

- **Cho trước** JWT token hợp lệ được tạo trước khi patch,
  **Khi** verify bằng phiên bản mới,
  **Thì** token vẫn valid — không yêu cầu user đăng nhập lại.

- **Cho trước** attacker gửi crafted JWT khai thác CVE-2024-21319,
  **Khi** hệ thống verify token,
  **Thì** trả về 401 Unauthorized và ghi audit log cảnh báo.

### Ghi chú
- Chạy regression test toàn bộ auth flow trước khi merge
- Rollback plan: revert dependency + redeploy trong 15 phút
- ⚠️ BANK-102 (QR thanh toán) phụ thuộc story này — hoàn thành trước cuối tuần 1

---

## BANK-103 — Cập nhật báo cáo NHNN quý 2/2026

**Ưu tiên:** P0 | **Story Points:** 5 | **Assignee:** Lan | **Status:** Ready

**Với vai trò** chuyên viên tuân thủ,
**Tôi muốn** hệ thống tự động tạo báo cáo an toàn CNTT quý 2/2026 theo mẫu NHNN,
**Để** nộp đúng deadline 15/07/2026 và tránh bị xử phạt hành chính.

### Acceptance Criteria

- **Cho trước** dữ liệu giao dịch Q2 (01/04 — 30/06/2026) đã được extract từ Data Warehouse,
  **Khi** chạy job tạo báo cáo,
  **Thì** file báo cáo đúng mẫu NHNN, bao gồm: tổng giao dịch, phân loại theo kênh, sự cố CNTT, biện pháp khắc phục.

- **Cho trước** báo cáo đã tạo xong,
  **Khi** chuyên viên tuân thủ review,
  **Thì** có thể export PDF + Excel, có chữ ký số của người phê duyệt.

- **Cho trước** dữ liệu Data Warehouse chưa sẵn sàng (extract chưa chạy),
  **Khi** chạy job tạo báo cáo,
  **Thì** hiển thị lỗi rõ ràng "Dữ liệu Q2 chưa sẵn sàng — liên hệ team DW" và ghi log.

- **Cho trước** báo cáo Q1 đã nộp trước đó,
  **Khi** tạo báo cáo Q2,
  **Thì** số liệu lũy kế (YTD) phải khớp = Q1 + Q2, đơn vị VND không decimal.

### Ghi chú
- Phụ thuộc: Data Warehouse team extract data — confirm trước sprint start
- Deadline pháp quy 15/07 — không thương lượng
- Dev Lan nghỉ phép 3 ngày (27-29/04) — pair với Minh trước khi nghỉ

---

## BANK-101 — Fix lỗi hiển thị số dư sai sau chuyển khoản

**Ưu tiên:** P1 | **Story Points:** 3 | **Assignee:** Minh | **Status:** In Progress

**Với vai trò** khách hàng cá nhân,
**Tôi muốn** số dư tài khoản hiển thị chính xác ngay sau khi chuyển khoản thành công,
**Để** tôi biết chính xác số tiền còn lại mà không cần refresh.

### Acceptance Criteria

- **Cho trước** tài khoản có số dư 10.000.000 VND,
  **Khi** chuyển khoản 2.000.000 VND thành công,
  **Thì** số dư hiển thị ngay lập tức là 8.000.000 VND (VND, không decimal, có dấu chấm phân cách).

- **Cho trước** chuyển khoản thành công trên mobile banking,
  **Khi** mở web banking cùng lúc,
  **Thì** số dư trên web cũng cập nhật trong vòng 3 giây (eventual consistency).

- **Cho trước** chuyển khoản bị lỗi giữa chừng (timeout/network error),
  **Khi** hệ thống rollback giao dịch,
  **Thì** số dư giữ nguyên 10.000.000 VND và hiển thị thông báo "Giao dịch không thành công".

### Ghi chú
- Root cause nghi ngờ: cache invalidation không trigger sau khi Core Banking API trả success
- Tính toán số dư dùng BigDecimal — không floating point
- Audit trail: ghi log mọi thay đổi số dư

---

## BANK-105 — Fix timeout chuyển khoản Napas giờ cao điểm

**Ưu tiên:** P1 | **Story Points:** 5 | **Assignee:** Hùng | **Status:** Ready

**Với vai trò** khách hàng chuyển khoản liên ngân hàng,
**Tôi muốn** giao dịch qua Napas không bị timeout trong giờ cao điểm (11:00-13:00),
**Để** tôi chuyển tiền thành công mà không phải thử lại nhiều lần.

### Acceptance Criteria

- **Cho trước** khung giờ cao điểm 11:00-13:00 với traffic gấp 3 lần bình thường,
  **Khi** khách hàng chuyển khoản liên ngân hàng qua Napas,
  **Thì** tỷ lệ timeout giảm từ 15% xuống dưới 2%.

- **Cho trước** giao dịch Napas timeout sau 30 giây,
  **Khi** hệ thống tự động retry (tối đa 2 lần, interval 5 giây),
  **Thì** giao dịch thành công ở lần retry và khách hàng KHÔNG bị trừ tiền 2 lần.

- **Cho trước** giao dịch Napas fail sau 2 lần retry,
  **Khi** hệ thống dừng retry,
  **Thì** hiển thị thông báo "Giao dịch tạm thời không thể thực hiện. Vui lòng thử lại sau 5 phút"
  và ghi audit log với mã lỗi Napas (N001/N002/...).

- **Cho trước** connection pool Napas gateway hiện tại là 50 connections,
  **Khi** tăng lên 150 connections + circuit breaker (threshold 60% fail trong 30s),
  **Thì** latency P99 giảm từ 45s xuống dưới 10s trong giờ cao điểm.

### Ghi chú
- Phụ thuộc: cần log từ Napas gateway ngày 14/04 — đã gửi request
- Root cause nghi ngờ: connection pool không đủ + không có circuit breaker
- Cần load test trên staging trước khi deploy production
- ⚠️ Đảm bảo idempotency — không trừ tiền 2 lần khi retry

---

## BANK-102 — Tính năng scan QR thanh toán

**Ưu tiên:** P2 | **Story Points:** 8 | **Assignee:** Tuấn | **Status:** Ready

**Với vai trò** khách hàng cá nhân,
**Tôi muốn** scan mã QR để thanh toán nhanh tại cửa hàng,
**Để** tôi không cần nhập thủ công số tài khoản và số tiền.

### Acceptance Criteria

- **Cho trước** khách hàng mở camera scan QR trên mobile banking,
  **Khi** scan mã QR hợp lệ theo chuẩn VietQR (NAPAS),
  **Thì** hiển thị màn hình xác nhận với: tên người nhận, ngân hàng, số tiền (VND, có dấu chấm phân cách), nội dung chuyển khoản.

- **Cho trước** mã QR chứa số tiền cố định (static amount),
  **Khi** khách hàng xác nhận thanh toán và nhập OTP,
  **Thì** giao dịch thành công, trừ đúng số tiền trên QR, gửi biên lai qua push notification.

- **Cho trước** mã QR không chứa số tiền (dynamic amount),
  **Khi** khách hàng scan,
  **Thì** hiển thị form nhập số tiền (validate: dương, khác 0, trong hạn mức tài khoản, VND không decimal).

- **Cho trước** mã QR hết hạn (expired > 5 phút) hoặc không đúng chuẩn VietQR,
  **Khi** scan,
  **Thì** hiển thị lỗi "Mã QR không hợp lệ hoặc đã hết hạn. Vui lòng yêu cầu mã mới."

- **Cho trước** giao dịch QR thanh toán > 10.000.000 VND,
  **Khi** xác nhận,
  **Thì** yêu cầu xác thực 2FA (OTP SMS hoặc Soft Token) theo TT 35/2016/TT-NHNN.

### Ghi chú
- Phụ thuộc: BANK-107 (JWT patch) phải hoàn thành trước — QR flow dùng JWT auth
- Tuân thủ: PCI-DSS scope assessment cần hoàn thành trước dev
- Tuân thủ: 2FA bắt buộc cho giao dịch trên hạn mức (TT 35/2016)
- Format tiền: VND, BigDecimal, làm tròn đến đồng, dấu chấm phân cách hàng nghìn
- TODO: confirm hạn mức 2FA với Product (10 triệu hay 20 triệu?)

---

## Stretch Goal

### BANK-108 — Cải thiện tốc độ load danh sách giao dịch

**Ưu tiên:** P2 | **Story Points:** 3 | **Assignee:** — | **Status:** Backlog

**Với vai trò** khách hàng,
**Tôi muốn** danh sách giao dịch load trong vòng 2 giây,
**Để** tôi xem lịch sử giao dịch nhanh chóng mà không phải chờ đợi.

### Acceptance Criteria

- **Cho trước** tài khoản có > 1.000 giao dịch trong 3 tháng gần nhất,
  **Khi** mở màn hình lịch sử giao dịch,
  **Thì** 20 giao dịch gần nhất hiển thị trong ≤ 2 giây (P95).

- **Cho trước** khách hàng scroll xuống cuối danh sách,
  **Khi** trigger infinite scroll load thêm 20 giao dịch,
  **Thì** load trong ≤ 1 giây, không bị giật/lag UI.

- **Cho trước** API trả về danh sách giao dịch,
  **Khi** hiển thị,
  **Thì** mỗi dòng giao dịch gồm: ngày giờ, mô tả, số tiền (VND format), số dư sau giao dịch.

### Ghi chú
- Chỉ kéo vào nếu team hoàn thành 5 stories chính trước ngày 05/05
- Giải pháp dự kiến: thêm index + pagination phía DB, cache layer cho query phổ biến
