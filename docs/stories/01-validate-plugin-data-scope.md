# User Story: 01 - Validate Plugin Data Scope

**As a** Security Architect,
**I want** to verify that the Legora Outlook plugin only accesses and transmits the currently selected email (not the entire mailbox),
**so that** I can confirm the technical containment model matches our risk assumptions and data exposure surface is limited.

## Acceptance Criteria

* Plugin architecture is documented to show it operates only on one email item at a time
* A technical proof or demo confirms that the plugin cannot read other emails from the mailbox
* Plugin performs no background mailbox scanning or automatic data collection
* Plugin requires explicit user interaction (opening an email and triggering AI action) before any data transmission
* Verification results are documented in the security assessment for compliance review

## Notes

* Earlier assumption was plugin could access full mailbox; this story validates the actual behavior
* Critical for establishing the baseline that data exposure is **user-triggered**, not system-driven
* Must be validated before proceeding with DLP effectiveness testing
* Reference: Office.js API permissions model (ItemReadWrite) supports this containment
