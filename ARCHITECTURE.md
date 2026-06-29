# ATLAS PA Agent — Studio-Only Architecture V5

## Overview

The ATLAS PA Agent generates Practical Activity Worksheets from source documents.  
This architecture uses **only Copilot Studio + Power Automate** — zero Azure Functions, zero Azure OpenAI, zero custom code.

**Last updated:** Based on deep research across Microsoft Agent Academy, CPSAgentKit, and official docs.

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| User Interface | Microsoft Teams | Where users interact with the agent |
| Orchestration | Copilot Studio (Generative Orchestration) | Routes conversation, manages state, runs AI |
| AI / LLM | GPT-5 Reasoning (Copilot Studio Prompt Builder) | PA field extraction (400K token context) |
| State Management | Single JSON string (bot-scoped variable) | All 18 PA fields in one variable, parsed when needed |
| Knowledge | SharePoint RAG (Knowledge Source) | SCORM content understanding (not primary matching) |
| SCORM Matching | Hybrid: Flow lists files + Prompt picks match | Reliable file selection from 100+ docs |
| Automation | Power Automate (2-3 flows) | File handling + document generation |
| Document Generation | "Populate a Microsoft Word template" (GA connector) | Word doc from content control template |
| File Storage | SharePoint Online | Source files (SCORM) + output archive |
| Delivery | Outlook + Teams | Email with attachment + chat download link |

---

## Research-Validated Design Decisions

### Decision 1: Single JSON variable instead of 18 globals
**Source:** CPSAgentKit anti-patterns documentation

❌ **Old plan:** 18 separate global variables (gblPATitle, gblDuration, etc.)  
✅ **New plan:** One bot-scoped variable `bot.paFieldsJSON` containing all fields as a JSON string

**Why:** Research found that using global variables as an invocation contract is a known anti-pattern:
> "The planner may narrate success while the topic still sees blank variables."

The JSON string is parsed into individual values only inside the GenerateDoc flow (using Power Automate's "Parse JSON" action).

### Decision 2: "Populate a Microsoft Word template" instead of Document Output
**Source:** Architecture review — risk reduction

❌ **Old plan:** Document Output (Preview) generates .docx inside a Prompt Builder node  
✅ **New plan:** Power Automate's "Populate a Microsoft Word template" connector (GA)

**Why:** Document Output is a Preview feature that could be removed. The Word template connector is:
- **GA (Generally Available)** — stable, not going anywhere
- **Direct file output** — no `binary()` formula needed to extract bytes
- **Template in SharePoint** — portable, moves with Solutions, no re-upload bugs
- **Well-documented** — thousands of production examples

The Prompt Builder still extracts 18 fields as JSON. The flow then parses the JSON and maps each field to a Word content control in the template.

**Template format:** Word Content Controls (Developer tab → Plain Text Content Control), NOT `{{curly braces}}`.

### Decision 3: Hybrid SCORM matching (not Knowledge alone)
**Source:** CPSAgentKit knowledge constraints

❌ **Old plan:** Knowledge source alone finds the right SCORM file  
✅ **New plan:** Knowledge for understanding + Flow+Prompt for reliable matching

**Why:** Knowledge retrieval is non-deterministic and has limitations:
> "Identical queries can return different results"
> "Without M365 Copilot license: SharePoint files limited to 7MB (silently ignored)"

### Decision 4: Everything stays in ONE topic
**Source:** CPSAgentKit pipeline patterns

The entire PA generation flow (collect → extract → preview → edit → generate) stays in a single topic so variables remain in scope. No topic handoffs during the pipeline.

### Decision 5: Pass file directly to prompt (skip separate text extraction)
**Source:** GPT-5/GPT-4.1 multimodal capabilities

For user-provided SharePoint files, we can pass the file directly to the extraction prompt as a document input. GPT-5 reads the document natively — no separate text extraction step needed.

Text extraction flow is still needed for SCORM files (where we get file content from SharePoint programmatically).

### Decision 6: Adaptive Cards for preview
**Source:** Copilot Studio capabilities research

Use Adaptive Cards instead of plain text messages for the preview display. Structured card layout for 18 fields is more readable and professional.

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
  │   • Used for SCORM content understanding                     │
  │   • NOT used as sole matching mechanism                      │
  └──────────────────────────────────────────────────────────────┘
  │
  ├── TOPIC: Create PA (single topic, all steps) ───────────────┐
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
  │   [Collect Input]                                             │
  │     user: paste text or SP link                               │
  │     scorm: topic description                                  │
  │     mixed: both                                               │
  │         │                                                     │
  │         ▼                                                     │
  │   [SCORM path: Call Action → ListSCORM flow]                  │
  │     Returns file list → Prompt picks best match               │
  │     → Call Action → ExtractText flow                          │
  │         │                                                     │
  │         ▼                                                     │
  │   [Call Action: PA Extraction Prompt]                         │
  │     Input: source text (or file as document input)            │
  │     Model: GPT-5 Reasoning (400K context)                     │
  │     Output: JSON string with 18 PA fields                     │
  │         │                                                     │
  │         ▼                                                     │
  │   [Set Variable] ── bot.paFieldsJSON = prompt output          │
  │         │                                                     │
  │         ▼                                                     │
  │   [Message Node: Adaptive Card] ── Preview all fields         │
  │     "Say 'generate' or tell me what to change"                │
  │         │                                                     │
  │         ▼                                                     │
  │   [Edit Loop via Slot Filling]                                │
  │     User: "change Duration to 2 hours"                        │
  │     Agent parses JSON → updates field → re-serializes         │
  │     Re-displays Adaptive Card preview                         │
  │     (repeats until user says "generate")                      │
  │         │                                                     │
  │         ▼                                                     │
  │   [Call Action: GenerateDoc Flow]                             │
  │     Input: bot.paFieldsJSON                                   │
  │     Flow parses JSON → fills Word template via connector       │
  │     Saves to SharePoint → emails user                         │
  │     Returns: downloadURL                                      │
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
| Role | Content understanding + supplementary search (NOT primary file matching) |

### 2. Topic: Create PA

**Orchestration mode:** Generative  
**Trigger:** User expresses intent to create a PA  
**Scope:** Entire pipeline in ONE topic (no topic handoffs)

**Nodes used:**

| Node Type | Count | Purpose |
|-----------|-------|---------|
| Question | 2-3 | Content source, topic description, paste text |
| Condition | 1-2 | Branch on user/scorm/mixed |
| Call Action | 3-4 | ListSCORM flow, ExtractText flow, Extraction prompt, GenerateDoc flow |
| Set Variable | 1 | Store JSON string (bot.paFieldsJSON) |
| Message | 2 | Adaptive Card preview, delivery message |

### 3. Prompt: PA Field Extraction

| Setting | Value |
|---------|-------|
| Type | Prompt Builder (Call Action) |
| Model | GPT-5 Reasoning (400K token context) |
| Input | sourceText (string) or file (document input for multimodal) |
| Output | JSON (18 PA fields) |
| Content Moderation | Set to Low (domain-specific training content may trigger false positives) |

**Prompt instructions include:**
- PA is a FACILITATOR-LEVEL training document, not an SOP copy
- ActivitySteps: 10-12 summarized phases, not every sub-step verbatim
- Never fabricate PPE, tools, fault codes, or steps — use TBD/N/A
- Duration: extract from source or TBD, never estimate
- WhatIsNeeded: 4 categories (Safety Guidelines, Required PPE, Tooling, Additional Equipment)
- If source has Training Plan learning objectives, extract directly into Skills field
- Output: strict JSON with all 18 field keys

### 4. State Management

**Storage:** Single bot-scoped variable `bot.paFieldsJSON`  
**Format:** JSON string containing all 18 PA fields  
**Lifetime:** Entire conversation session  
**Not stored in:** LLM token window, Dataverse, or any external database

```json
{
  "PA_Title": "...",
  "PA_Subtitle": "...",
  "PA_DocumentLabel": "Practical Activity Worksheet",
  "CourseReference": "...",
  "Authors": "...",
  "Contributors": "...",
  "LastUpdated": "...",
  "TargetAudience": "...",
  "Duration": "...",
  "ActivityDescription": "...",
  "TrainerGuidelines": "...",
  "DesiredLearningOutcome": "...",
  "WhatIsNeeded": "...",
  "SkillsBasedLearningObjectives": "...",
  "DocumentationAndReferences": "...",
  "ActivitySteps": "...",
  "Validation": "...",
  "Notes": "..."
}
```

**Edit loop:** Agent parses JSON → modifies requested field → re-serializes → updates bot.paFieldsJSON

### 5. Document Generation (Word Template Connector)

| Setting | Value |
|---------|-------|
| Connector | "Populate a Microsoft Word template" (GA) |
| Template | PA_Template_V2.docx (stored in SharePoint) |
| Template location | SharePoint: ATLAS-PA-Outputs/Templates/ |
| Placeholder format | Word Content Controls (Plain Text, Developer tab) |
| Output | .docx file content (direct — no formula needed) |
| Access | Power Automate flow action |

**Template setup:** Open PA_Template_V2.docx in Word → Developer tab → Plain Text Content Control for each field. Each control's Title/Tag matches the JSON field name (PATitle, CourseReference, etc.).

**Template structure:** Same table layout, shading, column widths, and formatting as current PA output — with content controls where dynamic content is injected.

---

## Power Automate Flows (2-3 total)

### Flow 1: ExtractText

| Property | Value |
|----------|-------|
| Trigger | Copilot Studio |
| Input | fileURL (string) |
| Output | plainText (string) |

**Actions:**
1. SharePoint → "Get file content" (download the file)
2. Pass to prompt as document input (GPT reads natively) OR AI Builder "Extract text"
3. Return value → plainText

### Flow 2: GenerateDoc

| Property | Value |
|----------|-------|
| Trigger | Copilot Studio |
| Input | paFieldsJSON (string — the full JSON) |
| Output | downloadURL (string) |

**Actions:**
1. Parse JSON → extract 18 individual field values
2. "Populate a Microsoft Word template" → PA_Template_V2.docx from SharePoint
   - Map each parsed field to corresponding content control
   - Direct file output (no binary formula needed)
3. SharePoint → "Create file" (save filled doc to ATLAS-PA-Outputs)
4. SharePoint → "Create sharing link"
5. Outlook → "Send email" (attach document to user)
6. Return value → downloadURL

**Note:** Cloud flow timeout is 100 seconds. Place email/archive steps AFTER the return step if needed to avoid timeout.

### Flow 3: ListSCORM (for SCORM matching)

| Property | Value |
|----------|-------|
| Trigger | Copilot Studio |
| Input | none |
| Output | fileList (array of filenames) |

**Actions:**
1. SharePoint → "List folder" (Course Analysis Reports V3)
2. Select → extract filenames
3. Return value → fileList (array)

---

## What This Replaces

| Retired Component | Replaced By |
|---|---|
| Azure Function App (atlas-pa-generator) | Copilot Studio Prompt Node + Word Template Connector |
| Azure OpenAI resource | Built-in GPT-5 in Copilot Studio |
| Azure Blob Storage (pa-sessions container) | bot.paFieldsJSON variable |
| GitHub Actions CI/CD pipeline | Not needed (no code) |
| function_app.py (1600+ lines Python) | Prompt instructions + 2-3 Power Automate flows |
| requirements.txt (6 dependencies) | Nothing |
| 5 HTTP endpoints | 0 endpoints |
| Monthly Azure cost (~$1-5) | $0 additional |

---

## What We Keep

| Component | Purpose |
|---|---|
| Copilot Studio agent | Orchestration, conversation, AI |
| Power Automate (2-3 flows) | File handling + doc generation |
| SharePoint (CO+I Learning) | SCORM Course Analysis library |
| SharePoint (ATLAS-PA-Outputs) | Generated document archive |
| Outlook | Email delivery |
| Teams | User interface |

---

## SCORM Integration (Hybrid Approach)

### Discovery Layer (Knowledge Source)
- Agent CAN search Knowledge source for general SCORM understanding
- Helps answer questions about course content
- NOT relied upon for deterministic file matching

### Matching Layer (Flow + Prompt)
- ListSCORM flow returns all filenames from SharePoint folder
- Prompt node receives: file list + user's topic description
- Prompt picks the best match → returns filename
- Deterministic: same input always produces same file selection

### Extraction Layer (ExtractText Flow)
- Gets full document content from matched file
- Feeds into PA Extraction prompt
- For "mixed" path: SCORM text + user's pasted text concatenated

---

## Security & Compliance

| Concern | How It's Handled |
|---------|-----------------|
| Data residency | All within Microsoft tenant — no third-party services |
| Authentication | Microsoft Entra ID (user's own credentials) |
| SharePoint access | Respects user's existing permissions |
| No secrets/keys | No API keys, connection strings, or stored credentials |
| No custom code | Nothing to audit, patch, or maintain |
| PII | Never stored in global variables (bot variable cleared per session) |

---

## Known Platform Constraints

| Constraint | Limit | Impact |
|-----------|-------|--------|
| Agent instructions | 8,000 characters | Keep instructions focused |
| Conversation history | Last 10 turns visible to orchestrator | Long edit loops may lose early context |
| Cloud flow timeout | 100 seconds | Return download URL before doing email/archive |
| Connector payload | 5 MB (public cloud) | Large docs may need chunking |
| Knowledge source files | 7 MB without M365 Copilot license | Some SCORM files may be silently ignored |
| Knowledge source max | 500 objects per agent, 1000 files | Well within our 100+ files |
| Prompt context (GPT-5 Reasoning) | 400K tokens (~300K words) | Handles any document size |

---

## Risks & Mitigations

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| Multiline content in content controls | Medium | Test in Phase 1; Word preserves formatting within controls |
| Knowledge retrieval non-deterministic | Medium | Hybrid approach: flow+prompt for reliable matching |
| Large document exceeds payload | Low | 5MB limit; most SOPs are under 1MB |
| Flow timeout on doc generation | Medium | Return URL first, do email/archive after return step |
| Content control tag naming mismatch | Low | Document all 18 tag names; verify during template setup |

---

## References

- Microsoft Docs: "Populate a Microsoft Word template" connector
- CPSAgentKit: Anti-patterns, Pipeline Patterns, Knowledge Constraints
- Microsoft Docs: Prompt Builder, Variables, Copilot Studio
- Microsoft Agent Academy: Document generation patterns

---

## Milestone Tracker

See: `MILESTONES.md`
