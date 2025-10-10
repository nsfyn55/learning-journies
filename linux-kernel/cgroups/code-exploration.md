# Cgroups - Code Exploration

**Goal:** Understand cgroups implementation by reading kernel source.

**Approach:** Start with key data structures, then follow execution paths.

---

## Source Navigation

**Primary files:**
- `kernel/cgroup/cgroup.c` - Core cgroup implementation
- `kernel/cgroup/cgroup-v1.c` - v1 specific code
- `include/linux/cgroup.h` - Data structures and API
- `kernel/sched/core.c` - CPU controller integration
- `mm/memcontrol.c` - Memory controller

**Browsing online:**
- https://elixir.bootlin.com/linux/latest/source
- Can search, cross-reference, jump to definitions

---

## Key Data Structures

### struct cgroup

**Location:** `include/linux/cgroup-defs.h`

```c
struct cgroup {
    /* Cgroup hierarchy link */
    struct cgroup_subsys_state self;

    /* Parent and children */
    struct cgroup *parent;
    struct kernfs_node *kn;        // sysfs directory node

    /* List of children */
    struct list_head children;
    struct list_head sibling;

    /* Processes in this cgroup */
    struct list_head cset_links;

    /* Per-subsystem state */
    struct cgroup_subsys_state *subsys[CGROUP_SUBSYS_COUNT];

    /* Root this cgroup belongs to */
    struct cgroup_root *root;

    /* ... more fields ... */
};
```

**Key insights:**
- Cgroups form a tree (parent, children, sibling)
- Each cgroup has state for each subsystem (subsys array)
- Connected to sysfs via kernfs_node

### struct cgroup_subsys

**Location:** `include/linux/cgroup-defs.h`

```c
struct cgroup_subsys {
    /* Controller operations */
    int (*css_online)(struct cgroup_subsys_state *css);
    void (*css_offline)(struct cgroup_subsys_state *css);
    int (*can_attach)(struct cgroup_taskset *tset);
    void (*attach)(struct cgroup_taskset *tset);

    /* Name and ID */
    const char *name;
    int id;

    /* ... more ... */
};
```

**Key insight:** Each controller implements these callbacks.

### struct task_struct connection

**Location:** `include/linux/sched.h`

Every process has:
```c
struct task_struct {
    /* ... */
    struct cgroup_subsys_state *cgroups;  // Cgroup membership
    /* ... */
};
```

**Key insight:** Process → cgroup link is in task_struct.

---

## Code Path 1: Adding Process to Cgroup

**User action:**
```bash
echo $PID > /sys/fs/cgroup/cpu/my_cgroup/cgroup.procs
```

**Kernel path:**

1. **VFS write operation** → `cgroup_procs_write()`
   - Location: `kernel/cgroup/cgroup.c`
   - Parses PID from userspace

2. **`cgroup_attach_task()`**
   - Validates PID
   - Calls `cgroup_migrate()`

3. **`cgroup_migrate()`**
   - For each subsystem:
     - Call `can_attach()` callback
     - If all allow, call `attach()` callback
   - Update task's cgroup pointers

4. **Controller-specific attach:**
   - CPU: Update scheduler state
   - Memory: Move memory accounting

**Code to read:**
```c
// kernel/cgroup/cgroup.c
static ssize_t cgroup_procs_write(struct kernfs_open_file *of,
                                   char *buf, size_t nbytes, loff_t off)
{
    /* Parse PID, find task, migrate to cgroup */
}
```

**Questions while reading:**
- What locks are held during migration?
- What happens if can_attach() fails?
- How are child processes handled?

---

## Code Path 2: CPU Throttling Check

**When:** Scheduler picks next task to run

**Path:**

1. **`pick_next_task_fair()`** in `kernel/sched/fair.c`
   - CFS scheduler's main function

2. **Check cgroup bandwidth:**
```c
if (cfs_rq->runtime_remaining <= 0 && cfs_rq->throttled)
    return NULL;  // Skip this cgroup
```

3. **Account runtime:**
```c
static void account_cfs_rq_runtime(struct cfs_rq *cfs_rq, u64 delta_exec)
{
    cfs_rq->runtime_remaining -= delta_exec;
    if (cfs_rq->runtime_remaining <= 0)
        throttle_cfs_rq(cfs_rq);  // Throttle!
}
```

4. **Throttling:**
```c
static void throttle_cfs_rq(struct cfs_rq *cfs_rq)
{
    cfs_rq->throttled = 1;
    dequeue_task_fair(...);  // Remove from runqueue
    /* Schedule timer to unthrottle at next period */
}
```

**Code to read:**
- `kernel/sched/fair.c` - Search for "throttle"
- `struct cfs_bandwidth` - Per-cgroup bandwidth state

**Questions:**
- How is runtime_remaining replenished?
- What timer is used for period boundary?
- Can a cgroup be throttled in middle of quantum?

---

## Code Path 3: Memory Limit Enforcement

**When:** Process allocates memory (malloc, page fault)

**Path:**

1. **Page fault** → `do_anonymous_page()`
   - Location: `mm/memory.c`

2. **Allocate page** → `mem_cgroup_charge()`
   - Location: `mm/memcontrol.c`

3. **Check limit:**
```c
static int try_charge(struct mem_cgroup *memcg, gfp_t gfp_mask,
                      unsigned int nr_pages)
{
    unsigned long new_usage = page_counter_read(&memcg->memory) + nr_pages;

    if (new_usage > memcg->memory.max) {
        /* Try to reclaim memory */
        if (!mem_cgroup_reclaim(...)) {
            /* Reclaim failed, trigger OOM */
            mem_cgroup_oom(...);
            return -ENOMEM;
        }
    }

    /* Update usage */
    page_counter_charge(&memcg->memory, nr_pages);
}
```

4. **OOM:**
```c
static void mem_cgroup_oom(struct mem_cgroup *memcg)
{
    /* Find victim in this cgroup */
    select_bad_process(...);
    oom_kill_process(...);
}
```

**Code to read:**
- `mm/memcontrol.c` - Search for "try_charge"
- `mm/oom_kill.c` - OOM killer logic

**Questions:**
- What memory can be reclaimed?
- How is OOM victim selected?
- Can OOM happen even if system has free memory?

---

## Code Path 4: Creating Cgroup (mkdir)

**User action:**
```bash
mkdir /sys/fs/cgroup/cpu/my_cgroup
```

**Path:**

1. **VFS mkdir** → `cgroup_mkdir()`
   - Location: `kernel/cgroup/cgroup.c`

2. **Allocate cgroup struct:**
```c
struct cgroup *cgrp = kzalloc(sizeof(*cgrp), GFP_KERNEL);
```

3. **Initialize cgroup:**
```c
cgrp->parent = parent;
cgrp->root = root;
INIT_LIST_HEAD(&cgrp->children);
```

4. **Initialize subsystem state:**
```c
for_each_subsys(ss, ssid) {
    struct cgroup_subsys_state *css = ss->css_alloc(cgrp);
    rcu_assign_pointer(cgrp->subsys[ssid], css);
}
```

5. **Create sysfs files:**
```c
kernfs_create_dir(...);  // Directory
cgroup_populate_dir(...); // Files (cpu.cfs_quota_us, etc.)
```

**Code to read:**
```c
// kernel/cgroup/cgroup.c
static int cgroup_mkdir(struct kernfs_node *parent_kn, const char *name, umode_t mode)
{
    /* Allocate and initialize cgroup */
}
```

---

## VFS Integration

**Key insight:** cgroups uses kernfs (kernel filesystem library).

**File operations mapped to cgroup code:**

```c
// kernel/cgroup/cgroup.c
static struct kernfs_ops cgroup_kf_ops = {
    .write = cgroup_file_write,
    .read  = cgroup_file_read,
    /* ... */
};
```

**When you do:**
```bash
cat /sys/fs/cgroup/cpu/cpu.stat
```

**Kernel does:**
1. VFS read() → kernfs_fop_read()
2. kernfs calls → cgroup_file_read()
3. cgroup_file_read() → cpu controller's show() callback
4. Callback returns stats string

---

## Exercises

### Exercise 1: Trace a write
1. Open `kernel/cgroup/cgroup.c` in browser
2. Find `cgroup_procs_write()`
3. Follow the code path
4. Draw a call graph

### Exercise 2: Find data structure connections
1. Look at `struct cgroup`
2. Find `subsys[]` array
3. Look at `struct cgroup_subsys_state`
4. How do they connect?

### Exercise 3: Find throttling logic
1. Search for "throttle" in `kernel/sched/fair.c`
2. Find `throttle_cfs_rq()`
3. What does it do?
4. Who calls it?

---

## Questions to Answer Through Code Reading

1. **How many controllers are there in total?**
   - Hint: Search for `CGROUP_SUBSYS` macro usage

2. **What happens when you remove a cgroup (rmdir)?**
   - Find `cgroup_rmdir()` or `cgroup_destroy_locked()`

3. **How does kernel track which cgroup a process is in?**
   - Look at `task_struct` and `cgroup_subsys_state`

4. **What's the difference between cgroup.procs and tasks files?**
   - One is per-process, one is per-thread!

---

## Next Steps

1. Pick one code path above
2. Read it in detail with kernel source open
3. Take notes on what you learn
4. Try to explain it without looking

**Remember:** You don't need to understand every detail. Focus on:
- Overall flow
- Key data structures
- Where enforcement happens
- How user space interacts with kernel
