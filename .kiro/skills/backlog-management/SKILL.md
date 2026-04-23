---
name: backlog-management
description: Phân loại Jira issues, refine user stories với acceptance criteria, quản lý grooming sessions cho ngân hàng Việt Nam
---

# Skill Quản lý Backlog

## Khi nào dùng
Gọi skill này cho backlog refinement, viết story, phân loại issue, hoặc grooming sessions. KHÔNG dùng cho sprint planning — dùng skill sprint-planning.

## Framework ưu tiên (Đặc thù ngân hàng Việt Nam)
1. **Deadline NHNN** (P0): Phát hiện kiểm toán, yêu cầu báo cáo NHNN, remediation
2. **Lỗ hổng bảo mật** (P0-P1): CVE Critical/High, phát hiện từ pentest
3. **Bug chặn khách hàng** (P1): Sự cố production ảnh hưởng giao dịch
4. **Tính năng doanh thu** (P1-P2): Sản phẩm mới, thu hút khách hàng
5. **Nợ kỹ thuật** (P2-P3): Hiệu năng, bảo trì, test coverage

## Lập kế hoạch năng lực Sprint
- Velocity team: theo dõi trung bình 3 sprint gần nhất
- Buffer: dự trữ 20% năng lực cho hỗ trợ production và công việc phát sinh
- Carry-over: nếu > 30% stories carry over, giảm cam kết sprint tiếp

## Jira Workflow
```
Backlog → Ready → In Progress → In Review → QA → Done
```

## Definition of Done
- [ ] Code reviewed và approved
- [ ] Unit tests passing (≥ 80% coverage)
- [ ] Integration tests passing
- [ ] Không có lỗ hổng critical/high từ security scan
- [ ] Tài liệu API cập nhật
- [ ] Checklist tuân thủ hoàn thành (nếu applicable)
- [ ] Deploy lên staging và verify

## Lưu ý lịch Việt Nam
- Tết Nguyên đán: code freeze 2 tuần trước Tết, không deploy
- Ngày lễ dài (30/4-1/5, 2/9): không deploy trong kỳ nghỉ
- Cuối quý: hạn chế deploy tuần cuối quý tài chính
- Báo cáo NHNN: deadline hàng quý — ưu tiên tuyệt đối
