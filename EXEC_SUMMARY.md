# PA Studio — Executive Summary

## Long Story Short

PA Studio is live and production-ready. It generates formatted Practical Activity Worksheets from SCORM courses, pasted content, or document links — fully automated, zero Azure, zero code. Users interact via Teams or M365 Copilot. All 46 milestones complete across 7 build phases.

---

## Long Story

### What It Is

PA Studio is an AI agent built in Microsoft Copilot Studio that automates the creation of Practical Activity Worksheets for CO+I Learning. Trainers and PMs provide source content (course name, pasted SOP, or document link) and receive a complete, formatted Word document — emailed and saved to SharePoint — in under 60 seconds.

### What It Replaces

Previously, PAs were created manually by reading source documents and filling in 17 fields by hand. This took 1-3 hours per document. The agent reduces that to a single conversation.

### How It Works

1. User tells the agent what they need (course name, paste content, or drop a link)
2. Agent finds and reads the source content via Knowledge source or Work IQ
3. AI extracts all 17 PA fields — specific to the source, no fabrication
4. User previews, optionally edits any fields
5. Agent generates a formatted .docx, emails it, and provides a download link

### Architecture

- Copilot Studio + Power Automate only — no Azure Functions, no custom code, no deployments to manage
- Knowledge source indexes ~300 SCORM Course Analysis Reports (.doc files)
- GPT-5 Reasoning handles extraction with field-by-field rules
- Document Output generates production-quality Word docs with bullets, checkboxes, and tables
- Orchestrator constrained via tool lockdown — can only search and route, can't go rogue

### What's Been Validated

- ✅ SCORM course search → pick → extract → generate
- ✅ Paste content directly → extract → generate
- ✅ Public URLs (Microsoft Learn) → extract → generate
- ✅ Internal SharePoint links (PDFs, .docx) → extract → generate
- ✅ Multi-source input (2 URLs combined)
- ✅ Edit loop (change fields, regenerate preview)
- ✅ Input validation (rejects empty/nonsense input)
- ✅ Deployed to Teams and M365 Copilot

### Key Metrics

| Metric | Value |
|--------|-------|
| Milestones complete | 46/46 |
| Phases shipped | 7 |
| Azure cost | $0 |
| End-to-end generation time | ~60 seconds |
| Fields extracted per PA | 17 |
| SCORM courses searchable | ~300 |

### What's Next (If We Choose to Extend)

- Broader user testing with Rachel's team
- Adaptive Card preview (visual polish)
- Error handling for edge cases (flow timeouts)
- Potential expansion to other document types beyond PAs
