# Linux Kernel Learning Plan

## Overall Goal
Understand Linux kernel internals with focus on primitives used by Kubernetes and container runtimes.

## Learning Philosophy
- **Depth over breadth**: Master one subsystem at a time
- **Connect to existing knowledge**: Build on Kubernetes understanding
- **Read real code**: Linux kernel source is the ground truth
- **Hands-on exploration**: Use tracing tools (ftrace, perf, eBPF)

## Learning Path

### Phase 1: Container Primitives (Foundation)
**Goal:** Understand the kernel features that enable containers

1. 🔄 **Cgroups** (IN PROGRESS - Week 1-3)
   - What containers use for resource limits
   - Connects to: Kubernetes resource requests/limits

2. ⏳ **Namespaces** (Week 4-6)
   - What containers use for isolation
   - Connects to: Pod networking, process isolation

### Phase 2: Networking Stack
**Goal:** Understand how packets flow through the kernel

3. ⏳ **Network Namespaces Deep Dive** (Week 7-8)
   - How CNI creates isolated networks
   - veth pair implementation

4. ⏳ **Socket Layer & Packet Flow** (Week 9-11)
   - System calls to packet transmission
   - Connection tracking

5. ⏳ **Netfilter/iptables** (Week 12-14)
   - How kube-proxy rules work
   - Hooks, tables, chains internals

### Phase 3: Process & Memory Management
**Goal:** Understand scheduling and memory

6. ⏳ **Process Scheduler (CFS)** (Week 15-17)
   - How CPU is allocated
   - Cgroup CPU controller integration

7. ⏳ **Memory Management** (Week 18-22)
   - Virtual memory, page tables
   - Memory cgroups internals
   - OOM killer

### Phase 4: Modern Observability
8. ⏳ **eBPF** (Week 23-26)
   - Modern tracing and observability
   - BPF programs, maps, hooks

---

## Sub-Journeys

Each subsystem gets its own directory with:
- `learning-notes.md` - Deep dive notes and concepts
- `code-exploration.md` - Source code walkthrough
- `experiments.md` - Hands-on testing and observations
- `retention-check.md` - Test your understanding

**Current sub-journeys:**
- `cgroups/` - Control Groups (resource limiting)
- `namespaces/` - Isolation primitives
- `networking/` - Network stack
- `scheduler/` - Process scheduler
- `memory/` - Memory management

---

## Prerequisites

### Required Skills
- ✅ C programming (pointers, structs, memory management)
- ✅ Understanding of operating system concepts
- ✅ Comfortable with command line
- ✅ Git for exploring kernel source

### Tools Setup
- [ ] Linux kernel source: `git clone https://github.com/torvalds/linux.git`
- [ ] Code navigation: ctags/cscope or clangd
- [ ] Kernel source browser: https://elixir.bootlin.com/linux/latest/source
- [ ] Tracing tools: ftrace, perf (already installed on most Linux systems)
- [ ] Optional: Build custom kernel for experimentation

### Reference Materials
- Linux kernel documentation: `Documentation/` in kernel source
- Man pages: `man 2` (syscalls), `man 7` (overviews)
- Books (optional):
  - "Linux Kernel Development" by Robert Love (overview)
  - "Understanding the Linux Kernel" by Bovet & Cesati (deep)
  - LWN.net articles (excellent current content)

---

## Learning Strategy

**Same approach that worked for Kubernetes:**

1. **Understand the problem** - Why does this subsystem exist?
2. **Learn the concepts** - Key data structures, algorithms
3. **Read the code** - Explore actual implementation
4. **Experiment** - Use tools to observe behavior
5. **Retention check** - Explain it back without notes

**Time investment:**
- 3-5 hours per week
- Each subsystem: 2-4 weeks
- Total: 6-12 months for Phase 1-3

---

## Success Metrics

**For each subsystem, you should be able to:**
- Explain key concepts to a colleague
- Trace execution through relevant code
- Answer "how does this actually work" questions
- Connect it to real-world systems (containers, Kubernetes)

---

## Current Status

**Started:** 2025-10-10
**Current focus:** Cgroups (Phase 1)
**Completed subsystems:** None yet
**Next up after cgroups:** Namespaces
