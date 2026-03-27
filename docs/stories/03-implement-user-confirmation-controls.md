# User Story: 03 - Implement User Confirmation and Warning Controls

**As a** End User / Security Administrator,
**I want** the plugin to display a confirmation prompt or warning message before transmitting sensitive email content to the AI service,
**so that** I have an explicit opportunity to review what will be sent and prevent accidental data leakage.

## Acceptance Criteria

* A confirmation dialog is displayed before the plugin sends email content to Legora
* The dialog clearly shows what data will be transmitted (email subject, body preview, attachments list)
* The dialog includes a warning if the email may contain sensitive information (configurable keywords or user labels)
* The user must explicitly click "Confirm" to proceed; "Cancel" aborts the transmission
* The confirmation is logged for audit purposes
* Dialog message can be customized per organization policy
* Feature is tested with end users to ensure usability without creating friction that breaks the workflow

## Notes

* Addresses the **user-induced data leakage risk** where users may not realize sensitive data is being sent
* Acts as a compensating control if native MIP label support is not available
* Low technical complexity, high risk reduction value
* Aligns with governance-first approach mentioned in the architect discussion
