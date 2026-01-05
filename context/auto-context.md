# Developer Memory - Auto-Loaded Context

> **System Note**: This context is automatically loaded at the start of every amplifier session.

## Your Memory System

You have a persistent memory system at `~/.amplifier/dev-memory/` that maintains context across sessions.

### Quick Reference

**Commands you understand:**
- "remember this: [fact]" → Add to memory store
- "what was I working on?" → Show work status
- "what do you remember about [X]?" → Search memories
- "remind me how the memory system works" → Show full guide

### Auto-Loaded Context

At session start, I automatically load:
1. Recent memories (last 30 days) from `memory-store.yaml`
2. Current work items from `work-log.yaml`
3. Active project status

This gives me continuity about your:
- Development preferences
- Pending decisions
- Active projects
- Recent discoveries

### How to Use

**Natural Language:**
Just talk naturally - I understand memory commands in conversational form:
- "Remember that I prefer X"
- "What was that thing we discussed about Y?"
- "What's my current work status?"

**Explicit Commands:**
Or use explicit commands:
- `/remember <text>`
- `/recall <query>`
- `/work-status`
- `/memory-guide`

### Files You Can Edit

All files in `~/.amplifier/dev-memory/` are human-readable and editable:
- Edit directly in your text editor
- Commit to git if you want version control
- Add to .gitignore if you want privacy

---

*I'm your development partner - I remember our context so you don't have to.*
