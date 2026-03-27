# User Story: 07 - Validate Attachment Security and Malware Scanning

**As a** Security Architect,
**I want** to verify that attachments uploaded via the plugin are subject to malware scanning, file type validation, and size limits,
**so that** we can confirm that plugin usage does not introduce malware, ransomware, or other file-based threats into the Legora platform or corporate environment.

## Acceptance Criteria

* Attachment file types are validated against an allowlist (e.g., PDF, DOCX, TXT, XLS only)
* Malware scanning is performed on all attachments before processing (antivirus integration confirmed)
* File size limits are enforced (e.g., max 25 MB per attachment, max 100 MB per request)
* Quarantine or rejection behavior is defined for suspicious files
* Validation rules and limits are configurable per organization
* A test suite is executed using safe malware samples (EICAR test file) to confirm detection and quarantine
* Behavior is documented with sample error messages shown to end users
* Audit log captures all rejected attachments with reason codes

## Notes

* Whitepaper mentions "file validation" and "malware scanning" via standard ingestion pipeline
* Scope of scanning (plugin-level vs. backend level) must be clarified
* Complements DLP testing by addressing file-based data exfiltration vectors
* May be lower priority if backend validation is already enforced
