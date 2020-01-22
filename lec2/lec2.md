# 2. OS Structure

## Exokernel

Compare three different kinds of kernels

![image-20200122103449671](lec2.assets/image-20200122103449671.png)

### Traditional OS

1. Only privileged servers and the kernel can manage system resources
2. Both resource management & protection are done by kernel
   - Centralized control
3. Untrusted applications are limited to the interface
   - Limited functionality
   - Hurt application performance
   - Hide information (page fault etc.)
4. An interface designed to accommodate **every** application
   - Flaw actually
5. Solution
   - Allow applications enough control over resources
   - Separating application from management

### The idea behind exokernel

1. Separate resource management from protection

2. **Kernel**: only protect resources

3. **Application**: only manage resources

4. Specialisation is common

   - Application spécialization
     - Millions of apps
   - OS specialization 
     - For server, desktop, phone … and more
   - Hardware specialization
     - CPU, GPU, ...

5. OS as a library

   - Different libos can coexist on the same **exokernel**

   ![image-20200122104422476](lec2.assets/image-20200122104422476.png)

### Exokernel: Design Challenge

1. Kernel’s new role
   - Tracking ownership of resources
   - Ensuring resources protection
   - Revoking resource access
2. And smart guys introduce three techniques
   - Secure binding
   - Visible revocation
   - Abort protocol

### Exokernel: Principles

1. Separate resource protocol and management
2. Expose allocation
3. Expose names
4. Expose revocations
5. Expose information

### Exokernel: Benefits

1. Expose kernel data structure
   - Without lots of system calls
2. Flexibility
   - Libos can modified and debugged more easily
3. Performance improvement

### Exokernel: Drawbacks

1. The interface design is not simple
2. The ease of creation and mixing libOSes could lead to code messes
3. Customer support is harder than before
   - What’s OS you are choosing?

## Flexible System Call

Flexible System Call Scheduling with Exception-Less System Calls, OSDI'10

### Motivation

1. How to further reduce the latency of `syscall`
   - Mostly of the state switch (from kernel mode to user mode)
   - Cache pollution
2. Could we do `syscall` without state switching?

### Overview

1. Introduce `system-call-page` that is shared by user and kernel
   - User threads can **push** the system call requests into the system call page
   - Kernel threads will **poll** the system call requests out the system call page
2. Another way of `syscall`
   ![image-20200122151133878](lec2.assets/image-20200122151133878.png)



## Reference

1. [CSP Lecture 2](https://ipads.se.sjtu.edu.cn/courses/csp/slides/CSP_02_OS_Structure.pptx)
2. [MIT-Exokernel](https://pdos.csail.mit.edu/archive/exo/)

2. [Flex-SC](https://www.usenix.org/conference/osdi10/flexsc-flexible-system-call-scheduling-exception-less-system-calls)

