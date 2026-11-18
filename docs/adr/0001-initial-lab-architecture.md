
Milestone 0 - Hardware and Architecture


1. Architecture: Why we're starting with two Linux VMs and containers rather than only containers on macOS.
The main reason is that we want to experiment with independent guest operating systems, not simply isolated applications.

2. Resource allocation: How you'd divide CPU and RAM between the VMs on your 36 GB MacBook.
I’d allocate 4GB, 2vCPU for each VM

3. Failure domains: Which failures can be isolated and which remain shared.

Physical host failure is shared between all virtual hosts, virtual host failure is shared between containers. A container level failure (process-level failure) is isolated from other containers, VM level failure is isolated from other VMs. Isolation isn't absolute. For example, a container exhausting shared host resources can affect other containers if resource limits aren't properly configured.

3. Limitations: What this home lab cannot reproduce compared with a real multi-machine cluster.
This home lab (as for now) cannot replicate physical machine availability. Other limitations include independent hardware faults, realistic inter-machine network characteristics, and physical network-device failures.