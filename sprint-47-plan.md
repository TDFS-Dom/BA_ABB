# Sprint 47 — Plan

> **Sprint Goal:** Đảm bảo an toàn hệ thống (patch CVE, fix bugs tài chính) và hoàn thành báo cáo NHNN Q2 đúng deadline pháp quy.

---

## Tổng quan

| Thông số | Giá trị |
|----------|---------|
| Duration | 23/04 — 07/05/2026 (2 tuần) |
| Team | 6 người |
| Velocity TB (3 sprint gần nhất) | 36 pts (34, 38, 36) |
| Capacity điều chỉnh | 26 pts |
| Committed | 23 pts (5 stories) |
| Buffer | 3 pts |
| Stretch goal | BANK-108 (3 pts) |

---

## Capacity Calculation

| Yếu tố | Tác động |
|---------|----------|
| Velocity trung bình | 36 pts |
| Lan nghỉ phép 3 ngày (27-29/04) | −2 pts |
| Kiểm toán nội bộ tuần 2 | −3 pts (hỗ trợ cung cấp data, trả lời câu hỏi) |
| Buffer bugs/unexpected (15%) | −5 pts |
| **Capacity khả dụng** | **26 pts** |

---

## Committed Stories

| # | Key | Summary | P | Pts | Owner | Status |
|---|-----|---------|---|-----|-------|--------|
| 1 | BANK-107 | Vá lỗ hổng CVE JWT | P0 | 2 | Hùng | Ready |
| 2 | BANK-103 | Báo cáo NHNN Q2 | P0 | 5 | Lan | Ready |
| 3 | BANK-101 | Fix lỗi số dư sai sau chuyển khoản | P1 | 3 | Minh | In Progress |
| 4 | BANK-105 | Fix timeout Napas giờ cao điểm | P1 | 5 | Hùng | Ready |
| 5 | BANK-102 | Scan QR thanh toán | P2 | 8 | Tuấn | Ready |
| | | **Tổng** | | **23** | | |

### Stretch Goal
| Key | Summary | Pts | Điều kiện |
|-----|---------|-----|-----------|
| BANK-108 | Cải thiện tốc độ load giao dịch | 3 | Kéo vào nếu hoàn thành 5 stories trước 05/05 |

### Không kéo vào sprint
| Key | Summary | Pts | Lý do |
|-----|---------|-----|-------|
| BANK-104 | Refactor module xác thực | 5 | Vượt capacity, không urgent |
| BANK-106 | Dark mode mobile | 5 | P3, nice-to-have |

---

## Dependencies

```
BANK-107 (CVE JWT) ──→ BANK-102 (QR thanh toán)
  └─ JWT patch phải merge trước khi dev QR flow

BANK-103 (Báo cáo NHNN) ──→ Data Warehouse team
  └─ Cần data extract Q2 — confirm trước sprint start

BANK-105 (Timeout Napas) ──→ Napas Gateway (bên ngoài)
  └─ Cần log từ Napas ngày 14/04 — đã gửi request

BANK-101 (Fix số dư) ──→ Core Banking API (nội bộ)
  └─ Team tự xử lý, không phụ thuộc ngoài
```

**Critical path:** BANK-107 → BANK-102

---

## Sequencing

### Tuần 1 (23/04 — 29/04)
| Ngày | Hùng | Lan | Minh | Tuấn |
|------|------|-----|------|------|
| T4-T5 | BANK-107 (CVE) | BANK-103 (NHNN) | BANK-101 (số dư) | Support / PCI-DSS assessment cho QR |
| T6 | BANK-107 review + merge | Pair với Minh trước nghỉ phép | BANK-101 | PCI-DSS assessment |
| T2-T3 (tuần sau) | BANK-105 (Napas) | 🏖 Nghỉ phép | BANK-101 wrap up | BANK-102 (QR) bắt đầu |

### Tuần 2 (30/04 — 07/05)
| Ngày | Hùng | Lan | Minh | Tuấn |
|------|------|-----|------|------|
| T4-T5 | BANK-105 (Napas) | Quay lại — BANK-103 tiếp | Hỗ trợ kiểm toán | BANK-102 (QR) |
| T6-T2 | BANK-105 load test | BANK-103 review + export | Hỗ trợ kiểm toán | BANK-102 (QR) |
| T3 | Buffer / stretch | Buffer / stretch | Buffer / stretch | BANK-102 wrap up |

> **Lưu ý:** 2 dev còn lại (chưa assign cụ thể) hỗ trợ code review, testing, và đầu mối kiểm toán.

---

## Risks

| Mã | Rủi ro | Khả năng | Tác động | Mức | Giảm thiểu |
|----|--------|----------|----------|-----|------------|
| R1 | Kiểm toán nội bộ yêu cầu thêm data/giải trình | Có thể | Trung bình | 🟡 | Buffer 3 pts. 1 người làm đầu mối, không để cả team bị interrupt |
| R2 | CVE JWT patch gây regression auth flow | Ít có thể | Cao | 🟠 | Regression test trước merge. Rollback plan 15 phút |
| R3 | Napas không cung cấp log kịp thời | Có thể | Cao | 🟠 | Request log ngày 1. Fallback: reproduce trên staging với load test |
| R4 | BANK-102 QR phức tạp hơn estimate (8 pts) | Có thể | Trung bình | 🟡 | Defer sang sprint 48 nếu vượt capacity, giữ P0/P1 |
| R5 | Lan (owner BANK-103) nghỉ 3 ngày | Chắc chắn | Trung bình | 🟡 | Pair với Minh trước khi nghỉ. Lan quay lại tuần 2 hoàn thành |

---

## Action Items trước Sprint Start

- [ ] Confirm Data Warehouse sẵn sàng extract data Q2 cho BANK-103
- [ ] Gửi request log Napas gateway ngày 14/04 cho BANK-105
- [ ] Lan pair với Minh về module báo cáo NHNN trước 27/04
- [ ] Tuấn hoàn thành PCI-DSS scope assessment cho BANK-102 trước khi dev
- [ ] Chỉ định 1 người làm đầu mối kiểm toán nội bộ tuần 2
- [ ] Confirm hạn mức 2FA cho QR thanh toán với Product (10 triệu hay 20 triệu?)

---

## Definition of Done (Sprint-level)

- [ ] Tất cả 5 stories merged vào main branch
- [ ] Unit test coverage ≥ 80% cho code mới
- [ ] Không có lỗ hổng critical/high từ security scan
- [ ] BANK-107: dependency scan pass (Snyk/Trivy)
- [ ] BANK-103: báo cáo export PDF/Excel thành công, review bởi chuyên viên tuân thủ
- [ ] BANK-105: load test pass trên staging (timeout < 2% giờ cao điểm)
- [ ] BANK-102: PCI-DSS checklist ký duyệt
- [ ] Sprint retrospective scheduled
