# eKYC Module — Requirements Analysis

## 1. Tổng quan

### 1.1 Mục tiêu
Xây dựng module eKYC (Electronic Know Your Customer) cho nền tảng ngân hàng, cho phép định danh khách hàng điện tử qua kênh Mobile Banking và Internet Banking, tuân thủ Thông tư 16/2020/TT-NHNN và các quy định liên quan.

### 1.2 Phạm vi
| Hạng mục | Phạm vi |
|----------|---------|
| Use cases | Onboarding khách hàng mới, nâng cấp xác thực, re-KYC định kỳ |
| Kênh | Mobile Banking, Internet Banking (Web) |
| Xác minh | OCR CCCD, NFC chip CCCD, Face matching, Video call (fallback), Xác minh CSDL quốc gia |
| Vendor | Tích hợp vendor bên thứ 3 (VNPT eKYC / FPT.AI / tương đương) |
| Tự động hóa | Auto-approve score ≥ threshold, manual review score thấp |
| Tích hợp | Core Banking (CIF), AML/CFT screening, Document Management System |

### 1.3 Personas
| Persona | Mô tả |
|---------|-------|
| Khách hàng cá nhân (KH) | Người dùng Mobile/Web Banking muốn mở tài khoản hoặc nâng cấp xác thực |
| Nhân viên KYC (NV-KYC) | Nhân viên back-office review hồ sơ eKYC chưa đạt auto-approve |
| Quản lý KYC (QL-KYC) | Quản lý phê duyệt hồ sơ escalated, cấu hình threshold |
| Nhân viên Video Call (NV-VC) | Nhân viên thực hiện video call xác minh khi fallback |
| Admin hệ thống | Cấu hình vendor, threshold, quy tắc re-KYC |

### 1.4 Tuân thủ pháp quy
| Quy định | Yêu cầu liên quan |
|----------|-------------------|
| Thông tư 16/2020/TT-NHNN | Quy định eKYC cho ngân hàng — phương thức xác minh, lưu trữ hồ sơ |
| Nghị định 13/2023/NĐ-CP | Bảo vệ dữ liệu cá nhân — consent, DPIA, quyền chủ thể dữ liệu |
| Thông tư 09/2020/TT-NHNN | An toàn CNTT — mã hóa, audit log, kiểm soát truy cập |
| Thông tư 35/2016/TT-NHNN | 2FA cho giao dịch tài chính, session timeout |
| Luật Phòng chống rửa tiền 2022 | AML/CFT screening bắt buộc trước khi mở tài khoản |

---

## 2. User Stories

### Epic 1: Onboarding khách hàng mới

#### US-001: Thu thập thông tin CCCD bằng OCR
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** chụp ảnh CCCD và hệ thống tự động nhận dạng thông tin,
**Để** không phải nhập tay thông tin cá nhân, tiết kiệm thời gian mở tài khoản.

**Acceptance Criteria:**
- **Cho trước** KH đang ở bước xác minh giấy tờ, **Khi** KH chụp mặt trước CCCD, **Thì** hệ thống OCR trích xuất: họ tên, số CCCD, ngày sinh, giới tính, quê quán, nơi thường trú, ngày cấp, ngày hết hạn — hiển thị để KH xác nhận
- **Cho trước** KH đã chụp mặt trước, **Khi** KH chụp mặt sau CCCD, **Thì** hệ thống trích xuất: đặc điểm nhận dạng, ngày cấp — và cross-check với mặt trước
- **Cho trước** ảnh CCCD bị mờ/lóa/cắt góc, **Khi** OCR không đạt confidence threshold (< 80%), **Thì** hệ thống yêu cầu KH chụp lại với hướng dẫn cụ thể (ánh sáng, góc chụp)
- **Cho trước** CCCD đã hết hạn, **Khi** hệ thống kiểm tra ngày hết hạn, **Thì** từ chối và thông báo KH cần dùng CCCD còn hiệu lực

**Priority**: P0
**Estimate**: L
**Dependencies**: Vendor eKYC SDK (OCR module)
**Screen**: Chụp CCCD (mặt trước/sau)

---

#### US-002: Đọc chip NFC trên CCCD gắn chip
**Với vai trò** khách hàng cá nhân sử dụng Mobile Banking,
**Tôi muốn** dùng NFC trên điện thoại để đọc chip CCCD,
**Để** xác minh tính xác thực của CCCD và tăng độ tin cậy hồ sơ.

**Acceptance Criteria:**
- **Cho trước** điện thoại KH có NFC và CCCD gắn chip, **Khi** KH đặt CCCD lên điện thoại, **Thì** hệ thống đọc dữ liệu từ chip: ảnh chân dung, thông tin cá nhân, vân tay (nếu có) — và cross-check với dữ liệu OCR
- **Cho trước** điện thoại không có NFC hoặc CCCD không gắn chip, **Khi** hệ thống detect không hỗ trợ NFC, **Thì** cho phép skip bước NFC và tiếp tục flow với OCR + face matching (giảm mức xác thực)
- **Cho trước** đọc NFC thất bại (timeout, lỗi kết nối), **Khi** thử lại 3 lần không thành công, **Thì** cho phép skip NFC với cảnh báo mức xác thực thấp hơn

**Priority**: P0
**Estimate**: L
**Dependencies**: US-001, Vendor eKYC SDK (NFC module), thiết bị hỗ trợ NFC
**Screen**: Đọc NFC CCCD

---

#### US-003: Xác minh khuôn mặt (Face Matching)
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** chụp ảnh selfie để hệ thống so khớp với ảnh trên CCCD,
**Để** chứng minh tôi là chủ sở hữu CCCD.

**Acceptance Criteria:**
- **Cho trước** KH đã hoàn thành OCR CCCD, **Khi** KH chụp selfie, **Thì** hệ thống so khớp khuôn mặt với ảnh trên CCCD (hoặc ảnh từ chip NFC nếu có) — yêu cầu similarity score ≥ threshold cấu hình được
- **Cho trước** KH đang chụp selfie, **Khi** hệ thống detect liveness, **Thì** yêu cầu KH thực hiện hành động ngẫu nhiên (nháy mắt, quay đầu, mỉm cười) để chống giả mạo (anti-spoofing)
- **Cho trước** face matching score < threshold, **Khi** thử lại 3 lần không đạt, **Thì** chuyển sang video call với nhân viên (fallback)
- **Cho trước** phát hiện ảnh in/màn hình/mask, **Khi** liveness check fail, **Thì** từ chối và ghi log cảnh báo fraud

**Priority**: P0
**Estimate**: L
**Dependencies**: US-001, Vendor eKYC SDK (Face matching + Liveness)
**Screen**: Chụp selfie xác minh

---

#### US-004: Xác minh qua Cơ sở dữ liệu quốc gia
**Với vai trò** hệ thống eKYC,
**Tôi muốn** xác minh thông tin CCCD với CSDL dân cư quốc gia (qua gateway VNPT/FPT),
**Để** đảm bảo CCCD là thật và thông tin chính xác.

**Acceptance Criteria:**
- **Cho trước** KH đã hoàn thành OCR + face matching, **Khi** hệ thống gửi request xác minh, **Thì** nhận kết quả: match/mismatch/not-found cho từng trường (họ tên, ngày sinh, số CCCD, ảnh)
- **Cho trước** CSDL quốc gia trả về mismatch, **Khi** có ≥ 1 trường critical không khớp (số CCCD, họ tên), **Thì** từ chối hồ sơ và chuyển manual review
- **Cho trước** gateway CSDL quốc gia timeout/lỗi, **Khi** retry 3 lần thất bại, **Thì** đánh dấu hồ sơ "pending verification" và thông báo KH sẽ xử lý trong 24h

**Priority**: P0
**Estimate**: M
**Dependencies**: US-001, US-003, Gateway CSDL quốc gia (VNPT/FPT)
**Screen**: Màn hình chờ xác minh

---

#### US-005: Video call xác minh (Fallback)
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** thực hiện video call với nhân viên ngân hàng khi xác minh tự động không thành công,
**Để** vẫn có thể hoàn thành eKYC mà không cần đến quầy.

**Acceptance Criteria:**
- **Cho trước** face matching hoặc CSDL verification fail, **Khi** KH chọn video call, **Thì** hệ thống kết nối KH với nhân viên video call khả dụng (queue management)
- **Cho trước** đang video call, **Khi** nhân viên yêu cầu KH giơ CCCD, **Thì** nhân viên có thể capture frame và so sánh trực quan
- **Cho trước** video call hoàn thành, **Khi** nhân viên xác nhận danh tính, **Thì** hệ thống ghi nhận kết quả + lưu recording video call (tối thiểu 5 năm theo TT16)
- **Cho trước** không có nhân viên khả dụng (ngoài giờ), **Khi** KH chọn video call, **Thì** hiển thị khung giờ hẹn và cho phép đặt lịch

**Priority**: P1
**Estimate**: XL
**Dependencies**: US-003, Video call infrastructure, Queue management
**Screen**: Video call, Đặt lịch hẹn

---

#### US-006: Đồng ý điều khoản & Thu thập consent
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** đọc và đồng ý điều khoản sử dụng dịch vụ và chính sách bảo mật,
**Để** hiểu rõ quyền lợi và nghĩa vụ trước khi cung cấp dữ liệu cá nhân.

**Acceptance Criteria:**
- **Cho trước** KH bắt đầu flow eKYC, **Khi** hiển thị màn hình consent, **Thì** hiển thị đầy đủ: mục đích thu thập, loại dữ liệu, thời gian lưu trữ, quyền rút consent — theo Nghị định 13/2023
- **Cho trước** KH chưa tick đồng ý, **Khi** KH nhấn tiếp tục, **Thì** không cho phép tiến hành và highlight checkbox consent
- **Cho trước** KH đã đồng ý, **Khi** hệ thống ghi nhận consent, **Thì** lưu: timestamp, phiên bản điều khoản, IP/device info, nội dung consent — immutable audit log

**Priority**: P0
**Estimate**: S
**Dependencies**: Không
**Screen**: Điều khoản & Consent

---

#### US-007: Tính điểm eKYC và quyết định tự động
**Với vai trò** hệ thống eKYC,
**Tôi muốn** tính tổng điểm xác minh dựa trên kết quả các bước,
**Để** tự động phê duyệt hồ sơ đạt chuẩn hoặc chuyển manual review.

**Acceptance Criteria:**
- **Cho trước** KH hoàn thành tất cả bước xác minh, **Khi** hệ thống tính điểm, **Thì** tổng hợp score từ: OCR confidence, NFC verification, face matching score, CSDL quốc gia result — theo trọng số cấu hình được
- **Cho trước** tổng điểm ≥ auto-approve threshold, **Khi** AML screening pass, **Thì** tự động phê duyệt hồ sơ và tạo CIF trên Core Banking
- **Cho trước** tổng điểm < auto-approve threshold nhưng ≥ review threshold, **Khi** hệ thống phân loại, **Thì** chuyển hồ sơ vào queue manual review với priority dựa trên score
- **Cho trước** tổng điểm < review threshold hoặc AML screening flag, **Khi** hệ thống phân loại, **Thì** từ chối tự động và ghi lý do

**Priority**: P0
**Estimate**: M
**Dependencies**: US-001 → US-004, AML/CFT system
**Screen**: Kết quả eKYC (thành công/chờ review/từ chối)

---

#### US-008: AML/CFT Screening
**Với vai trò** hệ thống eKYC,
**Tôi muốn** kiểm tra khách hàng qua danh sách đen AML/CFT trước khi phê duyệt,
**Để** tuân thủ Luật Phòng chống rửa tiền và bảo vệ ngân hàng khỏi rủi ro.

**Acceptance Criteria:**
- **Cho trước** KH hoàn thành xác minh danh tính, **Khi** hệ thống gửi screening request, **Thì** kiểm tra: danh sách cấm vận (sanctions), PEP (Politically Exposed Persons), danh sách đen nội bộ ngân hàng
- **Cho trước** KH match với danh sách đen, **Khi** confidence ≥ threshold, **Thì** từ chối tự động + alert Compliance team + ghi audit log
- **Cho trước** KH match mờ (fuzzy match), **Khi** confidence < threshold, **Thì** chuyển Compliance team review thủ công

**Priority**: P0
**Estimate**: M
**Dependencies**: US-007, Hệ thống AML/CFT hiện tại
**Screen**: Không có UI riêng (background process)

---

### Epic 2: Nâng cấp xác thực khách hàng hiện tại

#### US-009: Nâng cấp mức xác thực
**Với vai trò** khách hàng hiện tại (đã có tài khoản, KYC cấp thấp),
**Tôi muốn** thực hiện eKYC bổ sung để nâng cấp mức xác thực,
**Để** được sử dụng hạn mức giao dịch cao hơn và nhiều dịch vụ hơn.

**Acceptance Criteria:**
- **Cho trước** KH đã đăng nhập và có mức xác thực cấp 1 (chỉ OTP), **Khi** KH chọn nâng cấp, **Thì** hiển thị các bước eKYC cần bổ sung (skip các bước đã hoàn thành)
- **Cho trước** KH đã có OCR nhưng chưa NFC, **Khi** KH thực hiện NFC + face matching, **Thì** nâng cấp lên mức xác thực cấp 2 và tăng hạn mức giao dịch tương ứng
- **Cho trước** KH hoàn thành nâng cấp, **Khi** hệ thống cập nhật CIF, **Thì** ghi nhận mức xác thực mới + timestamp + phương thức xác minh đã dùng

**Priority**: P1
**Estimate**: M
**Dependencies**: US-001 → US-007, Core Banking CIF
**Screen**: Nâng cấp xác thực

---

### Epic 3: Re-KYC định kỳ

#### US-010: Thông báo re-KYC
**Với vai trò** hệ thống eKYC,
**Tôi muốn** tự động thông báo khách hàng khi đến hạn re-KYC,
**Để** đảm bảo thông tin khách hàng luôn cập nhật theo quy định AML.

**Acceptance Criteria:**
- **Cho trước** KH có hồ sơ KYC sắp hết hạn (trước 30 ngày), **Khi** batch job chạy hàng ngày, **Thì** gửi push notification + in-app notification nhắc KH cập nhật
- **Cho trước** KH không cập nhật sau 30 ngày thông báo, **Khi** hồ sơ KYC hết hạn, **Thì** hạn chế giao dịch (chỉ cho phép rút tiền, không cho chuyển khoản) + gửi thông báo khẩn
- **Cho trước** KH thuộc nhóm rủi ro cao, **Khi** đến chu kỳ re-KYC (1 năm), **Thì** yêu cầu re-KYC đầy đủ (OCR + NFC + face matching)
- **Cho trước** KH thuộc nhóm rủi ro thấp, **Khi** đến chu kỳ re-KYC (3 năm), **Thì** chỉ yêu cầu xác nhận thông tin + selfie

**Priority**: P1
**Estimate**: M
**Dependencies**: US-007, Core Banking CIF, Notification service
**Screen**: Thông báo re-KYC, Flow re-KYC

---

#### US-011: Thực hiện re-KYC
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** cập nhật thông tin KYC khi được yêu cầu,
**Để** duy trì tài khoản hoạt động bình thường.

**Acceptance Criteria:**
- **Cho trước** KH nhận thông báo re-KYC, **Khi** KH mở flow re-KYC, **Thì** hiển thị thông tin hiện tại và các bước cần cập nhật (dựa trên mức rủi ro)
- **Cho trước** KH đã thay đổi CCCD (cấp mới), **Khi** KH chụp CCCD mới, **Thì** hệ thống so sánh với dữ liệu cũ + xác minh CSDL quốc gia + cập nhật CIF
- **Cho trước** re-KYC hoàn thành, **Khi** hệ thống phê duyệt, **Thì** reset chu kỳ re-KYC + gỡ hạn chế giao dịch (nếu có) + ghi audit log

**Priority**: P1
**Estimate**: M
**Dependencies**: US-010, US-001 → US-007
**Screen**: Flow re-KYC (reuse screens từ Epic 1)

---

### Epic 4: Back-office — Quản lý hồ sơ eKYC

#### US-012: Review hồ sơ eKYC (Manual Review)
**Với vai trò** nhân viên KYC (NV-KYC),
**Tôi muốn** xem và đánh giá hồ sơ eKYC chưa đạt auto-approve,
**Để** quyết định phê duyệt hoặc từ chối dựa trên đánh giá chuyên môn.

**Acceptance Criteria:**
- **Cho trước** NV-KYC đăng nhập back-office, **Khi** mở queue review, **Thì** hiển thị danh sách hồ sơ pending sắp xếp theo priority (score thấp nhất trước) với SLA countdown
- **Cho trước** NV-KYC mở 1 hồ sơ, **Khi** xem chi tiết, **Thì** hiển thị: ảnh CCCD (trước/sau), ảnh selfie, kết quả OCR, face matching score, kết quả CSDL quốc gia, AML screening result — side-by-side comparison
- **Cho trước** NV-KYC quyết định phê duyệt, **Khi** nhấn Approve + ghi lý do, **Thì** hệ thống tạo CIF + thông báo KH + ghi audit log (ai duyệt, lúc nào, lý do)
- **Cho trước** NV-KYC quyết định từ chối, **Khi** nhấn Reject + chọn lý do từ chối, **Thì** thông báo KH kèm lý do + hướng dẫn khắc phục + ghi audit log

**Priority**: P0
**Estimate**: L
**Dependencies**: US-007, US-008
**Screen**: Back-office: Queue review, Chi tiết hồ sơ

---

#### US-013: Escalate hồ sơ phức tạp
**Với vai trò** nhân viên KYC,
**Tôi muốn** escalate hồ sơ phức tạp lên quản lý,
**Để** có quyết định từ cấp có thẩm quyền cao hơn.

**Acceptance Criteria:**
- **Cho trước** NV-KYC không chắc chắn về hồ sơ, **Khi** nhấn Escalate + ghi lý do, **Thì** chuyển hồ sơ sang queue của QL-KYC với context đầy đủ
- **Cho trước** QL-KYC nhận hồ sơ escalated, **Khi** quyết định, **Thì** ghi nhận quyết định + lý do + gửi feedback cho NV-KYC
- **Cho trước** hồ sơ escalated quá SLA (4 giờ), **Khi** chưa có quyết định, **Thì** alert QL-KYC và ghi log SLA breach

**Priority**: P1
**Estimate**: M
**Dependencies**: US-012
**Screen**: Back-office: Escalation queue

---

#### US-014: Dashboard & Báo cáo eKYC
**Với vai trò** quản lý KYC,
**Tôi muốn** xem dashboard tổng quan về hoạt động eKYC,
**Để** giám sát hiệu suất, phát hiện bất thường, và báo cáo lãnh đạo.

**Acceptance Criteria:**
- **Cho trước** QL-KYC mở dashboard, **Khi** trang load, **Thì** hiển thị: tổng hồ sơ hôm nay, tỷ lệ auto-approve, tỷ lệ reject, hồ sơ pending, SLA compliance rate
- **Cho trước** QL-KYC chọn khoảng thời gian, **Khi** filter, **Thì** hiển thị trend chart: số lượng hồ sơ, tỷ lệ thành công, thời gian xử lý trung bình
- **Cho trước** QL-KYC cần báo cáo, **Khi** nhấn Export, **Thì** xuất báo cáo Excel/PDF với dữ liệu chi tiết (không chứa ảnh CCCD/selfie — chỉ metadata)

**Priority**: P2
**Estimate**: L
**Dependencies**: US-007, US-012
**Screen**: Back-office: Dashboard eKYC

---

### Epic 5: Cấu hình & Quản trị

#### US-015: Cấu hình threshold & scoring
**Với vai trò** Admin hệ thống,
**Tôi muốn** cấu hình các threshold cho scoring eKYC,
**Để** điều chỉnh mức độ tự động hóa phù hợp với chính sách ngân hàng.

**Acceptance Criteria:**
- **Cho trước** Admin mở trang cấu hình, **Khi** thay đổi threshold, **Thì** cho phép cấu hình: auto-approve threshold, review threshold, face matching threshold, OCR confidence threshold — với validation min/max
- **Cho trước** Admin thay đổi threshold, **Khi** lưu, **Thì** yêu cầu xác nhận 2 lần (maker-checker) + ghi audit log thay đổi + effective date
- **Cho trước** threshold mới có hiệu lực, **Khi** hồ sơ mới vào, **Thì** áp dụng threshold mới (không retroactive cho hồ sơ đã xử lý)

**Priority**: P1
**Estimate**: M
**Dependencies**: US-007
**Screen**: Back-office: Cấu hình eKYC

---

#### US-016: Cấu hình chu kỳ re-KYC
**Với vai trò** Admin hệ thống,
**Tôi muốn** cấu hình chu kỳ re-KYC theo nhóm rủi ro,
**Để** tuân thủ quy định AML và tối ưu nguồn lực.

**Acceptance Criteria:**
- **Cho trước** Admin mở cấu hình re-KYC, **Khi** thiết lập, **Thì** cho phép cấu hình: chu kỳ theo nhóm rủi ro (cao: 1 năm, trung bình: 2 năm, thấp: 3 năm), mức độ re-KYC (đầy đủ/rút gọn)
- **Cho trước** thay đổi chu kỳ, **Khi** lưu, **Thì** hệ thống tính lại ngày re-KYC cho tất cả KH thuộc nhóm bị ảnh hưởng
- **Cho trước** cấu hình mới, **Khi** batch job chạy, **Thì** gửi thông báo cho KH sắp đến hạn theo chu kỳ mới

**Priority**: P2
**Estimate**: S
**Dependencies**: US-010, US-015
**Screen**: Back-office: Cấu hình re-KYC

---

## 3. Dependency Map

```
US-006 (Consent) ─────────────────────────────────────────────────┐
                                                                   │
US-001 (OCR CCCD) ← US-002 (NFC Chip)                            │
       │                    │                                      │
       └────────┬───────────┘                                      │
                ▼                                                  │
         US-003 (Face Matching) ──fallback──→ US-005 (Video Call) │
                │                                                  │
                ▼                                                  │
         US-004 (CSDL Quốc gia)                                   │
                │                                                  │
                ▼                                                  │
         US-008 (AML Screening)                                    │
                │                                                  │
                ▼                                                  │
         US-007 (Scoring & Decision) ◄─────────────────────────────┘
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
   Auto-OK   Review   Reject
              │
              ▼
     US-012 (Manual Review) ──escalate──→ US-013 (Escalation)
              │
              ▼
     US-014 (Dashboard)

── Parallel tracks ──
US-009 (Nâng cấp xác thực) ← reuse US-001→US-007
US-010 (Thông báo re-KYC) → US-011 (Thực hiện re-KYC) ← reuse US-001→US-007
US-015 (Cấu hình threshold) ← standalone
US-016 (Cấu hình re-KYC) ← US-015
```

### Critical Path
```
US-006 → US-001 → US-003 → US-004 → US-008 → US-007 → US-012
```
Đây là flow chính từ consent → xác minh → scoring → review. Tất cả stories khác phụ thuộc hoặc chạy song song.

### Parallel Tracks
| Track | Stories | Có thể bắt đầu khi |
|-------|---------|---------------------|
| Core eKYC flow | US-001 → US-004, US-007, US-008 | Ngay lập tức |
| Consent & Legal | US-006 | Ngay lập tức (song song) |
| Video call fallback | US-005 | Sau US-003 |
| Back-office review | US-012, US-013 | Sau US-007 |
| Nâng cấp xác thực | US-009 | Sau Epic 1 hoàn thành |
| Re-KYC | US-010, US-011 | Sau Epic 1 hoàn thành |
| Dashboard & Config | US-014, US-015, US-016 | Sau US-007 |

---

## 4. Technical Constraints

| Constraint | Impact | Mitigation |
|-----------|--------|-----------|
| Vendor eKYC SDK (OCR, NFC, Face) | Phụ thuộc SDK bên thứ 3, latency mạng | Abstraction layer (adapter pattern), timeout config, retry policy, fallback flow |
| Gateway CSDL quốc gia | Availability không kiểm soát được, rate limit | Circuit breaker, async verification, queue pending hồ sơ |
| NFC chỉ hỗ trợ mobile | Web Banking không có NFC | Graceful degradation: Web chỉ dùng OCR + Face, mức xác thực thấp hơn |
| Video call infrastructure | Cần WebRTC/SRTP, recording storage lớn | Dùng managed service (Twilio/Vonage), S3 cho recording, lifecycle policy xóa sau 5 năm |
| Dữ liệu cá nhân nhạy cảm | DPIA bắt buộc, mã hóa, data residency VN | AES-256 at rest, TLS 1.3 in transit, HSM cho key management, storage tại VN |
| Ảnh CCCD + Selfie storage | Dung lượng lớn, lưu trữ lâu dài | Object storage (S3), tiered storage (hot → cold sau 1 năm), encryption |
| Liveness detection | Anti-spoofing phức tạp | Vendor SDK (passive + active liveness), không tự build |
| AML/CFT screening | Real-time check, danh sách cập nhật liên tục | Tích hợp hệ thống AML hiện tại qua API, batch update danh sách hàng ngày |
| Audit log immutable | 10 năm lưu trữ, không được sửa/xóa | Append-only log (DynamoDB/Elasticsearch), separate audit DB, backup cross-region |
| Concurrent eKYC requests | Peak hours có thể spike | Auto-scaling, queue-based processing cho scoring, rate limiting per user |

---

## 5. Screen ↔ Story Mapping

| # | Screen | User Stories | Components | Complexity | Kênh |
|---|--------|-------------|------------|-----------|------|
| 1 | Điều khoản & Consent | US-006 | Checkbox, ScrollView, Button | S | Mobile, Web |
| 2 | Chụp CCCD mặt trước | US-001 | Camera, Overlay guide, OCR result | L | Mobile, Web |
| 3 | Chụp CCCD mặt sau | US-001 | Camera, Overlay guide, OCR result | L | Mobile, Web |
| 4 | Xác nhận thông tin OCR | US-001 | Form (pre-filled), Edit fields | M | Mobile, Web |
| 5 | Đọc NFC CCCD | US-002 | NFC animation, Progress, Instructions | L | Mobile only |
| 6 | Chụp Selfie + Liveness | US-003 | Camera (front), Liveness guide, Animation | L | Mobile, Web |
| 7 | Chờ xác minh | US-004, US-007 | Loading, Progress steps | S | Mobile, Web |
| 8 | Kết quả: Thành công | US-007 | Success state, CTA | S | Mobile, Web |
| 9 | Kết quả: Chờ review | US-007 | Pending state, Timeline | S | Mobile, Web |
| 10 | Kết quả: Từ chối | US-007 | Error state, Lý do, CTA retry | S | Mobile, Web |
| 11 | Video call | US-005 | WebRTC video, Controls, Capture | XL | Mobile, Web |
| 12 | Đặt lịch video call | US-005 | Calendar, Time slots, Confirm | M | Mobile, Web |
| 13 | Nâng cấp xác thực | US-009 | Progress steps, Reuse screens 2-7 | M | Mobile, Web |
| 14 | Thông báo re-KYC | US-010 | Notification card, CTA | S | Mobile, Web |
| 15 | BO: Queue review | US-012 | Table, Filters, SLA badge | L | Web (back-office) |
| 16 | BO: Chi tiết hồ sơ | US-012 | Side-by-side comparison, Actions | L | Web (back-office) |
| 17 | BO: Escalation queue | US-013 | Table, Priority tags | M | Web (back-office) |
| 18 | BO: Dashboard eKYC | US-014 | Charts, KPIs, Date filter, Export | L | Web (back-office) |
| 19 | BO: Cấu hình threshold | US-015 | Form, Sliders, Maker-checker | M | Web (back-office) |
| 20 | BO: Cấu hình re-KYC | US-016 | Form, Risk group table | S | Web (back-office) |

---

## 6. Yêu cầu phi chức năng (NFR)

### Performance
| Metric | Target | Ghi chú |
|--------|--------|---------|
| OCR processing time | ≤ 3 giây | Bao gồm upload + OCR + response |
| Face matching time | ≤ 5 giây | Bao gồm liveness check |
| End-to-end eKYC flow | ≤ 5 phút | Từ consent đến kết quả (happy path, không video call) |
| CSDL quốc gia verification | ≤ 10 giây | Timeout 30 giây, retry 3 lần |
| Manual review SLA | ≤ 4 giờ | Từ lúc vào queue đến quyết định |
| Concurrent users | ≥ 500 | eKYC sessions đồng thời |

### Security
| Yêu cầu | Chi tiết |
|----------|---------|
| Mã hóa at rest | AES-256 cho ảnh CCCD, selfie, video recording |
| Mã hóa in transit | TLS 1.3 bắt buộc |
| Key management | HSM hoặc KMS, key rotation 90 ngày |
| Data residency | Tất cả dữ liệu KH lưu tại Việt Nam |
| Access control | RBAC, principle of least privilege |
| Anti-fraud | Liveness detection, device fingerprinting, velocity check (max 3 attempts/ngày/device) |
| Session | Timeout 5 phút (mobile), 10 phút (web) theo TT35 |

### Data Retention
| Loại dữ liệu | Thời gian lưu | Cơ sở pháp lý |
|--------------|--------------|---------------|
| Ảnh CCCD, selfie | 10 năm | TT16/2020, Luật Kế toán 2015 |
| Video call recording | 5 năm | TT16/2020 |
| Audit log | 10 năm | Luật Kế toán 2015, TT09/2020 |
| Consent records | Suốt thời gian quan hệ KH + 5 năm | NĐ 13/2023 |
| Scoring data | 10 năm | Quy định nội bộ |

### DPIA (Data Protection Impact Assessment)
Theo Nghị định 13/2023/NĐ-CP, module eKYC BẮT BUỘC thực hiện DPIA vì:
- Thu thập dữ liệu sinh trắc học (ảnh khuôn mặt, vân tay từ NFC)
- Xử lý dữ liệu cá nhân nhạy cảm (CCCD, thông tin tài chính)
- Xử lý quy mô lớn (toàn bộ khách hàng mới)
- Sử dụng công nghệ mới (AI face matching, NFC)

---

## 7. Thứ tự triển khai đề xuất

### Phase 1 — Core eKYC (MVP) — ~8 tuần
| Priority | Stories | Mô tả |
|----------|---------|-------|
| Sprint 1-2 | US-006, US-001 | Consent + OCR CCCD |
| Sprint 3-4 | US-003, US-004 | Face matching + CSDL quốc gia |
| Sprint 5-6 | US-007, US-008 | Scoring + AML screening |
| Sprint 7-8 | US-012 | Back-office manual review |

### Phase 2 — Enhanced Verification — ~4 tuần
| Priority | Stories | Mô tả |
|----------|---------|-------|
| Sprint 9-10 | US-002 | NFC chip reading (mobile) |
| Sprint 11-12 | US-005 | Video call fallback |

### Phase 3 — Lifecycle Management — ~4 tuần
| Priority | Stories | Mô tả |
|----------|---------|-------|
| Sprint 13-14 | US-009 | Nâng cấp xác thực |
| Sprint 15-16 | US-010, US-011 | Re-KYC flow |

### Phase 4 — Operations & Config — ~3 tuần
| Priority | Stories | Mô tả |
|----------|---------|-------|
| Sprint 17 | US-013, US-015 | Escalation + Config threshold |
| Sprint 18 | US-014, US-016 | Dashboard + Config re-KYC |

### Tổng ước lượng: ~19 tuần (~5 tháng)

---

## 8. Rủi ro & Giảm thiểu

| # | Rủi ro | Xác suất | Tác động | Giảm thiểu |
|---|--------|----------|----------|-----------|
| R1 | Vendor eKYC SDK chất lượng OCR/face matching thấp | Trung bình | Cao | POC với 2-3 vendor trước khi chọn, benchmark với dataset VN |
| R2 | Gateway CSDL quốc gia không ổn định | Cao | Cao | Circuit breaker, async flow, SLA agreement với provider |
| R3 | Tỷ lệ auto-approve thấp → quá tải manual review | Trung bình | Trung bình | Tune threshold dần, monitor metrics, tăng team review nếu cần |
| R4 | Fraud/spoofing vượt qua liveness detection | Thấp | Rất cao | Multi-layer: passive + active liveness, device fingerprint, velocity check |
| R5 | DPIA không pass → delay go-live | Trung bình | Cao | Bắt đầu DPIA song song với development, engage DPO sớm |
| R6 | Thay đổi quy định NHNN giữa chừng | Thấp | Cao | Thiết kế flexible (config-driven), monitor regulatory updates |
| R7 | NFC adoption thấp (ít điện thoại hỗ trợ) | Trung bình | Thấp | NFC là optional, graceful degradation |

---

## 9. Ngoài phạm vi (Out of Scope)

- eKYC cho khách hàng doanh nghiệp (legal entity) — cần module riêng
- Tích hợp eKYC với sản phẩm vay/thẻ tín dụng — phase sau
- Xây dựng AI/ML model OCR/face matching in-house — dùng vendor
- eKYC qua kênh quầy giao dịch — chỉ Mobile + Web
- Open Banking API cho đối tác — phase sau
- Xác minh bằng hộ chiếu/giấy phép lái xe — chỉ CCCD/CMND


---

## 10. Bổ sung sau Validation

### Findings từ Requirements Validator

#### Finding 1 (BLOCKING): DPIA Implementation
Cần thực hiện DPIA trước Sprint 1. Thêm story:

#### US-017: Thực hiện DPIA cho module eKYC
**Với vai trò** Data Protection Officer (DPO),
**Tôi muốn** thực hiện đánh giá tác động xử lý dữ liệu cá nhân (DPIA) cho module eKYC,
**Để** tuân thủ Nghị định 13/2023/NĐ-CP trước khi go-live.

**Acceptance Criteria:**
- **Cho trước** module eKYC thu thập dữ liệu sinh trắc học, **Khi** DPO đánh giá, **Thì** DPIA phải cover: mục đích xử lý, loại dữ liệu, cơ sở pháp lý, biện pháp bảo vệ, đánh giá rủi ro, biện pháp giảm thiểu
- **Cho trước** DPIA hoàn thành, **Khi** submit cho Bộ Công an (nếu yêu cầu), **Thì** lưu bản DPIA + approval + timestamp vào DMS
- **Cho trước** có thay đổi scope xử lý dữ liệu, **Khi** thêm phương thức xác minh mới, **Thì** cập nhật DPIA và re-assess rủi ro

**Priority**: P0 (BLOCKING — phải hoàn thành trước Sprint 1)
**Estimate**: M
**Dependencies**: Không (song song với development planning)
**Screen**: Không có UI

---

#### Finding 2: Quyền chủ thể dữ liệu (NĐ 13/2023)

#### US-018: Quản lý quyền chủ thể dữ liệu
**Với vai trò** khách hàng cá nhân,
**Tôi muốn** thực hiện quyền chủ thể dữ liệu (truy cập, chỉnh sửa, xóa, rút consent),
**Để** kiểm soát dữ liệu cá nhân của mình theo quy định pháp luật.

**Acceptance Criteria:**
- **Cho trước** KH đã hoàn thành eKYC, **Khi** KH yêu cầu xem dữ liệu đã thu thập, **Thì** hiển thị: loại dữ liệu, mục đích, thời gian lưu trữ, bên thứ 3 đã chia sẻ (nếu có)
- **Cho trước** KH muốn rút consent, **Khi** KH submit yêu cầu, **Thì** hệ thống xử lý trong 72 giờ + thông báo hậu quả (hạn chế dịch vụ) + ghi audit log
- **Cho trước** KH yêu cầu xóa dữ liệu, **Khi** không còn nghĩa vụ pháp lý lưu trữ, **Thì** xóa/ẩn danh hóa dữ liệu + xác nhận cho KH. Nếu còn nghĩa vụ lưu trữ (10 năm) → thông báo lý do không thể xóa ngay

**Priority**: P1
**Estimate**: M
**Dependencies**: US-006 (Consent), Core Banking CIF
**Screen**: Quản lý dữ liệu cá nhân (trong Settings)

---

#### Finding 3 (Khuyến nghị): Audit trail cho eKYC flow

#### US-019: Audit trail toàn bộ eKYC flow
**Với vai trò** Kiểm toán nội bộ,
**Tôi muốn** truy vấn audit trail đầy đủ cho mỗi hồ sơ eKYC,
**Để** kiểm tra tuân thủ quy trình và phục vụ thanh tra NHNN.

**Acceptance Criteria:**
- **Cho trước** 1 hồ sơ eKYC, **Khi** kiểm toán viên truy vấn, **Thì** hiển thị timeline đầy đủ: consent → OCR → NFC → face matching → CSDL verification → AML screening → scoring → decision → review (nếu có) — với timestamp, actor, kết quả từng bước
- **Cho trước** audit log, **Khi** kiểm tra tính toàn vẹn, **Thì** log phải immutable (append-only), có checksum, không thể sửa/xóa
- **Cho trước** yêu cầu xuất báo cáo kiểm toán, **Khi** filter theo thời gian/trạng thái, **Thì** xuất được báo cáo chi tiết (không chứa ảnh sinh trắc học — chỉ metadata + kết quả)

**Priority**: P1
**Estimate**: M
**Dependencies**: US-007, US-012
**Screen**: Back-office: Audit trail eKYC

---

#### Finding 4 (Khuyến nghị): Xử lý CCCD cũ (CMND 9 số / 12 số)

#### US-020: Hỗ trợ CMND cũ (9 số / 12 số)
**Với vai trò** khách hàng cá nhân chưa đổi CCCD gắn chip,
**Tôi muốn** dùng CMND cũ (9 số hoặc 12 số) để thực hiện eKYC,
**Để** vẫn có thể mở tài khoản dù chưa có CCCD mới.

**Acceptance Criteria:**
- **Cho trước** KH chụp CMND cũ (không phải CCCD), **Khi** OCR nhận dạng loại giấy tờ, **Thì** chuyển sang flow CMND: OCR + face matching (không có NFC) — mức xác thực thấp hơn CCCD
- **Cho trước** CMND cũ đã hết hạn hoặc không còn giá trị pháp lý, **Khi** hệ thống kiểm tra, **Thì** từ chối và hướng dẫn KH đổi CCCD gắn chip
- **Cho trước** KH dùng CMND cũ, **Khi** eKYC thành công, **Thì** đánh dấu mức xác thực thấp + nhắc KH nâng cấp khi có CCCD mới

**Priority**: P1
**Estimate**: M
**Dependencies**: US-001 (OCR module cần hỗ trợ cả CMND và CCCD)
**Screen**: Reuse screens Epic 1 (OCR detect loại giấy tờ tự động)

---

### Cập nhật thứ tự triển khai

| Bổ sung | Khi nào | Ghi chú |
|---------|---------|---------|
| US-017 (DPIA) | Trước Sprint 1 | BLOCKING — không go-live nếu chưa có |
| US-018 (Quyền chủ thể) | Phase 3 (Sprint 13-14) | Song song với US-009 |
| US-019 (Audit trail) | Phase 1 (Sprint 7-8) | Song song với US-012 |
| US-020 (CMND cũ) | Phase 1 (Sprint 1-2) | Mở rộng US-001 |

### Tổng kết sau validation
- Tổng user stories: 20 (16 gốc + 4 bổ sung)
- Stories P0: 9 (US-001→004, US-006→008, US-012, US-017)
- Stories P1: 8 (US-005, US-009→011, US-013, US-015, US-018→020)
- Stories P2: 3 (US-014, US-016)
- DPIA status: Cần hoàn thành trước Sprint 1
