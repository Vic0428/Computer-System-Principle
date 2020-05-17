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