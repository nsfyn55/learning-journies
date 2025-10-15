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

### Topics Mastered (13/15 Core Fundamentals - 87%)
The owner has deep understanding of:
- Kubernetes architecture (control plane and data plane)
- Pod networking with AWS VPC CNI (ENIs, veth pairs, network namespaces)
- Service networking (ClusterIP, iptables, Endpoints)
- Ingress controllers (AWS Load Balancer Controller, ALB, IP vs Instance mode)
- DNS / Service Discovery (CoreDNS, Prometheus discovery, watch mechanisms)
- Workload Resources (Deployments, StatefulSets, resource limits, cgroups)
- ConfigMaps and Secrets (base64 encoding, AWS Secrets Manager, CSI Driver, Operators)
- Health Checks and Lifecycle (liveness, readiness, graceful shutdown)
- Storage Fundamentals (PV, PVC, StorageClasses, dynamic provisioning)
- RBAC and ServiceAccounts (roles, bindings, IRSA)
- Network Policies (Calico, ipsets, policy logic)
- Pod Security (SecurityContext, fsGroup, Pod Security Standards)
- Advanced Scheduling (taints/tolerations, affinity, anti-affinity)

### Critical Mental Models
See learning-progress.md for complete list (19 mental models). Key highlights:
- **Watch Loop Pattern** - Controllers continuously watch API server and reconcile state
- **Pod IPs are ephemeral** - Services provide stable ClusterIPs for abstraction
- **Taints are defensive** - Repel ALL pods unless they tolerate
- **Scheduling constraints are AND'd** - All must be satisfied simultaneously
- **topologyKey defines "near"** - hostname = same node, zone = same AZ

### Current Learning Plan
**Progress:** 13/15 topics complete (87%)
**Remaining:** 2 topics (~1 day)
**Average Retention:** 92%

**Next priorities:**
1. Resource Management (ResourceQuotas, LimitRanges, QoS)
2. Disruptions and Availability (PodDisruptionBudgets, HA patterns)

**After completion:** Begin observability platform implementation (Prometheus + Grafana)

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

### Learning Sessions - EFFECTIVE METHODOLOGY (Use This!)
The owner learns best with this proven approach:

1. **Small Topics**: Break down each major topic into digestible sub-topics
2. **One at a Time**: Present ONE concept, wait for questions/clarification, then move to next
3. **Immediate Reinforcement**: After explaining each sub-topic, ask if there are questions
4. **Quiz Before Moving On**: Test understanding with scenario-based questions before marking topic complete
5. **Allow Self-Explanation**: Let the owner explain concepts back to confirm understanding

**DO NOT:**
- Dump all sub-topics at once (overwhelming cognitive load)
- Move to next concept without confirmation of understanding
- Skip the quiz/retention check phase

**Example flow:**
1. Explain Taints → answer questions → confirm understanding
2. Explain Tolerations → answer questions → confirm understanding
3. Explain NodeSelectors → answer questions → confirm understanding
4. Quiz on all three → grade answers → update learning docs

### Documentation
- Track learning progress and keep markdown files up to date after each topic completion
- Update retention scores in both learning-plan.md and learning-progress.md
- Add new mental models to learning-progress.md as they emerge