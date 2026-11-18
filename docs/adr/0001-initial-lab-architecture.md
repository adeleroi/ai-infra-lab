# ADR 0001 — Initial Home Lab Architecture

**Status:** Accepted  
**Date:** 2026-11-18

## Context

The AI Infrastructure Lab is a learning environment for exploring Linux, networking, distributed systems, and eventually AI inference infrastructure.

The initial environment must support experiments involving process isolation, networking, resource constraints, request routing, and failure recovery.

The available hardware is a MacBook Pro M3 Pro with 36 GB RAM. The architecture should use existing hardware, remain relatively simple to operate, and allow future expansion.

Two approaches were considered:

1. A single Linux VM running multiple containers.
2. Multiple Linux VMs, each capable of running containerized services.

A single VM with containers would be sufficient for many networking, process isolation, and application-level failure experiments. Linux namespaces provide isolation for resources such as networking and mounts, while cgroups provide resource accounting and control.

However, all containers within the same VM share one Linux kernel. This limits experiments involving independent kernel failures, operating-system reboots, and node-level configuration.

## Decision

The initial environment will use **two Linux ARM64 virtual machines** running on the MacBook Pro.

Each VM will initially receive:

- 2 virtual CPUs
- 4 GB RAM

The VMs will communicate through a virtual network.

The planned service topology is:

- **VM 1:** Go inference gateway and simulated Worker A.
- **VM 2:** Simulated Worker B.

Services will be introduced progressively and eventually run in containers.

This topology was chosen to provide separate guest operating systems while keeping hardware requirements and operational complexity manageable.

The virtualization software and container runtime will be selected during subsequent milestones.

## Consequences

### Benefits

- Independent guest kernels allow experiments with VM-level failures and reboots.
- Workers can be distributed across separate VM failure domains.
- The gateway can continue routing requests to Worker A if VM 2 becomes unavailable.
- The environment supports learning both virtualization and container isolation.
- No additional physical hardware is required initially.

### Tradeoffs

- Multiple VMs introduce additional memory, storage, and management overhead compared with a single VM.
- VM 1 hosts both the gateway and Worker A, creating a shared failure domain.
- If VM 1 becomes unavailable, the gateway is lost even if Worker B remains operational.
- Running multiple services on one VM can introduce resource contention.

## Limitations

Both VMs share the same physical computer.

A host-level failure, such as a power loss or hardware failure, can make the entire environment unavailable.

The environment also cannot fully reproduce independent physical hardware failures, physical network-device failures, or realistic inter-machine network characteristics.

Virtualized networking and fault injection can approximate some distributed failure scenarios, but they do not replace experiments across independent physical machines.

## Future Considerations

The architecture may evolve to include additional virtual machines, physical Linux nodes, dedicated gateway instances, and GPU compute resources.

These changes will be introduced when specific experiments or learning objectives justify the additional complexity.
