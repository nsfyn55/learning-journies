# EKS/Kubernetes Learning Plan

## Deep Dives Completed ✅

### Kubernetes Architecture
**Status:** Deep dive complete
**Topics covered:**
- Control Plane: API Server, etcd, Scheduler, Controller Manager, Cloud Controller Manager
- Data Plane: kubelet, Container Runtime, kube-proxy, CNI Plugin, Pods
- Watch loop pattern and reconciliation model

**Gaps identified:**
- Scheduler: resource requirements, constraints, affinity rules, taints/tolerations
- Controller Manager: details of individual controllers
- Cloud Controller Manager: AWS-specific integrations

---

### Pod Networking
**Status:** Deep dive complete
**Topics covered:**
- Kubernetes networking model (pod IPs, no NAT, flat network)
- AWS VPC CNI specifics (real VPC IPs, ENIs, warm pool)
- Network namespaces and isolation
- veth pairs (virtual ethernet cables)
- Complete pod startup flow

---

### Service Networking
**Status:** Deep dive complete
**Topics covered:**
- Problem Services solve (ephemeral pods, stable endpoints, load balancing)
- How Services work (ClusterIP, Endpoint Controller, kube-proxy)
- iptables mode (DNAT, probability rules, local rewriting)
- Pod lifecycle and IP changes

**Gaps identified:**
- IPVS mode details
- Other Service types (NodePort, LoadBalancer, ExternalName)

---

### Ingress and Ingress Controllers
**Status:** Deep dive complete
**Topics covered:**
- Problem with LoadBalancer Services (cost, no path/host routing)
- Ingress resources (routing configuration objects)
- Ingress Controller pattern (watch loop, reconciliation)
- AWS Load Balancer Controller specifics
- Two modes: Instance mode vs IP mode
- Layer 7 (HTTP) routing: path-based and host-based
- ALB Target Groups and pod IP registration
- Integration with Endpoints controller
- Security group and subnet architecture

**Gaps identified:**
- Other Ingress Controllers (nginx, Traefik)
- TLS/SSL certificate management with Ingress
- Advanced annotations and features

---

### DNS / Service Discovery
**Status:** Deep dive complete
**Topics covered:**
- Problem DNS solves (name-based discovery vs hard-coded IPs)
- CoreDNS architecture (Service + Pods in kube-system, watches API)
- How pod DNS works (/etc/resolv.conf, nameserver, search domains)
- DNS naming convention (`<service>.<namespace>.svc.cluster.local`)
- Short names (same namespace) vs cross-namespace requirements
- DNS vs API-based discovery (name resolution vs dynamic discovery)
- Prometheus service discovery (`kubernetes_sd_configs`, relabeling)
- Watch mechanism at HTTP protocol level (chunked transfer encoding, RFC 7230)
- Two types of namespaces (Linux kernel vs Kubernetes API)
- Complete DNS flow: name → CoreDNS → ClusterIP → kube-proxy → Pod IP

**Gaps identified:**
- Prometheus Operator vs manual Prometheus deployment
- ConfigMap size limits for large Prometheus configurations
- Advanced CoreDNS configurations and customizations

---

### Workload Resources
**Status:** Deep dive complete
**Topics covered:**
- Deployments (stateless applications)
- Deployment → ReplicaSet → Pod hierarchy
- Deployment Controller and ReplicaSet Controller watch loops
- Rolling update mechanism (maxUnavailable, maxSurge)
- Why Deployments use ReplicaSets (enables versioning and rollback)
- StatefulSets (stateful applications with stable identity)
- Stable pod names (postgres-0, postgres-1) vs random names
- Stable DNS names that persist across node moves
- Pod IPs still change when rescheduled (only DNS names are stable)
- Headless Services (ClusterIP: None, returns pod IPs directly)
- Ordered operations (scale up/down, updates)
- PersistentVolumeClaims for storage that follows pods
- When to use Deployments vs StatefulSets
- Resource requests and limits (CPU, memory)
- How Scheduler uses requests for placement decisions
- How kubelet enforces limits via Linux cgroups
- CPU throttling vs memory OOMKill
- cgroups v1 and v2 architecture
- CFS bandwidth control (period/quota for CPU)
- Integration with Ingress/ALB (Endpoints Controller updates regardless of workload type)

**Gaps identified:**
- DaemonSets (one pod per node)
- Jobs and CronJobs (batch workloads)
- Deployment strategies (Recreate vs RollingUpdate)
- StatefulSet update strategies (OnDelete, RollingUpdate)
- Pod Disruption Budgets
- HorizontalPodAutoscaler (HPA)

---

### ConfigMaps and Secrets
**Status:** Deep dive complete
**Topics covered:**
- Problem solved: Externalize configuration from application code/images
- Two consumption methods: environment variables and volume mounts
- ConfigMaps for non-sensitive config, Secrets for sensitive data
- Storage in etcd: ConfigMaps as plaintext, Secrets as base64 encoded
- Base64 encoding reasons: binary data support, character safety, etcd storage, API transport
- Base64 is NOT security - trivial to decode
- Secret types: Opaque (default, no validation), TLS, basic-auth, dockerconfigjson
- Opaque type: catch-all for arbitrary key-value pairs, no structure requirements
- AWS Secrets Manager integration patterns
- External Secrets Operator: syncs from AWS to K8s Secrets (stored in etcd)
- CSI Driver: mounts directly from AWS, never stored in etcd (most secure)
- IRSA (IAM Roles for Service Accounts) for AWS authentication
- Security: etcd encryption at rest, RBAC, external secret managers
- Lifecycle: file mounts auto-update, env vars require pod restart
- Rolling restarts for credential rotation (kubectl rollout restart)
- Operators pattern: custom controllers for custom resources (CRDs)

**Gaps identified:**
- etcd encryption at rest configuration
- Advanced External Secrets Operator features (multi-region, failover)
- CSI driver performance implications
- Secret rotation strategies and best practices

---

### Health Checks and Lifecycle
**Status:** Deep dive complete
**Topics covered:**
- Three probe types: liveness, readiness, startup
- Liveness: detects deadlocks/hangs, only checks internal health, restart on failure
- Readiness: controls traffic, checks all dependencies, removes from service on failure
- Startup: patient during startup phase, stops once app is running
- Probe methods: HTTP GET, TCP Socket, Exec Command
- Probe configuration: periodSeconds, failureThreshold, formula (total time = period × failures)
- Lifecycle hooks: postStart (initialization), preStop (graceful shutdown)
- Graceful shutdown flow: two parallel paths (stop traffic + shutdown container)
- Race condition: shutdown can happen before traffic removal completes
- Solution: preStop with sleep 15 (gives time for iptables/ALB updates)
- Integration: readiness → Pod status → Endpoints → kube-proxy/ALB
- Best practices: separate liveness from readiness, different endpoints, SIGTERM handling

**Gaps identified:**
- Advanced health check patterns for distributed systems
- Custom probe implementations
- Metrics and alerting for probe failures

---

### Storage Fundamentals
**Status:** Deep dive complete
**Topics covered:**
- Problem solved: Persistent storage independent of pod lifecycle
- Three core concepts: PV (actual storage), PVC (storage request), StorageClass (provisioning template)
- Static vs dynamic provisioning (manual vs automatic EBS creation)
- Dynamic provisioning flow: PVC → StorageClass → CSI Controller → AWS CreateVolume → PV
- StatefulSet integration: volumeClaimTemplates with deterministic naming
- StatefulSet deletion does NOT delete PVCs (safety feature)
- Access modes: RWO (single node, EBS supports), RWX (many nodes, requires EFS)
- Reclaim policies: Delete (data gone) vs Retain (data preserved)
- Pod movement: CSI Node Plugin detaches/attaches EBS across nodes
- Controllers: PVC Controller (watches, binds, provisions), StatefulSet Controller (creates PVCs/Pods)
- CSI plugins are services (not watchers), called by controllers
- Integration with probes: startup protects slow storage init, preStop allows flush

**Gaps identified:**
- EFS configuration and use cases
- Volume snapshots and backups
- Storage performance tuning
- Multi-AZ storage considerations

---

### RBAC and ServiceAccounts
**Status:** Deep dive complete
**Topics covered:**
- Problem solved: Fine-grained access control for humans and applications
- Two identity systems: User Accounts (external, IAM) vs ServiceAccounts (K8s objects, namespaced)
- Four RBAC building blocks: ServiceAccount (who), Role (what+where), RoleBinding (who+what), verbs (actions)
- Role anatomy: apiGroups (resource grouping) + resources (object types) + verbs (actions)
- API groups: core (""), apps, networking.k8s.io, rbac.authorization.k8s.io
- Role vs ClusterRole: namespace-scoped vs cluster-wide permissions
- RoleBinding can reference ServiceAccounts from different namespaces
- ClusterRole + RoleBinding pattern: reusable role, grant per-namespace
- ServiceAccount token mounted at /var/run/secrets/kubernetes.io/serviceaccount/token
- API Server enforces RBAC: authentication → RBAC check → allow/deny
- Update vs Patch: full replacement vs partial update (no race conditions)
- Subresources: ingresses/status for status-only updates (prevents spec modification)
- Watch trio: get, list, watch (needed by most controllers)
- Default ServiceAccount: minimal permissions, use for apps that don't need API access
- automountServiceAccountToken: false to disable for security
- IRSA: Two-layer permissions (K8s RBAC + AWS IAM) for AWS API access

**Gaps identified:**
- User authentication methods (OIDC, certificates)
- Advanced RBAC patterns (aggregated ClusterRoles)
- RBAC troubleshooting and auditing
- Service Account token projection and expiration

---

### Network Policies
**Status:** Deep dive complete
**Topics covered:**
- Problem solved: Flat network security risks (lateral movement, compliance, defense in depth)
- Default behavior: allow-all switches to deny-all when policy selects a pod
- Three main parts: podSelector (which pods), policyTypes (Ingress/Egress), ingress/egress rules
- Multiple policies are OR'd (additive), within one policy entry conditions are AND'd
- Ingress/egress selectors: podSelector, namespaceSelector, ipBlock
- AWS VPC CNI doesn't support Network Policies (needs separate engine)
- Calico architecture: calico-kube-controllers + Felix DaemonSet
- Felix manages ipsets (IP-to-label mapping) and writes iptables rules
- ipsets are Linux kernel feature (hash tables for efficient IP matching)
- Enforcement at Layer 3 via iptables chains (cali-tw-* ingress, cali-fw-* egress)
- Traffic flow: DNS → DNAT (kube-proxy) → egress check → VPC routing → ingress check
- Pre-computed rules (no API queries at packet time)
- DNS egress must be explicitly allowed if egress policy exists

**Gaps identified:**
- Cilium eBPF implementation details
- Network Policy troubleshooting and debugging
- Performance implications at scale
- Advanced patterns (deny rules via empty allow lists)

---

### Pod Security
**Status:** Deep dive complete
**Topics covered:**
- Problem solved: Control container privileges, minimize blast radius, defense-in-depth
- Two mechanisms: SecurityContext (pod/container specs) + Pod Security Standards (namespace enforcement)
- SecurityContext: pod-level (applies to all) and container-level (overrides pod-level)
- Key fields: runAsUser, runAsGroup, fsGroup, runAsNonRoot, allowPrivilegeEscalation, readOnlyRootFilesystem
- Linux capabilities: Fine-grained privileges (NET_BIND_SERVICE, CHOWN, etc.), drop ALL then add back
- fsGroup: Enables non-root containers to access volumes (kubelet chowns to root:fsGroup)
- fsGroup added as supplementary group to all containers in pod
- Three Pod Security Standards: Privileged (no restrictions), Baseline (blocks dangerous), Restricted (heavily locked down)
- Pod Security Admission Controller: Enforces standards at Pod creation time (not Deployment)
- Three modes: enforce (block), audit (log), warn (show warning)
- Async enforcement: Deployment succeeds, Pod creation fails, check ReplicaSet events
- Best practices: Baseline for most apps, Restricted for sensitive workloads, Privileged only for system components

**Gaps identified:**
- Advanced capability use cases
- Pod Security Admission customization
- Migration strategies for existing workloads
- seccomp, AppArmor, SELinux profiles

---

## Surface-Level Knowledge (No Deep Dive Yet) 📖

### Service Types
- ClusterIP: Covered basics
- NodePort: Mentioned, explored in context of Ingress Instance mode
- LoadBalancer: Mentioned, explored in context of Ingress problem space
- ExternalName: Not covered

---

## Not Yet Covered ⏳

### Security
- ✅ Pod Security Standards (Complete)
- ✅ Network Policies (Complete)

### Observability Specifics
- Prometheus architecture
- Prometheus Operator
- Service discovery mechanisms
- Grafana deployment patterns
- SLO/SLI definitions

### Advanced Topics
- Service Mesh (Istio, Linkerd)
- eBPF-based networking
- Custom Resource Definitions (CRDs)
- Operators

---

## Learning Pace

**Target:** 2 topics per day
**Core fundamentals remaining:** 3 topics = 1.5 days
**Progress:** 12/15 complete (80%)
**Hardest topics (8-9/10 complexity):** ✅ Complete!
**Total to K8s mastery:** 1.5 days to core, 2 weeks to expert

---

## Core Fundamentals Roadmap (Essential - 15 Topics Total)

### ✅ Completed (12/15) - 80%
1. Kubernetes Architecture - Complexity: 6/10
2. Pod Networking - Complexity: 9/10 ⭐ (Hardest topic)
3. Service Networking - Complexity: 8/10
4. Ingress Controllers - Complexity: 7/10
5. DNS / Service Discovery - Complexity: 7/10
6. Workload Resources - Complexity: 7/10
7. ConfigMaps and Secrets - Complexity: 5/10
8. Health Checks and Lifecycle - Complexity: 6/10
9. Storage Fundamentals - Complexity: 8/10
10. RBAC and ServiceAccounts - Complexity: 6/10
11. Network Policies - Complexity: 5/10
12. Pod Security - Complexity: 4/10 ✅

**Average completed complexity: 6.5/10** (Hardest topics behind you!)

### 🔄 Remaining (3/15) - Target: 1.5 Days

**Day 1 (Next Session):**
13. Advanced Scheduling (20-25 min) - Complexity: 7/10
14. Resource Management (15-20 min) - Complexity: 5/10

**Day 2:**
15. Disruptions and Availability (15-20 min) - Complexity: 6/10

**Average remaining complexity: 6/10** (Moderate difficulty!)

---

## Optional Topics for Deep Expertise (13 Topics)

### Advanced Workloads (Day 6)
16. DaemonSets (10-15 min)
17. Jobs and CronJobs (15-20 min)

### Autoscaling (Day 7-8)
18. Horizontal Pod Autoscaling - HPA (20-25 min)
19. Vertical Pod Autoscaling - VPA (15-20 min)
20. Cluster Autoscaler (15-20 min)

### Service Types (Day 8-9)
21. NodePort and LoadBalancer Services (10-15 min)
22. ExternalName and Headless Services (10-15 min)

### Advanced Networking (Day 9-10)
23. IPVS Mode (15-20 min)
24. Multi-cluster Networking (20-25 min) - Optional

### Extension Points (Day 10-12)
25. Custom Resource Definitions - CRDs (20-25 min)
26. Operators Pattern (25-30 min)
27. Admission Controllers (20-25 min)

### API Internals (Day 12)
28. Kubernetes API Deep Dive (20-25 min)

---

## Milestones

**Milestone 1 (Day 5):** Complete K8s Core Fundamentals ✅
- Can deploy and manage any application on Kubernetes
- Ready to build observability platform

**Milestone 2 (Day 12):** Kubernetes Expert ✅
- Deep understanding of K8s internals
- Can extend and customize Kubernetes
- Platform engineering level knowledge

**After Milestone 1:** Begin Prometheus/Grafana/SLO implementation

---

## Retention Scores 📊

**Date: 2025-10-11**

### Control Plane Architecture
**Score: 🟡 Good (75%)**

**What you nailed:**
- Two-tier architecture (control plane vs data plane)
- Four main components: API Server, etcd, Controller Manager, Scheduler
- etcd as state store
- Controller manager's watch-and-reconcile pattern
- Scheduler's pod placement role

**What needed work:**
- ❌ Missed Cloud Controller Manager entirely
- ❌ Misunderstood API Server role (said "external communication" vs "central hub for ALL communication")
- Gap: Scheduler decision-making criteria (correctly noted you haven't done deep dive on this)

**Action items:**
- Review Cloud Controller Manager's AWS-specific integrations
- Remember: API Server is the hub for all components, not just external clients

---

### Data Plane Architecture
**Score: 🟢 Strong (85%)**

**What you nailed (after refinement):**
- All components accounted for: kubelet, kube-proxy, container runtime, CNI, Pods
- kubelet as bridge to API server
- kubelet watches API server for pod assignments
- kubelet calls CNI for networking setup
- Distinguished ephemeral Pod IPs vs stable Service IPs
- Pods can have multiple containers sharing network namespace

**What needed work:**
- Initially said kubelet "scans" vs "watches" (watch = active push, scan = polling)
- Needed clarification on kube-proxy's role (implements vs creates ClusterIPs)

**Result:** Massive improvement from first attempt! Solid understanding.

---

### Service Networking
**Score: 🟢 Strong (92%)**

**Learning journey:**
- Initial attempt: 🔴 40% - Confused Services with Pods, thought kubelet was involved
- After corrections: 🟢 92% - Nailed the complete picture!

**What you nailed (final attempt):**
- Pod IPs are ephemeral and in flux
- Services provide stability via ClusterIP
- **kube-proxy watches BOTH Services and Endpoints** (critical insight!)
- Service contains ClusterIP and label selectors
- Endpoints contains actual pod IPs matching selectors
- iptables rewrites ClusterIP → pod IPs
- Clear flow and explanation structure

**Critical errors corrected:**
- ❌ Initially: "API server assigns ClusterIP to pods" → ✅ ClusterIPs assigned to Services, not pods
- ❌ Initially: "kubelet interfaces with kube-proxy" → ✅ kube-proxy watches API server independently

**Key insight mastered:** kube-proxy bridges the virtual ClusterIP world to the real pod IP world by watching Services AND Endpoints.

---

### Pod Networking
**Score: 🟢 Strong (95%)**

**What you absolutely crushed:**
- Pod = one or more containers sharing network namespace
- ENI with primary + secondary IPs (warm pool concept)
- kubelet watches API server → invokes CNI
- veth pair connects host and pod namespaces (host side = hashed name, pod side = eth0)
- **Routing rules** (not iptables) direct traffic for pod IPs through veth
- **KEY INSIGHT**: Primary IP = host traffic, Secondary IPs = pod pool
- Secondary IP without assignment = dropped
- Secondary IP with assignment = routing rule exists → traffic flows to pod

**Your money quote:**
> "The host knows that packets for the primary address should be treated as the host's traffic. If packets arrive destined for a secondary address they are dropped if not assigned to a pod, otherwise a route will exist that directs those packets to exit the target veth which makes them arrive on eth0 inside the pod's network namespace."

**Minor refinements learned:**
- veth naming clarified (host's eth0 ≠ veth interface)
- Routing rules vs iptables (Layer 3 routing vs packet filtering)
- CNI creates network namespace before veth pair

**Result:** Explanation is colleague-ready! Could explain this to your team today.

---

### Ingress and Ingress Controllers
**Score: 🟢 Strong (95%)**

**What you nailed:**
- Problem space: LoadBalancer Services are expensive and lack HTTP-layer routing
- Ingress = routing configuration (Kubernetes API object)
- Ingress Controller = software that implements Ingress (watch loop pattern)
- AWS Load Balancer Controller creates ALB + Target Groups + Listener Rules
- Two modes: Instance mode (via NodePort + kube-proxy) vs IP mode (direct to pods)
- **Chose IP mode** for efficiency and lower latency
- Watches Endpoints to get pod IPs for Target Group registration
- Layer 7 routing: path-based (`/grafana`) and host-based (`grafana.mycompany.com`)
- Security group and subnet architecture (ALB public, pods private)

**Strong architectural reasoning:**
- ✅ Understood why ClusterIPs don't work for external ALBs (virtual IPs, no iptables outside cluster)
- ✅ Correctly identified IP mode targets pod IPs directly
- ✅ Made valid argument: "ALB must re-implement load balancing because it can't use virtual ClusterIPs"

**Key insight mastered:**
> "The ALB can't rely on kube-proxy to route and load balance using the cluster IP because of its virtual nature"

This shows deep understanding of the networking stack layers!

**Complete traffic flow:**
```
Internet → ALB (Layer 7 routing) → Pod IP directly
```

vs internal:
```
Pod A → Service ClusterIP → kube-proxy iptables → Pod B IP
```

**What you defended successfully:**
- ALB "re-implements" load balancing - Valid! Both kube-proxy and ALB Controller watch Endpoints and distribute traffic across pod IPs, just using different mechanisms (iptables vs Target Groups)

**Result:** Ready to architect and explain this to your team. Solid grasp of the three networking layers!

---

### DNS / Service Discovery
**Score: 🟢 Strong (98%)**

**Learning journey:**
- Initial explanation: 🟡 75-85% - Good concepts, missing some details
- After deep dive: 🟢 98% - Exceptional retention!

**What you nailed (second attempt):**
- DNS returns Service ClusterIPs, never Pod IPs ✅
- CoreDNS architecture: Service + Pods in kube-system, watches API server ✅
- Two-layer translation: DNS (name→ClusterIP) + iptables (ClusterIP→Pod IP) ✅
- Prometheus uses API queries (not DNS) because it needs individual pod IPs ✅
- `kubernetes_sd_configs` watches API server for pods ✅
- Relabeling filters targets based on annotations/labels ✅
- Watch mechanism is NOT polling - event-driven HTTP streaming ✅
- Distinction between Linux kernel namespaces vs Kubernetes namespaces ✅

**Initial gaps corrected:**
- ❌ First attempt: Said DNS returns "postgres pod cluster IP" (confused terminology)
- ✅ Correction: DNS returns Service ClusterIP, Services have ClusterIPs (not pods)
- ❌ First attempt: Fuzzy on how Prometheus discovers pods
- ✅ Correction: Explicitly stated it watches API server via `kubernetes_sd_configs`

**Deep dive bonus:**
- Asked clarifying questions about HTTP protocol (chunked transfer encoding) ✅
- Distinguished Kubernetes namespaces from Linux network namespaces ✅
- Understood CoreDNS is authoritative for *.cluster.local, forwarder for external ✅
- Recognized `kubernetes_sd_configs` is Prometheus config stored in ConfigMap ✅

**Key insight mastered:**
> "CoreDNS sits on top of the resolution stack. It gets the first crack at resolution."

Perfect mental model! Shows understanding of DNS hierarchy and forwarding.

**Result:** Colleague-ready explanation. Could explain DNS flow and Prometheus service discovery to your team today!

---

### ConfigMaps and Secrets
**Score: 🟢 Strong (90%)**

**What you nailed:**
- Problem solved: Externalize configuration from application code ✅
- Two consumption methods: environment variables and volume mounts ✅
- Storage: ConfigMaps plaintext, Secrets base64 encoded ✅
- Base64 reasons: PRIMARY = binary data support, SECONDARY = character safety ✅
- Base64 is NOT security - trivial to decode ✅
- Opaque type: default catch-all with no validation ✅
- ALL Secret types use base64 (not just Opaque) ✅
- CSI Driver vs External Secrets Operator: etcd storage difference ✅
- Security: RBAC, etcd encryption, external secret managers ✅

**What needed clarification:**
- Initially mixed up consumption methods (mentioned CSI for basic mounts)
- CSI driver is for external secrets only, not required for basic ConfigMap/Secret volume mounts
- Flow walkthrough for CSI vs External Secrets needed prompting
- Application restart implications for credential rotation

**Key insights mastered:**
- Base64 = transport encoding for binary data and special chars, NOT encryption
- Opaque type = flexible catch-all (any keys), other types have validation
- CSI Driver = secrets never touch etcd (most secure)
- External Secrets Operator = secrets stored in etcd (more convenient)
- File mounts auto-update, but apps typically need restart to use new values

**Bonus topics covered:**
- Operators pattern: custom controllers for custom resources (CRDs)
- IRSA (IAM Roles for Service Accounts) for AWS auth
- Rolling restart mechanism: `kubectl rollout restart deployment`
- Practical reality: most apps cache credentials at startup

**Result:** Strong understanding of configuration management and security tradeoffs. Ready to implement secret management for observability platform!

---

### Health Checks and Lifecycle
**Score: 🟢 Strong (94%)**

**Learning journey:**
- Question 1 (Liveness vs Readiness): 🟡 65% → 🟢 95% (after one correction)
- Question 2 (Graceful Shutdown): 🟠 50% → 🟢 95% (after explanation)
- Question 3 (Startup Probe Config): 🟡 75% → 🟢 92% (minor refinement)

**What you absolutely crushed (final attempts):**
- ✅ "A liveness check should only fail if a restart could solve the problem" (money quote!)
- ✅ Liveness checks container health in isolation
- ✅ Readiness checks everything needed to complete tasks
- ✅ NFS mount example showing external dependency understanding
- ✅ Two parallel paths during pod deletion: stop traffic + shutdown container
- ✅ Race condition: shutdown can finish before traffic removal propagates
- ✅ preStop sleep solution: gives time for kube-proxy and ALB to respond
- ✅ Startup probe configuration: periodSeconds × failureThreshold = total time
- ✅ Different timing requirements for different phases

**Initial misconceptions corrected:**
- ❌ First attempt: "Liveness could check same thing as readiness with longer threshold"
- ✅ Correction: They check DIFFERENT things (internal vs everything), not just different timing
- ❌ First attempt: Confused about why kubelet wasn't doing both activities
- ✅ Correction: kubelet is local to one node, traffic control is distributed across cluster
- ❌ First attempt: Said "75 second probe" instead of configuration parameters
- ✅ Correction: Use periodSeconds and failureThreshold to calculate total time

**Key insights mastered:**
- Liveness = "Is the container the problem?" → restart if yes
- Readiness = "Can you handle requests now?" → remove from traffic if no
- Only fail liveness for problems that restart would fix
- External dependencies (DB, NFS) should only affect readiness, not liveness
- Two parallel paths exist because traffic control is distributed (many nodes, ALB zones)
- preStop sleep is best practice to prevent race condition
- Startup probe allows patient startup + aggressive health checks after

**Your money quotes:**
1. "A liveness check should only fail if a restart could solve the problem"
2. "A container can be healthy without being able to service requests"
3. "Here it is a good practice to set a wait to give kube-proxy and ingress controller enough time to respond"

**Result:** Colleague-ready! Went from 50-75% initial attempts to 92-95% after single correction cycles. Excellent retention and quick learning.

---

### Network Policies
**Score: 🟢 Strong (92%)**

**Date: 2025-10-13**

**Learning journey:**
- Question 1 (Problem/Behavior): 🟡 80% → 🟢 95% (minor refinement)
- Question 2 (Policy Anatomy): 🟢 90% (got concepts, missed policyTypes field)
- Question 3 (Traffic Flow): 🟠 65% → 🟢 90% (after two retries with deep dives)
- Question 4 (Multiple Policies): 🟢 100% (perfect!)
- Question 5 (AWS/EKS Specifics): 🟡 70% → 🟢 92% (after refinement)

**What you absolutely crushed:**
- ✅ Flat network security problem (lateral movement, compliance, defense in depth)
- ✅ Default behavior switch: allow-all → deny-all when policy selects pod
- ✅ Multiple policies are OR'd (additive logic)
- ✅ Traffic flow with Felix/ipsets/iptables integration
- ✅ Felix manages ipset membership based on pod labels
- ✅ ipsets are Linux kernel feature (not Calico-specific)
- ✅ Layer 3 enforcement with pre-computed rules (no API queries at packet time)
- ✅ VPC CNI separation of concerns (networking vs policy enforcement)

**Initial gaps corrected:**
- ❌ First attempt Q1: Said "all traffic explicitly DENY" (too broad)
- ✅ Correction: Only pods matched by podSelector switch to deny-all
- ❌ First attempt Q3: Said traffic would be allowed (missed that frontend ≠ backend label)
- ✅ Correction: Understood complete flow including ipset membership checks
- ❌ Initially confused: "Does Felix query API at packet time for labels?"
- ✅ Correction: Felix pre-populates ipsets, packet time is pure Layer 3 lookup

**Key insights mastered:**
- podSelector scope: Policy only affects pods it selects, not cluster-wide
- policyTypes field: Declares which traffic types are controlled (Ingress/Egress)
- Multiple policies OR'd: Union of rules, composable without conflict
- Within policy AND'd: Conditions in same `from` entry must all match
- ipsets = IP-to-label mapping maintained by Felix via continuous watch
- Calico chains: cali-tw-* (to-workload/ingress), cali-fw-* (from-workload/egress)
- Traffic flow ordering: DNS → DNAT → egress check → VPC → ingress check
- DNS egress must be whitelisted if egress policy exists

**Your money realizations:**
1. "Felix/Calico's job is to manage membership in those ipsets" (spot on!)
2. Understanding ipsets as hash tables for O(1) lookup efficiency
3. Recognizing traffic evaluation happens BEFORE reaching destination pod

**Result:** Strong understanding of Network Policies from problem space to technical implementation. Ready to design and implement policies for observability platform!

---

### Pod Security
**Score: 🟢 Strong (88%)**

**Date: 2025-10-14**

**Learning journey:**
- Question 1 (SecurityContext): 🟡 70% → 🟢 100% (after fsGroup deep dive)
- Question 2 (Problem Space): 🟢 90%
- Question 3 (Real-World Fix): 🟡 70% (conceptual correct, lacked specifics)
- Question 4 (Pod Security Standards): 🟢 95%
- Question 5 (Enforcement Flow): 🟢 92% (after refresher)
- Question 6 (Production Config): 🟡 65% → 🟢 98% (second attempt)

**What you absolutely crushed:**
- ✅ Blast radius framing (consistent mental model throughout)
- ✅ Risk-based security decisions (Privileged for system, Baseline for apps, Restricted for sensitive)
- ✅ Async enforcement flow (Deployment succeeds, Pod creation fails, ReplicaSet retries)
- ✅ Container-level overrides pod-level SecurityContext
- ✅ fsGroup mental model after deep dive (enables non-root volume access)
- ✅ Principle of least privilege thinking
- ✅ Defense-in-depth (not just attacks, but bugs too)

**Initial gaps corrected:**
- ❌ Forgot fsGroup added as supplementary group to ALL containers
- ❌ Didn't know Linux capabilities (NET_BIND_SERVICE for port 80)
- ❌ Confused "baseline" standard with capability name
- ❌ Initially used generic UIDs instead of image-specific (postgres = 999)
- ❌ Needed refresher on when PSA checks happen (Pod creation, not Deployment)

**Key insights mastered:**
- fsGroup = "volume access group" that enables non-root containers (vs running as root)
- fsGroup added to all containers as supplementary group (primary group can differ)
- Capabilities are fine-grained (NET_BIND_SERVICE, CHOWN) - drop ALL, add back minimum
- Pod Security Admission Controller enforces at Pod creation (async from Deployment)
- ReplicaSet remains in etcd, keeps retrying, errors in events
- Baseline = recommended default, Restricted = high-security, Privileged = system only
- readOnlyRootFilesystem requires writable volumes (emptyDir for /tmp)

**Your money quotes:**
1. "The blast radius can be devastating. Therefore it is worth the extra complexity of locking them down."
2. "Why can't you give both app and sidecar the same UID?" (critical thinking about necessity vs convention)
3. "If I put 2000 wouldn't 999 get this as a supplementary group?" (testing mental model with alternatives)

**Strongest moments:**
- Catching async behavior implications (ReplicaSet in etcd?)
- Questioning fsGroup design (could use same UID) - showed deep thinking
- Risk-based reasoning for Pod Security Standards per namespace

**Result:** Colleague-ready on concepts and best practices. Would reference documentation for specific capabilities and image UIDs, but understands the security model deeply.

---

### Overall Assessment

**Strengths:**
- Strong understanding of networking fundamentals once concepts click
- Good at incorporating feedback and refining explanations
- Excellent retention of complex technical details (warm pool, veth pairs, routing rules)

**Areas for improvement:**
- Watch for component interaction errors (which component talks to which)
- Be precise about what's assigned to what (Service vs Pod IPs)
- Cloud Controller Manager needs a refresh

**Next retention exercises to try:**
- Complete pod startup flow (end-to-end)
- Service request flow (client → ClusterIP → pod)
- What happens when a pod dies and gets recreated
- External request flow through Ingress (internet → ALB → pod)
- DNS resolution flow (app queries service name → DNS → ClusterIP → pod)
- Explain why Prometheus can't use DNS for pod discovery
- Trace a watch connection from component to API server to etcd

---

## Learning Strategy

1. **Deep Dive** = 30+ min focused study with hands-on practice
2. **Surface Level** = Basic understanding of what it is and when to use it
3. **Retention Exercise** = Explain concept back without notes, get feedback

**Scoring System:**
- 🟢 **Strong (90-100%)**: Confident explanation with key details - ready to explain to colleagues
- 🟡 **Good (70-89%)**: Solid understanding with minor gaps - needs minor refinement
- 🟠 **Fair (50-69%)**: Basics understood but missing important details - needs review
- 🔴 **Needs Review (<50%)**: Significant gaps or misconceptions - requires re-study
