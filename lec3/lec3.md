# 3. Micro Kernel

## What is a Microkernel?

1. Micro kernel

   - Kernel with minimal features (IPC, interrupt …)
   - Memory management process is like a **server** which handle memory management request.
   - **Move as much from the kernel into user space**
     - Address spaces are different
     - Use IPC to communicate 
     - Scheduling is frequent

   ![image-20200429092549050](lec3.assets/image-20200429092549050.png)

2. Pros and Cons

   - Pros
     - Flexibility
       - Implement kernel function in **user space**
     - Safety
       - User process has **protection method** (using virtual memory) 
     - Modularity
       - From Software Engineering perspective
   - Cons
     - **Address spaces**
       - Driver and server and kernel are in **different** user space
     - **IPC**
       - **Huge** IPC overhead
     - **Scheduling**
       - **Frequent** scheduling

   ![image-20200429092908074](lec3.assets/image-20200429092908074.png)

   

4. Good design but **bad performance**

## Microkernel - Mach

1. Mach kernel
   ![image-20200429093048335](lec3.assets/image-20200429093048335.png)

   - Can support various kinds of **OS Application** and **database**
     - Provide similar interface like different OS
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

## Reference

1. [THU AOS](https://github.com/chyyuu/aos_course_info)

