# ATLAS PA Agent — Studio-Only Architecture V3

## Overview

The ATLAS PA Agent generates Practical Activity Worksheets from source documents.  
This architecture uses **only Copilot Studio + Power Automate** — zero Azure Functions, zero Azure OpenAI, zero custom code.

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| User Interface | Microsoft Teams | Where users interact with the agent |
| Orchestration | Copilot Studio (Generative Orchestration) | Routes conversation, manages state, runs AI |
| AI / LLM | Built-in GPT (Copilot Studio Prompt Builder) | PA field extraction + SCORM matching |
| State Management | Global Variables (Platform State Layer) | 18 PA fields persisted across conversation turns |
| Knowledge | SharePoint RAG (Knowledge Source) | SCORM Course Analysis library search |
| Automation | Power Automate (2 flows) | File extraction + document generation |
| Document Generation | Document Output (Preview) | Word doc from {{placeholder}} template |
| File Storage | SharePoint Online | Source files (SCORM) + output archive |
| Delivery | Outlook + Teams | Email with attachment + chat download link |

---

## End-to-End Flow

```
USER (Teams)
  │
  │ "Create a PA about CDU introduction"
  │
  ▼
COPILOT STUDIO AGENT
  │
  ├── KNOWLEDGE SOURCE ──────────────────────────────────────────┐
  │   SharePoint: /teams/COILearning/Agents/                     │
  │   Course Analysis Reports V3                                  │
  │   • 100+ Word docs, auto-synced                              │
  │   • Agent searches here for SCORM content                    │
  └──────────────────────────────────────────────────────────────┘
  │
  ├── TOPIC: Create PA ──────────────────────────────────────────┐
  │                                                               │
  │   [Question Node] ─── "How would you like to create?"        │
  │         │               • Own content                         │
  │         │               • SCORM library                       │
  │         │               • Both                                │
  │         ▼                                                     │
  │   [Condition Node] ── Branch on content path                  │
  │         │                                                     │
  │    ┌────┼────┐                                                │
  │    │    │    │                                                 │
  │  user scorm mixed                                             │
  │    │    │    │                                                 │
  │    ▼    ▼    ▼                                                │
  │   [Collect Input] ── paste text / topic desc / both           │
  │         │                                                     │
  │         ▼                                                     │
  │   [Call Action: ExtractText Flow] ── (if file/SCORM)          │
  │         │                                                     │
  │         ▼                                                     │
  │   [Call Action: PA Extraction Prompt]                         │
  │         │   Input: source text                                │
  │         │   Instructions: facilitator-level PA extraction     │
  │         │   Output: 18 PA fields (JSON)                       │
  │         │                                                     │
  │         ▼                                                     │
  │   [Set Variable × 18] ── Parse JSON → global variables        │
  │         │                                                     │
  │         ▼                                                     │
  │   [Message Node] ── "Here's your preview:"                   │
  │         │            Display all 18 fields                    │
  │         │            "Say 'generate' or tell me what          │
  │         │             to change"                              │
  │         │                                                     │
  │         ▼                                                     │
  │   [Edit Loop] ── User: "change Duration to 2 hours"          │
  │         │         Agent updates gblDuration                   │
  │         │         Re-displays preview                         │
  │         │         (repeats until user says "generate")        │
  │         │                                                     │
  │         ▼                                                     │
  │   [Call Action: GenerateDoc Flow]                             │
  │         │   Passes all 18 global variables                    │
  │         │   Returns: downloadURL                              │
  │         │                                                     │
  │         ▼                                                     │
  │   [Message Node] ── "Your PA has been generated and           │
  │                      emailed. [Download here](link)"          │
  │                                                               │
  └───────────────────────────────────────────────────────────────┘
  │
  ▼
OUTPUTS
  ├── SharePoint: ATLAS-PA-Outputs (archived .docx)
  ├── Email: User receives .docx attachment
  └── Teams Chat: Download link displayed
```

---

## Copilot Studio Components

### 1. Knowledge Source

| Setting | Value |
|---------|-------|
| Type | SharePoint |
| Site | https://microsoft.sharepoint.com/teams/COILearning |
| Folder | /Agents/Course Analysis Reports V3 |
| Content | 100+ Course Analysis Word documents |
| Sync | Automatic (reflects new files) |
| Auth | User's Microsoft credentials |

**Purpose:** When user picks "SCORM library," the agent searches this knowledge source to find the matching course document. Grounded answers cite which document was used.

### 2. Topic: Create PA

**Orchestration mode:** Generative  
**Trigger:** User expresses intent to create a PA

**Nodes used:**

| Node Type | Count | Purpose |
|-----------|-------|---------|
| Question | 2-3 | Content source, topic description, paste text |
| Condition | 1-2 | Branch on user/scorm/mixed |
| Call Action | 3 | ExtractText flow, Extraction prompt, GenerateDoc flow |
| Set Variable | 18 | Store each PA field from prompt output |
| Message | 3 | Preview display, edit confirmation, delivery |

### 3. Prompt: PA Field Extraction

| Setting | Value |
|---------|-------|
| Type | Prompt Builder (Call Action) |
| Model | Built-in GPT |
| Input | sourceText (string — raw document text) |
| Output | Structured JSON (18 PA fields) |

**Prompt instructions include:**
- PA is a FACILITATOR-LEVEL training document, not an SOP copy
- ActivitySteps: 10-12 summarized phases, not every sub-step verbatim
- Never fabricate PPE, tools, fault codes, or steps — use TBD/N/A
- Duration: extract from source or TBD, never estimate
- WhatIsNeeded: 4 categories (Safety Guidelines, Required PPE, Tooling, Additional Equipment)
- If source has Training Plan learning objectives, extract directly into Skills field

### 4. Global Variables (State)

**Storage:** Power Platform state layer (key-value store, separate from LLM context)  
**Lifetime:** Entire conversation session  
**Not stored in:** LLM token window, Dataverse, or any external database

| Variable | PA Field |
|----------|----------|
| gblPATitle | PA_Title |
| gblPASubtitle | PA_Subtitle |
| gblPADocumentLabel | PA_DocumentLabel |
| gblCourseReference | CourseReference |
| gblAuthors | Authors |
| gblContributors | Contributors |
| gblLastUpdated | LastUpdated |
| gblTargetAudience | TargetAudience |
| gblDuration | Duration |
| gblActivityDescription | ActivityDescription |
| gblTrainerGuidelines | TrainerGuidelines |
| gblDesiredLearningOutcome | DesiredLearningOutcome |
| gblWhatIsNeeded | WhatIsNeeded |
| gblSkillsObjectives | SkillsBasedLearningObjectives |
| gblDocReferences | DocumentationAndReferences |
| gblActivitySteps | ActivitySteps |
| gblValidation | Validation |
| gblNotes | Notes |
| gblContentPath | Content path (user/scorm/mixed) |

### 5. Document Output

| Setting | Value |
|---------|-------|
| Feature | Document Output (Preview) |
| Template | PA_Template_V2.docx |
| Placeholder format | {{FieldName}} (double curly braces) |
| Output | .docx file bytes |

**Template structure:** Same table layout, shading, column widths, and formatting as current PA output — with {{placeholders}} where content is injected.

---

## Power Automate Flows (2 total)

### Flow 1: ExtractText

| Property | Value |
|----------|-------|
| Trigger | Copilot Studio |
| Input | fileURL (string) |
| Output | plainText (string) |

**Actions:**
1. SharePoint → "Get file content" (download the file)
2. AI Builder → "Extract text from documents" OR Word Online → "Read text from file"
3. Return value → plainText

### Flow 2: GenerateDoc

| Property | Value |
|----------|-------|
| Trigger | Copilot Studio |
| Inputs | 18 PA field values (strings) |
| Output | downloadURL (string) |

**Actions:**
1. Run a Prompt → Document Output enabled (PA_Template_V2.docx template)
2. SharePoint → "Create file" (save to ATLAS-PA-Outputs)
3. SharePoint → "Create sharing link"
4. Outlook → "Send email" (attach document to user)
5. Return value → downloadURL

---

## What This Replaces

| Retired Component | Replaced By |
|---|---|
| Azure Function App (atlas-pa-generator) | Copilot Studio Prompt Node + Document Output |
| Azure OpenAI resource | Built-in GPT in Copilot Studio |
| Azure Blob Storage (pa-sessions container) | Global Variables |
| GitHub Actions CI/CD pipeline | Not needed (no code) |
| function_app.py (1600+ lines Python) | Prompt instructions + 2 Power Automate flows |
| requirements.txt (6 dependencies) | Nothing |
| 5 HTTP endpoints | 0 endpoints |
| Monthly Azure cost (~$1-5) | $0 additional |

---

## What We Keep

| Component | Purpose |
|---|---|
| Copilot Studio agent | Orchestration, conversation, AI |
| Power Automate (2 flows) | File extraction + doc generation |
| SharePoint (CO+I Learning) | SCORM Course Analysis library |
| SharePoint (ATLAS-PA-Outputs) | Generated document archive |
| Outlook | Email delivery |
| Teams | User interface |

---

## SCORM Integration

### Discovery (Knowledge Source)
- Agent searches the Knowledge source when user picks "SCORM library"
- User describes topic → agent finds matching Course Analysis Report
- Grounded answer cites the specific document

### Extraction (ExtractText Flow)
- Once the right document is identified, ExtractText flow gets the full text
- Full document content feeds into the PA Extraction prompt
- For "mixed" path: SCORM text + user's pasted text are concatenated

---

## Security & Compliance

| Concern | How It's Handled |
|---------|-----------------|
| Data residency | All within Microsoft tenant — no third-party services |
| Authentication | Microsoft Entra ID (user's own credentials) |
| SharePoint access | Respects user's existing permissions |
| No secrets/keys | No API keys, connection strings, or stored credentials |
| No custom code | Nothing to audit, patch, or maintain |

---

## Risks & Mitigations

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| Document Output discontinued | Low | Production-ready preview; Azure build exists as fallback |
| Large document token limits | Medium | Use SharePoint link path (extract in flow, not agent) |
| SharePoint SCORM permissions | Low | Test Knowledge source access early (Phase 4) |
| Prompt quality differs from Azure | Low | Same prompt instructions, same GPT model family |

---

## Milestone Tracker

See: `MILESTONES_STUDIO.md`
