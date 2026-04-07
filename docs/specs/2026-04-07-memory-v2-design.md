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
│   ├── consolidation-log.md           # Audit trail of what consolidation did
│   ├── consolidation-health.json      # Last run status, metrics, test results
│   └── retrieval-test.yaml            # Smoke test queries with expected results
└── .gitignore                          # Optional: if the dir is a git repo
```

## Components

### The Write Path

When the user says "remember this: X":

1. **Read index** -- Main agent reads MEMORY.md (~200 lines, cheap).
2. **Decide placement + contradiction check** -- Does X fit an existing topic? Which one? Or does X need a new topic file? Before storing, the agent also checks: does this new memory directly contradict any existing index line? If a conflict is detected (e.g., new memory says "We chose Auth0" but an index line references "chose Clerk"), present the conflict to the user: "This appears to contradict an existing memory about [topic]. Store anyway, or update the existing entry?" This check is O(index size) -- the index is already in context, so it adds zero extra IO.
3. **Write to topic file** -- Append the new entry to `topics/<topic-name>.md`. Entry format: timestamp + content + optional temporal tag. Entries that reference a current state get a `[temporal:YYYY-MM-DD]` tag (e.g., "Kai is working on Project Orion [temporal:2026-04-07]", "The deploy deadline is April 15 [temporal:2026-04-15]"). Entries that are persistent facts get a `[persistent]` tag or no tag (e.g., "Prefer PostgreSQL for new projects"). The temporal tag is the date the fact was recorded; it tells the consolidation agent when to start questioning whether the fact is still true.
4. **Update index** -- If existing topic: rewrite that index line to reflect new content. If new topic: append a new index line + file pointer.
5. **Check consolidation threshold + health** -- Read `.meta/writes-since-consolidation`, increment it. Also read `.meta/consolidation-health.json` if it exists. Warn the user if: (a) `last_run` is more than 7 days old AND the write counter exceeds threshold, (b) `status` is not "success", or (c) `retrieval_test_results.failed` > 0 ("retrieval quality degraded after last consolidation"). If counter > 10: trigger consolidation (spawn sub-agent). If not: done.

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
4. **Prune stale** -- Delete entries that are vague, derivable from code, or temporally expired. Specifically: entries tagged `[temporal:DATE]` where DATE is more than 60 days old are investigated ("is this still current?"). Entries with a specific deadline date that has passed are pruned or flagged. Entries where staleness is uncertain get a `[stale?]` tag -- the retrieval agent will note the uncertainty when returning these entries. Convert "I think the config is at..." to either the absolute path or delete it.
5. **Resolve contradictions** -- If two entries contradict each other, keep the newer one. If unclear, keep neither and flag it in `.meta/consolidation-log.md`.
6. **Derive cross-references** -- After reading all topic files, identify topics that reference shared entities or concepts. Add `see-also:` lines to the index for topics that are structurally related but stored separately. (Example: if dgx-cluster-setup and amplifier-architecture both reference NFS mounts, add a cross-reference.) Remove stale cross-references that no longer apply.
7. **Rewrite index** -- Regenerate MEMORY.md from the consolidated topic files. Each line is a fresh one-liner summary of that topic file's current content, with `see-also:` cross-references where applicable.
8. **Enforce cap** -- If the index still exceeds 200 lines after consolidation, force-prune the lowest-value topics (oldest, least specific, most derivable).
9. **Run retrieval smoke tests** -- Execute queries from `.meta/retrieval-test.yaml` against the updated index and topic files. For each test query, verify that the expected topics are returned. Log pass/fail results.
10. **Reset counter** -- Zero out `.meta/writes-since-consolidation`.
11. **Write health file** -- Write `.meta/consolidation-health.json` with: `last_run` timestamp, `status` (success/failure), `index_lines_before`/`index_lines_after`, `topics_merged`, `topics_pruned`, `contradictions_found`, and `retrieval_test_results` (passed/failed counts and failure details).
12. **Log** -- Append a summary to `.meta/consolidation-log.md` (what was merged, pruned, flagged, cross-referenced, and retrieval test results).

**Key principle:** Memory is continuously edited, not appended. The consolidation agent rewrites topic files in place. Old versions are gone. If you need history, that's what git is for.

### Git Safety Net for Consolidation

**Recommended practice** (not a hard requirement): If `~/amplifier-dev-memory/` is a git repo, the consolidation agent should commit a snapshot before starting and after completing. This provides:

- **Reversibility** -- If consolidation damages data, `git diff` shows exactly what changed and `git revert` restores it.
- **Audit trail** -- The git log becomes a history of how the memory store evolved over time, complementing `.meta/consolidation-log.md`.
- **Corruption recovery** -- If the consolidation agent crashes mid-rewrite, the working tree has partially rewritten files. With git, `git checkout .` restores the last known-good state.

The consolidation agent's workflow becomes: `git commit -am "pre-consolidation snapshot"` -> do all consolidation work -> `git commit -am "post-consolidation: merged N, pruned M"`. If the agent crashes between commits, the pre-consolidation commit is the recovery point.

### The Read Path (Retrieval)

**Stage 1: Search (always)**

- The retrieval agent calls the memory-search capability with the user's query
- Today, this capability reads MEMORY.md (~200 lines) and uses LLM judgment to identify relevant index lines. The retrieval agent does not know or care about the search implementation -- it receives a list of relevant topic file paths.
- This abstraction is the **vector search seam**: tomorrow, the search capability could query an embedding index instead of scanning the full index with LLM judgment. The retrieval agent's instructions say "use the search tool" not "read MEMORY.md directly." The implementation can be swapped without rewriting the retrieval agent.
- When to add vector search: when the retrieval smoke test (see below) shows retrieval misses at scale (~50+ topic files). Not before.

**Stage 2: Topic file fetch (on-demand)**

- For each relevant index line, the agent reads the referenced topic file
- Also reads topic files for any `see-also:` cross-references on the matched index lines
- Extracts the specific entries that answer the query
- Entries tagged `[stale?]` are returned with an explicit uncertainty note
- Returns a concise summary to the main session (~200 tokens)

**Stage 3: Transcript search (rare, only when needed)**

- If the query references something that might be in session history but not in memory, the retrieval agent can delegate to `session-analyst` to search transcripts
- This is Layer 2 -- never automatic, only when Layers 0 and 1 don't have the answer

**Key design choice:** Retrieval is skeptical, not blind. Memories are *hints*, not truth. The retrieval agent returns memories with the caveat that the main agent should verify against current reality before acting on them. Entries tagged `[temporal:DATE]` where the date is old get a staleness warning. Entries tagged `[stale?]` get an explicit uncertainty note. This is enforced in the retrieval agent's instructions: "Present memories as potentially stale hints. Flag any temporal memory older than 60 days or any `[stale?]` entry with a staleness warning."

### Retrieval Smoke Test

A regression test suite at `.meta/retrieval-test.yaml` validates that the retrieval path returns expected results for known queries. Format:

```yaml
tests:
  - query: "what database do I prefer?"
    expected_topics: ["workspace-preferences"]
    expected_keywords: ["PostgreSQL"]

  - query: "how is the DGX cluster set up?"
    expected_topics: ["dgx-cluster-setup"]

  - query: "who owns the provider modules?"
    expected_topics: ["team-org"]
    expected_keywords: ["Salil"]
```

**When it runs:** The consolidation agent executes these tests after every consolidation run (step 9). For each test query, it runs the retrieval path against the updated index and topic files and verifies the expected topics are returned. Pass/fail results are recorded in `.meta/consolidation-health.json`.

**What it catches:** Consolidation breaking retrieval (merged a topic that shouldn't have been merged), index rewriting losing important signal, and gradual retrieval quality degradation.

**Bootstrapping:** Start with 10-15 tests seeded from real queries after the V1-to-V2 migration. Add more as retrieval failures are discovered in real use. The test file is human-editable and committed alongside the memory store.

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
  see-also: amplifier-architecture, team-org
- Amplifier: single-namespace memory, filesystem-pure, index+topics → topics/amplifier-architecture.md
  see-also: dgx-cluster-setup
- Team: Sam (ramparte), Salil (robotdad), org structure → topics/team-org.md
  see-also: dgx-cluster-setup
```

Cross-references (`see-also:` lines) are maintained by the consolidation agent, not the write path. They are derived automatically during consolidation step 6 by identifying topics that reference shared entities or concepts. The retrieval agent follows cross-references when fetching topic files (read path stage 2).

### Topic File (topics/workspace-preferences.md)

```markdown
# Workspace Preferences

- Prefers PostgreSQL for all new projects [2026-01-15] [persistent]
- Uses Inactive/ folders to archive old work without deleting [2026-01-20] [persistent]
- Clean workspaces: no stale branches, no orphan files [2026-02-03] [persistent]
- Tab preference: 2-space indent for YAML, 4-space for Python [2026-03-10] [persistent]
- Kai is working on Project Orion [temporal:2026-04-07]
- The deploy deadline is April 15 [temporal:2026-04-15]
```

Entries tagged `[persistent]` are stable facts that don't expire on a calendar. Entries tagged `[temporal:DATE]` reference a current state that may change -- the consolidation agent investigates these when the date is more than 60 days old. Entries where staleness can't be determined get a `[stale?]` tag during consolidation; the retrieval agent flags these with an uncertainty note when returning them.

## Open Questions

1. **What happens when index + consolidation can't compress below 200 lines?** The 200-line cap is policy, not architecture. If the user generates 3 new topics per week, the index hits 200 in ~15 months. Consolidation can merge and prune, but can't invent compression that doesn't exist. The spec should define an escape hatch: raise the cap, split into sub-indexes (e.g., by domain), or activate the vector search seam. Recommended: rank these options and define the trigger condition for escalation.

2. **How does the write path handle concurrent sessions?** Two Amplifier sessions running simultaneously can both write to memory. File-level locking, last-write-wins, or advisory locking via `.meta/write-lock`? This isn't hypothetical -- users tab-switch between terminal windows. At minimum, the write path should check for a lock file before writing and warn if another session is mid-write.

3. **What's the consolidation agent's failure mode?** If the consolidation agent crashes mid-rewrite (e.g., LLM context limit, network failure, Amplifier session timeout), topic files may be partially rewritten. The git safety net (pre-consolidation commit) mitigates this, but the spec should describe the expected recovery behavior: detect the incomplete consolidation (e.g., missing `.meta/consolidation-health.json` update), warn the user, and offer to roll back via git.

4. **How do you bootstrap the retrieval smoke test?** The first 10 memories won't have meaningful test coverage. When does the test suite become useful? Recommended: the migration agent (step 7) seeds `.meta/retrieval-test.yaml` with 10-15 test queries derived from the migrated V1 data. Additional tests are added manually as retrieval failures are discovered in real use.
