# Dev Memory Bundle

Persistent memory system for maintaining context across all amplifier sessions.

## What It Does

Provides a global memory system that works in any directory:
- Remember facts and preferences across sessions
- Track active work and pending decisions
- Maintain development context automatically
- Natural language and slash command interface

## Installation

```bash
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
amplifier bundle use dev-memory
```

## Usage

### Slash Commands

- `/memory-recall [query]` - Search memories
- `/memory-remember [text]` - Add a memory
- `/memory-status` - Show work log
- `/memory-guide` - Show full guide

### Natural Language

- "what do you remember about X?"
- "remember this: [fact]"
- "what was I working on?"
- "remind me how the memory system works"

## Memory Storage

All data stored locally at `~/.amplifier/dev-memory/`:
- `memory-store.yaml` - Facts to remember
- `work-log.yaml` - Active work tracking
- `project-notes.md` - Free-form notes

## Features

- ✓ Works globally (any directory)
- ✓ Auto-loads context at session start
- ✓ Natural language recognition
- ✓ Human-readable YAML files
- ✓ 100% local storage (no external APIs)
- ✓ Privacy-focused

## Agent

`memory-partner` - Handles all memory operations

## Context Files

- `memory-system-guide.md` - User guide
- `auto-context.md` - Session start context

## Configuration

```yaml
config:
  memory_path: "${HOME}/.amplifier/dev-memory"
  auto_load_context: true
```

## License

MIT

## Author

ramparte

## Repository

https://github.com/ramparte/amplifier-collection-dev-memory
