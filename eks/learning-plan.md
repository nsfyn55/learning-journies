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

## Surface-Level Knowledge (No Deep Dive Yet) 📖

### Service Types
- ClusterIP: Covered basics
- NodePort: Mentioned, explored in context of Ingress Instance mode
- LoadBalancer: Mentioned, explored in context of Ingress problem space
- ExternalName: Not covered

### DNS / Service Discovery
- HIGH PRIORITY for Prometheus
- CoreDNS
- Service name resolution
- Prometheus service discovery

### Network Policies
- Pod-to-pod traffic restrictions
- Security isolation
- Egress controls

---

## Not Yet Covered ⏳

### Workload Resources
- Deployments
- ReplicaSets
- StatefulSets (for databases)
- DaemonSets
- Jobs and CronJobs

### Configuration and Secrets
- ConfigMaps
- Secrets
- Environment variables
- Volume mounts

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

## Next Deep Dive

**Target:** DNS / Service Discovery (15 min)
**Why:** Essential for Prometheus service discovery, understanding how pods resolve service names
**Prerequisites:** Service networking ✅, Ingress ✅

**After that:** Workload Resources - Deployments (20 min)
**Why:** Need to understand how to actually deploy and manage your applications

---

## Retention Scores 📊

**Date: 2025-10-10**

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
