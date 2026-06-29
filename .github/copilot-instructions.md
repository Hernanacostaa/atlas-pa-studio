# ATLAS PA Studio — Project Instructions

## Project Context
This is the ATLAS PA Agent built entirely in Copilot Studio + Power Automate.
No Azure Functions. No custom code. No deployments.

Owner: Hernan Acosta (PM, not a developer — needs step-by-step GUI guidance)

## Architecture (V6 — Document Output Validated)
- Copilot Studio agent with Generative Orchestration
- Knowledge source: SharePoint SCORM library (content understanding, NOT primary matching)
- Prompt Builder: PA field extraction (GPT-5 Reasoning, 400K context)
- Document Generation: Document Output (Prompt Builder) — tested, handles bullets/line breaks/checkboxes
- Template: PA_Template_DocOutput.docx with {{placeholder}} syntax
- State: Single bot-scoped JSON variable `bot.paFieldsJSON` (NOT 18 separate globals)
- SCORM matching: Hybrid (ListSCORM flow + prompt picks match)
- Power Automate: 2-3 flows (ExtractText + GenerateDoc + ListSCORM)
- Preview: Adaptive Cards (not plain text)
- Delivery: SharePoint archive + email + Teams chat link
- Everything stays in ONE topic (no topic handoffs — preserves variable scope)
- Requires AI Builder (✅ confirmed in target environment)

## Critical Design Decisions
1. **Single JSON variable** — 18 globals is anti-pattern ("planner narrates success while topic sees blanks")
2. **Document Output** — tested and validated; handles bullets, line breaks, checkboxes, fully editable output
3. **Rejected "Populate a Word template"** — plain text controls don't support line breaks/bullets
4. **Hybrid SCORM** — Knowledge is non-deterministic; flow+prompt gives reliable matching
5. **GPT-5 Reasoning** — available in our tenant, 400K token context
6. **Content moderation: Low** — domain training content triggers false positives at default
7. **Flow timeout: 100s** — place email/archive AFTER return step

## Key Files
- ARCHITECTURE.md — Full system design (V4, research-validated)
- MILESTONES.md — Build tracker (5 phases, 22 milestones)

## Conventions
- Always push docs to GitHub immediately after creating/updating
- Update MILESTONES.md after completing any milestone
- Use plain language — no jargon without explanation
- Test before declaring something complete
- When giving Copilot Studio instructions, specify exact clicks and selections

## SharePoint Locations
- SCORM Library: https://microsoft.sharepoint.com/teams/COILearning
  Folder: /Agents/Course Analysis Reports V3
- PA Outputs: https://microsoft.sharepoint.com/sites/86dae876-a7f6-43da-824a-83a2c42644bb
  Folder: /Shared Documents/ATLAS-PA-Outputs

## PA Fields (18 total — stored as single JSON string)
PA_Title, PA_Subtitle, PA_DocumentLabel, CourseReference, Authors,
Contributors, LastUpdated, TargetAudience, Duration, ActivityDescription,
TrainerGuidelines, DesiredLearningOutcome, WhatIsNeeded,
SkillsBasedLearningObjectives, DocumentationAndReferences,
ActivitySteps, Validation, Notes

## Template Format
- {{FieldName}} placeholders (double curly braces, no spaces)
- Template uploaded in Prompt Builder → Document Output settings
- Requires AI Builder enabled ✅

## Document Output Key Formula (for flow)
```
binary(outputs('Run_a_prompt')?['body/responsev2/predictionOutput/documentOutput/contentBytes'])
```

## Platform Constraints
- Agent instructions: 8,000 chars max
- Conversation history: last 10 turns visible to orchestrator
- Cloud flow timeout: 100 seconds
- Connector payload: 5MB (public cloud)
- Knowledge files: 7MB without M365 Copilot license (silently ignored above)
