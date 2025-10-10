# Cgroups - Learning Notes

**Started:** 2025-10-10
**Status:** In Progress

---

## What Problem Do Cgroups Solve?

### The Problem (Pre-cgroups)
Before cgroups, there was no way to:
- Limit resources for a group of processes
- Track resource usage by application
- Isolate workloads from each other
- Prevent "noisy neighbor" problems

**Example:** Web server spawns 100 worker processes. How do you limit total memory to 1GB?
- Can't use ulimit (per-process, not per-group)
- Can't trust application to police itself
- Need kernel enforcement

### The Solution
**cgroups (control groups)** = Kernel feature to organize processes into hierarchical groups and apply resource limits/accounting.

**Key capabilities:**
1. **Resource limiting** - Don't let group use more than X
2. **Prioritization** - Group A gets more CPU than group B
3. **Accounting** - How much CPU/memory did group use?
4. **Control** - Freeze/resume groups of processes

---

## Connection to What You Already Know

**From your Kubernetes learning:**

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

**Under the hood, kubelet does:**
```bash
# Create cgroup for pod
mkdir /sys/fs/cgroup/kubepods/pod<uuid>/<container-id>

# Write CPU limit (200m = 20000 quota per 100000 period)
echo 20000 > /sys/fs/cgroup/kubepods/.../cpu.cfs_quota_us
echo 100000 > /sys/fs/cgroup/kubepods/.../cpu.cfs_period_us

# Write memory limit (256Mi = 268435456 bytes)
echo 268435456 > /sys/fs/cgroup/kubepods/.../memory.limit_in_bytes

# Start container process in this cgroup
```

**This is the bridge between your YAML and kernel enforcement!**

---

## Core Concepts

### 1. Hierarchy
Cgroups are organized in a tree (like directories):

```
root
├── system.slice         (system services)
├── user.slice           (user sessions)
└── kubepods             (Kubernetes pods)
    ├── burstable
    │   └── pod-abc123
    │       ├── container-1
    │       └── container-2
    └── guaranteed
        └── pod-def456
```

**Rules:**
- Every process belongs to exactly one cgroup (per controller)
- Child cgroups inherit limits from parents
- Can't use more resources than parent allows

### 2. Controllers (Subsystems)
Each controller manages a different resource type:

| Controller | What it controls | Key files |
|------------|------------------|-----------|
| **cpu** | CPU time (CFS bandwidth) | cpu.cfs_quota_us, cpu.cfs_period_us |
| **cpuset** | Which CPUs/NUMA nodes to use | cpuset.cpus, cpuset.mems |
| **memory** | RAM, swap, kernel memory | memory.limit_in_bytes, memory.usage_in_bytes |
| **blkio** | Block device I/O | blkio.throttle.read_bps_device |
| **pids** | Number of processes | pids.max, pids.current |
| **devices** | Device access control | devices.allow, devices.deny |
| **freezer** | Pause/resume processes | freezer.state |

### 3. cgroups v1 vs v2

**v1 (traditional):**
- Each controller has separate hierarchy
- Flexible but complex
- `/sys/fs/cgroup/cpu/`, `/sys/fs/cgroup/memory/`, etc.

**v2 (modern, unified hierarchy):**
- Single unified hierarchy for all controllers
- Cleaner design, better resource distribution
- `/sys/fs/cgroup/` (single root)
- Not all controllers ported yet

**Kubernetes uses v1 by default** (but v2 support is coming).

---

## How CPU Controller Works

### CFS Bandwidth Control

You already learned this! Let's go deeper:

**Key parameters:**
- `cpu.cfs_period_us` - Time window (default: 100000 = 100ms)
- `cpu.cfs_quota_us` - CPU time allowed in that period
- Ratio = quota/period = fraction of CPU

**Kernel enforcement:**
1. CFS scheduler tracks runtime per cgroup
2. When runtime exceeds quota in current period:
   - Mark cgroup as **throttled**
   - Don't schedule any tasks from this cgroup
3. At start of new period:
   - Reset runtime counter
   - Unthrottle cgroup

**Statistics:**
```bash
cat cpu.stat
nr_periods 1000       # How many periods elapsed
nr_throttled 250      # How many times throttled
throttled_time 50000  # Total time spent throttled (microseconds)
```

### Questions to Explore in Code:
- Where does throttling check happen in scheduler?
- How does kernel track per-cgroup runtime?
- What happens if multiple cgroups compete for CPU?

---

## How Memory Controller Works

### Memory Types Tracked

1. **Anonymous memory** (heap, stack)
2. **Page cache** (file-backed memory)
3. **Kernel memory** (sockets, dentry cache, etc.)

### Memory Limit Enforcement

**Key files:**
- `memory.limit_in_bytes` - Hard limit
- `memory.usage_in_bytes` - Current usage
- `memory.failcnt` - How many times limit was hit

**What happens when limit exceeded:**

```
Process in cgroup tries to allocate memory
    ↓
Current usage + allocation > limit?
    ↓ YES
Kernel tries to reclaim memory:
  1. Drop page cache
  2. Swap anonymous pages (if swap enabled)
  3. Try to free slab caches
    ↓
Reclaim successful?
    ↓ NO
OOM killer invoked for this cgroup
    ↓
Kill largest process in cgroup
    ↓
Process dies, container restarts
```

### Memory Pressure

Before OOM kill, kernel applies "memory pressure":
- `memory.pressure_level` - low, medium, critical
- Applications can register for pressure notifications
- Allows graceful degradation

### Questions to Explore in Code:
- Where is OOM killer invoked?
- How does kernel decide which process to kill?
- What's the relationship between memory cgroup and global OOM?

---

## VFS Interface: /sys/fs/cgroup

**Key insight:** cgroups are configured via a **virtual filesystem**.

**Why filesystem interface?**
- ✅ Familiar operations (mkdir, echo, cat)
- ✅ Standard permissions model
- ✅ Easy scripting
- ✅ Works across languages

**Creating a cgroup:**
```bash
# Create = mkdir
sudo mkdir /sys/fs/cgroup/memory/my_app

# Set limit = write to file
echo 1G > /sys/fs/cgroup/memory/my_app/memory.limit_in_bytes

# Add process = write PID
echo $$ > /sys/fs/cgroup/memory/my_app/cgroup.procs

# Now this shell and all children are limited to 1GB
```

**Kernel implementation:** VFS hooks call into cgroup code.

---

## How kubelet Uses Cgroups

**Kubernetes QoS classes map to cgroup hierarchies:**

```
/sys/fs/cgroup/cpu/kubepods/
├── besteffort/          (no requests/limits)
│   └── pod-abc/
├── burstable/           (requests < limits)
│   └── pod-def/
└── guaranteed/          (requests = limits)
    └── pod-ghi/
```

**For each container, kubelet:**
1. Creates cgroup under appropriate QoS class
2. Writes CPU quota/period based on limits
3. Writes memory limit
4. Adds container PID to cgroup.procs
5. Kernel automatically enforces!

---

## Key Data Structures (Kernel Code)

**To explore:**
- `struct cgroup` - Represents a cgroup
- `struct cgroup_subsys` - Defines a controller
- `struct cgroup_subsys_state` - Per-cgroup state for a controller

**Key files to read:**
- `kernel/cgroup/cgroup.c` - Core cgroup logic
- `kernel/sched/core.c` - CPU controller integration
- `mm/memcontrol.c` - Memory controller

---

## Experiments to Try

1. **Create your own cgroup and limit memory**
2. **Trigger OOM kill and observe**
3. **Monitor throttling with cpu.stat**
4. **Inspect kubelet's cgroup hierarchy**
5. **Use systemd-cgtop to view live cgroup stats**

---

## Next Learning Steps

1. **Hands-on:** Create cgroups manually, test limits
2. **Code exploration:** Read kernel/cgroup/cgroup.c
3. **Deep dive:** CPU controller throttling logic
4. **Deep dive:** Memory controller OOM path
5. **Connection:** Trace kubelet creating pod cgroups

---

## Retention Check Questions

(To be answered without looking at notes)

1. What problem do cgroups solve?
2. What's the difference between cgroups v1 and v2?
3. How does CPU throttling work (quota/period)?
4. What happens when memory limit is exceeded?
5. Why does kubelet use different cgroup hierarchies for QoS classes?
6. How do you add a process to a cgroup?
