---
bundle:
  name: dev-memory
  version: 1.0.0
  description: Persistent memory system for maintaining context across all amplifier sessions

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

## Memory Partner Agent

The `memory-partner` agent handles all memory operations using slash commands and natural language.

### Slash Commands

- `/memory-recall [query]` - Search memories
- `/memory-remember [text]` - Add a memory
- `/memory-status` - Show work log
- `/memory-guide` - Show full guide

### Natural Language

- "what do you remember about X?"
- "remember this: [fact]"
- "what was I working on?"

## Memory Storage

All data is stored locally at `~/.amplifier/dev-memory/`:

- `memory-store.yaml` - Facts to remember
- `work-log.yaml` - Active work tracking
- `project-notes.md` - Free-form notes

## Instructions

@dev-memory:context/memory-system-guide.md

---

@foundation:context/shared/common-system-base.md
