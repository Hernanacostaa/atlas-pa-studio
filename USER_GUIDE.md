# ATLAS PA Agent — User Guide

## What Is This?

The ATLAS PA Agent creates **Practical Activity Worksheets** from your source documents. Just tell the agent what you need in Microsoft Teams, and it generates a formatted Word document — complete with all 18 PA fields filled in.

---

## How to Use

### Step 1: Start a Conversation

Open **Microsoft Teams** and find the **ATLAS PA Agent**. Say something like:

- "Create a PA about generator maintenance"
- "I need a Practical Activity for CDU installation"
- "Help me build a PA worksheet"

### Step 2: Choose Your Content Source

The agent will ask how you'd like to provide source content:

| Option | When to Use |
|--------|------------|
| **Own content** | You have an SOP, MOP, or training document to paste or link |
| **SCORM library** | You want to use a Course Analysis Report from the library |
| **Both** | You want to combine SCORM content with your own material |

### Step 3: Provide Your Content

Depending on what you chose:

- **Own content:** Paste text directly into chat, OR provide a SharePoint link to a document
- **SCORM library:** Describe the topic (e.g., "CDU introduction" or "generator maintenance") and the agent finds the matching course document
- **Both:** Describe the SCORM topic AND paste/link your additional content

### Supported Input Combinations

| What You Provide | Works? | Notes |
|-----------------|--------|-------|
| Pasted text only | ✅ | Paste your SOP/MOP text directly into chat |
| One SharePoint link | ✅ | Agent extracts text from the document |
| Pasted text + one link | ✅ | Both sources combined into one PA |
| SCORM topic description | ✅ | Agent finds matching Course Analysis Report |
| SCORM + pasted text | ✅ | Combines SCORM content with your material |
| Multiple files/links | ⬜ Coming soon | Currently supports one file at a time |

### Step 4: Review the Preview

The agent extracts all 18 PA fields and shows you a preview card:

- **PA Title** — Name of the Practical Activity
- **Duration** — How long the activity takes
- **Target Audience** — Who this PA is for
- **Activity Steps** — The step-by-step procedure
- **What Is Needed** — PPE, tools, safety guidelines
- *(and 13 more fields)*

### Step 5: Request Edits (Optional)

If anything needs changing, just tell the agent:

- "Change the Duration to 2 hours"
- "Update the Target Audience to include Level 2 technicians"
- "Add steel-toed boots to the Required PPE"
- "Remove the last activity step"

The agent updates the field and shows the updated preview. Repeat until you're satisfied.

### Step 6: Generate the Document

When you're happy with the preview, say:

- "Generate"
- "Create the document"
- "Looks good, generate it"

The agent will:
1. Generate a formatted Word document (.docx)
2. Save it to SharePoint (ATLAS-PA-Outputs)
3. Email you a copy with the document attached
4. Show a download link in chat

---

## The 18 PA Fields

| # | Field | Description |
|---|-------|-------------|
| 1 | PA Title | Name of the Practical Activity |
| 2 | PA Subtitle | Brief description under the title |
| 3 | PA Document Label | Always "Practical Activity Worksheet" |
| 4 | Course Reference | Source document reference number |
| 5 | Authors | Who created this PA |
| 6 | Contributors | Additional contributors |
| 7 | Last Updated | Date of last update |
| 8 | Target Audience | Who should complete this PA and prerequisites |
| 9 | Duration | Estimated time to complete |
| 10 | Activity Description | Overview of what learners will do |
| 11 | Trainer Guidelines | Instructions for the facilitator |
| 12 | Desired Learning Outcome | Objectives, training opportunities, task types |
| 13 | What Is Needed | Safety guidelines, PPE, tooling, equipment |
| 14 | Skills-Based Learning Objectives | What learners will be able to do after completion |
| 15 | Documentation & References | Related SOPs, manuals, reference cards |
| 16 | Activity Steps | Step-by-step procedure with checkboxes |
| 17 | Validation | Criteria for successful completion |
| 18 | Notes | Additional information |

---

## Tips

- **Be specific** when describing your topic — the more detail you give, the better the extraction
- **Paste full text** rather than summaries — the agent extracts what it needs
- **Use the edit loop** — it's faster to tweak fields than to regenerate from scratch
- **Don't fabricate** — if the agent can't find information in your source, it marks fields as "TBD" rather than making things up
- **SharePoint links** work best when they point directly to .docx, .pdf, or .pptx files

---

## Output

Your generated PA document includes:
- ✅ Formatted Word document (.docx) with table layout, shading, and borders
- ✅ All 18 fields filled from your source content
- ✅ Bullet points, checkboxes (☐), and multi-line content preserved
- ✅ Fully editable — you can make manual changes after generation
- ✅ Saved to SharePoint for archiving
- ✅ Emailed to you as an attachment

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Agent asks for source text when using SCORM | Say "SCORM only" or set source text to "N/A" |
| Fields show "TBD" | The source document didn't contain that information — fill in manually |
| Document formatting looks off | Try regenerating — the LLM occasionally varies output |
| Can't find the SCORM course | Try different keywords — the agent searches by topic description |
| Didn't receive email | Check spam/junk folder; verify your email is correct |

---

## Where Things Are Stored

| What | Location |
|------|----------|
| Generated PA documents | SharePoint: ATLAS-PA-Outputs |
| SCORM Course Analysis Reports | SharePoint: CO+I Learning / Agents / Course Analysis Reports V3 |
| Your conversation history | Microsoft Teams chat (auto-saved) |
