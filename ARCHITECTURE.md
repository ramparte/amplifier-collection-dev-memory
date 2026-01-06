# Architecture: Token-Efficient Memory System

## Problem

Traditional approach: Load entire memory file for every operation

**Token costs as memories grow:**
- 10 memories = ~1k tokens per operation
- 100 memories = ~10k tokens per operation  
- 1000 memories = ~100k tokens per operation ❌ Context overflow!

## Solution: Hybrid Architecture

### Write Operations (Main Agent)
**User:** "remember this: I prefer PostgreSQL"

**Main agent:** Appends to file without loading entire contents

```bash
# Get next ID (read last few lines only)
LAST_ID=$(tail -20 ~/.amplifier/dev-memory/memory-store.yaml | grep -oP 'id: "mem-\K\d+' | tail -1)
NEXT_ID=$(printf "mem-%03d" $((10#${LAST_ID:-0} + 1)))

# Append new entry
cat >> ~/.amplifier/dev-memory/memory-store.yaml <<EOF

  - id: "$NEXT_ID"
    timestamp: "$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
    category: "preference"
    content: "I prefer PostgreSQL"
    tags: ["postgresql", "database", "tools"]
EOF
```

**Token cost:** ~100 tokens (just the tail + append operation)

---

### Read Operations (Sub-Agent)
**User:** "what do you remember about database?"

**Main agent:** Delegates to memory-retrieval sub-agent

```
[Invoke task tool]
Agent: dev-memory:memory-retrieval
Instruction: Search memory store for: "database"
```

**Sub-agent:**
1. Loads full memory-store.yaml (10k tokens in SUB-AGENT context)
2. Searches for matches
3. Returns only 2-3 relevant entries (200 tokens to MAIN context)

**Token cost in main context:** ~200 tokens (just the matches)
**Token cost in sub-agent context:** 10k tokens (absorbed by sub-agent, cleared after task)

---

## Token Efficiency Comparison

| Operation | Traditional | Hybrid | Savings |
|-----------|------------|--------|---------|
| **Write** (10 memories) | 1,000 tokens | 100 tokens | 90% |
| **Write** (1000 memories) | 100,000 tokens | 100 tokens | 99.9% |
| **Read** (10 memories) | 1,000 tokens | 200 tokens | 80% |
| **Read** (1000 memories) | 100,000 tokens | 200 tokens | 99.8% |

**Result:** Scale to thousands of memories with constant token usage!

---

## Architecture Diagram

```
User: "remember this: X"
  ↓
Main Agent (writes directly)
  ↓ append operation (~100 tokens)
  ↓
~/.amplifier/dev-memory/memory-store.yaml
  ✓ Written efficiently


User: "what do you remember about Y?"
  ↓
Main Agent (delegates)
  ↓
memory-retrieval sub-agent
  ↓ loads full file (10k tokens in SUB context)
  ↓ searches and filters
  ↓ returns matches (~200 tokens)
  ↓
Main Agent receives results
  ↓ displays to user
  ✓ Token efficient!
```

---

## Component Responsibilities

### Main Agent
**Context includes:**
- `context/auto-context.md` - Overview
- `context/memory-writer-instructions.md` - Append-only write operations
- `context/memory-system-guide.md` - User guide

**Handles:**
- Memory writes (append-only)
- Work log updates (small file, OK to load)
- Recognizing memory patterns
- Delegating retrieval to sub-agent
- Formatting responses

### memory-retrieval Sub-Agent
**Agent file:**
- `agents/memory-retrieval.md` - Search and retrieval logic

**Handles:**
- Loading full memory store
- Searching by keywords, tags, categories
- Filtering and ranking results
- Returning top N matches
- Formatting search results

**Key:** Absorbs token cost of loading large files!

---

## Scaling Characteristics

**Traditional approach:**
```
Tokens per operation = Size of memory file
100 memories → 10k tokens → OK
1000 memories → 100k tokens → Context overflow ❌
```

**Hybrid approach:**
```
Tokens per write = ~100 (constant)
Tokens per read = ~200 (constant, just matches)
Sub-agent tokens = Isolated, cleared after task

100 memories → 100 write / 200 read → ✓
1000 memories → 100 write / 200 read → ✓
10000 memories → 100 write / 200 read → ✓
```

**Result:** Linear scaling in storage, constant tokens in main context!

---

## File Structure

```
~/.amplifier/dev-memory/
├── memory-store.yaml       # Growing file (1k-100k+ tokens)
│                          # Written: append-only by main agent
│                          # Read: by memory-retrieval sub-agent
├── work-log.yaml          # Small file (~500 tokens)
│                          # Read/write: main agent directly
├── project-notes.md       # Free-form notes
└── README.md              # System guide
```

---

## Implementation Details

### Append-Only Write (Bash)

```bash
# Efficient: Read just last 20 lines to get next ID
tail -20 memory-store.yaml | grep 'id: "mem-' | tail -1

# Append with proper YAML formatting
cat >> memory-store.yaml <<'EOF'

  - id: "mem-XXX"
    timestamp: "..."
    category: "..."
    content: "..."
    tags: [...]
EOF
```

**Why this works:**
- `tail -20` loads ~200 bytes, not 100k+ bytes
- Append operation is O(1), not O(n)
- No need to parse or rewrite entire file

### Sub-Agent Retrieval (Tool Delegation)

```
Main agent recognizes: "what do you remember about X?"

Main agent invokes:
  tool: task
  agent: dev-memory:memory-retrieval
  instruction: "Search memory store for: X"

Sub-agent:
  1. read_file(memory-store.yaml)  # Full file in sub-context
  2. Parse YAML, search content/tags/category
  3. Return formatted matches (2-3 entries)

Main agent receives:
  "Found 2 memories matching 'X': ..." (200 tokens)
```

**Why this works:**
- Sub-agent context is isolated and cleared after task
- Only results pass back to main agent
- Main agent stays token-lean

---

## Trade-offs

### Advantages ✅
- Constant token usage regardless of memory store size
- Main agent context stays lean
- Can scale to thousands of memories
- Write operations are fast (no file parsing)
- Read operations return only relevant results

### Considerations ⚠️
- Sub-agent invocation has small latency (~1-2 seconds)
- Append-only writes can't edit existing entries (by design)
- Need to maintain YAML formatting in appends
- Sub-agent needs to parse YAML (handled internally)

### Limitations
- If work-log.yaml grows large (>5k tokens), may need similar treatment
- Very large memory stores (>10k entries) may slow sub-agent search
  - Solution: Use grep/ripgrep for pre-filtering in sub-agent

---

## Future Optimizations

1. **Indexed Search** - Add SQLite index for sub-agent (O(log n) search)
2. **Semantic Search** - Embed memories, search by meaning not keywords
3. **Time-based Loading** - Sub-agent loads only recent memories by default
4. **Compression** - Archive old memories to separate file
5. **Parallel Search** - Multi-threaded grep for very large stores

---

## Comparison to Alternatives

### Traditional RAG
- External vector DB required
- API calls add latency and cost
- Complex infrastructure

**Our approach:**
- Local YAML file (simple, portable)
- Bash + sub-agent (built-in tools)
- No external dependencies

### Pure Context Loading
- Load everything into context
- Simple but doesn't scale
- Context overflow at ~1000 memories

**Our approach:**
- Hybrid load strategy
- Scales to 10k+ memories
- Constant token usage

---

## Summary

**Token efficiency through delegation:**
- Main agent: Lightweight append operations
- Sub-agent: Heavy lifting (searches), isolated context
- Result: Scale to thousands of memories, constant main context size

**Best of both worlds:**
- ✅ Write permissions (main agent)
- ✅ Token efficiency (sub-agent isolation)
- ✅ Simple YAML storage
- ✅ No external dependencies
- ✅ Scales linearly

🎯 **Target: 1000+ memories at constant ~300 token overhead per operation**
