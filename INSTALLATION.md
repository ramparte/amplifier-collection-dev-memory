# Installation Guide

## Quick Install

```bash
# 1. Create symlink in amplifier collections
cd ~/.amplifier/collections
ln -s /mnt/c/ANext/amplifier-collection-dev-memory dev-memory

# 2. Memory files are already set up at:
ls ~/.amplifier/dev-memory/

# 3. Reload amplifier (if running)
# Just restart amplifier or it will auto-detect on next launch
```

## Verify Installation

```bash
# Check collection is linked
ls -la ~/.amplifier/collections/ | grep dev-memory

# Check memory files exist
ls ~/.amplifier/dev-memory/

# You should see:
# - memory-store.yaml
# - work-log.yaml
# - project-notes.md
# - README.md
# - DESIGN.md
```

## First Use

Start amplifier and try:

```
amplifier

# Then in the session:
You: remind me how the memory system works

# Or:
You: remember this: testing the memory system
You: what do you remember about testing?
You: what was I working on?
```

## How It Works

### Auto-Loading
The collection is configured with `auto_load: true`, which means:

1. Every amplifier session automatically loads the dev-memory collection
2. The memory-partner agent is always available
3. Context files are loaded into every session
4. Memory commands work everywhere

### Global Memory
Memory is stored at `~/.amplifier/dev-memory/` so it's available:
- In all projects
- In all directories
- Across all sessions
- For all amplifier invocations

### No Per-Project Setup
Once installed, it just works. No need to:
- Activate it per-project
- Load it manually
- Remember special commands
- Set up per-directory configs

## Troubleshooting

### Collection not loading?
```bash
# Check if symlink is correct
ls -la ~/.amplifier/collections/dev-memory

# Should point to: /mnt/c/ANext/amplifier-collection-dev-memory
```

### Memory files not found?
```bash
# Create the directory
mkdir -p ~/.amplifier/dev-memory

# Copy initial files
cp /mnt/c/ANext/.dev-memory/* ~/.amplifier/dev-memory/
```

### Commands not recognized?
The collection uses natural language understanding, not slash commands. Just talk naturally:
- "remember this: [text]" ← Natural
- "what do you remember about X?" ← Natural
- "what was I working on?" ← Natural

## Updating

To update the collection:

```bash
cd /mnt/c/ANext/amplifier-collection-dev-memory
git pull  # if it becomes a git repo

# Restart amplifier to reload
```

## Uninstalling

```bash
# Remove symlink
rm ~/.amplifier/collections/dev-memory

# Optionally remove memory files (you'll lose your memories!)
rm -rf ~/.amplifier/dev-memory
```

## Development

The collection source is at:
```
/mnt/c/ANext/amplifier-collection-dev-memory/
```

Edit files there and changes take effect on next amplifier restart.

---

*The symlink approach means you can develop and use simultaneously!*
