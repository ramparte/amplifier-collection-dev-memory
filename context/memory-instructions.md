# Dev Memory System

Persistent memory at `~/amplifier-dev-memory/`. Three operations:

## Writes: "remember this: [text]"

Append directly to memory-store.yaml (never load the full file):

```bash
LAST_ID=$(grep -oP 'id: "mem-\K\d+' ~/amplifier-dev-memory/memory-store.yaml | tail -1)
NEXT_ID=$(printf "mem-%03d" $((10#${LAST_ID:-0} + 1)))
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
cat >> ~/amplifier-dev-memory/memory-store.yaml <<EOF

  - id: "$NEXT_ID"
    timestamp: "$TIMESTAMP"
    category: "CATEGORY"
    content: "TEXT"
    tags: [TAG1, TAG2]
EOF
```

Categories: architecture, workflow, environment, git, research, pattern, preference, tools.

## Reads: "what do you remember about [topic]?"

NEVER use read_file on memory-store.yaml. ALWAYS delegate:

```
delegate(agent="dev-memory:memory-retrieval", instruction="Search memory store for: [query]")
```

The sub-agent loads the file in its own context and returns only matches (~200 tokens).

## Work Status: "what was I working on?"

Read `~/amplifier-dev-memory/work-log.yaml` directly (small file, safe to load).
