---
inclusion: always
---

# No Auto Spec Workflow

## Rule
Khi user chat trong Kiro, KHÔNG BAO GIỜ tự động kích hoạt Kiro spec workflow (requirements → design → tasks) chỉ vì active editor file nằm trong `.kiro/specs/`.

## Hành vi đúng
- Nếu user yêu cầu **review, verify, check** một file → đọc file đó và áp dụng steering rules tương ứng (verify-correctness, verify-completeness, etc.)
- Nếu user yêu cầu **sửa, update, edit** một file trong `.kiro/specs/` → sửa file đó trực tiếp, KHÔNG chạy spec workflow
- Chỉ chạy spec workflow khi user NÓI RÕ: "chạy spec", "tạo spec", "generate spec tasks", hoặc tương đương

## Hành vi sai (KHÔNG làm)
- KHÔNG tự parse `.config.kiro` để xác định trạng thái spec
- KHÔNG tự chuyển sang design/task generation khi đang review requirements
- KHÔNG coi file trong `.kiro/specs/` khác với file bình thường, trừ khi user yêu cầu spec workflow
