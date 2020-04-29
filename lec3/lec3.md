# 3. Micro Kernel

## What is a Microkernel?

1. Purpose of Operating Systems

   - **Manage hardware resources**
     - E.g., CPU, Memory
   - **Provide ease-to-use interfaces to access resources**
     - E.g., network sockets
   - **Perform privileged / HW-specific operations**
     - E.g., ring 0/3
   - **Provide separation and collaboration**
     - E.g., users/processes **isolation** and **communication**

2. Problem with Monolithic kernel

   - Example: Linux, Windows, MacOS

   - Design: all kernel are in kernel mode. User applications are in user mode. 

     ![image-20200429143016477](lec3.assets/image-20200429143016477.png)

   - **System components run in privileged mode**

     - No **protection** between **system components**
       - Security issues: e.g., faulty driver
     - No **isolation** between **system components**
       - Resilience issues: e.g., crash together
       - 80% code of Linux are driver code. But drivers are not written by **senior linux kernel developer**
     - **Big** and **inflexible**
       - Difficult to replace system components
       - Difficult to understand and maintain
       - **A disaster from software engineering perspective**

3. Micro kernel

   - Example

     - QNX (used in car)
     - Fuchsia / Zircon (Google)
     - Fiasco (L4)
     - $M^3$ (Heterogeneous hardware)

   - Kernel with minimal features (IPC, interrupt …)

   - Memory management process is like a **server** which handle memory management request.

   - **Move as much from the kernel into user space**

     - Address spaces are different for system services implemented in user level.
     - Use IPC to communicate 
     - Scheduling is frequent
     - Only necessary services are running in **kernel mode**. Others are all in **user mode**.

     ![image-20200429143922877](lec3.assets/image-20200429143922877.png)

4. Example: File Creation

   - In monolithic kernel, user applications used **system call**  to use `file system` service in kernel. 

   - In micro kernel, we need to **use IPC to use system service**

     - Application use IPC to send request to File System 
     - File system use IPC to send request to Disk Driver
     - Disk driver did work on disk
     - Similar steps to return the result to APP

     ![image-20200429144337757](lec3.assets/image-20200429144337757.png)

5. Pros and Cons

   - Pros
     - Flexibility
       - Implement kernel function in **user space**
       - System services in user-level services
       - Flexible and **extensible**
     - Safety
       - User process has **protection method** (using virtual memory) 
       - Protection between individual components
       - More resilient (**no crash whole**)
       - More secure (**inter-component isolation**)
     - Modularity
       - From Software Engineering perspective
     - Minimal OS kernel
       - Small TCB (Trusted Computing Base) (~10 kLOC order)
         - Compare to Linux (Big)
       - **Suitable for verification**
   - Cons
     - **Address spaces**
       - Driver and server and kernel are in **different** user space
     - **IPC**
       - **Huge** IPC overhead
     - **Scheduling**
       - **Frequent** scheduling

   ![image-20200429092908074](lec3.assets/image-20200429092908074.png)

   

4. Good design but **bad performance**. What Microkernels can Give Us?

   - **Customizability**
     - Configure suitable servers (embedded, desktop)
     - Remove unneeded servers 
       - We can simple remove system service in user level
   - **Enforce reasonable system design**
     - **Well-defined interfaces** between components
     - **No access to components besides the interfaces**
     - **Improved maintainability**

5. Comparison between Monolithic kernel VS. Microkernel

   ![image-20200429145623411](lec3.assets/image-20200429145623411.png)

   - Monolithic kernel also improved their customisability and maintainability.

     - Now linux can use `menuconfig` to select `modules` need to be compiled. And `Linux module` not allowed other modules to use their `variables`   unless explicitly exported. 
     - **Micro kernel** not have strong advantages considering flexibility and maintainability nowadays.

   - Why micro kernel has bad performance?

     - **Consider a simple system call**

       ![image-20200429150043946](lec3.assets/image-20200429150043946.png)

     - Consider calls between services
       ![image-20200429150121598](lec3.assets/image-20200429150121598.png)

## What do Micro Kernels Have?

1. Most system services are **implemented in user level**, so the **Microkernel size is very small**. 
2. L4 Family Microkernel
   ![image-20200429162757765](lec3.assets/image-20200429162757765.png)
   - L4 Micro kernel is the most fully evolved **micro kernel**. 
   - Two active open-source groups
     - UNSW
     - TU Dresden
3. L4 Concepts
   - **A microkernel does no real work**
     - Kernel only provides **inevitable mechanisms**
     - **Kernel does not enforce policies**

### Case study: L4/Fiasco.OC

1. Everything is an object  (Like `Every thing is a file descriptor` in Unix) => An abstraction

   - Task -> Address Space
   - Thread -> Scheduling
   - IPC Gate -> Communication
   - IRQ -> Communication

   One system call: `invoke_object()`

2. Kernel only provide limited objects

   - Threads
   - Tasks
   - IRQs
   - IPC Gate: **Generic Communication Object**
     - Send message from **sender** and **receiver**
     - Used to implement **new objects in user-level applications **

3. User level objects
   ![image-20200429165152954](lec3.assets/image-20200429165152954.png)

   - Kernel even doesn’t know user level objects (like networking stack and file system)

4. How to call objects?

   - A simple idea
     ![image-20200429165845672](lec3.assets/image-20200429165845672.png)
   - Local names for Objects
     ![image-20200429165950667](lec3.assets/image-20200429165950667.png)
     - Namespace is **per-task**
   - Object Capabilities
     ![image-20200429171039515](lec3.assets/image-20200429171039515.png)
   - Communication
     ![image-20200429171154860](lec3.assets/image-20200429171154860.png)
   - Capabilities (Local Names)
     ![image-20200429171318506](lec3.assets/image-20200429171318506.png)
     - Each process used local name and **capability table** to find the true object and used IPC gate. 

## Go deeper into details

1. Abstraction in Micro-kernel

   - Object
   - Capability
   - IPC

2. Abstraction: Thread (Abstraction provided by kernel)

   - An independent flow of control **inside an address space**
   - Communicates with other threads **using IPC**
   - Characterized by **a set of registers and the thread state**
   - Dispatched by the kernel **according to a defined schedule**

   Implementation in L4/NOVA
   ![image-20200429172128871](lec3.assets/image-20200429172128871.png)

   Thread Variants
   ![image-20200429172319286](lec3.assets/image-20200429172319286.png)

   - Global thread is similar to thread in Linux (need schedule context to scheduled by internal scheduler)
   - Local thread doesn’t have schedule context and internal scheduler can not schedule **local thread** actively. 

3. Portals

   - A portal is an IPC endpoint (One kind of **IPC Gate**)
   - Executed by local threads
   - CPU time is donated from caller
   - Called via system call
   - Message is transferred from send UTCP to receiver UTCP

   ![image-20200429220440420](lec3.assets/image-20200429220440420.png)

   Kernel schedules **schedule context**. When global thread call portal, and it will **allow local thread to use its schedule context**. Also local thread can calls **portal** and passed schedule context to it. 

### IPC

1. Microkernel Performance Problem
   ![image-20200429221059438](lec3.assets/image-20200429221059438.png)

2. Improving IPC by Kernel Design

   - **IPC is the most important operation in a microkernel**
   - **The way to make IPC fast is to design the whole system around it**
   - **Design principle: aim at a concrete performance goal**
   - **Applied to the L3 kernel**

3. L3/L4 Implementation Techniques
   ![image-20200429221317800](lec3.assets/image-20200429221317800.png)

   ![image-20200429221417822](lec3.assets/image-20200429221417822.png)

4. Result (L3)
   ![image-20200429221454581](lec3.assets/image-20200429221454581.png)

5. Lightweight RPC (LRPC): Basic Concepts
   ![image-20200429221546142](lec3.assets/image-20200429221546142.png)

#### L4/Fiasco.OC IPC

1. New IPC Design
   ![image-20200429221645744](lec3.assets/image-20200429221645744.png)

2. Three kinds of IPC

   - Register IPC

     ![image-20200429221808351](lec3.assets/image-20200429221808351.png)

   - Kernel Memory IPC

     ![image-20200429221828994](lec3.assets/image-20200429221828994.png)

   - Shared Memory IPC

     ![image-20200429221852235](lec3.assets/image-20200429221852235.png)

3. Comparison of them
   ![image-20200429221928646](lec3.assets/image-20200429221928646.png)

4. Usage of IPC in L4/NOVA
   ![image-20200429222013857](lec3.assets/image-20200429222013857.png)

### Abstraction: Capabilities

1. Capability
   - Answer two questions
     - How do you find/access resources?
     - How do you restrict access to resources?
   - Give each subject **a local namespace**
   - Operations to **exchange objects** between namespaces
   - **Permission** depends on what you have
2. Overview
   ![image-20200429222733831](lec3.assets/image-20200429222733831.png)
   - Delegate: **copy**
   - Grant: **move**
3. Hierarchical Memory Capability
   ![image-20200429222839718](lec3.assets/image-20200429222839718.png)

### Memory Management

1. Page Fault Handling

   ![image-20200429222935819](lec3.assets/image-20200429222935819.png)

2. Pagers blocking waits for page fault messages
   ![image-20200429223017367](lec3.assets/image-20200429223017367.png)

3. Memory Mapping
   ![image-20200429223047212](lec3.assets/image-20200429223047212.png)

4. User-Level Pagers
   ![image-20200429223127292](lec3.assets/image-20200429223127292.png)

   ![image-20200429223147732](lec3.assets/image-20200429223147732.png)

5. How does the kernel know **which pager is responsible for handling a fault **?

   - Another layer of indirection: Region Mapper

     ![image-20200429223327442](lec3.assets/image-20200429223327442.png)

   - Region Mapper and Page Faults

     - Step-1: All threads of a task **have the RM assigned as their pager**

     - Step-2: Fiasco.OC redirects all page faults to the RM

     - Step-3: RM then uses **synchronous IPC calls** to obtain real memory mappings from the external pager responsible for a region

## Microkernel - Mach

1. Mach kernel
   ![image-20200429093048335](lec3.assets/image-20200429093048335.png)

   - Foundation for several **real systems**

     - IBM Workspace OS
     - Next OS -> Max OS X

   - Big goals

     - Multiple OS personalities

       - System services enables application can cross different OS
       - Both windows software and linux software can run on Mach

     - Run on multiple HW architectures

       - ARM, x86, MIPS ...

       ![image-20200429145115702](lec3.assets/image-20200429145115702.png)

   - Can support various kinds of **OS Application** and **database**

   - Different threads have different **ports**

   - Memory object concept

     - **More flexible**
     - **Memory object** -> we can transfer memory from one process to another process
     - **Hierarchical pagers**

   ![image-20200429093659880](lec3.assets/image-20200429093659880.png)

2. Performance problem with `Mach`

   - the use of IPC for almost all tasks turned out to have serious performance impact.
   - **system calls take 5-6X** as long as UNIX
   - given a `syscall` that does nothing, a full round-trip under BSD would require about 40μs, whereas on **a user-space Mach system it would take just under 500μs**.
   - benchmarks on 1997 hardware showed that Mach 3.0-based UNIX single-server implementations were about **50% slower than native UNIX**.

3. Lessons Learned from `Mach`

   ![image-20200429145227567](lec3.assets/image-20200429145227567.png)

   **Micro kernel may be good for embedding system and some realtime system!**

## Microkernel - L4

1. Second generation microkernel - L4 by Jochen Liedtke (GMD)

   - Target for **embedding system**, not **general system**

   - Performance

     - **synchronous IPCs** -> async IPCs (like `epoll` in Linux)
       - Reduce scheduling frequency
     - Smaller, Mach 3 (330 KB) -> L4 (12 KB)
     - **IPC security checks moved to user process**
       - Better for performance
     - IPC is **hardware dependent**
       - Wrote in assembly

     ![image-20200429094430603](lec3.assets/image-20200429094430603.png)

2. L4 family
   ![image-20200429094457390](lec3.assets/image-20200429094457390.png)

   One-way IPC cost over years
   ![image-20200429094540301](lec3.assets/image-20200429094540301.png)

   Sources lines of code
   ![image-20200429094609939](lec3.assets/image-20200429094609939.png)

   - Because driver is not in the kernel (microkernel design). `75%` code of Linux kernel are driver program.  
   - Now totally target for **embedding system**
   - **Micro kernel is good for scalability** ( better in distributed system and network system)

## Summary

1. Architecture: **move as much thing as possible out of kernel**
2. Capability and IPC are **two core abstractions**
3. **Make IPC as fast as possible**

## Reference

1. [THU AOS](https://github.com/chyyuu/aos_course_info)
2. [SJTU IPADS CSP](https://ipads.se.sjtu.edu.cn/courses/csp/)

