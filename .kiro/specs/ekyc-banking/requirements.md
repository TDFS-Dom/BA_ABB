# Requirements Document — eKYC (Electronic Know Your Customer)

## Introduction

Tính năng eKYC cho phép khách hàng cá nhân thực hiện xác minh danh tính trực tuyến khi mở tài khoản hoặc nâng cấp dịch vụ ngân hàng, thay thế quy trình KYC truyền thống tại quầy. Hệ thống tích hợp OCR giấy tờ tùy thân, nhận diện khuôn mặt (face matching), phát hiện người thật (liveness detection), và xác minh thông tin qua cơ sở dữ liệu quốc gia — tuân thủ quy định của Ngân hàng Nhà nước Việt Nam (NHNN) theo Thông tư 09/2020/TT-NHNN, Thông tư 16/2020/TT-NHNN, và Nghị định 13/2023/NĐ-CP về bảo vệ dữ liệu cá nhân.

## Glossary

- **eKYC_System**: Hệ thống xác minh danh tính điện tử tổng thể, bao gồm các module OCR, Face Matching, Liveness Detection, và quản lý hồ sơ KYC
- **OCR_Engine**: Module nhận dạng ký tự quang học, trích xuất thông tin từ ảnh chụp giấy tờ tùy thân (CCCD, CMND, Hộ chiếu)
- **Face_Matcher**: Module so khớp khuôn mặt giữa ảnh selfie của khách hàng và ảnh trên giấy tờ tùy thân
- **Liveness_Detector**: Module phát hiện người thật, đảm bảo khách hàng đang thực hiện xác minh trực tiếp (không dùng ảnh tĩnh, video ghi sẵn, hoặc mặt nạ)
- **ID_Document**: Giấy tờ tùy thân hợp lệ do cơ quan nhà nước Việt Nam cấp, bao gồm: Căn cước công dân gắn chip (CCCD), Chứng minh nhân dân (CMND), Hộ chiếu Việt Nam
- **CCCD**: Căn cước công dân gắn chip — giấy tờ tùy thân chính được ưu tiên trong quy trình eKYC
- **NFC_Reader**: Module đọc chip NFC trên CCCD gắn chip để xác minh thông tin điện tử
- **KYC_Profile**: Hồ sơ KYC của khách hàng, bao gồm thông tin cá nhân, ảnh giấy tờ, ảnh selfie, kết quả xác minh, và trạng thái phê duyệt
- **Verification_Score**: Điểm tin cậy (0-100) thể hiện mức độ khớp giữa thông tin khai báo và thông tin xác minh
- **Customer**: Khách hàng cá nhân thực hiện quy trình eKYC
- **Reviewer**: Nhân viên ngân hàng có quyền xem xét và phê duyệt hồ sơ eKYC
- **Audit_Logger**: Module ghi nhận toàn bộ hành động trong quy trình eKYC phục vụ kiểm toán
- **NHNN**: Ngân hàng Nhà nước Việt Nam — cơ quan quản lý ngành ngân hàng
- **Consent_Manager**: Module quản lý sự đồng ý của khách hàng về thu thập và xử lý dữ liệu cá nhân theo Nghị định 13/2023/NĐ-CP

## Requirements

### Requirement 1: Khởi tạo quy trình eKYC

**User Story:** As a Customer, I want to initiate the eKYC process from the mobile or web banking channel, so that I can verify my identity remotely without visiting a bank branch.

#### Acceptance Criteria

1. WHEN the Customer selects "Mở tài khoản" or "Xác minh danh tính", THE eKYC_System SHALL display the eKYC consent screen with clear description of data to be collected, purpose of processing, and data retention period
2. WHEN the Customer provides consent, THE Consent_Manager SHALL record the consent timestamp, consent version, Customer identifier, and IP address
3. IF the Customer declines consent, THEN THE eKYC_System SHALL terminate the eKYC process and display a message explaining that eKYC cannot proceed without consent
4. THE eKYC_System SHALL support initiation from Mobile Banking (iOS, Android) and Web Portal channels
5. WHEN the Customer initiates eKYC, THE eKYC_System SHALL generate a unique session identifier with a maximum validity of 30 minutes

### Requirement 2: Chụp và xử lý giấy tờ tùy thân (ID Document Capture)

**User Story:** As a Customer, I want to capture images of my identity document, so that the system can extract my personal information automatically.

#### Acceptance Criteria

1. THE eKYC_System SHALL accept the following ID_Document types: CCCD (Căn cước công dân gắn chip), CMND (Chứng minh nhân dân), and Hộ chiếu Việt Nam
2. WHEN the Customer captures the front side of the ID_Document, THE eKYC_System SHALL validate image quality including minimum resolution of 1280x720 pixels, no blur detection, adequate lighting, and complete document visibility
3. WHEN the Customer captures the back side of the ID_Document, THE eKYC_System SHALL validate image quality using the same criteria as the front side
4. IF the captured image fails quality validation, THEN THE eKYC_System SHALL display a specific error message indicating the quality issue (blur, low light, incomplete document, or glare) and prompt the Customer to recapture
5. WHEN both sides of the ID_Document pass quality validation, THE OCR_Engine SHALL extract the following fields: full name, date of birth, gender, ID number, issue date, expiry date, place of origin, and permanent address
6. THE OCR_Engine SHALL achieve a minimum accuracy rate of 95% for Vietnamese text extraction on CCCD documents
7. WHEN the OCR_Engine completes extraction, THE eKYC_System SHALL display the extracted information and allow the Customer to review and correct any field before proceeding

### Requirement 3: Đọc chip NFC trên CCCD

**User Story:** As a Customer with a chip-enabled CCCD, I want the system to read my CCCD chip data via NFC, so that my identity information can be verified with higher accuracy and security.

#### Acceptance Criteria

1. WHERE the Customer device supports NFC and the ID_Document is a CCCD, THE NFC_Reader SHALL attempt to read the electronic data from the CCCD chip
2. WHEN the NFC_Reader successfully reads the CCCD chip, THE eKYC_System SHALL extract: full name, date of birth, gender, ID number, portrait photo, fingerprint template, and digital signature from the chip
3. WHEN NFC chip data is available, THE eKYC_System SHALL cross-validate chip data against OCR-extracted data and flag any discrepancies
4. IF the NFC reading fails after 3 attempts, THEN THE eKYC_System SHALL allow the Customer to proceed with OCR-only verification and record the NFC failure reason in the KYC_Profile
5. THE NFC_Reader SHALL complete the chip reading process within 15 seconds per attempt

### Requirement 4: Xác minh khuôn mặt (Face Matching)

**User Story:** As a Customer, I want to take a selfie for face verification, so that the system can confirm that I am the person shown on the identity document.

#### Acceptance Criteria

1. WHEN the Customer reaches the face verification step, THE eKYC_System SHALL activate the front-facing camera and display a face alignment guide overlay
2. WHEN the Customer captures a selfie, THE Face_Matcher SHALL compare the selfie against the portrait photo extracted from the ID_Document (OCR or NFC chip)
3. THE Face_Matcher SHALL produce a Verification_Score between 0 and 100 for each face comparison
4. WHEN the Verification_Score is equal to or greater than 80, THE Face_Matcher SHALL mark the face verification as PASSED
5. WHEN the Verification_Score is less than 80, THE Face_Matcher SHALL mark the face verification as FAILED and allow the Customer to retry up to 3 times
6. IF the Customer fails face verification after 3 retries, THEN THE eKYC_System SHALL suspend the eKYC session and flag the KYC_Profile for manual review by a Reviewer
7. THE Face_Matcher SHALL reject selfie images that contain multiple faces, obscured faces, or sunglasses

### Requirement 5: Phát hiện người thật (Liveness Detection)

**User Story:** As a bank security officer, I want the system to verify that the person performing eKYC is physically present, so that identity fraud using photos, videos, or masks is prevented.

#### Acceptance Criteria

1. WHEN the Customer reaches the liveness verification step, THE Liveness_Detector SHALL require the Customer to perform at least 2 random actions from the following set: blink eyes, turn head left, turn head right, smile, and nod
2. THE Liveness_Detector SHALL complete the liveness challenge within a maximum duration of 30 seconds
3. WHEN the Customer successfully completes all required actions, THE Liveness_Detector SHALL mark liveness verification as PASSED
4. IF the Customer fails to complete the required actions within 30 seconds, THEN THE Liveness_Detector SHALL mark the attempt as FAILED and allow the Customer to retry up to 3 times
5. THE Liveness_Detector SHALL detect and reject presentation attacks including printed photos, screen replay attacks, video replay attacks, and 3D masks with a minimum detection accuracy of 99%
6. IF the Customer fails liveness detection after 3 retries, THEN THE eKYC_System SHALL suspend the eKYC session and flag the KYC_Profile for manual review by a Reviewer

### Requirement 6: Xác minh thông tin qua cơ sở dữ liệu (Data Verification)

**User Story:** As a compliance officer, I want the system to cross-check customer information against national databases, so that the bank can verify the authenticity of the customer's identity.

#### Acceptance Criteria

1. WHEN all biometric verifications pass (face matching and liveness detection), THE eKYC_System SHALL submit the extracted ID information for verification against the National Population Database (Cơ sở dữ liệu quốc gia về dân cư)
2. WHEN the national database verification returns a match, THE eKYC_System SHALL record the verification result and timestamp in the KYC_Profile
3. IF the national database verification returns a mismatch or is unavailable, THEN THE eKYC_System SHALL flag the KYC_Profile for manual review and notify the Reviewer
4. THE eKYC_System SHALL also verify the ID_Document number against the NHNN blacklist database for flagged or stolen documents
5. WHEN the ID_Document number appears on the blacklist, THE eKYC_System SHALL immediately reject the eKYC application and generate a security alert to the Reviewer

### Requirement 7: Tính điểm và quyết định KYC (Scoring and Decision)

**User Story:** As a bank operations manager, I want the system to automatically score and decide on eKYC applications, so that low-risk applications are approved instantly while high-risk ones are escalated for review.

#### Acceptance Criteria

1. WHEN all verification steps are completed, THE eKYC_System SHALL calculate a composite Verification_Score based on: OCR confidence (weight 20%), NFC verification result (weight 15%), face matching score (weight 30%), liveness detection result (weight 20%), and national database verification result (weight 15%)
2. WHEN the composite Verification_Score is equal to or greater than 85, THE eKYC_System SHALL automatically approve the KYC_Profile with status APPROVED
3. WHEN the composite Verification_Score is between 60 and 84 (inclusive), THE eKYC_System SHALL set the KYC_Profile status to PENDING_REVIEW and assign it to a Reviewer
4. WHEN the composite Verification_Score is less than 60, THE eKYC_System SHALL automatically reject the KYC_Profile with status REJECTED and notify the Customer with the rejection reason
5. THE eKYC_System SHALL complete the automated scoring and decision process within 5 seconds after all verification results are available

### Requirement 8: Xem xét thủ công (Manual Review)

**User Story:** As a Reviewer, I want to review flagged eKYC applications with all supporting evidence, so that I can make informed approval or rejection decisions.

#### Acceptance Criteria

1. WHEN a KYC_Profile is assigned for manual review, THE eKYC_System SHALL display to the Reviewer: Customer selfie, ID_Document images (front and back), OCR-extracted data, NFC chip data (if available), face matching score, liveness detection result, national database verification result, and composite Verification_Score
2. THE Reviewer SHALL be able to approve, reject, or request additional information for each KYC_Profile under review
3. WHEN the Reviewer approves a KYC_Profile, THE eKYC_System SHALL update the KYC_Profile status to APPROVED and notify the Customer
4. WHEN the Reviewer rejects a KYC_Profile, THE eKYC_System SHALL require the Reviewer to provide a rejection reason and notify the Customer with the reason
5. WHEN the Reviewer requests additional information, THE eKYC_System SHALL notify the Customer with specific instructions on what additional information or documents are needed
6. THE eKYC_System SHALL enforce a maximum review turnaround time of 24 hours for PENDING_REVIEW profiles and escalate overdue profiles to a senior Reviewer

### Requirement 9: Bảo vệ dữ liệu cá nhân (Personal Data Protection)

**User Story:** As a data protection officer, I want the eKYC system to comply with Vietnamese personal data protection regulations, so that customer biometric and identity data is handled securely and lawfully.

#### Acceptance Criteria

1. THE eKYC_System SHALL encrypt all biometric data (facial images, fingerprint templates) and ID_Document images using AES-256 encryption at rest and TLS 1.2 or higher in transit
2. THE Consent_Manager SHALL obtain explicit consent from the Customer before collecting each category of personal data: identity information, biometric data (facial image), and ID_Document images
3. WHEN the Customer requests data deletion, THE eKYC_System SHALL delete all biometric data within 30 days, except data required for regulatory retention under banking laws
4. THE eKYC_System SHALL retain KYC_Profile data for a minimum of 10 years from the date of account closure, as required by Vietnamese banking regulations
5. THE eKYC_System SHALL implement role-based access control ensuring that only authorized Reviewers and compliance officers can access Customer biometric data
6. THE Audit_Logger SHALL record every access to Customer biometric data including accessor identity, timestamp, accessed data fields, and access purpose
7. IF a data breach involving biometric data is detected, THEN THE eKYC_System SHALL notify the data protection authority within 72 hours as required by Nghị định 13/2023/NĐ-CP

### Requirement 10: Ghi nhận kiểm toán (Audit Trail)

**User Story:** As an internal auditor, I want a complete audit trail of all eKYC activities, so that I can verify compliance with NHNN regulations and internal policies.

#### Acceptance Criteria

1. THE Audit_Logger SHALL record every step of the eKYC process including: session initiation, consent recording, document capture, OCR processing, NFC reading, face matching, liveness detection, database verification, scoring, decision, and manual review actions
2. THE Audit_Logger SHALL capture the following metadata for each event: event timestamp (UTC+7), event type, actor (Customer or Reviewer), session identifier, IP address, device information, and result status
3. THE eKYC_System SHALL store audit logs for a minimum of 10 years in compliance with Vietnamese banking regulations
4. THE Audit_Logger SHALL ensure audit log integrity using tamper-evident mechanisms (cryptographic hashing of log entries)
5. THE eKYC_System SHALL provide audit log search and export functionality for authorized compliance officers and internal auditors

### Requirement 11: Xử lý lỗi và khả năng phục hồi (Error Handling and Recovery)

**User Story:** As a Customer, I want the eKYC process to handle errors gracefully and allow me to resume, so that I do not lose my progress due to technical issues.

#### Acceptance Criteria

1. IF the Customer loses network connectivity during the eKYC process, THEN THE eKYC_System SHALL save the current progress and allow the Customer to resume from the last completed step within the session validity period of 30 minutes
2. IF an external service (OCR, Face Matching, NFC, or National Database) is unavailable, THEN THE eKYC_System SHALL retry the request up to 3 times with exponential backoff and display a user-friendly error message if all retries fail
3. IF the eKYC session expires (exceeds 30 minutes), THEN THE eKYC_System SHALL notify the Customer and require a new session to be initiated while preserving previously uploaded documents for 24 hours
4. WHEN an unrecoverable error occurs, THE eKYC_System SHALL log the error details, notify the Customer with a reference number, and create a support ticket for the operations team
5. THE eKYC_System SHALL maintain a service health dashboard displaying the availability status of all dependent services (OCR, Face Matching, Liveness Detection, NFC, National Database)

### Requirement 12: Hiệu năng và khả năng mở rộng (Performance and Scalability)

**User Story:** As a system architect, I want the eKYC system to handle peak loads efficiently, so that customers experience minimal wait times during high-traffic periods.

#### Acceptance Criteria

1. THE eKYC_System SHALL process a complete eKYC flow (from document capture to decision) within 3 minutes under normal load conditions
2. THE eKYC_System SHALL support a minimum of 500 concurrent eKYC sessions during peak hours
3. THE OCR_Engine SHALL complete document text extraction within 5 seconds per document image
4. THE Face_Matcher SHALL complete face comparison within 3 seconds per comparison
5. THE Liveness_Detector SHALL process liveness challenge verification within 2 seconds after the Customer completes the required actions
6. THE eKYC_System SHALL maintain 99.5% uptime availability measured on a monthly basis
