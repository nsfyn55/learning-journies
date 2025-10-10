# Cgroups - Hands-On Experiments

**Goal:** Learn by doing. Observe cgroup behavior firsthand.

---

## Setup

**Prerequisites:**
- Linux system (VM or native)
- Root access (sudo)
- cgroups v1 mounted at /sys/fs/cgroup

**Check if cgroups are available:**
```bash
mount | grep cgroup
ls /sys/fs/cgroup/
```

---

## Experiment 1: Memory Limiting

**Goal:** Create a cgroup with memory limit and trigger OOM kill.

### Steps

1. **Create memory cgroup:**
```bash
sudo mkdir /sys/fs/cgroup/memory/test_memory
```

2. **Set 100MB limit:**
```bash
echo 100M > /sys/fs/cgroup/memory/test_memory/memory.limit_in_bytes
```

3. **Verify limit:**
```bash
cat /sys/fs/cgroup/memory/test_memory/memory.limit_in_bytes
```

4. **Create program that allocates memory:**
```c
// mem_hog.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main() {
    int mb = 0;
    while (1) {
        char *buf = malloc(1024 * 1024);  // 1MB
        if (!buf) {
            printf("malloc failed\n");
            break;
        }
        memset(buf, 0, 1024 * 1024);  // Actually use the memory
        mb++;
        printf("Allocated %d MB\n", mb);
        sleep(1);
    }
    return 0;
}
```

5. **Compile:**
```bash
gcc mem_hog.c -o mem_hog
```

6. **Run in cgroup:**
```bash
# Add current shell to cgroup
echo $$ | sudo tee /sys/fs/cgroup/memory/test_memory/cgroup.procs

# Run program (will be in same cgroup)
./mem_hog
```

### Expected Results
- Program allocates ~100MB
- Then OOM killer terminates it
- Check: `dmesg | tail` should show OOM kill message

### Observations to Record
- How many MB did it allocate before being killed?
- What does `memory.failcnt` show?
- What does dmesg say about which process was killed?

---

## Experiment 2: CPU Throttling

**Goal:** Limit CPU usage and observe throttling.

### Steps

1. **Create CPU cgroup:**
```bash
sudo mkdir /sys/fs/cgroup/cpu/test_cpu
```

2. **Set CPU limit to 20% of one core:**
```bash
# 20ms out of every 100ms = 20%
echo 20000 | sudo tee /sys/fs/cgroup/cpu/test_cpu/cpu.cfs_quota_us
echo 100000 | sudo tee /sys/fs/cgroup/cpu/test_cpu/cpu.cfs_period_us
```

3. **Create CPU-intensive program:**
```bash
# Infinite loop
yes > /dev/null &
PID=$!
```

4. **Monitor CPU (before cgroup):**
```bash
top -p $PID
# Should show ~100% CPU
```

5. **Add to cgroup:**
```bash
echo $PID | sudo tee /sys/fs/cgroup/cpu/test_cpu/cgroup.procs
```

6. **Monitor CPU (after cgroup):**
```bash
top -p $PID
# Should show ~20% CPU
```

7. **Check throttling stats:**
```bash
cat /sys/fs/cgroup/cpu/test_cpu/cpu.stat
```

### Expected Results
- CPU usage drops from 100% → 20%
- `nr_throttled` increases over time
- `throttled_time` accumulates

### Observations to Record
- How quickly does throttling take effect?
- What values do you see in cpu.stat?
- What happens if you increase quota to 50000?

---

## Experiment 3: Inspect Kubernetes Pod Cgroups

**Goal:** See how kubelet uses cgroups in practice.

### Prerequisites
- Running Kubernetes cluster with pods
- SSH access to worker node

### Steps

1. **SSH to worker node**

2. **Find kubepods cgroup:**
```bash
ls /sys/fs/cgroup/cpu/kubepods/
ls /sys/fs/cgroup/memory/kubepods/
```

3. **Explore hierarchy:**
```bash
# See QoS classes
ls /sys/fs/cgroup/cpu/kubepods/burstable/
ls /sys/fs/cgroup/cpu/kubepods/guaranteed/

# Pick a pod directory
cd /sys/fs/cgroup/cpu/kubepods/burstable/pod<UUID>/
```

4. **Examine pod limits:**
```bash
# CPU
cat cpu.cfs_quota_us
cat cpu.cfs_period_us

# Memory
cat /sys/fs/cgroup/memory/kubepods/burstable/pod<UUID>/memory.limit_in_bytes

# Process list
cat cgroup.procs
```

5. **Map to Kubernetes pod:**
```bash
# On host, find which pod
kubectl get pods -A -o wide | grep <node-name>

# Match PID to container
ps aux | grep <PID>
```

### Observations to Record
- What CPU quotas do you see?
- How do they match your pod's resource limits?
- What's the cgroup hierarchy structure?
- How many levels deep?

---

## Experiment 4: Manual Container-like Isolation

**Goal:** Create a simple "container" using cgroups (no Docker).

### Steps

1. **Create cgroups for multiple resources:**
```bash
sudo mkdir /sys/fs/cgroup/{cpu,memory}/my_container
```

2. **Set limits:**
```bash
# 50% CPU
echo 50000 | sudo tee /sys/fs/cgroup/cpu/my_container/cpu.cfs_quota_us

# 512MB memory
echo 512M | sudo tee /sys/fs/cgroup/memory/my_container/memory.limit_in_bytes
```

3. **Start shell in cgroup:**
```bash
# CPU
echo $$ | sudo tee /sys/fs/cgroup/cpu/my_container/cgroup.procs

# Memory
echo $$ | sudo tee /sys/fs/cgroup/memory/my_container/cgroup.procs

# Now in limited environment!
```

4. **Test limits:**
```bash
# Try memory hog
./mem_hog  # Should be killed at ~512MB

# Try CPU hog
yes > /dev/null &  # Should use max 50% CPU
```

### Observations to Record
- Does it feel like a container?
- What's missing compared to Docker? (namespaces, root filesystem, etc.)
- How would you automate this?

---

## Experiment 5: cgroups v2 Exploration

**Goal:** Compare v1 and v2 interfaces.

### Check if v2 is available:
```bash
mount | grep cgroup2
ls /sys/fs/cgroup/unified/
```

### If available, explore:
```bash
# Unified hierarchy
ls /sys/fs/cgroup/unified/

# Different file names
cat /sys/fs/cgroup/unified/cgroup.controllers
cat /sys/fs/cgroup/unified/cgroup.subtree_control
```

### Observations to Record
- What's different from v1?
- Simpler or more complex?
- What controllers are available?

---

## Cleanup

**After experiments:**
```bash
# Kill test processes
killall mem_hog yes

# Remove test cgroups
sudo rmdir /sys/fs/cgroup/memory/test_memory
sudo rmdir /sys/fs/cgroup/cpu/test_cpu
sudo rmdir /sys/fs/cgroup/*/my_container
```

---

## Results Log

**Document your findings here as you do experiments:**

### Experiment 1 Results:
```
Date:
Observations:
-
Surprises:
-
```

### Experiment 2 Results:
```
Date:
Observations:
-
Surprises:
-
```

(Continue for each experiment...)
