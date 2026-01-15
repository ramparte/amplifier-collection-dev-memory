# Installation Guide

## ✅ Quick Install

```bash
# Add the bundle from GitHub
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main

# Activate it globally
amplifier bundle use dev-memory
```

That's it! The memory system is now available in **all amplifier sessions**, regardless of directory.

---

## 📁 First-Time Setup

Create the memory directory:

```bash
mkdir -p ~/amplifier-dev-memory
```

The memory system will create the initial files on first use.

---

## 🧪 Verify Installation

```bash
# Check if bundle is installed
amplifier bundle list | grep dev-memory

# See bundle details
amplifier bundle show dev-memory

# Verify it's active
amplifier bundle current
```

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

## 🗑️ Uninstalling

```bash
# Remove the bundle
amplifier bundle remove dev-memory

# Optionally delete your memory files (WARNING: loses your memories!)
rm -rf ~/amplifier-dev-memory
```

---

## 🐛 Troubleshooting

### Bundle not found?

Make sure you've added it:
```bash
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
```

### Commands not recognized?

Make sure the bundle is active:
```bash
amplifier bundle use dev-memory
amplifier bundle current  # Should show dev-memory
```

### Memory files not working?

Check if directory exists:
```bash
ls -la ~/amplifier-dev-memory/

# Create if missing
mkdir -p ~/amplifier-dev-memory
```

---

## 👥 For Other Users

Share this one-liner for quick installation:

```bash
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main && amplifier bundle use dev-memory
```

---

## 🔧 Local Development

To test changes locally:

```bash
# Clone the repo
git clone https://github.com/ramparte/amplifier-collection-dev-memory
cd amplifier-collection-dev-memory

# Add as local bundle
amplifier bundle add file://$(pwd)

# Use it
amplifier bundle use dev-memory
```

---

**Ready to use!** Try saying: "what do you remember about installation?"
