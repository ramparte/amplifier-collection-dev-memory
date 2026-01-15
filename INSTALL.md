# Installation Guide

## Quick Install

```bash
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
amplifier bundle use dev-memory
```

That's it! The bundle will automatically:
1. Configure write permissions for `~/amplifier-dev-memory/`
2. Create the memory directory if it doesn't exist
3. Load memory context in all sessions

---

## What Gets Created

After installation, you'll have:

```
~/amplifier-dev-memory/
├── memory-store.yaml      # Your memories (auto-created on first write)
├── work-log.yaml          # Work tracking (auto-created on first write)
├── project-notes.md       # Free-form notes (auto-created on first write)
└── README.md              # System guide (auto-created)
```

Files are created automatically when first used.

---

## Verification

Check that the bundle is active:

```bash
amplifier bundle list
# Should show: dev-memory (active)
```

Test it works:

```bash
amplifier
> remember this: installation successful!
> what do you remember about installation?
```

You should see the memory stored and retrieved!

---

## Write Permissions

The bundle automatically configures `~/amplifier-dev-memory/` as an allowed write directory in the bundle configuration:

```yaml
config:
  allowed_write_dirs:
    - ~/amplifier-dev-memory
```

This allows the main agent to write memory files without permission errors.

**Note:** If you had the bundle installed before this update, you may need to reinstall:

```bash
amplifier bundle remove dev-memory
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
amplifier bundle use dev-memory
```

---

## Manual Setup (Advanced)

If you want to initialize the directory manually:

```bash
# Create directory
mkdir -p ~/amplifier-dev-memory

# Initialize empty memory store
cat > ~/amplifier-dev-memory/memory-store.yaml <<'EOF'
memories: []
EOF

# Initialize work log
cat > ~/amplifier-dev-memory/work-log.yaml <<'EOF'
active_work: []
pending_decisions: []
completed_items: []
EOF

# Add notes file
touch ~/amplifier-dev-memory/project-notes.md
```

But this is optional - files are auto-created when first used!

---

## Updating

To get the latest version:

```bash
amplifier bundle update dev-memory
```

Or reinstall:

```bash
amplifier bundle remove dev-memory
amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
amplifier bundle use dev-memory
```

---

## Troubleshooting

### "Access denied" errors

If you see write permission errors:

1. **Check bundle configuration:**
   ```bash
   amplifier bundle show dev-memory
   # Should show allowed_write_dirs: ~/amplifier-dev-memory
   ```

2. **Reinstall bundle:**
   ```bash
   amplifier bundle remove dev-memory
   amplifier bundle add git+https://github.com/ramparte/amplifier-collection-dev-memory@main
   amplifier bundle use dev-memory
   ```

3. **Verify directory exists:**
   ```bash
   ls -la ~/amplifier-dev-memory/
   ```

### Sub-agent can't write

This is expected! The architecture is:
- **Main agent:** Writes (has permissions)
- **memory-retrieval sub-agent:** Reads only (searches efficiently)

If the main agent is trying to delegate writes to a sub-agent, that's a bug in the instructions.

### Memory files not found

Files are created on first use. Try:

```
remember this: first memory!
```

This will initialize the memory-store.yaml file.

---

## Uninstall

To remove the bundle:

```bash
amplifier bundle remove dev-memory
```

**Note:** This does NOT delete your memory files at `~/amplifier-dev-memory/`. If you want to delete them:

```bash
rm -rf ~/amplifier-dev-memory/
```

---

## Support

Issues or questions? Open an issue at:
https://github.com/ramparte/amplifier-collection-dev-memory/issues
