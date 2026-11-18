# AI Infrastructure Lab

A hands-on engineering project to explore the systems behind distributed AI inference, from Linux and networking fundamentals to inference serving, orchestration, and GPU optimization.

## Overview

The goal of this project is to understand how the infrastructure supporting AI inference works, by building progressively more capable systems from first principles.

Rather than starting with a production-ready framework, I'll begin with small, understandable components, investigate their behavior under failure, and gradually introduce more sophisticated infrastructure.

The long-term objective is to build a small distributed inference platform capable of routing requests, managing workers, handling failures, scaling workloads, and eventually serving real machine learning models.

This is a learning project, not a production inference platform.

## Learning Objectives

The project focuses on five areas:

1. **Linux & Systems Fundamentals** — Processes, memory, filesystems, virtualization, namespaces, cgroups, and containers.
2. **Networking & Distributed Systems** — TCP/IP, routing, load balancing, timeouts, retries, backpressure, coordination, and failure recovery.
3. **Infrastructure & Orchestration** — Deployment, service discovery, Kubernetes, scheduling, autoscaling, and observability.
4. **AI Inference Systems** — Transformer execution, prefill and decode, batching, KV caching, model serving, and quantization.
5. **GPU Computing & Optimization** — GPU architecture, CUDA, memory hierarchy, kernel execution, and performance optimization.

## Architecture

The initial environment runs on a MacBook Pro M3 Pro with 36 GB RAM.

Two Linux ARM64 virtual machines will provide separate operating-system environments for experimenting with networking, resource isolation, and node failures.

### Initial topology

```text
                 MacBook Pro (M3 Pro)
                      macOS
                        |
                 Virtualization
                        |
              Private Virtual Network
                        |
             +----------+----------+
             |                     |
        Linux VM 1             Linux VM 2
        2 vCPU / 4 GB          2 vCPU / 4 GB
             |                     |
        +----+----+                |
        |         |                |
     Gateway   Worker A         Worker B
       (Go)   (Simulated)      (Simulated)
```

The gateway and workers will initially run as containerized services.

This topology allows experiments with worker failures, request routing, networking, and resource constraints.

All virtual machines share one physical host, so the environment does not provide physical high availability.

The architecture will evolve as the project progresses.

## Roadmap

### Phase 1 — Systems Foundations

**December 2026**

- [x] Milestone 0: Hardware and architecture planning
- [ ] Linux virtualization and VM provisioning
- [ ] Linux processes, memory, and system administration
- [ ] Networking fundamentals and virtual networking
- [ ] Namespaces, cgroups, and containers
- [ ] Initial gateway and simulated worker experiments

### Phase 2 — Inference Gateway

**Q1 2027**

- [ ] Implement a Go-based inference gateway
- [ ] Introduce worker registration and health checks
- [ ] Implement request routing and load balancing
- [ ] Explore deadlines, timeouts, and retries
- [ ] Implement backpressure and overload protection
- [ ] Benchmark latency, throughput, and failure behavior

### Phase 3 — Distributed Coordination

**Q2 2027**

- [ ] Explore distributed state and coordination
- [ ] Implement worker scheduling experiments
- [ ] Investigate service discovery and failure detection
- [ ] Expand toward a physical multi-node cluster
- [ ] Study capacity planning and resource allocation

### Phase 4 — Orchestration

**Q3 2027**

- [ ] Deploy services using Kubernetes
- [ ] Explore controllers and reconciliation loops
- [ ] Implement autoscaling experiments
- [ ] Study rolling deployments, canaries, and rollbacks
- [ ] Introduce comprehensive metrics, logs, and tracing

### Phase 5 — Real AI Inference

**Q4 2027**

- [ ] Serve real models using vLLM or SGLang
- [ ] Understand transformer inference execution
- [ ] Investigate prefill and decode
- [ ] Explore continuous batching and KV caching
- [ ] Benchmark inference throughput and latency
- [ ] Investigate quantization and memory constraints

### Phase 6 — GPU Internals & Optimization

**2028 and beyond**

- [ ] Study GPU architecture and execution models
- [ ] Learn CUDA programming
- [ ] Explore GPU memory hierarchy and synchronization
- [ ] Implement and benchmark small GPU kernels
- [ ] Investigate inference-specific performance optimizations

The roadmap is intentionally flexible. Progression depends on demonstrated understanding, not fixed deadlines.

## Engineering Approach

Each milestone follows the same process:

1. **Understand:** Study the fundamental concepts before selecting abstractions.
2. **Design:** Define the problem, assumptions, constraints, and tradeoffs.
3. **Implement:** Build a small working system independently.
4. **Experiment:** Introduce failures, observe behavior, and measure results.
5. **Reflect:** Write a short technical report documenting design decisions, limitations, mistakes, and lessons learned.

A milestone is considered complete only after its implementation and learning objectives have been reviewed.

## Documentation

Project documentation will include:

- **Architecture Decision Records (ADRs):** Why significant architectural decisions were made.
- **Experiment logs:** Hypotheses, commands, observations, and conclusions.
- **Milestone reflections:** Technical explanations of what was learned.

The repository will evolve alongside the implementation rather than starting with a large predefined structure.

## Current Status

**Milestone 0 — Hardware & Architecture: Completed**

Initial hardware and topology selected. Next: Linux virtualization, provisioning, and understanding the guest operating-system environment.

## Guiding Principle

Build to understand, not simply to make things work.

The objective is to develop a deeper understanding of the mechanisms, constraints, and tradeoffs underlying reliable distributed infrastructure.