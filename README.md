# ATLAS PA Studio

AI-powered Practical Activity Worksheet generator for CO+I Learning.

**Current architecture:** PA Studio running in Copilot Studio + Power Automate, with SharePoint Knowledge for SCORM search and Work IQ for URL reading. Zero Azure.

## What It Does

PA Studio turns SCORM Course Analysis Reports, pasted source content, and document links into formatted Practical Activity Worksheet Word documents.

The orchestrator gathers raw content and routes everything through the **Create PA** topic for extraction, preview, editing, document generation, and delivery.

## Technology Stack

- **Copilot Studio** — agent orchestration, Create PA topic, prompts, and conversation flow
- **Power Automate** — `ATLAS-PA-GenerateDoc` flow for document creation, SharePoint save, link creation, and email delivery
- **Prompt Builder** — `ExtractPA`, `EditPA`, `GeneratePA`, and `FormatPreview`
- **SharePoint Knowledge** — SCORM Course Analysis Reports V3 library
- **Work IQ** — public URL and SharePoint document reading
- **Document Output** — native Word generation from the PA template
- **Teams / M365 Copilot / Copilot Studio test chat** — user entry points

## Documentation

- [Architecture](ARCHITECTURE.md) — current end-to-end system design
- [User Guide](USER_GUIDE.md) — current user experience and conversation flow
- [SCORM Integration](SCORM_INTEGRATION.md) — blockers, constraints, and final mitigation strategy
- [Milestones](MILESTONES.md) — build tracker (7 phases, 46 milestones)
- [Production Migration Milestones](PRODUCTION_MIGRATION_MILESTONES.md) — separate production deployment, validation, cutover, and operations tracker
- [Implementation Dashboard](https://hernanacostaa.github.io/atlas-pa-studio/) — interactive GitHub Pages status view

## Status

🟢 **Complete** — 7 phases, 46/46 milestones finished (100%)

## Current Runtime Flow

1. User starts in Teams, M365 Copilot, or Copilot Studio test chat
2. The orchestrator either searches SCORM Knowledge or reads pasted/linked content
3. The Create PA topic extracts fields, shows an emoji-labeled preview, and supports edits
4. The GenerateDoc flow creates the Word file, saves it to SharePoint, emails the user, and returns the link

## Key Safeguard

All prompts and flows are locked to **Only when referenced by topics or agents**, so the orchestrator cannot bypass the Create PA topic and call them directly.
