# Amplifier Memory V2 Design

## Inspiration

This design is directly inspired by an analysis of Claude Code's memory architecture. Claude Code's system is "insanely well-designed" -- not "store everything" but constrained, structured, and self-healing. The key insights adopted here: memory as index not storage, bandwidth-aware layering, strict write discipline, background consolidation (autoDream), staleness as first-class, isolation of consolidation, skeptical retrieval, and the principle that what you *don't* store is the real insight.

## Goal

Replace the flat append-only YAML memory store with a 3-layer index+topics architecture that is self-healing, context-efficient, and scales to hundreds of memories without degrading retrieval quality or bloating agent context.

## Background

The current dev-memory system stores 44+ entries in a single `memory-store.yaml` file. It works today but has zero consolidation, deduplication, staleness awareness, or background cleanup. Every retrieval loads the entire file. As memories grow, this approach will bloat sub-agent context, degrade retrieval quality, and accumulate stale/contradictory entries with no mechanism for self-correction.

## Non-Goals

The following are explicitly out of scope for V2 and deferred to future iterations:

- **Vector/semantic search** -- The experimental Qdrant + OpenAI embeddings system exists but is not deployed. LLM judgment on a small index is sufficient. Embeddings can be added later as an optimization layer.
- **Multi-namespace** -- All memories live in a single namespace. No per-project partitions.
- **Agent-scoped memory** -- Agents are ephemeral; their knowledge is derivable from the codebase. No per-agent memory stores.
- **Real-time indexing hooks** -- No filesystem watchers or event-driven indexing. The write path is explicit and LLM-assisted.

## Design Decisions

| # | Decision | Rationale |
|---|----------|-----------|
| 1 | **Filesystem-pure** | No vector DB, no embeddings, no external services. Intelligence comes from LLM judgment during consolidation, not retrieval technology. Zero infra dependencies. |
| 2 | **Single namespace** | No agent-scoped or project-scoped partitions. Tags and topic organization handle categorization. Avoids partition logic, cross-namespace queries, and "where did I put that?" confusion. |
| 3 | **One-liner index with file pointers** | ~150 char summaries in the index give the retrieval agent enough to filter without opening topic files. File pointers enable the 3-layer design. |
| 4 | **LLM-assisted writes** | Main agent reads the index, files to the correct topic, and updates the index in one operation. No sub-agent for writes -- the main agent has the user's context. |
| 5 | **Threshold-triggered consolidation** | Fires after 10 new writes or when the index exceeds 200 lines. Session-end hooks run too often; user-triggered consolidation won't happen in practice. |

## Architecture

### The 3-Layer Design

```
Layer 0 - INDEX (always loaded)
  ~/amplifier-dev-memory/MEMORY.md
  ~200 lines max, hard-capped
  Each line: one-liner summary + relative path to topic file
  Loaded by retrieval agent on every search

Layer 1 - TOPIC FILES (loaded on-demand)
  ~/amplifier-dev-memory/topics/<topic-name>.md
  Each file: 5-50 entries on a single theme
  Only opened when a search hits the relevant index line
  Free-form markdown, not YAML

Layer 2 - SESSION TRANSCRIPTS (never loaded, only searched)
  ~/.amplifier/projects/*/sessions/*/events.jsonl
  Already exist -- Amplifier's session logs
  The consolidation agent can use session-analyst to search them
  (not raw grep -- session-analyst handles the 100k+ token lines safely)
  Used to verify/enrich memories, never read into memory context
```

**Key rules:**

- The index is the *only* file that's always read. Everything else is on-demand.
- Topic files are the unit of consolidation -- the autoDream agent rewrites entire topic files, not individual entries.
- Nothing derivable from code is stored. If you can `grep` for it, don't remember it.
- The hard cap on index lines (200) is enforced by consolidation -- when the index grows past the cap, consolidation must run and prune.

### File Structure

```
~/amplifier-dev-memory/
├── MEMORY.md                           # Layer 0: The index (always loaded)
├── topics/                             # Layer 1: Topic files (on-demand)
│   ├── workspace-preferences.md
│   ├── dgx-cluster-setup.md
│   ├── amplifier-architecture.md
│   ├── team-org.md
│   └── ...
├── .meta/                              # Housekeeping (never loaded by retrieval)
│   ├── writes-since-consolidation      # Counter file (just a number)
│   └── consolidation-log.md           # Audit trail of what consolidation did
└── .gitignore                          # Optional: if the dir is a git repo
```

## Components

### The Write Path

When the user says "remember this: X":

1. **Read index** -- Main agent reads MEMORY.md (~200 lines, cheap).
2. **Decide placement** -- Does X fit an existing topic? Which one? Or does X need a new topic file?
3. **Write to topic file** -- Append the new entry to `topics/<topic-name>.md`. Entry format: timestamp + content (markdown, not YAML).
4. **Update index** -- If existing topic: rewrite that index line to reflect new content. If new topic: append a new index line + file pointer.
5. **Check consolidation threshold** -- Read `.meta/writes-since-consolidation`, increment it. If counter > 10: trigger consolidation (spawn sub-agent). If not: done.

**Design choices:**

- The main agent does the write because it has the user's context and can make a good filing decision. No sub-agent needed for writes.
- Topic files are markdown, not YAML. Easier for the consolidation agent to read/rewrite, and human-editable.
- The index line for an existing topic gets *rewritten* on each write to stay current. This is the "write to file, then update index" discipline.
- The consolidation threshold counter is a simple file, not in-memory state, so it persists across sessions.

### The Consolidation Agent (autoDream)

**When it triggers:**

- A counter file (`.meta/writes-since-consolidation`) tracks new writes
- Threshold: 10 new writes since last consolidation
- Also triggers if index exceeds 200 lines (hard cap enforcement)
- Spawned via `delegate(agent="self", ...)` with `context_depth="none"` -- clean slate, no session contamination

**What tools it gets:**

- Read/write to `~/amplifier-dev-memory/` (MEMORY.md, topics/\*, .meta/\*)
- `session-analyst` delegation (for Layer 2 transcript verification)
- Nothing else. No bash, no web, no code tools. Isolation prevents corruption.

**What it does (in order):**

1. **Load** -- Read MEMORY.md (index) and all referenced topic files.
2. **Merge** -- Within each topic file, merge entries that say the same thing differently. "I prefer Postgres" + "Use PostgreSQL for new projects" becomes one entry.
3. **Deduplicate across topics** -- If two topic files cover overlapping ground, merge the smaller into the larger and delete the empty file.
4. **Prune stale** -- Delete entries that are vague, temporal ("I'm currently working on X"), or derivable from code. Convert "I think the config is at..." to either the absolute path or delete it.
5. **Resolve contradictions** -- If two entries contradict each other, keep the newer one. If unclear, keep neither and flag it in `.meta/consolidation-log.md`.
6. **Rewrite index** -- Regenerate MEMORY.md from the consolidated topic files. Each line is a fresh one-liner summary of that topic file's current content.
7. **Enforce cap** -- If the index still exceeds 200 lines after consolidation, force-prune the lowest-value topics (oldest, least specific, most derivable).
8. **Reset counter** -- Zero out `.meta/writes-since-consolidation`.
9. **Log** -- Append a summary to `.meta/consolidation-log.md` (what was merged, pruned, flagged).

**Key principle:** Memory is continuously edited, not appended. The consolidation agent rewrites topic files in place. Old versions are gone. If you need history, that's what git is for.

### The Read Path (Retrieval)

**Stage 1: Index scan (always)**

- The retrieval agent reads MEMORY.md (~200 lines, cheap)
- Uses LLM judgment to identify which index lines are relevant to the query
- Replaces keyword matching -- the LLM can understand semantic relevance

**Stage 2: Topic file fetch (on-demand)**

- For each relevant index line, the agent reads the referenced topic file
- Extracts the specific entries that answer the query
- Returns a concise summary to the main session (~200 tokens)

**Stage 3: Transcript search (rare, only when needed)**

- If the query references something that might be in session history but not in memory, the retrieval agent can delegate to `session-analyst` to search transcripts
- This is Layer 2 -- never automatic, only when Layers 0 and 1 don't have the answer

**Key design choice:** Retrieval is skeptical, not blind. Memories are *hints*, not truth. The retrieval agent returns memories with the caveat that the main agent should verify against current reality before acting on them. Memories older than 30 days get a staleness warning. This is enforced in the retrieval agent's instructions: "Present memories as potentially stale hints. Flag any memory older than 30 days with a staleness warning."

## Data Flow

```
User: "remember this: X"
         │
         ▼
┌─────────────────────┐
│  Main Agent          │
│  1. Read MEMORY.md   │──── Layer 0 (always cheap)
│  2. Decide topic     │
│  3. Append to topic  │──── Layer 1 (write)
│  4. Update index     │──── Layer 0 (rewrite line)
│  5. Increment counter│──── .meta/
└─────────┬───────────┘
          │
          ▼ (if counter > 10)
┌─────────────────────────┐
│  Consolidation Agent     │
│  (isolated, no session   │
│   context, limited tools)│
│                          │
│  Read all → Merge →      │
│  Dedup → Prune stale →   │
│  Resolve contradictions → │
│  Rewrite index →         │
│  Enforce cap → Log       │
└──────────────────────────┘

User: "what do I know about X?"
         │
         ▼
┌─────────────────────┐
│  Retrieval Agent     │
│  1. Read MEMORY.md   │──── Layer 0 (always)
│  2. LLM judges       │
│     relevance        │
│  3. Read topic files │──── Layer 1 (on-demand)
│  4. Return summary   │
│  5. (rare) delegate  │──── Layer 2 (session-analyst)
│     to session-analyst│
└─────────────────────┘
```

## The "Don't Store Derivable" Discipline

### What's Derivable (Don't Persist)

- **Code structure** ("the auth module is at X") -- grep/LSP can find this
- **File contents** ("the config has these settings") -- read the file
- **Git history** ("we merged PR #42 last week") -- git log
- **Debug traces** ("the error was caused by X") -- session transcripts
- **Anything re-derivable** from the codebase or environment

### What's NOT Derivable (Do Persist)

- **Preferences** ("I prefer PostgreSQL for new projects")
- **Decisions with rationale** ("We chose single-namespace because...")
- **Team knowledge** ("Salil owns the provider modules")
- **Workflow conventions** ("Always push core before modules")
- **Non-obvious gotchas** ("The DGX NFS mount silently drops writes over 2GB")
- **External context** ("The deadline for X is April 15")

### Enforcement

This is instruction-level, not code-level. The write path instructions tell the main agent: "Before storing a memory, ask: could this be found by reading code, searching files, or checking git? If yes, don't store it. If the user insists, store it with a `[derivable]` tag so consolidation can prune it later."

The consolidation agent also enforces this -- one of its prune criteria is "is this entry derivable from the codebase?"

## Bundle Changes and Agent Definitions

### Agents (3 Total)

| Agent | Role | Tools Allowed |
|-------|------|---------------|
| `memory-retrieval` (existing, rewritten) | 2-stage retrieval: scan index, fetch relevant topic files | `read_file` on `~/amplifier-dev-memory/`, delegate to `session-analyst` |
| `memory-consolidation` (new) | autoDream: merge, dedup, prune, rewrite | `read_file` + `write_file` on `~/amplifier-dev-memory/`, delegate to `session-analyst` |
| `memory-migration` (new, one-time) | Converts legacy YAML to index+topics | `read_file` + `write_file` on `~/amplifier-dev-memory/` |

### Context Files (Updated)

- **`memory-writer-instructions.md`** -- Rewritten for the new write path (read index, file to topic, update index, check threshold)
- **`DELEGATION-RULES.md`** -- Updated: "never read topic files directly, always delegate retrieval"
- **`auto-context.md`** -- Updated to describe the new architecture
- **`derivability-rules.md`** (new) -- The "don't store derivable" discipline, referenced by both writer instructions and consolidation agent

### What's Removed

- All YAML-specific instructions and examples
- References to `work-log.yaml` and `project-notes.md`
- The keyword-matching search logic in the retrieval agent (replaced by LLM judgment on index lines)

### Consolidation Trigger Integration

Added to the write path instructions: "After incrementing the counter, if counter > 10, spawn `memory-consolidation` agent with `context_depth='none'`." The consolidation agent runs asynchronously -- the main session doesn't wait for it.

## Migration Plan

1. The `memory-migration` agent reads the existing `memory-store.yaml` (all 44 entries)
2. It groups entries by theme and creates topic files under `topics/`
3. It generates `MEMORY.md` with one-liner summaries pointing to each topic file
4. It creates `.meta/writes-since-consolidation` initialized to `0`
5. It renames `memory-store.yaml` to `.meta/legacy-memory-store.yaml` (preserved, not deleted)
6. It cleans up the nested `~/amplifier-dev-memory/amplifier-dev-memory/` directory artifact
7. It removes `work-log.yaml` (empty, never used) and folds `project-notes.md` into a topic file if it has content

This is a one-time operation run as the first step of the V2 rollout.

## Example Formats

### MEMORY.md (Index)

```markdown
# Memory Index

- Prefers PostgreSQL, clean workspaces, Inactive/ folder convention → topics/workspace-preferences.md
- DGX Spark: spark-1 (.10), spark-2 (.11), NFS /mnt/shared, Ubuntu 22.04 → topics/dgx-cluster-setup.md
- Amplifier: single-namespace memory, filesystem-pure, index+topics → topics/amplifier-architecture.md
- Team: Sam (ramparte), Salil (robotdad), org structure → topics/team-org.md
```

### Topic File (topics/workspace-preferences.md)

```markdown
# Workspace Preferences

- Prefers PostgreSQL for all new projects [2026-01-15]
- Uses Inactive/ folders to archive old work without deleting [2026-01-20]
- Clean workspaces: no stale branches, no orphan files [2026-02-03]
- Tab preference: 2-space indent for YAML, 4-space for Python [2026-03-10]
```

## Open Questions

None -- all design questions were resolved during the brainstorming phase. Implementation details (exact prompt wording for agents, threshold tuning, topic naming conventions) will be determined during implementation.
