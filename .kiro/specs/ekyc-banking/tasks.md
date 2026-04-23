# Implementation Plan: eKYC (Electronic Know Your Customer)

## Overview

Triển khai hệ thống eKYC theo kiến trúc microservices với TypeScript, bao gồm 10 core services, cross-cutting concerns (audit, encryption, consent), và tích hợp external services. Mỗi task xây dựng incremental trên các task trước, kết thúc bằng wiring toàn bộ hệ thống.

## Tasks

- [ ] 1. Set up project structure, shared types, and database schema
  - [ ] 1.1 Initialize TypeScript project with monorepo structure
    - Create directory structure for each service module (session, document, nfc, face-matching, liveness, data-verification, scoring, manual-review, audit, consent)
    - Configure TypeScript, ESLint, Prettier, and fast-check for property-based testing
    - Set up shared types package for cross-service interfaces
    - _Requirements: 1.4, 12.6_

  - [ ] 1.2 Define all shared TypeScript interfaces and types
    - Implement `EKYCSession`, `EKYCStep`, `CreateSessionRequest`, `DeviceInfo` interfaces
    - Implement `ConsentRecord`, `ConsentRequest`, `DataCategory` types
    - Implement `DocumentData`, `OCRResult`, `ImageQualityResult`, `ImageQualityIssue` types
    - Implement `NFCChipData`, `NFCReadResult`, `NFCReadRequest` types
    - Implement `FaceCompareRequest`, `FaceCompareResult`, `SelfieValidationResult`, `SelfieIssue` types
    - Implement `LivenessChallenge`, `LivenessAction`, `LivenessResult`, `LivenessVerifyRequest` types
    - Implement `NationalDBRequest`, `NationalDBResult`, `BlacklistResult` types
    - Implement `ScoreRequest`, `ScoreResult` types
    - Implement `ReviewItem`, `ReviewDetail`, `EscalationResult` types
    - Implement `AuditEvent`, `AuditEventType`, `AuditSearchQuery` types
    - Implement `EncryptedData`, `EncryptedImageRef` utility types
    - _Requirements: 1.1–1.5, 2.1–2.7, 3.1–3.5, 4.1–4.7, 5.1–5.6, 6.1–6.5, 7.1–7.5, 8.1–8.6, 9.1–9.7, 10.1–10.5_

  - [ ] 1.3 Create database schema and migration files
    - Create PostgreSQL migration for all 10 tables: `kyc_profile`, `ekyc_session`, `consent_record`, `document_record`, `face_match_record`, `liveness_record`, `verification_record`, `score_record`, `review_record`, `audit_log`
    - Define indexes, foreign keys, and constraints per ER diagram
    - Configure encrypted JSON columns for `ocr_data`, `nfc_data`
    - Set up Redis schema for session cache with 30-minute TTL
    - _Requirements: 9.1, 9.4, 10.3_

- [ ] 2. Implement Session Service and Consent Manager
  - [ ] 2.1 Implement Session Service
    - Implement `createSession()` — generate unique UUID session ID, set expiresAt = createdAt + 30 minutes, validate channel and device info
    - Implement `getSession()` — retrieve session from Redis cache with PostgreSQL fallback
    - Implement `updateProgress()` — update currentStep and completedSteps
    - Implement `resumeSession()` — restore session state from Redis, position at next uncompleted step
    - Implement `cancelSession()` — mark session CANCELLED with reason
    - Implement session expiry logic — auto-expire after 30 minutes, preserve documents for 24 hours
    - _Requirements: 1.1, 1.4, 1.5, 11.1, 11.3_

  - [ ]* 2.2 Write property test: Session creation invariants
    - **Property 2: Session creation invariants** — For any two session creation requests, generated session IDs are unique; expiresAt = createdAt + 30 minutes exactly
    - **Validates: Requirements 1.5**

  - [ ]* 2.3 Write property test: Session resume preserves progress
    - **Property 22: Session resume preserves progress** — For any active session with completed steps, resuming restores exact same completed steps and positions at next uncompleted step
    - **Validates: Requirements 11.1**

  - [ ] 2.4 Implement Consent Manager
    - Implement `recordConsent()` — record consent with timestamp, version, customerId, IP address; validate all DataCategory values
    - Implement `validateConsent()` — check consent exists and is not revoked for given session and data category
    - Implement `revokeConsent()` — mark consent as revoked with timestamp
    - Implement `getConsentHistory()` — retrieve all consent records for a customer
    - _Requirements: 1.2, 1.3, 9.2, 9.3_

  - [ ]* 2.5 Write property test: Consent recording completeness
    - **Property 1: Consent recording completeness** — For any valid consent request, the resulting ConsentRecord contains all four fields (customerId, sessionId, consentVersion, ipAddress) non-null, timestamp within 1 second of request
    - **Validates: Requirements 1.2**

  - [ ]* 2.6 Write property test: Consent precondition enforcement
    - **Property 18: Consent precondition enforcement** — Data collection operations succeed only if valid non-revoked consent exists for that customer and data category; operations without matching consent are blocked
    - **Validates: Requirements 9.2**

  - [ ]* 2.7 Write unit tests for Session Service and Consent Manager
    - Test consent screen display contains required content (Req 1.1)
    - Test consent decline terminates session (Req 1.3)
    - Test session creation works for iOS, Android, Web channels (Req 1.4)
    - Test session expiry handling — notification + document preservation 24h (Req 11.3)
    - _Requirements: 1.1, 1.3, 1.4, 11.3_

- [ ] 3. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 4. Implement Document Service (OCR) and NFC Service
  - [ ] 4.1 Implement Document Service — image quality validation
    - Implement `validateImageQuality()` — check resolution >= 1280x720, blur detection, lighting adequacy, document completeness
    - Return specific `ImageQualityIssue` types (BLUR, LOW_LIGHT, INCOMPLETE_DOCUMENT, GLARE) with distinct error messages
    - Apply identical validation for front and back document images
    - _Requirements: 2.2, 2.3, 2.4_

  - [ ]* 4.2 Write property test: Image quality validation
    - **Property 3: Image quality validation** — For any image metadata, validator accepts if and only if resolution >= 1280x720, blur below threshold, lighting adequate, document fully visible; applies identically to front and back
    - **Validates: Requirements 2.2, 2.3**

  - [ ]* 4.3 Write property test: Quality issue to error message mapping
    - **Property 4: Quality issue to error message mapping** — For any image failing quality validation, error message identifies the specific issue type; each distinct issue type maps to a distinct non-empty error message
    - **Validates: Requirements 2.4**

  - [ ] 4.4 Implement Document Service — OCR extraction
    - Implement `extractDocumentData()` — extract fullName, dateOfBirth, gender, idNumber, issueDate, expiryDate, placeOfOrigin, permanentAddress from CCCD/CMND/Passport
    - Implement `confirmExtractedData()` — allow customer to review and correct OCR fields
    - Implement `crossValidate()` — compare OCR data vs NFC data, flag all mismatched fields
    - Store document images encrypted in S3 with encrypted pointer in DB
    - _Requirements: 2.1, 2.5, 2.6, 2.7, 3.3_

  - [ ]* 4.5 Write property test: OCR extraction completeness
    - **Property 5: OCR extraction completeness** — For any valid document image pair passing quality validation, OCR result contains all required fields non-empty
    - **Validates: Requirements 2.5**

  - [ ]* 4.6 Write property test: OCR vs NFC cross-validation
    - **Property 8: OCR vs NFC cross-validation** — For any pair of OCR and NFC data, cross-validation identifies every field where two sources differ; flagged discrepancies equal exactly the set of non-matching fields
    - **Validates: Requirements 3.3**

  - [ ] 4.7 Implement NFC Service
    - Implement `readChipData()` — read NFC chip data with max 3 attempts, 15-second timeout per attempt
    - Extract fullName, dateOfBirth, gender, idNumber, portraitPhoto, fingerprintTemplate, digitalSignature
    - Handle failure reasons: TIMEOUT, CHIP_ERROR, DEVICE_NOT_SUPPORTED, AUTHENTICATION_FAILED
    - Implement fallback to OCR-only after 3 NFC failures
    - Encrypt fingerprint template data always
    - _Requirements: 3.1, 3.2, 3.4, 3.5_

  - [ ]* 4.8 Write property test: NFC trigger condition
    - **Property 6: NFC trigger condition** — NFC reading attempted if and only if deviceSupportsNFC is true AND documentType is 'CCCD'
    - **Validates: Requirements 3.1**

  - [ ]* 4.9 Write property test: NFC extraction completeness
    - **Property 7: NFC extraction completeness** — For any successful NFC read, result contains all required fields non-null
    - **Validates: Requirements 3.2**

  - [ ]* 4.10 Write unit tests for Document Service and NFC Service
    - Test document type acceptance — CCCD, CMND, Passport (Req 2.1)
    - Test OCR review step — after OCR, session moves to review (Req 2.7)
    - Test NFC fallback after 3 failures — proceeds with OCR-only (Req 3.4)
    - _Requirements: 2.1, 2.7, 3.4_

- [ ] 5. Implement Face Matching and Liveness Detection Services
  - [ ] 5.1 Implement Face Matching Service
    - Implement `validateSelfie()` — reject images with multiple faces, obscured face, or sunglasses; return specific SelfieIssue
    - Implement `compareFaces()` — compare selfie vs reference image (ID document or NFC chip portrait), produce score 0-100
    - Set passed = true if score >= 80, false otherwise
    - Track attempt numbers 1-3; after 3 failures, flag for manual review
    - Enforce processing time < 3000ms
    - _Requirements: 4.1, 4.2, 4.3, 4.4, 4.5, 4.6, 4.7_

  - [ ]* 5.2 Write property test: Face match threshold and attempt tracking
    - **Property 9: Face match threshold and attempt tracking** — For any score in [0,100], passed is true iff score >= 80; attempt numbers increment correctly from 1 to max 3
    - **Validates: Requirements 4.3, 4.4, 4.5**

  - [ ]* 5.3 Write property test: Selfie validation rejection
    - **Property 10: Selfie validation rejection** — For any selfie with multiple faces, obscured face, or sunglasses, validator rejects and returns specific issue type
    - **Validates: Requirements 4.7**

  - [ ] 5.4 Implement Liveness Detection Service
    - Implement `createChallenge()` — generate exactly 2 random actions from {BLINK, TURN_LEFT, TURN_RIGHT, SMILE, NOD}, max duration 30 seconds
    - Implement `verifyLiveness()` — verify all required actions completed within time limit
    - Detect presentation attacks: printed photos, screen replay, video replay, 3D masks (>= 99% accuracy)
    - Track attempt numbers 1-3; after 3 failures, flag for manual review
    - Enforce processing time < 2000ms
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5, 5.6_

  - [ ]* 5.5 Write property test: Liveness challenge generation invariant
    - **Property 11: Liveness challenge generation invariant** — Generated challenge contains exactly 2 actions, each from valid set {BLINK, TURN_LEFT, TURN_RIGHT, SMILE, NOD}
    - **Validates: Requirements 5.1**

  - [ ]* 5.6 Write property test: Liveness completion implies PASSED
    - **Property 12: Liveness completion implies PASSED** — When all required challenge actions completed within time limit, result is PASSED
    - **Validates: Requirements 5.3**

  - [ ]* 5.7 Write unit tests for Face Matching and Liveness Detection
    - Test camera activation with guide overlay (Req 4.1)
    - Test face match escalation — 3 failures → manual review (Req 4.6)
    - Test liveness timeout retry — timeout → FAILED + retry allowed (Req 5.4)
    - Test liveness escalation — 3 failures → manual review (Req 5.6)
    - _Requirements: 4.1, 4.6, 5.4, 5.6_

- [ ] 6. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 7. Implement Data Verification and Scoring Engine
  - [ ] 7.1 Implement Data Verification Service
    - Implement `verifyAgainstNationalDB()` — submit ID info for verification, handle match/mismatch/unavailable responses
    - Implement `checkBlacklist()` — check ID number against NHNN blacklist; if blacklisted, immediately reject and generate security alert
    - Handle external service unavailability — flag for manual review when National DB is down
    - Record verification results and timestamps in KYC_Profile
    - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ]* 7.2 Write unit tests for Data Verification Service
    - Test national DB match recording — result + timestamp recorded (Req 6.2)
    - Test national DB mismatch/unavailable → manual review (Req 6.3)
    - Test blacklist rejection — immediate rejection + security alert (Req 6.5)
    - _Requirements: 6.2, 6.3, 6.5_

  - [ ] 7.3 Implement Scoring Engine
    - Implement `calculateScore()` — compute composite score with weights: OCR confidence 20%, NFC verification 15%, face match score 30%, liveness result 20%, national DB verification 15%
    - Convert boolean results to 0 or 100 for scoring
    - Handle null NFC (redistribute weight when NFC not available)
    - Map score to decision: APPROVED (>= 85), PENDING_REVIEW (60-84), REJECTED (< 60)
    - Enforce processing time < 5000ms
    - _Requirements: 7.1, 7.2, 7.3, 7.4, 7.5_

  - [ ]* 7.4 Write property test: Composite score weighted calculation
    - **Property 13: Composite score weighted calculation** — Composite score equals ocrConfidence × 0.20 + nfcScore × 0.15 + faceMatchScore × 0.30 + livenessScore × 0.20 + nationalDBScore × 0.15, booleans converted to 0 or 100
    - **Validates: Requirements 7.1**

  - [ ]* 7.5 Write property test: Score-to-decision threshold mapping
    - **Property 14: Score-to-decision threshold mapping** — APPROVED if score >= 85, PENDING_REVIEW if 60 <= score < 85, REJECTED if score < 60; ranges exhaustive and mutually exclusive
    - **Validates: Requirements 7.2, 7.3, 7.4**

- [ ] 8. Implement Manual Review Service
  - [ ] 8.1 Implement Manual Review Service
    - Implement `getPendingReviews()` — list profiles assigned to reviewer
    - Implement `getReviewDetail()` — return all required fields: selfie, ID images, OCR data, NFC data, face match score, liveness result, national DB result, composite score
    - Implement `approveProfile()` — update status to APPROVED, notify customer
    - Implement `rejectProfile()` — require non-empty rejection reason, update status to REJECTED, notify customer with reason
    - Implement `requestAdditionalInfo()` — notify customer with specific instructions
    - Implement `escalateOverdue()` — detect reviews past 24-hour deadline, escalate to senior reviewer
    - _Requirements: 8.1, 8.2, 8.3, 8.4, 8.5, 8.6_

  - [ ]* 8.2 Write property test: Review detail completeness
    - **Property 15: Review detail completeness** — For any profile assigned for review, ReviewDetail contains all required fields non-null
    - **Validates: Requirements 8.1**

  - [ ]* 8.3 Write property test: Rejection requires non-empty reason
    - **Property 16: Rejection requires non-empty reason** — Rejection reason must be non-empty and non-whitespace; system rejects empty/whitespace-only reasons
    - **Validates: Requirements 8.4**

  - [ ]* 8.4 Write property test: Review deadline and overdue detection
    - **Property 17: Review deadline and overdue detection** — Deadline = assignedAt + 24 hours exactly; overdue detection flags exactly those records where current time exceeds deadline and review not completed
    - **Validates: Requirements 8.6**

  - [ ]* 8.5 Write unit tests for Manual Review Service
    - Test reviewer actions — approve, reject, request info all functional (Req 8.2)
    - Test reviewer approval flow — status APPROVED + notification (Req 8.3)
    - Test additional info request — customer notification with instructions (Req 8.5)
    - _Requirements: 8.2, 8.3, 8.5_

- [ ] 9. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 10. Implement Audit Logger and Data Protection
  - [ ] 10.1 Implement Audit Logger
    - Implement `logEvent()` — record audit event with all required metadata: eventTimestamp (UTC+7), eventType, actor, sessionId, ipAddress, deviceInfo, resultStatus
    - Implement tamper-evident hash chain — each entry's previousHash = preceding entry's currentHash; currentHash = SHA-256(entry content); first entry uses genesis hash
    - Implement `searchLogs()` — search audit logs via Elasticsearch
    - Implement `exportLogs()` — export logs in CSV or JSON format
    - Implement `verifyIntegrity()` — verify hash chain integrity for date range
    - Store logs in both PostgreSQL (immutable records) and Elasticsearch (search)
    - _Requirements: 10.1, 10.2, 10.3, 10.4, 10.5_

  - [ ]* 10.2 Write property test: Audit event completeness and logging
    - **Property 20: Audit event completeness** — For any eKYC step completion or biometric data access, audit log entry contains all required metadata non-null
    - **Validates: Requirements 9.6, 10.1, 10.2**

  - [ ]* 10.3 Write property test: Audit log hash chain integrity
    - **Property 21: Audit log hash chain integrity** — Each entry's previousHash equals preceding entry's currentHash; currentHash = SHA-256(entry content); first entry uses genesis hash
    - **Validates: Requirements 10.4**

  - [ ] 10.4 Implement Encryption Service and RBAC
    - Implement AES-256 encryption for biometric data and ID document images at rest
    - Configure TLS 1.2+ for all in-transit communication
    - Implement role-based access control — only REVIEWER and COMPLIANCE_OFFICER can access biometric data
    - Implement biometric data deletion — delete within 30 days on customer request, preserve regulatory-required data
    - Log every biometric data access in audit trail
    - _Requirements: 9.1, 9.3, 9.4, 9.5, 9.6, 9.7_

  - [ ]* 10.5 Write property test: RBAC for biometric data access
    - **Property 19: RBAC for biometric data access** — Access granted iff role is REVIEWER or COMPLIANCE_OFFICER; all other roles denied
    - **Validates: Requirements 9.5**

  - [ ]* 10.6 Write unit tests for Audit Logger and Data Protection
    - Test audit search/export endpoints functional (Req 10.5)
    - Test data deletion — biometric data removed, regulatory data preserved (Req 9.3)
    - Test breach notification — notification within 72h (Req 9.7)
    - _Requirements: 9.3, 9.7, 10.5_

- [ ] 11. Implement Error Handling and Session Recovery
  - [ ] 11.1 Implement retry mechanism with exponential backoff
    - Implement retry logic for external service calls (OCR, Face Matching, NFC, National DB)
    - Retry up to 3 times with exponential delay: delay_n = base_delay × 2^n
    - Return user-friendly error message after all retries exhausted
    - Implement fallback strategies: NFC failure → OCR-only; National DB down → manual review
    - _Requirements: 11.2, 3.4, 6.3_

  - [ ]* 11.2 Write property test: Retry with exponential backoff
    - **Property 23: Retry with exponential backoff** — For any failed external service call, retry up to 3 times with exponentially increasing delays; after 3 failures, return user-friendly error
    - **Validates: Requirements 11.2**

  - [ ] 11.3 Implement unrecoverable error handling
    - Implement error workflow: (1) log error details in audit trail, (2) notify customer with unique reference number, (3) create support ticket
    - Ensure reference number in notification matches support ticket reference
    - _Requirements: 11.4_

  - [ ]* 11.4 Write property test: Unrecoverable error handling workflow
    - **Property 24: Unrecoverable error handling workflow** — For any unrecoverable error, system performs exactly 3 actions: log error, notify customer with reference number, create support ticket; reference numbers match
    - **Validates: Requirements 11.4**

  - [ ] 11.5 Implement session recovery
    - Implement `onDisconnect()` — save current state to Redis, keep session active until expiry
    - Implement `onReconnect()` — restore state from Redis if session still valid, return null if expired
    - Implement `onSessionExpired()` — notify customer, preserve uploaded documents in S3 for 24 hours, mark session EXPIRED, log audit event
    - _Requirements: 11.1, 11.3_

  - [ ]* 11.6 Write unit tests for Error Handling and Session Recovery
    - Test session expiry handling — notification + document preservation 24h (Req 11.3)
    - _Requirements: 11.3_

- [ ] 12. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 13. Integration wiring and end-to-end flow
  - [ ] 13.1 Wire API Gateway and service routing
    - Implement API Gateway with rate limiting and JWT authentication
    - Configure REST endpoints for synchronous eKYC flow
    - Configure message queue for async operations (audit logging, notifications)
    - Wire session management with Redis + JWT
    - _Requirements: 1.4, 1.5, 12.2_

  - [ ] 13.2 Wire the complete eKYC flow pipeline
    - Connect Session Service → Document Service → NFC Service → Face Matching → Liveness Detection → Data Verification → Scoring Engine → Manual Review
    - Wire Audit Logger to all services for event logging
    - Wire Consent Manager checks before each data collection step
    - Wire Encryption Service for all biometric data paths
    - Wire Notification Service for customer and reviewer notifications
    - _Requirements: 1.1–1.5, 6.1, 7.1, 8.3, 8.4, 8.5, 9.2, 9.6, 10.1_

  - [ ] 13.3 Implement health dashboard endpoint
    - Create service health endpoint returning availability status of all dependent services (OCR, Face Matching, Liveness Detection, NFC, National Database)
    - _Requirements: 11.5_

  - [ ]* 13.4 Write integration tests for end-to-end flow
    - Test complete eKYC happy path flow
    - Test OCR accuracy >= 95% on CCCD test set (Req 2.6)
    - Test NFC read within 15 seconds (Req 3.5)
    - Test face comparison with correct inputs (Req 4.2)
    - Test spoof detection >= 99% (Req 5.5)
    - Test verification triggered after biometric pass (Req 6.1)
    - Test scoring within 5 seconds (Req 7.5)
    - Test complete flow within 3 minutes (Req 12.1)
    - Test 500 concurrent sessions (Req 12.2)
    - Test OCR within 5 seconds per image (Req 12.3)
    - Test face comparison within 3 seconds (Req 12.4)
    - Test liveness verification within 2 seconds (Req 12.5)
    - _Requirements: 2.6, 3.5, 4.2, 5.5, 6.1, 7.5, 12.1–12.5_

- [ ] 14. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation after each major module group
- Property tests validate the 24 universal correctness properties defined in the design document using fast-check
- Unit tests validate specific examples and edge cases from acceptance criteria
- Integration tests validate cross-service interactions and performance requirements
- All biometric data paths must use AES-256 encryption at rest and TLS 1.2+ in transit
