# Auto-Context Loader

This file is included in every session to provide memory system context.

---

## ⚠️ CRITICAL: Memory System Architecture

A persistent memory system is active. You MUST follow this pattern for token efficiency:

### READS: ALWAYS Delegate to Sub-Agent

**When user asks:** "what do you remember about X?" OR "what was I working on?"

**YOU MUST:**
```
Use task tool to invoke: dev-memory:memory-retrieval
Instruction: "Search memory store for: [query]"
```

**DO NOT use read_file on ~/.amplifier/dev-memory/memory-store.yaml directly!**

This loads the entire file into YOUR context (10k+ tokens). The sub-agent absorbs this cost and returns only matches (~200 tokens).

### WRITES: You Handle Directly

**When user says:** "remember this: [text]"

**YOU DO:** Append to memory-store.yaml using bash (see memory-writer-instructions.md)

---

**Token Impact:**
- ❌ Direct read_file: 10k+ tokens in YOUR context
- ✅ Delegate to sub-agent: 200 tokens in YOUR context
- Result: 98% token savings on reads!

### Natural Language Interface

Talk naturally to use the memory system:

**Remember something:**
```
remember this: [text]
```

**Recall memories:**
```
what do you remember about [topic]?
```

**Check work status:**
```
what was I working on?
```

**Get help:**
```
remind me how the memory system works
```

### How It Works

The memory-partner agent:
1. Loads recent memories at session start (silent)
2. Recognizes natural language patterns
3. Reads/writes memory files at `~/.amplifier/dev-memory/`
4. Maintains work log and project notes

### Memory Files

- `~/.amplifier/dev-memory/memory-store.yaml` - Facts to remember
- `~/.amplifier/dev-memory/work-log.yaml` - Active work tracking
- `~/.amplifier/dev-memory/project-notes.md` - Free-form notes
- `~/.amplifier/dev-memory/README.md` - Full guide

### Categories

Memories are auto-categorized as:
- architecture, workflow, environment, git
- research, pattern, preference, tools

### Privacy

All data stays local. No external APIs. You control what's remembered.

---

**The system is ready - just talk naturally!**
