# Experiment 001 — VM Networking and Failure Isolation

## Objective

Verify communication between two Linux virtual machines and observe the impact of an individual VM failure on the other machine.

## Environment

- Host: MacBook Pro M3 Pro, 36 GB RAM
- Hypervisor environment: UTM with Apple Virtualization
- Guest OS: Ubuntu Server 24.04 ARM64
- Network mode: NAT Network
- Subnet: 192.168.96.0/24
- VM 1: 192.168.96.3
- VM 2: 192.168.96.2

## Procedure

1. Establish bidirectional connectivity between both VMs.
2. Continuously send ICMP echo requests from VM 1 to VM 2.
3. Shut down VM 2 using `sudo shutdown -h now`.
4. Observe network errors from VM 1.
5. Verify VM 1's uptime and network configuration.
6. Restart VM 2 and verify recovery.

## Observations

- VM 1 received 19 successful ICMP replies before VM 2 became unreachable.
- The average round-trip time for successful replies was 0.983 ms.
- A period without replies occurred before explicit `Destination Host Unreachable` errors appeared.
- VM 1 remained operational and retained its network configuration.
- VM 2 restarted successfully, retained its previous IP address, and accepted SSH connections from the host.
- Post-recovery inter-VM connectivity: pending verification.

## Conclusion

An individual VM shutdown does not necessarily affect the availability of another VM running on the same physical host. However, both VMs still share host-level failure domains.

The experiment also demonstrates that failure detection is not instantaneous. A remote machine can become unavailable before the sending machine explicitly reports a network failure.

Our future inference gateway must account for this uncertainty through health checks, request deadlines, bounded retries, and routing decisions based on worker availability and capacity.

## Limitations

- The experiment uses a graceful VM shutdown rather than an abrupt crash.
- ICMP reachability does not guarantee application-level health.
- Both VMs depend on the same physical host and virtual networking infrastructure.
- No inference gateway or worker services have been deployed yet.
- The exact cause of the delayed unreachable errors was not verified through packet capture.