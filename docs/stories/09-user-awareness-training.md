# User Story: 09 - Develop User Awareness and Policy Training

**As a** Security Awareness Officer / CISO,
**I want** to create training materials and a deployment communication plan that educates users on Legora plugin capabilities, data handling practices, and acceptable use policies,
**so that** we reduce user-driven data leakage risk through awareness and establish clear expectations before plugin rollout.

## Acceptance Criteria

* A training module is created covering:
  * How the plugin works and what data it can access
  * What constitutes acceptable use (e.g., non-confidential contract analysis, permitted document types)
  * What constitutes prohibited use (e.g., uploading unredacted M&A strategies, personal data)
  * How to verify MIP labels and recognize warnings before sending data
  * Incident reporting procedures if accidental sensitive data exposure occurs
* Training is delivered to pilot users before general rollout (online module + live Q&A recommended)
* A user-facing "Quick Reference" card is created summarizing do's and don'ts
* Policy document is published in internal knowledge base (SharePoint, intranet, etc.)
* Sign-off of training completion is tracked (learning management system or simple acknowledgment)
* Training materials are reviewed annually and updated if control changes (e.g., new MIP policies)
* Post-deployment, user feedback is collected to identify misunderstandings or additional training needs

## Notes

* Complements technical controls (DLP, confirmation prompts, MIP labels) with human-centered risk reduction
* Critical component of the "hybrid control model" described in architect discussion
* Low cost, high impact on reducing user-driven data leakage
* Prerequisite: Technical controls (stories 03, 04) are defined so training content is accurate
