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

---

## Topics In Progress 🔄

### Service Types
- ClusterIP (covered basics)
- NodePort (not covered)
- LoadBalancer (not covered)

---

## Topics Not Yet Covered ⏳

### Critical for Observability Platform
1. **Ingress and Ingress Controllers** (HIGH PRIORITY)
   - Path/host-based HTTP routing
   - AWS Load Balancer Controller
   - Exposing Grafana to users
   - Single LB for multiple services

2. **DNS / Service Discovery** (HIGH PRIORITY)
   - CoreDNS
   - How pods resolve service names
   - Prometheus service discovery

3. **Service Types Deep Dive** (MEDIUM PRIORITY)
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
- Persistent Volumes and Storage
- StatefulSets for databases
- ConfigMaps and Secrets
- RBAC and security

---

## Next Steps

**Immediate:** Learn Ingress and Ingress Controllers (30 min)
- Most practical for exposing Grafana
- Builds on Service networking knowledge
- Required for real-world deployments

**Then:** DNS basics (15 min)
- Essential for Prometheus service discovery
- Understanding how services are discovered

**After:** Complete service types (10 min)
- LoadBalancer for external access
- Understanding when to use each type

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

---

## Questions to Revisit Later

- When to use IPVS vs iptables mode?
- How does Prometheus actually discover services?
- Best practices for Network Policies?
- How to size node pools for observability workloads?
- What metrics should we collect from the cluster itself?
