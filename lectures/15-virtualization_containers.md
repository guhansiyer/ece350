# Virtualization and Containers

## Virtualization

The goal of virtual machines is to abstract the hardware of a single computer into several different execution environments, in which we may have different operating systems running, or multiple copies of the same OS. From the perspective of an OS, it does not usually know that it is executing on an abstraction of the hardware.

At the lowest level, there is the *host*, the underlying hardware. Above this, there is the virtual machine manager (VMM) also referred to as the *hypervisor*, which creates an interface that looks like the host but can have multiple copies. The *guests* interact with their own copy of the host, allowing multiple operating systems to exist concurrently on one physical machine.

This is different from emulation. With virtualization, Windows and Linux can both run on the same `x86_64` architecture as guests. An Android emulator running on the same architecture has code to simulate an entirely different CPU. So an Android app runs in a simulation of a mobile environment.

Virtual machines can also have their execution suspended much like pausing a process. Also like processes, they can be moved to another system or cloned, and there is protection between guests and the host.

## Behind the Scenes

A key building block to virtualization is the virtual CPU (VCPU). This is just the state of the CPU according to a guest machine. The hypervisor is responsible for maintaining the state of the VCPU. Similar to a PCB, the VCPU is a data structure that stores the state when the guest isn't running, and the state is restored from the VCPU structure when the guest is scheduled to run.

The guest OS runs in user mode, but will inevitably want to do something that requires kernel mode, so we need a virtual user and kernel mode.

The first strategy to implement this is *trap-and-emulate*. If the guest attempts a privileged instruction, it will generate a trap because it is in user mode. The hypervisor should then pick this up and execute, emulating the requested operation. Non-privileged instructions execute natively on the hardware, so they are fast, but privileged instructions have extra overhead in this case. To get around this, some CPUs have more than just two modes and keep track of virtual user and kernel mode.

For other CPUs which do not have clear definitions of privileged vs. non-privileged instructions, we use a technique called *binary translation*. If the guest VCPU is in user mode, instructions can be run natively. If it is in kernel mode, the guest believes it is running in kernel mode. The hypervisor looks at every instruction before they get to the CPU; if they are special instructions, they are translated into alternative instructions that produce the same result. This results in a small performance decrease, but allows most instructions to run natively, with only a small number needing to be emulated.

## The Impact

There are more demands on and difficulties in resources.

### Scheduling

Even with a single CPU in the physical machine, the challenge is to schedule the VCPU operations on it. There are hypervisor and guest threads.

A guest system is configured with some number of CPUs, and as long as there are enough CPUs in the physical system to meet allocation commitments, we won't have a problem.

If the resources are fully committed, the hypervisor does not need too much time on its own, so it can "steal" cycles occasionally. Hypervisor operations run on CPUs that aren't too busy at the moment.

If the resources are overcommitted, the hypervisor will have to figure out how to map the VCPUs to physical ones according to some scheduling strategy. The expectation of the guest OS of certain deadlines becomes inaccurate in this case.

### Memory Management

Virtualization makes memory much more complex to deal with. Where we previously had one operating system and its structures in memory, there are now multiple. There are a few strategies to alleviate the problem:

* **Nested Page Tables**
  * The hypervisor has a nested page table that translates the guest's page table to the physical one. The problem is that the hypervisor knows less about the guest's memory patterns than the guest does.
*  **Device Drivers**
   *  We install a driver in the guest that allows the hypervisor to exercise some measure of control over the guest. When needed, this "balloon" memory manager requests a bunch of empty memory and asks the guest to pin its pages in physical memory. This makes the guest think memory is in short supply and will start to free up memory. The hypervisor knows the balloon pages are not real and can allocate them to some other guest.
*  **Duplicate Detection**
   *  The hypervisor looks to see if the same page is loaded more than once.

## Containerization

Containerization gives many of the advantages of VM separation without the overhead of a guest OS. Containers are run by a container engine so there is some abstraction of the underlying hardware, assembled from a specification that says what libraries, tools, etc. are needed. When the container is built and deployed it is sufficiently isolated but shares where it can. They can be thought of as a very lightweight VM.