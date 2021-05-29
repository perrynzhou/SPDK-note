## DPDK

**DPDK (Data Plane Development Kit)** is a framework (under the Linux Foundation) comprised of various userspace libraries and drivers for fast packet processing[[1\]](https://www.packetflow.co.uk/what-is-dpdk/#fn1). Originally developed by Intel to run on x86 based CPUs, DPDK now supports other CPU types, such as IBM POWER and ARM.

Though DPDK uses a number of techniques to optimise packet throughput, how it works (and the key to its performance) is based upon **Fast-Path** and **PMD**.

- **Fast-Path (Kernel bypass)** - A fast-path is created from the NIC to the application within user space, in turn, bypassing the kernel. This eliminates context switching when moving the frame between user space/kernel space. Additionally, further gains are also obtained by negating the kernel stack/network driver, and the performance penalties they introduce.
- **Poll Mode Driver** - Instead of the NIC raising an interrupt to the CPU when a frame is received, the CPU runs a poll mode driver (PMD) to constantly poll the NIC for new packets. However, this does mean that a CPU core must be dedicated and assigned to running PMD.

Additionally, the DPDK framework includes the following:

- **Buffer Managers** to optimize the allocation/deallocation of the network buffer queues.
- **NUMA** awareness to avoid expensive memory operations across memory nodes.
- **Hugepages** to optimise physical-to-virtual page mappings within the CPU's TLB.

![image2-7](../images/image2-7.png)
**Figure 1** - Standard data path vs DPDK.

NOTE: As well as the need for a DPDK supported CPU, a DPDK enabled networking adapter is also required.

## OVS-DPDK

Out of the box OVS implements a kernel based data path, along with with a userspace daemon - ovs-vswitchd (Figure 2). This daemon processes new flows, allowing northbound controllers to programmatically update the control plane. However, this leads to a problem: even though the frame is cached within the kernel, it must be moved to userspace for processing by the virtual function. As we have already seen, context switching the frame between address spaces is expensive with regards to CPU cycles.

![image1-7](../images/image1-7.png)

**Figure 2** - Standard OVS vs DPDK-OVS.

Open vSwitch (OVS) introduced DPDK support in OVS 2.2.

OVS-DPDK moves the OVS kernel data path into userspace. Frames are then retrieved directly from the NIC using a poll mode driver (PMD), allowing us to bypass the kernel.

With the frame and the VNF both now within userspace, further techniques can be used to move the frame between OVS and the VNF, such as vHost User.

**vHost User** is a driver, introduced within OVS 2.4.0, that allows QEMU (the virtualization software) to share with DPDK the memory location of the guests' virtual queues. QEMU shares the memory allocations via the drivers' backend implementations within OVS-DPDK. This allows OVS-DPDK to send and receive frames between the queues via DMA, resulting in a huge performance benefit.