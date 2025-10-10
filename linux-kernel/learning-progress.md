# Linux Kernel Learning Progress

## Overall Goal
Master Linux kernel internals, focusing on container primitives and networking stack used by Kubernetes.

## Learning Journey Status
**Started:** 2025-10-10
**Current Phase:** Phase 1 - Container Primitives
**Current Focus:** Cgroups

---

## Phase 1: Container Primitives ⏳

### Cgroups (Control Groups) 🔄
**Status:** IN PROGRESS
**Started:** 2025-10-10
**Target completion:** Week 3

**Topics to cover:**
- [ ] What are cgroups and why do they exist?
- [ ] cgroups v1 vs v2 architecture
- [ ] Hierarchies and subsystems
- [ ] CPU controller (CFS bandwidth control)
- [ ] Memory controller
- [ ] How kubelet uses cgroups
- [ ] VFS interface (/sys/fs/cgroup)
- [ ] Key data structures in kernel code

**Connection to Kubernetes:**
- Resource requests/limits
- QoS classes (Guaranteed, Burstable, BestEffort)
- How kubelet enforces pod resource constraints

---

### Namespaces ⏳
**Status:** NOT STARTED
**Target:** Week 4-6

**Topics to cover:**
- Network namespaces
- PID namespaces
- Mount namespaces
- IPC, UTS, User namespaces
- How namespaces are created/destroyed
- Entering namespaces (setns syscall)

---

## Phase 2: Networking Stack ⏳

### Network Namespaces Deep Dive
**Status:** NOT STARTED

### Socket Layer & Packet Flow
**Status:** NOT STARTED

### Netfilter/iptables
**Status:** NOT STARTED

---

## Phase 3: Process & Memory Management ⏳

### Process Scheduler (CFS)
**Status:** NOT STARTED

### Memory Management
**Status:** NOT STARTED

---

## Key Mental Models Established

*None yet - will build these as we learn!*

---

## Tools Mastered

*None yet - will learn ftrace, perf, etc. as needed*

---

## Code Explored

*Will track significant code paths explored*

---

## Questions to Revisit Later

- How does OOM killer decide which process to kill?
- What's the performance difference between cgroups v1 and v2?
- How does eBPF integrate with cgroups?
- Why are some cgroup controllers not available in v2?

---

## Next Steps

**Immediate:** Start cgroups learning
1. Understand what problem cgroups solve
2. Learn cgroups v1 architecture
3. Explore /sys/fs/cgroup filesystem
4. Trace how kubelet writes cgroup limits
5. Read kernel code: kernel/cgroup/cgroup.c

**After cgroups:** Namespaces deep dive
