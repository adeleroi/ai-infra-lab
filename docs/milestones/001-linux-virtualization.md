# Milestone 001 — Linux & Virtualization

## 1. Milestone Metadata

| Field | Value |
|---|---|
| Milestone | 001 |
| Phase | Phase 1 — Systems Foundations |
| Status | Completed |
| Period | November 2026 |
| Previous milestone | Hardware & Architecture |
| Next milestone | Resource Isolation & Containers |

## 2. Objective

Establish a working Linux-based virtual infrastructure and develop a foundational understanding of the operating-system and networking mechanisms that will support the distributed inference platform.

The objective is to understand these mechanisms sufficiently to reason about application behavior, resource constraints, communication, and failures.

Detailed kernel internals are outside the scope of this milestone and will be revisited when required.

## 3. Completion Criteria

- [x] Provision two Linux ARM64 virtual machines.
- [x] Understand the relationship between guest and host resources.
- [x] Explore Linux process states and CPU scheduling.
- [x] Inspect Linux memory accounting and swap.
- [x] Understand the relationship between block devices, LVM, and filesystems.
- [x] Understand basic IP addressing, routing, and virtual networking.
- [x] Establish bidirectional VM connectivity.
- [x] Perform a VM shutdown and failure isolation experiment.
- [x] Verify bidirectional connectivity following VM recovery.
- [x] Finalize and approve the technical reflection.

## 4. Lab Environment

### Physical Host

| Resource | Configuration |
|---|---|
| Machine | MacBook Pro |
| Processor | Apple M3 Pro |
| Architecture | ARM64 |
| Memory | 36 GB |
| Host OS | macOS |
| Virtualization software | UTM |
| Virtualization backend | Apple Virtualization |

### Virtual Machines

| Property | VM 1 | VM 2 |
|---|---|---|
| Hostname | ai-lab-vm1 | ai-lab-vm2 |
| Guest OS | Ubuntu Server 24.04 LTS | Ubuntu Server 24.04 LTS |
| Architecture | ARM64 | ARM64 |
| vCPUs | 2 | 2 |
| RAM | 4096 MiB | 4096 MiB |
| Virtual disk | 25 GB | 25 GB |
| Username | labuser | labuser |
| Planned role | Gateway + Worker A | Worker B |

### Network Configuration

| Property | Value |
|---|---|
| Network mode | NAT Network |
| Subnet | 192.168.96.0/24 |
| Default gateway | 192.168.96.1 |
| VM 1 address | 192.168.96.3 |
| VM 2 address | 192.168.96.2 |
| Network interface | enp0s1 |
| Address allocation | DHCP |

IP addresses represent the observed configuration and are not guaranteed to remain unchanged.

### Accessing the Environment

From the macOS terminal:

```bash
ssh labuser@192.168.96.3
ssh labuser@192.168.96.2
```

If addresses change, inspect them from the VM console:

```bash
hostname -I
```

## 5. Concepts Explored

### 5.1 Virtualization

Each virtual machine runs its own Linux kernel and receives virtual hardware resources.

The guest kernel schedules its processes onto virtual CPUs, while the virtualization layer schedules those virtual CPUs onto physical host CPU resources.

Because the guest and host both use ARM64, guest CPU instructions can execute natively with hardware-assisted virtualization rather than requiring full instruction-set emulation.

Memory involves multiple address spaces:

```text
Process virtual address
          |
Guest page table translation
          |
Guest physical address
          |
Second-stage address translation
          |
Host physical address
```

The guest kernel manages process address spaces, while the virtualization machinery controls the mapping between guest physical memory and host-backed physical memory.

Virtual machines provide separate operating-system environments, but their resources ultimately depend on the same physical host.

### 5.2 Linux Processes and CPU Scheduling

Linux processes can exist in several states:

| State | Meaning |
|---|---|
| R | Running or runnable |
| S | Interruptible sleep |
| D | Uninterruptible sleep |
| T | Stopped or traced |
| Z | Zombie; execution has terminated but status has not been collected |

A large process count does not necessarily imply high CPU utilization.

Many processes spend most of their time sleeping or waiting for events.

With two virtual CPUs, multiple CPU-bound processes can make progress through preemptive scheduling. They do not all execute simultaneously.

Guest CPU utilization also does not necessarily reflect the amount of physical CPU execution capacity available from the host.

### 5.3 Memory Management

Commands explored:

```bash
free -h
cat /proc/meminfo
cat /proc/swaps
```

Initial VM 1 observations:

| Metric | Approximate value |
|---|---|
| Configured RAM | 4 GiB |
| Guest-reported total memory | 3.8 GiB |
| Used memory | 328 MiB |
| Free memory | 3.4 GiB |
| Available memory | 3.5 GiB |
| Swap capacity | 2.1 GiB |
| Swap usage | 0 |

Important distinctions:

- Free memory is currently unused physical memory.
- Available memory estimates how much memory can be provided to applications without swapping.
- Some memory occupied by caches can be reclaimed.
- Virtual memory allocations do not necessarily result in immediate physical memory consumption.
- Memory pressure may result in reclaiming pages, swapping, or OOM termination.
- Excessive swapping can degrade performance through thrashing.

These mechanisms will become important when introducing worker resource limits and investigating workload contention.

### 5.4 Storage and Logical Volume Management

Commands explored:

```bash
lsblk
df -h
sudo pvs
sudo vgs
sudo lvs
```

Observed VM 1 storage organization:

```text
Virtual disk: /dev/vda (~25 GB)
    |
    +-- /dev/vda1     EFI partition
    |
    +-- /dev/vda2     Boot partition
    |
    +-- /dev/vda3     LVM physical volume
           |
           +-- ubuntu-vg (~21.95 GiB)
                  |
                  +-- ubuntu-lv (~10.97 GiB)
                  |       |
                  |       +-- Root filesystem (/)
                  |
                  +-- Unallocated VG capacity (~10.97 GiB)
```

The virtual disk size, logical volume size, and filesystem capacity are different abstractions.

Although the virtual disk has approximately 25 GB of capacity, the root filesystem initially uses only a portion of the capacity available through LVM.

The existing free volume-group capacity could be used to extend the logical volume without increasing the virtual disk size.

No storage resizing was performed during this milestone.

### 5.5 Networking Fundamentals

Commands explored:

```bash
ip -br -4 addr
ip route
ip route get 8.8.8.8
ip neigh
sudo ss -lntp
ping
```

Key concepts:

- IP addresses identify network interfaces at the network layer.
- Subnet prefixes identify connected IP networks.
- Routing tables determine the next hop for outgoing packets.
- The default gateway handles destinations without a more specific route.
- ARP resolves IPv4 neighbor addresses to link-layer addresses.
- TCP and UDP sockets expose application-level communication endpoints.
- NAT translates network addresses between network environments.

For an external destination, the VM selects its default gateway.

For a directly connected destination, the VM can communicate with the destination without using its IP default gateway.

## 6. Problems Encountered and Resolutions

### 6.1 Installation Medium During Reboot

Ubuntu reported an error unmounting the installation CD-ROM after installation.

This required handling the attached installation ISO before booting the installed system.

The installation did not need to be repeated solely because of this message.

### 6.2 Different Virtualization Backends

VM 1 was initially created using Apple Virtualization, while VM 2 was created using QEMU.

Although both guests used ARM64 Ubuntu, the available network configuration options differed between the backends.

This demonstrated that identical guest operating systems do not imply identical underlying infrastructure.

### 6.3 VM-to-VM Connectivity Failure

The original network configuration was:

| Property | VM 1 | VM 2 |
|---|---|---|
| Backend | Apple Virtualization | QEMU |
| Network mode | Shared Network | Shared Network |
| IP address | 192.168.64.2 | 192.168.65.2 |
| Subnet | 192.168.64.0/24 | 192.168.65.0/24 |
| Gateway | 192.168.64.1 | 192.168.65.1 |

Both VMs were accessible from the host, but neither could successfully ping the other.

Routing lookups showed that each VM selected its respective default gateway when communicating with the other subnet.

Different subnets alone do not prevent communication; routing and applicable network policies determine whether communication is possible.

The exact packet-drop mechanism was not investigated.

### Resolution

VM 2 was recreated using Apple Virtualization, and both VMs were configured to use UTM's NAT Network mode.

This provided a shared virtual network with direct IP connectivity between the guests.

The resulting configuration was verified experimentally rather than assumed from the configuration alone.

## 7. Experiments and Verified Results

### Experiment A — Bidirectional Connectivity

From VM 1:

```bash
ip route get 192.168.96.2
ping -c 4 -W 2 192.168.96.2
```

From VM 2:

```bash
ip route get 192.168.96.3
ping -c 4 -W 2 192.168.96.3
```

Results:

| Direction | Packets received | Packet loss | Average RTT |
|---|---|---|---|
| VM 1 → VM 2 | 4/4 | 0% | 1.020 ms |
| VM 2 → VM 1 | 4/4 | 0% | 1.174 ms |

The route lookups no longer included a default gateway as the next hop.

Both machines were directly connected IPv4 neighbors.

### Experiment B — VM Failure Isolation

Hypothesis:

Shutting down VM 2 should interrupt communication with that VM without stopping VM 1.

Procedure:

1. Continuously ping VM 2 from VM 1.
2. Gracefully shut down VM 2.
3. Observe the network behavior.
4. Verify VM 1 remains operational.
5. Restart VM 2.
6. Verify recovery.

Commands:

```bash
# VM 1
ping -i 1 192.168.96.2

# VM 2
sudo shutdown -h now

# VM 1, following shutdown
uptime
ip -br -4 addr
```

Observed results:

| Measurement | Result |
|---|---|
| Packets transmitted | 223 |
| Successful replies | 19 |
| Reported errors | 183 |
| Packet loss | 91.48% |
| Average successful RTT | 0.983 ms |
| Last successful reply | Sequence 19 |
| First explicit unreachable error observed | Sequence 40 |

VM 1 remained operational and retained its IP address and network interface.

After restarting, VM 2 retained its previous IPv4 address and accepted SSH connections from the host.

### Recovery Verification

Following VM 2's restart, bidirectional connectivity was successfully restored.

| Direction | Packets received | Packet loss | Average RTT |
|---|---|---|---|
| VM 1 → VM 2 | 4/4 | 0% | 1.460 ms |
| VM 2 → VM 1 | 4/4 | 0% | 1.244 ms |

Both VMs retained their previous IP addresses.

The experiment confirms that an individual VM can be shut down and restarted without stopping the other VM, and that network connectivity can be restored following recovery without manual network reconfiguration.

This validates VM-level failure isolation and network recovery within the current lab environment. It does not establish application-level fault tolerance or physical high availability.

### Failure Detection Observation

There was a period without replies before VM 1 started reporting `Destination Host Unreachable`.

A possible explanation is the interaction between cached neighbor information and subsequent ARP reachability checks.

This explanation was not verified through packet capture or neighbor-table monitoring.

The important conclusion is that failure occurrence and failure detection are separate events.

This distinction motivates application-level health checks, deadlines, and failure-handling policies.

### Experimental Limitations

- The shutdown was graceful rather than an abrupt crash.
- ICMP reachability does not establish application health.
- Both VMs share the same physical host.
- No gateway or worker service was running during the experiment.
- Failure-detection timing was observed but not precisely instrumented.

## 8. Architectural Decisions and Implications

### Independent Guest Operating Systems

Two VMs provide separate Linux kernels and independently managed operating-system environments.

This allows experimentation with node availability, resource constraints, networking, and failure isolation.

### Shared Virtual Network

A shared NAT Network was selected to simplify initial communication between the VMs.

Inter-subnet routing is deferred until it becomes relevant to the project.

### Shared Failure Domains

The two VMs remain dependent on the same host, virtualization environment, and underlying physical resources.

The current architecture therefore supports VM-level failure experiments but does not provide physical high availability.

### Implications for the Future Gateway

The gateway will need to consider:

- Worker availability and health.
- Delayed or uncertain failure detection.
- Remaining worker capacity.
- Request deadlines.
- Bounded retries.
- Backpressure and overload protection.

These are future implementation requirements, not capabilities already present in the lab.

## 9. Personal Technical Reflection

### Architecture

We chose two Linux VMs because they provide separate guest kernels and are sufficient for our initial experiments involving node availability, failure detection, request routing, and server-to-server communication.

Although the VMs are independently managed, they share failure domains such as physical host failure and contention for underlying hardware resources.

### Virtualization

The guest kernel schedules processes onto virtual CPUs, while the hypervisor schedules those virtual CPUs onto the host's physical CPU resources.

A process may be running, runnable, sleeping, waiting uninterruptibly, stopped, or terminated without having its exit status collected.

These states help explain whether a process is executing, waiting, or no longer able to execute. A large number of processes does not necessarily imply high CPU utilization.

Memory also involves multiple layers. A process operates in its own virtual address space, the guest kernel manages mappings to guest physical memory, and the virtualization machinery maps guest physical memory onto host-backed memory.

### Networking

Our original VMs used different virtualization backends and virtual network configurations.

They were assigned addresses on different subnets. Consequently, communication between them required routing through their respective gateways.

However, neither VM received replies from the other.

By standardizing the virtualization backend and switching both VMs to NAT Network, we established a shared virtual subnet and verified direct communication between them.

This simplified the network architecture and removed the need for inter-subnet routing in our initial environment.

### Failure Handling

When a worker disappears, the gateway must detect the failure through mechanisms such as health checks, timeouts, connection errors, or other transport-level signals.

It should stop routing new requests to the unavailable worker and select healthy workers with sufficient capacity.

In-flight requests are more difficult because the gateway may not know whether a worker failed before processing the request or after completing it but before returning a response.

Retrying on another healthy worker may be appropriate, but retries can increase latency, waste compute resources, repeat side effects, or produce different generated responses.

A reliable gateway must consider whether retrying is safe, whether capacity remains available, and whether the original request deadline can still be respected.

### Lessons Learned

What surprised me most was the distinction between free memory, reclaimable memory, and actively used memory.

Before this milestone, I did not have a clear understanding of how memory occupied by caches could remain available for future allocations.

I would also like to investigate logical volume management, filesystems, and logical volume extension in greater depth, since storage administration is one of the areas I am less familiar with.

The networking experiment also demonstrated the importance of validating infrastructure assumptions through observed behavior rather than relying solely on configuration.

## 10. Open Questions and Deferred Investigations

These topics are intentionally deferred rather than treated as milestone blockers.

| Topic | Future context |
|---|---|
| Linux memory reclaim and page faults | Memory pressure and container experiments |
| OOM behavior and swapping under load | Container memory limits |
| Detailed CPU scheduling algorithms | Worker contention and performance experiments |
| LVM and filesystem expansion | Storage administration requirements |
| ARP and neighbor-cache state transitions | Network failure diagnostics |
| TCP connection behavior during failures | Gateway transport and timeout experiments |
| Inter-subnet routing | Distributed and physical cluster networking |
| Stable service addressing | Service discovery and orchestration |

## 11. Continuation Checkpoint

### Completion Status

- Status: Completed
- Completion date: November 21, 2026
- All required experiments completed.
- Technical reflection reviewed.
- Bidirectional network connectivity and recovery verified.

### Current Infrastructure State

- Two Ubuntu ARM64 VMs running through UTM and Apple Virtualization.
- Shared NAT Network: 192.168.96.0/24.
- VM 1: 192.168.96.3.
- VM 2: 192.168.96.2.
- SSH access and bidirectional connectivity verified.
- VM-level failure isolation and recovery demonstrated.
- No containers, gateway, or worker services deployed yet.

### Next Milestone

Milestone 002 — Resource Isolation & Containers

Begin with the distinction between machine-level isolation and process-level isolation.

Learning sequence:

1. Linux process isolation.
2. Linux namespaces.
3. Cgroups v2 and resource accounting.
4. CPU and memory contention experiments.
5. Container fundamentals.
6. Docker and containerized workloads.
7. Container networking and failure isolation.
8. Technical reflection and milestone review.

### Learning Approach

Continue using incremental, experiment-driven learning. Prioritize working knowledge over exhaustive memorization, and revisit deeper operating-system mechanisms when required.

The learner should reason independently and perform experiments before receiving complete solutions.

Deferred investigations from Milestone 001 are not blockers for progression.