# Lecture 21: Concurrency in OS kernel

## Introduction

1. Locking in the Linux Kernel
   - Why do we need locking in the kernel?
   - Which problems are we trying to solve?
   - What implementation choices do we have?
   - Is there a one-size-fits-all solution?
2. Concurrency in Linux
   - Linux is a symmetric multiprocessing (SMP) preemptible kernel
   - Its has true concurrency
   - And various forms of pseudo concurrency

### Pseudo Concurrency in Linux

1. Software based preemption
   ![image-20200714201132614](lec21.assets/image-20200714201132614.png)

2. Hardware preemption
   ![image-20200714201228765](lec21.assets/image-20200714201228765.png)

3. The following assertion is all valid in Uniprocessor world

   ```
   // prog1
   preempt disable;
   	r1 = x;
   	r2 = x;
   preempt enable;
   assert (r1 == r2)
   
   // prog2
   preempt disable;
   	r1 = x;
   	yield();
   	r2 = x;
   preempt enable;
   assert (r1 == r2)
   
   // prog3
   interrupt disable;
   	r1 = x;
   	r2 = x;
   interrupt enable;
   assert (r1 == r2)
   ```

   - Notice: interrupt disable is even stricter than preempt disable

### True Concurrency in Linux

1. What’s true concurrency?
   ![image-20200714201531601](lec21.assets/image-20200714201531601.png)

2. Following assertion is not valid in **Multiprocessor** world

   ```
   interrupt disable;
   	r1 = x;
   	r2 = x;
   interrupt enable;
   assert (r1 == r2)
   ```

3. Atomic operators in Linux
   ![image-20200714201653929](lec21.assets/image-20200714201653929.png)

4. Spin locks in Linux

   - Spin lock
     ![image-20200714202044122](lec21.assets/image-20200714202044122.png)

   - Basic use of spinlock
     ![image-20200714202110988](lec21.assets/image-20200714202110988.png)

   - Spinlocks and interrupts
     ![image-20200714202211188](lec21.assets/image-20200714202211188.png)

   - Solution to Spinlocks and interrupts
     ![image-20200714202530832](lec21.assets/image-20200714202530832.png)

   - Spinlocks & Interrupt disabling
     ![image-20200714211746435](lec21.assets/image-20200714211746435.png)

   - Bottom Halves and Softirqs
     ![image-20200714212328607](lec21.assets/image-20200714212328607.png)

     ![image-20200714212456018](lec21.assets/image-20200714212456018.png)

   - Rules
     - Do not try to re-acquire a spin lock you already have
     - Spinlocks shouldn’t be held for a long time
     - Do not sleep while holding spin locks

5. Semaphores

   - What is Semaphore?
     ![image-20200714212621505](lec21.assets/image-20200714212621505.png)

   - Implementation
     ![image-20200714212754566](lec21.assets/image-20200714212754566.png)

     ![image-20200714212817841](lec21.assets/image-20200714212817841.png)

6. Reader-Writer Locks

   - Why differentiate reader and writer?
     - No need to synchronize concurrent readers unless a writer is present
     - Both spin locks and semaphores have reader/writer variants