# PA Studio — User Guide

## What Is This?

PA Studio creates Practical Activity Worksheets from SCORM course content, pasted source material, and document links. It guides you through search, preview, editing, generation, and delivery in one conversation.

You can use PA Studio in:
- Microsoft Teams
- Microsoft 365 Copilot
- Copilot Studio test chat

---

## Welcome Message

```text
Hello, I'm PA Studio! I create Practical Activity Worksheets.

To get started, you can:
• Tell me a course name or topic to search
• Paste your source content directly
• Share a link to a document

What would you like to do?
```

---

## How to Use

### Step 1: Start the request

You can begin with any of the following:
- a course name
- a topic or keywords
- pasted source text
- a public link
- a SharePoint link
- more than one link
- SCORM content plus your own notes

If you ask for a PA without giving any content, PA Studio asks:
> Would you like me to search for a SCORM course, or do you have content to paste/provide a link?

### Step 2: Choose or provide the content

#### Option A: Search the SCORM library
If you mention a course, topic, or keywords, PA Studio:
1. searches the SCORM Knowledge library
2. shows the **top 5 matching courses**
3. includes the **course name and course ID** for each match
4. waits for you to choose one
5. sends the **full selected course content** into the Create PA topic

#### Option B: Paste your own content
Paste the source text directly into chat. PA Studio passes that text into the Create PA topic as-is.

#### Option C: Share a link
Send a public link or a SharePoint link. PA Studio reads the content and passes the full text into the Create PA topic.

#### Option D: Combine sources
PA Studio can also combine:
- multiple links
- SCORM content + pasted notes
- SCORM content + additional instructions
- pasted text + linked content

### Step 3: Input validation

If the topic needs manual pasted content and the message is too short, PA Studio asks you to provide more detail before it continues.

---

## Supported Input Patterns

| What you provide | Supported? | What PA Studio does |
|------------------|-----------|---------------------|
| Course name, topic, or keywords | ✅ | Searches SCORM Knowledge, shows top 5, waits for your selection |
| Pasted text only | ✅ | Sends the text straight to the Create PA topic |
| Public link | ✅ | Reads the page content and passes it into the topic |
| SharePoint link | ✅ | Reads the document content and passes it into the topic |
| Multiple links | ✅ | Combines the content before extraction |
| SCORM + extra notes | ✅ | Concatenates both sources before extraction |
| Pasted text + link | ✅ | Combines both sources before extraction |

---

## Preview and Edit Loop

After PA Studio extracts the content, it shows a readable preview in the chat.

### What the preview looks like

The preview is **emoji-labeled text**. Typical sections include:
- 🧾 PA Title
- ⏱️ Duration
- 👥 Target Audience
- 📝 Activity Description
- 🧰 What Is Needed
- 🪜 Activity Steps
- ✅ Validation
- 📌 Notes

### How to request changes

If you want to revise anything, say it plainly. For example:
- "Change the Duration to 2 hours"
- "Update the Target Audience to include Level 2 technicians"
- "Add steel-toed boots to What Is Needed"
- "Rewrite the Activity Steps to be more facilitator focused"

PA Studio applies the edit, rebuilds the preview, and lets you review it again. You can repeat this loop until the preview looks right.

### How to generate

When you are ready, say:
- "Generate"
- "Create the document"
- "Looks good, generate it"

---

## What Happens After You Generate

PA Studio will:
1. generate the Word document from the approved PA fields
2. save it to the **ATLAS-PA-Outputs** SharePoint library
3. create a sharing link
4. email you a copy
5. show the download link in the conversation

---

## Conversation Ending

After delivery, PA Studio ends the conversation cleanly and offers a **create another?** option so you can start a new PA without guessing the next step.

---

## Tips

- Use the full source text when possible; raw content works better than summaries
- Be specific when searching SCORM courses so the top 5 results are more accurate
- Use the edit loop instead of starting over
- If information is missing from the source, PA Studio should not invent it
- When adding extra notes, include them clearly so they can be merged with the main content

---

## Troubleshooting

| Issue | What to do |
|-------|------------|
| The agent asks for more content | Send a fuller source passage; very short pasted content is rejected |
| The SCORM course you want is not in the top 5 | Try different course keywords, a course ID, or a more specific topic |
| A section looks incomplete | Check whether the source content actually included that detail |
| A link does not work | Make sure the URL is reachable and you have permission if it is a SharePoint document |
| You did not receive the email | Check junk/spam and confirm the mailbox tied to your Microsoft account |

---

## Where Things Are Stored

| What | Location |
|------|----------|
| SCORM source library | SharePoint: `https://microsoft.sharepoint.com/teams/COILearning` → `/Agents/Course Analysis Reports V3` |
| Generated PA documents | SharePoint: `https://microsoft.sharepoint.com/sites/86dae876-a7f6-43da-824a-83a2c42644bb` → `/Shared Documents/ATLAS-PA-Outputs` |
| Conversation | Teams, M365 Copilot, or Copilot Studio chat history |
