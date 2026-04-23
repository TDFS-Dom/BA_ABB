---
inclusion: manual
---

# Write SRS — Steering
## Capability: Viết Software Requirements Specification

## Khi nào áp dụng
Khi user yêu cầu viết SRS, tạo requirements document, hoặc chuyển từ analysis/research sang spec chi tiết.

## Input cần có
1. **Analysis file** hoặc **research document** — chứa domain knowledge, business context
2. **Scope** — user chỉ định viết SRS cho phần nào (toàn bộ hệ thống hay 1 module cụ thể)
3. **Audience** — dev team, vendor, hay cả hai

Nếu thiếu input → hỏi user trước khi viết.

## Cấu trúc SRS (IEEE 830 adapted)

```markdown
# SRS: {Tên Hệ Thống/Module}

## 1. Giới Thiệu
### 1.1 Mục đích
### 1.2 Phạm vi
### 1.3 Định nghĩa & Viết tắt
### 1.4 Tài liệu tham khảo

## 2. Mô Tả Tổng Quan
### 2.1 Bối cảnh sản phẩm (product perspective)
### 2.2 Chức năng chính (product functions) — high-level list
### 2.3 Đặc điểm người dùng (user characteristics)
### 2.4 Ràng buộc (constraints)
### 2.5 Giả định & Phụ thuộc (assumptions & dependencies)

## 3. Yêu Cầu Chức Năng (Functional Requirements)
### FR-{module}-{number}: {Tên requirement}
- **Mô tả**: ...
- **Input**: ...
- **Processing**: ...
- **Output**: ...
- **Business Rules**: ...
- **Acceptance Criteria**: (Given/When/Then hoặc checklist)

## 4. Yêu Cầu Phi Chức Năng (Non-Functional Requirements)
### 4.1 Performance
### 4.2 Security
### 4.3 Availability & Reliability
### 4.4 Compliance & Regulatory
### 4.5 Usability & Accessibility
### 4.6 Data Retention & Privacy

## 5. Giao Diện Hệ Thống (External Interfaces)
### 5.1 User Interfaces
### 5.2 Hardware Interfaces
### 5.3 Software Interfaces (API, SDK, 3rd-party)
### 5.4 Communication Interfaces

## 6. Luồng Nghiệp Vụ (Business Flows)
### 6.1 Happy Path
### 6.2 Alternative Flows
### 6.3 Exception Flows

## 7. Data Requirements
### 7.1 Data Model (entities, relationships)
### 7.2 Data Dictionary
### 7.3 Data Migration (nếu có)

## 8. Appendix
### 8.1 Traceability Matrix (requirement → user story → test case)
### 8.2 Open Questions
### 8.3 Change Log
```

## Quy tắc viết

### Requirement ID Convention
```
FR-{MODULE}-{NNN}    # Functional: FR-EKYC-001, FR-AUTH-012
NFR-{CATEGORY}-{NNN} # Non-functional: NFR-PERF-001, NFR-SEC-003
BR-{MODULE}-{NNN}    # Business Rule: BR-EKYC-001
```

### Mỗi Functional Requirement PHẢI có:
1. **Mô tả** — 1-2 câu, rõ ràng, không mơ hồ
2. **Actor** — ai thực hiện (user, system, admin, 3rd-party)
3. **Precondition** — điều kiện trước khi thực hiện
4. **Input** — data đầu vào, format, validation rules
5. **Processing** — logic xử lý, step-by-step
6. **Output** — kết quả mong đợi
7. **Business Rules** — quy tắc nghiệp vụ áp dụng
8. **Acceptance Criteria** — Given/When/Then hoặc checklist cụ thể
9. **Error Handling** — các lỗi có thể xảy ra và cách xử lý

### Ngôn ngữ
- Dùng "Hệ thống PHẢI" (SHALL) cho mandatory requirements
- Dùng "Hệ thống NÊN" (SHOULD) cho recommended
- Dùng "Hệ thống CÓ THỂ" (MAY) cho optional
- KHÔNG dùng ngôn ngữ mơ hồ: "nhanh", "thân thiện", "bảo mật cao" → thay bằng số liệu cụ thể
- Mỗi requirement phải testable — QA đọc xong biết test thế nào

### Acceptance Criteria format
```
**AC1**: Given [precondition], When [action], Then [expected result]
**AC2**: Given [precondition], When [action], Then [expected result]
```

Hoặc checklist:
```
- [ ] Điều kiện 1 được đáp ứng
- [ ] Điều kiện 2 được đáp ứng
- [ ] Error case X hiển thị message Y
```

### Số liệu & Thresholds
- Lấy từ analysis file nếu có (ví dụ: FAR <0.1%, FRR <5%)
- Lấy từ regulations nếu applicable (ví dụ: ngưỡng 10 triệu VND theo QĐ 2345)
- Nếu không có source → ghi "[TBD — cần confirm với stakeholder]" thay vì bịa số

### Cross-reference
- Khi requirement liên quan đến regulation → cite: "(theo QĐ 2345/QĐ-NHNN)"
- Khi requirement liên quan đến analysis → cite: "(xem Section X trong analysis)"
- Khi requirement phụ thuộc requirement khác → link: "(xem FR-EKYC-003)"

## Workflow viết SRS

1. **Đọc analysis/research file** → extract danh sách features, flows, NFRs
2. **Hỏi user về scope** nếu chưa rõ (toàn bộ hay 1 module?)
3. **Tạo skeleton** — outline tất cả sections, list requirement IDs
4. **Viết từng section** — bắt đầu từ Section 3 (Functional Requirements) vì đây là phần quan trọng nhất
5. **Cross-check** — mỗi flow trong analysis phải có ít nhất 1 FR tương ứng
6. **Đánh dấu gaps** — nếu analysis thiếu thông tin cho requirement → ghi [TBD]
7. **Output** — 1 file markdown hoàn chỉnh

## Anti-patterns (KHÔNG làm)

- KHÔNG copy nguyên văn từ analysis vào SRS — phải chuyển thành requirement format
- KHÔNG viết requirement quá dài (>10 dòng) — tách thành nhiều requirements nhỏ
- KHÔNG mix functional và non-functional trong cùng 1 requirement
- KHÔNG viết AC mơ hồ kiểu "hệ thống hoạt động đúng" — phải có expected result cụ thể
- KHÔNG bỏ qua error handling — mỗi happy path phải có ít nhất 1 error case
- KHÔNG tự bịa business rules — nếu không có trong analysis, ghi [TBD]
