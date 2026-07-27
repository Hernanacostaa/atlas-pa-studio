# PA Studio - Production Migration Milestone Tracker

## Purpose

The existing `MILESTONES.md` tracks completion of the PA capability. This separate tracker covers migration of that working capability into a controlled production environment.

Development success does not automatically prove production readiness. A production item remains incomplete until it is configured and tested in the target environment with evidence.

## Summary

| Metric | Value |
|--------|-------|
| Total phases | 12 |
| Total production milestones | 157 |
| Production milestones complete | 0 |
| Current stage | Immediate Media field enhancement, then planning and inventory |
| Last updated | July 27, 2026 |

## Status Legend

- `[x]` Complete in production with evidence
- `[ ]` Not complete in production
- `[~]` In progress
- `[!]` Blocked

## Immediate Action Item: SCORM Media URLs

> **Priority:** Immediate
>
> **Production impact:** Complete before the production baseline is frozen.

- [ ] **IA.1** Add a new PA document row labeled **Media** and populate it with Media URLs found in the selected SCORM source file.

**Current state:** The live PA contract has 17 extracted fields. Completing this action will add `Media` as the 18th extracted field.

| Affected component | Required change |
|--------------------|-----------------|
| SCORM retrieval | Confirm the complete SCORM content passed into Create PA includes the original Media URLs without summarizing or rewriting them |
| Field contract | Add a string field named `Media`; define ordering, duplicate handling, separator format, and missing-value behavior |
| `ExtractPA` | Extract only Media URLs present in the source; never create or infer URLs |
| `EditPA` | Preserve `Media` during unrelated edits and allow an explicit Media edit |
| `FormatPreview` | Add a readable **Media** section to the chat preview |
| PA Word template | Add a new table row labeled **Media** with a mapped Document Output placeholder |
| `GeneratePA` | Add the `Media` input and map it to the new template row |
| `ATLAS-PA-GenerateDoc` | Update Parse JSON and the `GeneratePA` action mapping to include `Media` |
| Documentation | Update the PA data model from 17 to 18 fields only after the complete path is implemented |
| Regression testing | Test one URL, multiple URLs, duplicates, no Media URL, malformed source text, edit preservation, clickable output, and end-to-end SCORM generation |

**Acceptance criteria:** The selected SCORM file's Media URLs appear in the preview and in the generated Word document's **Media** row, unchanged from the source, with no invented links and no regression to the existing 17 fields.

## Complete Component Inventory

| # | Component | Current role | Production evidence required |
|---|-----------|--------------|------------------------------|
| 1 | PA Studio Copilot Studio agent | Orchestration and user conversation | Production agent ID, owner, published version |
| 2 | Agent instructions | Search/read/route behavior and raw-content handoff | Exported instructions and routing tests |
| 3 | Create PA topic | Deterministic extract-to-delivery pipeline | Production topic export and end-to-end tests |
| 4 | System and conversation-ending topics | Welcome, fallback, close, create another | Channel tests without unwanted survey/escalation |
| 5 | `ExtractPA` | Grounded field extraction, including pending `Media` expansion | Model/config capture and quality tests |
| 6 | `EditPA` | Updates current PA JSON and must preserve `Media` | Multi-edit regression tests |
| 7 | `FormatPreview` | Emoji-labeled readable preview, including `Media` | Cross-channel rendering tests |
| 8 | `GeneratePA` | Document Output generation, including the `Media` mapping | Template mapping and generated-file tests |
| 9 | PA Word template | Final document structure, including the pending **Media** row | Approved version, owner, placeholder inventory |
| 10 | `ATLAS-PA-GenerateDoc` | Parse and map every PA field, then generate, store, share, email, and return URL | Production flow run history |
| 11 | `SearchSCORM` backup flow | Retained backup search path | Exported, disabled/unused state documented, recovery test if retained |
| 12 | SharePoint Knowledge source | Finds and reads SCORM Course Analysis reports | Production source configuration and retrieval tests |
| 13 | Work IQ | Reads public and internal links | Production connection and link tests |
| 14 | SCORM source library | Authoritative course source | Active folder, permissions, file-size/count review |
| 15 | PA output library/folder | Stores generated Word documents | Production path, permissions, retention, capacity |
| 16 | SharePoint connector | Source/output access and sharing link | Production connection reference and owner |
| 17 | Outlook connector | Email delivery | Production connection reference and mailbox owner |
| 18 | Copilot Studio/Power Automate connection | Topic-to-flow execution | Production connection reference and non-owner test |
| 19 | User authentication and connector consent | Runs actions as the intended identity | First-time-user test |
| 20 | Power Platform solution | Packages all deployable components | Versioned import/export evidence |
| 21 | Environment variables | Holds environment-specific paths and values | Production values and dependency check |
| 22 | DLP policy | Controls approved connector combinations | Policy review and successful production flow run |
| 23 | Security groups and sharing | Controls who can discover and run the agent | Pilot and broad-access tests |
| 24 | Teams channel | User entry point | Published app and end-to-end channel test |
| 25 | Microsoft 365 Copilot channel | User entry point | Published agent and end-to-end channel test |
| 26 | Licensing and capacity | Enables Knowledge, Work IQ, prompts, and flows | Named entitlement/capacity owner and validation |
| 27 | Manual regression suite | Validates realistic production behavior | Completed signed test record |
| 28 | Evaluation CSVs | Reusable test definitions | Versioned files and documented runner limitation |
| 29 | Monitoring and alerting | Detects failed or degraded runs | Owner, alert route, dashboard/runbook |
| 30 | Support and escalation | Resolves user and platform incidents | Published support process |
| 31 | Backup and rollback | Restores the last known good version | Tested export/import or rollback procedure |
| 32 | Change and release management | Controls prompt, model, flow, and template updates | Versioning and approval record |
| 33 | `Media` field enhancement | Carries Media URLs from the SCORM source through preview and Word output | IA.1 completion and Media regression evidence |

---

## Phase 0: Production Scope and Ownership

> Goal: Establish who owns the release, what "production" means, and what must be true at launch.

- [ ] **0.1** Identify the target Power Platform/Copilot Studio production environment
- [ ] **0.2** Assign the business owner and final content-quality approver
- [ ] **0.3** Assign the Copilot Studio and Power Automate technical owner
- [ ] **0.4** Assign the SharePoint source and output owners
- [ ] **0.5** Assign the production support and escalation owner
- [ ] **0.6** Define pilot users, broad audience, and excluded audiences
- [ ] **0.7** Define measurable go-live success criteria
- [ ] **0.8** Approve the migration window, change freeze, and go/no-go authority

**Gate:** Named owners approve the production scope and success criteria.

---

## Phase 1: Capture the Working Development Baseline

> Goal: Preserve every component and setting that currently makes PA Studio work.

- [ ] **1.1** Export the current PA Studio solution or document why components are not yet solution-aware
- [ ] **1.2** Record the development environment, agent ID, owner, and published version
- [ ] **1.3** Export or capture the complete agent instructions
- [ ] **1.4** Export or capture the Create PA topic, triggers, variables, conditions, and go-to steps
- [ ] **1.5** Capture welcome, fallback, escalation, end-of-conversation, and survey settings
- [ ] **1.6** Capture `ExtractPA`, `EditPA`, `FormatPreview`, and `GeneratePA` prompt configurations
- [ ] **1.7** Record each prompt's model, moderation, inputs, outputs, and tool-availability setting
- [ ] **1.8** Version the current 17-field schema and the approved 18-field schema after `Media` action IA.1 is complete
- [ ] **1.9** Version the approved PA Word template and all placeholder mappings
- [ ] **1.10** Export `ATLAS-PA-GenerateDoc` and capture all expressions, branches, and timeout behavior
- [ ] **1.11** Export the backup `SearchSCORM` flow and document that it is not in the current runtime path
- [ ] **1.12** Capture Knowledge, Work IQ, SharePoint, Outlook, connection, and channel configurations

**Gate:** A reviewer can reconstruct the development build from the captured baseline.

---

## Phase 2: Production Solution and Environment

> Goal: Package dependencies so imports are repeatable and environment values are controlled.

- [ ] **2.1** Confirm production environment region, type, security group, and administrative owners
- [ ] **2.2** Create or approve the solution publisher, prefix, and versioning convention
- [ ] **2.3** Add the PA Studio agent and custom topics to the solution
- [ ] **2.4** Add all four PA prompts and the Document Output configuration
- [ ] **2.5** Add `ATLAS-PA-GenerateDoc` and retained backup flows
- [ ] **2.6** Add every required connection reference
- [ ] **2.7** Create environment variables for source site, source folder, output site, output folder, and other environment-specific settings
- [ ] **2.8** Run a dependency check and resolve missing references
- [ ] **2.9** Perform a non-production export/import rehearsal
- [ ] **2.10** Document the approved managed/unmanaged deployment strategy

**Gate:** The complete solution imports without manually recreating hidden components.

---

## Phase 3: Knowledge, Source Content, and Retrieval

> Goal: Ensure production can find and read every source type the agent advertises.

- [ ] **3.1** Confirm the canonical SCORM source site, library, and active folder path
- [ ] **3.2** Verify production identities have least-privilege access to the SCORM source
- [ ] **3.3** Attach and configure the SharePoint Knowledge source in production
- [ ] **3.4** Verify the source remains under the 1,000-file Knowledge limit
- [ ] **3.5** Identify files above the applicable 7 MB Knowledge limit and define remediation
- [ ] **3.6** Verify representative legacy `.doc` files are indexed and retrievable
- [ ] **3.7** Test exact course-name and course-ID retrieval
- [ ] **3.8** Test ambiguous keyword search and top-match presentation
- [ ] **3.9** Configure and test Work IQ for public URLs
- [ ] **3.10** Configure and test Work IQ for authorized internal Word and PDF links
- [ ] **3.11** Test multiple-link and mixed SCORM-plus-notes retrieval
- [ ] **3.12** Verify the orchestrator passes full raw content, including Media URLs, rather than a summary

**Gate:** Every advertised source path produces sufficient content for extraction.

---

## Phase 4: Prompts, Schema, and Word Template

> Goal: Reproduce the tested content behavior and document fidelity in production.

- [ ] **4.1** Import or recreate `ExtractPA`
- [ ] **4.2** Confirm `ExtractPA` uses the approved model and moderation level
- [ ] **4.3** Confirm all approved field names, including `Media`, match extraction, JSON, flow, and template mappings
- [ ] **4.4** Confirm field-by-field no-fabrication rules are intact
- [ ] **4.5** Confirm ActivitySteps retains ordered, observable, source-specific actions
- [ ] **4.6** Import or recreate `EditPA`
- [ ] **4.7** Verify `EditPA` preserves unchanged fields across multiple edits
- [ ] **4.8** Import or recreate `FormatPreview`
- [ ] **4.9** Verify preview labels and formatting across channels
- [ ] **4.10** Import or recreate `GeneratePA` with Document Output enabled
- [ ] **4.11** Upload and map the approved PA Word template
- [ ] **4.12** Test bullets, checkboxes, multiline text, tables, and fixed labels
- [ ] **4.13** Confirm prompt text outputs are consumed through `.text`
- [ ] **4.14** Lock all prompts to **Only when referenced by topics or agents**

**Gate:** Production extraction and generated documents match approved development examples.

---

## Phase 5: Power Automate, Storage, and Delivery

> Goal: Reproduce the complete document-delivery pipeline with production-owned dependencies.

- [ ] **5.1** Import `ATLAS-PA-GenerateDoc`
- [ ] **5.2** Bind production Copilot Studio, SharePoint, and Outlook connection references
- [ ] **5.3** Validate the Parse JSON schema against every approved field, including `Media`
- [ ] **5.4** Validate field mappings into `GeneratePA`
- [ ] **5.5** Confirm the Document Output byte conversion expression uses `base64ToBinary`
- [ ] **5.6** Create or approve the production PA output library/folder
- [ ] **5.7** Apply least-privilege write, read, and sharing permissions
- [ ] **5.8** Define file naming, invalid-character handling, and duplicate-name behavior
- [ ] **5.9** Configure the required sharing-link type and scope
- [ ] **5.10** Configure production email sender, recipients, subject, body, and attachment/link behavior
- [ ] **5.11** Confirm the flow returns the document URL to the topic before the platform timeout
- [ ] **5.12** Add visible failure handling for prompt, file, sharing-link, and email failures
- [ ] **5.13** Confirm successful files open without corruption
- [ ] **5.14** Document output retention, deletion, capacity, and ownership
- [ ] **5.15** Import `SearchSCORM` if retained and keep it outside the active runtime path

**Gate:** A production flow run creates, stores, shares, emails, and returns a valid Word document.

---

## Phase 6: Agent, Orchestration, and Topic Control

> Goal: Preserve the constrained-orchestrator behavior that prevents bypasses and partial execution.

- [ ] **6.1** Import or create the production PA Studio agent
- [ ] **6.2** Apply the approved agent instructions within the 8,000-character limit
- [ ] **6.3** Attach only the approved Knowledge and retrieval capabilities
- [ ] **6.4** Verify all prompts and flows are unavailable for direct orchestrator execution
- [ ] **6.5** Import and enable the Create PA topic
- [ ] **6.6** Verify `SourceContent` handoff and internal `SearchQuery` mapping
- [ ] **6.7** Verify blank-input routing asks search-versus-paste/link
- [ ] **6.8** Verify optional additional-content concatenation
- [ ] **6.9** Verify manual input shorter than the approved threshold is rejected and re-requested
- [ ] **6.10** Verify extraction output is stored in `paFieldsJSON`
- [ ] **6.11** Verify Choice/EmbeddedOptionSet comparisons use actual option values
- [ ] **6.12** Verify Generate and Edit branches loop correctly
- [ ] **6.13** Verify the clean close and create-another path
- [ ] **6.14** Disable or replace unwanted default survey, escalation, or "did that answer" behavior

**Gate:** The orchestrator cannot bypass the Create PA topic in production.

---

## Phase 7: Connections, Identity, Security, and Compliance

> Goal: Remove builder-account dependencies and confirm approved access boundaries.

- [ ] **7.1** Decide whether production connections use named owners, service accounts, or another approved identity model
- [ ] **7.2** Assign at least one backup owner for the agent, solution, and flows
- [ ] **7.3** Create and bind production connection references
- [ ] **7.4** Confirm connection owners retain required licenses and mailbox/site access
- [ ] **7.5** Review DLP policy compatibility for all connector combinations
- [ ] **7.6** Confirm no blocked direct SharePoint HTTP dependency exists in the active path
- [ ] **7.7** Review source and output data classification, retention, and sharing-link policy
- [ ] **7.8** Apply least-privilege permissions to Knowledge, flows, output storage, and users
- [ ] **7.9** Define the agent security group and sharing process
- [ ] **7.10** Test an authorized non-builder user
- [ ] **7.11** Test an unauthorized user cannot retrieve restricted source or output content
- [ ] **7.12** Test first-time connector consent and document the user experience
- [ ] **7.13** Confirm audit, compliance, and records requirements with the appropriate owners

**Gate:** Production does not depend on Hernan's personal builder session or excessive permissions.

---

## Phase 8: Licensing, Capacity, Channels, and Access

> Goal: Ensure intended users can discover and run the agent in supported channels.

- [ ] **8.1** Confirm Copilot Studio licensing and environment capacity
- [ ] **8.2** Confirm Power Automate licensing and request/run limits
- [ ] **8.3** Confirm Work IQ and M365 Copilot entitlements for intended users
- [ ] **8.4** Confirm AI Builder or prompt capacity and its owner
- [ ] **8.5** Resolve any organization policy blocking agent installation or sharing
- [ ] **8.6** Publish and validate the Teams channel
- [ ] **8.7** Publish and validate the Microsoft 365 Copilot channel
- [ ] **8.8** Validate Copilot Studio test chat as the builder diagnostic channel
- [ ] **8.9** Configure agent name, icon, description, welcome message, and discoverability
- [ ] **8.10** Confirm the same supported behavior across all production channels

**Gate:** A pilot user can find and run the agent without builder intervention.

---

## Phase 9: Production Regression and User Acceptance

> Goal: Validate functionality, quality, permissions, and resilience in the target environment.

- [ ] **9.1** Run a basic production smoke test
- [ ] **9.2** Test SCORM exact-name search through document delivery
- [ ] **9.3** Test SCORM ambiguous-keyword search and selection
- [ ] **9.4** Test pasted source content
- [ ] **9.5** Test a public Microsoft Learn URL
- [ ] **9.6** Test an authorized internal SharePoint Word link
- [ ] **9.7** Test an authorized internal SharePoint PDF link
- [ ] **9.8** Test multiple public or internal links
- [ ] **9.9** Test SCORM content plus supplemental user content
- [ ] **9.10** Test empty, short, and nonsense input recovery
- [ ] **9.11** Verify all approved fields, including `Media`, and fixed document labels
- [ ] **9.12** Verify no unsupported facts or actions are invented
- [ ] **9.13** Verify ActivitySteps specificity and source order
- [ ] **9.14** Verify preview readability in every channel
- [ ] **9.15** Test multiple sequential edits without field loss
- [ ] **9.16** Verify Word formatting and file integrity
- [ ] **9.17** Verify SharePoint path, naming, sharing link, email, and returned URL
- [ ] **9.18** Test first-time user consent
- [ ] **9.19** Test unauthorized link and source access
- [ ] **9.20** Test timeout, connector failure, and visible error behavior
- [ ] **9.21** Test near-limit source and output payloads
- [ ] **9.22** Test clean conversation ending and creation of a second PA
- [ ] **9.23** Record defects, fixes, and regression results
- [ ] **9.24** Obtain business-owner and pilot-user acceptance signoff

**Gate:** All critical tests pass with recorded evidence and no unresolved go-live blocker.

---

## Phase 10: Cutover and Rollback

> Goal: Release deliberately and preserve a tested recovery path.

- [ ] **10.1** Freeze development changes for the final migration package
- [ ] **10.2** Create the final versioned solution export and template backup
- [ ] **10.3** Back up current production components before replacement, if applicable
- [ ] **10.4** Import the approved release into production
- [ ] **10.5** Resolve connection references and environment variables
- [ ] **10.6** Publish the production agent and channels
- [ ] **10.7** Run immediate post-deployment smoke tests
- [ ] **10.8** Communicate pilot availability, access steps, and known limitations
- [ ] **10.9** Define rollback triggers and decision authority
- [ ] **10.10** Test or rehearse rollback to the last known good export
- [ ] **10.11** Record final go-live approval, release version, and date

**Gate:** Production is live with a recoverable, documented release.

---

## Phase 11: Monitoring, Support, and Continuous Control

> Goal: Keep the agent reliable after launch and prevent undocumented drift.

- [ ] **11.1** Assign an owner to review failed Power Automate runs
- [ ] **11.2** Define alerting for repeated generation, SharePoint, link, or email failures
- [ ] **11.3** Monitor flow duration against the 100-second platform limit
- [ ] **11.4** Monitor AI/prompt usage and capacity
- [ ] **11.5** Monitor SharePoint output growth, retention, and permission drift
- [ ] **11.6** Review Knowledge indexing, source additions, oversized files, and access changes
- [ ] **11.7** Publish user support and escalation instructions
- [ ] **11.8** Maintain a defect, feedback, and enhancement log
- [ ] **11.9** Run the manual regression suite after prompt, model, topic, flow, connector, or template changes
- [ ] **11.10** Version every production prompt, model, schema, flow, and template change
- [ ] **11.11** Review owner accounts, licenses, and connection health on a schedule
- [ ] **11.12** Complete a post-launch review after the pilot period
- [ ] **11.13** Approve broad rollout only after pilot evidence meets success criteria

**Gate:** The agent has durable ownership and a repeatable production operating model.

## Production Evidence Log

Record proof here or link to the approved system of record.

| Milestone | Environment | Evidence/link | Owner | Date | Result |
|-----------|-------------|---------------|-------|------|--------|
| Pending | Pending | Pending | Pending | Pending | Pending |
