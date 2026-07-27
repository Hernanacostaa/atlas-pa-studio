# ATLAS PA Studio — Project Instructions

## Project Context
This repo documents PA Studio, a Practical Activity Worksheet generator built entirely in Copilot Studio + Power Automate.
Zero Azure Functions. Zero custom code. Zero Azure services in the active runtime architecture.

Owner: Hernan Acosta (PM, not a developer — needs step-by-step GUI guidance)

## Architecture (V7 — Constrained Orchestrator, July 25 2026)
- Copilot Studio agent with a constrained orchestrator
- Channels: Teams, M365 Copilot, and Copilot Studio test chat
- Knowledge source: SharePoint SCORM library (`/Agents/Course Analysis Reports V3`, ~300 `.doc` files)
- Orchestrator handles SCORM Knowledge search and URL reading through Work IQ
- All prompts and flows are locked to **Only when referenced by topics or agents**
- The orchestrator cannot call prompts or flows directly; it must route through the **Create PA** topic
- Topic input: `SourceContent` from the orchestrator, stored internally as `Topic.SearchQuery`
- Create PA topic handles extraction, preview, editing, document generation, delivery, and clean ending
- `ExtractPA` prompt: GPT-5 Reasoning, 17-field extraction, field-by-field rules ported from the retired Azure Function, no fabrication
- `EditPA` prompt: inputs `CurrentJSON` + `EditRequest`, returns updated JSON
- `FormatPreview` prompt: GPT-4.1 mini, returns emoji-labeled readable text
- `GeneratePA` prompt: 17 inputs, Document Output enabled, template uploaded
- Power Automate: `ATLAS-PA-GenerateDoc` flow is active; `SearchSCORM` exists only as a backup
- Delivery: SharePoint archive + sharing link + email
- End of Conversation offers a clean "create another?" finish

## Critical Design Decisions
1. **Tool lockdown** — the Create PA topic is the only approved execution path for prompts and flows
2. **Pass raw content** — never summarize, shorten, or reorganize content before handing it to the topic
3. **Single deterministic topic** — extraction → preview → edit → generate → deliver
4. **FormatPreview** — current preview is emoji-labeled chat text
5. **Input validation** — on the manual path, pasted content shorter than 50 characters is rejected
6. **SearchSCORM is backup only** — current SCORM runtime path uses Knowledge search in the orchestrator
7. **No direct SharePoint HTTP** — DLP blocks that pattern, so stay inside approved tools and connectors

## Key Files
- ARCHITECTURE.md — Full current architecture and runtime flow
- USER_GUIDE.md — Current user experience
- SCORM_INTEGRATION.md — Blockers, mitigations, and why the constrained orchestrator exists
- MILESTONES.md — Build tracker (7 phases, 46 milestones, 100%)
- PRODUCTION_MIGRATION_MILESTONES.md — Separate tracker for production packaging, dependencies, security, testing, cutover, rollback, and operations
- README.md — Project overview and current status

## Conventions
- Always push docs to GitHub immediately after creating or updating them
- Update MILESTONES.md after completing any milestone
- Update PRODUCTION_MIGRATION_MILESTONES.md only when production evidence supports the status change
- Use plain language — no jargon without explanation
- Test before declaring something complete
- When giving Copilot Studio instructions, specify exact clicks and selections
- Keep documentation aligned to the live Copilot Studio build, not retired experiments

## SharePoint Locations
- SCORM Library: https://microsoft.sharepoint.com/teams/COILearning
  Folder: /Agents/Course Analysis Reports V3
- PA Outputs: https://microsoft.sharepoint.com/sites/86dae876-a7f6-43da-824a-83a2c42644bb
  Folder: /Shared Documents/ATLAS-PA-Outputs

## PA Data Model
**17 extracted fields**
PA_Title, PA_Subtitle, CourseReference, Authors,
Contributors, LastUpdated, TargetAudience, Duration,
ActivityDescription, TrainerGuidelines, DesiredLearningOutcome,
WhatIsNeeded, SkillsBasedLearningObjectives,
DocumentationAndReferences, ActivitySteps, Validation, Notes

**Fixed label**
- `PA_DocumentLabel` = `Practical Activity Worksheet`

## Immediate Pending Enhancement
- Add `Media` as an 18th extracted field containing hyperlink targets embedded behind clickable text in the selected SCORM Word source
- The user selects the SCORM course; never ask the user to paste or provide the Media URL
- First verify that Knowledge retrieval preserves the hidden hyperlink target in `Topic.SearchQuery`; display text alone is insufficient
- Add a new Word template row labeled **Media**
- Update `ExtractPA`, `EditPA`, `FormatPreview`, `GeneratePA`, Parse JSON, flow mappings, tests, and documentation together
- Do not describe `Media` as live until the complete path is implemented and validated
- Canonical action item: `PRODUCTION_MIGRATION_MILESTONES.md` → **IA.1**

## Proven Formulas

### Document Output file bytes
```
base64ToBinary(body('Run_a_prompt')?['responsev2']?['predictionOutput']?['documentOutput']?['contentBytes'])
```

### Additional content concatenation
```
Topic.SearchQuery & " ADDITIONAL NOTES: " & Topic.AdditionalContent
```

## Platform Constraints
- Agent instructions: 8,000 characters max
- Conversation history: last 10 turns visible to the orchestrator
- Cloud flow timeout: 100 seconds
- Connector payload: 5MB
- Knowledge files: 7MB without M365 Copilot license (silently ignored above that size)
- Knowledge files per agent: 1,000
- DLP blocks HTTP requests to SharePoint

## Guidance for Future Updates
- Describe the preview as emoji-labeled chat text
- Treat SearchSCORM as backup-only unless the runtime architecture changes
- Keep SCORM_INTEGRATION.md and ARCHITECTURE.md consistent when blockers or mitigations change
