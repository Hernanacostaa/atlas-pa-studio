# ATLAS PA Studio — Project Instructions

## Project Context
This is the ATLAS PA Agent built entirely in Copilot Studio + Power Automate.
No Azure Functions. No custom code. No deployments.

Owner: Hernan Acosta (PM, not a developer — needs step-by-step GUI guidance)

## Architecture
- Copilot Studio agent with Generative Orchestration
- Knowledge source: SharePoint SCORM library (100+ Course Analysis Reports)
- Prompt Builder: PA field extraction (18 fields from source text)
- Document Output: Word doc generation from {{placeholder}} template
- Global Variables: Session state for preview/edit loop
- Power Automate: 2 flows (ExtractText + GenerateDoc)
- Delivery: SharePoint archive + email + Teams chat link

## Key Files
- ARCHITECTURE.md — Full system design
- MILESTONES.md — Build tracker (5 phases, 20 milestones)

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

## PA Fields (18 total)
PA_Title, PA_Subtitle, PA_DocumentLabel, CourseReference, Authors,
Contributors, LastUpdated, TargetAudience, Duration, ActivityDescription,
TrainerGuidelines, DesiredLearningOutcome, WhatIsNeeded,
SkillsBasedLearningObjectives, DocumentationAndReferences,
ActivitySteps, Validation, Notes
