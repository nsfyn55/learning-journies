# Linux Network Administration - Mental Model Learning Plan

## Overview

**Goal:** Build clear mental models for Linux network administration topics through essay-based question sets.

**Approach:** Mental model presentation → Question set → Essay answers → Grading (0-100%) → Gap discussion

**Total Topics:** 24
**Total Time:** 48-54 hours (10 weeks at 5-6 hours/week)
**Pace:** 2-3 topics per week

---

## Phase 1: Packet Flow Foundation (Week 1)

### Topic 1: Routing Fundamentals - 2-3 hours
**Mental Model:** How packets decide where to go next
**Question Set:** 6 questions covering routing table lookup, default routes, metric selection, multi-homed hosts
**Status:** Not started

### Topic 2: iptables/Netfilter Architecture - 2-3 hours
**Mental Model:** Where and how packets get inspected/modified as they flow through the kernel
**Question Set:** 7 questions covering chains, tables, NAT flow, connection tracking, rule ordering
**Status:** Not started

---

## Phase 2: Virtual Networking (Week 2)

### Topic 3: Bridge Interfaces - 2 hours
**Mental Model:** Software switches - how Linux bridges act like physical switches
**Question Set:** 5 questions covering bridge forwarding, MAC learning, STP, bridge + veth pairs
**Status:** Not started

### Topic 4: VLANs - 1.5 hours
**Mental Model:** 802.1Q tagging and traffic isolation on a single interface
**Question Set:** 5 questions covering tagged/untagged traffic, trunk ports, VLAN routing
**Status:** Not started

### Topic 5: Bonding/Teaming - 1.5 hours
**Mental Model:** Link aggregation for redundancy and throughput
**Question Set:** 4 questions covering modes (active-backup, LACP), failover, load distribution
**Status:** Not started

### Topic 6: TUN/TAP and veth pairs - 2 hours
**Mental Model:** Virtual cables and userspace networking
**Question Set:** 6 questions covering TUN vs TAP, veth pair connectivity, routing between endpoints
**Status:** Not started

---

## Phase 3: Isolation and Namespaces (Week 3)

### Topic 7: Network Namespaces - 2-3 hours
**Mental Model:** Complete network stack isolation
**Question Set:** 7 questions covering namespace creation, veth pair connectivity, routing between namespaces, default namespace behavior
**Status:** Not started

### Topic 8: Container Networking - 2-3 hours
**Mental Model:** How Docker/containers combine namespaces + bridges + veth + iptables
**Question Set:** 6 questions covering docker0 bridge, port publishing, --net=host, inter-container communication
**Status:** Not started

---

## Phase 4: Advanced Routing (Week 4)

### Topic 9: Multiple Routing Tables - 2 hours
**Mental Model:** RPDB (Routing Policy Database) and table lookup order
**Question Set:** 6 questions covering `ip rule`, table selection, local/main/default tables
**Status:** Not started

### Topic 10: Policy-Based Routing - 2-3 hours
**Mental Model:** Routing decisions based on more than just destination IP
**Question Set:** 6 questions covering source-based routing, marking packets, routing based on fwmark, multi-ISP scenarios
**Status:** Not started

---

## Phase 5: Traffic Control (Week 5)

### Topic 11: Traffic Control Fundamentals - 2 hours
**Mental Model:** Where and how packets get queued/shaped
**Question Set:** 5 questions covering qdisc hierarchy, ingress vs egress, default queuing
**Status:** Not started

### Topic 12: Rate Limiting and Shaping - 2-3 hours
**Mental Model:** HTB, TBF, and controlling bandwidth
**Question Set:** 6 questions covering token buckets, classes, rate vs ceil, burst
**Status:** Not started

---

## Phase 6: Deep Inspection (Week 6)

### Topic 13: Connection Tracking (conntrack) - 2 hours
**Mental Model:** How the kernel tracks connection state
**Question Set:** 6 questions covering NEW/ESTABLISHED/RELATED, NAT and conntrack, conntrack table, timeouts
**Status:** Not started

### Topic 14: Advanced tcpdump/BPF - 2 hours
**Mental Model:** Filtering at kernel level for efficient capture
**Question Set:** 5 questions covering BPF syntax, complex filters, capturing specific flows
**Status:** Not started

---

## Phase 7: VPN and Tunneling (Week 7)

### Topic 15: SSH Tunneling - 1.5 hours
**Mental Model:** Port forwarding and SOCKS proxies
**Question Set:** 5 questions covering -L, -R, -D, practical scenarios
**Status:** Not started

### Topic 16: WireGuard - 2 hours
**Mental Model:** Modern VPN with routing table integration
**Question Set:** 5 questions covering interface creation, peer configuration, allowed-ips, routing
**Status:** Not started

### Topic 17: IPsec Concepts - 2 hours
**Mental Model:** ESP, AH, SAs, and the IPsec stack
**Question Set:** 5 questions covering tunnel vs transport mode, IKE, SA establishment
**Status:** Not started

---

## Phase 8: Performance and Kernel Internals (Week 8)

### Topic 18: Socket Buffers and TCP Tuning - 2-3 hours
**Mental Model:** Where packets queue in the networking stack
**Question Set:** 6 questions covering SO_SNDBUF/RCVBUF, rmem/wmem, TCP window scaling, congestion control
**Status:** Not started

### Topic 19: Ring Buffers and NIC Queuing - 2 hours
**Mental Model:** Hardware queues and interrupt handling
**Question Set:** 5 questions covering rx/tx rings, drops, ethtool, RSS/RPS
**Status:** Not started

---

## Phase 9: Modern Packet Processing (Week 9)

### Topic 20: eBPF Fundamentals - 2-3 hours
**Mental Model:** Programmable hooks throughout the kernel networking stack
**Question Set:** 6 questions covering hook points, XDP vs tc-bpf, maps, use cases
**Status:** Not started

### Topic 21: XDP (eXpress Data Path) - 2 hours
**Mental Model:** Earliest possible packet processing for high performance
**Question Set:** 5 questions covering XDP actions (DROP, PASS, TX, REDIRECT), driver vs generic mode, use cases
**Status:** Not started

---

## Phase 10: Services and High Availability (Week 10)

### Topic 22: IPVS (IP Virtual Server) - 2 hours
**Mental Model:** Kernel-level Layer 4 load balancing
**Question Set:** 5 questions covering scheduling algorithms, NAT vs DR mode, connection persistence
**Status:** Not started

### Topic 23: HAProxy Fundamentals - 2 hours
**Mental Model:** Layer 7 load balancing and proxy architecture
**Question Set:** 5 questions covering frontend/backend, ACLs, health checks, balancing algorithms
**Status:** Not started

### Topic 24: Keepalived and VRRP - 1.5 hours
**Mental Model:** IP failover and virtual IPs
**Question Set:** 4 questions covering VRRP protocol, master/backup election, split-brain scenarios
**Status:** Not started

---

## Learning Session Structure

Each topic follows this format:

1. **Read Mental Model** (10-15 min) - Core concepts, key components, mental shortcuts
2. **Answer Questions** (60-90 min) - Brief essay format, explain reasoning
3. **Review Grades** (30-45 min) - 0-100% scoring per question with specific feedback
4. **Discuss Gaps** - Clarify misconceptions and fill holes

---

## Progress Tracking

**Current Topic:** None
**Topics Completed:** 0/24
**Current Phase:** Phase 1
**Hours Invested:** 0

---

## Notes

- Focus is on building mental models, not mastery
- Essay answers reveal reasoning process and gaps
- Real-world scenarios test application
- Mastery comes from applying these mental models in production
