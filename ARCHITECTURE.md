# PA Studio — Current Architecture (July 25, 2026)

## Overview

PA Studio generates Practical Activity Worksheets from SCORM Course Analysis Reports, pasted source content, and document links.

The live solution is built with **Copilot Studio + Power Automate** and uses **SharePoint Knowledge** plus **Work IQ** for content retrieval. There are **zero Azure services, zero custom APIs, and zero custom code** in the active architecture.

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| User channels | Microsoft Teams, M365 Copilot, Copilot Studio test chat | Where users interact with PA Studio |
| Orchestration | Copilot Studio agent | Routes requests, controls topic entry, and manages the conversation |
| SCORM search | SharePoint Knowledge source | Lets the orchestrator search the SCORM library and retrieve the selected course content |
| URL reading | Work IQ | Reads public URLs and internal SharePoint links on behalf of the orchestrator |
| Extraction | `ExtractPA` prompt (GPT-5 Reasoning) | Extracts 17 PA fields using field-by-field rules |
| Preview formatting | `FormatPreview` prompt (GPT-4.1 mini) | Converts JSON into a readable emoji-labeled preview |
| Editing | `EditPA` prompt | Applies user-requested changes to the current JSON |
| Document generation | `GeneratePA` prompt + Document Output | Fills the PA template and returns a Word document |
| Automation | `ATLAS-PA-GenerateDoc` flow | Parses JSON, runs `GeneratePA`, saves the file, creates a link, and emails the user |
| Storage | SharePoint Online | Hosts the SCORM library and the generated PA output library |
| Backup search | `SearchSCORM` flow | Built and retained as a backup, but not used in the current runtime path |

---

## Current Operating Principles

### 1. Constrained orchestrator
The orchestrator is intentionally narrow. It handles:
- SCORM Knowledge search
- URL content reading through Work IQ
- routing everything into the **Create PA** topic

It does **not** perform extraction, preview, editing, or document generation itself.

### 2. Tool lockdown
All prompts and flows are set to **Only when referenced by topics or agents**. In practice, this means the orchestrator cannot call them directly. Its only safe path is to pass full source content into the topic.

### 3. Topic-first execution
Every successful run goes through the **Create PA** topic. The topic owns:
- extraction
- preview
- edit loop
- document generation
- delivery
- clean conversation ending

### 4. Pass raw content, not summaries
The orchestrator passes **all raw content** into the topic through the topic input. It does not summarize, filter, shorten, or reorganize the source text before handoff.

### 5. Deterministic input handling
Inside the topic, input handling is explicit:
- if content is already present, continue
- if content is missing, ask the user to paste it
- if pasted content is too short, fail validation and ask for more detail

---

## Supported Content Paths

| Content path | How it works now |
|-------------|------------------|
| SCORM search | Orchestrator searches the Knowledge source, shows top matches, user selects one, and the full course content is passed into the topic |
| Pasted content | User pastes text directly, and the orchestrator passes it into the topic |
| Public link | Work IQ reads the URL content and passes it into the topic |
| SharePoint link | Work IQ reads the internal document content and passes it into the topic |
| Multi-link | The orchestrator combines content from multiple URLs before calling the topic |
| Mixed input | SCORM content and additional user content are concatenated before extraction |

**Topic input name:** The orchestrator passes **SourceContent**, which is stored inside the topic as **`SearchQuery`**.

---

## End-to-End Flow

```text
USER
  │
  ├── Teams
  ├── M365 Copilot
  └── Copilot Studio test chat
  │
  ▼
PA STUDIO ORCHESTRATOR
  │
  ├── If user mentions a course, topic, or keywords:
  │     1. Search SharePoint Knowledge (SCORM library)
  │     2. Present the top 5 matching courses with course name + course ID
  │     3. Wait for user selection
  │     4. Retrieve the full course content
  │     5. Call Create PA topic with SourceContent
  │
  ├── If user pastes content:
  │     1. Call Create PA topic with SourceContent
  │
  ├── If user provides one or more links:
  │     1. Read content through Work IQ
  │     2. Combine content if needed
  │     3. Call Create PA topic with SourceContent
  │
  └── If user asks for a PA but gives no content:
        Ask whether to search SCORM or use pasted/linked content
  │
  ▼
CREATE PA TOPIC
  │
  ├── Trigger
  ├── Condition: Is SearchQuery blank?
  │   ├── No: Ask whether to add more content, then continue
  │   └── Yes: Ask user to paste content
  │             └── Validate input (Len < 50 = reject and re-ask)
  │
  ├── ExtractPA
  ├── Set paFieldsJSON
  ├── FormatPreview
  ├── Show emoji-labeled preview
  │
  ├── Generate?
  │   ├── Yes → ATLAS-PA-GenerateDoc flow
  │   │         ├── Parse JSON
  │   │         ├── Run GeneratePA
  │   │         ├── Convert contentBytes to binary
  │   │         ├── Save to SharePoint
  │   │         ├── Create sharing link
  │   │         └── Email + return URL
  │   │
  │   └── No → Edit?
  │             ├── Yes → EditPA → update paFieldsJSON → FormatPreview → preview loop
  │             └── Other → Ask user to choose Generate or Edit → loop
  │
  ▼
END OF CONVERSATION
  └── Clean close with a "create another?" option
```

---

## Copilot Studio Components

### 1. Agent instructions

The orchestrator follows four core routes:
1. **Course or topic request** → search Knowledge, present top 5 matches, wait for selection, pass full content to the topic
2. **User-provided content** → pass the content directly to the topic
3. **User-provided link** → read the link content, then pass it to the topic
4. **No content yet** → ask whether to search SCORM or use pasted/linked content

**Non-negotiable rule:** Always route through the **Create PA** topic and pass **all raw content**.

### 2. Knowledge source

| Setting | Current value |
|---------|---------------|
| Type | SharePoint Knowledge source |
| Site | `https://microsoft.sharepoint.com/teams/COILearning` |
| Folder | `/Agents/Course Analysis Reports V3` |
| Content | ~300 SCORM Course Analysis `.doc` files |
| Purpose | SCORM search and full-content handoff through the orchestrator |
| Note | Files above 7 MB may be silently ignored without an M365 Copilot license |
| File cap | 1,000 files per agent |

### 3. Topic: Create PA

**Design:** One deterministic topic that handles the full generation pipeline.

**Current topic logic**
```text
Trigger → Condition (SearchQuery blank?)
├── Not blank (orchestrator filled): Additional content? → Yes/No → ExtractPA
└── Blank (manual): "Paste your content" → Input validation (Len < 50) → ExtractPA
→ Set paFieldsJSON → FormatPreview → Preview message → Generate/Edit choice
├── Generate: GenerateDoc flow → email + download link → End
├── Edit: EditPA → update paFieldsJSON → FormatPreview → loop to preview
└── Other: "Please select Generate or Edit" → loop
```

### 4. Prompt: ExtractPA

| Setting | Current value |
|---------|---------------|
| Model | GPT-5 Reasoning |
| Purpose | Extract the PA content from raw source text |
| Output | JSON for **17 extracted fields** |
| Rules | Field-by-field extraction rules ported from the retired Azure Function |
| Safeguard | No fabrication |
| Activity steps | Facilitator-level `ActivitySteps`, not source text dumped verbatim |

**Important:** The document still includes the fixed label **"Practical Activity Worksheet"**, but that label is not treated as one of the extracted inputs.

### 5. Prompt: EditPA

| Setting | Current value |
|---------|---------------|
| Inputs | `CurrentJSON`, `EditRequest` |
| Purpose | Update the existing PA JSON without redoing the full extraction |
| Output | Revised JSON |

### 6. Prompt: FormatPreview

| Setting | Current value |
|---------|---------------|
| Model | GPT-4.1 mini |
| Purpose | Turn the JSON into a readable preview for chat |
| Output style | Emoji-labeled text preview |
| Role | Makes the preview readable across chat channels |

### 7. Prompt: GeneratePA

| Setting | Current value |
|---------|---------------|
| Inputs | 17 PA fields |
| Feature | Document Output enabled |
| Template | Uploaded PA template |
| Output | Base64 document payload for the `.docx` file |

### 8. Flow: ATLAS-PA-GenerateDoc

**Current flow sequence**
1. Parse JSON
2. Run `GeneratePA`
3. Convert the returned file bytes
4. Save the document to SharePoint
5. Create a sharing link
6. Email the user
7. Return the document URL to the topic

**Proven file-bytes formula**
```text
base64ToBinary(body('Run_a_prompt')?['responsev2']?['predictionOutput']?['documentOutput']?['contentBytes'])
```

### 9. Flow: SearchSCORM

`SearchSCORM` still exists, but it is **not used in the current architecture**. It is retained as a backup option only.

### 10. End of Conversation

After successful generation and delivery, the agent closes cleanly and offers the user a chance to create another PA.

---

## Data Handling and State

| Item | Current behavior |
|------|------------------|
| Topic input | `SourceContent` from the orchestrator, stored internally as `SearchQuery` |
| Optional added notes | Concatenated with `SearchQuery` before extraction |
| Working state | `paFieldsJSON` stores the current extracted or edited PA content |
| Preview source | `FormatPreview` reads `paFieldsJSON` |
| Generation source | The GenerateDoc flow parses `paFieldsJSON` and maps it into the 17 `GeneratePA` inputs |

**Proven Power Fx concatenation**
```text
Topic.SearchQuery & " ADDITIONAL NOTES: " & Topic.AdditionalContent
```

---

## What This Replaces

| Retired / avoided approach | Current replacement |
|---------------------------|---------------------|
| Azure Functions | Copilot Studio prompts + Power Automate |
| Custom code services | Native Copilot Studio + Power Automate components |
| Direct orchestrator tool execution | Topic-only tool execution with lockdown |
| Older preview formatting experiments | `FormatPreview` emoji-text formatting |
| Separate SCORM listing / text extraction runtime path | Knowledge search + orchestrator handoff |

---

## Security and Control Model

| Concern | How it is handled now |
|---------|------------------------|
| Rogue tool use | Prompts and flows are locked to topic/agent reference mode |
| Over-summarization | Agent instructions require raw content handoff |
| Link access | Work IQ reads public and internal links on the orchestrator's behalf |
| SharePoint access | Uses existing Microsoft permissions and connectors |
| HTTP to SharePoint | Avoided because tenant DLP blocks that pattern |
| Custom code risk | Removed from the active architecture |

---

## Known Platform Constraints

| Constraint | Limit / behavior | Impact |
|-----------|------------------|--------|
| Agent instructions | 8,000 characters | Keep orchestrator instructions tight |
| Conversation history | Last 10 turns visible | Long edit loops can lose early context |
| Cloud flow timeout | 100 seconds | Keep generation and delivery efficient |
| Connector payload | 5 MB | Large inputs or outputs can fail |
| Knowledge file size | 7 MB without M365 Copilot license | Oversized files may be silently ignored |
| Knowledge file cap | 1,000 files per agent | Current library fits |
| DLP | HTTP requests to SharePoint blocked | Must stay inside approved connectors and Work IQ |

---

## Known Blockers and Mitigations

For the full history, see [SCORM_INTEGRATION.md](SCORM_INTEGRATION.md).

| Blocker | Current understanding | Mitigation |
|---------|-----------------------|------------|
| Generative Answers output does not persist in topic variables | Data drops after the node finishes | Do not build the runtime path around Generative Answers output |
| Child agents cannot access Knowledge the same way as the parent | SCORM reading is unreliable in child paths | Keep SCORM search/read in the orchestrator |
| DLP blocks SharePoint HTTP requests | Direct HTTP workarounds are not viable | Use approved connectors and Work IQ |
| Legacy `.doc` files limit Power Automate options | Native extraction choices are limited | Let Knowledge handle SCORM reading |
| Orchestrator non-determinism | Instructions alone are not enough | Constrain the orchestrator and lock down tools |

---

## Status

- **Architecture state:** Current as of **July 25, 2026**
- **Delivery model:** Copilot Studio + Power Automate + Work IQ
- **Azure usage:** None in the active solution
- **Milestones:** **46/46 complete (100%) across 7 phases**

See also:
- [README.md](README.md)
- [USER_GUIDE.md](USER_GUIDE.md)
- [SCORM_INTEGRATION.md](SCORM_INTEGRATION.md)
- [MILESTONES.md](MILESTONES.md)
