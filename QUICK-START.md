# Quick Start Guide

Get started with the dev-memory bundle in 2 minutes.

---

## Install

```bash
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
amplifier bundle use dev-memory
```

That's it! The memory system is now active globally.

---

## Use It

Just talk naturally! The AI recognizes these patterns:

### Remember Something

```
remember this: I like keeping my workspace organized
```

### Recall Memories

```
what do you remember about workspace?
```

### Check Work Status

```
what was I working on?
```

### Get Help

```
remind me how the memory system works
```

---

## Where Data Lives

All memory files are at `~/.amplifier/dev-memory/`:

- `memory-store.yaml` - Facts you've asked to remember
- `work-log.yaml` - Active work and pending decisions
- `project-notes.md` - Free-form notes

All files are human-readable - you can edit them directly!

---

## Example Session

```
You: what was I working on?

AI: Current Status:
    Active Work:
    - Project cleanup (completed)
    
    Pending Decisions:
    - Database migration approach
    
You: remember this: prefer PostgreSQL for new projects

AI: ✓ Remembered: database preference
    Category: preference
    Tags: postgresql, database, tools

You: what do you remember about database?

AI: I found 1 memory:
    "prefer PostgreSQL for new projects"
    Category: preference
    Added: 2026-01-05
```

---

## Tips

1. **Be specific** - More context = better memory
2. **Search broadly** - The AI searches content, tags, and context
3. **Edit directly** - Memory files are yours to modify
4. **Use naturally** - Just talk, don't overthink it

---

**Ready!** Start your next amplifier session and try it out.
