# EKS & Kubernetes Learning Journey

A structured learning repository focused on mastering EKS, Kubernetes, and building an observability platform with Prometheus and AWS-hosted Grafana.

## 🎯 Project Goals

1. ✅ **Learn EKS and Kubernetes fundamentals**
2. ✅ **Write Terraform to build EKS cluster**
3. 🔄 **Build web app/db stack in cluster** (IN PROGRESS)
4. ⏳ **Build observability prototype** (Prometheus + Grafana)
5. ⏳ **Build out SLOs**

## 📚 Documentation

- **[learning-plan.md](./learning-plan.md)** - Comprehensive learning plan with deep dive notes, retention scores, and next topics
- **[learning-progress.md](./learning-progress.md)** - High-level progress tracking and key mental models
- **[eks.md](./eks.md)** - Initial project requirements and motivation

## 🧠 Topics Mastered

### Kubernetes Architecture
- Control Plane: API Server, etcd, Scheduler, Controller Manager, Cloud Controller Manager
- Data Plane: kubelet, Container Runtime, kube-proxy, CNI Plugin
- Watch loop pattern and reconciliation model

### Pod Networking (Deep Dive)
- AWS VPC CNI specifics (real VPC IPs, ENIs, warm pool)
- Network namespaces and veth pairs
- Complete pod startup flow

### Service Networking (Deep Dive)
- ClusterIP and service abstraction
- Endpoint Controller and kube-proxy iptables mode
- Pod lifecycle and IP management

### Ingress and Ingress Controllers (Deep Dive)
- AWS Load Balancer Controller
- IP mode vs Instance mode (chose IP mode)
- Layer 7 routing with ALB
- Security group and subnet architecture

## 🎓 Key Mental Models

1. **Everything is a Watch Loop** - Controllers watch API server, compare desired vs actual state, reconcile
2. **Pod IPs are Ephemeral** - Services provide stable ClusterIPs for abstraction
3. **AWS VPC CNI Uses Real VPC IPs** - No overlay network, secondary ENI IPs form pod warm pool
4. **iptables Rewrites ClusterIP → Pod IPs** - Virtual ClusterIPs never travel on network
5. **Distributed by Design** - Each node independently handles its pods, all nodes have complete Service/Endpoint info

## 📋 Next Learning Priorities

1. **DNS / Service Discovery** (15 min) - Essential for Prometheus
2. **Workload Resources** (20 min) - Deployments, ReplicaSets, StatefulSets
3. **Configuration & Secrets** - ConfigMaps, Secrets, environment variables
4. **Storage** - Persistent Volumes, Storage Classes
5. **Observability Specifics** - Prometheus architecture, Operator, service discovery

## 🏗️ Architecture Decisions

- **Ingress**: IP mode (direct pod IP targeting for lower latency)
- **ALB Setup**: Public subnets for ALB, private subnets for pods
- **Networking**: AWS VPC CNI with real VPC IPs
- **Service Discovery**: kube-proxy watches Services and Endpoints for iptables rules

## 📊 Learning Retention Scores

As of 2025-10-10:
- Control Plane Architecture: 🟡 Good (75%)
- Data Plane Architecture: 🟢 Strong (85%)
- Service Networking: 🟢 Strong (92%)
- Pod Networking: 🟢 Strong (95%)
- Ingress Controllers: 🟢 Strong (95%)

**Scoring System:**
- 🟢 Strong (90-100%): Ready to explain to colleagues
- 🟡 Good (70-89%): Solid understanding with minor gaps
- 🟠 Fair (50-69%): Basics understood, needs review
- 🔴 Needs Review (<50%): Significant gaps, requires re-study

## 🚀 Getting Started

This repository currently contains learning documentation. As the project progresses:

1. **Terraform Phase**: Add EKS cluster infrastructure code
2. **Application Phase**: Add Kubernetes manifests for web app/db stack
3. **Observability Phase**: Add Prometheus and Grafana deployment configurations
4. **SLO Phase**: Define and implement Service Level Objectives

## 📝 Learning Strategy

- **Deep Dive** = 30+ min focused study with hands-on practice
- **Surface Level** = Basic understanding of what it is and when to use it
- **Retention Exercise** = Explain concept back without notes, get feedback
