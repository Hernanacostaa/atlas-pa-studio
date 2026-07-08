# ATLAS PA Agent (Studio Edition)

AI-powered Practical Activity Worksheet generator for CO+I Learning.

**Architecture:** Copilot Studio + Power Automate only — zero Azure Functions, zero custom code.

## What It Does

Takes source documents (SOPs, MOPs, Training Plans, SCORM Course Analysis Reports) and generates formatted PA Worksheet Word documents automatically.

## Technology Stack

- **Copilot Studio** — Agent orchestration, AI prompts, conversation flow
- **Power Automate** — File extraction, document generation, email delivery
- **SharePoint** — SCORM library (Knowledge source) + output archive
- **Document Output (Preview)** — Native Word doc generation from templates
- **Teams** — User interface

## Documentation

- [Architecture](ARCHITECTURE.md) — Full system design and component details
- [Milestones](MILESTONES.md) — Build tracker (6 phases, 29 milestones)
- [Implementation Dashboard](https://hernanacostaa.github.io/atlas-pa-studio/) — Interactive GitHub Pages status view

## Status

🟡 Phase 1 — Proving Document Output with PA template
