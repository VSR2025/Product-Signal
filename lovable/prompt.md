# Lovable Prompt: LLM Trace Review Tool (MVP)

Build a single-page web application for reviewing LLM agent conversation traces to discover failure modes through manual error analysis.

## Core Purpose
Help me systematically review 98 agent conversation traces from a CSV file, annotate each with Pass/Fail/Defer + notes, discover patterns, and save my annotations. The goal is to answer: "What went wrong and why?"

## Data Source
- CSV file with columns: `trace_id`, `customer_persona`, `user_query`, `conversation_messages`, `tool_calls`
- `conversation_messages` format: `"USER: query | ASSISTANT: response | ASSISTANT: TOOL_CALL[ToolName] Description/Error | ..."`
- Delimiter is ` | ` (space-pipe-space)
- Roles can be USER, ASSISTANT, or AGENT
- Tool calls look like: `TOOL_CALL[ParseRequest] Analyzing your request...`
- Errors look like: `TOOL_CALL[GenRecipeArgs] Error: unable to generate...`
- Note: `trace_id` and `customer_persona` columns may be empty - handle gracefully

## MVP Features (Build These)

### 1. Trace Display (One at a Time)
Show ONE complete trace per screen with clear visual hierarchy:

**Always Visible:**
- User query (prominent, large)
- Final agent response (prominent)
- Metadata panel (trace info)

**Collapsible Sections (Collapsed by Default):**
- System prompt (if you extract/infer it)
- Agent reasoning steps (intermediate "Let me help..." messages)
- Tool calls with clear success/error indicators

**Visual Treatment:**
- Tool call errors: Red background + ❌ icon + bold text (ALWAYS expanded, never collapsed)
- Tool call success: Green ✓ checkmark, normal text
- User messages: White background
- Agent reasoning: Light blue background
- Tool calls section: Light yellow background

### 2. Metadata Panel (Top of Screen)
Display for current trace:
- Trace ID (generate as TRACE_001, TRACE_002... if CSV column empty)
- Customer Persona (show "N/A" if CSV column empty)
- Review Status: Not Reviewed | Pass | Fail | Deferred (with color coding)
- Reviewer: [Current user name - can hardcode "Reviewer" for MVP]
- Timestamp: [When reviewed, if reviewed]

### 3. Feedback Panel (Prominent, Easy Access)
**Three Large, Distinct Buttons:**
- [PASS] - Green
- [FAIL] - Red  
- [DEFER] - Yellow/Orange

**Notes Field:**
- Multi-line text input
- Placeholder: "What went wrong? What did you observe?"
- Autocomplete: As user types, suggest completions from their previous notes
- Example: If they typed "tone mismatch" on trace 5, typing "tone..." on trace 20 suggests "tone mismatch"

**Save Behavior:**
- Click Pass/Fail/Defer → Save immediately + show "✓ Saved" indicator
- Type in notes → Auto-save on blur (when they click away or navigate)
- Always show save status: "Saving..." or "✓ Saved"

### 4. Navigation Controls
**Buttons:**
- [◄ Previous] - Go to previous trace
- [Next ►] - Go to next trace
- [Skip/Defer] - Mark as Deferred and move to next
- [Jump to Trace #] - Text input + Go button

**Progress Indicator:**
Show clearly: "Trace 5 of 98"

**Progress Stats:**
- Total: 98 traces
- Reviewed: 23 (23.5%)
  - Pass: 15
  - Fail: 6
  - Deferred: 2
- Remaining: 75

### 5. Sorting & Filtering (Top of Screen Controls)

**Sort By Dropdown:**
- Sequential (1, 2, 3...)
- Errors First (show traces with tool errors first) ← **Default**
- Random
- Not Reviewed First

**Filter Checkboxes:**
- ☐ Has tool errors (47 traces)
- ☐ Review status: ☐ Not reviewed ☐ Pass ☐ Fail ☐ Deferred

When filters active, show: "Showing: 47 of 98 traces"

### 6. Edit Past Labels
**Allow user to:**
- Navigate back to ANY previously reviewed trace
- Change their Pass/Fail/Defer decision
- Edit their notes
- See edit history: "Last edited: 2025-10-28 (Pass → Fail)"

### 7. Data Persistence
**Save annotations to:**
- Browser localStorage (simple MVP approach) OR
- JSON file download after session OR
- Your choice of simple storage

**Data Structure to Save:**
```json
{
  "trace_id": "TRACE_001",
  "status": "fail",
  "notes": "Agent called wrong tool - should use check_calendar",
  "reviewer": "Reviewer",
  "reviewed_at": "2025-10-28T10:30:00Z",
  "revision_history": [
    {
      "timestamp": "2025-10-28T10:30:00Z",
      "action": "created",
      "status": "fail"
    }
  ]
}
```

## Parsing Requirements

**Critical:** Parse the `conversation_messages` string correctly:

1. Split on ` | ` delimiter
2. For each segment, detect:
   - Role: USER, ASSISTANT, or AGENT
   - Type: regular message OR tool call (contains "TOOL_CALL[")
   - If tool call: Extract tool name from brackets
   - If error: Detect "Error:" keyword
   - Content: The actual message text

3. Display in structured sections:
   - Group all USER messages together
   - Group ASSISTANT/AGENT reasoning (messages without tool calls)
   - Group TOOL_CALL messages in their own section
   - Highlight errors prominently

## UI/UX Priorities

**Make it:**
- **Fast to use:** Review a trace in <2 minutes
- **Easy to scan:** Important info (query, errors, response) immediately visible
- **Hard to lose work:** Auto-save everything, show save status
- **Forgiving:** Defer button for unclear cases, can edit past decisions
- **Focused:** One trace at a time, no distractions

**Design Philosophy:**
- Clean, minimal interface
- Emphasis on readability (large fonts for key content)
- Strong visual hierarchy (collapsible sections, color coding)
- Clear error indicators (you can't miss tool errors)

## Technical Preferences

**Framework:** Your choice (React, Vue, whatever works best in Lovable)

**Styling:** 
- Modern, clean design
- Good spacing and typography
- Responsive (works on laptop screens)
- Color coding for status (green=pass, red=fail/error, yellow=defer)

**CSV Loading:**
- File upload button to load CSV
- Parse CSV client-side
- Show error if parsing fails

## What NOT to Build (Out of Scope for MVP)

❌ Structured failure mode checkboxes (user discovers patterns through notes first)
❌ Clustering or semantic search
❌ Keyboard shortcuts (nice-to-have, not MVP)
❌ Multi-user collaboration
❌ Integration with external tools
❌ Export to multiple formats
❌ Advanced analytics/dashboards

## Success Criteria

✅ Can load a CSV with 98 traces
✅ Can navigate through all traces (Previous/Next/Jump)
✅ Can parse and display conversation_messages correctly
✅ Can see tool errors highlighted in red
✅ Can mark traces as Pass/Fail/Defer
✅ Can write notes with autocomplete suggestions
✅ Can edit past annotations
✅ Can sort by "Errors First" to prioritize review
✅ Can filter to show only traces with errors
✅ Progress tracked (know how many reviewed)
✅ All annotations saved (don't lose work)

## Example Use Case

**Reviewer workflow:**
1. Upload traces.csv
2. Tool defaults to "Errors First" sorting (smart!)
3. See Trace 1: User asks for recipe, agent calls wrong tool, error occurs
4. Reviewer expands "Tool Calls" section, sees error clearly highlighted
5. Clicks [FAIL], types notes: "Called GenRecipeArgs instead of GetRecipes"
6. Clicks [Next ►], auto-saved, moves to Trace 2
7. After 30 traces, notices pattern: types "called wrong tool" and autocomplete suggests full note
8. Can jump back to Trace 5 if they remember something
9. After reviewing all traces, can export annotations or continue session later

---

**Build this MVP. Focus on making trace review fast, pleasant, and reliable. The goal is discovering patterns through systematic manual review.**
