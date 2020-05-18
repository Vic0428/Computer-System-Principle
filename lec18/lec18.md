# Lecture 18: The interface of OS

## Introduction

1. The definition of interface
   ![image-20200515190952265](lec18.assets/image-20200515190952265.png)

2. API
   ![image-20200515191017835](lec18.assets/image-20200515191017835.png)

   Two kinds of OS interface

   - POSIX
   - Windows API (Win32) 

3. Implementation of APIs is the **lesser** problem

   - Performance can be improved later; bugs are irritating, but can be **fixed**

   API design is the **big** problem

   - Bugs
     - Developer misunderstood API
   - Performance
     - Transferring data (streaming or not?)
   - Many apps will be affected 
     - ABI: application binary interface
     - Change OS is hard

4. Why is API design a problem?
   ![image-20200515191506923](lec18.assets/image-20200515191506923.png)

   - OS is a long term project and **hard to predict the future**
   - Want to extend one feature, old program will break!
     - Developer are unhappy!

5. Win32 API: From the view of software engineering

   ![image-20200515191738060](lec18.assets/image-20200515191738060.png)

   Elements
   ![image-20200515192020202](lec18.assets/image-20200515192020202.png)

   - More than 3000 functions, more than 1000 error codes

   ![image-20200515192110109](lec18.assets/image-20200515192110109.png)

   - Clear API description => formal proof will make less bugs, and better API design (may improve performance)

   ![image-20200515192307535](lec18.assets/image-20200515192307535.png)

   ![image-20200515192417897](lec18.assets/image-20200515192417897.png)

   - VOID pointer can point to any type
   - Windows is a process of development. (From DOS). It can’t even imagine the need of future. 
   - Windows use C language (pointer is flexible). Flexible, but high risks. 
     Type system problem may be the fault of C programming language. 

## Overview of POSIX

### History

1. First Research Edition
   ![image-20200516170655220](lec18.assets/image-20200516170655220.png)

2. FreeBSD’s system call evolution
   ![image-20200516170726372](lec18.assets/image-20200516170726372.png)

   C library functions
   ![image-20200516170746076](lec18.assets/image-20200516170746076.png)

3. The sequence of system call
   ![image-20200516170845055](lec18.assets/image-20200516170845055.png)

   An overview of syscall data collection
   ![image-20200516170956974](lec18.assets/image-20200516170956974.png)

   - `man 2 write`
   - Find `syscall` related commits

4. An overview of `syscall` empirical study
   ![image-20200516171130929](lec18.assets/image-20200516171130929.png)

5. System call categories
   ![image-20200516171205957](lec18.assets/image-20200516171205957.png)

   - Lack `gui` `syscall`
   - Lack IPC `syscall`

   Sibling `syscalls`
   ![image-20200516171223477](lec18.assets/image-20200516171223477.png)

    
   
   - Real time system calls have `rt` as prefix
   
   **New system calls**
   ![image-20200517091128307](lec18.assets/image-20200517091128307.png)
   
   - New feature for server 
   - Adding new system calls to satisfy the need of user
   
6. Classifying of Decade of System Call Commits
   ![image-20200517091441970](lec18.assets/image-20200517091441970.png)

7. The system calls with the most commits
   ![image-20200517091455944](lec18.assets/image-20200517091455944.png)

   - `ptrace` and `signal` are very complex. 

8. Classifying Bufflehead Fixes of `Syscalls`
   ![image-20200517091550753](lec18.assets/image-20200517091550753.png)

   - Concurrency bug is hard to reproduce
   - Semantic bug (not as expected)

9. Classifying Bug Fixes of Syscalls
   ![image-20200517091828023](lec18.assets/image-20200517091828023.png)

   - Signal handling system calls have the highest number of semantic (9.33) and compatibility-related (1.70) bug fixes per system call
   - Driver bug is not considered. 

## POSIX in modern OS

1. Introduction
   ![image-20200517092625188](lec18.assets/image-20200517092625188.png)

   - Modern OS: Android, IOS ...
   - Modern workloads: WeChat, Twitter ..
   - Some unpopular POSIX abstractions and what we need to have ?

2. Workloads & Methodology
   ![image-20200517092813284](lec18.assets/image-20200517092813284.png)

   - Ubuntu is not so modern ...

3. Question 1
   ![image-20200517092920352](lec18.assets/image-20200517092920352.png)

   - **Large numbers of unused or unimplemented abstractions**, Departure from traditional IPC and

     async I/O

   - IPC is not widely used..

4. Question 2
   ![image-20200517093048625](lec18.assets/image-20200517093048625.png)

   - IOCTL: Extension API used to shortcut POSIX; Directly interact with the kernel; Build

     functionality not expressed from POSIX APIs

     - Read special file ..
     - GPU is controller by IOCTL

   - THREAD => more effective and high performance

5. Question 3
   ![image-20200517093255576](lec18.assets/image-20200517093255576.png)

   - POSIX not provided GUI abstraction
   - GPU is not block device, or character device … => so we need too use IOCTL
   - Binder IPC is more high performant. 
     - Not only transferring data
     - But also interact with other processes

6. Evolution of systems and applications
   ![image-20200517093635441](lec18.assets/image-20200517093635441.png)

   

## How to design a Linux kernel interface?

1. The Linux Programming Interface
   ![image-20200517132639236](lec18.assets/image-20200517132639236.png)
2. Moral 1: diverse user usages
   ![image-20200517132740714](lec18.assets/image-20200517132740714.png)
   - The max size of queue is 1024 => to small for nowadays usage
   - **Semantics has changed**...
3. Moral 2: unit tests
   ![image-20200517132942061](lec18.assets/image-20200517132942061.png)
4. Moral 3: Specification
   ![image-20200517133154547](lec18.assets/image-20200517133154547.png)
   - Now description is based on human language => also Leeds some bugs
5. Moral 4: feedback loop
   ![image-20200517133357023](lec18.assets/image-20200517133357023.png)
6.  Moral 5: Into real world
   ![image-20200517133430181](lec18.assets/image-20200517133430181.png)
   - In real word usage, we can find the bug
7. Moral 6: technical checklist
   ![image-20200517133555116](lec18.assets/image-20200517133555116.png)
   - Extensibility is **important**

## Interface of Performance

1. Three papers want to change the system call
   ![image-20200517205648169](lec18.assets/image-20200517205648169.png)
   - `Flexsc`: **change sys call from sync to async**
   - MegaPipe: change syscall interface to **utilize high performant Network I/O**
   - Make `syscall` **better on multi-core**
2. Improve performance
   - Sync system call is a legacy
   - Syscall is not for high-speed net
   - Syscall is not for multicore arch
     - The **semantic** of syscall

### FlexSC

1. Sync => Expensive
   ![image-20200517210350714](lec18.assets/image-20200517210350714.png)
2. Processor structure pollution is **bad** !
   ![image-20200517210431113](lec18.assets/image-20200517210431113.png)
3. Key source of performance impact
   ![image-20200517210537469](lec18.assets/image-20200517210537469.png)
   - IO operation => switch process => TLB miss ...

4. Exception-less syscalls
   ![image-20200517210658024](lec18.assets/image-20200517210658024.png)

   Contributions: **Exception-less sys calls** & **FlexSC-Threads**
   ![image-20200517210738588](lec18.assets/image-20200517210738588.png)

   

   - Use `perf` tool to find CPU inside data
   - Core idea: async + batch processing + share memory + core specialization
   - Complex: need to change `glibc` … Mainly reduce user-mode switch cost but not processor pollution structure cost ...

   ### MegaPipe

   1. Introduction
      ![image-20200517220139486](lec18.assets/image-20200517220139486.png)

      - Different message size 
        - low message size => low throughput
        - Large message size => low CPU usage

      - Multicore => we want ideal scaling

   2. BSD Socket API Performance Issues
      ![image-20200517220548291](lec18.assets/image-20200517220548291.png)

      - `listen_fd` => share data structure (write sync cost)
      - File abstraction => VFS overhead & lock overhead

      MegaPipe
      ![image-20200517220720136](lec18.assets/image-20200517220720136.png)

      - Each core has one listen socket … (reduce shared socket overhead)
      - Bypass VFS => use light weight socket!

   3. Left: traditional design, right: Mega Pipe Design
      ![image-20200517220854823](lec18.assets/image-20200517220854823.png)

   4. Completion Notification Model
      ![image-20200517220931030](lec18.assets/image-20200517220931030.png)

   5. Multicore scalability

      ![image-20200517221129053](lec18.assets/image-20200517221129053.png)

      

   ### Commuter

   1. The real bottlenecks may be in the **interface design**

      ![image-20200518092653249](lec18.assets/image-20200518092653249.png)

      - ‘Create(x)’ => the smallest integer as `fd`
        - Traditional multiple `create` commands => need a global lock
      - Now change `Create(x)` => the usable integer is OK
        - Now parallel is possible

   2. Commute rule
      ![image-20200518093132036](lec18.assets/image-20200518093132036.png)

   3. Design & Implement & Test
      ![image-20200518093339354](lec18.assets/image-20200518093339354.png)

   4. Formalizing the rule
      ![image-20200518093519780](lec18.assets/image-20200518093519780.png)

      Example
      ![image-20200518100439076](lec18.assets/image-20200518100439076.png)

       

   5. An example of commuter
      ![image-20200518101157158](lec18.assets/image-20200518101157158.png)

      Using the rules to build the scalable OS
      ![image-20200518102028758](lec18.assets/image-20200518102028758.png)

       ![image-20200518102040039](lec18.assets/image-20200518102040039.png)

      ![image-20200518102045682](lec18.assets/image-20200518102045682.png)

   6. Refining the POSIX with the rule
      ![image-20200518102115011](lec18.assets/image-20200518102115011.png)

      

