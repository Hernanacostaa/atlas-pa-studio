# ATLAS PA Agent — Studio-Only Milestone Tracker

## Summary

| Metric | Value |
|--------|-------|
| Total Phases | 5 |
| Total Milestones | 20 |
| Complete | 0/20 (0%) |
| Architecture | Copilot Studio + Power Automate (Zero Azure) |

---

## Phase 1: Prove the Core (Document Output)

> Goal: Confirm Document Output feature works with our PA template formatting

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 1.1 | Create PA_Template_V2.docx with {{placeholders}} | ⬜ | 18 fields, same table/shading format as current output |
| 1.2 | Upload template to Document Output prompt | ⬜ | Verify all placeholders are detected |
| 1.3 | Test with sample PA data | ⬜ | Confirm formatting, line breaks, multiline content |
| 1.4 | Build GenerateDoc flow (prompt → SharePoint → email → return link) | ⬜ | End-to-end doc generation via Power Automate |

**Decision Gate:** If 1.3 fails on formatting → stay on Azure build. If it works → proceed.

---

## Phase 2: LLM Extraction (Prompt Node)

> Goal: Replace Azure Function's generate_all_pa_fields() with a Studio prompt node

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 2.1 | Create PA extraction prompt in Copilot Studio | ⬜ | Same proven prompt (facilitator-level, no fabrication, TBD for missing) |
| 2.2 | Define structured output (18 PA fields as JSON) | ⬜ | Test parsing with Power Fx |
| 2.3 | Store output in 18 global variables | ⬜ | Set Variable nodes after prompt call |
| 2.4 | Test extraction quality with real source text | ⬜ | Side-by-side vs Azure build output |

---

## Phase 3: Agent Conversation Flow (Topic)

> Goal: Build the full user-facing conversation in a Create PA topic

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 3.1 | Build Create PA topic with content source question | ⬜ | User / SCORM / Mixed options |
| 3.2 | Wire text extraction flow (SharePoint file → plain text) | ⬜ | ExtractText flow for links and SCORM files |
| 3.3 | Wire extraction prompt into topic | ⬜ | Call Action → prompt node → Set Variables |
| 3.4 | Build preview display (Message node with all fields) | ⬜ | Formatted readable output |
| 3.5 | Build edit loop (agent updates global variables) | ⬜ | "Change Duration to 2 hours" → update variable → re-display |
| 3.6 | Wire GenerateDoc flow into topic | ⬜ | Pass 18 globals → flow → return download link |
| 3.7 | Delivery message with download link | ⬜ | "Your PA has been generated and emailed" |

---

## Phase 4: SCORM Integration

> Goal: Agent can create PAs from the SCORM Course Analysis library

| # | Milestone | Status | Notes |
|---|-----------|--------|-------|
| 4.1 | Add Knowledge source (SharePoint SCORM folder) | ⬜ | /teams/COILearning/Agents/Course Analysis Reports V3 |
| 4.2 | Test Knowledge search with topic descriptions | ⬜ | "CDU introduction" → finds right doc |
| 4.3 | Build SCORM file retrieval (Knowledge → ExtractText flow) | ⬜ | Get full document text for PA extraction |
| 4.4 | Build mixed content path (SCORM + user text combined) | ⬜ | Concatenate both sources before extraction |

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

## Blockers

| ID | Description | Status | Impact |
|----|------------|--------|--------|
| B1 | Document Output is "preview" feature — could change | ⚠️ Open | If removed, need Azure fallback |
| B2 | SharePoint SCORM folder permissions in Knowledge | ⚠️ Open | May need site access granted |

---

## Architecture Reference

See: `ARCHITECTURE_STUDIO_ONLY_V3.md`
