# Linux Kernel Learning Journey

**Goal:** Master Linux kernel internals with focus on container primitives and networking.

**Approach:** Systematic deep dives into key subsystems, connecting to real-world use (Kubernetes, containers).

---

## Structure

- **`learning-plan.md`** - Overall roadmap, phases, timeline
- **`learning-progress.md`** - Current status, completed topics, mental models
- **Sub-journey directories** - Deep dive into each subsystem:
  - `cgroups/` - Control Groups (resource limiting)
  - `namespaces/` - Process isolation primitives
  - `networking/` - Network stack internals
  - `scheduler/` - Process scheduler (CFS)
  - `memory/` - Memory management subsystem

---

## Current Focus

🔄 **Cgroups** (Week 1-3)
- Understanding kernel resource limiting
- How Kubernetes pod limits actually work
- CPU throttling and memory OOM internals

---

## Learning Methodology

Each subsystem follows the same pattern:

1. **Understand the problem** - Why does this exist?
2. **Learn concepts** - Key abstractions, data structures
3. **Read code** - Actual kernel implementation
4. **Experiment** - Hands-on with tools and syscalls
5. **Retention check** - Explain without notes

**Same approach that works for Kubernetes!**

---

## Sub-Journey Structure

Each subsystem directory contains:

```
cgroups/
├── learning-notes.md      # Concepts, theory, how it works
├── code-exploration.md    # Kernel source walkthrough
├── experiments.md         # Hands-on labs and results
└── retention-check.md     # Test your understanding
```

---

## Prerequisites

- C programming experience
- Comfort with Linux command line
- Understanding of OS concepts
- Git for exploring kernel source

---

## Getting Started

1. Read `learning-plan.md` for the roadmap
2. Check `learning-progress.md` for current status
3. Dive into current sub-journey (start with `cgroups/`)

---

**Started:** 2025-10-10
**Phase:** Container Primitives (Phase 1)
