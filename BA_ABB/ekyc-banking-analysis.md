# Phân Tích eKYC Cho Ngân Hàng

## 1. Tổng Quan eKYC

eKYC (Electronic Know Your Customer) là quy trình xác minh danh tính khách hàng bằng phương thức điện tử, thay thế việc gặp mặt trực tiếp truyền thống. Trong ngành ngân hàng, eKYC cho phép khách hàng mở tài khoản, đăng ký dịch vụ và xác thực giao dịch từ xa thông qua công nghệ sinh trắc học, OCR giấy tờ tùy thân, và đọc chip NFC.

### Tại sao eKYC quan trọng với ngân hàng?

| Khía cạnh | KYC truyền thống | eKYC |
|---|---|---|
| Thời gian onboarding | 2-5 ngày làm việc | 3-10 phút |
| Chi phí per customer | $20-$50 | $1-$5 |
| Tỷ lệ bỏ ngang (drop-off) | 30-40% | 5-15% |
| Khả năng scale | Giới hạn bởi nhân sự | Gần như không giới hạn |
| Audit trail | Giấy tờ, khó truy vết | Digital, truy vết 100% |

---

## 2. Khung Pháp Lý

### 2.1 Quốc tế — FATF Guidelines

FATF (Financial Action Task Force) là cơ quan quốc tế đặt ra chuẩn mực AML/CFT. Các nguyên tắc chính liên quan eKYC:

- **Risk-Based Approach (RBA)**: Mức độ xác minh tỷ lệ thuận với rủi ro. Khách hàng rủi ro thấp có thể dùng Simplified Due Diligence (SDD), rủi ro cao phải Enhanced Due Diligence (EDD).
- **Customer Due Diligence (CDD)**: Xác minh danh tính, xác minh beneficial owner, hiểu mục đích và bản chất mối quan hệ kinh doanh.
- **Digital Identity Guidance (2020, cập nhật 2025)**: FATF công nhận digital identity systems có thể đạt mức assurance tương đương hoặc cao hơn face-to-face, nếu đáp ứng tiêu chuẩn kỹ thuật.
- **Ongoing Monitoring**: KYC không phải one-time — phải continuous monitoring và periodic review.

> Nguồn: [FATF 2025 Guidance on AML/CFT Compliance](https://www.finreg-e.com/fatfs-2025-guidance-smarter-aml-cft-compliance/) — Content was rephrased for compliance with licensing restrictions.

### 2.2 Việt Nam — Quy Định NHNN

#### Nghị định 87/2019/NĐ-CP (sửa đổi NĐ 116/2013)
- Cho phép tổ chức tín dụng tự quyết định có cần gặp mặt trực tiếp hay không khi thiết lập quan hệ khách hàng mới.
- Đây là nền tảng pháp lý cho eKYC tại Việt Nam.

#### Thông tư 16/2020/TT-NHNN
- Quy định chi tiết về mở tài khoản thanh toán cá nhân bằng phương thức điện tử (eKYC).
- Ngân hàng được tự chọn phương thức, hình thức và công nghệ để xác minh khách hàng.
- Yêu cầu: thu thập thông tin sinh trắc học (khuôn mặt), chụp/scan giấy tờ tùy thân, đối chiếu với cơ sở dữ liệu.

#### Quyết định 2345/QĐ-NHNN (có hiệu lực từ 01/07/2024)
- **Bắt buộc xác thực sinh trắc học** cho giao dịch online trên 10 triệu VND hoặc tổng giao dịch trong ngày trên 20 triệu VND.
- Dữ liệu sinh trắc phải khớp với thông tin trên chip CCCD gắn chip.
- Mở rộng sang khách hàng doanh nghiệp từ 01/07/2025: pháp nhân phải xác thực sinh trắc của người đại diện pháp luật.

#### Thông tư 45 (hiệu lực 2026)
- Bắt buộc xác thực sinh trắc khi phát hành thẻ mới.
- Từ 01/01/2026: chỉ chấp nhận CCCD gắn chip, CMND còn hạn, hoặc định danh điện tử mức 2 (VNeID Level 2).

> Nguồn: [Aegona - NFC Face Authentication](https://www.aegona.com/technology/july-1-2024-use-nfc-face-authentication-online-transactions-over-10-million-vnd), [VietnamNet - Biometric Verification](https://vietnamnet.vn/en/vietnam-to-freeze-corporate-bank-accounts-without-biometric-verification-2404387.html), [Biometric Update - Vietnam IDs](https://www.biometricupdate.com/202511/vietnam-changes-ids-allowed-for-bank-transaction-identity-verification) — Content was rephrased for compliance with licensing restrictions.

---

## 3. Kiến Trúc Kỹ Thuật eKYC

### 3.1 Luồng eKYC Chuẩn (Happy Path)

```
┌─────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  1. Chụp    │───>│  2. OCR &    │───>│  3. Liveness  │───>│  4. Face     │
│  CMND/CCCD  │    │  Data Extract│    │  Detection   │    │  Matching    │
│  (2 mặt)    │    │              │    │              │    │              │
└─────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                                                  │
┌─────────────┐    ┌──────────────┐    ┌──────────────┐          │
│  7. Account │<───│  6. AML/PEP  │<───│  5. NFC Chip │<─────────┘
│  Created    │    │  Screening   │    │  Reading     │
│             │    │              │    │  (CCCD)      │
└─────────────┘    └──────────────┘    └──────────────┘
```

### 3.2 Chi Tiết Từng Bước

#### Bước 1: Document Capture (Chụp giấy tờ)
- Chụp mặt trước + mặt sau CCCD/CMND
- Yêu cầu kỹ thuật:
  - Độ phân giải tối thiểu: 1280x720px
  - Auto-crop và perspective correction
  - Phát hiện glare, blur, occlusion
  - Phát hiện giấy tờ giả (photo of photo, screen capture)
- Anti-fraud checks:
  - Edge detection để verify giấy tờ thật (không phải ảnh chụp màn hình)
  - Kiểm tra security features (hologram, microprint — nếu camera đủ tốt)

#### Bước 2: OCR & Data Extraction
- Trích xuất thông tin từ ảnh giấy tờ:
  - Họ tên, ngày sinh, giới tính
  - Số CCCD/CMND
  - Ngày cấp, nơi cấp, ngày hết hạn
  - Địa chỉ thường trú
  - MRZ (Machine Readable Zone) — nếu có
- Accuracy target: >98% cho tiếng Việt có dấu
- Cross-validation: so sánh data mặt trước vs mặt sau, so sánh OCR vs MRZ

#### Bước 3: Liveness Detection (Phát hiện người thật)
- **Mục đích**: Đảm bảo người đang thực hiện eKYC là người thật, không phải ảnh/video/deepfake.
- **Hai loại chính**:

| Loại | Mô tả | Ưu điểm | Nhược điểm |
|---|---|---|---|
| Active Liveness | Yêu cầu user thực hiện hành động (quay đầu, chớp mắt, mỉm cười) | Khó fake hơn | UX kém hơn, accessibility issues |
| Passive Liveness | Phân tích frame đơn hoặc video ngắn, không yêu cầu hành động | UX tốt, nhanh | Dễ bị deepfake tấn công hơn |

- **Threat landscape hiện tại (2025-2026)**:
  - Deepfake chiếm 24% các nỗ lực gian lận qua biometric checks dạng motion-based ([Infosecurity Magazine](https://www.infosecurity-magazine.com/news/deepfake-identity-attack-every/))
  - Trung bình cứ 5 phút có 1 cuộc tấn công deepfake vào hệ thống identity verification (Entrust, 2024)
  - Thiệt hại từ deepfake fraud: >$200 triệu chỉ trong Q1/2025 ([OZ Forensics](https://www.ozforensics.com/blog/articles/the-deepfake-surge-why-liveness-detection-is-the-answer))
  - Deepfake detections toàn cầu tăng gấp 4 lần từ 2023 đến 2024

- **Khuyến nghị**: Kết hợp cả Active + Passive liveness, cộng thêm injection attack detection (phát hiện video/ảnh được inject trực tiếp vào camera stream thay vì chụp thật).

- **Tiêu chuẩn**: ISO/IEC 30107-3 (Presentation Attack Detection), iBeta Level 1 & Level 2 certification.

#### Bước 4: Face Matching (Đối chiếu khuôn mặt)
- So sánh ảnh selfie (từ liveness) với ảnh trên giấy tờ (từ OCR)
- Threshold khuyến nghị:
  - FAR (False Acceptance Rate): <0.1% (1 trong 1,000)
  - FRR (False Rejection Rate): <5%
- Lưu ý: ảnh trên CMND/CCCD thường chất lượng thấp, cần model robust với low-quality input

#### Bước 5: NFC Chip Reading (Đọc chip CCCD)
- **Bắt buộc theo QĐ 2345/QĐ-NHNN** cho giao dịch trên ngưỡng.
- Quy trình:
  1. User đặt CCCD gắn chip lên mặt sau điện thoại (NFC)
  2. App đọc dữ liệu từ chip: ảnh chân dung, vân tay, thông tin cá nhân
  3. Verify chip authenticity (passive authentication — kiểm tra chữ ký số của chip)
  4. So sánh ảnh từ chip với ảnh selfie → face matching lần 2 (độ tin cậy cao hơn vì ảnh chip chất lượng tốt)
- Ưu điểm: Chip data được ký số bởi cơ quan cấp → gần như không thể giả mạo
- Hạn chế: Không phải điện thoại nào cũng có NFC, CMND cũ không có chip

#### Bước 6: AML/PEP/Sanctions Screening
- Kiểm tra khách hàng có nằm trong danh sách:
  - PEP (Politically Exposed Persons)
  - Sanctions lists (UN, OFAC, EU)
  - Adverse media
  - Internal blacklist của ngân hàng
- Frequency: Tại thời điểm onboarding + ongoing monitoring (daily/weekly batch)
- Nếu match → escalate cho Compliance team review thủ công

#### Bước 7: Account Creation
- Nếu tất cả bước pass → tạo tài khoản tự động
- Giới hạn ban đầu (theo risk-based approach):
  - Tài khoản eKYC có thể bị giới hạn hạn mức giao dịch ban đầu
  - Nâng hạn mức sau khi hoàn tất xác minh bổ sung (NFC chip, video call, hoặc đến quầy)

---

## 4. Rủi Ro & Biện Pháp Giảm Thiểu

### 4.1 Ma Trận Rủi Ro

| # | Rủi ro | Mức độ | Xác suất | Biện pháp giảm thiểu |
|---|---|---|---|---|
| R1 | Deepfake bypass liveness | Cao | Trung bình-Cao | Multi-layer liveness (active + passive + injection detection), ISO 30107-3 certified SDK |
| R2 | Giấy tờ giả/chỉnh sửa | Cao | Trung bình | Document forensics AI, NFC chip cross-check, database verification với C06 (CSDL quốc gia về dân cư) |
| R3 | Synthetic identity fraud | Cao | Thấp-Trung bình | Cross-reference nhiều nguồn data (telco, credit bureau), device fingerprinting |
| R4 | Account takeover sau eKYC | Cao | Trung bình | Step-up authentication cho giao dịch nhạy cảm, behavioral biometrics |
| R5 | Data breach thông tin sinh trắc | Rất cao | Thấp | Encryption at rest + in transit, không lưu raw biometric (chỉ lưu template/hash), tokenization |
| R6 | Bias trong AI model | Trung bình | Trung bình | Test trên diverse dataset (skin tone, age, gender), monitor FRR theo demographic |
| R7 | NFC không khả dụng | Thấp | Cao | Fallback flow: video call với agent, hoặc đến quầy giao dịch |
| R8 | Regulatory non-compliance | Rất cao | Thấp | Legal review định kỳ, automated compliance monitoring, audit trail đầy đủ |

### 4.2 Deepfake — Phân Tích Chuyên Sâu

Deepfake là mối đe dọa lớn nhất với eKYC hiện tại. Các vector tấn công:

1. **Face swap real-time**: Dùng app face swap để thay thế khuôn mặt trong video stream → bypass active liveness
2. **Pre-recorded deepfake video**: Tạo video deepfake trước, phát lại qua virtual camera
3. **Injection attack**: Bypass camera hoàn toàn, inject video/ảnh trực tiếp vào app stream
4. **3D mask**: In 3D mask từ ảnh nạn nhân → bypass passive liveness (nhưng thường fail active liveness)

**Chiến lược phòng thủ đa lớp:**

```
Layer 1: Device Integrity
├── Detect rooted/jailbroken device
├── Detect virtual camera / screen sharing
├── Detect hooking frameworks (Frida, Xposed)
└── App attestation (SafetyNet / App Attest)

Layer 2: Passive Liveness
├── Texture analysis (skin vs screen/paper)
├── Depth estimation (2D vs 3D)
├── Moiré pattern detection (screen replay)
└── Reflection/lighting consistency

Layer 3: Active Liveness
├── Random challenge (head turn direction, blink sequence)
├── Challenge-response timing analysis
└── Lip sync verification (nói số ngẫu nhiên)

Layer 4: Injection Detection
├── Camera API integrity check
├── Frame metadata validation
└── Sensor correlation (gyroscope + camera movement)

Layer 5: Post-Processing
├── Deepfake detection model (GAN artifact analysis)
├── Cross-session comparison (same person, different attempts)
└── Anomaly scoring + manual review queue
```

---

## 5. Yêu Cầu Phi Chức Năng (NFR)

### 5.1 Performance
| Metric | Target | Ghi chú |
|---|---|---|
| OCR processing time | <3 giây | Từ lúc chụp đến hiển thị kết quả |
| Liveness check | <5 giây | Bao gồm active challenge |
| Face matching | <2 giây | 1:1 comparison |
| NFC chip reading | <10 giây | Phụ thuộc thiết bị |
| End-to-end eKYC flow | <3 phút | Từ bắt đầu đến account created |
| API availability | 99.9% uptime | SLA với vendor |

### 5.2 Security
- Encryption: TLS 1.3 cho data in transit, AES-256 cho data at rest
- Biometric data: KHÔNG lưu raw image — chỉ lưu biometric template (irreversible hash)
- Data retention: Theo quy định NHNN — tối thiểu 5 năm cho hồ sơ KYC
- Access control: Role-based, principle of least privilege
- Audit log: Mọi thao tác trên dữ liệu KYC phải được ghi log (who, what, when, from where)
- Penetration testing: Ít nhất 1 lần/năm bởi bên thứ ba độc lập

### 5.3 Compliance
- FATF Recommendations (đặc biệt Rec. 10 — CDD)
- Luật Phòng chống rửa tiền 2022 (Việt Nam)
- Nghị định 87/2019/NĐ-CP
- Thông tư 16/2020/TT-NHNN
- Quyết định 2345/QĐ-NHNN
- Thông tư 45 (hiệu lực 2026)
- PDPA / Nghị định 13/2023/NĐ-CP (Bảo vệ dữ liệu cá nhân)
- ISO/IEC 30107-3 (Presentation Attack Detection)
- ISO/IEC 19795 (Biometric Performance Testing)

### 5.4 Accessibility & Inclusivity
- Hỗ trợ người dùng lớn tuổi: font size lớn, hướng dẫn bằng giọng nói
- Hỗ trợ điều kiện ánh sáng kém: auto-flash, guidance overlay
- Fallback cho thiết bị không có NFC: video call KYC hoặc đến quầy
- Hỗ trợ CMND cũ (9 số) trong giai đoạn chuyển tiếp (đến hết 2025)

---

## 6. Vendor Landscape

### 6.1 Các Vendor eKYC Phổ Biến Tại Việt Nam

| Vendor | Thế mạnh | Lưu ý |
|---|---|---|
| VNPT eKYC | Kết nối trực tiếp CSDL quốc gia dân cư (C06), NFC chip reading | Vendor nội địa, compliance tốt |
| FPT.AI eKYC | OCR tiếng Việt mạnh, liveness detection | Vendor nội địa |
| Jumio | Global leader, ISO 30107-3 certified | Chi phí cao, data sovereignty concern |
| Onfido | AI-powered document + biometric verification | Cần đánh giá data residency |
| Sumsub | All-in-one (KYC + AML + fraud), 220+ countries | Flexible pricing |
| iDenfy | Liveness + document verification, competitive pricing | Newer player |

### 6.2 Tiêu Chí Đánh Giá Vendor

1. **Accuracy**: FAR, FRR, liveness detection rate (yêu cầu iBeta Level 2)
2. **Compliance**: ISO 30107-3, SOC 2 Type II, data residency tại Việt Nam
3. **NFC support**: Đọc chip CCCD Việt Nam (bắt buộc theo QĐ 2345)
4. **OCR tiếng Việt**: Accuracy >98% cho tên có dấu
5. **Deepfake resistance**: Injection attack detection, GAN artifact detection
6. **Integration**: SDK (iOS/Android/Web), REST API, webhook support
7. **SLA**: Uptime 99.9%, response time <3s, support 24/7
8. **Pricing model**: Per-transaction vs subscription, volume discounts
9. **Data sovereignty**: Data phải được xử lý và lưu trữ tại Việt Nam (theo NĐ 13/2023)

---

## 7. Continuous KYC (cKYC) — Xu Hướng Mới

eKYC truyền thống là one-time tại onboarding. Xu hướng mới là Continuous KYC (cKYC):

- **Periodic re-verification**: Xác minh lại danh tính định kỳ (6-12 tháng) hoặc khi thay đổi thông tin
- **Transaction monitoring**: Phát hiện bất thường trong hành vi giao dịch → trigger re-KYC
- **Adverse media monitoring**: Quét tin tức liên tục về khách hàng
- **PEP/Sanctions list update**: Cập nhật danh sách hàng ngày, re-screen toàn bộ customer base
- **Behavioral biometrics**: Phân tích cách user tương tác với app (typing pattern, swipe behavior) để phát hiện account takeover

> Nguồn: [TrustDecision - eKYC Guide](https://trustdecision.com/articles/ekyc-guide-types-new-methods-and-how-ai-improves-smarter-verification) — Content was rephrased for compliance with licensing restrictions.

---

## 8. Roadmap Triển Khai Đề Xuất

### Phase 1: Foundation (Tháng 1-3)
- Lựa chọn vendor eKYC (RFP, POC, benchmark)
- Thiết kế kiến trúc tích hợp (SDK mobile + backend API)
- Xây dựng document capture + OCR flow
- Tích hợp liveness detection (passive + active)
- Face matching 1:1 (selfie vs document photo)

### Phase 2: Compliance (Tháng 4-6)
- Tích hợp NFC chip reading (CCCD gắn chip)
- Kết nối CSDL quốc gia dân cư (qua VNPT hoặc trực tiếp C06)
- AML/PEP/Sanctions screening integration
- Audit trail và reporting cho compliance team
- Penetration testing + security audit

### Phase 3: Enhancement (Tháng 7-9)
- Deepfake detection layer bổ sung
- Device integrity checks (root/jailbreak, virtual camera)
- Behavioral biometrics cho ongoing authentication
- Video call KYC fallback flow
- A/B testing UX optimization (giảm drop-off rate)

### Phase 4: Continuous (Tháng 10-12)
- cKYC framework (periodic re-verification)
- Transaction monitoring integration
- Adverse media + PEP monitoring automation
- Dashboard analytics cho fraud team
- Model retraining pipeline (cập nhật deepfake detection)

---

## 9. KPIs Đo Lường Hiệu Quả

| KPI | Target | Đo bằng |
|---|---|---|
| Onboarding completion rate | >85% | Số user hoàn thành / số user bắt đầu eKYC |
| Average onboarding time | <3 phút | Từ bước 1 đến account created |
| False Rejection Rate (FRR) | <5% | Số user thật bị reject / tổng user thật |
| False Acceptance Rate (FAR) | <0.1% | Số fraud pass / tổng attempts |
| Deepfake detection rate | >99% | Số deepfake bị chặn / tổng deepfake attempts |
| NFC success rate | >90% | Số lần đọc chip thành công / tổng attempts |
| Compliance audit pass rate | 100% | Số findings = 0 trong audit |
| System uptime | 99.9% | Monitoring |
| Customer satisfaction (CSAT) | >4.0/5.0 | Survey sau onboarding |

---

## 10. Tài Liệu Tham Khảo

1. FATF 2025 Guidance on AML/CFT Compliance — [finreg-e.com](https://www.finreg-e.com/fatfs-2025-guidance-smarter-aml-cft-compliance/)
2. Vietnam Decision 2345/QĐ-NHNN — NFC biometric authentication — [aegona.com](https://www.aegona.com/technology/july-1-2024-use-nfc-face-authentication-online-transactions-over-10-million-vnd)
3. Vietnam Circular 16/2020/TT-NHNN — eKYC for payment accounts — [lexology.com](https://www.lexology.com/library/detail.aspx?g=43219f60-4073-49a7-83c9-1eef62f89ef2)
4. Vietnam biometric banking mandate 2025-2026 — [corbado.com](https://www.corbado.com/blog/vietnam-banking-biometrics)
5. Deepfake fraud statistics 2024-2025 — [infosecurity-magazine.com](https://www.infosecurity-magazine.com/news/deepfake-identity-attack-every/)
6. Liveness detection and deepfake threats — [ozforensics.com](https://www.ozforensics.com/blog/articles/the-deepfake-surge-why-liveness-detection-is-the-answer)
7. Circular 45 — Biometric for card issuance 2026 — [vietnamnet.vn](https://vietnamnet.vn/en/banks-to-require-biometric-verification-for-card-issuance-from-2026-2468773.html)
8. Vietnam ID changes for banking 2026 — [biometricupdate.com](https://www.biometricupdate.com/202511/vietnam-changes-ids-allowed-for-bank-transaction-identity-verification)
9. Decree 87/2019/NĐ-CP — eKYC legal foundation — [vci-legal.com](https://vci-legal.com/news/official-deployment-of-e-kyc-in-banking-sector)
10. Continuous KYC trends — [trustdecision.com](https://trustdecision.com/articles/ekyc-guide-types-new-methods-and-how-ai-improves-smarter-verification)
