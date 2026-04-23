# Business Analyst — Kiro Configuration (Ngân hàng Việt Nam)

Bộ config Kiro tách riêng cho role **Business Analyst** trong nền tảng ngân hàng Việt Nam.

## Bao gồm

| Element | Files | Mục đích |
|---------|-------|----------|
| Steering (3) | product.md, backlog-standards.md, compliance-sbv.md | Bối cảnh nghiệp vụ + backlog + tuân thủ NHNN |
| Agents (3) | requirements-validator, story-prioritizer, risk-assessor | Validate, ưu tiên, đánh giá rủi ro |
| Skills (3) | requirements-gathering, backlog-management, sprint-planning | User stories, grooming, sprint planning |
| Hooks (1) | post-batch-sync | Check TODO/FIXME + uncommitted changes |
| MCP (2) | Jira, Confluence | Issue tracker + wiki |

## Quy định pháp luật áp dụng
- Thông tư 09/2020/TT-NHNN — An toàn hệ thống CNTT ngân hàng
- Thông tư 35/2016/TT-NHNN — Dịch vụ ngân hàng trực tuyến
- Nghị định 13/2023/NĐ-CP — Bảo vệ dữ liệu cá nhân
- Thông tư 41/2018/TT-NHNN — Kiểm soát nội bộ
- PCI-DSS v4.0 — Xử lý thẻ thanh toán

## Cách dùng

```bash
cp -r ba-config/.kiro /path/to/your-project/
cp ba-config/AGENTS.md /path/to/your-project/
chmod +x /path/to/your-project/.kiro/hooks/scripts/sync-issue-status.sh
```

Thêm env vars vào Kiro IDE settings:
- `JIRA_API_TOKEN`, `JIRA_EMAIL`, `JIRA_DOMAIN`
- `CONFLUENCE_API_TOKEN`, `CONFLUENCE_EMAIL`, `CONFLUENCE_DOMAIN` (disabled by default)

## Slash Commands

| Command | Khi nào dùng |
|---------|-------------|
| `/requirements-gathering` | Viết user stories + acceptance criteria |
| `/backlog-management` | Grooming, phân loại issues, refine stories |
| `/sprint-planning` | Planning session, theo dõi velocity |

## Subagents

| Agent | Cách gọi |
|-------|---------|
| requirements-validator | "Validate requirements trong [file]" |
| story-prioritizer | "Ưu tiên các backlog items này" |
| risk-assessor | "Đánh giá rủi ro cho [feature/sprint]" |
