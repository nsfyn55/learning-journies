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

### ConfigMaps and Secrets (Deep Dive)
- **The Problem ConfigMaps and Secrets Solve:**
  - Need to externalize configuration from application code and container images
  - Same image should run in different environments (dev/staging/prod)
  - Secrets provide basic mechanism for sensitive data storage

- **Two Consumption Methods:**
  - Environment variables: injected at pod start (requires restart to change)
  - Volume mounts: mounted as files in container (can auto-update)

- **Storage in etcd:**
  - ConfigMaps: stored as plaintext strings
  - Secrets: stored as base64 encoded strings
  - Base64 is NOT encryption - trivially decodable by anyone with RBAC permissions

- **Why Base64 Encoding Exists:**
  - PRIMARY reason: Binary data support (certificates, keys) - JSON/YAML can't handle binary
  - SECONDARY reason: Character safety - special chars break JSON/YAML parsing
  - Also: etcd string storage, API transport compatibility
  - NOT for security!

- **Secret Types:**
  - `Opaque`: Default catch-all type, no validation, any keys allowed
  - `kubernetes.io/tls`: Requires `tls.crt` and `tls.key` keys
  - `kubernetes.io/basic-auth`: Requires `username` and `password` keys
  - `kubernetes.io/dockerconfigjson`: Requires `.dockerconfigjson` key
  - ALL types use base64 encoding - type only affects validation

- **AWS Secrets Manager Integration:**
  - **External Secrets Operator**: Custom controller that syncs AWS → K8s Secrets in etcd
  - **CSI Driver**: Mounts secrets directly from AWS to pod, never stored in etcd (most secure)
  - IRSA (IAM Roles for Service Accounts) for AWS authentication

- **Security Considerations:**
  - Base64 encoding provides NO security
  - Default: Secrets stored unencrypted in etcd
  - Required: RBAC to restrict Secret access
  - Recommended: Enable etcd encryption at rest
  - Best practice: Use external secret managers (AWS Secrets Manager + CSI Driver)

- **Lifecycle and Updates:**
  - Volume mounts: Kubernetes auto-updates files when Secret/ConfigMap changes
  - Environment variables: Require pod restart to reflect changes
  - Practical reality: Most apps cache config at startup, need restart anyway
  - Rolling restart: `kubectl rollout restart deployment` (graceful, zero downtime)

- **Operators Pattern (Bonus):**
  - Operator = Custom controller + domain knowledge
  - Same watch-and-reconcile pattern as built-in controllers
  - Uses CRDs (Custom Resource Definitions) to extend Kubernetes API
  - Examples: External Secrets Operator, Prometheus Operator

- **Key Insights:**
  - base64 = transport encoding (binary/char safety), NOT encryption
  - Opaque type = maximum flexibility, no validation
  - CSI Driver security > External Secrets Operator > Native K8s Secrets
  - File mounts auto-update files, but apps typically need restart
  - ConfigMaps = non-sensitive config, Secrets = sensitive (but properly secure them!)

### Health Checks and Lifecycle (Deep Dive)
- **The Problem Health Checks Solve:**
  - Container running ≠ application working (deadlocks, hangs, crash loops)
  - Need automated health monitoring and self-healing
  - Need to control when pods receive traffic

- **Three Types of Probes:**
  - **Liveness Probe**: "Is container alive or should I restart it?"
    - Detects deadlocks, hangs, memory leaks
    - Failure → kubelet kills container, restart policy applies
    - Should only check things restart would fix (internal process health)
    - Should NOT check external dependencies (DB, APIs)
  - **Readiness Probe**: "Is container ready to receive traffic?"
    - Temporarily removes pod from Service Endpoints
    - Failure → removed from iptables/ALB, container keeps running
    - Should check everything needed to serve traffic (including external deps)
    - When passes again → traffic resumes automatically
  - **Startup Probe**: "Has container finished starting up?"
    - Only runs during startup phase, then stops permanently
    - Disables liveness/readiness until startup succeeds
    - For slow-starting apps (legacy monoliths, ML models, DB recovery)
    - Allows patient startup + aggressive health checks after

- **Probe Methods:**
  - HTTP GET: Most common for web apps (success = 200-399)
  - TCP Socket: For databases and TCP services (success = connection established)
  - Exec Command: Run command in container (success = exit code 0)

- **Probe Configuration:**
  - `initialDelaySeconds`: Wait before first probe
  - `periodSeconds`: How often to probe
  - `timeoutSeconds`: Probe timeout
  - `failureThreshold`: Consecutive failures needed (prevents flapping)
  - `successThreshold`: Consecutive successes needed
  - Formula: `total time = periodSeconds × failureThreshold`

- **Lifecycle Hooks:**
  - **postStart**: Runs after container starts (before ENTRYPOINT guaranteed)
    - Use: Register with service, warm caches, setup config
    - If fails → container killed, restart policy applies
  - **preStop**: Runs before container terminates
    - Use: Graceful shutdown, drain connections, save state
    - Kubernetes waits for completion (up to terminationGracePeriodSeconds)

- **Graceful Shutdown Flow:**
  1. Pod marked for deletion in API Server (deletionTimestamp set)
  2. **Two parallel paths:**
     - **Path A (Stop Traffic)**: Endpoints Controller removes pod IP → kube-proxy updates iptables → ALB Controller deregisters from Target Group
     - **Path B (Shutdown)**: kubelet runs preStop → sends SIGTERM → waits grace period → SIGKILL if needed
  3. Race condition: Path B might finish before Path A propagates
  4. Solution: preStop hook with sleep (give time for traffic removal)

- **Integration with Services and Load Balancers:**
  - Readiness failure → kubelet updates Pod status → Endpoints Controller removes pod IP
  - kube-proxy watches Endpoints → updates iptables (removes pod from load balancing)
  - ALB Controller watches Endpoints → deregisters from Target Group
  - ALB has own health checks, but they're SECONDARY (primary is readiness probe)

- **Best Practices:**
  - Always use readiness probes (essential for zero-downtime deployments)
  - Use liveness carefully (only for stuck/deadlocked apps, not transient failures)
  - Separate liveness from readiness (different endpoints)
    - Liveness: `/alive` (simple: is process responding?)
    - Readiness: `/ready` (comprehensive: can serve traffic? includes DB check)
  - Use startup probe for slow apps (allows short liveness timeout after startup)
  - Always use preStop hook with sleep 15 (prevents race condition)
  - Handle SIGTERM gracefully in application code

- **Key Insights:**
  - Liveness = "Is container the problem?" (restart if yes)
  - Readiness = "Can you handle requests now?" (remove from traffic if no)
  - Rule: Only fail liveness if restart would fix the problem
  - Startup probe = temporary shield during vulnerable startup phase
  - preStop sleep prevents race between endpoint removal and shutdown
  - Total time = periodSeconds × failureThreshold (not just one value)

### Storage Fundamentals (Deep Dive)
- **The Problem Storage Solves:**
  - Pods are ephemeral - die, recreate, move between nodes
  - Databases and stateful apps need data to persist
  - Need storage independent of pod lifecycle

- **Three Core Concepts:**
  - **PersistentVolume (PV)**: Actual storage in cluster (like EBS volume)
    - Cluster-scoped resource
    - Represents real storage (EBS, EFS, NFS, etc.)
    - Has size and access modes
  - **PersistentVolumeClaim (PVC)**: Request for storage by user
    - Namespaced resource
    - Specifies size and access mode needed
    - Binds to PV that satisfies requirements
  - **StorageClass**: Template for dynamic storage provisioning
    - Cluster-scoped resource
    - Defines provisioner (e.g., AWS EBS CSI driver)
    - Contains provider-specific parameters (type, encryption, etc.)

- **Static vs Dynamic Provisioning:**
  - **Static (old way)**: Admin creates EBS volume + PV manually, PVC binds to existing PV
  - **Dynamic (modern)**: PVC references StorageClass → CSI creates EBS + PV automatically
  - Control plane role varies (create vs find), data plane role identical (kubelet → CSI Node Plugin)

- **Dynamic Provisioning Flow:**
  1. PVC Controller watches PVCs, sees storageClassName
  2. Looks up StorageClass, reads provisioner and parameters
  3. Calls CSI Controller Plugin (passes parameters + size from PVC)
  4. CSI Controller Plugin calls AWS CreateVolume
  5. CSI Controller Plugin creates PV object referencing new EBS volume
  6. PVC Controller binds PVC to new PV
  7. PVC status: Bound

- **StatefulSet Integration:**
  - volumeClaimTemplates generate PVCs automatically
  - Pattern: `<template-name>-<statefulset-name>-<ordinal>` (e.g., data-postgres-0)
  - StatefulSet Controller creates PVCs before pods
  - Each pod gets dedicated storage (1 replica = 1 PVC = 1 PV = 1 EBS volume)
  - **StatefulSet deletion does NOT delete PVCs** (safety feature)
  - Stable naming enables pod recreation to reconnect to same PVC

- **Access Modes:**
  - **ReadWriteOnce (RWO)**: Single node can mount read-write (EBS supports)
    - Multiple pods on same node can share
    - Use for databases, single-node apps
  - **ReadOnlyMany (ROX)**: Many nodes can mount read-only
  - **ReadWriteMany (RWX)**: Many nodes can mount read-write (EBS does NOT support)
    - EBS is block storage (single node attachment)
    - Must use EFS (shared filesystem) for RWX

- **Reclaim Policies:**
  - **Delete**: PVC deleted → PV deleted → EBS volume deleted (data gone forever!)
  - **Retain**: PVC deleted → PV becomes Released (not deleted), EBS volume preserved (data safe, admin can recover)
  - Production databases should use Retain policy

- **Pod Movement Across Nodes:**
  - Pod dies/moves, StatefulSet recreates with same name
  - Same name → references same PVC (deterministic naming)
  - CSI Node Plugin detaches EBS from old node, attaches to new node
  - Pod reconnects to same data automatically

- **Controllers and Watch Loops:**
  - **PVC Controller**: Watches PVCs and PVs, maintains bindings, calls CSI for provisioning
  - **StatefulSet Controller**: Watches StatefulSets, creates PVCs (from templates) and Pods
  - **CSI Controller Plugin**: Service (not watcher), called by PVC Controller, creates volumes and PVs
  - **CSI Node Plugin**: Service (not watcher), called by kubelet, attaches/mounts volumes
  - **kubelet**: Watches pods, calls CSI Node Plugin when pod needs volume

- **Integration with Probes and Lifecycle:**
  - Startup probe protects slow storage initialization (database loading data from disk)
  - preStop hook allows flush to disk before shutdown
  - Pod deletion: PVC/PV/EBS volume all retained, data accessible after pod gone
  - Can recreate pod or mount in different pod to access data

- **Key Insights:**
  - PV is cluster-scoped, PVC is namespaced, StorageClass is cluster-scoped
  - StorageClass defines HOW to provision (not size - size comes from PVC)
  - CSI plugins are services called by controllers, not independent watchers
  - StatefulSet deletion ≠ PVC deletion (two-stage cleanup for safety)
  - Deterministic naming (`data-database-0`) enables pod-to-storage reconnection
  - EBS = ReadWriteOnce only, need EFS for ReadWriteMany
  - Retain policy = production safety, Delete policy = dev convenience

### RBAC and ServiceAccounts (Deep Dive)
- **The Problem RBAC Solves:**
  - Need fine-grained access control for humans and applications
  - Principle of least privilege (grant only necessary permissions)
  - Prevent apps from accessing unauthorized resources

- **Two Identity Systems:**
  - **User Accounts**: External to K8s (AWS IAM, OIDC, certificates), no K8s User object
  - **ServiceAccounts**: K8s objects (namespaced), for pods/applications
  - Design: K8s handles authorization (RBAC), external systems handle authentication

- **Four RBAC Building Blocks:**
  - **ServiceAccount**: Identity (who)
  - **Role**: Permissions in one namespace (what + where)
  - **RoleBinding**: Connects ServiceAccount to Role (who + what)
  - **ClusterRole/ClusterRoleBinding**: Same but cluster-wide or for cluster-scoped resources

- **Role Anatomy (Three Parts):**
  - **apiGroups**: Resource grouping (core "", apps, networking.k8s.io, rbac.authorization.k8s.io)
  - **resources**: Object types (pods, services, deployments, ingresses)
  - **verbs**: Actions (get, list, watch, create, update, patch, delete)

- **Role vs ClusterRole:**
  - **Role**: Permissions in single namespace, created in that namespace
  - **ClusterRole**: Permissions across all namespaces OR cluster-scoped resources (nodes, PVs)
  - **ClusterRole + RoleBinding**: Reusable role definition, grant per-namespace

- **Cross-Namespace Access:**
  - RoleBinding location determines where permissions apply
  - RoleBinding can reference ServiceAccount from different namespace
  - Example: Frontend app (frontend ns) reading ConfigMap (backend ns) via RoleBinding in backend ns

- **ServiceAccount Token:**
  - Automatically mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`
  - Pod includes token in API requests (HTTP Authorization header)
  - API Server validates token and checks RBAC rules

- **API Server Flow:**
  1. Authentication: Is token valid? (validates ServiceAccount)
  2. Authorization: Does this ServiceAccount have permission? (checks RBAC rules)
  3. Allow or deny request

- **Update vs Patch:**
  - **Update**: Replace entire resource (requires GET first, race condition risk)
  - **Patch**: Partial update (no GET needed, no race, more efficient)
  - Controllers prefer patch (fewer API calls, safer concurrency)
  - RBAC typically grants both for flexibility

- **Subresources:**
  - Example: `ingresses/status` (status field only)
  - Controllers can update status without modifying spec
  - Separation: users/tools define spec, controllers report status

- **Watch Trio:**
  - Most controllers need: `get`, `list`, `watch`
  - `get`: Read single resource, `list`: Read all, `watch`: Stream changes

- **Default ServiceAccount:**
  - Every namespace has `default` ServiceAccount
  - Minimal permissions (can authenticate, not much else)
  - Apps without API access should use default or disable token mounting

- **Security Best Practices:**
  - Create dedicated ServiceAccounts for apps needing API access
  - Use `automountServiceAccountToken: false` for apps not needing API
  - Principle of least privilege (grant minimum necessary permissions)

- **IRSA (IAM Roles for ServiceAccounts):**
  - Two-layer permissions for AWS integration
  - **K8s RBAC**: Can pod talk to K8s API? (watch Ingresses, Services, etc.)
  - **AWS IAM**: Can pod call AWS APIs? (CreateLoadBalancer, GetSecret, etc.)
  - ServiceAccount annotation: `eks.amazonaws.com/role-arn`
  - Used by: AWS LB Controller, External Secrets Operator, CSI Driver

- **Key Insights:**
  - User Accounts external (auth flexibility), ServiceAccounts internal (K8s objects)
  - Role = 3 parts: apiGroups (where to look) + resources (what) + verbs (actions)
  - RoleBinding location determines permission scope, not ServiceAccount location
  - Patch safer than update (no race conditions, partial updates)
  - Subresources enable spec/status separation (users vs controllers)
  - Default ServiceAccount sufficient for apps without API access
  - IRSA combines K8s RBAC + AWS IAM for AWS API access

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

---

## Learning Pace & Timeline

**Current pace:** 2 topics per day
**Progress:** 10/15 core fundamentals complete (67%)
**Remaining:** 5 topics = 2.5 days to complete K8s fundamentals
**Hardest topics (8-9/10 complexity):** ✅ Complete!
**Average remaining complexity:** 5.2/10 (vs 6.9/10 already done)
**Target completion:** Day 2.5 (core), Day 12 (expert level)

---

## Next Steps - Core Fundamentals Track

**Day 1 (Next Session):**
1. Pod Security - SecurityContext, Pod Security Standards (Complexity: 4/10)

**Day 2:**
3. Network Policies - Network isolation between pods (Complexity: 5/10)
4. Advanced Scheduling - Affinity, anti-affinity, taints, tolerations (Complexity: 7/10)

**Day 3:**
5. Resource Management - ResourceQuotas, LimitRanges, QoS classes (Complexity: 5/10)
6. Disruptions and Availability - PodDisruptionBudgets, high availability (Complexity: 6/10)

**🎉 After Day 3:** Core Kubernetes fundamentals COMPLETE!

**Days 6-12 (Optional):** Advanced topics for deep expertise
- DaemonSets, Jobs, CronJobs
- HPA, VPA, Cluster Autoscaler
- Additional Service types
- CRDs, Operators, Admission Controllers

---

## Immediate Next Topics

**Up Next (Next Session):**
- RBAC and ServiceAccounts (Security and permissions model)

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

9. **Configuration Externalization**
   - ConfigMaps = non-sensitive config, Secrets = sensitive data
   - base64 = transport encoding for binary/special chars, NOT security
   - All Secret types use base64 - type field is for validation, not encoding
   - Default K8s Secrets insecure (base64 + etcd) - need RBAC + etcd encryption
   - Best security: CSI Driver bypasses etcd entirely (AWS → pod directly)
   - Operators = custom controllers that extend K8s with CRDs

10. **Health Checks and Self-Healing**
   - Liveness = "Should we restart?" (only fails if restart would fix it)
   - Readiness = "Should we send traffic?" (fails for any inability to serve requests)
   - Startup = temporary shield for slow-starting apps (runs once, then stops)
   - Different probes check different things: liveness = internal only, readiness = everything
   - Graceful shutdown = two parallel paths (stop traffic + shutdown container)
   - preStop with sleep prevents race condition (traffic removal finishes before shutdown)
   - Integration: readiness → Endpoints → kube-proxy/ALB (automatic traffic control)

11. **Storage and Data Persistence**
   - PV (cluster-scoped) = actual storage, PVC (namespaced) = storage request, StorageClass = provisioning template
   - Dynamic provisioning: PVC → StorageClass → CSI creates EBS + PV automatically
   - StatefulSet volumeClaimTemplates: deterministic naming enables pod-to-storage reconnection
   - StatefulSet deletion does NOT delete PVCs (safety feature for data)
   - Access modes: RWO (single node), RWX (many nodes, requires EFS not EBS)
   - Reclaim policies: Delete = data gone, Retain = data preserved (use for production)
   - Controllers watch (PVC Controller, StatefulSet Controller), CSI plugins are services
   - Data persists independently of pod lifecycle, can survive pod deletion and movement

12. **Access Control and Authorization**
   - Two identity systems: User Accounts (external, auth) vs ServiceAccounts (K8s objects, authz)
   - Four RBAC pieces: ServiceAccount (who) + Role (what) + RoleBinding (connection) + verbs (actions)
   - Role = 3 parts: apiGroups (where to look) + resources (what) + verbs (actions)
   - Role vs ClusterRole: namespace-scoped vs cluster-wide
   - RoleBinding location determines permission scope (not ServiceAccount location)
   - Cross-namespace access: RoleBinding references ServiceAccount from different namespace
   - ServiceAccount token auto-mounted, API Server validates and checks RBAC
   - Update (full replace, race risk) vs Patch (partial, no race, efficient)
   - Subresources (ingresses/status) enable spec/status separation
   - Watch trio: get, list, watch (needed by most controllers)
   - Default ServiceAccount minimal permissions, disable token if not needed
   - IRSA = K8s RBAC + AWS IAM (two-layer permissions for AWS APIs)

---

## Questions to Revisit Later

- When to use IPVS vs iptables mode?
- Best practices for Network Policies?
- How to size node pools for observability workloads?
- What metrics should we collect from the cluster itself?
- How does Prometheus Operator work vs manual Prometheus deployment?
- ConfigMap size limits and best practices for large configurations?
