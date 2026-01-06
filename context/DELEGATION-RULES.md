# CRITICAL: Memory Read Delegation Rules

## ⚠️ YOU MUST FOLLOW THESE RULES ⚠️

### Rule 1: NEVER Read Memory Files Directly

**FORBIDDEN:**
```
read_file(~/.amplifier/dev-memory/memory-store.yaml)  ❌ NEVER DO THIS!
```

**Why?** This loads the entire memory file (10k+ tokens) into YOUR context. With 1000 memories, this is 100k tokens - context overflow!

### Rule 2: ALWAYS Delegate Memory Reads

**REQUIRED:**
```
task tool with:
  agent: dev-memory:memory-retrieval
  instruction: "Search memory store for: [user's query]"  ✅ ALWAYS DO THIS!
```

**Why?** The sub-agent loads the file in its isolated context, searches, and returns only matches (~200 tokens).

---

## Pattern Recognition

When you see these user patterns:

- "what do you remember about X?"
- "what are all the things you remember about Y?"
- "recall memories related to Z"
- "search memory for A"
- "do you have any memories about B?"

**YOU MUST:**
1. Recognize this as a memory read operation
2. Extract the search query/topic
3. Invoke the memory-retrieval sub-agent via task tool
4. Wait for results
5. Present the formatted results to the user

**YOU MUST NOT:**
1. Use read_file on memory-store.yaml
2. Try to load and search the file yourself
3. Parse YAML in your own context

---

## Correct Invocation Example

**User:** "what do you remember about workspace organization?"

**Your thinking:**
```
User wants to search memories for "workspace organization"
This is a READ operation
I must delegate to memory-retrieval sub-agent
```

**Your action:**
```
[Invoke task tool]
agent: dev-memory:memory-retrieval
instruction: Search memory store for: "workspace organization"
```

**Your response to user:**
```
Let me search the memory store for you...

[Results from sub-agent appear]

Found 2 memories matching "workspace organization":
1. ...
2. ...
```

---

## Incorrect Invocation Example

**User:** "what do you remember about workspace organization?"

**Your thinking:**
```
User wants to search memories
I should read the memory file and search it  ❌ WRONG!
```

**Your action:**
```
read_file(~/.amplifier/dev-memory/memory-store.yaml)  ❌ FORBIDDEN!
[File loads into your context - 10k tokens wasted]
```

**Problem:** You just blew your token budget and defeated the entire architecture!

---

## Token Impact

| Your Approach | Token Cost | Result |
|---------------|------------|--------|
| Read file directly (WRONG) | 10,000 tokens | Context bloat ❌ |
| Delegate to sub-agent (RIGHT) | 200 tokens | Efficient ✅ |

**Savings: 98%!**

---

## Exception: Work Log

The work-log.yaml file is small (~500 tokens). You CAN read it directly:

```
read_file(~/.amplifier/dev-memory/work-log.yaml)  ✅ OK - it's small
```

**But memory-store.yaml is NEVER safe to read directly!**

---

## Self-Check

Before using read_file, ask yourself:

1. Am I reading memory-store.yaml? → **STOP! Delegate to sub-agent!**
2. Is this a memory search operation? → **STOP! Delegate to sub-agent!**
3. Is the user asking "what do you remember"? → **STOP! Delegate to sub-agent!**

Only read directly if:
- Reading work-log.yaml (small file)
- Checking if memory directory exists
- Reading README or other metadata

---

## Why This Matters

**With 100 memories:**
- Direct read: 10k tokens per query × 10 queries = 100k tokens (context overflow!)
- Delegated read: 200 tokens per query × 10 queries = 2k tokens (efficient!)

**With 1000 memories:**
- Direct read: 100k tokens per query (immediate context overflow!)
- Delegated read: 200 tokens per query (still efficient!)

The architecture ONLY works if you delegate reads. Otherwise, it scales terribly.

---

## Summary

🚫 **NEVER** use read_file on memory-store.yaml  
✅ **ALWAYS** delegate memory reads to memory-retrieval sub-agent  
📊 **Result:** 98% token savings on reads  

**This is not optional. This is the core architecture.**
