# User Story: 06 - Implement Field-Level Security Controls

**As a** Data Owner,
**I want** field-level security controls to be enforced at the database layer,
**so that** users can only access specific columns of data that are relevant to their role and responsibilities.

## Acceptance Criteria

* Column-level access controls are defined for sensitive tables
* Field-level permissions are enforced at the database layer (PostgreSQL row/column security)
* A user accessing an unpermitted field receives an error or NULL value
* Security controls prevent data exfiltration through alternative query paths
* Data Owners can configure and manage field-level permissions
* Field-level access decisions are logged in the audit trail
* Performance impact is minimal and queries remain efficient
* Documentation clearly describes supported field-level control patterns

## Notes

* Currently, field-level security is not enforced, creating data leakage risk
* This provides defense-in-depth alongside application-level RBAC
* PostgreSQL row-level security (RLS) or similar mechanisms can be used
* Field masking and field-level security work together to protect sensitive data
