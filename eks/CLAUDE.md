# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **learning repository** for EKS/Kubernetes concepts with the goal of building an observability platform using Prometheus and AWS-hosted Grafana. The repository tracks learning progress through documentation rather than containing production code.

## Project Goals (In Order)

1. ✅ Learn EKS and Kubernetes fundamentals
2. ✅ Write Terraform to build EKS cluster
3. 🔄 Build web app/db stack in cluster (IN PROGRESS - learning phase)
4. ⏳ Build observability prototype (Prometheus + Grafana)
5. ⏳ Build out SLOs

## Repository Structure

- `learning-plan.md` - Comprehensive learning plan with topics covered, retention scores, and next steps
- `learning-progress.md` - High-level tracking of completed topics and key mental models
- `eks.md` - Initial project requirements and goals

## Key Context for Future Development

### Topics Mastered (7/15 Core Fundamentals - 47%)
The owner has deep understanding of:
- Kubernetes architecture (control plane and data plane)
- Pod networking with AWS VPC CNI (ENIs, veth pairs, network namespaces)
- Service networking (ClusterIP, iptables, Endpoints)
- Ingress controllers (AWS Load Balancer Controller, ALB, IP vs Instance mode)
- DNS / Service Discovery (CoreDNS, Prometheus discovery, watch mechanisms)
- Workload Resources (Deployments, StatefulSets, resource limits, cgroups)
- ConfigMaps and Secrets (base64 encoding, AWS Secrets Manager, CSI Driver, Operators)

### Critical Mental Models
1. **Watch Loop Pattern** - Controllers continuously watch API server and reconcile state
2. **Pod IPs are ephemeral** - Services provide stable ClusterIPs for abstraction
3. **AWS VPC CNI uses real VPC IPs** - Secondary IPs on ENIs form warm pool for pods
4. **iptables rewrites ClusterIP → Pod IPs** - Virtual ClusterIPs never travel on network
5. **StatefulSets = Stable DNS names, NOT stable IPs** - DNS persists across node moves
6. **Deployments use ReplicaSets for versioning** - Each update creates new ReplicaSet
7. **Resource limits enforced by kernel (cgroups)** - CPU throttling, memory OOM kill
8. **base64 = encoding, NOT encryption** - Trivially decodable, use etcd encryption + RBAC + external secret managers
9. **CSI Driver bypasses etcd** - Most secure for secrets (AWS → pod directly, never stored in cluster)

### Current Learning Plan
**Pace:** 2 topics per day
**Timeline:** 4 days to complete core K8s fundamentals (8 topics remaining)

**Next priorities (Days 1-4):**
1. Health Checks and Lifecycle
2. Storage Fundamentals (PV, PVC, StorageClasses)
3. RBAC and ServiceAccounts
4. Pod Security
5. Network Policies
6. Advanced Scheduling
7. Resource Management
8. Disruptions and Availability

**After Day 4:** Begin observability platform implementation (Prometheus + Grafana)

## Development Commands

**Note:** No build, test, or deployment commands exist yet. When Terraform code is added:
- Standard Terraform workflow: `terraform init`, `terraform plan`, `terraform apply`
- When Kubernetes manifests are added: `kubectl apply -f <file>`

## Architecture Notes

### Networking Decisions
- **Ingress Mode**: IP mode preferred over Instance mode (direct pod IP targeting, lower latency)
- **ALB Architecture**: ALB in public subnets, pods in private subnets
- **Service Discovery**: kube-proxy watches both Services and Endpoints to maintain iptables rules

### AWS-Specific Considerations
- AWS VPC CNI assigns real VPC IPs to pods (not overlay network)
- Cloud Controller Manager handles AWS integrations (LoadBalancers, node lifecycle)
- AWS Load Balancer Controller watches Endpoints to register pod IPs in ALB Target Groups

## Communication Preferences

### Learning Sessions
- Track learning progress and keep markdown files up to date
- **Always restate remaining questions at the end of explanations** - helps maintain context and flow during learning sessions
- When explaining complex topics with multiple follow-up questions, conclude with a clear list of what's still pending