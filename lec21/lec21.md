

# Lecture 21: Concurrency in OS kernel

## Introduction

1. Locking in the **Linux Kernel**
   - Why do we need locking in the kernel?
   - Which problems are we trying to solve?
   - What implementation choices do we have?
   - Is there a one-size-fits-all solution?
2. **Concurrency in Linux**
   - Linux is a **symmetric multiprocessing** (SMP) preemptible kernel
   - Its has true concurrency
   - And various forms of **pseudo concurrency**

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

   - Notice: **interrupt disable** is even stricter than **preempt disable**

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

## Scalable lock

### Ticket spinlock

1. Goal

   - **Correctness**: Mutual exclusion, Progress, Bounded wait
   - **Fairness**
   - **Performance**

2. Idea

   - Reserve each thread’s turn to use a lock
   - Each thread spins until their turn
   - Use new atomic primitive: fetch-and-add (FAA)
   - Spin while not thread’s != turn
   - Release: Advance to next turn

   ```
   typedef struct {
   	int ticket;
   	int turn;
   } lock_t;
   void lock_init(lock_t *lock) {
   	lock->ticket = 0;
   	lock->turn = 0;
   }
   void acquire(lock_t *lock) {
     int myturn = FAA(&lock->ticket);
     while (lock->turn != myturn); // spin
   }
   void release(lock_t *lock) { lock->turn += 1; }
   ```

3. Analysis of ticket spinlock
   ![image-20200718092330017](lec21.assets/image-20200718092330017.png)

   

### MCS lock

1. Spin on different cache line

   - Goal: O(1) message release time
   - Can we wake just one core at a time?
   - Idea: Have each core spin on a different cache-line

2. MCS lock
   ![image-20200718100726788](lec21.assets/image-20200718100726788.png)

   ![image-20200718100742886](lec21.assets/image-20200718100742886.png)

   - Acquiring MCS locks

     ```c++
     acquire (qnode *L, qnode *I) {
       I->next = NULL;
       qnode *predecessor = I;
       XCHG (*L, predecessor);
       if (predecessor != NULL) {
         I->locked = true;
         predecessor->next = I;
         while (I->locked) ;
       }
     }
     ```

   - Releasing MCS locks

     ```c++
     release (lock *L, qnode *I) {
     	if (!I->next)
     		if (CAS (*L, I, NULL))
     			return;
       while (!I->next) ;
       I->next->locked = false;
     }
     ```

     

3. Compare Ticket lock and MCS lock

   ![image-20200718101450585](lec21.assets/image-20200718101450585.png)

   - When the number of cores is low and the critical region is small, the MCS lock complex acquiring and releasing logic cost a lot of time. 

### CLH lock

1. An illustrated diagram
   ![image-20200718102112143](lec21.assets/image-20200718102112143.png)

2. CLH lock

   ```c++
   type qnode = record
     prev : ^qnode
     succ_must_wait : Boolean
   
   type lock = ^qnode // initialized to point to an unowned qnode
   
   procedure acquire_lock (L : ^lock, I : ^qnode)
     I->succ_must_wait := true
     pred : ^qnode := I->prev := fetch_and_store(L, I)
     repeat while pred->succ_must_wait
   
   procedure release_lock (ref I : ^qnode)
     pred : ^qnode := I->prev
     I->succ_must_wait := false
     I := pred // take pred's qnode
   ```

3. Waiting on previous one variable on different CPU. 
   - Suitable for SMP architecture
   - But a disaster for NUMA

### Comparison

1. Locking strategy comparison
   ![image-20200718102529513](lec21.assets/image-20200718102529513.png)























