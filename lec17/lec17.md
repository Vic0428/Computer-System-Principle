# 17. The Programming Language of OS

## Introduction

1. Some questions
   - Is Programming Language **important for OS**?
   - Is C **the best language of OS**?
   - **Which parts of language** affect OS deeply?
   - Other than languages, **which components affect the development of OS**?
   - Why do we need/needn’t to **change the language for OS**?
   - Other Langs(such as Rust, Go, etc.) based OS will be the future?

2. What is OS?
   ![image-20200501220624230](lec17.assets/image-20200501220624230.png)

   ![image-20200501220645635](lec17.assets/image-20200501220645635.png)

   What is the **Programming Language**?
   ![image-20200501220732230](lec17.assets/image-20200501220732230.png)

   What is the **Programming Languages for OS**?
   ![image-20200501220812128](lec17.assets/image-20200501220812128.png)

   What is System Programming language?
   ![image-20200501221000908](lec17.assets/image-20200501221000908.png)

   - **Focus on performance, reliability**
   - Java is not a system programming language and **Java is a general programming language.** 

   ![image-20200501221433846](lec17.assets/image-20200501221433846.png)

   ![image-20200501221518731](lec17.assets/image-20200501221518731.png)

   - C is both system programming language and **general programming language**
   - More focus on **performance**

   Major System Programming languages
   ![image-20200501221632779](lec17.assets/image-20200501221632779.png)

3. Some OS based on Non-C language
   ![image-20200501221810882](lec17.assets/image-20200501221810882.png)

## History

1. MCP & ESPOL
   ![image-20200501221948179](lec17.assets/image-20200501221948179.png)

   ![image-20200501222226139](lec17.assets/image-20200501222226139.png)

2. MULTICS OS & PL/I language
   ![image-20200501222401381](lec17.assets/image-20200501222401381.png)

   ​	![image-20200501222508093](lec17.assets/image-20200501222508093.png)

   ![image-20200501222622140](lec17.assets/image-20200501222622140.png)

3. UNIX & C
   ![image-20200501222825956](lec17.assets/image-20200501222825956.png)

   ![image-20200501222859938](lec17.assets/image-20200501222859938.png)

   ![image-20200501223204271](lec17.assets/image-20200501223204271.png)

   ​	![image-20200501224214036](lec17.assets/image-20200501224214036.png)

   ![image-20200501224332364](lec17.assets/image-20200501224332364.png)

   ![image-20200501224639425](lec17.assets/image-20200501224639425.png)

   ![image-20200501224857406](lec17.assets/image-20200501224857406.png)

   ![image-20200501225135834](lec17.assets/image-20200501225135834.png)

   ## The Evolution of C Programming Practices: A Study of the Unix Operating System 1973-2015

   ### History of C

   1. History of C
      ![image-20200502093327050](lec17.assets/image-20200502093327050.png)
   2. Why was C be used to develop Unix?
      ![image-20200502093402806](lec17.assets/image-20200502093402806.png)
      - B’s performance is not good as **compiled language**. It not generated machine code.
      - Another successful reason is `DEC VAX 11/780` becoming popular. 
      - The product of **compiler**, **OS**, **hardware** made the success

3. Why C ?
   ![image-20200502094935330](lec17.assets/image-20200502094935330.png)
   - **Close to machine** abstraction
   - **Type safety** => solve bugs in **compiler level**
   - **Small and Simple** => Use big library to **implement complex algorithms**

4. Why success?
   ![image-20200502095133282](lec17.assets/image-20200502095133282.png)

   - The core of C is very **stable**. 

     

### A study of the Unix Operating System 1973-2015

1. Objective
   ![image-20200502095618688](lec17.assets/image-20200502095618688.png)

2. Hypothesis
   ![image-20200502095701502](lec17.assets/image-20200502095701502.png)

   **From software engineering perspective**

3. Timeline of indicative **analyzed revisions** and **milestones** in (from top to bottom): **C language evolution**, **developer interfaces**, **programming guidelines**, **processing capacity**, **collaboration**, **mechanisms**, and **tools**.
   ![image-20200502095841883](lec17.assets/image-20200502095841883.png)

4. Code size changing
   ![image-20200502144046901](lec17.assets/image-20200502144046901.png)

5. H1: Programming practices reflect technology affordances

   - Increase in mean file length (lines / file)
     ![image-20200503215124837](lec17.assets/image-20200503215124837.png)

   - Increase in mean file functionality (statements / file)
     ![image-20200503215136973](lec17.assets/image-20200503215136973.png)

   - Increase in mean line length (characters / line)
     ![image-20200503215244441](lec17.assets/image-20200503215244441.png)

   - Increase in mean identifier length (characters / line)

     ![image-20200503215255737](lec17.assets/image-20200503215255737.png)

   - Increase in mean function length (lines / function)

     ![image-20200503215305083](lec17.assets/image-20200503215305083.png)

   H2: Modularity increases with code size

   - Increase in number of **static** declarations / statement

     ![image-20200503215318655](lec17.assets/image-20200503215318655.png)

   - Increase in number of `#include` directives

     ![image-20200503215339435](lec17.assets/image-20200503215339435.png)

   H3: New language features are increasingly used to saturation point

   - Increase in number of **const** declarations / statement
     ![image-20200503215407128](lec17.assets/image-20200503215407128.png)
   - Increase in number of **enum** declarations / statement
     ![image-20200503215426126](lec17.assets/image-20200503215426126.png)
   - Increase in number of **inline** declarations / statement
     ![image-20200503215443664](lec17.assets/image-20200503215443664.png)
   - Increase in number of **void** declarations / statement
     ![image-20200503215458711](lec17.assets/image-20200503215458711.png)
   - Increase in number of **volatile** declarations / statement
     ![image-20200503215509476](lec17.assets/image-20200503215509476.png)
   - Increase in number of **unsigned** declarations / statement
     ![image-20200503215519381](lec17.assets/image-20200503215519381.png)

   H4: Programmers trust the compiler for register allocation

   - Decreasing number of **register** declarations / statement
     ![image-20200503215535162](lec17.assets/image-20200503215535162.png)

   H5: Code formatting practices converge to a common standard

   - Decrease in code **inconsistency**
     ![image-20200503215549021](lec17.assets/image-20200503215549021.png)
   - Decrease in **indentation spaces** standard deviation
     ![image-20200503215603594](lec17.assets/image-20200503215603594.png)

   H6: Software complexity evolution follows self correction

   - Mean lines / function
     ![image-20200503215612580](lec17.assets/image-20200503215612580.png)
   - Mean statement nesting
     ![image-20200503215623528](lec17.assets/image-20200503215623528.png)
   - Density of C preprocessor conditonals 
     ![image-20200503215641490](lec17.assets/image-20200503215641490.png)
   - Density of C preprocessor non-include directives
     ![image-20200503215653319](lec17.assets/image-20200503215653319.png)
   - **goto** keyword density
     ![image-20200503215711968](lec17.assets/image-20200503215711968.png)

   H7: Code readability increases

   - Mean indentation spaces converge around 6
     ![image-20200503215723375](lec17.assets/image-20200503215723375.png)
   - Statements / line decrease
     ![image-20200503215731937](lec17.assets/image-20200503215731937.png)
   - Comment character density
     ![image-20200503215743107](lec17.assets/image-20200503215743107.png)
   - Kluge word density
     ![image-20200503215753188](lec17.assets/image-20200503215753188.png)

## A Study of Bugs on Linux

### Introduction

Why study faults/bugs/Vulnerabilities in OS code?

- Find the relation between **language** and **OS**

Some questions

- **Where are the errors?**
  - Driver code has error rates three to seven times higher for certain types of errors than code in **the rest of kernel**
- **How are bugs distributed?**
  - The error distribution is readily matched to a logarithmic series distribution
- **How long do bugs live?**
  - the average bug lifetime for certain types of bugs is about 1.8 years.

### 19 years ago

1. Linux Code Base Growth
   ![image-20200504140337531](lec17.assets/image-20200504140337531.png)

   The twelve bug checkers
   ![image-20200504140357042](lec17.assets/image-20200504140357042.png)

2. **Drivers are the one major source of bugs in operating systems**

   ![image-20200504140455966](lec17.assets/image-20200504140455966.png)

   - `Minix 3`: microkernel
   - `Singularity`: use another language writing OS kernel
     - Not good performance
     - Bad backward compatibility

3. Deadlock/data race related bugs
   ![image-20200504140813265](lec17.assets/image-20200504140813265.png)

   - `kmalloc` may sleep

4. NULL/Free related bugs
   ![image-20200504140918345](lec17.assets/image-20200504140918345.png)
   
   - Garbage collection language may prevent these problems..
5. Var related bug
   ![image-20200504141123995](lec17.assets/image-20200504141123995.png)
   - User-mode application stack will grow as demand increase
   - Kernel stack has **fixed** size
6. Range related bugs
   ![image-20200504141222400](lec17.assets/image-20200504141222400.png)
   
   - Java / C# will provide boundary check but not good performance. 

### 9 years ago

1. Line of Code in Linux

   ![image-20200504141423262](lec17.assets/image-20200504141423262.png)

2. Comparative fault count
   ![image-20200504141451660](lec17.assets/image-20200504141451660.png)

   - With code complexity increases, bugs still exist. 

   Per finding and fixing difficulty, and impact likelihood
   ![image-20200504141556017](lec17.assets/image-20200504141556017.png)

3. Linux kernel vulnerability
   ![image-20200504141618349](lec17.assets/image-20200504141618349.png)

   ![image-20200504141731054](lec17.assets/image-20200504141731054.png)

4. Hard to find bugs even with advanced tools
   ![image-20200504141851843](lec17.assets/image-20200504141851843.png)

   - Root cause is **C language** 
   - Semantics bug in C -> **buffer overflow**, **uninitialized data**



## The benefits and costs of writing kernel in a high-level language

1. Should we use high-level languages to build OS kernels?

   - Pros

     - **Easier to program**
     - **Simpler concurrency with GC**
     - Prevents **classes of kernel bugs**

   - Downside

     - Bounds, cast, nil-pointer checks
     - **Reflection**
     - **Garbage collection**

     ![image-20200507214301358](lec17.assets/image-20200507214301358.png)

2. Goal: measure HLL impact

   - Pros
     - **Reduction of bugs**
     - **Simpler** code
   - Cons
     - **HLL safety tax**
     - **GC CPU and memory overhead**
     - **GC pause times**

3. **Methodology**

   - None measure HLL impact in a monolithic POSIX kernel
   - Build new HLL kernel, compare with Linux
   - Isolate HLL impact
     - **Same apps, POSIX interface, and monolithic organization**

   ![image-20200507214432008](lec17.assets/image-20200507214432008.png)

4. Why Go-lang?

   - **Easy to call asm**
   - **Compiled to machine code with good compiler**
   - **Easy concurrency & static analysis**
   - **GC**
     - **Concurrent mark and sweep**
     - Stop-the-world of 10s of us

5. Biscuit
   ![image-20200507214647338](lec17.assets/image-20200507214647338.png)

   ![image-20200507214711643](lec17.assets/image-20200507214711643.png)

   - Can’t allocate heap memory ==> nothing works

   - All kernel face this problem
     ![image-20200507214847813](lec17.assets/image-20200507214847813.png)

     ![image-20200507214911634](lec17.assets/image-20200507214911634.png)

     - Sleep with lock => deadlock

     ![image-20200507215056641](lec17.assets/image-20200507215056641.png)

     - Use static analysis to reserve **memory size**
     - Combination of static analysis and **dynamic runtime**

     ![image-20200507215147406](lec17.assets/image-20200507215147406.png)

     - **RCU** is good for parallel
     - More on **optimizations** (algorithm and hardware), not use feature of HLL
     - **HLL** is only for reducing bugs.

     ![image-20200507215530516](lec17.assets/image-20200507215530516.png)

6. Evaluate 
   ![image-20200507215602470](lec17.assets/image-20200507215602470.png)

   ![image-20200507215656314](lec17.assets/image-20200507215656314.png)

7. Conclusion
   ![image-20200507215811633](lec17.assets/image-20200507215811633.png)

   