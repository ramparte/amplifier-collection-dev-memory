# Auto-Context Loader

This file is included in every session to provide memory system context.

---

## Memory System Available

A persistent memory system is active in this session. It maintains context across all amplifier sessions.

**Important:** You (the main agent) handle all memory operations directly. Do not delegate to sub-agents.

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
