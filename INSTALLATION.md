# Installation Guide - Bundle Version

## ✅ Quick Install (Recommended)

The memory system is now a proper Amplifier **bundle** that works globally!

### Install from GitHub

```bash
# Add the bundle to your amplifier
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main

# Set it as your active bundle (or include it in your profile)
amplifier bundle use dev-memory
```

That's it! The memory system is now available in **all amplifier sessions**, regardless of directory.

---

## 🧪 Verify Installation

```bash
# Check if bundle is installed
amplifier bundle list

# You should see:
# dev-memory | git+https://github.com/ramparte/amplifier-collection-dev-memory@main

# Start amplifier and test
amplifier

# Try these commands:
/memory-recall test
/memory-remember Testing the bundle installation
/memory-status
```

---

## 📁 Set Up Memory Files (First Time)

The memory system stores data at `~/.amplifier/dev-memory/`. On first use, create the directory:

```bash
mkdir -p ~/.amplifier/dev-memory
```

Then create the initial memory files:

**memory-store.yaml:**
```yaml
# Memory Store - "Remember This" Entries

memories:
  - id: "mem-001"
    timestamp: "2026-01-05T12:00:00Z"
    category: "preference"
    content: "Testing the memory system"
    context: "Initial setup"
    tags: ["test", "setup"]

# Categories: architecture, workflow, environment, git, research, pattern, preference, tools
```

**work-log.yaml:**
```yaml
# Work Log - Current Development Context

active_work:
  - item: "Set up dev-memory bundle"
    status: "completed"
    date: "2026-01-05"

pending_decisions: []

active_projects: {}

environment:
  workspace_root: "/your/workspace"
  amplifier_version: "latest"
```

**README.md:**
```markdown
# Developer Memory System

This directory contains your persistent development memory.

## Usage
- `/memory-remember <text>` - Add a memory
- `/memory-recall <query>` - Search memories
- `/memory-status` - Show work log
- `/memory-guide` - Show full guide
```

Or let the system create them on first use when you run `/memory-remember`.

---

## 🎯 Usage

### Slash Commands

These work in **any** amplifier session:

```
/memory-recall workspace
/memory-remember I prefer clean code
/memory-status
/memory-guide
```

### Natural Language

The system also understands conversational patterns:

```
what do you remember about workspace organization?
remember this: I like keeping projects organized
what was I working on?
remind me how the memory system works
```

---

## 🔄 Updating

To update to the latest version:

```bash
amplifier bundle update dev-memory
```

---

## ⚙️ Advanced: Use with Profiles

If you use amplifier profiles, you can include dev-memory in your profile:

**~/.amplifier/profiles/my-profile.yaml:**
```yaml
profile:
  name: my-profile
  
includes:
  - bundle: foundation
  - bundle: dev-memory  # Add this line
```

---

## 🗑️ Uninstalling

```bash
# Remove the bundle
amplifier bundle remove dev-memory

# Optionally delete your memory files (WARNING: loses your memories!)
rm -rf ~/.amplifier/dev-memory
```

---

## 🐛 Troubleshooting

### Bundle not found?
```bash
# Re-add the bundle
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
```

### Commands not recognized?
```bash
# Make sure you're using the bundle
amplifier bundle use dev-memory

# Or check your profile includes it
amplifier bundle current
```

### Memory files not working?
```bash
# Check if directory exists
ls -la ~/.amplifier/dev-memory/

# Create if missing
mkdir -p ~/.amplifier/dev-memory
```

---

## 📚 For Developers

### Local Development

To test changes locally:

```bash
# Clone the repo
git clone https://github.com/ramparte/amplifier-collection-dev-memory
cd amplifier-collection-dev-memory

# Add as local bundle
amplifier bundle add file:///$(pwd)

# Use it
amplifier bundle use dev-memory
```

### Bundle Structure

```
amplifier-collection-dev-memory/
├── bundle.yaml              # Bundle manifest
├── agents/
│   └── memory-partner.md    # Memory agent with YAML frontmatter
├── context/
│   ├── memory-system-guide.md
│   └── auto-context.md
├── README.md
└── INSTALLATION.md
```

---

## 🌐 Share with Others

Others can install your memory system with one command:

```bash
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
amplifier bundle use dev-memory
```

---

**Installed successfully?** Try saying: "what do you remember about installation?"
