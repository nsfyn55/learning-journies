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

## Surface-Level Knowledge (No Deep Dive Yet) 📖

### Service Types
- ClusterIP: Covered basics
- NodePort: Mentioned, explored in context of Ingress Instance mode
- LoadBalancer: Mentioned, explored in context of Ingress problem space
- ExternalName: Not covered

### Network Policies
- Pod-to-pod traffic restrictions
- Security isolation
- Egress controls

---

## Not Yet Covered ⏳

### Storage
- Persistent Volumes (PV)
- Persistent Volume Claims (PVC)
- Storage Classes
- StatefulSet storage

### Security
- RBAC (Role-Based Access Control)
- Service Accounts
- Pod Security Standards
- Network Policies (mentioned above)

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
**Core fundamentals remaining:** 8 topics = 4 days
**Optional expertise topics:** 13 topics = 6-7 days
**Total to K8s mastery:** 2-3 weeks

---

## Core Fundamentals Roadmap (Essential - 15 Topics Total)

### ✅ Completed (7/15) - 47%
1. Kubernetes Architecture
2. Pod Networking
3. Service Networking
4. Ingress Controllers
5. DNS / Service Discovery
6. Workload Resources
7. ConfigMaps and Secrets

### 🔄 Remaining (8/15) - Target: 4 Days

**Day 1 (cont.):**
8. Health Checks and Lifecycle (15-20 min) - Liveness, readiness, graceful shutdown

**Day 2:**
9. Storage Fundamentals (25-30 min) - PV, PVC, StorageClasses
10. RBAC and ServiceAccounts (20-25 min) - Security and permissions

**Day 3:**
11. Pod Security (15-20 min) - SecurityContext, Pod Security Standards
12. Network Policies (20-25 min) - Network isolation and firewall rules

**Day 4:**
13. Advanced Scheduling (20-25 min) - Affinity, taints, tolerations
14. Resource Management (15-20 min) - Quotas, LimitRanges, QoS

**Day 5:**
15. Disruptions and Availability (15-20 min) - PodDisruptionBudgets, high availability

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
