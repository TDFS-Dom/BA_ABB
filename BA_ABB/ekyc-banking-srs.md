# SRS: Hệ Thống eKYC Ngân Hàng

## 1. Giới Thiệu

### 1.1 Mục đích
Tài liệu này mô tả đầy đủ yêu cầu phần mềm (Software Requirements Specification) cho Hệ thống eKYC (Electronic Know Your Customer) của ngân hàng. SRS phục vụ làm cơ sở cho:
- Dev team nội bộ triển khai các module core
- Vendor eKYC tích hợp SDK/API
- QA team xây dựng test plan
- Compliance team đánh giá tuân thủ pháp lý

### 1.2 Phạm vi
Hệ thống eKYC bao gồm toàn bộ quy trình xác minh danh tính khách hàng điện tử, từ thu thập giấy tờ đến tạo tài khoản, và giám sát liên tục (Continuous KYC). Cụ thể:
- Document Capture & OCR
- Liveness Detection (Active + Passive)
- Face Matching
- NFC Chip Reading (CCCD gắn chip)
- AML/PEP/Sanctions Screening
- Account Creation & Risk Scoring
- Continuous KYC (cKYC) — periodic re-verification, transaction monitoring
- Fallback flows (Video Call KYC, đến quầy)
- Admin Dashboard & Compliance Reporting

### 1.3 Định nghĩa & Viết tắt

| Viết tắt | Định nghĩa |
|---|---|
| eKYC | Electronic Know Your Customer |
| cKYC | Continuous KYC — giám sát danh tính liên tục sau onboarding |
| CCCD | Căn cước công dân (gắn chip) |
| CMND | Chứng minh nhân dân (9 số, không chip) |
| OCR | Optical Character Recognition |
| NFC | Near Field Communication |
| FAR | False Acceptance Rate — tỷ lệ chấp nhận sai (fraud lọt qua) |
| FRR | False Rejection Rate — tỷ lệ từ chối sai (user thật bị reject) |
| PAD | Presentation Attack Detection — phát hiện tấn công giả mạo sinh trắc |
| AML | Anti-Money Laundering |
| CFT | Combating the Financing of Terrorism |
| PEP | Politically Exposed Person |
| CDD | Customer Due Diligence |
| EDD | Enhanced Due Diligence |
| SDD | Simplified Due Diligence |
| MRZ | Machine Readable Zone |
| FATF | Financial Action Task Force |
| NHNN | Ngân hàng Nhà nước Việt Nam |
| C06 | Cục Cảnh sát quản lý hành chính về trật tự xã hội (CSDL quốc gia về dân cư) |
| VNeID | Ứng dụng định danh điện tử quốc gia |

### 1.4 Tài liệu tham khảo

| # | Tài liệu | Mô tả |
|---|---|---|
| REF-01 | Nghị định 87/2019/NĐ-CP | Nền tảng pháp lý cho eKYC tại Việt Nam |
| REF-02 | Thông tư 16/2020/TT-NHNN | Quy định mở tài khoản thanh toán cá nhân bằng eKYC |
| REF-03 | Quyết định 2345/QĐ-NHNN | Bắt buộc xác thực sinh trắc cho giao dịch online trên ngưỡng |
| REF-04 | Thông tư 45 (hiệu lực 2026) | Bắt buộc xác thực sinh trắc khi phát hành thẻ mới |
| REF-05 | Nghị định 13/2023/NĐ-CP | Bảo vệ dữ liệu cá nhân (PDPA Việt Nam) |
| REF-06 | Luật Phòng chống rửa tiền 2022 | Quy định AML/CFT tại Việt Nam |
| REF-07 | FATF Recommendations (2025) | Chuẩn mực quốc tế AML/CFT, Digital Identity Guidance |
| REF-08 | ISO/IEC 30107-3 | Tiêu chuẩn Presentation Attack Detection |
| REF-09 | ISO/IEC 19795 | Tiêu chuẩn Biometric Performance Testing |
| REF-10 | ekyc-banking-analysis.md | Tài liệu phân tích eKYC nội bộ |

---

## 2. Mô Tả Tổng Quan

### 2.1 Bối cảnh sản phẩm (Product Perspective)
Hệ thống eKYC là một module tích hợp vào hệ sinh thái ngân hàng số (Digital Banking Platform), tương tác với:
- **Mobile Banking App** (iOS/Android): Giao diện chính cho khách hàng thực hiện eKYC
- **Core Banking System (CBS)**: Nhận kết quả eKYC để tạo tài khoản, cập nhật hồ sơ khách hàng
- **eKYC Vendor SDK**: Cung cấp engine OCR, liveness detection, face matching
- **CSDL Quốc gia dân cư (C06)**: Xác minh thông tin CCCD qua kết nối VNPT hoặc trực tiếp
- **AML/Sanctions Database**: Danh sách PEP, sanctions (UN, OFAC, EU), adverse media
- **Fraud Detection System**: Nhận signal từ eKYC để scoring rủi ro
- **Admin Portal**: Dashboard cho compliance team, fraud team, operations

### 2.2 Chức năng chính (Product Functions)
1. Thu thập và xác minh giấy tờ tùy thân (Document Capture & Verification)
2. Phát hiện người thật (Liveness Detection)
3. Đối chiếu khuôn mặt (Face Matching)
4. Đọc chip NFC trên CCCD (NFC Chip Reading)
5. Sàng lọc AML/PEP/Sanctions
6. Tạo tài khoản tự động với risk scoring
7. Giám sát danh tính liên tục (Continuous KYC)
8. Luồng fallback (Video Call KYC, đến quầy)
9. Quản trị và báo cáo compliance

### 2.3 Đặc điểm người dùng (User Characteristics)

| Vai trò | Mô tả | Kỹ năng kỹ thuật |
|---|---|---|
| Khách hàng cá nhân | Người dùng cuối thực hiện eKYC qua mobile app | Thấp — cần UX đơn giản, hướng dẫn rõ ràng |
| Khách hàng doanh nghiệp | Người đại diện pháp luật xác thực sinh trắc (từ 01/07/2025 theo QĐ 2345) | Thấp-Trung bình |
| Compliance Officer | Xem xét hồ sơ eKYC bị flag, quyết định approve/reject | Cao — hiểu quy định AML/KYC |
| Fraud Analyst | Phân tích các case nghi ngờ gian lận, deepfake | Cao — hiểu kỹ thuật fraud |
| System Admin | Cấu hình hệ thống, quản lý vendor integration, monitoring | Cao — kỹ thuật IT |
| Video Call Agent | Thực hiện xác minh qua video call (fallback flow) | Trung bình — được đào tạo KYC |

### 2.4 Ràng buộc (Constraints)
- **Pháp lý**: Tuân thủ toàn bộ quy định NHNN (REF-01 đến REF-06) và FATF (REF-07)
- **Data sovereignty**: Toàn bộ dữ liệu eKYC PHẢI được xử lý và lưu trữ tại Việt Nam (theo NĐ 13/2023/NĐ-CP)
- **Biometric data**: KHÔNG lưu raw biometric image — chỉ lưu biometric template (irreversible hash)
- **Thiết bị**: Hỗ trợ iOS 14+ và Android 8+; NFC chỉ khả dụng trên thiết bị có phần cứng NFC
- **Giấy tờ chấp nhận**: CCCD gắn chip, CMND còn hạn (đến hết 2025), VNeID Level 2 (từ 01/01/2026 theo Thông tư 45)
- **Vendor dependency**: Phụ thuộc vendor eKYC cho engine OCR/liveness/face matching; cần SLA rõ ràng

### 2.5 Giả định & Phụ thuộc (Assumptions & Dependencies)

| # | Giả định / Phụ thuộc | Loại |
|---|---|---|
| A1 | Vendor eKYC cung cấp SDK hỗ trợ iOS/Android với NFC chip reading cho CCCD Việt Nam | Phụ thuộc |
| A2 | Kết nối CSDL quốc gia dân cư (C06) khả dụng qua VNPT hoặc kênh trực tiếp | Phụ thuộc |
| A3 | Core Banking System có API tạo tài khoản tự động từ kết quả eKYC | Phụ thuộc |
| A4 | Khách hàng có smartphone với camera ≥8MP và kết nối internet ổn định | Giả định |
| A5 | AML/Sanctions database được cập nhật ít nhất hàng ngày | Phụ thuộc |
| A6 | Ngân hàng có hạ tầng cloud/on-premise đáp ứng data residency tại Việt Nam | Giả định |
| A7 | Vendor eKYC đạt chứng nhận ISO/IEC 30107-3 và iBeta Level 2 | Phụ thuộc |

---

## 3. Yêu Cầu Chức Năng (Functional Requirements)

### 3.1 Module: Document Capture & Verification

#### FR-DOC-001: Chụp giấy tờ tùy thân
- **Mô tả**: Hệ thống PHẢI cho phép khách hàng chụp ảnh mặt trước và mặt sau giấy tờ tùy thân (CCCD gắn chip, CMND còn hạn) thông qua camera điện thoại.
- **Actor**: Khách hàng
- **Precondition**: Khách hàng đã cài đặt mobile app và cấp quyền truy cập camera
- **Input**: Ảnh chụp mặt trước + mặt sau giấy tờ, độ phân giải tối thiểu 1280x720px
- **Processing**:
  1. Hiển thị khung hướng dẫn (guide overlay) trên camera để khách hàng căn chỉnh giấy tờ
  2. Auto-detect cạnh giấy tờ (edge detection)
  3. Auto-crop và perspective correction
  4. Kiểm tra chất lượng ảnh: blur detection (ngưỡng Laplacian variance >100), glare detection, occlusion detection
  5. Nếu ảnh không đạt chất lượng → hiển thị thông báo cụ thể và yêu cầu chụp lại
- **Output**: 2 ảnh giấy tờ (trước + sau) đã crop, đạt chất lượng, sẵn sàng cho OCR
- **Business Rules**:
  - BR-DOC-001: Chỉ chấp nhận CCCD gắn chip, CMND 9 số còn hạn, hoặc VNeID Level 2 (từ 01/01/2026 theo Thông tư 45)
  - BR-DOC-002: Giấy tờ hết hạn → reject, hiển thị thông báo "Giấy tờ đã hết hạn, vui lòng sử dụng giấy tờ còn hiệu lực"
  - BR-DOC-003: Tối đa 3 lần chụp lại cho mỗi mặt; sau 3 lần fail → chuyển sang fallback flow
- **Acceptance Criteria**:
  - **AC1**: Given khách hàng mở camera, When đặt CCCD vào khung hướng dẫn, Then hệ thống auto-detect cạnh và chụp tự động khi ảnh đạt chất lượng
  - **AC2**: Given ảnh bị blur (Laplacian variance ≤100), When hệ thống kiểm tra chất lượng, Then hiển thị "Ảnh bị mờ, vui lòng giữ tay vững và chụp lại"
  - **AC3**: Given ảnh bị glare, When hệ thống kiểm tra, Then hiển thị "Ảnh bị chói, vui lòng tránh ánh sáng trực tiếp"
  - **AC4**: Given khách hàng chụp fail 3 lần liên tiếp, When hệ thống đếm số lần fail, Then chuyển sang fallback flow (xem FR-FALL-001)
- **Error Handling**:
  - Camera permission denied → hiển thị hướng dẫn cấp quyền trong Settings
  - Camera hardware fail → hiển thị "Không thể truy cập camera, vui lòng thử lại hoặc liên hệ hỗ trợ"
  - Mất kết nối mạng trong lúc upload → retry tự động 3 lần, sau đó hiển thị "Mất kết nối, vui lòng kiểm tra mạng"

#### FR-DOC-002: Phát hiện giấy tờ giả
- **Mô tả**: Hệ thống PHẢI phát hiện giấy tờ giả mạo bao gồm: ảnh chụp từ màn hình (screen capture), ảnh chụp từ ảnh in (photo of photo), giấy tờ bị chỉnh sửa (tampered document).
- **Actor**: Hệ thống (tự động)
- **Precondition**: Ảnh giấy tờ đã qua bước FR-DOC-001
- **Input**: Ảnh giấy tờ đã crop
- **Processing**:
  1. Edge detection: kiểm tra cạnh giấy tờ có tự nhiên (không phải cạnh màn hình)
  2. Moiré pattern detection: phát hiện pattern lưới từ chụp màn hình
  3. Reflection analysis: phát hiện phản chiếu bất thường từ bề mặt màn hình/ảnh in
  4. Tamper detection: phát hiện vùng ảnh bị chỉnh sửa (font inconsistency, pixel anomaly)
  5. Security feature verification: kiểm tra hologram, microprint (nếu camera đủ độ phân giải)
- **Output**: Kết quả: PASS (giấy tờ thật) hoặc FAIL (nghi ngờ giả mạo) kèm confidence score (0-100)
- **Business Rules**:
  - BR-DOC-004: Confidence score <70 → FAIL tự động, chuyển fallback flow
  - BR-DOC-005: Confidence score 70-85 → FLAG, đưa vào hàng đợi manual review
  - BR-DOC-006: Confidence score >85 → PASS tự động
- **Acceptance Criteria**:
  - **AC1**: Given ảnh chụp từ màn hình điện thoại/máy tính, When hệ thống phân tích, Then detect moiré pattern và trả về FAIL với confidence <70
  - **AC2**: Given giấy tờ thật chụp trực tiếp, When hệ thống phân tích, Then trả về PASS với confidence >85
  - **AC3**: Given giấy tờ bị chỉnh sửa Photoshop (thay đổi tên/ngày sinh), When hệ thống phân tích tamper detection, Then trả về FAIL hoặc FLAG
- **Error Handling**:
  - Vendor API timeout (>5s) → retry 2 lần, sau đó đưa vào manual review queue
  - Vendor API trả về lỗi không xác định → log error, đưa vào manual review queue

#### FR-DOC-003: OCR & Trích xuất dữ liệu
- **Mô tả**: Hệ thống PHẢI trích xuất thông tin cá nhân từ ảnh giấy tờ bằng OCR với accuracy >98% cho tiếng Việt có dấu.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Ảnh giấy tờ đã qua FR-DOC-001 và FR-DOC-002 (PASS hoặc FLAG)
- **Input**: Ảnh mặt trước + mặt sau giấy tờ
- **Processing**:
  1. OCR mặt trước: trích xuất họ tên, ngày sinh, giới tính, số CCCD/CMND, quốc tịch, ảnh chân dung
  2. OCR mặt sau: trích xuất ngày cấp, nơi cấp, ngày hết hạn, đặc điểm nhận dạng
  3. MRZ parsing (nếu có): trích xuất data từ Machine Readable Zone
  4. Cross-validation: so sánh data mặt trước vs mặt sau (số CCCD phải khớp), so sánh OCR vs MRZ (nếu có)
  5. Format validation: kiểm tra format số CCCD (12 số), ngày sinh (dd/mm/yyyy), ngày hết hạn
- **Output**: Structured data object chứa toàn bộ thông tin trích xuất + confidence score per field
- **Business Rules**:
  - BR-DOC-007: Nếu bất kỳ field nào có confidence <90% → highlight field đó, yêu cầu khách hàng xác nhận/sửa thủ công
  - BR-DOC-008: Nếu cross-validation fail (số CCCD mặt trước ≠ mặt sau) → reject, yêu cầu chụp lại
  - BR-DOC-009: Nếu giấy tờ hết hạn (ngày hết hạn < ngày hiện tại) → reject (xem BR-DOC-002)
- **Acceptance Criteria**:
  - **AC1**: Given CCCD gắn chip chụp rõ nét, When OCR xử lý, Then trích xuất đúng họ tên có dấu tiếng Việt với accuracy >98%
  - **AC2**: Given số CCCD mặt trước là "001234567890" và mặt sau là "001234567891", When cross-validation, Then reject và hiển thị "Thông tin 2 mặt không khớp, vui lòng chụp lại"
  - **AC3**: Given field "Họ tên" có confidence 85%, When hiển thị kết quả, Then highlight field và cho phép khách hàng sửa thủ công
- **Error Handling**:
  - OCR không đọc được (confidence toàn bộ <50%) → yêu cầu chụp lại với hướng dẫn "Đặt giấy tờ trên nền phẳng, đủ ánh sáng"
  - OCR timeout (>3s) → retry 1 lần, nếu vẫn fail → hiển thị "Đang xử lý, vui lòng đợi" và tăng timeout lên 10s



### 3.2 Module: Liveness Detection

#### FR-LIVE-001: Passive Liveness Detection
- **Mô tả**: Hệ thống PHẢI thực hiện passive liveness detection trên ảnh/video selfie của khách hàng để phát hiện ảnh in, ảnh màn hình, hoặc mặt nạ 3D mà KHÔNG yêu cầu khách hàng thực hiện hành động cụ thể.
- **Actor**: Hệ thống (tự động), khách hàng (chụp selfie)
- **Precondition**: Khách hàng đã hoàn thành bước Document Capture (FR-DOC-001)
- **Input**: Ảnh selfie hoặc video ngắn (2-3 giây) từ camera trước
- **Processing**:
  1. Texture analysis: phân biệt da thật vs bề mặt giấy/màn hình
  2. Depth estimation: phân biệt khuôn mặt 3D thật vs ảnh 2D
  3. Moiré pattern detection: phát hiện replay từ màn hình
  4. Reflection/lighting consistency: kiểm tra ánh sáng phản chiếu tự nhiên
- **Output**: Kết quả PASS/FAIL kèm confidence score (0-100)
- **Business Rules**:
  - BR-LIVE-001: Passive liveness PHẢI đạt tiêu chuẩn ISO/IEC 30107-3 Level 1
  - BR-LIVE-002: Confidence score <60 → FAIL tự động
  - BR-LIVE-003: Confidence score 60-80 → chuyển sang Active Liveness (FR-LIVE-002)
  - BR-LIVE-004: Confidence score >80 → PASS, tiếp tục Face Matching
- **Acceptance Criteria**:
  - **AC1**: Given ảnh in khuôn mặt đặt trước camera, When passive liveness phân tích, Then trả về FAIL với confidence <60
  - **AC2**: Given khuôn mặt thật trước camera trong điều kiện ánh sáng bình thường, When passive liveness phân tích, Then trả về PASS với confidence >80
  - **AC3**: Given video phát lại từ màn hình điện thoại khác, When passive liveness phân tích, Then detect moiré pattern và trả về FAIL
- **Error Handling**:
  - Camera trước không khả dụng → hiển thị "Không thể truy cập camera trước, vui lòng kiểm tra thiết bị"
  - Ánh sáng quá tối (brightness <30%) → hiển thị "Ánh sáng không đủ, vui lòng di chuyển đến nơi sáng hơn"

#### FR-LIVE-002: Active Liveness Detection
- **Mô tả**: Hệ thống PHẢI thực hiện active liveness detection bằng cách yêu cầu khách hàng thực hiện hành động ngẫu nhiên (quay đầu, chớp mắt, mỉm cười, hoặc đọc số) để xác nhận người thật.
- **Actor**: Khách hàng (thực hiện hành động), Hệ thống (xác minh)
- **Precondition**: Passive liveness trả về confidence 60-80 (BR-LIVE-003), HOẶC hệ thống cấu hình luôn yêu cầu active liveness
- **Input**: Video stream từ camera trước (5-10 giây)
- **Processing**:
  1. Hệ thống chọn ngẫu nhiên 2-3 challenge từ pool: quay đầu trái/phải, chớp mắt, mỉm cười, đọc số hiển thị trên màn hình
  2. Hiển thị hướng dẫn bằng animation + text + audio (accessibility)
  3. Phân tích video: kiểm tra hành động đúng, timing phù hợp (không quá nhanh — nghi pre-recorded)
  4. Lip sync verification (nếu challenge là đọc số): so sánh audio với chuyển động môi
  5. Challenge-response timing analysis: thời gian phản hồi phải trong khoảng 0.5-5 giây (quá nhanh = bot, quá chậm = video chỉnh sửa)
- **Output**: Kết quả PASS/FAIL kèm confidence score
- **Business Rules**:
  - BR-LIVE-005: Active liveness PHẢI đạt tiêu chuẩn ISO/IEC 30107-3 Level 2 và iBeta Level 2
  - BR-LIVE-006: Tối đa 3 lần thử active liveness; sau 3 lần fail → chuyển fallback flow
  - BR-LIVE-007: Challenge pool PHẢI có ít nhất 5 loại challenge khác nhau để tránh replay attack
  - BR-LIVE-008: Thứ tự và tổ hợp challenge PHẢI ngẫu nhiên mỗi lần thử
- **Acceptance Criteria**:
  - **AC1**: Given hệ thống yêu cầu "Quay đầu sang trái", When khách hàng quay đầu sang trái trong 0.5-5 giây, Then hệ thống detect hành động đúng và PASS challenge đó
  - **AC2**: Given video deepfake pre-recorded thực hiện đúng hành động, When challenge-response timing <0.3 giây (quá nhanh), Then hệ thống FAIL với lý do "Phản hồi bất thường"
  - **AC3**: Given khách hàng fail active liveness 3 lần, When hệ thống đếm số lần fail, Then chuyển sang fallback flow và hiển thị "Xác minh không thành công, vui lòng thử phương thức khác"
- **Error Handling**:
  - Khách hàng không thực hiện hành động trong 15 giây → timeout, hiển thị "Hết thời gian, vui lòng thử lại"
  - Audio không khả dụng (thiết bị tắt mic) → bỏ qua lip sync challenge, dùng challenge khác

#### FR-LIVE-003: Injection Attack Detection
- **Mô tả**: Hệ thống PHẢI phát hiện injection attack — khi kẻ gian lận bypass camera và inject video/ảnh trực tiếp vào app stream.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Đang trong quá trình liveness detection (FR-LIVE-001 hoặc FR-LIVE-002)
- **Input**: Camera stream metadata, device sensor data
- **Processing**:
  1. Camera API integrity check: xác minh stream đến từ camera vật lý, không phải virtual camera
  2. Frame metadata validation: kiểm tra timestamp, resolution, encoding consistency
  3. Sensor correlation: so sánh dữ liệu gyroscope/accelerometer với chuyển động trong video (nếu user di chuyển điện thoại, gyroscope phải phản ánh)
  4. Detect virtual camera apps (ManyCam, OBS Virtual Camera, v.v.)
  5. Detect screen sharing / screen mirroring
- **Output**: Kết quả PASS/FAIL
- **Business Rules**:
  - BR-LIVE-009: Nếu phát hiện virtual camera hoặc injection → FAIL ngay lập tức, KHÔNG cho retry
  - BR-LIVE-010: Log toàn bộ injection attempt vào fraud monitoring system
  - BR-LIVE-011: Block device fingerprint trong 24 giờ sau injection attempt
- **Acceptance Criteria**:
  - **AC1**: Given kẻ gian lận dùng OBS Virtual Camera để inject video, When hệ thống kiểm tra camera API, Then detect virtual camera và FAIL ngay lập tức
  - **AC2**: Given kẻ gian lận dùng Frida framework để hook camera API, When hệ thống kiểm tra app integrity, Then detect hooking và FAIL
  - **AC3**: Given injection attempt bị detect, When hệ thống log event, Then fraud monitoring system nhận được alert trong <30 giây
- **Error Handling**:
  - Sensor data không khả dụng (thiết bị cũ không có gyroscope) → bỏ qua sensor correlation, tăng weight cho các check khác

#### FR-LIVE-004: Device Integrity Check
- **Mô tả**: Hệ thống PHẢI kiểm tra tính toàn vẹn của thiết bị trước khi bắt đầu liveness detection.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Khách hàng mở flow eKYC trên mobile app
- **Input**: Device metadata, OS information, app attestation token
- **Processing**:
  1. Detect rooted (Android) / jailbroken (iOS) device
  2. Detect hooking frameworks: Frida, Xposed, Substrate
  3. App attestation: Google SafetyNet/Play Integrity (Android), App Attest (iOS)
  4. Detect emulator / virtual machine
  5. Detect screen recording / screen sharing đang active
- **Output**: Kết quả PASS/FAIL kèm danh sách risk factors detected
- **Business Rules**:
  - BR-LIVE-012: Rooted/jailbroken device → FAIL, hiển thị "Thiết bị không đảm bảo an toàn, vui lòng sử dụng thiết bị khác"
  - BR-LIVE-013: Emulator detected → FAIL, KHÔNG hiển thị lý do cụ thể (tránh leak detection logic)
  - BR-LIVE-014: App attestation fail → FAIL, yêu cầu cập nhật app từ store chính thức
- **Acceptance Criteria**:
  - **AC1**: Given thiết bị Android đã root bằng Magisk, When hệ thống kiểm tra, Then detect root và FAIL
  - **AC2**: Given thiết bị iOS chưa jailbreak, app cài từ App Store, When hệ thống kiểm tra App Attest, Then PASS
  - **AC3**: Given app chạy trên Android emulator (BlueStacks), When hệ thống kiểm tra, Then detect emulator và FAIL
- **Error Handling**:
  - SafetyNet/Play Integrity API không phản hồi (Google service down) → cho phép tiếp tục với warning flag, tăng monitoring level

### 3.3 Module: Face Matching

#### FR-FACE-001: Face Matching — Selfie vs Document Photo
- **Mô tả**: Hệ thống PHẢI so sánh ảnh selfie (từ liveness detection) với ảnh chân dung trên giấy tờ (từ OCR) để xác minh cùng một người.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Liveness detection PASS (FR-LIVE-001 hoặc FR-LIVE-002), OCR đã trích xuất ảnh chân dung (FR-DOC-003)
- **Input**: Ảnh selfie (từ liveness), ảnh chân dung trên giấy tờ (từ OCR)
- **Processing**:
  1. Face detection: xác định vùng khuôn mặt trong cả 2 ảnh
  2. Face alignment: chuẩn hóa góc, kích thước
  3. Feature extraction: trích xuất face embedding vector
  4. Similarity comparison: tính cosine similarity giữa 2 embedding vectors
  5. Threshold check: so sánh similarity score với ngưỡng cấu hình
- **Output**: Kết quả MATCH/NO_MATCH kèm similarity score (0-100)
- **Business Rules**:
  - BR-FACE-001: FAR (False Acceptance Rate) PHẢI <0.1% (1 trong 1,000)
  - BR-FACE-002: FRR (False Rejection Rate) PHẢI <5%
  - BR-FACE-003: Similarity score ≥75 → MATCH
  - BR-FACE-004: Similarity score 60-74 → FLAG, đưa vào manual review
  - BR-FACE-005: Similarity score <60 → NO_MATCH, reject
  - BR-FACE-006: Nếu NO_MATCH → cho phép retry tối đa 2 lần (chụp lại selfie); sau 2 lần → fallback flow
- **Acceptance Criteria**:
  - **AC1**: Given selfie và ảnh CCCD của cùng một người, When face matching, Then trả về MATCH với similarity ≥75
  - **AC2**: Given selfie của người A và ảnh CCCD của người B, When face matching, Then trả về NO_MATCH với similarity <60
  - **AC3**: Given selfie chất lượng thấp (ánh sáng yếu) nhưng cùng người, When face matching, Then vẫn trả về MATCH (FRR <5%)
- **Error Handling**:
  - Không detect được khuôn mặt trong ảnh giấy tờ (ảnh quá mờ) → yêu cầu chụp lại giấy tờ
  - Face matching API timeout (>2s) → retry 2 lần, sau đó đưa vào manual review

#### FR-FACE-002: Face Matching — Selfie vs NFC Chip Photo
- **Mô tả**: Hệ thống PHẢI so sánh ảnh selfie với ảnh chân dung từ chip NFC trên CCCD (face matching lần 2, độ tin cậy cao hơn vì ảnh chip chất lượng tốt).
- **Actor**: Hệ thống (tự động)
- **Precondition**: NFC chip reading thành công (FR-NFC-001), liveness detection PASS
- **Input**: Ảnh selfie (từ liveness), ảnh chân dung từ chip NFC
- **Processing**: Tương tự FR-FACE-001 nhưng với ảnh chip chất lượng cao hơn
- **Output**: Kết quả MATCH/NO_MATCH kèm similarity score
- **Business Rules**:
  - BR-FACE-007: Ngưỡng similarity cho NFC photo matching: ≥80 (cao hơn document photo vì ảnh chip chất lượng tốt)
  - BR-FACE-008: Kết quả NFC face matching có weight cao hơn document face matching trong quyết định cuối cùng
  - BR-FACE-009: Nếu NFC face matching FAIL nhưng document face matching PASS → FLAG, manual review bắt buộc
- **Acceptance Criteria**:
  - **AC1**: Given selfie và ảnh chip NFC của cùng một người, When face matching, Then trả về MATCH với similarity ≥80
  - **AC2**: Given NFC face matching FAIL (score 55) nhưng document face matching PASS (score 78), When hệ thống đánh giá, Then FLAG case và đưa vào manual review queue
- **Error Handling**:
  - Ảnh từ chip NFC bị corrupt → bỏ qua NFC face matching, dựa vào document face matching + tăng monitoring level



### 3.4 Module: NFC Chip Reading

#### FR-NFC-001: Đọc chip NFC trên CCCD
- **Mô tả**: Hệ thống PHẢI cho phép khách hàng đọc dữ liệu từ chip NFC trên CCCD gắn chip bằng cách đặt CCCD lên mặt sau điện thoại.
- **Actor**: Khách hàng (đặt CCCD lên điện thoại), Hệ thống (đọc và xác minh chip)
- **Precondition**: Thiết bị có phần cứng NFC và NFC đã bật; khách hàng có CCCD gắn chip
- **Input**: Dữ liệu từ chip NFC trên CCCD
- **Processing**:
  1. Hiển thị hướng dẫn đặt CCCD (animation vị trí NFC antenna trên điện thoại)
  2. Thiết lập kết nối NFC với chip CCCD
  3. BAC (Basic Access Control): dùng MRZ data (từ OCR) làm key để mở khóa chip
  4. Đọc dữ liệu từ chip: ảnh chân dung, vân tay (template), thông tin cá nhân, chữ ký số
  5. Passive Authentication: xác minh chữ ký số của chip (do cơ quan cấp ký) → đảm bảo data không bị tamper
  6. Cross-validation: so sánh data từ chip vs data từ OCR (họ tên, ngày sinh, số CCCD phải khớp)
- **Output**: Structured data từ chip + kết quả Passive Authentication (VALID/INVALID) + ảnh chân dung từ chip
- **Business Rules**:
  - BR-NFC-001: NFC chip reading là BẮT BUỘC cho giao dịch online trên 10 triệu VND hoặc tổng giao dịch trong ngày trên 20 triệu VND (theo QĐ 2345/QĐ-NHNN)
  - BR-NFC-002: Passive Authentication FAIL → reject, hiển thị "Chip CCCD không hợp lệ, vui lòng đến quầy giao dịch"
  - BR-NFC-003: Cross-validation fail (data chip ≠ data OCR) → reject, đưa vào fraud investigation
  - BR-NFC-004: Tối đa 5 lần thử đọc NFC; sau 5 lần fail → chuyển fallback flow
- **Acceptance Criteria**:
  - **AC1**: Given CCCD gắn chip hợp lệ đặt đúng vị trí NFC, When hệ thống đọc chip, Then trích xuất thành công ảnh chân dung + thông tin cá nhân trong <10 giây
  - **AC2**: Given chip CCCD bị tamper (data đã bị sửa), When Passive Authentication, Then trả về INVALID
  - **AC3**: Given data chip: tên "Nguyễn Văn A", data OCR: tên "Nguyễn Văn B", When cross-validation, Then reject và flag fraud investigation
  - **AC4**: Given khách hàng thử đọc NFC 5 lần đều fail (vị trí sai), When hệ thống đếm, Then chuyển fallback flow
- **Error Handling**:
  - NFC không bật trên thiết bị → hiển thị hướng dẫn bật NFC trong Settings
  - Thiết bị không có NFC → hiển thị "Thiết bị không hỗ trợ NFC" và chuyển fallback flow (FR-FALL-001)
  - Kết nối NFC bị ngắt giữa chừng (user nhấc CCCD quá sớm) → hiển thị "Vui lòng giữ CCCD trên điện thoại cho đến khi hoàn tất"
  - Chip CCCD bị hỏng vật lý → hiển thị "Không thể đọc chip, vui lòng đến quầy giao dịch hoặc liên hệ hỗ trợ"

#### FR-NFC-002: Xác minh với CSDL quốc gia dân cư (C06)
- **Mô tả**: Hệ thống NÊN xác minh thông tin CCCD với Cơ sở dữ liệu quốc gia về dân cư (C06) thông qua kết nối VNPT hoặc kênh trực tiếp.
- **Actor**: Hệ thống (tự động)
- **Precondition**: NFC chip reading thành công (FR-NFC-001) hoặc OCR hoàn tất (FR-DOC-003)
- **Input**: Số CCCD, họ tên, ngày sinh, ảnh chân dung
- **Processing**:
  1. Gửi request xác minh đến C06 API (qua VNPT gateway hoặc trực tiếp)
  2. C06 trả về kết quả: MATCH / NO_MATCH / NOT_FOUND
  3. Nếu MATCH → xác nhận danh tính hợp lệ
  4. Nếu NO_MATCH → flag, có thể giấy tờ giả hoặc thông tin đã thay đổi
  5. Nếu NOT_FOUND → có thể CCCD chưa được cập nhật vào hệ thống
- **Output**: Kết quả xác minh C06: MATCH / NO_MATCH / NOT_FOUND / UNAVAILABLE
- **Business Rules**:
  - BR-NFC-005: C06 trả về NO_MATCH → reject eKYC, chuyển manual review
  - BR-NFC-006: C06 trả về NOT_FOUND → cho phép tiếp tục với warning flag, compliance team review sau
  - BR-NFC-007: C06 không khả dụng (UNAVAILABLE) → cho phép tiếp tục eKYC dựa trên NFC + face matching, nhưng flag để re-verify khi C06 khả dụng
- **Acceptance Criteria**:
  - **AC1**: Given CCCD hợp lệ có trong C06, When hệ thống gửi request xác minh, Then C06 trả về MATCH trong <5 giây
  - **AC2**: Given C06 API down, When hệ thống gửi request, Then timeout sau 10 giây, trả về UNAVAILABLE, cho phép tiếp tục eKYC với flag
  - **AC3**: Given C06 trả về NO_MATCH, When hệ thống xử lý, Then reject eKYC và tạo case trong manual review queue
- **Error Handling**:
  - C06 API timeout (>10s) → retry 1 lần, sau đó trả về UNAVAILABLE
  - C06 API rate limit → queue request, retry sau 30 giây
  - C06 API trả về lỗi format → log error, trả về UNAVAILABLE

### 3.5 Module: AML/PEP/Sanctions Screening

#### FR-AML-001: Sàng lọc AML/PEP/Sanctions tại thời điểm onboarding
- **Mô tả**: Hệ thống PHẢI sàng lọc khách hàng qua các danh sách AML/PEP/Sanctions ngay sau khi xác minh danh tính thành công.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Face matching PASS (FR-FACE-001), thông tin cá nhân đã trích xuất (FR-DOC-003)
- **Input**: Họ tên, ngày sinh, quốc tịch, số CCCD
- **Processing**:
  1. Screening qua danh sách PEP (Politically Exposed Persons) — quốc tế + Việt Nam
  2. Screening qua Sanctions lists: UN Security Council, OFAC (US), EU Sanctions
  3. Screening qua Adverse Media: tin tức tiêu cực liên quan tội phạm tài chính, rửa tiền
  4. Screening qua Internal Blacklist của ngân hàng
  5. Fuzzy matching: tìm kiếm gần đúng (Levenshtein distance ≤2) để tránh bỏ sót do sai chính tả, alias
  6. Tính risk score tổng hợp dựa trên kết quả screening
- **Output**: Kết quả: CLEAR / POTENTIAL_MATCH / CONFIRMED_MATCH kèm danh sách matches (nếu có) và risk score
- **Business Rules**:
  - BR-AML-001: CLEAR → tiếp tục tạo tài khoản
  - BR-AML-002: POTENTIAL_MATCH → đưa vào hàng đợi Compliance Officer review; KHÔNG tạo tài khoản cho đến khi review xong
  - BR-AML-003: CONFIRMED_MATCH (sanctions list) → reject ngay lập tức, báo cáo cho Compliance team và cơ quan chức năng (nếu required)
  - BR-AML-004: PEP match → KHÔNG tự động reject; áp dụng Enhanced Due Diligence (EDD), yêu cầu thêm thông tin về nguồn thu nhập, mục đích mở tài khoản
  - BR-AML-005: Danh sách AML/PEP/Sanctions PHẢI được cập nhật ít nhất hàng ngày (theo Luật Phòng chống rửa tiền 2022)
- **Acceptance Criteria**:
  - **AC1**: Given khách hàng tên "Nguyễn Văn A" không có trong bất kỳ danh sách nào, When screening, Then trả về CLEAR trong <3 giây
  - **AC2**: Given khách hàng tên gần giống PEP "Nguyen Van An" (Levenshtein distance = 1), When fuzzy matching, Then trả về POTENTIAL_MATCH và đưa vào review queue
  - **AC3**: Given khách hàng nằm trong OFAC sanctions list, When screening, Then trả về CONFIRMED_MATCH, reject ngay, tạo alert cho Compliance team
  - **AC4**: Given khách hàng là PEP (quan chức cấp tỉnh), When screening, Then trả về POTENTIAL_MATCH với tag "PEP", trigger EDD flow
- **Error Handling**:
  - AML database không khả dụng → KHÔNG cho phép tạo tài khoản, queue request, retry khi database khả dụng
  - Screening timeout (>5s) → retry 2 lần, sau đó đưa vào manual review queue (KHÔNG auto-approve)

#### FR-AML-002: Ongoing AML/PEP Monitoring
- **Mô tả**: Hệ thống PHẢI thực hiện sàng lọc AML/PEP/Sanctions định kỳ cho toàn bộ customer base hiện tại.
- **Actor**: Hệ thống (batch job tự động)
- **Precondition**: Khách hàng đã có tài khoản active
- **Input**: Toàn bộ customer records trong hệ thống
- **Processing**:
  1. Daily batch: re-screen toàn bộ customer base qua danh sách PEP/Sanctions mới cập nhật
  2. Nếu phát hiện match mới → tạo alert cho Compliance team
  3. Weekly batch: adverse media screening
  4. Ghi log kết quả mỗi lần screening
- **Output**: Danh sách new matches (nếu có), alert cho Compliance team
- **Business Rules**:
  - BR-AML-006: Daily screening PHẢI hoàn thành trước 6:00 AM mỗi ngày
  - BR-AML-007: New match → freeze tài khoản tạm thời cho đến khi Compliance review xong (theo Luật Phòng chống rửa tiền 2022)
  - BR-AML-008: Compliance team PHẢI review new match trong vòng 24 giờ làm việc
- **Acceptance Criteria**:
  - **AC1**: Given danh sách sanctions được cập nhật thêm "Nguyễn Văn B", When daily batch chạy, Then phát hiện match với khách hàng "Nguyễn Văn B" và tạo alert
  - **AC2**: Given daily batch bắt đầu lúc 2:00 AM với 500,000 customers, When batch chạy, Then hoàn thành trước 6:00 AM
  - **AC3**: Given new match detected, When hệ thống tạo alert, Then Compliance Officer nhận notification qua email + dashboard trong <5 phút
- **Error Handling**:
  - Batch job fail giữa chừng → resume từ customer cuối cùng đã screen, KHÔNG restart từ đầu
  - AML database down trong giờ batch → retry mỗi 30 phút, escalate nếu down >2 giờ

### 3.6 Module: Account Creation & Risk Scoring

#### FR-ACCT-001: Tạo tài khoản tự động
- **Mô tả**: Hệ thống PHẢI tự động tạo tài khoản ngân hàng cho khách hàng khi toàn bộ quy trình eKYC pass.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Document verification PASS, Liveness PASS, Face matching MATCH, AML screening CLEAR
- **Input**: Thông tin cá nhân từ OCR/NFC, kết quả eKYC (tất cả PASS)
- **Processing**:
  1. Tổng hợp kết quả tất cả bước eKYC → tính overall risk score
  2. Xác định loại tài khoản và hạn mức dựa trên risk score + phương thức xác minh
  3. Gọi Core Banking System API để tạo tài khoản
  4. Gán hạn mức giao dịch ban đầu
  5. Gửi thông báo cho khách hàng (push notification + SMS)
  6. Lưu hồ sơ eKYC vào hệ thống lưu trữ
- **Output**: Tài khoản mới với số tài khoản, hạn mức, trạng thái active
- **Business Rules**:
  - BR-ACCT-001: Nếu eKYC hoàn tất bằng NFC chip + face matching → tài khoản full limit (không giới hạn đặc biệt)
  - BR-ACCT-002: Nếu eKYC hoàn tất KHÔNG có NFC (fallback) → tài khoản giới hạn: giao dịch tối đa 10 triệu VND/lần, 20 triệu VND/ngày (theo QĐ 2345)
  - BR-ACCT-003: Nếu bất kỳ bước nào FLAG (manual review pending) → tài khoản tạo ở trạng thái PENDING, không active cho đến khi review xong
  - BR-ACCT-004: Mỗi số CCCD/CMND chỉ được mở tối đa 1 tài khoản thanh toán tại ngân hàng (theo Thông tư 16/2020)
- **Acceptance Criteria**:
  - **AC1**: Given toàn bộ eKYC PASS (bao gồm NFC), When hệ thống tạo tài khoản, Then tài khoản active với full limit trong <30 giây
  - **AC2**: Given eKYC PASS nhưng không có NFC, When hệ thống tạo tài khoản, Then tài khoản active với giới hạn 10 triệu/lần, 20 triệu/ngày
  - **AC3**: Given khách hàng đã có tài khoản với cùng số CCCD, When hệ thống kiểm tra, Then reject và hiển thị "Số CCCD đã được đăng ký tài khoản"
  - **AC4**: Given tài khoản tạo thành công, When hệ thống gửi thông báo, Then khách hàng nhận push notification + SMS trong <1 phút
- **Error Handling**:
  - Core Banking API fail → retry 3 lần (interval 5s, 15s, 30s), sau đó queue request và thông báo khách hàng "Tài khoản đang được xử lý, chúng tôi sẽ thông báo khi hoàn tất"
  - Duplicate CCCD detected → reject, log event, KHÔNG tạo tài khoản thứ 2

#### FR-ACCT-002: Risk Scoring
- **Mô tả**: Hệ thống PHẢI tính risk score tổng hợp cho mỗi hồ sơ eKYC dựa trên kết quả từng bước xác minh.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Ít nhất 1 bước eKYC đã hoàn thành
- **Input**: Kết quả + confidence score từ tất cả bước eKYC
- **Processing**:
  1. Tính weighted score từ các thành phần:
     - Document verification: weight 15%
     - Liveness detection: weight 20%
     - Face matching (document): weight 15%
     - Face matching (NFC): weight 20%
     - NFC chip authentication: weight 15%
     - AML screening: weight 10%
     - Device integrity: weight 5%
  2. Phân loại risk level: LOW / MEDIUM / HIGH / CRITICAL
  3. Áp dụng risk-based decision
- **Output**: Risk score (0-100), risk level, recommended action
- **Business Rules**:
  - BR-ACCT-005: Risk score 80-100 → LOW risk → auto-approve
  - BR-ACCT-006: Risk score 60-79 → MEDIUM risk → auto-approve với monitoring flag
  - BR-ACCT-007: Risk score 40-59 → HIGH risk → manual review required
  - BR-ACCT-008: Risk score 0-39 → CRITICAL risk → reject, fraud investigation
- **Acceptance Criteria**:
  - **AC1**: Given tất cả bước PASS với confidence >90%, When tính risk score, Then score ≥80 (LOW risk), auto-approve
  - **AC2**: Given liveness confidence 65%, face matching 70%, các bước khác >90%, When tính risk score, Then score trong khoảng 60-79 (MEDIUM risk)
  - **AC3**: Given document verification FAIL, liveness PASS, When tính risk score, Then score <40 (CRITICAL risk), reject
- **Error Handling**:
  - Thiếu kết quả 1 bước (ví dụ NFC skip do thiết bị không hỗ trợ) → tính score với weight còn lại, redistribute weight của bước thiếu cho các bước khác



### 3.7 Module: Continuous KYC (cKYC)

#### FR-CKYC-001: Periodic Re-verification
- **Mô tả**: Hệ thống PHẢI thực hiện xác minh lại danh tính khách hàng định kỳ hoặc khi có sự kiện trigger.
- **Actor**: Hệ thống (tự động trigger), Khách hàng (thực hiện re-verification)
- **Precondition**: Khách hàng có tài khoản active đã qua eKYC
- **Input**: Hồ sơ KYC hiện tại, trigger event
- **Processing**:
  1. Xác định trigger re-verification:
     - Định kỳ: 12 tháng kể từ lần KYC gần nhất (risk LOW), 6 tháng (risk MEDIUM), 3 tháng (risk HIGH)
     - Sự kiện: thay đổi thông tin cá nhân, thay đổi thiết bị đăng nhập, giao dịch bất thường
  2. Gửi notification yêu cầu khách hàng re-verify
  3. Khách hàng thực hiện liveness + face matching (không cần chụp lại giấy tờ nếu CCCD chưa hết hạn)
  4. So sánh kết quả mới với hồ sơ KYC hiện tại
  5. Cập nhật hồ sơ KYC + reset timer
- **Output**: Kết quả re-verification: PASS / FAIL / EXPIRED (khách hàng không thực hiện trong thời hạn)
- **Business Rules**:
  - BR-CKYC-001: Khách hàng có 30 ngày để hoàn thành re-verification kể từ ngày nhận notification
  - BR-CKYC-002: Sau 30 ngày không re-verify → giới hạn tài khoản (chỉ cho phép nhận tiền, không cho chuyển/rút)
  - BR-CKYC-003: Sau 60 ngày không re-verify → freeze tài khoản, yêu cầu đến quầy
  - BR-CKYC-004: Re-verification FAIL → giới hạn tài khoản ngay lập tức, yêu cầu đến quầy
  - BR-CKYC-005: Nếu CCCD hết hạn tại thời điểm re-verify → yêu cầu cập nhật giấy tờ mới (chụp + OCR + NFC)
- **Acceptance Criteria**:
  - **AC1**: Given khách hàng risk LOW, KYC lần cuối 11 tháng trước, When hệ thống kiểm tra, Then gửi notification re-verify trước 30 ngày (tháng thứ 11)
  - **AC2**: Given khách hàng nhận notification re-verify, When khách hàng không thực hiện trong 30 ngày, Then tài khoản bị giới hạn (chỉ nhận tiền)
  - **AC3**: Given khách hàng thực hiện re-verify thành công, When hệ thống cập nhật, Then reset timer và gửi xác nhận "Xác minh danh tính thành công"
- **Error Handling**:
  - Notification delivery fail (push + SMS đều fail) → retry hàng ngày trong 7 ngày, sau đó escalate cho operations team liên hệ trực tiếp

#### FR-CKYC-002: Transaction Monitoring Integration
- **Mô tả**: Hệ thống PHẢI tích hợp với hệ thống giám sát giao dịch để phát hiện hành vi bất thường và trigger re-KYC khi cần.
- **Actor**: Hệ thống (tự động)
- **Precondition**: Khách hàng có tài khoản active, hệ thống transaction monitoring đang hoạt động
- **Input**: Dữ liệu giao dịch real-time từ Core Banking System
- **Processing**:
  1. Nhận transaction events từ CBS qua message queue
  2. Phân tích pattern: so sánh với baseline behavior của khách hàng
  3. Detect anomalies: giao dịch lớn bất thường, tần suất cao bất thường, giao dịch đến/từ quốc gia rủi ro cao
  4. Nếu anomaly score vượt ngưỡng → trigger step-up authentication (liveness + face matching)
  5. Nếu step-up fail → freeze giao dịch, alert Fraud team
- **Output**: Anomaly alerts, step-up authentication triggers, transaction freeze events
- **Business Rules**:
  - BR-CKYC-006: Giao dịch >10 triệu VND → bắt buộc xác thực sinh trắc (theo QĐ 2345)
  - BR-CKYC-007: Tổng giao dịch trong ngày >20 triệu VND → bắt buộc xác thực sinh trắc cho giao dịch tiếp theo
  - BR-CKYC-008: Giao dịch đến/từ quốc gia trong FATF grey list → flag + alert Compliance
  - BR-CKYC-009: >5 giao dịch trong 1 giờ (bất thường so với baseline) → trigger step-up authentication
- **Acceptance Criteria**:
  - **AC1**: Given khách hàng chuyển 15 triệu VND, When hệ thống kiểm tra ngưỡng, Then yêu cầu xác thực sinh trắc trước khi thực hiện giao dịch
  - **AC2**: Given khách hàng đã giao dịch tổng 18 triệu trong ngày, When thực hiện giao dịch 3 triệu tiếp theo (tổng >20 triệu), Then yêu cầu xác thực sinh trắc
  - **AC3**: Given khách hàng thực hiện 6 giao dịch trong 1 giờ (baseline trung bình 2/giờ), When anomaly detection, Then trigger step-up authentication
- **Error Handling**:
  - Message queue lag >30 giây → alert operations team, KHÔNG block giao dịch (tránh ảnh hưởng UX)
  - Step-up authentication timeout (khách hàng không phản hồi trong 5 phút) → cancel giao dịch đang chờ, thông báo "Giao dịch đã bị hủy do không xác thực kịp thời"

#### FR-CKYC-003: Behavioral Biometrics
- **Mô tả**: Hệ thống NÊN thu thập và phân tích behavioral biometrics để phát hiện account takeover.
- **Actor**: Hệ thống (tự động, chạy nền)
- **Precondition**: Khách hàng đang sử dụng mobile app
- **Input**: Dữ liệu tương tác: typing pattern, swipe behavior, hold angle, navigation pattern
- **Processing**:
  1. Thu thập behavioral data trong background (với consent của khách hàng)
  2. Xây dựng behavioral profile cho mỗi khách hàng (sau 7-14 ngày sử dụng)
  3. So sánh real-time behavior với profile → tính deviation score
  4. Nếu deviation score vượt ngưỡng → trigger step-up authentication
- **Output**: Behavioral deviation score, step-up trigger (nếu cần)
- **Business Rules**:
  - BR-CKYC-010: Behavioral biometrics chỉ được thu thập SAU KHI khách hàng đồng ý (opt-in consent) theo NĐ 13/2023
  - BR-CKYC-011: Behavioral data KHÔNG được dùng cho mục đích nào khác ngoài fraud detection
  - BR-CKYC-012: Khách hàng có quyền opt-out bất kỳ lúc nào; khi opt-out → xóa toàn bộ behavioral profile trong 24 giờ
- **Acceptance Criteria**:
  - **AC1**: Given khách hàng đã có behavioral profile, When người khác sử dụng tài khoản (typing pattern khác biệt >70%), Then trigger step-up authentication
  - **AC2**: Given khách hàng opt-out behavioral biometrics, When hệ thống xử lý, Then xóa toàn bộ behavioral profile trong 24 giờ và xác nhận qua notification
- **Error Handling**:
  - Behavioral data collection fail (sensor không khả dụng) → bỏ qua, không ảnh hưởng UX; dựa vào các layer bảo mật khác

### 3.8 Module: Fallback Flows

#### FR-FALL-001: Video Call KYC
- **Mô tả**: Hệ thống PHẢI cung cấp luồng xác minh qua video call với agent khi eKYC tự động fail hoặc không khả dụng.
- **Actor**: Khách hàng, Video Call Agent
- **Precondition**: eKYC tự động fail (vượt số lần retry) HOẶC thiết bị không hỗ trợ (không NFC, camera kém)
- **Input**: Thông tin khách hàng đã thu thập (nếu có), video call stream
- **Processing**:
  1. Khách hàng chọn "Xác minh qua video call" hoặc hệ thống tự chuyển sau khi eKYC fail
  2. Đặt lịch video call (chọn khung giờ khả dụng) hoặc kết nối ngay nếu agent available
  3. Agent thực hiện:
     a. Xác minh khuôn mặt khách hàng qua video
     b. Yêu cầu khách hàng show giấy tờ trước camera
     c. Đặt câu hỏi xác minh (tên, ngày sinh, địa chỉ)
     d. Chụp screenshot làm bằng chứng
  4. Agent quyết định: APPROVE / REJECT / ESCALATE
  5. Ghi log toàn bộ video call (lưu trữ theo quy định)
- **Output**: Kết quả video call KYC: APPROVED / REJECTED / ESCALATED
- **Business Rules**:
  - BR-FALL-001: Video call PHẢI được ghi hình và lưu trữ tối thiểu 5 năm (theo quy định NHNN)
  - BR-FALL-002: Agent PHẢI hoàn thành checklist xác minh trước khi approve
  - BR-FALL-003: Thời gian chờ video call tối đa 15 phút; nếu không có agent → cho phép đặt lịch
  - BR-FALL-004: Video call KYC tạo tài khoản với giới hạn tương tự eKYC không có NFC (BR-ACCT-002)
- **Acceptance Criteria**:
  - **AC1**: Given eKYC tự động fail, When khách hàng chọn video call, Then hệ thống kết nối với agent trong <15 phút hoặc cho phép đặt lịch
  - **AC2**: Given agent approve video call KYC, When hệ thống tạo tài khoản, Then tài khoản active với giới hạn 10 triệu/lần, 20 triệu/ngày
  - **AC3**: Given video call hoàn tất, When hệ thống lưu trữ, Then video được lưu với metadata (agent ID, timestamp, kết quả) và retention 5 năm
- **Error Handling**:
  - Video call bị ngắt giữa chừng → cho phép reconnect trong 5 phút, resume từ bước cuối
  - Không có agent available → hiển thị "Hiện không có nhân viên, vui lòng đặt lịch" + form đặt lịch
  - Chất lượng video quá kém → agent yêu cầu khách hàng đến quầy

#### FR-FALL-002: Đến quầy giao dịch
- **Mô tả**: Hệ thống PHẢI hỗ trợ luồng chuyển tiếp từ eKYC online sang xác minh tại quầy khi tất cả phương thức online fail.
- **Actor**: Khách hàng, Nhân viên quầy
- **Precondition**: eKYC tự động fail VÀ video call KYC fail/không khả dụng
- **Input**: Mã tham chiếu eKYC (reference code)
- **Processing**:
  1. Hệ thống tạo mã tham chiếu (reference code) cho hồ sơ eKYC chưa hoàn tất
  2. Hiển thị mã + danh sách chi nhánh gần nhất (dựa trên GPS)
  3. Khách hàng đến quầy, cung cấp mã tham chiếu
  4. Nhân viên quầy tra cứu hồ sơ, hoàn tất xác minh trực tiếp
  5. Cập nhật trạng thái hồ sơ eKYC → COMPLETED_AT_BRANCH
- **Output**: Mã tham chiếu, danh sách chi nhánh, hồ sơ eKYC cập nhật
- **Business Rules**:
  - BR-FALL-005: Mã tham chiếu có hiệu lực 30 ngày kể từ ngày tạo
  - BR-FALL-006: Sau 30 ngày không đến quầy → hủy hồ sơ, khách hàng phải bắt đầu lại từ đầu
  - BR-FALL-007: Dữ liệu đã thu thập từ eKYC online (ảnh giấy tờ, OCR data) được tái sử dụng tại quầy để giảm thời gian
- **Acceptance Criteria**:
  - **AC1**: Given eKYC online fail, When hệ thống tạo mã tham chiếu, Then hiển thị mã 8 ký tự + QR code + danh sách 3 chi nhánh gần nhất
  - **AC2**: Given nhân viên quầy nhập mã tham chiếu, When hệ thống tra cứu, Then hiển thị hồ sơ eKYC với dữ liệu đã thu thập
  - **AC3**: Given mã tham chiếu đã quá 30 ngày, When nhân viên quầy tra cứu, Then hiển thị "Mã đã hết hạn, vui lòng yêu cầu khách hàng đăng ký lại"
- **Error Handling**:
  - GPS không khả dụng → hiển thị danh sách chi nhánh theo tỉnh/thành phố (chọn thủ công)
  - Mã tham chiếu không tìm thấy → hiển thị "Mã không hợp lệ, vui lòng kiểm tra lại"

### 3.9 Module: Admin Dashboard & Compliance Reporting

#### FR-ADMIN-001: Dashboard giám sát eKYC
- **Mô tả**: Hệ thống PHẢI cung cấp dashboard cho Compliance Officer và Fraud Analyst giám sát hoạt động eKYC real-time.
- **Actor**: Compliance Officer, Fraud Analyst, System Admin
- **Precondition**: User đã đăng nhập Admin Portal với role phù hợp
- **Input**: N/A (dashboard tự động hiển thị data)
- **Processing**:
  1. Hiển thị metrics real-time: số lượng eKYC đang xử lý, tỷ lệ pass/fail, average processing time
  2. Hiển thị queue: hồ sơ chờ manual review (FLAG cases)
  3. Hiển thị alerts: fraud attempts, injection attacks, AML matches
  4. Filter theo thời gian, trạng thái, risk level, phương thức xác minh
  5. Drill-down vào từng hồ sơ eKYC cụ thể
- **Output**: Dashboard với charts, tables, alerts
- **Business Rules**:
  - BR-ADMIN-001: Dashboard data refresh tối đa mỗi 30 giây (near real-time)
  - BR-ADMIN-002: Chỉ Compliance Officer và Fraud Analyst mới xem được chi tiết hồ sơ KYC (PII data)
  - BR-ADMIN-003: System Admin xem được metrics tổng hợp nhưng KHÔNG xem được PII
- **Acceptance Criteria**:
  - **AC1**: Given Compliance Officer đăng nhập, When mở dashboard, Then hiển thị metrics real-time với data delay <30 giây
  - **AC2**: Given có 5 hồ sơ FLAG chờ review, When Compliance Officer mở review queue, Then hiển thị 5 hồ sơ sắp xếp theo thời gian (cũ nhất trước)
  - **AC3**: Given System Admin đăng nhập, When mở dashboard, Then hiển thị metrics tổng hợp nhưng KHÔNG hiển thị họ tên, số CCCD, ảnh khách hàng
- **Error Handling**:
  - Data source không khả dụng → hiển thị "Dữ liệu đang được cập nhật" với timestamp data cuối cùng

#### FR-ADMIN-002: Manual Review Workflow
- **Mô tả**: Hệ thống PHẢI cung cấp workflow cho Compliance Officer review các hồ sơ eKYC bị FLAG.
- **Actor**: Compliance Officer
- **Precondition**: Có hồ sơ eKYC ở trạng thái FLAG (manual review required)
- **Input**: Hồ sơ eKYC bị FLAG
- **Processing**:
  1. Compliance Officer mở hồ sơ từ review queue
  2. Xem chi tiết: ảnh giấy tờ, ảnh selfie, kết quả OCR, confidence scores, AML matches, risk score
  3. So sánh side-by-side: ảnh selfie vs ảnh giấy tờ vs ảnh chip NFC
  4. Quyết định: APPROVE / REJECT / REQUEST_MORE_INFO
  5. Ghi lý do quyết định (bắt buộc)
  6. Hệ thống cập nhật trạng thái hồ sơ và thực hiện action tương ứng
- **Output**: Quyết định review + lý do, cập nhật trạng thái hồ sơ
- **Business Rules**:
  - BR-ADMIN-004: Mỗi quyết định review PHẢI có lý do (free text, tối thiểu 20 ký tự)
  - BR-ADMIN-005: Hồ sơ FLAG PHẢI được review trong vòng 24 giờ làm việc
  - BR-ADMIN-006: Nếu APPROVE → tạo tài khoản (FR-ACCT-001); nếu REJECT → thông báo khách hàng + ghi lý do
  - BR-ADMIN-007: Nếu REQUEST_MORE_INFO → gửi notification cho khách hàng yêu cầu bổ sung thông tin, khách hàng có 7 ngày để phản hồi
  - BR-ADMIN-008: Toàn bộ quyết định review PHẢI được ghi audit log (who, what, when, reason)
- **Acceptance Criteria**:
  - **AC1**: Given Compliance Officer mở hồ sơ FLAG, When xem chi tiết, Then hiển thị side-by-side comparison ảnh selfie / ảnh giấy tờ / ảnh chip NFC
  - **AC2**: Given Compliance Officer chọn REJECT với lý do "Ảnh selfie không khớp giấy tờ", When submit, Then hệ thống gửi notification cho khách hàng và ghi audit log
  - **AC3**: Given Compliance Officer chọn APPROVE, When submit, Then hệ thống tạo tài khoản trong <30 giây và thông báo khách hàng
  - **AC4**: Given Compliance Officer cố submit quyết định không có lý do, When validate, Then hiển thị "Vui lòng nhập lý do quyết định (tối thiểu 20 ký tự)"
- **Error Handling**:
  - Hồ sơ bị 2 Compliance Officer mở cùng lúc → lock hồ sơ cho người mở trước, người sau thấy "Hồ sơ đang được review bởi [tên]"

#### FR-ADMIN-003: Compliance Reporting
- **Mô tả**: Hệ thống PHẢI tạo báo cáo compliance định kỳ và theo yêu cầu.
- **Actor**: Compliance Officer, System Admin
- **Precondition**: Có dữ liệu eKYC trong hệ thống
- **Input**: Tham số báo cáo: khoảng thời gian, loại báo cáo
- **Processing**:
  1. Tổng hợp dữ liệu theo tham số
  2. Generate báo cáo theo template chuẩn
  3. Xuất file (PDF, Excel)
- **Output**: File báo cáo
- **Business Rules**:
  - BR-ADMIN-009: Báo cáo tháng PHẢI tự động generate vào ngày 1 hàng tháng
  - BR-ADMIN-010: Báo cáo PHẢI bao gồm: tổng số eKYC, tỷ lệ pass/fail, số case FLAG, số AML match, average processing time, fraud attempts detected
  - BR-ADMIN-011: Báo cáo cho NHNN PHẢI theo format quy định (nếu có template chuẩn từ NHNN)
- **Acceptance Criteria**:
  - **AC1**: Given ngày 1 hàng tháng, When hệ thống auto-generate báo cáo tháng trước, Then báo cáo sẵn sàng trên dashboard trước 8:00 AM
  - **AC2**: Given Compliance Officer yêu cầu báo cáo custom (01/01 - 15/01), When generate, Then báo cáo hoàn thành trong <2 phút
  - **AC3**: Given báo cáo được generate, When xuất PDF, Then file chứa đầy đủ metrics theo BR-ADMIN-010
- **Error Handling**:
  - Data source không đầy đủ (missing data cho 1 ngày) → generate báo cáo với note "Dữ liệu ngày X không đầy đủ" và highlight



---

## 4. Yêu Cầu Phi Chức Năng (Non-Functional Requirements)

### 4.1 Performance

| ID | Requirement | Target | Ghi chú |
|---|---|---|---|
| NFR-PERF-001 | OCR processing time | <3 giây | Từ lúc chụp đến hiển thị kết quả trích xuất |
| NFR-PERF-002 | Passive liveness check | <3 giây | Phân tích single frame/short video |
| NFR-PERF-003 | Active liveness check | <5 giây | Bao gồm thời gian challenge (không tính thời gian user thực hiện) |
| NFR-PERF-004 | Face matching (1:1) | <2 giây | So sánh selfie vs document/NFC photo |
| NFR-PERF-005 | NFC chip reading | <10 giây | Phụ thuộc thiết bị và vị trí đặt CCCD |
| NFR-PERF-006 | AML screening | <3 giây | Screening qua tất cả danh sách |
| NFR-PERF-007 | End-to-end eKYC flow | <3 phút | Từ bước 1 (chụp giấy tờ) đến account created (happy path) |
| NFR-PERF-008 | Account creation API | <5 giây | Từ lúc gọi CBS API đến nhận response |
| NFR-PERF-009 | Dashboard data refresh | <30 giây | Near real-time cho admin dashboard |
| NFR-PERF-010 | Daily AML batch screening | <4 giờ | Cho 500,000 customers, hoàn thành trước 6:00 AM |
| NFR-PERF-011 | Concurrent eKYC sessions | ≥1,000 | Số session eKYC đồng thời hệ thống phải xử lý được |
| NFR-PERF-012 | Report generation | <2 phút | Cho báo cáo custom bất kỳ khoảng thời gian |

### 4.2 Security

| ID | Requirement | Mô tả |
|---|---|---|
| NFR-SEC-001 | Encryption in transit | TLS 1.3 cho toàn bộ API communication giữa mobile app, backend, và vendor |
| NFR-SEC-002 | Encryption at rest | AES-256 cho toàn bộ dữ liệu KYC lưu trữ (database, file storage) |
| NFR-SEC-003 | Biometric data storage | KHÔNG lưu raw biometric image (ảnh khuôn mặt, vân tay). Chỉ lưu biometric template (irreversible hash). Ảnh gốc PHẢI xóa sau khi tạo template, tối đa trong 24 giờ |
| NFR-SEC-004 | API authentication | OAuth 2.0 + mTLS cho server-to-server communication; JWT cho mobile-to-backend |
| NFR-SEC-005 | Access control | Role-Based Access Control (RBAC) với principle of least privilege. Tối thiểu 4 roles: Customer, Agent, Compliance Officer, System Admin |
| NFR-SEC-006 | Audit logging | Mọi thao tác trên dữ liệu KYC PHẢI được ghi log: who (user/system), what (action), when (timestamp), where (IP/device), result (success/fail). Log KHÔNG được chứa raw biometric data |
| NFR-SEC-007 | Penetration testing | Ít nhất 1 lần/năm bởi bên thứ ba độc lập. Kết quả PHẢI được remediate trong 30 ngày (critical), 90 ngày (high) |
| NFR-SEC-008 | Key management | Encryption keys PHẢI được quản lý bằng HSM (Hardware Security Module) hoặc cloud KMS. Key rotation tối thiểu mỗi 12 tháng |
| NFR-SEC-009 | Session management | eKYC session timeout: 15 phút không hoạt động → session expired, phải bắt đầu lại. Session token PHẢI single-use và bound to device |
| NFR-SEC-010 | Rate limiting | API rate limit: 10 requests/phút/IP cho eKYC endpoints. Brute force protection: lock sau 5 failed attempts trong 15 phút |
| NFR-SEC-011 | Data masking | PII data (số CCCD, họ tên) PHẢI được mask trong logs và non-production environments. Format: 001***890, Nguyễn V** A |
| NFR-SEC-012 | Vulnerability management | Scan dependencies hàng tuần (OWASP Dependency Check hoặc tương đương). Critical vulnerabilities PHẢI patch trong 7 ngày |

### 4.3 Availability & Reliability

| ID | Requirement | Target |
|---|---|---|
| NFR-AVL-001 | System uptime | 99.9% (tối đa ~8.76 giờ downtime/năm) |
| NFR-AVL-002 | Vendor SLA | eKYC vendor API uptime ≥99.9%, response time P95 <3 giây |
| NFR-AVL-003 | Disaster recovery | RPO (Recovery Point Objective) <1 giờ, RTO (Recovery Time Objective) <4 giờ |
| NFR-AVL-004 | Failover | Automatic failover cho database và application servers. Failover time <60 giây |
| NFR-AVL-005 | Graceful degradation | Khi vendor API down → chuyển sang fallback flow (video call/đến quầy), KHÔNG block toàn bộ onboarding |
| NFR-AVL-006 | Backup | Database backup hàng ngày, retention 30 ngày. KYC document backup retention theo quy định (5 năm) |

### 4.4 Compliance & Regulatory

| ID | Requirement | Quy định tham chiếu |
|---|---|---|
| NFR-COM-001 | eKYC phải tuân thủ quy trình mở tài khoản điện tử | Thông tư 16/2020/TT-NHNN (REF-02) |
| NFR-COM-002 | Xác thực sinh trắc bắt buộc cho giao dịch >10 triệu VND/lần hoặc >20 triệu VND/ngày | QĐ 2345/QĐ-NHNN (REF-03) |
| NFR-COM-003 | Từ 01/01/2026: chỉ chấp nhận CCCD gắn chip, CMND còn hạn, hoặc VNeID Level 2 | Thông tư 45 (REF-04) |
| NFR-COM-004 | Dữ liệu cá nhân PHẢI được xử lý và lưu trữ tại Việt Nam | NĐ 13/2023/NĐ-CP (REF-05) |
| NFR-COM-005 | Sàng lọc AML/PEP/Sanctions bắt buộc tại onboarding và ongoing | Luật Phòng chống rửa tiền 2022 (REF-06) |
| NFR-COM-006 | Hồ sơ KYC PHẢI lưu trữ tối thiểu 5 năm kể từ ngày kết thúc quan hệ khách hàng | Luật Phòng chống rửa tiền 2022 (REF-06) |
| NFR-COM-007 | Liveness detection PHẢI đạt tiêu chuẩn ISO/IEC 30107-3 | ISO/IEC 30107-3 (REF-08) |
| NFR-COM-008 | Thu thập behavioral biometrics PHẢI có consent rõ ràng từ khách hàng | NĐ 13/2023/NĐ-CP (REF-05) |
| NFR-COM-009 | Xác thực sinh trắc bắt buộc khi phát hành thẻ mới (từ 2026) | Thông tư 45 (REF-04) |
| NFR-COM-010 | Áp dụng Risk-Based Approach cho CDD: SDD cho rủi ro thấp, EDD cho rủi ro cao | FATF Recommendations (REF-07) |

### 4.5 Usability & Accessibility

| ID | Requirement | Mô tả |
|---|---|---|
| NFR-USA-001 | Hướng dẫn trực quan | Mỗi bước eKYC PHẢI có hướng dẫn bằng animation + text. Không yêu cầu user đọc tài liệu trước |
| NFR-USA-002 | Hỗ trợ người dùng lớn tuổi | Font size tối thiểu 16px cho body text, 20px cho heading. Hỗ trợ tăng font size trong Settings |
| NFR-USA-003 | Audio guidance | Hệ thống NÊN cung cấp hướng dẫn bằng giọng nói (tiếng Việt) cho từng bước eKYC |
| NFR-USA-004 | Điều kiện ánh sáng | Hệ thống PHẢI hoạt động trong điều kiện ánh sáng từ 50 lux đến 10,000 lux. Auto-flash khi ánh sáng <100 lux |
| NFR-USA-005 | Error messages | Mọi error message PHẢI rõ ràng, bằng tiếng Việt, chỉ rõ nguyên nhân và hướng khắc phục. KHÔNG hiển thị technical error codes cho end user |
| NFR-USA-006 | Progress indicator | Hiển thị progress bar/steps indicator cho toàn bộ flow eKYC (ví dụ: "Bước 2/5 — Xác minh khuôn mặt") |
| NFR-USA-007 | Ngôn ngữ | Giao diện chính: tiếng Việt. Hệ thống NÊN hỗ trợ tiếng Anh cho khách hàng nước ngoài |
| NFR-USA-008 | Thời gian hoàn thành | 90% khách hàng PHẢI hoàn thành eKYC trong <5 phút (bao gồm thời gian đọc hướng dẫn) |

### 4.6 Data Retention & Privacy

| ID | Requirement | Mô tả |
|---|---|---|
| NFR-DRP-001 | KYC records retention | Tối thiểu 5 năm kể từ ngày kết thúc quan hệ khách hàng (theo Luật Phòng chống rửa tiền 2022) |
| NFR-DRP-002 | Video call recordings | Lưu trữ tối thiểu 5 năm (theo quy định NHNN) |
| NFR-DRP-003 | Audit logs | Lưu trữ tối thiểu 5 năm |
| NFR-DRP-004 | Raw biometric images | Xóa trong vòng 24 giờ sau khi tạo biometric template. KHÔNG lưu trữ dài hạn |
| NFR-DRP-005 | Behavioral biometric data | Xóa trong vòng 24 giờ khi khách hàng opt-out. Retention tối đa 12 tháng nếu opt-in |
| NFR-DRP-006 | Data subject rights | Khách hàng có quyền: truy cập dữ liệu cá nhân, yêu cầu chỉnh sửa, yêu cầu xóa (trừ dữ liệu bắt buộc lưu theo pháp luật) — theo NĐ 13/2023 |
| NFR-DRP-007 | Data breach notification | Thông báo cho khách hàng bị ảnh hưởng trong vòng 72 giờ kể từ khi phát hiện data breach — theo NĐ 13/2023 |
| NFR-DRP-008 | Cross-border data transfer | KHÔNG chuyển dữ liệu KYC ra ngoài Việt Nam trừ khi có đánh giá tác động và phê duyệt của cơ quan chức năng — theo NĐ 13/2023 |

---

## 5. Giao Diện Hệ Thống (External Interfaces)

### 5.1 User Interfaces

| # | Interface | Mô tả | Platform |
|---|---|---|---|
| UI-001 | eKYC Flow (Mobile) | Giao diện chính cho khách hàng thực hiện eKYC: chụp giấy tờ, selfie, NFC, xác nhận thông tin | iOS 14+, Android 8+ |
| UI-002 | Video Call Screen | Giao diện video call với agent (fallback flow) | iOS, Android |
| UI-003 | Admin Dashboard (Web) | Dashboard giám sát eKYC cho Compliance/Fraud team | Web browser (Chrome, Safari, Edge) |
| UI-004 | Manual Review Screen | Giao diện review hồ sơ FLAG cho Compliance Officer | Web browser |
| UI-005 | Report Viewer | Giao diện xem và xuất báo cáo compliance | Web browser |

### 5.2 Hardware Interfaces

| # | Interface | Mô tả |
|---|---|---|
| HW-001 | Camera (rear) | Chụp giấy tờ tùy thân. Yêu cầu: ≥8MP, autofocus |
| HW-002 | Camera (front) | Chụp selfie cho liveness + face matching. Yêu cầu: ≥5MP |
| HW-003 | NFC reader | Đọc chip CCCD. Yêu cầu: NFC-A (ISO 14443 Type A) |
| HW-004 | Microphone | Thu âm cho lip sync verification (active liveness). Optional |
| HW-005 | Gyroscope/Accelerometer | Sensor correlation cho injection attack detection. Optional |

### 5.3 Software Interfaces (API, SDK, 3rd-party)

| # | Interface | Mô tả | Protocol |
|---|---|---|---|
| SW-001 | eKYC Vendor SDK | SDK tích hợp vào mobile app: OCR, liveness, face matching, NFC | Native SDK (iOS/Android) |
| SW-002 | eKYC Backend API | API giữa mobile app và eKYC backend | REST API, HTTPS (TLS 1.3), JSON |
| SW-003 | Core Banking System API | API tạo tài khoản, kiểm tra duplicate CCCD | REST API hoặc SOAP, mTLS |
| SW-004 | C06 / VNPT API | API xác minh CCCD với CSDL quốc gia dân cư | REST API, mTLS |
| SW-005 | AML/Sanctions API | API sàng lọc PEP/Sanctions (vendor hoặc internal) | REST API, HTTPS |
| SW-006 | Notification Service | Gửi push notification + SMS cho khách hàng | Firebase Cloud Messaging (push), SMS gateway |
| SW-007 | Video Call Service | WebRTC-based video call cho fallback flow | WebRTC, SRTP |
| SW-008 | Fraud Detection System | Nhận signal từ eKYC, gửi risk score | Message queue (Kafka/RabbitMQ) |
| SW-009 | Audit Log Service | Ghi và truy vấn audit logs | REST API hoặc direct database write |

### 5.4 Communication Interfaces

| # | Interface | Mô tả |
|---|---|---|
| COM-001 | HTTPS | Toàn bộ API communication qua HTTPS (TLS 1.3) |
| COM-002 | NFC (ISO 14443) | Giao tiếp giữa điện thoại và chip CCCD |
| COM-003 | WebRTC | Video call real-time cho fallback flow |
| COM-004 | Message Queue | Async communication giữa eKYC backend và Fraud Detection System |
| COM-005 | WebSocket | Real-time updates cho Admin Dashboard |

---

## 6. Luồng Nghiệp Vụ (Business Flows)

### 6.1 Happy Path — eKYC Onboarding (Full NFC)

```
Bước 1: Khách hàng mở app → chọn "Mở tài khoản"
Bước 2: Đồng ý điều khoản + consent thu thập dữ liệu sinh trắc
Bước 3: Chụp mặt trước CCCD → auto-crop, quality check → PASS
Bước 4: Chụp mặt sau CCCD → auto-crop, quality check → PASS
Bước 5: OCR trích xuất thông tin → hiển thị cho khách hàng xác nhận
Bước 6: Document fraud detection → PASS (confidence >85)
Bước 7: Passive liveness → PASS (confidence >80)
Bước 8: Face matching (selfie vs document photo) → MATCH (score ≥75)
Bước 9: NFC chip reading → thành công, Passive Authentication VALID
Bước 10: Face matching (selfie vs NFC photo) → MATCH (score ≥80)
Bước 11: Cross-validation (OCR data vs NFC data) → MATCH
Bước 12: C06 verification → MATCH
Bước 13: AML/PEP screening → CLEAR
Bước 14: Risk scoring → LOW (score ≥80)
Bước 15: Account creation → thành công, full limit
Bước 16: Notification (push + SMS) → "Tài khoản đã được tạo thành công"
```

### 6.2 Alternative Flows

#### AF-001: eKYC không có NFC (thiết bị không hỗ trợ)
```
Bước 1-8: Giống Happy Path (bước 1-8)
Bước 9: Hệ thống detect thiết bị không có NFC → skip NFC
Bước 10: Skip face matching NFC
Bước 11: Skip cross-validation NFC
Bước 12: C06 verification (dựa trên OCR data) → MATCH
Bước 13: AML/PEP screening → CLEAR
Bước 14: Risk scoring → MEDIUM (thiếu NFC verification)
Bước 15: Account creation → thành công, giới hạn 10 triệu/lần, 20 triệu/ngày
Bước 16: Notification → "Tài khoản đã tạo với hạn mức giới hạn. Hoàn tất xác minh NFC để nâng hạn mức"
```

#### AF-002: Passive liveness confidence thấp → Active liveness
```
Bước 1-6: Giống Happy Path
Bước 7: Passive liveness → confidence 65% (trong khoảng 60-80)
Bước 7b: Trigger active liveness → yêu cầu quay đầu + chớp mắt
Bước 7c: Active liveness → PASS
Bước 8-16: Tiếp tục Happy Path
```

#### AF-003: OCR confidence thấp → Manual correction
```
Bước 1-4: Giống Happy Path
Bước 5: OCR trích xuất → field "Họ tên" confidence 82% (<90%)
Bước 5b: Highlight field "Họ tên", hiển thị giá trị OCR đọc được
Bước 5c: Khách hàng xác nhận hoặc sửa thủ công
Bước 6-16: Tiếp tục Happy Path
```

#### AF-004: AML screening → PEP match → EDD
```
Bước 1-12: Giống Happy Path
Bước 13: AML screening → POTENTIAL_MATCH (PEP)
Bước 13b: Trigger EDD flow → yêu cầu thêm thông tin: nguồn thu nhập, mục đích mở tài khoản
Bước 13c: Khách hàng cung cấp thông tin bổ sung
Bước 13d: Đưa vào Compliance review queue
Bước 14: Tài khoản ở trạng thái PENDING cho đến khi Compliance approve
```

#### AF-005: Re-verification (cKYC) trigger
```
Bước 1: Hệ thống detect: KYC đã 12 tháng (risk LOW) → trigger re-verify
Bước 2: Gửi notification: "Vui lòng xác minh lại danh tính"
Bước 3: Khách hàng mở app → thực hiện liveness + face matching
Bước 4: Face matching (selfie mới vs biometric template lưu trữ) → MATCH
Bước 5: Cập nhật hồ sơ KYC, reset timer
Bước 6: Notification: "Xác minh danh tính thành công"
```

### 6.3 Exception Flows

#### EF-001: Document capture fail 3 lần
```
Bước 1-2: Giống Happy Path
Bước 3: Chụp CCCD → fail (blur) → retry 1
Bước 3b: Chụp lại → fail (glare) → retry 2
Bước 3c: Chụp lại → fail (occlusion) → retry 3
Bước 4: Hệ thống: "Không thể chụp giấy tờ. Bạn muốn thử phương thức khác?"
Bước 5: Chuyển sang Video Call KYC (FR-FALL-001) hoặc Đến quầy (FR-FALL-002)
```

#### EF-002: Deepfake / Injection attack detected
```
Bước 1-6: Giống Happy Path
Bước 7: Injection attack detected (virtual camera)
Bước 8: FAIL ngay lập tức, KHÔNG cho retry
Bước 9: Block device fingerprint 24 giờ
Bước 10: Alert Fraud team
Bước 11: Hiển thị: "Xác minh không thành công. Vui lòng liên hệ hotline [số] để được hỗ trợ"
```

#### EF-003: NFC chip authentication FAIL (chip bị tamper)
```
Bước 1-8: Giống Happy Path
Bước 9: NFC chip reading → Passive Authentication INVALID
Bước 10: Reject eKYC ngay lập tức
Bước 11: Flag fraud investigation
Bước 12: Hiển thị: "Chip CCCD không hợp lệ. Vui lòng đến quầy giao dịch gần nhất"
Bước 13: Tạo mã tham chiếu cho đến quầy (FR-FALL-002)
```

#### EF-004: AML sanctions match
```
Bước 1-12: Giống Happy Path
Bước 13: AML screening → CONFIRMED_MATCH (OFAC sanctions list)
Bước 14: Reject ngay lập tức
Bước 15: Alert Compliance team + tạo STR (Suspicious Transaction Report) nếu required
Bước 16: Hiển thị: "Không thể mở tài khoản. Vui lòng liên hệ chi nhánh gần nhất"
```

#### EF-005: Mất kết nối giữa chừng
```
Bước 1-N: Đang thực hiện eKYC
Bước N+1: Mất kết nối internet
Bước N+2: Hiển thị: "Mất kết nối. Dữ liệu đã được lưu tạm"
Bước N+3: Khi có kết nối lại → resume từ bước cuối cùng thành công
Bước N+4: Session vẫn valid trong 15 phút. Sau 15 phút → session expired, bắt đầu lại
```

#### EF-006: App bị kill / crash giữa chừng
```
Bước 1-N: Đang thực hiện eKYC
Bước N+1: App bị kill (user swipe, OS kill, crash)
Bước N+2: Khi mở lại app trong 15 phút → hiển thị "Bạn có muốn tiếp tục xác minh?"
Bước N+3: Nếu "Có" → resume từ bước cuối cùng thành công (dữ liệu đã upload lên server)
Bước N+4: Nếu "Không" hoặc >15 phút → bắt đầu lại từ đầu
```



---

## 7. Data Requirements

### 7.1 Data Model (Entities & Relationships)

```
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│    Customer       │ 1───N │   eKYC_Session    │ 1───1 │    Account       │
│──────────────────│       │──────────────────│       │──────────────────│
│ customer_id (PK) │       │ session_id (PK)  │       │ account_id (PK)  │
│ cccd_number (UQ) │       │ customer_id (FK) │       │ session_id (FK)  │
│ full_name        │       │ status           │       │ account_number   │
│ date_of_birth    │       │ risk_score       │       │ account_type     │
│ gender           │       │ risk_level       │       │ transaction_limit│
│ nationality      │       │ created_at       │       │ daily_limit      │
│ address          │       │ completed_at     │       │ status           │
│ kyc_status       │       │ expires_at       │       │ created_at       │
│ kyc_level        │       │ device_info      │       └──────────────────┘
│ last_kyc_date    │       │ ip_address       │
│ next_kyc_date    │       │ channel          │
│ created_at       │       └──────────────────┘
│ updated_at       │              │
└──────────────────┘              │ 1───N
                           ┌──────────────────┐
                           │ Verification_Step │
                           │──────────────────│
                           │ step_id (PK)     │
                           │ session_id (FK)  │
                           │ step_type        │
                           │ status           │
                           │ confidence_score │
                           │ vendor_ref_id    │
                           │ error_code       │
                           │ error_message    │
                           │ started_at       │
                           │ completed_at     │
                           │ raw_response     │
                           └──────────────────┘

┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│  Document_Record  │       │ Biometric_Template│       │   AML_Screening  │
│──────────────────│       │──────────────────│       │──────────────────│
│ document_id (PK) │       │ template_id (PK) │       │ screening_id (PK)│
│ session_id (FK)  │       │ customer_id (FK) │       │ session_id (FK)  │
│ document_type    │       │ template_type    │       │ customer_id (FK) │
│ document_number  │       │ template_hash    │       │ screening_type   │
│ ocr_data (JSON)  │       │ vendor_ref       │       │ result           │
│ ocr_confidence   │       │ created_at       │       │ matches (JSON)   │
│ fraud_check_result│      │ updated_at       │       │ risk_score       │
│ fraud_confidence │       └──────────────────┘       │ screened_at      │
│ nfc_data (JSON)  │                                   │ lists_version    │
│ nfc_auth_result  │       ┌──────────────────┐       └──────────────────┘
│ c06_result       │       │   Audit_Log      │
│ created_at       │       │──────────────────│
└──────────────────┘       │ log_id (PK)      │
                           │ actor_type       │
┌──────────────────┐       │ actor_id         │
│  Manual_Review    │       │ action           │
│──────────────────│       │ resource_type    │
│ review_id (PK)   │       │ resource_id      │
│ session_id (FK)  │       │ details (JSON)   │
│ reviewer_id (FK) │       │ ip_address       │
│ decision         │       │ device_info      │
│ reason           │       │ timestamp        │
│ reviewed_at      │       └──────────────────┘
│ escalated_from   │
└──────────────────┘
```

### 7.2 Data Dictionary

| Entity | Field | Type | Constraints | Mô tả |
|---|---|---|---|---|
| Customer | customer_id | UUID | PK, NOT NULL | ID nội bộ khách hàng |
| Customer | cccd_number | VARCHAR(12) | UNIQUE, NOT NULL | Số CCCD (12 số) hoặc CMND (9 số) |
| Customer | full_name | NVARCHAR(200) | NOT NULL | Họ tên đầy đủ (Unicode, tiếng Việt có dấu) |
| Customer | date_of_birth | DATE | NOT NULL | Ngày sinh |
| Customer | gender | ENUM | NOT NULL | Nam / Nữ |
| Customer | kyc_status | ENUM | NOT NULL | PENDING / VERIFIED / EXPIRED / SUSPENDED |
| Customer | kyc_level | ENUM | NOT NULL | BASIC (no NFC) / FULL (with NFC) / EDD |
| Customer | last_kyc_date | TIMESTAMP | | Ngày KYC gần nhất |
| Customer | next_kyc_date | TIMESTAMP | | Ngày re-KYC tiếp theo |
| eKYC_Session | session_id | UUID | PK, NOT NULL | ID session eKYC |
| eKYC_Session | status | ENUM | NOT NULL | IN_PROGRESS / COMPLETED / FAILED / EXPIRED / PENDING_REVIEW |
| eKYC_Session | risk_score | INTEGER | 0-100 | Điểm rủi ro tổng hợp |
| eKYC_Session | risk_level | ENUM | | LOW / MEDIUM / HIGH / CRITICAL |
| eKYC_Session | expires_at | TIMESTAMP | NOT NULL | Session hết hạn (created_at + 15 phút) |
| Verification_Step | step_type | ENUM | NOT NULL | DOC_CAPTURE / DOC_FRAUD / OCR / PASSIVE_LIVENESS / ACTIVE_LIVENESS / INJECTION_CHECK / DEVICE_CHECK / FACE_MATCH_DOC / FACE_MATCH_NFC / NFC_READ / C06_VERIFY / AML_SCREEN |
| Verification_Step | status | ENUM | NOT NULL | PENDING / IN_PROGRESS / PASS / FAIL / FLAG / SKIPPED |
| Document_Record | document_type | ENUM | NOT NULL | CCCD / CMND / VNEID |
| Document_Record | nfc_auth_result | ENUM | | VALID / INVALID / NOT_AVAILABLE |
| Document_Record | c06_result | ENUM | | MATCH / NO_MATCH / NOT_FOUND / UNAVAILABLE |
| Biometric_Template | template_type | ENUM | NOT NULL | FACE / FINGERPRINT / BEHAVIORAL |
| Biometric_Template | template_hash | BYTEA | NOT NULL, ENCRYPTED | Biometric template (irreversible hash) |
| AML_Screening | result | ENUM | NOT NULL | CLEAR / POTENTIAL_MATCH / CONFIRMED_MATCH |
| AML_Screening | screening_type | ENUM | NOT NULL | ONBOARDING / DAILY_BATCH / TRIGGERED |
| Manual_Review | decision | ENUM | NOT NULL | APPROVE / REJECT / REQUEST_MORE_INFO / ESCALATE |
| Manual_Review | reason | NVARCHAR(1000) | NOT NULL, MIN 20 chars | Lý do quyết định |
| Audit_Log | actor_type | ENUM | NOT NULL | CUSTOMER / SYSTEM / COMPLIANCE_OFFICER / FRAUD_ANALYST / ADMIN |

### 7.3 Data Migration

| # | Scenario | Mô tả | Approach |
|---|---|---|---|
| DM-001 | KYC truyền thống → eKYC | Khách hàng hiện tại đã KYC tại quầy, cần migrate hồ sơ vào hệ thống eKYC | Batch import hồ sơ KYC hiện tại; khách hàng re-verify bằng eKYC khi đến kỳ re-KYC |
| DM-002 | CMND → CCCD | Khách hàng có hồ sơ KYC bằng CMND cũ, cần cập nhật sang CCCD gắn chip | Trigger re-KYC khi khách hàng đăng nhập; yêu cầu cập nhật CCCD mới. Deadline: 01/01/2026 (Thông tư 45) |
| DM-003 | Vendor migration | Chuyển đổi vendor eKYC (nếu cần) | Biometric templates KHÔNG portable giữa vendors; cần re-enroll khách hàng |

---

## 8. Appendix

### 8.1 Traceability Matrix

| Requirement ID | Analysis Section | Business Flow | Test Scenario |
|---|---|---|---|
| FR-DOC-001 | Section 3.2 — Bước 1: Document Capture | Happy Path bước 3-4 | Chụp CCCD thành công / fail blur / fail glare / fail 3 lần |
| FR-DOC-002 | Section 3.2 — Bước 1: Anti-fraud checks | Happy Path bước 6 | Giấy tờ thật PASS / ảnh màn hình FAIL / ảnh chỉnh sửa FLAG |
| FR-DOC-003 | Section 3.2 — Bước 2: OCR & Data Extraction | Happy Path bước 5 | OCR tiếng Việt có dấu / cross-validation fail / confidence thấp |
| FR-LIVE-001 | Section 3.2 — Bước 3: Passive Liveness | Happy Path bước 7 | Người thật PASS / ảnh in FAIL / video replay FAIL |
| FR-LIVE-002 | Section 3.2 — Bước 3: Active Liveness | AF-002 | Challenge đúng PASS / deepfake FAIL / timeout |
| FR-LIVE-003 | Section 4.2 — Deepfake defense Layer 4 | EF-002 | Virtual camera FAIL / Frida hook FAIL |
| FR-LIVE-004 | Section 4.2 — Deepfake defense Layer 1 | EF-002 | Root FAIL / jailbreak FAIL / emulator FAIL |
| FR-FACE-001 | Section 3.2 — Bước 4: Face Matching | Happy Path bước 8 | Cùng người MATCH / khác người NO_MATCH / ánh sáng yếu |
| FR-FACE-002 | Section 3.2 — Bước 5: NFC + Face Matching | Happy Path bước 10 | NFC photo MATCH / NFC FAIL + doc PASS → FLAG |
| FR-NFC-001 | Section 3.2 — Bước 5: NFC Chip Reading | Happy Path bước 9 | Đọc chip OK / chip tamper INVALID / thiết bị không NFC |
| FR-NFC-002 | Section 3.2 — Bước 5: C06 verification | Happy Path bước 12 | C06 MATCH / NO_MATCH / UNAVAILABLE |
| FR-AML-001 | Section 3.2 — Bước 6: AML/PEP Screening | Happy Path bước 13 | CLEAR / PEP match / sanctions match |
| FR-AML-002 | Section 7 — Continuous KYC | AF-005 | Daily batch / new match detected |
| FR-ACCT-001 | Section 3.2 — Bước 7: Account Creation | Happy Path bước 15 | Full limit / limited / duplicate CCCD |
| FR-ACCT-002 | Section 4.1 — Ma trận rủi ro | Happy Path bước 14 | LOW auto-approve / HIGH manual review / CRITICAL reject |
| FR-CKYC-001 | Section 7 — Continuous KYC | AF-005 | Periodic trigger / event trigger / expired |
| FR-CKYC-002 | Section 7 — Transaction monitoring | N/A | Threshold trigger / anomaly trigger |
| FR-CKYC-003 | Section 7 — Behavioral biometrics | N/A | Account takeover detect / opt-out |
| FR-FALL-001 | Section 3.2 — Fallback | EF-001 | Video call success / no agent / disconnect |
| FR-FALL-002 | Section 3.2 — Fallback | EF-001, EF-003 | Reference code valid / expired |
| FR-ADMIN-001 | Section 9 — KPIs | N/A | Dashboard load / data refresh / role-based access |
| FR-ADMIN-002 | Section 4.1 — R1-R8 | N/A | Review approve / reject / concurrent access |
| FR-ADMIN-003 | Section 9 — KPIs | N/A | Monthly auto-report / custom report / export |

### 8.2 Open Questions

| # | Câu hỏi | Liên quan | Trạng thái |
|---|---|---|---|
| OQ-001 | Ngưỡng confidence score cụ thể cho từng bước (70/85 cho document fraud, 60/80 cho liveness, 75/80 cho face matching) — cần benchmark với vendor để xác nhận phù hợp | FR-DOC-002, FR-LIVE-001, FR-FACE-001 | [TBD — cần confirm với vendor sau POC] |
| OQ-002 | NHNN có template chuẩn cho báo cáo compliance eKYC không? Nếu có → cần cập nhật FR-ADMIN-003 | FR-ADMIN-003, BR-ADMIN-011 | [TBD — cần confirm với Compliance team] |
| OQ-003 | Hạn mức giao dịch cụ thể cho tài khoản eKYC không có NFC — 10 triệu/lần, 20 triệu/ngày là theo QĐ 2345, nhưng ngân hàng có muốn áp dụng hạn mức thấp hơn không? | BR-ACCT-002 | [TBD — cần confirm với Product team] |
| OQ-004 | Video call KYC: ngân hàng tự build hay dùng vendor? Nếu vendor → cần thêm integration requirements | FR-FALL-001 | [TBD — cần confirm với IT team] |
| OQ-005 | Behavioral biometrics: triển khai Phase 3 hay Phase 4? Cần consent flow design riêng | FR-CKYC-003 | [TBD — cần confirm với Legal + Product team] |
| OQ-006 | C06 integration: qua VNPT hay kết nối trực tiếp? Ảnh hưởng đến SLA và chi phí | FR-NFC-002, A2 | [TBD — cần confirm với IT + Procurement] |
| OQ-007 | Weight cho risk scoring (FR-ACCT-002) — cần calibrate dựa trên data thực tế sau Phase 1 | FR-ACCT-002 | [TBD — cần data từ pilot] |
| OQ-008 | Khách hàng doanh nghiệp (từ 01/07/2025 theo QĐ 2345): flow eKYC cho người đại diện pháp luật có khác gì so với cá nhân? | QĐ 2345 | [TBD — cần phân tích thêm] |

### 8.3 Change Log

| Ngày | Phiên bản | Thay đổi | Người thực hiện |
|---|---|---|---|
| 2026-04-22 | 1.0 | Tạo SRS ban đầu từ ekyc-banking-analysis.md. Cover toàn bộ hệ thống eKYC: Document Capture, Liveness, Face Matching, NFC, AML, Account Creation, cKYC, Fallback, Admin. | AI (từ analysis) |

