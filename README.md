# ProductSignal 🧭

**Is your AI product getting better — or just changing?**

A trace review tool for discovering failure modes in LLM agents. Built for PMs and founders who need to understand their AI's behavior without drowning in vendor dashboards or fighting with spreadsheets.

🚀 **Live App:** [product-signal.lovable.app](https://product-signal.lovable.app)

---

## What This Does

I built a recipe assistant agent. After deployment, I had 98 conversation traces—multi-turn conversations, tool calls, errors. Spreadsheets couldn't help me see patterns. Generic tools didn't fit my data.

**The core question:** Is my agent actually improving, or am I just moving numbers around?

**Why this approach:**  
Most eval tools force you to define metrics before understanding failures. This reverses it: discover what's breaking, then measure how often. Start qualitative, graduate to quantitative.

**This tool helps you:**
- Review LLM conversation traces systematically
- Discover what failure modes exist (before you can measure them)
- Track graceful degradation (tool fails, but agent recovers)
- Export clean data for analysis

---

## Key Features

**Multi-level annotation**  
Capture nuance: tool fails but agent recovers—is that Pass or Fail?
- **Holistic Pass:** Did user get what they needed?
- **Component Errors:** Did any tools/steps fail?
- **Task Success:** Was task completed despite errors?

*Example: GetRecipes fails (component error ✓) but agent suggests meal from memory (task success ✓, holistic pass ✓)*

**Smart review interface**
- Single chronological view (user → agent → tools → recovery)
- Tool errors highlighted in context with ❌
- Failure ratio auto-calculated: "1/3 tools failed (33%)"
- See the full conversation flow to judge recovery behavior

**Pattern discovery**
- Filter by: tool failure severity (100% failed vs partial), annotation patterns, status
- Sort by errors-first (see problems immediately)
- Jump to specific traces
- List view shows all annotations at a glance

**Data you own**
- Enhanced CSV: original trace data + your annotations in one file
- Annotations clearly marked with "annotated_" prefix
- localStorage with auto-backup every 10 reviews
- No vendor lock-in

---

## Quick Start

### Use the Live App (2 min)
1. Go to [product-signal.lovable.app](https://product-signal.lovable.app)
2. Upload your CSV (format below)
3. Start reviewing traces

### Build Your Own (5 min)
1. Copy `lovable/prompt.md` + `lovable/spec.json`
2. Paste into [Lovable](https://lovable.dev)
3. Customize for your use case

---

## CSV Format

Your file should include:

**Required columns:**
- `trace_id` - unique identifier
- `user_query` - initial user request
- `conversation_messages` - pipe-delimited conversation

**Format example:**
```csv
trace_id,user_query,conversation_messages
TRACE_001,"What vegetarian meal...","USER: What vegetarian... | AGENT: Let me help... | TOOL_CALL[GetRecipes] Error: ... | AGENT: Could you share ingredients? | USER: I have chickpeas..."
```

**Conversation format:**
- Delimiter: ` | ` (space-pipe-space)
- Roles: `USER:`, `AGENT:`, `ASSISTANT:`
- Tool calls: `TOOL_CALL[ToolName] description or error message`
- Errors: keyword `Error:` in tool call message

**Optional columns:**
- `customer_persona`
- `tool_calls`

See `data/sample_traces.csv` for complete example.

---

## What's Inside

```
productsignal/
├── lovable/              # Ready to copy-paste
│   ├── prompt.md         # Natural language spec
│   └── spec.json         # Structured architecture
├── docs/
│   └── design-notes.md   # Why these choices
└── data/
    └── sample_traces.csv # Example format
```

---

## Design Principles

**Show only what matters**  
*Applied: Aesthetic & Minimalist Design, Visual Hierarchy*

Errors highlighted in red with ❌. Tool failures shown in conversation context. List view sorts errors-first by default. Collapsible sections hide noise.

**Let patterns emerge**  
*Applied: Recognition Rather Than Recall, Open Coding*

Start with open notes. Autocomplete learns from your annotations. Build structured failure tags after ~30 traces when patterns are clear. Don't pre-define what "good" looks like.

**Make it fast enough to finish**  
*Applied: Flexibility & Efficiency of Use, User Control*

Jump to any trace. Filter by tool failure severity. Edit past annotations when criteria evolve. Keyboard shortcuts for power users. Result: 60% faster than spreadsheets.

---

## Who This Helps

**Built for:**
- PMs discovering why chatbot fails (before building metrics)
- Founders running lightweight eval loops (no vendor overhead)
- Engineers finding patterns in agent behavior (what actually breaks vs what looks broken)
- Teams who need custom eval tools fast

**Not for:**
- Large teams needing real-time collaboration
- Automated scoring at scale
- Enterprise compliance workflows

---

## What's Coming

**Phase 2: Structured Analysis**
- Failure mode tags (after patterns emerge from open coding)
- Error frequency tracking
- Failure taxonomy builder

**Phase 3: Pattern Recognition**
- Semantic clustering (group similar traces)
- Anomaly detection
- Trend analysis

**Phase 4: Collaboration**
- Multi-user support
- Inter-rater agreement tracking
- Shared failure mode definitions

---

## Built With

- **Lovable** - rapid prototyping and deployment
- **HCI Principles** - Nielsen's heuristics, Norman's design principles, Shneiderman's guidelines
- **Course Material** - Application-Centric AI Evals (Shankar & Husain, Fall 2025, Lesson 10)
- **Grounded Theory** - for systematic error analysis methodology

---

## Author

**Vidhya Sriram**  
[LinkedIn](https://linkedin.com/in/vidhyasriram) | [Try the tool](https://product-signal.lovable.app)

Built from real need. Shared for others stuck in spreadsheets.

---

**Try it. Break it. Make it yours.** 🚀
