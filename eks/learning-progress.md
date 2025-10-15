# EKS/Kubernetes Learning Progress

## Overall Goal
Build observability platform for apps running in EKS using Prometheus and AWS hosted Grafana.

## Progress: 12/15 Core Topics (80%)

---

## Key Mental Models

1. **Watch Loop Pattern**: Controllers watch API server, reconcile desired vs actual state
2. **Pod IPs ephemeral, Service ClusterIPs stable**: iptables rewrites ClusterIP → Pod IP
3. **AWS VPC CNI uses real VPC IPs**: Secondary IPs on ENIs = warm pool for pods
4. **StatefulSets = Stable DNS names, NOT stable IPs**: DNS persists across node moves
5. **Deployments use ReplicaSets for versioning**: Each update = new ReplicaSet
6. **Resource limits enforced by kernel (cgroups)**: CPU throttling, memory OOM kill
7. **base64 = encoding, NOT encryption**: Trivially decodable, use etcd encryption + RBAC
8. **CSI Driver bypasses etcd**: Most secure for secrets (AWS → pod, never stored in cluster)
9. **Liveness vs Readiness**: Liveness = "Is container the problem?", Readiness = "Can handle traffic?"
10. **preStop sleep prevents race**: Traffic removal finishes before shutdown
11. **PV cluster-scoped, PVC namespaced**: StatefulSet deletion ≠ PVC deletion (data safety)
12. **RBAC**: ServiceAccount (who) + Role (what+where) + RoleBinding (connection)
13. **Network Policies**: allow-all → deny-all when policy selects pod, Felix manages ipsets
14. **Pod Security**: fsGroup enables non-root volume access, Baseline = default, Restricted = sensitive
15. **Pod Security Admission**: Checks at Pod creation (async), Deployment succeeds, ReplicaSet retries

---

## Topics Completed

### 1. Kubernetes Architecture (6/10)
- Control Plane: API Server (hub), etcd (state), Scheduler, Controllers, Cloud Controller Manager
- Data Plane: kubelet, Container Runtime, kube-proxy, CNI, Pods

### 2. Pod Networking (9/10) ⭐
- ENI: primary IP (host) + secondary IPs (pod warm pool)
- veth pairs: pod namespace ↔ host namespace
- Routing rules (not iptables) direct traffic through veth

### 3. Service Networking (8/10)
- ClusterIP is virtual, iptables rewrites to Pod IPs (DNAT)
- kube-proxy watches Services AND Endpoints
- Every node has iptables for ALL Services

### 4. Ingress Controllers (7/10)
- ALB Controller watches Endpoints, registers pod IPs in Target Groups
- IP mode (direct to pods) vs Instance mode (via NodePort)
- Layer 7 routing: path-based, host-based

### 5. DNS / Service Discovery (7/10)
- CoreDNS watches Services, returns ClusterIPs
- Prometheus uses kubernetes_sd_configs (needs pod IPs, not ClusterIPs)
- Watch = HTTP streaming (chunked transfer), NOT polling

### 6. Workload Resources (7/10)
- Deployment → ReplicaSet → Pods (versioning via ReplicaSets)
- StatefulSets: stable names (postgres-0), headless Service, ordered ops
- Resource requests (Scheduler placement) vs limits (kubelet cgroups)

### 7. ConfigMaps and Secrets (5/10)
- base64 for binary data support, NOT security
- CSI Driver (never in etcd) > External Secrets Operator (syncs to etcd) > Native Secrets
- File mounts auto-update, env vars need pod restart

### 8. Health Checks and Lifecycle (6/10)
- Liveness: only fail if restart fixes it, Readiness: check all dependencies
- Graceful shutdown: two parallel paths (stop traffic + shutdown)
- preStop sleep 15 prevents race condition

### 9. Storage Fundamentals (8/10)
- Dynamic provisioning: PVC → StorageClass → CSI → EBS + PV
- StatefulSet volumeClaimTemplates: deterministic naming (data-postgres-0)
- StatefulSet deletion preserves PVCs (safety)
- RWO (EBS), RWX (EFS only)

### 10. RBAC and ServiceAccounts (6/10)
- Role = apiGroups + resources + verbs
- RoleBinding location determines scope, can reference SA from different namespace
- IRSA = K8s RBAC + AWS IAM (two-layer)

### 11. Network Policies (5/10)
- Calico: Felix DaemonSet manages ipsets, writes iptables
- ipsets = IP-to-label mapping, pre-computed (no API queries at packet time)
- Multiple policies OR'd, conditions within policy AND'd

### 12. Pod Security (4/10)
- SecurityContext: container-level overrides pod-level
- fsGroup: kubelet chowns to root:fsGroup, added as supplementary group to all containers
- Pod Security Standards: Privileged (system) / Baseline (default) / Restricted (sensitive)
- Pod Security Admission: enforces at Pod creation, check ReplicaSet events for errors

---

## Remaining Topics (3)

**Next session:**
- Advanced Scheduling (7/10)
- Resource Management (5/10)

**After that:**
- Disruptions and Availability (6/10)

---

## Retention Scores
- Architecture: 🟡 75%
- Pod Networking: 🟢 95%
- Service Networking: 🟢 92%
- Ingress: 🟢 95%
- DNS: 🟢 98%
- ConfigMaps/Secrets: 🟢 90%
- Health Checks: 🟢 94%
- Network Policies: 🟢 92%
- Pod Security: 🟢 88%
