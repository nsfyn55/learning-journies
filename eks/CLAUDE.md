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

### Topics Mastered
The owner has deep understanding of:
- Kubernetes architecture (control plane and data plane)
- Pod networking with AWS VPC CNI (ENIs, veth pairs, network namespaces)
- Service networking (ClusterIP, iptables, Endpoints)
- Ingress controllers (AWS Load Balancer Controller, ALB, IP vs Instance mode)

### Critical Mental Models
1. **Watch Loop Pattern** - Controllers continuously watch API server and reconcile state
2. **Pod IPs are ephemeral** - Services provide stable ClusterIPs for abstraction
3. **AWS VPC CNI uses real VPC IPs** - Secondary IPs on ENIs form warm pool for pods
4. **iptables rewrites ClusterIP → Pod IPs** - Virtual ClusterIPs never travel on network

### Next Learning Priorities
Per `learning-plan.md:129-135`:
1. DNS / Service Discovery (essential for Prometheus)
2. Workload Resources (Deployments, StatefulSets for databases)
3. Configuration (ConfigMaps, Secrets)
4. Storage (Persistent Volumes for databases)
5. Observability specifics (Prometheus architecture, Operator, service discovery)

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
- I would like you to track my learning progress and keep my md files up to date