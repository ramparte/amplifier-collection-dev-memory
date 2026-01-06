---
bundle:
  name: dev-memory
  version: 1.0.0
  description: Persistent memory system for maintaining context across all amplifier sessions

config:
  allowed_write_dirs:
    - ~/.amplifier/dev-memory

includes:
  - bundle: git+https://github.com/microsoft/amplifier-foundation@main
  - bundle: dev-memory:behaviors/dev-memory
---

# Developer Memory System

A persistent development partner that maintains context across all amplifier sessions.

## What This Provides

The dev-memory bundle adds persistent memory capabilities to amplifier:

- **Remember facts** - "remember this: [text]"
- **Recall context** - "what do you remember about X?"
- **Track work** - "what was I working on?"
- **Maintain continuity** - Auto-loads recent context at session start

## Token-Efficient Architecture

**Hybrid approach for scalability:**

- **Writes:** Main agent appends to memory files (~100 tokens)
- **Reads:** memory-retrieval sub-agent searches and returns only matches (~200 tokens)
- **Result:** Constant token usage even with 1000+ memories!

### Natural Language Interface

- "what do you remember about X?" - Search memories
- "remember this: [fact]" - Add a memory
- "what was I working on?" - Show work status
- "remind me how the memory system works" - Show guide

## Memory Storage

All data is stored locally at `~/.amplifier/dev-memory/`:

- `memory-store.yaml` - Facts to remember
- `work-log.yaml` - Active work tracking
- `project-notes.md` - Free-form notes

## Instructions

@dev-memory:context/memory-system-guide.md

---

@foundation:context/shared/common-system-base.md
