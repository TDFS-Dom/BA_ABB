---
inclusion: auto
name: backlog-standards
description: Quản lý backlog, viết user story, phân loại issue cho ngân hàng Việt Nam
---
## Tiêu chuẩn Backlog & Sprint — Ngân hàng Việt Nam

### Format User Story
```
Với vai trò [vai trò ngân hàng / loại khách hàng],
Tôi muốn [hành động],
Để [giá trị nghiệp vụ].
```

### Format Acceptance Criteria (Cho trước / Khi / Thì)
```
Cho trước [điều kiện tiên quyết],
Khi [hành động],
Thì [kết quả mong đợi].
```

### Phân loại ưu tiên
| Ưu tiên | Nhãn | SLA | Ví dụ |
|---------|------|-----|-------|
| P0 | Khẩn cấp | Trong ngày | Lỗ hổng bảo mật, rò rỉ dữ liệu, sự cố production |
| P1 | Cao | Trong sprint | Deadline NHNN, lỗi chặn khách hàng giao dịch |
| P2 | Bình thường | 2 sprint tới | Tính năng mới, tech debt |
| P3 | Thấp | Backlog | Nice-to-have, cải thiện giao diện |

### Thang Story Point (Fibonacci)
- 1: Thay đổi nhỏ, cập nhật config
- 2: Tính năng đơn giản, thay đổi 1 file
- 3: Tính năng tiêu chuẩn, 2-3 files
- 5: Tính năng phức tạp, nhiều components
- 8: Tính năng lớn, ảnh hưởng cross-service
- 13: Cấp Epic, cần tách nhỏ

### Sự kiện Sprint
- Sprint Planning: Thứ Hai 09:00 (ICT/GMT+7) — 2 giờ
- Daily Scrum: 09:00 (ICT/GMT+7) — 15 phút
- Sprint Review: Thứ Sáu 14:00 (ICT/GMT+7) — 1 giờ
- Retrospective: Thứ Sáu 15:30 (ICT/GMT+7) — 1 giờ
- Backlog Refinement: Thứ Tư 14:00 (ICT/GMT+7) — 1 giờ

### Definition of Ready
- [ ] User story đúng format
- [ ] Acceptance criteria đã định nghĩa (≥ 3)
- [ ] Story đã được team estimate point
- [ ] Dependencies đã xác định
- [ ] UX mockups đính kèm (nếu thay đổi UI)
- [ ] API contract đã thống nhất (nếu thay đổi API)
- [ ] Đánh giá tác động tuân thủ (nếu liên quan dữ liệu cá nhân/thanh toán)
