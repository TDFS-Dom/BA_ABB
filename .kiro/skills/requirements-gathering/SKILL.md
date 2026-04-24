---
name: requirements-gathering
description: Thu thập, tài liệu hóa và validate yêu cầu nghiệp vụ cho tính năng ngân hàng Việt Nam
---

# Skill Thu thập Yêu cầu

## Khi nào dùng
Gọi skill này khi viết user stories, acceptance criteria, hoặc tài liệu requirements cho tính năng ngân hàng.

## Quy trình
1. **Thu thập**: Đặt câu hỏi làm rõ về nhu cầu nghiệp vụ
2. **Tài liệu hóa**: Viết user stories theo format chuẩn (Với vai trò... Tôi muốn... Để...)
3. **Acceptance Criteria**: Định nghĩa kịch bản Cho trước/Khi/Thì (tối thiểu 3 mỗi story)
4. **Kiểm tra pháp quy**: Flag requirements liên quan đến lĩnh vực quản lý (thanh toán, dữ liệu cá nhân, xác thực)
5. **Phụ thuộc**: Xác định dependencies cross-service và hệ thống bên ngoài
6. **Validate**: Chạy requirements-validator subagent để kiểm tra tính đầy đủ

## Template User Story
```markdown
### [JIRA-ID] Tiêu đề Story

**Với vai trò** [vai trò ngân hàng / loại khách hàng],
**Tôi muốn** [hành động],
**Để** [giá trị nghiệp vụ].

#### Acceptance Criteria
- **Cho trước** [điều kiện], **Khi** [hành động], **Thì** [kết quả mong đợi]
- **Cho trước** [điều kiện], **Khi** [hành động], **Thì** [kết quả mong đợi]
- **Cho trước** [điều kiện], **Khi** [hành động], **Thì** [kết quả mong đợi]

#### Yêu cầu phi chức năng
- Hiệu năng: [mục tiêu latency/throughput]
- Bảo mật: [yêu cầu xác thực/phân quyền]
- Tuân thủ: [tham chiếu quy định]

#### Phụ thuộc
- [Dependencies service/team]

#### Ngoài phạm vi
- [Các items loại trừ rõ ràng]
```

## Ví dụ tham khảo
Các ví dụ user story chuẩn cho ngân hàng Việt Nam: #[[file:.kiro/skills/requirements-gathering/references/story-examples.md]]

## Lưu ý đặc thù ngân hàng Việt Nam
- Tính năng xử lý tiền phải ghi rõ đơn vị tiền tệ (VND — 0 decimal) và quy tắc làm tròn
- Tính năng dữ liệu cá nhân cần đánh giá tác động theo Nghị định 13/2023/NĐ-CP (DPIA)
- Tính năng thanh toán thẻ cần đánh giá phạm vi PCI-DSS
- Thay đổi xác thực cần review theo Thông tư 35/2016/TT-NHNN (2FA bắt buộc cho giao dịch tài chính)
- Chuyển tiền liên ngân hàng qua Napas: cần tuân thủ quy định Napas
- eKYC: tuân thủ Thông tư 16/2020/TT-NHNN về định danh khách hàng điện tử
- Số tài khoản theo chuẩn ngân hàng Việt Nam (thường 10-19 chữ số)
