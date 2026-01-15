# Memory System Guide

A persistent memory system that maintains context across all amplifier sessions.

## How to Use

The memory system responds to natural language - just talk normally!

### Remember Something

**Say:** "remember this: [text]"

**Example:**
```
remember this: I prefer clean workspaces with Inactive/ folders
```

**What happens:**
- The AI reads the current memory store
- Adds your memory with an auto-generated ID
- Categorizes it automatically
- Adds relevant tags
- Confirms what was remembered

---

### Recall Memories

**Say:** "what do you remember about [topic]?"

**Examples:**
```
what do you remember about workspace organization?
what do you remember about amplifier?
do we have any memories about git workflows?
```

**What happens:**
- The AI searches through memory-store.yaml
- Finds matches in content, tags, and context
- Shows you relevant memories with their categories and timestamps

---

### Check Work Status

**Say:** "what was I working on?"

**Examples:**
```
what was I working on?
what's my current status?
show me my pending decisions
```

**What happens:**
- The AI reads work-log.yaml
- Shows active work items
- Lists pending decisions
- Displays project status
- Suggests next steps

---

### Get Help

**Say:** "remind me how the memory system works"

**Examples:**
```
remind me how the memory system works
how do I use the memory system?
memory help
```

**What happens:**
- The AI shows this guide!

---

## Memory Storage

All data is stored locally at `~/amplifier-dev-memory/`:

- **memory-store.yaml** - Facts and learnings you've asked to remember
- **work-log.yaml** - Active work tracking and pending decisions
- **project-notes.md** - Free-form notes
- **README.md** - System documentation (this guide)

All files are human-readable YAML or Markdown - you can edit them directly!

---

## Memory Categories

Memories are automatically categorized:

- **architecture** - System design, patterns, structure
- **workflow** - How you like to work
- **environment** - Development environment setup
- **git** - Version control patterns and practices
- **research** - Things to investigate later
- **pattern** - Code patterns and conventions
- **preference** - Personal preferences
- **tools** - Tool configurations and usage

---

## Auto-Loading

The memory system automatically loads at session start:
- Recent memories (last 30 days)
- Active work items
- Pending decisions

This happens silently - the AI just "remembers" without announcing it.

---

## Privacy

- All data stays local at `~/amplifier-dev-memory/`
- No external API calls
- No data leaves your machine
- You control what's remembered
- Files are human-readable and editable

---

## Example Conversation

```
You: what was I working on?

AI: Current Status:

    Active Work:
    - Amplifier fork rebased and synced (completed)
    - Memory system bundle created (completed)
    
    Pending Decisions:
    - scenario-tools extraction (deferred)
    
    Active Projects: 10 in /mnt/c/ANext

You: remember this: scenario-tools needs review before extracting

AI: ✓ Remembered: scenario-tools review requirement
    Added to memory store as mem-006
    Category: research
    Tags: scenario-tools, review, extraction

You: what do you remember about scenario-tools?

AI: I found 2 memories about scenario-tools:

    1. "scenario-tools needs review before extracting"
       Category: research
       Added: 2026-01-05
       Context: Current session
       
    2. "recurse directory contains scenario-tools/hierarchical-task-decomposer"
       Category: research
       Added: 2026-01-05
       Context: Archived in Inactive/recurse
```

---

## Tips

1. **Be specific** - "remember this: I prefer X because Y" is better than "remember X"
2. **Search broadly** - The AI searches content, tags, and context
3. **Edit directly** - Memory files are yours to edit
4. **Use naturally** - Don't overthink it, just talk normally

---

**That's it!** Just talk naturally and the AI will handle the memory operations.
