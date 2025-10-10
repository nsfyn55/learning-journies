# EKS/Kubernetes Learning Progress

## Overall Goal
Build observability platform for apps running in EKS using Prometheus and AWS hosted Grafana.

## Learning Path
1. ✅ Learn EKS and Kubernetes fundamentals
2. ✅ Write Terraform to build EKS cluster
3. 🔄 Build web app/db stack in cluster (IN PROGRESS - learning phase)
4. ⏳ Build observability prototype (Prometheus + Grafana)
5. ⏳ Build out SLOs

---

## Topics Completed ✅

### Kubernetes Architecture
- **Control Plane Components:**
  - API Server - central hub, all communication flows through here
  - etcd - distributed key-value store, cluster state database
  - Scheduler - assigns pods to nodes based on resources/constraints
  - Controller Manager - runs ~20+ controllers (Endpoints, Deployment, ReplicaSet, etc.)
  - Cloud Controller Manager - AWS-specific integration (LoadBalancers, node lifecycle)

- **Data Plane Components (Worker Nodes):**
  - kubelet - node agent, orchestrates everything on the node
  - Container Runtime - actually runs containers (containerd)
  - kube-proxy - manages Service networking via iptables/IPVS rules
  - CNI Plugin - called by kubelet to set up pod networking (AWS VPC CNI)
  - Pods - the actual workloads (can contain multiple containers)

### Pod Networking (Deep Dive)
- **Kubernetes Networking Model:**
  - Every pod gets its own IP address
  - All pods can communicate without NAT
  - Flat network model

- **AWS VPC CNI Specifics:**
  - Pods get real VPC IP addresses (not overlay network)
  - ENIs have primary + secondary IPs
  - Secondary IPs form the "warm pool"
  - CNI assigns warm pool IPs to pods as they start

- **Network Namespaces:**
  - Linux kernel isolation feature
  - Each pod gets its own network namespace
  - All containers in a pod share the same network namespace
  - Containers in same pod can talk via localhost

- **veth Pairs:**
  - Virtual ethernet cable with two ends
  - One end in pod namespace (eth0)
  - One end in host namespace (veth-abc)
  - Traffic flows between namespaces through the pair

- **Pod Startup Flow:**
  1. Kubelet calls CNI to set up networking
  2. CNI creates network namespace for pod
  3. CNI creates veth pair
  4. CNI assigns secondary IP from warm pool to pod's eth0
  5. CNI sets up routing rules on host
  6. Packets destined for pod IP route through veth pair

### Service Networking (Deep Dive)
- **The Problem Services Solve:**
  - Pods are ephemeral (die and get recreated with new IPs)
  - Need stable endpoint for client connections
  - Need load balancing across pod replicas

- **How Services Work:**
  - Service gets stable ClusterIP (virtual IP)
  - Endpoint Controller watches Services and Pods
  - Creates/updates Endpoints object with matching pod IPs
  - kube-proxy watches Services and Endpoints
  - Programs iptables rules on every node

- **iptables Mode (Default):**
  - Service IP is virtual (not assigned to any interface)
  - iptables intercepts traffic destined for Service IP
  - Rewrites destination to pod IP (DNAT)
  - Load balances via probability rules
  - Rewrite happens locally on source node before routing

- **Key Insights:**
  - Every node has iptables rules for ALL Services and Endpoints
  - Service IP never travels on the network (rewritten immediately)
  - Translation is local, fast, and distributed
  - Scales poorly with iptables (IPVS mode better at scale)

- **Pod Lifecycle:**
  - Container crash → kubelet restarts container (same pod, same IP)
  - Pod deletion → scheduler creates new pod (new pod, new IP)
  - Deployment updates → rolling replacement (all new pods, new IPs)
  - Node failure → pods rescheduled elsewhere (new pods, new IPs)

### DNS / Service Discovery (Deep Dive)
- **The Problem DNS Solves:**
  - Services provide stable ClusterIPs, but apps need name-based discovery
  - Hard-coding IPs is brittle
  - Need automatic resolution of service names

- **CoreDNS Architecture:**
  - Runs as Pods (Deployment with 2+ replicas for HA)
  - Exposed via Service with ClusterIP (e.g., 10.100.0.10)
  - Lives in kube-system Kubernetes namespace
  - Watches API server for Service changes

- **How Pod DNS Works:**
  - kubelet writes /etc/resolv.conf in every pod
  - nameserver points to CoreDNS ClusterIP
  - search domains enable short names in same namespace

- **DNS Naming Convention:**
  - Full format: `<service>.<namespace>.svc.cluster.local`
  - Short name works in same namespace: `postgres`
  - Cross-namespace requires namespace: `grafana.monitoring`

- **DNS vs API-Based Discovery:**
  - DNS: name → Service ClusterIP (for app-to-app communication)
  - API-based: dynamic discovery of all resources (for Prometheus, controllers)

- **Prometheus Service Discovery:**
  - Uses `kubernetes_sd_configs` in Prometheus config (stored in ConfigMap)
  - Watches API server for pods, services, endpoints, nodes
  - Can't use DNS (DNS only returns ClusterIPs, not individual pod IPs)
  - Relabeling rules filter targets (e.g., annotation-based)

- **Watch Mechanism (HTTP Protocol Level):**
  - NOT polling - event-driven push using HTTP streaming
  - Uses HTTP/1.1 chunked transfer encoding (RFC 7230)
  - Single long-lived connection per watcher
  - API server pushes events immediately as they occur
  - All components use same pattern: CoreDNS, kube-proxy, Prometheus, controllers

- **Two Types of Namespaces:**
  - Linux kernel namespaces: network isolation for pods (network, PID, mount, etc.)
  - Kubernetes namespaces: logical grouping of API resources (default, kube-system, etc.)
  - Every pod has both: lives in K8s namespace AND has own Linux network namespace

- **Key Insight:**
  - DNS returns Service ClusterIPs (never Pod IPs)
  - kube-proxy then translates ClusterIP → Pod IP via iptables
  - Two-layer translation: DNS (name→ClusterIP) + iptables (ClusterIP→Pod IP)

### Workload Resources (Deep Dive)
- **The Problem Workload Resources Solve:**
  - Need automated pod lifecycle management
  - Need multiple replicas for high availability
  - Need rolling updates and rollbacks
  - Databases need stable identity and persistent storage

- **Deployments (Stateless Apps):**
  - Three-tier hierarchy: Deployment → ReplicaSet → Pods
  - Deployment Controller watches Deployments, creates/updates ReplicaSets
  - ReplicaSet Controller watches ReplicaSets, creates/deletes Pods to match replica count
  - Rolling updates create new ReplicaSet, gradually scale up/down
  - Old ReplicaSet kept (scaled to 0) for easy rollback
  - Strategy parameters: maxUnavailable, maxSurge

- **StatefulSets (Stateful Apps):**
  - Stable pod names: postgres-0, postgres-1 (not random hashes)
  - Stable DNS names: postgres-0.postgres.default.svc.cluster.local
  - **Pod IPs still change** when rescheduled to different nodes (only DNS is stable!)
  - Requires Headless Service (ClusterIP: None) for direct pod addressing
  - DNS returns all pod IPs directly (no load balancing)
  - Ordered operations: scale up 0→1→2, scale down 2→1→0
  - PersistentVolumeClaims follow pods across rescheduling
  - Used for databases, message queues, any stateful workload

- **Resource Requests and Limits:**
  - Requests: Minimum guaranteed resources (used by Scheduler for placement)
  - Limits: Maximum allowed resources (enforced by kubelet via cgroups)
  - CPU units: millicores (1000m = 1 core, 100m = 10% of core)
  - Memory units: Mi/Gi (binary) or M/G (decimal)

- **How Scheduler Uses Requests:**
  - Filters nodes with insufficient available resources
  - Scores nodes based on remaining capacity
  - Places pod on best-fit node

- **How kubelet Enforces Limits (Linux cgroups):**
  - CPU: CFS bandwidth control (period/quota), throttling when exceeded
  - Memory: Hard limit, OOMKill when exceeded (process killed, container restarted)
  - cgroups v1: Separate hierarchies (cpu, memory)
  - cgroups v2: Unified hierarchy
  - kubelet writes to /sys/fs/cgroup files to configure limits

- **Integration with Ingress/ALB:**
  - Endpoints Controller watches Pods (regardless of Deployment/StatefulSet)
  - Updates Endpoints object when pod IPs change
  - AWS Load Balancer Controller watches Endpoints
  - Automatically updates ALB Target Groups when pods move nodes
  - Same watch-and-reconcile pattern!

- **Key Insights:**
  - Deployments use ReplicaSets for versioning (each update = new ReplicaSet)
  - StatefulSets provide stable **names**, not stable **IPs**
  - DNS enables StatefulSet pods to be addressed individually
  - Resource limits are enforced by Linux kernel, not Kubernetes
  - CPU is compressible (throttle), memory is incompressible (kill)

---

## Topics Not Yet Covered ⏳

### Critical for Observability Platform
1. **Service Types Deep Dive** (MEDIUM PRIORITY)
   - NodePort services
   - LoadBalancer services
   - ExternalName services

4. **Network Policies** (MEDIUM PRIORITY)
   - Pod-to-pod traffic restrictions
   - Security isolation
   - Egress controls

### Nice to Have
- Advanced CNI topics
- IPVS mode details
- eBPF-based networking
- Service Mesh (Istio, Linkerd)
- DaemonSets (per-node agents)
- Jobs and CronJobs (batch workloads)
- Pod Disruption Budgets
- HorizontalPodAutoscaler (HPA)
- RBAC and security

---

## Next Steps

**Immediate:** Configuration and Secrets (15 min)
- ConfigMaps for application configuration
- Secrets for sensitive data
- Environment variables and volume mounts

**After:** Storage / Persistent Volumes (20 min)
- Persistent Volumes (PV) and Persistent Volume Claims (PVC)
- Storage Classes
- StatefulSet storage requirements for databases

---

## Key Mental Models Established

1. **Control Plane = Brain, Data Plane = Muscle**
   - Control plane makes decisions
   - Data plane executes them

2. **Pod Networking = Physical Layer**
   - Real IPs, real routing
   - CNI sets up the plumbing
   - veth pairs connect namespaces

3. **Service Networking = Abstraction Layer**
   - Virtual IPs that don't really exist
   - iptables intercepts and rewrites
   - Provides stability and load balancing

4. **Everything is a Watch Loop**
   - Controllers watch API server
   - Compare desired vs actual state
   - Reconcile differences
   - Report status

5. **Distributed by Design**
   - Each node independently handles its pods
   - All nodes have complete Service/Endpoint info
   - No central bottleneck for networking
   - Fast and resilient

6. **DNS Sits on Top**
   - CoreDNS gets first crack at all DNS queries
   - Authoritative for *.cluster.local domains
   - Forwarder for external domains
   - Two-layer translation: DNS (name→ClusterIP) + iptables (ClusterIP→Pod IP)

7. **Watch Pattern is Universal**
   - All components use same mechanism: HTTP streaming with chunked transfer encoding
   - CoreDNS watches Services
   - kube-proxy watches Services + Endpoints
   - Prometheus watches Pods/Services/Endpoints/Nodes
   - Controllers watch their relevant resources
   - NOT polling - event-driven push over long-lived connections

8. **Workload Resources = Pod Lifecycle Management**
   - Deployments: For stateless apps, rolling updates, random pod names
   - StatefulSets: For stateful apps, stable DNS names (NOT stable IPs!), ordered ops
   - Deployment → ReplicaSet → Pods (enables versioning and rollback)
   - Resource limits enforced by Linux kernel (cgroups), not Kubernetes
   - CPU = compressible (throttle), Memory = incompressible (kill)

---

## Questions to Revisit Later

- When to use IPVS vs iptables mode?
- Best practices for Network Policies?
- How to size node pools for observability workloads?
- What metrics should we collect from the cluster itself?
- How does Prometheus Operator work vs manual Prometheus deployment?
- ConfigMap size limits and best practices for large configurations?
