# ATLAS PA Agent — Studio-Only Milestone Tracker

## Summary

| Metric | Value |
|--------|-------|
| Total Phases | 6 |
| Total Milestones | 29 |
| Complete | 29/29 (100%) |
| Architecture | Copilot Studio + Power Automate (Zero Azure) |
| Last Updated | **ALL 29/29 MILESTONES COMPLETE** — FormatPreview validated (July 8, 2026) |

---

## Phase 1: Prove the Core (Document Output)

> Goal: Confirm Document Output works with PA template formatting, bullets, line breaks

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 1.1 | Create PA template with {{placeholders}} | ✅ | PA_Template_DocOutput.docx — 17 placeholders, same table/shading format |
| 1.2 | Upload template to Document Output prompt | ✅ | Tested in target environment |
| 1.3 | Test with complex PA data (bullets, checkboxes, multiline) | ✅ | Passed all formatting tests — bullets, ☐, line breaks, sub-headers, fully editable |
| 1.4 | Build GenerateDoc flow | ✅ | ATLAS-PA-GenerateDoc: Parse JSON → Run GeneratePA prompt (Document Output) → base64ToBinary → SharePoint save → sharing link → email w/ attachment → return URL. Expression: `base64ToBinary(body('Run_a_prompt')?['responsev2']?['predictionOutput']?['documentOutput']?['contentBytes'])` |

**Decision Gate:** ✅ PASSED — Document Output handles all PA formatting requirements.

**Note:** "Populate a Microsoft Word template" connector was tested and rejected — plain text content controls don't support line breaks or bullets.

---

## Phase 2: LLM Extraction (Prompt Node)

> Goal: Replace Azure Function's generate_all_pa_fields() with a Studio prompt node

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 2.1 | Create PA extraction prompt in Copilot Studio | ✅ | ExtractPA prompt — 1 input (SourceContent), GPT-5 Reasoning, facilitator-level rules, no fabrication, TBD for missing |
| 2.2 | Define JSON output (17 PA fields in single JSON string) | ✅ | All fields return as flat strings (no nested objects). Validated with sample SOP extraction |
| 2.3 | Store output in single bot-scoped variable | ✅ | paFieldsJSON (String) extracted from ExtractPAOutput.text via Set Variable node |
| 2.4 | Test extraction quality with real source text | ✅ | Tested with clean SOP and messy email draft — both produced accurate, complete JSON |
| 2.5 | Set content moderation to Low | ✅ | Already set to Low in Copilot Studio settings |

---

## Phase 3: Agent Conversation Flow (Topic)

> Goal: Build the full user-facing conversation in a single Create PA topic (no topic handoffs)

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 3.1 | Build Create PA topic with content source question | ✅ | Trigger + welcome message + Question (SourceContent) |
| 3.2 | Wire text extraction flow (SharePoint file → plain text) | ⏳ | Deferred — GPT reads pasted text natively; file/link extraction is Phase 4 enhancement |
| 3.3 | Wire extraction prompt into topic | ✅ | ExtractPA → ExtractPAOutput (Record) → Set Variable paFieldsJSON = .text (String) |
| 3.4 | Build preview display using Adaptive Card | ✅ | Text preview of paFieldsJSON with Generate/Edit choice (Adaptive Card upgrade later) |
| 3.5 | Build edit loop (single topic, slot filling) | ✅ | EditPA prompt updates JSON → Set Variable overwrites paFieldsJSON → Go to step loops back to preview |
| 3.6 | Wire GenerateDoc flow into topic | ✅ | GeneratePA flow called via Add a tool → paFieldsJSON input → downloadURL output |
| 3.7 | Delivery message with download link | ✅ | Success message + downloadURL + End conversation |

**Important:** Keep everything in ONE topic so variables stay in scope throughout the pipeline.

---

## Phase 4: SCORM Integration (Hybrid Approach)

> Goal: Agent can create PAs from the SCORM Course Analysis library using reliable hybrid matching

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 4.1 | Add Knowledge source (SharePoint SCORM folder) | ✅ | Knowledge source searches SCORM courses natively — found 5 GPU courses with IDs, URLs, details |
| 4.2 | Build ListSCORM flow (returns all filenames) | ⏭️ | **Skipped** — Knowledge source handles course search natively, no flow needed |
| 4.3 | Build SCORM matching prompt (file list + description → best match) | ⏭️ | **Skipped** — Agent orchestrator matches courses from Knowledge source automatically |
| 4.4 | Wire SCORM content into Create PA topic | ✅ | SourceContent accepts input from orchestrator; Condition node skips question when pre-filled; both SCORM and manual paths tested end-to-end |
| 4.5 | Build mixed content path (SCORM + user text combined) | ✅ | Topic has "additional content" question with Yes/No branch; Yes concatenates via `SourceContent & " ADDITIONAL NOTES: " & AdditionalContent`; No skips straight to ExtractPA |

**Note:** Knowledge source is for content understanding/supplementary search. Reliable file matching uses Flow + Prompt (hybrid approach).

---

## Phase 5: Testing & Cutover

> Goal: Validate quality, retire Azure build

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| ~~5.1~~ | ~~Side-by-side comparison: Studio vs Azure output~~ | ⏭️ | **Removed** — Studio is the only system; quality validated across 4 test paths |
| 5.2 | Test with 3+ real source documents | ✅ | Passed: power supply replacement, server decommissioning, fiber optic cable installation — all generated successfully |
| 5.3 | Test SCORM path end-to-end | ✅ | Test 1 passed: GPU baseboard course → search → pick → extract → preview → generate → email + download link |
| 5.4 | Disable Azure Function (or keep as fallback) | ✅ | **Decision: Keep as-is** — Azure Function left running as fallback; Studio is primary system |

---

## Blockers & Constraints

| ID | Description | Status | Impact |
|----|------------|--------|--------|
| ~~B1~~ | ~~Document Output is "preview" feature~~ | ✅ Resolved | Switched to GA "Populate a Microsoft Word template" connector |
| ~~B2~~ | ~~SharePoint SCORM folder permissions in Knowledge~~ | ✅ Resolved | Knowledge source reads SCORM .doc files successfully |
| B3 | 7MB file limit without M365 Copilot license | ⚠️ Open | Large SCORM files silently ignored by Knowledge source |
| B4 | Cloud flow timeout is 100 seconds | ℹ️ Known | Place email/archive AFTER return step in GenerateDoc flow |
| B5 | Connector payload limit 5MB (public cloud) | ℹ️ Known | Most SOPs under 1MB; monitor for large docs |

---

## Phase 6: ActivitySteps Depth Testing

> Goal: Ensure the orchestrator passes enough source detail to produce rich, granular ActivitySteps — not summarized/generic ones

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 6.1 | Baseline: check what orchestrator currently passes to SourceContent | ✅ | Identified issue: orchestrator was over-summarizing, ActivitySteps too generic |
| 6.2 | Compare SCORM source doc vs extracted ActivitySteps | ✅ | Steps were merged/summarized — not matching source procedure order |
| 6.3 | Test orchestrator instruction change: "include ALL procedural steps" | ✅ | Updated instructions with checklist framing: "each step is one verifiable action in the order the documentation describes it" |
| 6.4 | Test with a high-step-count course (15+ steps) | ✅ | GPU baseboard course retested — steps now match documentation order |
| 6.5 | Validate final ActivitySteps depth is production-acceptable | ✅ | Hernan confirmed: "It is perfect. all good." |
| 6.6 | Build FormatPreview prompt for readable preview display | ✅ | Created with GPT-4.1 mini — formats JSON into labeled list with emojis, works across all channels |
| 6.7 | Wire FormatPreview into Create PA topic | ✅ | Wired after both ExtractPA and EditPA Set Variable nodes; output stored as formattedPreview.text |
| 6.8 | Test preview rendering across channels | ✅ | Tested in Studio — formatted preview with emojis/labels on first view and after edits. Edit loop preserves prior changes. |

---

## Architecture Reference

See: `ARCHITECTURE.md`
