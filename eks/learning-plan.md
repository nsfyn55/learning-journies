# EKS/Kubernetes Learning Plan

## Progress: 12/15 Core Topics Complete (80%)

**Remaining:** 3 topics (~1.5 days)
**Next:** Advanced Scheduling, Resource Management, Disruptions & Availability

---

## Core Fundamentals Roadmap

### ✅ Completed (12/15)
1. Kubernetes Architecture (6/10)
2. Pod Networking (9/10) ⭐
3. Service Networking (8/10)
4. Ingress Controllers (7/10)
5. DNS / Service Discovery (7/10)
6. Workload Resources (7/10)
7. ConfigMaps and Secrets (5/10)
8. Health Checks and Lifecycle (6/10)
9. Storage Fundamentals (8/10)
10. RBAC and ServiceAccounts (6/10)
11. Network Policies (5/10)
12. Pod Security (4/10)

### 🔄 Remaining (3/15)
13. Advanced Scheduling (7/10) - Affinity, taints/tolerations, node selectors
14. Resource Management (5/10) - ResourceQuotas, LimitRanges, QoS
15. Disruptions and Availability (6/10) - PodDisruptionBudgets, HA patterns

---

## Topics Summary

### Kubernetes Architecture
- Control: API Server (hub), etcd, Scheduler, Controllers, Cloud Controller Manager
- Data: kubelet, Runtime, kube-proxy, CNI, Pods
- Watch loop reconciliation pattern

### Pod Networking
- ENI: primary (host) + secondaries (pod warm pool)
- veth pairs connect namespaces
- Routing rules (not iptables) direct traffic

### Service Networking
- Virtual ClusterIPs, iptables DNAT to Pod IPs
- kube-proxy watches Services + Endpoints
- Every node has rules for ALL Services

### Ingress Controllers
- ALB Controller: watches Endpoints, IP mode (direct to pods)
- Layer 7 routing (path/host-based)

### DNS / Service Discovery
- CoreDNS returns ClusterIPs
- Prometheus uses kubernetes_sd_configs (needs pod IPs)
- Watch = HTTP streaming, NOT polling

### Workload Resources
- Deployment → ReplicaSet → Pods (versioning)
- StatefulSets: stable names, headless Service, ordered ops
- Requests (Scheduler) vs Limits (cgroups)

### ConfigMaps and Secrets
- base64 for binary data, NOT security
- CSI Driver (never etcd) > External Secrets > Native
- File mounts auto-update, env vars need restart

### Health Checks and Lifecycle
- Liveness: only fail if restart fixes
- Readiness: check all dependencies
- preStop sleep 15 prevents race

### Storage Fundamentals
- Dynamic: PVC → StorageClass → CSI → EBS + PV
- StatefulSet volumeClaimTemplates: deterministic naming
- StatefulSet deletion preserves PVCs
- RWO (EBS), RWX (EFS)

### RBAC and ServiceAccounts
- Role = apiGroups + resources + verbs
- RoleBinding location = scope
- IRSA = K8s RBAC + AWS IAM

### Network Policies
- Calico: Felix manages ipsets, writes iptables
- Pre-computed rules (no API queries)
- Multiple policies OR'd, conditions AND'd

### Pod Security
- fsGroup: enables non-root volume access
- Standards: Privileged/Baseline/Restricted
- Pod Security Admission: checks at Pod creation (async)

---

## Retention Scores (Latest)

| Topic | Score | Date |
|-------|-------|------|
| Architecture | 🟡 75% | 2025-10-11 |
| Pod Networking | 🟢 95% | 2025-10-11 |
| Service Networking | 🟢 92% | 2025-10-11 |
| Ingress | 🟢 95% | 2025-10-11 |
| DNS | 🟢 98% | 2025-10-11 |
| Workload Resources | - | - |
| ConfigMaps/Secrets | 🟢 90% | 2025-10-11 |
| Health Checks | 🟢 94% | 2025-10-11 |
| Storage | - | - |
| RBAC | - | - |
| Network Policies | 🟢 92% | 2025-10-13 |
| Pod Security | 🟢 88% | 2025-10-14 |

**Average:** 🟢 91% (Strong)

---

## Optional Topics for Deep Expertise

### Advanced Workloads
- DaemonSets, Jobs, CronJobs

### Autoscaling
- HPA, VPA, Cluster Autoscaler

### Service Types
- NodePort, LoadBalancer, ExternalName

### Advanced Networking
- IPVS Mode, Multi-cluster

### Extension Points
- CRDs, Operators, Admission Controllers

### API Internals
- Kubernetes API Deep Dive

---

## Milestones

**Milestone 1 (~1.5 days):** Complete K8s Core Fundamentals
- Can deploy and manage any application on Kubernetes
- Ready to build observability platform

**After Milestone 1:** Begin Prometheus/Grafana/SLO implementation
