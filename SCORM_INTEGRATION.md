# SCORM Integration — Journey to Production

## Goal

Let users search the SCORM Course Analysis library (~300 .doc files), pick a course, and automatically create a PA from its content.

## Two Problems to Solve

1. **Find the course** — search 300 files and match user's query
2. **Read the course content** — extract text from .doc files for PA generation

---

## Solutions Attempted

| # | Solution | What it solved | Why it didn't work |
|---|----------|---------------|-------------------|
| 1 | **Orchestrator Knowledge search** | Both find + read | Orchestrator went rogue — skipped the topic, did its own thing |
| 2 | **Tighter instructions** | Tried to control orchestrator | Non-deterministic — can't guarantee behavior with instructions alone |
| 3 | **Generative Answers inside topic** | Find — worked | Read — data disappeared from variables after the node completed |
| 4 | **Various variable storage attempts** | Tried to capture Generative Answers output | Platform limitation — ephemeral by design, can't persist |
| 5 | **SearchSCORM Power Automate flow** | Find — ✅ fully working | Only solves finding, not reading .doc content |
| 6 | **Child agent with Knowledge** | Tried to read content from topic | Child agents can't access Knowledge source the same way the parent can |
| 7 | **Child agent completion settings** | Tried auto-populating output variable | Knowledge still returned nothing inside child agent |

---

## Core Tension

- The only thing that can **read .doc files** is the parent orchestrator's Knowledge source
- The orchestrator **can't be fully trusted** to follow instructions every time

---

## Solution: Constrain the Orchestrator

Lock down the orchestrator so it can only do one thing — read course content.

| Component | Who controls it | How |
|-----------|----------------|-----|
| **Find the course** | Topic (deterministic) | SearchSCORM flow — searches SharePoint, AI matches, returns results |
| **Read the course content** | Orchestrator (only job left) | Knowledge source reads .doc, passes content to topic |
| **Everything else** | Topic (deterministic) | Extract, preview, edit, generate, deliver |

**Key safeguard:** All prompts and flows locked to **"Only when referenced by topics"** — the orchestrator literally cannot call them directly. Its only available actions are search Knowledge and call the topic. This eliminates the rogue behavior.

---

## Platform Blockers Identified

| Blocker | Description |
|---------|-------------|
| **Generative Answers node** | Data doesn't persist in topic variables — ephemeral by design |
| **Child agent Knowledge** | Child agents can't access Knowledge source the same way parent orchestrator can |
| **DLP policy** | Tenant blocks "Send an HTTP request to SharePoint" connector |
| **Legacy .doc format** | Standard Power Automate connectors can't read/convert .doc files |
| **Orchestrator non-determinism** | Instructions alone cannot guarantee routing behavior |

---

## What's Working Today

- ✅ SearchSCORM flow — deterministic course search with fuzzy matching
- ✅ Tool lockdown — orchestrator can't call prompts/flows directly
- ✅ Parent orchestrator Knowledge — reliably reads .doc content
- ✅ Full PA pipeline — extract → preview → edit → generate → deliver
- ✅ Formatted preview — FormatPreview prompt renders clean output across all channels

---

## Status

- **Course search:** Solved (SearchSCORM flow)
- **Content retrieval:** Orchestrator Knowledge (constrained)
- **Next step:** Wire the handoff between topic (search) → orchestrator (read) → topic (process)
