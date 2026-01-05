# Developer Memory System Guide

## Overview
You have access to a persistent memory system that maintains context across sessions. Memory is stored at `~/.amplifier/dev-memory/`.

## Commands Available

### /remember <what-to-remember>
Add a memory to the store.

**Example:**
```
/remember I prefer keeping workspaces clean with Inactive/ folders
```

**What it does:**
- Adds entry to `~/.amplifier/dev-memory/memory-store.yaml`
- Includes timestamp, category, context, and tags
- Automatically categorizes by content

### /recall <query>
Search and retrieve memories.

**Example:**
```
/recall workspace organization
/recall DotRunner
```

**What it does:**
- Searches memory-store.yaml for matching content
- Returns relevant memories with context

### /work-status
Show current work items and pending decisions.

**Example:**
```
/work-status
```

**What it does:**
- Reads `~/.amplifier/dev-memory/work-log.yaml`
- Shows active work, pending decisions, project status

### /memory-guide
Display this guide.

**Example:**
```
/memory-guide
remind me how the memory system works
```

## File Structure

### ~/.amplifier/dev-memory/
- **memory-store.yaml** - "Remember this" facts
- **work-log.yaml** - Active work tracking
- **project-notes.md** - Free-form notes
- **README.md** - Documentation

## Automatic Context Loading

When you start an amplifier session, the memory system automatically:
1. Loads recent memories (last 30 days)
2. Loads current work items
3. Loads project status
4. Makes this context available to all agents

## Usage Patterns

### Start of Session
Say: "What was I working on?" or "What's my current status?"
→ AI reads work-log.yaml and tells you

### During Work
Say: "Remember this: [fact]" 
→ AI adds to memory-store.yaml

### End of Session
Say: "Update work log - completed [X], blocked on [Y]"
→ AI updates work-log.yaml

### Searching Context
Say: "What do you remember about [topic]?"
→ AI searches memories and responds

## Privacy
- All data stored locally
- No external services
- You control what's remembered
- Files are human-readable YAML/Markdown

## Integration
This collection auto-loads, so memory commands work in:
- All amplifier sessions
- All projects
- Any working directory

---

*To see this guide: say "remind me how the memory system works" or "/memory-guide"*
