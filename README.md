# SignalCompass 🧭
### Finding signal in the noise of LLM agent traces

A systematic review interface for discovering failure modes in LLM agents. Built on HCI principles from research.

**🚀 Live App:** [product-signal.lovable.app](https://product-signal.lovable.app/?utm_source=lovable-editor)

---

## What This Is

I built a recipe assistant agent. After deployment, I had 98 real conversation traces with tons of tool calls, errors, and multi-turn conversations. Reviewing them in spreadsheets was painful.

So I designed a proper review tool using Human-Computer Interaction principles.

**The goal:** Answer *"Is my product getting better or just changing?"*

---

## Why Share This?

This repo has everything you need to build your own review tool:

📝 **Lovable-ready specs** → Copy, paste, iterate  
🏗️ **Architecture breakdown** → See how HCI principles translate to features  
🎯 **Progressive Disclosure pattern** → Build for growth without over-engineering  

Perfect if you're:
- Building eval tools for LLM agents
- Learning to apply HCI principles practically  
- Need to review traces systematically

---

## Quick Start

### Build with Lovable (5 min)
1. Copy `lovable/prompt.md` + `lovable/spec.json`
2. Paste into Lovable
3. Get a working app
4. Iterate to your needs

### Study the Design (15 min)
1. Read `docs/full_specification.md`
2. See how 8 HCI principles → 14 concrete features
3. Understand the Progressive Disclosure architecture

### Adapt for Your Project
- Use the same HCI principles for your eval tool
- Adapt the phase-by-phase approach
- Steal the data structures

---

## Key Design Decisions

**Progressive Disclosure**  
Built data model for full lifecycle (discover → measure → compare), but only show Phase 1 UI initially. Add features as patterns emerge. No rework needed.

**Open Coding First**  
Don't pre-define failure modes. Let them emerge through ~30 reviews. Then add structured tags.

**Speed = Velocity**  
Every second saved per trace compounds. Autocomplete, error highlighting, smart sorting → massive ROI.

---

## What's Inside

```
signalcompass/
├── lovable/           # Copy-paste ready for Lovable
│   ├── prompt.md      # Natural language spec
│   └── spec.json      # Structured architecture
├── docs/              # Full design breakdown
└── data/              # Sample trace format
```

---

## The Recipe Assistant Context

**Agent:** Helps users find meals based on preferences and ingredients  
**Tools:** GetCustomerProfile, ParseRequest, GenRecipeArgs, GetRecipes  
**Data:** 98 real traces from production (multi-turn convos, tool errors, successes)  
**Need:** Discover why agent fails, where it struggles, what patterns exist

Sample trace format in `data/sample_traces.csv`

---

## Based On

**Application-Centric AI Evals for Engineers and Technical Product Managers**  
*Shreya Shankar and Hamel Husain, Fall 2025*

Specifically: Lesson 10 on review interfaces and error analysis

---

## Built By

**Vidhya Sriram**  
LinkedIn: [linkedin.com/in/vidhyasriram](https://www.linkedin.com/in/vidhyasriram/)

---

**Try it out. Break it. Make it yours.** 🚀
