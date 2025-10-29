# LLM Trace Review Tool - Complete Design Specification
## Based on Lesson 10: Interfaces for Continuous Human Review and Error Analysis

---

## 📋 EXECUTIVE SUMMARY

**Purpose:** Build a custom interface for reviewing LLM agent traces to discover failure modes through systematic error analysis.

**Core Goal:** Answer "Is my product getting better or just changing?"

**Design Philosophy:** Start minimal (error analysis), build extensibly (support full evaluation lifecycle), maintain backward compatibility.

---

## 🎯 PROJECT CONTEXT

### Your CSV Structure
- **File:** `traces.csv`
- **Columns:** `trace_id`, `customer_persona`, `user_query`, `conversation_messages`, `tool_calls`
- **Data Issues:**
  - `trace_id` and `customer_persona` columns are EMPTY (need to generate/populate)
  - `conversation_messages` format: `"USER: query | ASSISTANT: response | ASSISTANT: TOOL_CALL[Name] Error: ... | USER: followup"`
  - Role inconsistency: Sometimes "ASSISTANT", sometimes "AGENT"
  - Tool calls are inline within conversation (not separate)
  - Tool errors embedded in conversation text

### Conversation Format Details
```
Delimiter: " | "
Roles: USER, ASSISTANT, AGENT (inconsistent)
Tool calls: "TOOL_CALL[ToolName] Description/Error message"
Errors: "Error: unable to..." embedded inline
Multi-turn: Multiple back-and-forth exchanges per trace
```

---

## 🏛️ THE 8 HCI PRINCIPLES (Foundation)

All design decisions map to these core principles from Lesson 10:

| # | Principle | Design Implication |
|---|-----------|-------------------|
| 1 | **Visibility of System Status** | Show which trace is being reviewed, how many remain, feedback status |
| 2 | **Match Between System and Real World** | Use familiar terms from rubrics; make mappings intuitive |
| 3 | **User Control and Freedom** | Allow navigation (next/previous), undo, defer actions |
| 4 | **Consistency and Standards** | Reuse layouts, terminology, behaviors across traces |
| 5 | **Error Prevention** | Use distinct labeling controls; enforce requirements |
| 6 | **Recognition Rather Than Recall** | Show predefined failure modes visually to reduce cognitive load |
| 7 | **Flexibility and Efficiency of Use** | Offer shortcuts for experts; keep intuitive for novices |
| 8 | **Aesthetic and Minimalist Design** | Emphasize signal over noise; highlight inputs/outputs clearly |

---

## 📊 FEATURE INVENTORY: PHASE 1 (ERROR ANALYSIS - BUILD NOW)

### CATEGORY 1: CORE DISPLAY & TRACE PARSING

#### Feature 1.1: Parse Conversation Messages
**HCI Principle:** Aesthetic and Minimalist Design

**Problem from Lesson 10:**
> "Spreadsheets struggle with the complexity of LLM traces... Emails appear as raw text crammed into cells."

**Solution:**
Parse the `conversation_messages` string and display structured components:

**Parsing Logic:**
1. Split on delimiter: ` | `
2. Extract role (USER/ASSISTANT/AGENT)
3. Detect tool calls: `TOOL_CALL[...]`
4. Extract tool name from brackets
5. Detect errors: `Error:` keyword
6. Parse error message

**Display Structure:**
```
Component Type         | Always Visible? | Visual Treatment
--------------------- | --------------- | ----------------
System Prompt         | Collapsed       | Light gray background
User Query            | Expanded        | White background, bold
Agent Reasoning       | Collapsed       | Light blue background
Tool Calls            | Collapsed       | Yellow background
Tool Errors           | Expanded        | Red background + icon
Final Response        | Expanded        | White background
```

#### Feature 1.2: Visual Hierarchy with Domain-Specific Design
**HCI Principle:** Aesthetic and Minimalist Design

**From Lesson 10:**
> "This hierarchy should be domain specific, making it easy to quickly see important parts of the trace that tend to be important, whereas hiding the rest by default."

**For Recipe Assistant Domain:**
- **User query:** Prominent, large font
- **Tool errors:** Red highlight with ❌ icon
- **Intermediate "Let me help you...":** De-emphasized, smaller font
- **Final recipe suggestions:** Prominent, formatted

#### Feature 1.3: Collapsible Sections
**HCI Principle:** Aesthetic and Minimalist Design

**From Lesson 10:**
> "To manage complexity, interfaces can collapse less important sections by default, but the full trace should always remain accessible for accurate judgment."

**Implementation:**
```
📄 SYSTEM PROMPT [Click to expand]

USER QUERY (always visible)
─────────────────────────────
What vegetarian high-protein meal can I cook tonight?

▼ AGENT REASONING (3 steps) [Click to expand]
    ├─ Let me help you find...
    ├─ I'll look for dinner options...
    └─ For your request, I will note...

▼ TOOL CALLS (5 calls) [Click to expand]
    ├─ ✓ TOOL_CALL[ParseRequest]
    ├─ ✓ TOOL_CALL[PlanToolCalls]
    ├─ ❌ TOOL_CALL[GenRecipeArgs] Error: unable to generate...
    └─ ...

FINAL RESPONSE (always visible)
─────────────────────────────
Great! Chickpeas and quinoa are excellent sources...
```

**Why Show Prompt:**
> "Each screen shows the original document, the generated output, and the prompt that produced it" (DocWrangler case study)

When reviewer sees failure, they need to ask: "Was the prompt clear enough?"

---

### CATEGORY 2: FEEDBACK COLLECTION

#### Feature 2.1: Pass/Fail/Defer Buttons
**HCI Principle:** Error Prevention + User Control and Freedom

**From Lesson 10:**
> "Quick overall ratings, such as thumbs up/down or numeric scores, help speed up the process."

> "Provide a clear, prominent 'Defer' button for uncertain cases—critical for maintaining label accuracy and avoiding low-confidence judgments."

**Implementation:**
```
┌─────────────────────────────────────┐
│  [  PASS  ]  [  FAIL  ]  [  DEFER  ]│
│   (large, distinct buttons)         │
└─────────────────────────────────────┘
```

**Why Defer is Critical:**
Forces decision but allows uncertainty acknowledgment. Prevents low-confidence labels that contaminate data.

#### Feature 2.2: Open-Ended Notes with Autocomplete
**HCI Principle:** Error Prevention + Recognition Rather Than Recall

**From Lesson 10:**
> "Open-ended feedback fields allow reviewers to explain their reasoning—critical for surfacing unexpected problems or refining evaluation criteria over time."

**Implementation:**
```
┌─────────────────────────────────────────────────┐
│ Notes (what went wrong?):                       │
│ ┌─────────────────────────────────────────────┐ │
│ │ Agent called wrong tool...                  │ │
│ │ [autocomplete suggests: "wrong tool called"]│ │
│ └─────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

**Autocomplete Behavior:**
- Learn from user's previous notes
- Suggest completions as they type
- Help consistency WITHOUT forcing pre-defined categories
- Example: typing "tone..." suggests "tone mismatch" (from trace #5)

**Why Not Checkboxes Yet:**
In error analysis phase, you don't know your failure modes! Open coding discovers them.

#### Feature 2.3: Edit Past Labels
**HCI Principle:** User Control and Freedom

**From Lesson 10 (EvalGen case study):**
> "EvalGen... gave reviewers the ability to edit past labels."

> "Even human reviewers experience criteria drift. Over time, annotators refined what 'correctness' or 'polish' meant, revised prior judgments."

**Implementation:**
- Allow navigation back to ANY previously reviewed trace
- Show edit history: "Last edited: 2025-10-28 by Alice (Pass → Fail)"
- Track revision history in data structure

**Why Critical:**
After reviewing 30 traces, you realize your criteria changed. Need to update earlier annotations.

---

### CATEGORY 3: METADATA & CONTEXT

#### Feature 3.1: Contextual Metadata Display
**HCI Principle:** Visibility of System Status + Match Between System and Real World

**From Lesson 10:**
> "Contextual metadata—such as the trace ID, pipeline version, or timestamp—appears directly in the interface to aid reviewer decisions."

**Required Metadata (Always Visible):**
```
┌─────────────────────────────────────┐
│ 📋 TRACE METADATA                   │
├─────────────────────────────────────┤
│ Trace ID: [generate from row number]│
│ Customer Persona: [from CSV or N/A] │
│ Model: [add column to CSV]          │
│ Iteration: [add column to CSV]      │
│ Reviewer: [current user]            │
│ Status: ○ Not Reviewed              │
│ Timestamp: [when reviewed]          │
└─────────────────────────────────────┘
```

**Note on Empty Columns:**
Your CSV has empty `trace_id` and `customer_persona`. Must handle:
- Generate trace_id from row number (e.g., TRACE_001)
- Display "N/A" for empty persona
- Add model/iteration columns to data structure

**Why This Matters:**
Pattern discovery: "All iteration 3 traces fail" or "Investor persona always has tone issues"

---

### CATEGORY 4: NAVIGATION & PROGRESS

#### Feature 4.1: Single Trace View
**HCI Principle:** User Control and Freedom + Aesthetic and Minimalist Design

**From Lesson 10:**
> "The UI shows one trace at a time, with the user input, persona context, and generated email clearly separated."

**Why One at a Time:**
- Focused attention on single trace
- Easier to think deeply without distraction
- Simpler annotation (all feedback about THIS trace)

#### Feature 4.2: Navigation Controls
**HCI Principle:** User Control and Freedom

**Implementation:**
```
┌─────────────────────────────────────────────────────┐
│ [◄ Previous]  [Next ►]  [Skip/Defer]  [Jump to #__]│
│                                                      │
│ Trace 5 of 98                                       │
└─────────────────────────────────────────────────────┘
```

**Jump to Trace:**
Essential for:
- Collaboration: "Alice reviews 1-50, Bob reviews 51-98"
- Reference: "Someone said trace 47 is interesting"
- Re-review: "Let me look at trace 23 again"

#### Feature 4.3: Progress Tracking
**HCI Principle:** Visibility of System Status

**From Lesson 10:**
> "Progress Indicators: It can be useful to show counts of traces reviewed, potentially grouped by key dimensions."

**Display:**
```
┌─────────────────────────────────────────────────┐
│ PROGRESS                                         │
├──────────────────────────────────────────────────┤
│ Total: 98 traces                                 │
│ Reviewed: 23 (23.5%)                            │
│ ├─ Pass: 15                                      │
│ ├─ Fail: 6                                       │
│ └─ Deferred: 2                                   │
│ Remaining: 75                                    │
└──────────────────────────────────────────────────┘
```

---

### CATEGORY 5: SORTING & FILTERING (USER CONTROL)

#### Feature 5.1: User-Controlled Sorting
**HCI Principle:** Flexibility and Efficiency of Use + User Control and Freedom

**From Lesson 10:**
> "Even with a well-designed interface, showing the wrong examples leads to wasted effort."

**Implementation:**
```
┌─────────────────────────────────────────┐
│ Sort by: [▼ Dropdown]                   │
├─────────────────────────────────────────┤
│ ○ Sequential (1, 2, 3...)               │
│ ● Errors First (show tool errors first) │
│ ○ Random                                │
│ ○ Not Reviewed First                    │
└─────────────────────────────────────────┘
```

**Why User Control:**
YOU decide how to prioritize review work, not the system.

**Smart Default for Your Data:**
"Errors First" - your CSV has many tool call errors, show these first!

#### Feature 5.2: Filtering
**HCI Principle:** Flexibility and Efficiency of Use

**From Lesson 10:**
> "Filters: Allow filtering on important dimensions relevant to the product or domain."

**Implementation:**
```
┌─────────────────────────────────────────────┐
│ Filter by:                                   │
├──────────────────────────────────────────────┤
│ □ Has tool errors (47 traces)               │
│ □ Customer persona: [dropdown when populated]│
│ □ Review status:                            │
│   ☐ Not reviewed  ☐ Pass  ☐ Fail  ☐ Deferred│
│ □ Model: [dropdown when populated]          │
│ □ Iteration: [dropdown when populated]      │
│                                             │
│ Showing: 47 of 98 traces                    │
└──────────────────────────────────────────────┘
```

**Current State:**
Since `customer_persona` is empty, that filter disabled until populated.

---

### CATEGORY 6: ERROR HIGHLIGHTING (DOMAIN-SPECIFIC)

#### Feature 6.1: Visual Error Emphasis
**HCI Principle:** Aesthetic and Minimalist Design

**From Lesson 10:**
> "Emails are shown with clear visual formatting—highlighting key entities like price, location, or timing—to make errors easier to spot."

**For Your Tool Calls:**
```
TOOL_CALL[GenRecipeArgs]
❌ Error: unable to generate recipe arguments due to missing dietary restriction information
   ↑ Red background, bold text, error icon
```

**Visual Treatment:**
- **Tool call success:** ✓ Green checkmark, normal text
- **Tool call error:** ❌ Red icon, red background, bold
- **Critical errors:** Expanded by default (not collapsed)

**Why Domain-Specific:**
In recipe assistant, tool errors are critical failure indicator.

---

### CATEGORY 7: DATA PERSISTENCE

#### Feature 7.1: Save Annotations
**HCI Principle:** Visibility of System Status + Error Prevention

**From Lesson 10:**
> "The review interface should support lightweight actions that translate human insight into engineering outcomes."

**Auto-Save Behavior:**
- Click Pass/Fail/Defer → Save immediately
- Type in notes → Save on blur (when navigate away)
- Show save status: "✓ Saved" or "Saving..."

**Storage Options:**
```
Option A: New CSV with annotations
├─ Original columns: trace_id, customer_persona, user_query, etc.
├─ New columns: status, reviewer, notes, reviewed_at, structured_tags
└─ Filename: traces_annotated_2025-10-28.csv

Option B: Separate JSON/SQLite file
├─ Maps trace_id → annotations
├─ Allows revision history
└─ Easier to merge multiple annotators

Recommended: Option B (more flexible)
```

**Data Structure:**
```json
{
  "trace_annotations": [
    {
      "trace_id": "TRACE_001",
      "status": "fail",
      "reviewer": "Alice",
      "notes": "Agent called wrong tool - should use check_calendar not send_email",
      "structured_tags": [],
      "reviewed_at": "2025-10-28T10:30:00Z",
      "revision_history": [
        {
          "timestamp": "2025-10-28T10:30:00Z",
          "action": "created",
          "status": "fail"
        }
      ],
      "model": "gpt-4",
      "iteration": 3
    }
  ]
}
```

**Why Separate File:**
- Don't overwrite original CSV
- Support multiple annotators
- Track revision history
- Easy to add metadata columns

---

## 🔮 FEATURE INVENTORY: PHASE 2+ (FUTURE - DON'T BUILD YET)

### Progressive Disclosure Architecture
**From Lesson 10:**
> "Progressive Disclosure: Simplify the interface by showing only basic options (e.g., 'Pass,' 'Fail,' 'Defer') at first, revealing detailed tagging only when needed."

**Phase 1 → Phase 2 Evolution:**
```
┌─────────────────────────────────────────────────┐
│ Phase 1 UI (NOW)                                │
├─────────────────────────────────────────────────┤
│ [PASS] [FAIL] [DEFER]                          │
│ Notes: ________________________________         │
│                                                 │
│ [Hidden: Structured tags section]              │
│ [Hidden: Advanced filters]                     │
│ [Hidden: Clustering view]                      │
└─────────────────────────────────────────────────┘

After ~30 traces reviewed, you discover failure modes:
"tone mismatch", "wrong tool called", "missing constraint"

┌─────────────────────────────────────────────────┐
│ Phase 2 UI (AFTER PATTERN DISCOVERY)           │
├─────────────────────────────────────────────────┤
│ [PASS] [FAIL] [DEFER]                          │
│                                                 │
│ ⚙️ [Toggle Advanced Mode]                      │
│                                                 │
│ ▼ Structured Failure Tags (NOW VISIBLE)        │
│   ☐ Tone mismatch                              │
│   ☐ Wrong tool called                          │
│   ☐ Missing constraint                         │
│                                                 │
│ Notes: ________________________________         │
│ (still available for new observations)         │
└─────────────────────────────────────────────────┘
```

### Phase 2 Features (Add After Initial Error Analysis)

#### Structured Failure Tags
**When to add:** After reviewing ~30 traces, patterns emerge in notes
**How:** Convert recurring phrases into checkboxes
**Backward compatible:** Keep free-form notes alongside tags

#### Keyboard Shortcuts
**When to add:** When reviewing 50+ traces, speed becomes bottleneck
**Examples:** 
- P = Pass, F = Fail, D = Defer
- N = Next, B = Back
- J = Jump to trace

#### Feedback Templates
**From Lesson 10:**
> "Feedback Templates: Offer frequently used comments (e.g., 'Tone too casual,' 'Missing key fact') as quick-select or dropdown options."

**When to add:** After identifying recurring note patterns
**How:** Dropdown with common phrases, adapted to selected failure tag

---

### Phase 3 Features (Comparison & Measurement)

#### Clustering
**From Lesson 10:**
> "Clustering lets us group traces that share common features... Reviewing a few traces from the 'investor' cluster might reveal a recurring issue."

**Types:**
- **Metadata clustering:** Group by customer_persona, model, iteration
- **Semantic clustering:** Embed queries, group by vector similarity (k-means)

**When to add:** When you want to find patterns across >100 traces

#### Search & Similarity
**From Lesson 10:**
> "Review tools should support easy ways to find similar examples—by keyword, metadata, or semantic content."

**Types:**
- **Keyword search:** Find traces containing "chickpea" or "error"
- **Metadata search:** Find all traces from model=gpt-4, iteration=3
- **Semantic search:** "Find traces similar to this one" (embedding-based)

**When to add:** When asking "Are there more traces like this failure?"

#### Batch Actions
**From Lesson 10:**
> "Batch Review & Bulk Actions: Let users review trace clusters and optionally apply the same failure label to multiple examples—useful for repetitive issues, but should be used with care."

**When to add:** After clustering reveals 20 traces with identical issue
**Caution:** Use carefully, can introduce bias

---

### Phase 4 Features (Integration & Workflow)

#### Integration Actions
**From Lesson 10:**
> "A reviewer should be able to click a button to add a trace directly to the CI golden set, or file a bug report with trace details already prefilled."

**Actions:**
```
After marking trace as Fail:
┌─────────────────────────────────────┐
│ ⚙️ Additional Actions:              │
├─────────────────────────────────────┤
│ ☐ Add to CI golden test set        │
│ ☐ File bug report                  │
│ ☐ Flag for team review             │
└─────────────────────────────────────┘
```

**When to add:** When integrating with CI/CD pipeline

#### Prompt Improvement Suggestions
**From Lesson 10 (DocWrangler):**
> "Once enough notes are collected, users can invoke the prompt improvement feature. This takes the current prompt and the accumulated reviewer comments, and suggests an updated prompt."

**When to add:** After 50+ traces, you notice prompt-related failures
**How:** LLM analyzes notes, suggests prompt revisions

---

## 🎨 ARCHITECTURAL DECISIONS

### Decision 1: Progressive Disclosure with Extensibility
**Pattern:** Build data model for all phases, but only show Phase 1 UI now

**From Lesson 10:**
> "Progressive Disclosure: Simplify the interface by showing only basic options at first, revealing detailed tagging only when needed."

**Data Structure Design:**
```json
{
  "trace_id": "TRACE_001",
  "status": "fail",
  "notes": "Agent called wrong tool",
  "structured_tags": [],        ← Empty now, populate in Phase 2
  "reviewer": "Alice",
  "reviewed_at": "2025-10-28T10:30:00Z",
  
  // Future phase fields (in data, not UI yet)
  "ci_golden_set": false,       ← Phase 4
  "bug_filed": false,           ← Phase 4
  "cluster_id": null,           ← Phase 3
  "similarity_score": null      ← Phase 3
}
```

**Benefits:**
1. No rework needed when adding features
2. Data structure supports full lifecycle
3. Can add UI progressively without data migration
4. Backward compatible (old annotations still work)

**Interview Talking Points:**
- "Recognized user needs evolve as they understand data"
- "Designed for extensibility without over-engineering"
- "Applied HCI principle of Progressive Disclosure"
- "Avoided trap of building all features upfront OR three separate tools"

---

### Decision 2: One-Way Data Flow
**Pattern:** CSV (read-only) → Annotations (write) → Analytics (future)

```
traces.csv (original, never modified)
    ↓ (load into interface)
Review Interface
    ↓ (save annotations)
traces_annotations.json (or .db)
    ↓ (future: analyze)
Failure Mode Reports, Metrics Dashboards
```

**Why:**
- Preserve original data
- Support multiple annotators (parallel review)
- Easy to version control annotations separately
- Can re-review same traces with new criteria

---

### Decision 3: Handle Missing Metadata Gracefully
**Problem:** CSV has empty `trace_id` and `customer_persona` columns

**Solution:**
```python
# On load:
if csv_row['trace_id'] is empty:
    generate_trace_id = f"TRACE_{row_number:03d}"
    
if csv_row['customer_persona'] is empty:
    display = "N/A"
    disable_persona_filter = True
```

**Display:**
```
Trace ID: TRACE_001 (generated)
Customer Persona: N/A
```

**Why:** 
- Tool still works with imperfect data
- Clear to user what's missing
- Easy to populate later

---

## 🗺️ IMPLEMENTATION ROADMAP

### Sprint 1: MVP (Minimum Viable Product)
**Goal:** Review traces, take notes, discover patterns

**Features:**
- ✅ Parse conversation_messages (handle | delimiter, roles, tool calls)
- ✅ Display one trace at a time with visual hierarchy
- ✅ Collapsible sections (prompt, reasoning, tool calls)
- ✅ Pass/Fail/Defer buttons
- ✅ Open notes field with autocomplete
- ✅ Metadata display (generate trace_id, handle missing data)
- ✅ Navigation (Previous, Next, Jump to)
- ✅ Progress tracking
- ✅ Error highlighting (red background for tool errors)
- ✅ Save annotations to JSON/SQLite
- ✅ Edit past labels

**Deliverable:** Can review all 98 traces, discover failure modes

---

### Sprint 2: Efficiency Features
**Goal:** Faster review, better prioritization

**Features:**
- ✅ User-controlled sorting (Errors First, Sequential, Random)
- ✅ Basic filters (Has errors, Review status)
- ✅ Save status indicator ("✓ Saved")
- ✅ Keyboard shortcut hints (optional)

**Deliverable:** Can review 98 traces in <2 hours

---

### Sprint 3: Pattern Discovery (Only After Sprint 1 & 2 Complete)
**Goal:** Convert open notes into structured failure modes

**Process:**
1. Export all notes from Sprint 1 reviews
2. Manually group into failure categories (or use LLM to suggest)
3. Add Phase 2 UI features:
   - Structured tag checkboxes
   - Filters by failure mode
   - Analytics: count of each failure mode

**Deliverable:** Know exactly how many traces have each failure mode

---

### Future Sprints (Phase 3+)
- Clustering & semantic search
- Batch actions
- Integration with CI/CD
- Prompt improvement suggestions
- Multi-user collaboration features

---

## 📐 LESSON 10 PRINCIPLES → FEATURES MAPPING

### Visibility of System Status
- ✅ Progress tracking (Trace 5 of 98)
- ✅ Review status counts (Pass: 15, Fail: 6)
- ✅ Save status indicator
- ✅ Edit history timestamps

### Match Between System and Real World
- ✅ Use domain terms (recipe, tool call, error)
- ✅ Metadata reflects real pipeline (model, iteration)
- ❌ Integration actions (Phase 4)

### User Control and Freedom
- ✅ Navigation (Next/Previous/Jump)
- ✅ Edit past labels
- ✅ Defer option (don't force judgment)
- ✅ User-controlled sorting/filtering
- ❌ Undo (future enhancement)

### Consistency and Standards
- ✅ Same layout for all traces
- ✅ Consistent color coding (red=error, blue=reasoning)
- ✅ Reusable terminology across traces

### Error Prevention
- ✅ Distinct Pass/Fail/Defer buttons
- ✅ Autocomplete prevents typos in notes
- ✅ Defer prevents low-confidence labels
- ❌ Require rationale for Fail (optional future)

### Recognition Rather Than Recall
- ✅ Show prompt with each trace
- ✅ Display metadata (don't make user remember)
- ✅ Autocomplete shows past notes
- ❌ Structured failure tags (Phase 2)

### Flexibility and Efficiency of Use
- ✅ Sorting/filtering (expert features)
- ✅ Jump to trace (expert feature)
- ❌ Keyboard shortcuts (Sprint 2+)
- ❌ Batch actions (Phase 3)

### Aesthetic and Minimalist Design
- ✅ Visual hierarchy (prominent query, collapsed details)
- ✅ Error highlighting (red backgrounds)
- ✅ Collapsible sections (hide noise)
- ✅ Domain-specific design (recipe assistant)

---

## 🚨 CRITICAL CSV PARSING REQUIREMENTS

### Conversation Messages Format
```
Input: "USER: query | ASSISTANT: response | ASSISTANT: TOOL_CALL[Name] Error: ..."

Parse into:
[
  {
    "role": "USER",
    "content": "query",
    "type": "message"
  },
  {
    "role": "ASSISTANT",
    "content": "response",
    "type": "message"
  },
  {
    "role": "ASSISTANT",
    "content": "TOOL_CALL[Name] Error: ...",
    "type": "tool_call",
    "tool_name": "Name",
    "is_error": true,
    "error_message": "Error: ..."
  }
]
```

### Parsing Edge Cases
1. **Role inconsistency:** Handle both ASSISTANT and AGENT
2. **Empty fields:** trace_id, customer_persona may be empty
3. **Tool call detection:** Look for "TOOL_CALL[" pattern
4. **Error detection:** Look for "Error:" keyword
5. **Multi-line errors:** Error messages may span multiple lines

### Error Handling
- If parse fails, display raw text + warning
- Log parsing errors for debugging
- Gracefully handle malformed data

---

## 📊 SUCCESS METRICS

### Immediate (Sprint 1)
- Can parse and display all 98 traces correctly
- Can review 1 trace in <2 minutes
- Can save annotations without data loss
- Can edit past annotations

### Short-term (Sprint 2)
- Can review all 98 traces in <3 hours
- Discover 5-10 distinct failure modes
- 90%+ of notes use autocomplete (faster entry)

### Medium-term (Sprint 3)
- Convert open notes → structured failure modes
- Quantify: X% of traces have "tone mismatch"
- Answer: "Which failure mode is most common?"

### Long-term (Phase 3+)
- Compare iterations: "Did v2 reduce tool errors?"
- Answer: "Is my product getting better or just changing?"

---

## 🎯 WHAT NOT TO BUILD (Common Mistakes)

### ❌ Don't Build These in Phase 1:
1. **Structured failure tags** - Don't know failure modes yet
2. **Clustering** - Need annotations first
3. **Fancy dashboards** - Need data first
4. **Multi-user auth** - Single user for now
5. **Real-time collaboration** - Overkill for error analysis
6. **Export to 10 formats** - JSON/CSV sufficient
7. **LLM-powered auto-labeling** - Human judgment is the point

### ❌ Don't Do These Ever:
1. **Build 3 separate tools** for each phase
2. **Overwrite original CSV** with annotations
3. **Skip user control** over sorting/filtering
4. **Hide the prompt** (user needs to see it)
5. **Force judgment** on unclear traces (need Defer)
6. **Pre-define failure modes** before seeing data

---

## 🔍 LESSON 10 QUOTES (Key Principles)

### On Custom Interfaces
> "Spreadsheets struggle with the complexity of LLM traces. A single trace might contain a long input prompt, multiple intermediate LLM reasoning steps, several tool calls with structured arguments and responses, retrieved RAG context, and the final output."

### On Error Analysis
> "Human review tools must treat evaluation as an evolving sensemaking process, not just a static labeling task."

### On Progressive Disclosure
> "Simplify the interface by showing only basic options (e.g., 'Pass,' 'Fail,' 'Defer') at first, revealing detailed tagging only when needed."

### On Sampling
> "Even with a well-designed interface, showing the wrong examples leads to wasted effort."

### On Visual Hierarchy
> "This hierarchy should be domain specific, making it easy to quickly see important parts of the trace that tend to be important, whereas hiding the rest by default."

### On Integration
> "A reviewer should be able to click a button to add a trace directly to the CI golden set, or file a bug report with trace details already prefilled."

### On Edit Past Labels
> "Even human reviewers experience criteria drift. Over time, annotators refined what 'correctness' or 'polish' meant, revised prior judgments, and added new failure categories."

---

## 📚 REFERENCES

**Primary Source:** Lesson 10 - Interfaces for Continuous Human Review and Error Analysis

**Key Case Studies:**
- EvalGen (binary grading, edit past labels, criteria drift)
- DocWrangler (prompt improvement, open coding, side-by-side views)
- Real Estate Assistant (interface evolution across 3 phases)

**HCI Foundations:**
- Jakob Nielsen's Usability Heuristics (1994)
- Donald Norman's Design Principles (2013)
- Ben Shneiderman's Interface Guidelines (1997)

---

## ✅ FINAL CHECKLIST: Ready to Build?

Before implementation, confirm:

- [ ] Understand CSV structure and parsing requirements
- [ ] Know which metadata columns are empty (need handling)
- [ ] Clear on Phase 1 scope (error analysis ONLY)
- [ ] Designed data structure to support future phases
- [ ] Mapped all features to HCI principles
- [ ] Know what NOT to build yet
- [ ] Have plan for handling edge cases (empty fields, parse errors)
- [ ] Understand Progressive Disclosure architecture
- [ ] Ready to explain architectural decisions in interviews

---

## 🎓 INTERVIEW-READY SUMMARY

**The Problem:**
"I was building a trace review tool for LLM agent evaluation. Spreadsheets were painful - walls of text, no structure, slow navigation. I needed a custom interface but didn't want to over-engineer."

**The Architectural Decision:**
"I used Progressive Disclosure - built the data model to support the full evaluation lifecycle (error analysis → measurement → comparison), but only revealed UI features as needed. Phase 1 shows Pass/Fail/Defer + notes. Phase 2 adds structured tags after discovering failure modes. Phase 3 adds clustering and comparison."

**The Benefits:**
- No rework or data migration later
- Users start simple, graduate naturally
- Backward compatible (old features never break)
- Avoided building features I didn't need yet
- Avoided building 3 separate tools

**The Foundation:**
"Based on HCI principles from Lesson 10, especially: Aesthetic & Minimalist Design (visual hierarchy, collapsible sections), User Control (edit labels, custom sorting), and Recognition over Recall (show metadata, autocomplete notes)."

**The Result:**
"Tool that evolves with user understanding. Start by discovering 'what went wrong,' grow into measuring 'how often,' culminate in answering 'is it getting better?'"

---

*End of Specification Document*
