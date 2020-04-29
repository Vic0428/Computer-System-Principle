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

3. 

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

## Reference

1. [THU AOS](https://github.com/chyyuu/aos_course_info)
2. [SJTU IPADS CSP](https://ipads.se.sjtu.edu.cn/courses/csp/)

