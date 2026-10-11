# AI Lead Qualification & Follow-up with n8n and Gemini

A two-workflow system for form-based lead qualification, email follow-up, and runtime failure handling, built with **n8n**, **Google Gemini**, **Gmail**, **Google Sheets**, and **n8n Data Tables**.

The workflow validates inquiries, extracts structured lead information, applies deterministic scoring, drafts a customer response, and sends it only after email guardrails pass. Separate operational records and a human-facing review queue keep processing state and exceptions visible.

## Project status

- Main workflow with 25 nodes and companion Error Handler with 32 nodes
- Live regression tests reported passed by the project author before this public export
- Both public workflow templates have credentials and instance-specific references removed
- No client deployments, business results, or performance benchmarks claimed
- Both workflows are included; configure and link the Error Handler to enable the runtime-failure safeguards

## Included workflows

| Workflow | File | Role |
|---|---|---|
| AI Lead Qualification & Follow-up | [AI-Lead-Qualification-Followup.json](./AI-Lead-Qualification-Followup.json) | Intake, validation, qualification, guarded email delivery, and CRM updates |
| AI Lead Qualification - Error Handler | [AI-Lead-Qualification-Error-Handler.json](./AI-Lead-Qualification-Error-Handler.json) | Runtime failure classification, state persistence verification, and review queues |

## What the workflow does

1. `On form submission` receives the inquiry; `Normalize Input` standardizes fields and creates tracking identifiers.
2. `Input Validation` checks the submitted fields before AI processing.
3. `Duplicate Check - New Lead` checks the backend for the normalized email-and-message key.
4. `Save PROCESSING State` records the attempt in `lead_processing`.
5. `AI Lead Extraction` produces structured information; `Validate AI JSON` validates its shape, types, values, and consistency with the submitted choices.
6. `Business Rules - Lead Scoring` calculates a score; `Route Lead Status` selects the response or review path.
7. `AI Email Draft` generates a structured subject and HTML body; `Email Guardrails` validates the draft.
8. `Mark EMAIL_READY` records readiness before `Gmail Send` sends the response.
9. `Update Final State` records completion and the Gmail message ID, then `Update AI Lead CRM` appends or updates the lead in `CRM_Leads`.

`HOT`, `WARM`, `COLD`, and `NEEDS_INFO` all enter email drafting. `REVIEW` and the routing fallback enter manual review without sending an email.

## Lead validation and qualification

Input checks cover name and message lengths, email format, allowed service/budget/timeline choices, and optional company/business-type lengths. Invalid input is recorded by `Log INVALID_INPUT` without AI processing or email delivery.

AI extraction returns exactly seven fields: `service_category`, `intent`, `budget`, `timeline`, `requirements_summary`, `missing_fields`, and `confidence`. Validation checks allowed categories and intents, preserves the submitted service/budget/timeline choices, requires a 10–1500-character summary, validates the missing-fields array, and requires finite confidence between 0 and 1.

Null and type guards allow malformed output reaching the validation node to fail into the controlled path. An upstream model or parser exception is a runtime failure handled by the companion Error Handler instead.

## Deterministic lead scoring

The AI extracts information; the code node calculates the score from fixed rules.

| Component | Points | Rule |
|---|---:|---|
| Budget fit | 0–4 | Service-specific budget lookup |
| Intent | 0–3 | Ready to buy: 3; evaluating: 2; researching: 1; unclear: 0 |
| Timeline | 0–2 | 2–4 weeks or 1–2 months: 2; ASAP or flexible: 1; unsure: 0 |
| Clarity | 0–1 | No missing fields: 1; otherwise: 0 |

Routing precedence is fixed:

1. Confidence below `0.60` or category `Other` → `REVIEW`.
2. Missing fields or unclear intent → `NEEDS_INFO`.
3. Otherwise: score 8–10 → `HOT`; 5–7 → `WARM`; below 5 → `COLD`.

Confidence is a review signal, not a score component. The complete budget matrix remains in `Business Rules - Lead Scoring`.

## AI email drafting and guardrails

Drafts acknowledge the inquiry according to its status. `NEEDS_INFO` asks for missing details. Prompts treat customer content as untrusted data and prohibit invented pricing, commitments, results, or delivery promises.

The draft contract contains only `email_subject` and `email_body_html`. Guardrails check:

- Recipient format and length
- Plain-text subject of 3–150 trimmed characters
- HTML body of 20–5000 trimmed characters
- Only `p`, `strong`, `ul`, `li`, and `br` tags, without attributes
- Rejection of event handlers, JavaScript patterns, URLs, and link attributes
- Rejection of specified internal scoring, classification, and prompt-related phrases
- Exact output keys and safe handling of missing or incorrectly typed values

These are concrete checks and prompt restrictions, not a guarantee against every possible unsafe or inaccurate model response.

## Duplicate protection

The normal dedupe key combines normalized email and message. An existing matching row stops the valid-submission path before AI processing and Gmail.

`Log INVALID_INPUT` stores `INVALID_INPUT|<original dedupe_key>` instead. The original key remains intact after the prefix for audit, while a corrected valid submission can use the normal key. Matching non-INVALID_INPUT submissions remain blocked.

When upgrading an existing installation, back up the table and prefix old unprefixed keys only for rows whose `processing_status` is exactly `INVALID_INPUT`. Do not double-prefix rows or alter other statuses. A fresh table needs no migration.

This is a lookup followed by an insert; this project does not claim an atomic concurrency lock or exactly-once delivery guarantee.

## Storage and human review

| Resource | Role |
|---|---|
| `lead_processing` | Backend state, duplicate keys, execution tracking, scores, email status, Gmail message ID, and error details |
| `CRM_Leads` | Human-facing lead details and qualification results, written after the successful send/final-state path; matched by `Lead ID` |
| `Review Queue` | Human-facing exceptions with reason, execution/lead reference, email status, and review-resolution fields |

Controlled paths are:

| Condition | State node | Review node |
|---|---|---|
| Invalid AI extraction output | `Mark AI_OUTPUT_INVALID` | `Queue AI_OUTPUT_INVALID Review` |
| Lead needs manual review or unmatched routing | `Mark REVIEW` | `Queue LEAD_REVIEW` |
| Invalid email draft | `Mark EMAIL_DRAFT_INVALID` | `Queue EMAIL_DRAFT_INVALID Review` |

Invalid form input is logged in the backend; it does not use these three review paths. Review entries require human investigation; this export does not include an automatic review-resolution or resend workflow.

## Error Handler dependency

The complete system uses a separate **AI Lead Qualification - Error Handler** workflow. It classifies runtime failures relative to Gmail (`BEFORE_GMAIL`, `AT_GMAIL`, `AFTER_GMAIL`, or `UNKNOWN`) and distinguishes failed sends from uncertain outcomes (`SEND_FAILED` versus `SEND_UNCERTAIN`). It uses operational state and review queues to support investigation without blindly resending an email.

The main workflow's original `settings.errorWorkflow` ID was removed because it belongs to a private instance. Import the included companion JSON as a second workflow, configure its resources, and manually select **AI Lead Qualification - Error Handler** in the main workflow's **Settings → Error Workflow** field. Save the main workflow before enabling live use. The main JSON alone does not install these runtime-failure safeguards.

Keep all main-workflow node names unchanged: expressions and the companion handler depend on exact names, including `Normalize Input`, `Business Rules - Lead Scoring`, `Mark EMAIL_READY`, `Gmail Send`, `Update Final State`, and `Update AI Lead CRM`.

## Technologies and requirements

| Technology | Requirement |
|---|---|
| n8n | Instance supporting the exported node versions, including Data Table 1.1 and the Gemini/LangChain nodes |
| Google Gemini | API credential and access to the models referenced by the two chat-model nodes |
| Gmail | OAuth credential for the sending account |
| Google Sheets | OAuth credential with access to the CRM spreadsheet |
| n8n Data Tables | A `lead_processing` table in the workflow's project |

Create your own credentials and resources before configuring the imported workflows. Both workflows must use the same `lead_processing` table and the same spreadsheet's `Review Queue` tab. No Python dependency file is required.

## Resource setup

Create a Google spreadsheet with tabs named `CRM_Leads` and `Review Queue`. Use the exact headers below; spelling and capitalization matter.

### CRM_Leads headers

```text
Submitted At
Full Name
Email
Company
Business Type
Service Needed
Service Category
Budget Range
Timeline
Requirements Summary
Lead Score
Lead Status
Original Message
Intent
Lead ID
```

### Review Queue headers

```text
review_id
created_at
lead_id
execution_id
email
review_reason
failed_node
error_message
email_status
review_status
review_notes
resolved_at
```

### lead_processing columns

Create these user-defined columns; n8n's built-in row fields are separate.

| Column | Type |
|---|---|
| `lead_id` | String |
| `dedupe_key` | String |
| `email` | String |
| `processing_status` | String |
| `lead_status` | String |
| `lead_score` | Number |
| `email_status` | String |
| `gmail_message_id` | String |
| `error_code` | String |
| `error_message` | String |
| `model_name` | String |
| `prompt_version` | String |
| `workflow_version` | String |
| `created_at` | Date/time |
| `processed_at` | Date/time |
| `execution_id` | String |

## Import and configuration

1. Download and import both [`AI-Lead-Qualification-Followup.json`](./AI-Lead-Qualification-Followup.json) and [`AI-Lead-Qualification-Error-Handler.json`](./AI-Lead-Qualification-Error-Handler.json) as separate workflows in the same n8n project. Keep the main workflow inactive during setup.
2. Assign your Gemini credential to `Google Gemini Chat Model` and `Google Gemini Chat Model1`; assign Gmail OAuth to `Gmail Send`.
3. Assign Google Sheets OAuth to `Update AI Lead CRM`, `Queue AI_OUTPUT_INVALID Review`, `Queue LEAD_REVIEW`, and `Queue EMAIL_DRAFT_INVALID Review` in the main workflow, and to all eight Google Sheets review-queue nodes in the Error Handler. The Error Handler does not need Gemini or Gmail credentials.
4. Replace the resource placeholders in **both workflows** by selecting your resources in each relevant node:

   | Placeholder | Select |
   |---|---|
   | `REPLACE_WITH_LEAD_PROCESSING_TABLE_ID` | Your `lead_processing` table in every Data Table node |
   | `REPLACE_WITH_GOOGLE_SPREADSHEET_ID` | Your CRM spreadsheet in every Google Sheets node |
   | `REPLACE_WITH_CRM_LEADS_SHEET_ID` | The `CRM_Leads` tab |
   | `REPLACE_WITH_REVIEW_QUEUE_SHEET_ID` | The `Review Queue` tab |

5. Confirm the column mappings still match the headers and table schema above. The CRM display timestamp expression uses `Asia/Karachi`; this public export preserves that behavior.
6. Open the main workflow's **Settings → Error Workflow**, manually select the imported **AI Lead Qualification - Error Handler**, and save. Importing both files does not create this link automatically because workflow IDs are instance-specific. Confirm the handler points to the same backend table and Review Queue before enabling live use.
7. Confirm both configured Gemini models are available to your account. Model changes require renewed validation; the public export preserves the validated configuration.
8. Test in a separate workflow using resources you control: invalid/corrected submissions, normal duplicates, malformed AI output, invalid drafts, and one controlled successful send to your own test inbox. Check the backend, review queue, and CRM results.
9. When configuration and checks pass, activate the workflow and use the form URL generated by your n8n instance. No private form URL is included here.

## Public export and limitations

Both JSON files contain placeholders instead of private spreadsheet, tab, and Data Table IDs. Credential references, webhook IDs where present, and workflow/version/instance metadata are removed. The main workflow's private Error Handler link is also removed and must be configured after import. Neither file contains pinned execution data.

The workflow logic, node names, node IDs, connections, scoring rules, and validated safeguards are preserved. Importing both JSON files creates the two workflow definitions, but does not create the external table, sheet tabs, credentials, or the link between the main workflow and Error Handler. Do not commit live submissions, execution exports, credentials, or private resource IDs when documenting your own deployment.

## License

This repository is licensed under the MIT License.

[Back to project index](../README.md)
