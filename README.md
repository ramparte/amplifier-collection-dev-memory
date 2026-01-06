# Developer Memory Collection

A persistent memory system for Amplifier that maintains context across all sessions.

## What It Does

Provides a local, persistent memory that helps you and your AI assistant maintain continuity across development sessions:

- **Remember facts**: "Remember that I prefer X"
- **Recall context**: "What do you remember about Y?"
- **Track work**: "What was I working on?"
- **Maintain continuity**: Context loads automatically in every session

## Installation

### Option 1: Install as Collection (Recommended)

```bash
cd ~/.amplifier/collections
ln -s /mnt/c/ANext/amplifier-collection-dev-memory dev-memory
amplifier collection reload
```

### Option 2: Manual Setup

```bash
# Copy to collections directory
cp -r amplifier-collection-dev-memory ~/.amplifier/collections/dev-memory

# Memory files are stored globally at:
# ~/.amplifier/dev-memory/
```

## Usage

### Natural Language Interface

Talk naturally with the AI to use the memory system:

**Remember something:**
```
remember this: I prefer keeping workspaces clean
```

**Recall memories:**
```
what do you remember about workspace organization?
```

**Check work status:**
```
what was I working on?
```

**Get help:**
```
remind me how the memory system works
```

The AI recognizes these conversational patterns automatically - no special commands needed!

## Token Efficiency

**Built for scale:** This system uses a hybrid architecture that keeps token usage constant regardless of memory store size!

| Operation | 10 Memories | 1000 Memories | Savings |
|-----------|-------------|---------------|---------|
| Write | ~100 tokens | ~100 tokens | Constant ✅ |
| Read | ~200 tokens | ~200 tokens | Constant ✅ |

**How it works:**
- **Writes:** Append-only operations (no full file load)
- **Reads:** Sub-agent absorbs token cost, returns only matches
- **Result:** Scale to 10,000+ memories without context bloat!

See [ARCHITECTURE.md](ARCHITECTURE.md) for technical details.

### Files

Memory is stored at `~/.amplifier/dev-memory/`:

- **memory-store.yaml** - Facts and learnings
- **work-log.yaml** - Active work tracking
- **project-notes.md** - Free-form notes
- **README.md** - Documentation
- **DESIGN.md** - System architecture

All files are human-readable and editable!

## Auto-Loading

This collection is configured with `auto_load: true`, so:
- Memory loads automatically in every amplifier session
- Works in any project directory
- No manual activation needed

## Privacy

- All data stored locally at `~/.amplifier/dev-memory/`
- No external APIs
- No data leaves your machine
- You control what's remembered

## Features

### Persistent Context
The AI remembers across sessions:
- Your preferences and patterns
- Pending decisions
- Active projects
- Recent discoveries

### Smart Categorization
Memories are automatically categorized:
- architecture
- workflow
- environment
- git
- research
- pattern
- preference
- tools

### Natural Language
No need to learn special syntax:
- Talk naturally
- AI understands intent
- Commands work conversationally

### Work Tracking
Keep track of:
- Active work items
- Pending decisions
- Project status
- Next steps

## Example Session

```
You: What was I working on?

AI: You were working on:
    1. Amplifier fork sync (completed)
    2. Workspace cleanup (completed)
    
    Pending decisions:
    - scenario-tools extraction (deferred)

You: Remember that scenario-tools needs review before extracting

AI: ✓ Remembered: scenario-tools review requirement
    Added to memory store with tags: scenario-tools, review, deferred
    Category: research

You: What do you remember about scenario-tools?

AI: I remember:
    1. scenario-tools needs review before extracting to top-level
       (Added: Jan 5, 2026, Context: During workspace cleanup)
       
    2. recurse directory contains scenario-tools/hierarchical-task-decomposer
       (Added: Jan 5, 2026, Context: Archived in Inactive/recurse)
```

## Architecture

See `DESIGN.md` for complete system architecture and integration roadmap.

## Version

1.0.0 - Initial release

## Author

ramparte

## License

MIT
