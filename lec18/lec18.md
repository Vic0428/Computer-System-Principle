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