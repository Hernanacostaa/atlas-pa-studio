# ATLAS PA Agent — Studio-Only Milestone Tracker

## Summary

| Metric | Value |
|--------|-------|
| Total Phases | 5 |
| Total Milestones | 22 |
| Complete | 0/22 (0%) |
| Architecture | Copilot Studio + Power Automate (Zero Azure) |
| Last Updated | Research-validated design (V4) |

---

## Phase 1: Prove the Core (Word Template Connector)

> Goal: Confirm "Populate a Microsoft Word template" works with our PA layout and content controls

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 1.1 | Create PA_Template_V2.docx with content controls | ✅ | 18 Plain Text Content Controls with correct Title/Tags — done using separate file |
| 1.2 | Upload template to SharePoint (Templates folder) | ✅ | Uploaded to ATLAS-PA-Outputs/MVP Test/PA_Template_Updated.docx |
| 1.3 | Test "Populate a Microsoft Word template" with sample data | ⬜ | Quick flow: hardcoded values → fill template → verify formatting, multiline content |
| 1.4 | Build GenerateDoc flow | ⬜ | Parse JSON → Populate Word template → SharePoint save → email → return URL |

**Decision Gate:** If 1.3 fails on formatting → evaluate alternatives. If it works → proceed.

---

## Phase 2: LLM Extraction (Prompt Node)

> Goal: Replace Azure Function's generate_all_pa_fields() with a Studio prompt node

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 2.1 | Create PA extraction prompt in Copilot Studio | ⬜ | GPT-5 Reasoning model, facilitator-level rules, no fabrication, TBD for missing |
| 2.2 | Define JSON output (18 PA fields in single JSON string) | ⬜ | Output type: JSON. Single string stored in bot.paFieldsJSON |
| 2.3 | Store output in single bot-scoped variable | ⬜ | ~~18 global variables~~ → ONE variable `bot.paFieldsJSON` (anti-pattern fix) |
| 2.4 | Test extraction quality with real source text | ⬜ | Side-by-side vs Azure build output |
| 2.5 | Set content moderation to Low | ⬜ | Domain-specific training content may trigger false positives |

---

## Phase 3: Agent Conversation Flow (Topic)

> Goal: Build the full user-facing conversation in a single Create PA topic (no topic handoffs)

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 3.1 | Build Create PA topic with content source question | ⬜ | User / SCORM / Mixed options |
| 3.2 | Wire text extraction flow (SharePoint file → plain text) | ⬜ | ExtractText flow for links and SCORM files. For user files: pass directly to prompt (GPT reads docs natively) |
| 3.3 | Wire extraction prompt into topic | ⬜ | Call Action → prompt node → Set Variable (bot.paFieldsJSON) |
| 3.4 | Build preview display using Adaptive Card | ⬜ | ~~Plain text message~~ → Adaptive Card with structured field layout |
| 3.5 | Build edit loop (single topic, slot filling) | ⬜ | "Change Duration to 2 hours" → parse JSON → update field → re-serialize → re-display card |
| 3.6 | Wire GenerateDoc flow into topic | ⬜ | Pass bot.paFieldsJSON → flow → return download link |
| 3.7 | Delivery message with download link | ⬜ | "Your PA has been generated and emailed" |

**Important:** Keep everything in ONE topic so variables stay in scope throughout the pipeline.

---

## Phase 4: SCORM Integration (Hybrid Approach)

> Goal: Agent can create PAs from the SCORM Course Analysis library using reliable hybrid matching

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 4.1 | Add Knowledge source (SharePoint SCORM folder) | ⬜ | /teams/COILearning/Agents/Course Analysis Reports V3 |
| 4.2 | Build ListSCORM flow (returns all filenames) | ⬜ | Flow → SharePoint "List folder" → Select filenames → return array |
| 4.3 | Build SCORM matching prompt (file list + description → best match) | ⬜ | Deterministic: prompt receives full file list, picks best match by name |
| 4.4 | Wire ExtractText flow for matched SCORM file | ⬜ | Get full document text for PA extraction |
| 4.5 | Build mixed content path (SCORM + user text combined) | ⬜ | Concatenate both sources before extraction |

**Note:** Knowledge source is for content understanding/supplementary search. Reliable file matching uses Flow + Prompt (hybrid approach).

---

## Phase 5: Testing & Cutover

> Goal: Validate quality, retire Azure build

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 5.1 | Side-by-side comparison: Studio vs Azure output | ⬜ | Same source → compare PA quality |
| 5.2 | Test with 3+ real source documents | ⬜ | SOP, MOP, Training Plan |
| 5.3 | Test SCORM path end-to-end | ⬜ | SCORM search → extract → preview → generate |
| 5.4 | Disable Azure Function (or keep as fallback) | ⬜ | Decision: retire or archive |

---

## Blockers & Constraints

| ID | Description | Status | Impact |
|----|------------|--------|--------|
| ~~B1~~ | ~~Document Output is "preview" feature~~ | ✅ Resolved | Switched to GA "Populate a Microsoft Word template" connector |
| B2 | SharePoint SCORM folder permissions in Knowledge | ⚠️ Open | May need site access granted |
| B3 | 7MB file limit without M365 Copilot license | ⚠️ Open | Large SCORM files silently ignored by Knowledge source |
| B4 | Cloud flow timeout is 100 seconds | ℹ️ Known | Place email/archive AFTER return step in GenerateDoc flow |
| B5 | Connector payload limit 5MB (public cloud) | ℹ️ Known | Most SOPs under 1MB; monitor for large docs |

---

## Architecture Reference

See: `ARCHITECTURE.md`
