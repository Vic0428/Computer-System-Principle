# 6. Virtualization

## Introduction to virtualization

### Virtualization layers

![image-20200201111947912](lec6.assets/image-20200201111947912.png)

Machine from the perspective of a system

- ISA provides interface between system and machine
  ![image-20200201112124322](lec6.assets/image-20200201112124322.png)
- Design space (level vs. ISA)
  ![image-20200201112213061](lec6.assets/image-20200201112213061.png)
- As for system VMs and same ISA -> there are two kinds of virtual machines
  - Classic system VMs
    - Run directly on hardware
    - High performance
    - Eg: Xen, VMWare ESX Server
    - A.K.A Type 1
      ![image-20200201112442649](lec6.assets/image-20200201112442649.png)
  - Hosted VMs
    - Ease of construction/installation
    - Eg: VMware Workstation
    - A.K.A Type 2
      ![image-20200201112504124](lec6.assets/image-20200201112504124.png)

### Virtual Machine Monitor

1. The usage

   - For managing VMs which running guest OS

   - VMM runs underlying VMs (higher privilege)

     ![image-20200201112714557](lec6.assets/image-20200201112714557.png)

2. Virtualize hardware

   - CPU
   - Memory
   - Device

3. And there are several different architectures of VMM
   ![image-20200201112813668](lec6.assets/image-20200201112813668.png)

4. Principles in 1974

   - Efficiency
     - Innocuous instructions should execute directly on hardware
   - Resource control
     - Executed programs may not affect the system resources
   - Equivalence
     - The behavior of a program executing under the VMM should be the same as if the program were executed directly on the hardware (except possibly timing and resource availability)

   ![image-20200201113107908](lec6.assets/image-20200201113107908.png)

   ​							*“an efficient, isolated duplicate of the real machine”*

## CPU Virtualization

1. OS VS. VMM
   - Similarities
     - Multiplex hardware
     - Higher privilege
   - Differences
     - Different abstraction
     - VMM schedules VMs, OS schedule processes
2. How does OS use the CPU?
   - Each process thinks it has the 

## Container Virtualization

### Review: Hardware Virtualization

How did we create a virtual machine (VM)

- Start with a **physical machine**
- Create software (**hypervisor**) responsible for isolating the guest OS inside the VM
- VM resources (**memory, disk, network, etc.**) are provided by the physical machine but visibility outside of the VM is **limited**. 

What’s the relationship between virtual machine and physic machine?

- VM and physical machine **share same instruction set**, so must the host and guest
- Guest OS can provide **a different application binary interface (ABI)** inside the VM
- Lots of challenges in getting this to work because guest OS expects to have **privileged hardware access**. 

### Operating System Virtualization

How to do create a virtual operating system (container)?

- Start with **a real operating system**
- Create software responsible for **isolating guest software** inside the container
- Container resources (processes, files, network sockets, etc.) are provided by the real operating system but **visibility** outside the container is **limited**. 

What are the implications?

- Container and real OS **share same kernel**
- So applications inside and outside the kernel must **share the same ABI**
- Challenges is getting this to work are due to **share OS namespaces**

### Why virtualize an OS?

Share many (but not all) of the benefits of hardware virtualization with **much lower overhead**. 

Decoupling

1. **Cannot** run multiple operating systems on the same machine.
2. Can transfer software setups to another machine as long as it has an **identical or nearly identical kernel**. 
3. Can **adjust container resources** to system needs

Isolation

1. Container should not leak information inside and outside the container
2. Can isolate all of the configuration and software packages a particular application needs to run

## OS VS. Hardware Overhead

Hardware virtualization system call path

- Application inside in the VM makes a system call
- Trap to the host OS (hypervisor)
- Hand trap back to the guest OS

OS virtualization System call path

- Application inside the container makes a system call
- Trap to the OS

### OS Virtualization is About Names

What kind of names must the container virtualize?

- Process IDs
- File names
- User names
- Host name and IP address

OS Virtualization is about Control

- CPU time
- memory
- Disk or network bandwidth

### The tech behind container

1. Not a new idea: `chroot` -> run command or interactive shell with special root directory
2. Linux namespaces
   - Mount points
     - allow different namespaces to see different views of the file system
   - Process IDs
     - New processes are allocated IDs in their current namespace and all parent namespaces
   - Network
     - Namespace can have private IP address and their own routing table, and can communicate with other namespaces through virtual interfaces
   - Devices
     - Devices can be present or hidden in different namespaces. 
3. Cgroups
   - A linux kernel feature that limits, accounts for, and isolates the resource usage of a collection of processes
     - Processes and their children remain in the same `cgroup`
     - `cgroups` makes it possible to control the resources allocated to a set of processes

4. UnionFS: A stackable unification file system
5. COW File System
   - Copy on write
   - Only make modifications to the underlying file system when the container modifies files
   - Speeds start up and reduces storage usage
     - The container mainly needs read-only access to host-files

### What is Docker?

Docker builds on previous technologies

- Provides a unified set of tools for container management on a variety of systems
- Layered file system images for easy updates
- Now involved in development of containerization libraries on Linux

![image-20200131161800954](lec6.assets/image-20200131161800954.png)

## Reference

1. [Prof. Geoffrey Challen’s notes](https://www.ops-class.org/slides/2017-04-28-containers/)