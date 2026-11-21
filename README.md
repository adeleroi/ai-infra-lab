# AI Infrastructure Lab

A hands-on engineering project exploring the systems behind distributed AI inference, from Linux fundamentals to inference serving, distributed coordination, orchestration, and GPU optimization.

## 1. Motivation

The objective of this project is to develop a deep understanding of the foundational mechanisms that modern AI infrastructure relies on.

Rather than starting with production-ready frameworks, the project follows an incremental, first-principles approach: build small systems, investigate their behavior, introduce failures, understand their limitations, and gradually increase complexity.

The long-term goal is to build a distributed inference platform capable of routing requests, managing workers, handling failures, scheduling workloads, scaling resources, and eventually serving real machine learning models.

This is a learning and experimentation platform, not a production inference service.

## 2. Learning Objectives

| Domain                             | Topics                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------------------ |
| Systems fundamentals               | Linux, processes, memory, storage, virtualization, namespaces, cgroups               |
| Networking and distributed systems | TCP/IP, routing, load balancing, deadlines, retries, backpressure, failure detection |
| Infrastructure and orchestration   | Scheduling, service discovery, Kubernetes, autoscaling, observability                |
| AI inference systems               | Transformer execution, prefill, decode, batching, KV caching, quantization           |
| GPU computing                      | GPU architecture, CUDA, memory hierarchy, kernels, performance optimization          |

## 3. Current Status

**Active phase:** Phase 1 — Systems Foundations

**Next milestone:** Milestone 2 — Resource Isolation & Containers

| Milestone | Status | Documentation |
|---|---|---|
| 0 — Hardware & Architecture | Completed | [Architecture decision](docs/adr/0001-initial-lab-architecture.md) |
| 1 — Linux & Virtualization | Completed | [Milestone 1](docs/milestones/001-linux-virtualization.md) |
| 2 — Resource Isolation & Containers | Next | — |
| 3 — Initial Gateway & Workers | Planned | — |

### Latest Progress

- Provisioned two Ubuntu ARM64 virtual machines.
- Explored Linux processes, CPU scheduling, memory management, swap, and logical volume management.
- Investigated IP addressing, routing, and virtual networking.
- Resolved VM connectivity issues by standardizing virtualization and network configurations.
- Experimentally verified bidirectional connectivity, VM-level failure isolation, and network recovery.
- Completed the Milestone 1 technical reflection.

Next: Linux namespaces, cgroups, resource isolation, and containers.

## 4. Architecture

### Current Environment

| Component        | Configuration                  |
| ---------------- | ------------------------------ |
| Physical host    | MacBook Pro M3 Pro             |
| Host memory      | 36 GB                          |
| Virtualization   | UTM / Apple Virtualization     |
| Guest OS         | Ubuntu Server 24.04 LTS ARM64  |
| Virtual machines | 2                              |
| Resources per VM | 2 vCPUs, 4 GiB RAM, 25 GB disk |
| Networking       | UTM NAT Network                |

The current environment provides two independently managed guest operating systems connected through a shared virtual network.

### Target Initial Topology

```text
              MacBook Pro (M3 Pro)
                       macOS
                         |
                 Apple Virtualization
                         |
                    NAT Network
                         |
              +----------+----------+
              |                     |
           Linux VM 1            Linux VM 2
           2 vCPU / 4 GiB        2 vCPU / 4 GiB
              |                     |
         +----+----+                |
         |         |                |
      Gateway   Worker A         Worker B
        (Go)   (Simulated)      (Simulated)
```

The gateway and workers are planned components and have not yet been deployed.

They will initially run as containerized services.

### Architectural Constraints

- Both virtual machines depend on the same physical host.
- VM-level isolation does not provide physical high availability.
- CPU, memory, storage, and networking resources remain subject to shared host constraints.
- The initial environment prioritizes understandability and experimentation over production-scale performance.

The architecture will evolve as new requirements emerge.

See [ADR 0001 — Initial Lab Architecture](docs/adr/0001-initial-lab-architecture.md).

## 5. Roadmap

### Phase 1 — Systems Foundations

November–December 2026

- [x] Milestone 0 — Hardware and architecture planning
- [ ] Milestone 1 — Linux and virtualization
  - [x] VM provisioning
  - [x] Linux processes, memory, and storage
  - [x] Networking fundamentals and virtual networking
  - [x] VM failure isolation
  - [x] Recovery verification and milestone review
- [ ] Milestone 2 — Resource isolation and containers
- [ ] Milestone 3 — Initial gateway and simulated workers

### Phase 2 — Inference Gateway

Q1 2027

- Implement a Go-based inference gateway.
- Introduce worker registration and health checks.
- Implement request routing and load balancing.
- Explore deadlines, timeouts, and bounded retries.
- Implement backpressure and overload protection.
- Benchmark latency, throughput, and failure behavior.

### Phase 3 — Distributed Coordination

Q2 2027

- Explore distributed state and coordination.
- Implement worker scheduling experiments.
- Investigate service discovery and failure detection.
- Expand toward a physical multi-node cluster.
- Study capacity planning and resource allocation.

### Phase 4 — Orchestration

Q3 2027

- Deploy services using Kubernetes.
- Explore controllers and reconciliation loops.
- Implement autoscaling experiments.
- Study rolling deployments, canaries, and rollbacks.
- Introduce metrics, logs, and distributed tracing.

### Phase 5 — Real AI Inference

Q4 2027

- Serve real models using vLLM or SGLang.
- Understand transformer inference execution.
- Investigate prefill and decode.
- Explore continuous batching and KV caching.
- Benchmark inference throughput and latency.
- Investigate quantization and memory constraints.

### Phase 6 — GPU Internals & Optimization

2028 and beyond

- Study GPU architecture and execution models.
- Learn CUDA programming.
- Explore GPU memory hierarchy and synchronization.
- Implement and benchmark small GPU kernels.
- Investigate inference-specific optimizations.

The roadmap is flexible. Progression depends on practical understanding and completed learning objectives rather than fixed deadlines.

## 6. Engineering Approach

Each milestone follows an iterative learning cycle:

1. **Understand:** Develop a working understanding of the underlying mechanisms.
2. **Design:** Identify assumptions, constraints, alternatives, and tradeoffs.
3. **Implement:** Build a small working solution independently.
4. **Experiment:** Observe behavior, introduce failures, and measure outcomes.
5. **Reflect:** Document mistakes, discoveries, limitations, and lessons learned.
6. **Review:** Validate the technical understanding before progressing.

The project follows a spiral learning approach: establish foundational knowledge, apply it in practice, and revisit individual mechanisms in greater depth when the system requires it.

A milestone is complete only when its required experiments and technical reflection have been reviewed.

## 7. Documentation

The repository is the durable source of truth for the project's architecture, progress, experiments, and learning history.

| Directory           | Purpose                                                                              |
| ------------------- | ------------------------------------------------------------------------------------ |
| `docs/adr/`         | Architectural decisions, alternatives, rationale, and consequences                   |
| `docs/experiments/` | Reproducible procedures, measurements, observations, and limitations                 |
| `docs/milestones/`  | Learning objectives, technical understanding, progress, and continuation checkpoints |

Each milestone document records:

- Objectives and completion criteria.
- Environment and relevant configuration.
- Concepts explored.
- Experiments and verified observations.
- Technical decisions and corrections.
- Personal technical reflection.
- Open questions and deferred investigations.
- A continuation checkpoint for future work.

Documentation distinguishes between verified observations, technical hypotheses, and planned functionality.

## 8. Resuming Development

To resume the project after a break or in a new working session:

1. Read the current status and roadmap in this README.
2. Read the active milestone document.
3. Review relevant architectural decisions and experiment logs.
4. Continue from the milestone's continuation checkpoint.

Previously explored concepts do not need to be repeated unless they are required by the next implementation or experiment.

## Guiding Principle

**Build to understand, not simply to make things work.**

The objective is to develop a deeper understanding of the mechanisms, constraints, and tradeoffs underlying reliable distributed infrastructure.
