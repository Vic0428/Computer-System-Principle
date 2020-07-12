# Lecture 20: Multicore

## Introduction

1. Widely used in industry

   - Intel / AMD / IBM / ARM / RISC-V
   - Frequency increasing slow down => multicore arise!

   ![image-20200523180845755](lec20.assets/image-20200523180845755.png)

   - CPU performance increases are slowing

     ![image-20200523223012816](lec20.assets/image-20200523223012816.png)

   ​	

2. CMP VS SMP
   ![image-20200523180956154](lec20.assets/image-20200523180956154.png)

   - Separate => SMP

     - No shared cache

     - Each cache has bus to connect to memory

     - CPU inter-connect

       ![image-20200523223100829](lec20.assets/image-20200523223100829.png)

   - Shared => CMP

     - Share L2 cache
     - One CPU => many cores
     - CPU intra-connect

3. NUMA architecture

   ![image-20200523191310579](lec20.assets/image-20200523191310579.png)

   - Each node can be SMP architecture
   - Each core access memory with different latency.. => Non uniform memory access
   - CPU interconnect with other CPUs..

   ![image-20200523223337457](lec20.assets/image-20200523223337457.png)

4. Comparison
   ![image-20200523191548294](lec20.assets/image-20200523191548294.png)

   - All share-memory
   - But **may with different latency**..
   - Memory hierarchy comes to help!

   - In NUMA => 0-hop, 1-hop, 2-hop (different latency) (4x difference)

5. How to make OS better for multicore + NUMA platform?
   ![image-20200523192102324](lec20.assets/image-20200523192102324.png)

   - **Can traditional abstraction scales to multiple processors**?

   32Core + 4NUMA
   ![image-20200523192217660](lec20.assets/image-20200523192217660.png)

   - In one core => Linux is the best
   - Solaris => scales well ..
   - Time = user time + system time + idle time

6. Different benchmark
   ![image-20200523192649850](lec20.assets/image-20200523192649850.png)

   - 操作系统中保护**共享数据结构**的**同步原语**是影响可扩展性的重要因素
   - **锁竞争**可能导致可扩展性随着核数的增加而下降(**锁颠簸**现象)

7. What’s scalability?
   Amdahl’s law
   ![image-20200523192937928](lec20.assets/image-20200523192937928.png)

   ![image-20200523192944422](lec20.assets/image-20200523192944422.png)

   - Practical curve => **sync cost >> parallel speedup**


8. Locking / mutex / synchronization

   ![image-20200523193039305](lec20.assets/image-20200523193039305.png)

   Various types of synchronization techniques used by the Linux
   ![image-20200523193104782](lec20.assets/image-20200523193104782.png)

   

## Cache coherence

### What’s cache coherence?

1. Cache

   - Write back / write through
   - Write allocate / Write non-allocate

   ![image-20200524130546262](lec20.assets/image-20200524130546262.png)

   

2. Baseline system model
   ![image-20200523211507346](lec20.assets/image-20200523211507346.png)

   ![image-20200523211545169](lec20.assets/image-20200523211545169.png)

   - DMA => no cache !

3. **Hardware-based cache coherence**
   - Provide **a consistent view of memory across the machine**.
   - Read will **get the result of the last write to the memory hierarchy**
4. A real CPU
   ![image-20200524131150934](lec20.assets/image-20200524131150934.png)
   
   - n outstanding loads (stores): **ability to track n loads (stores) at the same time**, but it doesn't mean it happens in 1 cycle
5. Example of incoherence
   ![image-20200523211820537](lec20.assets/image-20200523211820537.png)

   Define of coherence
   ![image-20200523212010209](lec20.assets/image-20200523212010209.png)

   如果一个CPU 缓存了某块内存，那么在其他CPU 修改这块内存的时候，我们希望得到通知。**我们拥有多组缓存的时候，真的需要它们保持同步**。或者说，系统的内存在各个CPU 之间无法做到与生俱来的同步，我们实际上是需要一个大家都能遵守的方法来达到同步的目的

   ![image-20200523212248869](lec20.assets/image-20200523212248869.png)

	Naive Solution: Share one cache
   ![image-20200523212433942](lec20.assets/image-20200523212433942.png)
   ![image-20200524145718778](lec20.assets/image-20200524145718778.png)

## Intuition of shared memory

1. Intuitive expectation of shared memory
   ![image-20200524143420898](lec20.assets/image-20200524143420898.png)

   - The reason why DMA is an exception in uniprocessor is that **DMA will access memory via bus ignoring the value in cache**.

   - Coherence is an issue even in a single CPU system
     ![image-20200524143617857](lec20.assets/image-20200524143617857.png)
     - Cache coherence will be a problem **when different components connected to the memory hierarchy have different perceptions of what memory is at some certain time. A processor will include cache data in its perception of memory, an I/O device using DMA will not.** 

2. Problems with this intuition

   ![image-20200524144234841](lec20.assets/image-20200524144234841.png)
   
3. Definition of coherence
   ![image-20200524144516031](lec20.assets/image-20200524144516031.png)

   - We want a coherence view !
   - Another definition => coherence (said differently)
     ![image-20200524144935538](lec20.assets/image-20200524144935538.png)
   - **Sufficiently separated**
     - **it depends on the coherence implementation used**, so "sufficiently separated" should mean long enough for P2's cache to communicate the fact that there was a write to P1's cache. If P1 reads before that information is communicated, then **the system is still coherent as long as all processors "observe" the same order**

4. Write serialization
   ![image-20200524145309562](lec20.assets/image-20200524145309562.png)

   - In distributed systems, these consistency issues are very common as well, but in those situations they are solved by majority vote algorithms, such as Paxos. **In this case however, we will not be to use such algorithms because the overhead of having processors communicate P2P will be very high, and we would rather use the interconnect**.

5. Implementing cache coherence
   ![image-20200524145540245](lec20.assets/image-20200524145540245.png)

   - **Not uncachable, but rather as paged out**. Like you said, this will trigger a page fault and the OS can intervene and issue the appropriate network communication to execute the access.

6. Snooping cache coherence mechanisms
   ![image-20200524145830035](lec20.assets/image-20200524145830035.png)

7. The cache coherence logic

   - Each processor’s cache controller in response to
     - Loads and stores by the local processor
     - Message it receives from other caches
   - Cache coherence with write back caches
     ![image-20200524151012787](lec20.assets/image-20200524151012787.png)

### MSI / MESI

1. Solution2: MSI
   ![image-20200523212521619](lec20.assets/image-20200523212521619.png)

   ![image-20200523212542121](lec20.assets/image-20200523212542121.png)

   - Snoop => bus
   - Directory => NUMA
   - Reduce invalidate number ! 

   - MSI write back invalidation protocol
     ![image-20200524151153260](lec20.assets/image-20200524151153260.png)
   
     ![image-20200524151227452](lec20.assets/image-20200524151227452.png)
   
   - Summary of MSI
     ![image-20200524151639098](lec20.assets/image-20200524151639098.png)
   
   - MSI satisfies coherence
     ![image-20200524151827941](lec20.assets/image-20200524151827941.png)
   
2. MSEI protocol
   ![image-20200524152012371](lec20.assets/image-20200524152012371.png)

   MESI state transition diagram
![image-20200524152057477](lec20.assets/image-20200524152057477.png)
   

   
   ![image-20200524105140570](lec20.assets/image-20200524105140570.png)

   - MSI : read + write => two transactions
   
- MSEI: read + write => one transaction
  
- Each cache in the following states
     ![image-20200524123542230](lec20.assets/image-20200524123542230.png)

   - Clear explanation
  ![image-20200524123839272](lec20.assets/image-20200524123839272.png)
  
  
  

![image-20200524105533723](lec20.assets/image-20200524105533723.png)

![image-20200524105554845](lec20.assets/image-20200524105554845.png)

![image-20200524105736032](lec20.assets/image-20200524105736032.png)

![image-20200524105900635](lec20.assets/image-20200524105900635.png)

   ![image-20200524105947091](lec20.assets/image-20200524105947091.png)

   ![image-20200524110156358](lec20.assets/image-20200524110156358.png)

   ![image-20200524111108676](lec20.assets/image-20200524111108676.png)

   ![image-20200524111133615](lec20.assets/image-20200524111133615.png)

![image-20200524111211373](lec20.assets/image-20200524111211373.png)

![image-20200524111312277](lec20.assets/image-20200524111312277.png)

![image-20200524111330309](lec20.assets/image-20200524111330309.png)

- We need to insert `memory barrier` instruction..

### More advanced protocol

1. Low level choices
   ![image-20200524152502483](lec20.assets/image-20200524152502483.png)
2. Increasing efficiency
   ![image-20200524152532022](lec20.assets/image-20200524152532022.png)

### Update based coherence

1. Difference between invalidation based protocol
   ![image-20200524192207512](lec20.assets/image-20200524192207512.png)
   - The basic idea of update-based protocols is to **give everybody else the latest value**, so that **they don't need to reload them from main memory**, which is different from invalidation-based protocols.
2. Dragon write back update protocol
   ![image-20200524192311139](lec20.assets/image-20200524192311139.png)
3. State transition diagram
   ![image-20200524192430053](lec20.assets/image-20200524192430053.png)
4. Invalidate VS. Update based protocols
   ![image-20200524192729778](lec20.assets/image-20200524192729778.png)
   - Update based protocols tend to **occupy much higher bandwidth** than invalidate protocols.
5. ![image-20200524193024135](lec20.assets/image-20200524193024135.png)
   - **Update is not necessarily better**. **It looks like it is better because updating makes sure there aren't as many misses**, since the most recent data is in the cache. But, in update the cache will always want to be full, which leads to updates having to be communicated across the processors, **which can be expensive**.
6. Compare traffic
   ![image-20200524193210448](lec20.assets/image-20200524193210448.png)
   - Consider following two scenarios as bad for **update-based** protocol
     - **The updated value is never read again from the various processors whose cache lines were updated**.
     - **There are a lot of subsequent updates before the other processors read from the cache line**.

### Back to reality

1. Multi-level cache hierarchies
   ![image-20200524193435035](lec20.assets/image-20200524193435035.png)

   - **A multilevel cache hierarchy is a powerful tool to keep important/commonly accessed data close to the core using it**, however it can cause problems and increase overhead when it comes to maintaining cache coherence.

2. Inclusion property
   ![image-20200524193553692](lec20.assets/image-20200524193553692.png)

   ![image-20200524193635078](lec20.assets/image-20200524193635078.png)

3. Maintaining inclusion: **handling invalidations**
   ![image-20200524194733289](lec20.assets/image-20200524194733289.png)

   Maintaining inclusion: **L1 write hit**
   ![image-20200524194827278](lec20.assets/image-20200524194827278.png)

4. HW implications of implementing coherence
   ![image-20200524195027624](lec20.assets/image-20200524195027624.png)

5. NVIDIA GPUs don’t implement cache coherence
   ![image-20200524195211246](lec20.assets/image-20200524195211246.png)

### Implication of cache coherence to the programmer

1. Artificial communication via false sharing
   ![image-20200524195559185](lec20.assets/image-20200524195559185.png)

   - **The second version is better because it reduces false sharing between threads because each thread will now have a counter on its own cache line**. In the top version many of the counters share the same cache line so **when a thread updates its counter the invalidation of that line must be broadcast to all other threads** and they must then get the updated data before making their own changes.
   - And I think in order to avoid false sharing and at the same time exploit the benefit of locality, we need to put contents that will be sequentially accessed by one thread on the same cache line while putting contents that will be accessed by different threads on separate cache lines.

2. Demo false sharing
   ![image-20200524200110738](lec20.assets/image-20200524200110738.png)

   False sharing explain
   ![image-20200524202110057](lec20.assets/image-20200524202110057.png)

3. Impact of cache line size on missing rate
   ![image-20200524202356791](lec20.assets/image-20200524202356791.png)

   ![image-20200524202428488](lec20.assets/image-20200524202428488.png)

4. Summary snooping-based coherence
   ![image-20200524202514253](lec20.assets/image-20200524202514253.png)

### Directory based coherence

1. Problems we need to consider
  
- What limits the scalability of snooping-based cache coherence protocols?
     - **All processors/caches much communicate via the interconnect**. The interconnect traffic limits the scalability of snooping based approaches.
- How a directory based scheme avoid this problem?
   - How can the storage overhead of directory structure be reduced?
     - limited pointer schemes
     - sparse directories
   
2. One naive solution
   ![image-20200525100411755](lec20.assets/image-20200525100411755.png)

   ![ ](lec20.assets/image-20200525100551533.png)

   

3. Problem: scaling cache coherence to large machines
   ![image-20200525091542483](lec20.assets/image-20200525091542483.png)

4. NUMA systems nowadays are normal

   - Multi-socket intel systems
     ![image-20200525092120468](lec20.assets/image-20200525092120468.png)
   - Intel’s ring interconnect (on one chip)
     ![image-20200525092148953](lec20.assets/image-20200525092148953.png)

5. Scalable cache coherence using directories
   ![image-20200525092249742](lec20.assets/image-20200525092249742.png)

   - **Directory-based vs. snooping-based cache coherence is an instance of the space-time tradeoff**. By designating directory to cache lines, directory-based coherence uses additional space to avoid the inefficient broadcasting and high latency.

6. Implementing cache coherence on share-bus
   ![image-20200525091354560](lec20.assets/image-20200525091354560.png)

7. A very simple directory
   ![image-20200525092730414](lec20.assets/image-20200525092730414.png)

8. A distributed directory
   ![image-20200525092805862](lec20.assets/image-20200525092805862.png)

   ![image-20200525093156385](lec20.assets/image-20200525093156385.png)

   ![image-20200525093219823](lec20.assets/image-20200525093219823.png)

   ![image-20200525093316097](lec20.assets/image-20200525093316097.png)

9. Advantages of directories
   ![image-20200525094027618](lec20.assets/image-20200525094027618.png)

10. Cache invalidation patterns
   ![image-20200525094114419](lec20.assets/image-20200525094114419.png)

   In general, only a few shared during a write
   ![image-20200525094408060](lec20.assets/image-20200525094408060.png)

11. Directory based coherence
    ![image-20200524124003379](lec20.assets/image-20200524124003379.png)

    

1. How big is the directory?
   ![image-20200525094502227](lec20.assets/image-20200525094502227.png)

   Full-bit vector representation
   ![image-20200525094532394](lec20.assets/image-20200525094532394.png)

#### Reducing storage overhead of directory

1. Full-bid vector => huge storage overhead
   ![image-20200531112254918](lec20.assets/image-20200531112254918.png)

   - Increase cache line size => painful false sharing

2. Limited pointer schemes
   ![image-20200531112441379](lec20.assets/image-20200531112441379.png)

   Managing overflow of limited pointer scheme
   ![image-20200531112719646](lec20.assets/image-20200531112719646.png)

   Optimize for the common case
   ![image-20200531112835440](lec20.assets/image-20200531112835440.png)

   - This slide illustrates a key idea in Systems programming: **your solution should not be driven by its theoretical properties but rather by an careful analysis of the workload of the system**. Following the same idea, one should always **try to keep his implementations simple when a complex scheme doesn't bring extra performance**.

3. Sparse directories
   ![image-20200531113038835](lec20.assets/image-20200531113038835.png)

   ![image-20200531113106957](lec20.assets/image-20200531113106957.png)

    - Why doubly linked list?
      	- Node deletion in doubly linked list only takes O(1).
      	- On write, invalidation needs to be propagated to both directions.
   - Scaling properties
     ![image-20200531113604407](lec20.assets/image-20200531113604407.png)

   Recall: write miss in full bit vector scheme
   ![image-20200531113741569](lec20.assets/image-20200531113741569.png)

4. Optimizing directory based coherence
   ![image-20200531113834084](lec20.assets/image-20200531113834084.png)
   - Limited pointer schemes take advantage of the fact that the data is probably only in a few caches at once.
   - Sparse directories take advantage of the fact that most of memory is not resident in the cache, and the coherence protocol only needs to worry about sharing information for lines that are currently in cache.

5. Recall: read miss to dirty line
   ![image-20200531114053038](lec20.assets/image-20200531114053038.png)

6. Intervention forwarding
   ![image-20200531114211253](lec20.assets/image-20200531114211253.png)

   ![image-20200531114245216](lec20.assets/image-20200531114245216.png)

   		- In the normal directory based protocol, the home node responds to the requesting node with the owner's id. The requesting node then requests the data from the owner, which sends the data to both the home node and the requesting node. In the intervention forwarding optimization, however, the home node acts as a middleman by requesting data from the owner node, getting the response, and sending the response back to the requesting node.

7. Request forwarding
   ![image-20200531114451552](lec20.assets/image-20200531114451552.png)

8. Intel core i7 CPU
   ![image-20200531114611414](lec20.assets/image-20200531114611414.png)

   - Only L3 cache and inclusion properties
   - Ring connection

9. Coherence in multi-socket intel systems
   ![image-20200531115140517](lec20.assets/image-20200531115140517.png)

10. Xeon Phi
    ![image-20200531115315112](lec20.assets/image-20200531115315112.png)

    ![image-20200531115351927](lec20.assets/image-20200531115351927.png)

    ![image-20200531115441140](lec20.assets/image-20200531115441140.png)

    ![image-20200531115525336](lec20.assets/image-20200531115525336.png)

11. Summary: directory based coherency
    ![image-20200531115605422](lec20.assets/image-20200531115605422.png)



### A real CPU

2. **Hardware-based cache coherence**
   - Provide **a consistent view of memory across the machine**.
   - Read will **get the result of the last write to the memory hierarchy**
3. A real CPU
   ![image-20200524131150934](lec20.assets/image-20200524131150934.png)
   - n outstanding loads (stores): **ability to track n loads (stores) at the same time**, but it doesn't mean it happens in 1 cycle

## A Basic Snooping-Based Multi-Processor Implementation

1. Different between cache coherence protocols and implementation
   - Cache coherence protocol
     - What messages / transactions needed to be sent
     - Messages / transactions were atomic
2. The goals of our coherence implementation
   - Be **correct**
   - Achieve **high performance**
   - **Minimize cost** (minimize amount of extra hardware needed to implement coherence)

3. What we should know?
   ![image-20200603224626745](lec20.assets/image-20200603224626745.png)

   - An interconnect is the network connecting a number of clients in a parallel machine (**processors**, **caches**, **memories**)
   - A bus is **a particular type of interconnect**

   - Deadlock, livelock => program correctness

     - Required conditions for deadlock
       ![image-20200603225439374](lec20.assets/image-20200603225439374.png)

     - Livelock
       ![image-20200603230044364](lec20.assets/image-20200603230044364.png)
     - A difference between livelock and deadlock
       - A live lock is different from a dead lock in that, in a dead lock, everyone is simply waiting and no one is doing anything; in a live lock, at least some one is doing something, but the whole system is not making program.

   - Starvation => an issue of fairness

     - Starvation
       ![image-20200603230213941](lec20.assets/image-20200603230213941.png)

### A basic implementation of snooping (assuming an atomic bus)

1. Consider a basic system design
   ![image-20200603230406549](lec20.assets/image-20200603230406549.png)

   - **An atomic bus allows only one channel at a time to send a message**, but everyone connected to the bus can see the message.

2. Transactions on an atomic bus
   ![image-20200603230517607](lec20.assets/image-20200603230517607.png)

   - With this definition of an atomic bus, the "atom" includes the client who is granted access placing its command and data on the bus **and** other clients placing their responses on the bus. Within this atom the bus is "locked" to any other bus transaction.

3. Cache miss logic on a uniprocessor
   ![image-20200603230612167](lec20.assets/image-20200603230612167.png)

4. Multi-processor cache controller behavior
   ![image-20200603230812124](lec20.assets/image-20200603230812124.png)

   ![image-20200603231029038](lec20.assets/image-20200603231029038.png)

5. Reporting snoop results
   ![image-20200603231149915](lec20.assets/image-20200603231149915.png)

   Reporting snoop results: how
   ![image-20200603231216499](lec20.assets/image-20200603231216499.png)
   Reporting snoop results: when
   ![image-20200603231335419](lec20.assets/image-20200603231335419.png)

6. Handling write back of dirty cache lines
   ![image-20200604073730847](lec20.assets/image-20200604073730847.png)

7. Cache with write-back buffer
   ![image-20200604074016751](lec20.assets/image-20200604074016751.png)

8. In practice state transitions are not atomic
   ![image-20200604074324195](lec20.assets/image-20200604074324195.png)

   - An example of race condition
     ![image-20200604074451639](lec20.assets/image-20200604074451639.png)

   - Fetch deadlock
     ![image-20200604074612057](lec20.assets/image-20200604074612057.png)
   - Livelock
     ![image-20200604074809854](lec20.assets/image-20200604074809854.png)

9. Reminder: memory coherence
   ![image-20200604074925146](lec20.assets/image-20200604074925146.png)

10. Self check: when does a write commit?
    ![image-20200604075036760](lec20.assets/image-20200604075036760.png)

    - write-back buffer doesn't effect time of commit, this is because all processors will check the value in the write-back buffer before they check the memory, so if a processor successfully get to M state then modify a cache line, it will always be observed by other processors.

11. Starvation
    ![image-20200604080528147](lec20.assets/image-20200604080528147.png)

12. Design issues
    ![image-20200604080654602](lec20.assets/image-20200604080654602.png)

13. First-half summary
    ![image-20200604080812275](lec20.assets/image-20200604080812275.png)

### Building the system around non-atomic bus transactions

1. An atomic bus system creates a bottleneck in bus arbitration
   ![image-20200604081015723](lec20.assets/image-20200604081015723.png)

   - An atomic bus system creates a bottleneck in bus arbitration. In that any request a processor makes will probably have some time before it is served in which nothing else is able to be done on the bus. Since the bus is used by all processors and is a valuable resource, this is **wasting a lot of important time**.

2. Review: transaction on an atomic  bus
   ![image-20200604081123788](lec20.assets/image-20200604081123788.png)

3. Split transaction bus
   ![image-20200604081154289](lec20.assets/image-20200604081154289.png)

   - The atomic bus acted as a bottleneck. So we want to be able to perform operations in between the request and response. This allows room for race conditions however. Since the atomic bus bottlenecked the timeline, it forced a serial ordering on the operations.

4. New issues arise due to split transactions
   ![image-20200604081251779](lec20.assets/image-20200604081251779.png)

5. A basic design
   ![image-20200604081348347](lec20.assets/image-20200604081348347.png)

6. Initiating a request
   ![image-20200604081416549](lec20.assets/image-20200604081416549.png)

7. Read miss: cycle-by-cycle bus behavior
   ![image-20200604081513610](lec20.assets/image-20200604081513610.png)
   ![image-20200604081724092](lec20.assets/image-20200604081724092.png)

   ![image-20200604081810404](lec20.assets/image-20200604081810404.png)

8. Pipelined transactions
   ![image-20200604081856089](lec20.assets/image-20200604081856089.png)

   ![image-20200604081952683](lec20.assets/image-20200604081952683.png)

9. Key issues to resolve
   ![image-20200604082026608](lec20.assets/image-20200604082026608.png)

   ![image-20200604082157580](lec20.assets/image-20200604082157580.png)

   ![image-20200604082226413](lec20.assets/image-20200604082226413.png)

10. Why do we have queues in a parallel system
    ![image-20200604082328845](lec20.assets/image-20200604082328845.png)

11. Multi-level cache hierarchies
    ![image-20200604082552889](lec20.assets/image-20200604082552889.png)

    Recall the fetch-deadlock problem
    ![image-20200604082615118](lec20.assets/image-20200604082615118.png)

    ![image-20200604082705698](lec20.assets/image-20200604082705698.png)

    ![image-20200604082800413](lec20.assets/image-20200604082800413.png)

    ![image-20200604082827426](lec20.assets/image-20200604082827426.png)

12. ![image-20200604082947360](lec20.assets/image-20200604082947360.png)

## Memory Consistency

### Introduction

1. Cache
   ![image-20200612091007336](lec20.assets/image-20200612091007336.png)

   Memory Consistency Model
   ![image-20200612091035475](lec20.assets/image-20200612091035475.png)

2. Coherence concerns accesses only a **single memory location**

   - Making sure that caches & **stale copies** do not cause problems
   - Two invariants: **single-writer multiple-readers**, **data-value**

   Consistency concerns ordering for **accesses to many locations**

   - Making sure that ordering does not cause problems

   Comparison
   ![image-20200613221530799](lec20.assets/image-20200613221530799.png)

   ![image-20200613221703718](lec20.assets/image-20200613221703718.png)

   

3. Why we need cache coherence and why relaxed memory consistency?
   ![image-20200613222351478](lec20.assets/image-20200613222351478.png)

   - Aggressive memory operation ordering
     ![image-20200613222509343](lec20.assets/image-20200613222509343.png)
   - 

4. Synchronization
   ![image-20200612091340570](lec20.assets/image-20200612091340570.png)

   Atomic instructions
   ![image-20200612091403821](lec20.assets/image-20200612091403821.png)

   - Test & Set algorithm
     ![image-20200612091528819](lec20.assets/image-20200612091528819.png)

   Improving performance of Sync
   ![image-20200612091658392](lec20.assets/image-20200612091658392.png)

5. An example of memory ordering
   ![image-20200612101823689](lec20.assets/image-20200612101823689.png)

   - Seeing update of counter before update of mutex ..

6. Memory consistency model
   ![image-20200612091858207](lec20.assets/image-20200612091858207.png)

7. Sequential Consistency
   ![image-20200612092104826](lec20.assets/image-20200612092104826.png)

   Define the notion of order
   ![image-20200612092430578](lec20.assets/image-20200612092430578.png)

   Last in memory order
   ![image-20200612092557229](lec20.assets/image-20200612092557229.png)

8. Sequential Consistency affects performance ...
   ![image-20200612092816649](lec20.assets/image-20200612092816649.png)

   ![image-20200612092841698](lec20.assets/image-20200612092841698.png)

   ![image-20200612092929566](lec20.assets/image-20200612092929566.png)

   ![image-20200612092958083](lec20.assets/image-20200612092958083.png)

   ![image-20200612093039338](lec20.assets/image-20200612093039338.png)

   Realistic Memory Consistency Mode: modern hardware features can interfere with store order

   - **write buffer** (or store buffer or write-behind buffer)
     ![image-20200612102449888](lec20.assets/image-20200612102449888.png)
     - Total order storing: Stores are guaranteed to **occur in FIFO order**
     - Effect of write buffer: sometimes load **appears ** bypass store
       ![image-20200612102701514](lec20.assets/image-20200612102701514.png)
   - **instruction reordering** (out-of-order execution)
   - **superscalar execution** and pipelining

   Each CPU/core keeps its own execution consistent, but how is it viewed by others?

9. Total Store Order Consistency
   ![image-20200612093224557](lec20.assets/image-20200612093224557.png)

   - Fence <-> memory barrier
     ![image-20200612102743068](lec20.assets/image-20200612102743068.png)
   - Atomic operations ..
     ![image-20200612102829765](lec20.assets/image-20200612102829765.png)

10. QUIZ
    ![image-20200612093422733](lec20.assets/image-20200612093422733.png)

    - Fence => S1 before L1 and S2 before L2 ...

11. TSO implementation
    ![image-20200612093708464](lec20.assets/image-20200612093708464.png)

12. Partial store ordering (ARM)
    ![image-20200612102925454](lec20.assets/image-20200612102925454.png)

    ![image-20200612103015336](lec20.assets/image-20200612103015336.png)

13. Memory Fence

    - Example
      ![image-20200613222810925](lec20.assets/image-20200613222810925.png)

    Acquire and release semantics
    ![image-20200613222949159](lec20.assets/image-20200613222949159.png)

    C++11 Atomic T
    ![image-20200613223201770](lec20.assets/image-20200613223201770.png)

    

14. Conflicting data access
    ![image-20200613223253775](lec20.assets/image-20200613223253775.png)

    ![image-20200613223318845](lec20.assets/image-20200613223318845.png)

    Relaxed consistency
    ![image-20200613223412321](lec20.assets/image-20200613223412321.png)

    Parallel Technique
    ![image-20200613223616869](lec20.assets/image-20200613223616869.png)
    ![image-20200613223647072](lec20.assets/image-20200613223647072.png)

    

15. Even more relaxed ..
    ![image-20200612093839838](lec20.assets/image-20200612093839838.png)

16. Memory consistency hardware takeaway
    ![image-20200612103103914](lec20.assets/image-20200612103103914.png)

    ![image-20200612103128213](lec20.assets/image-20200612103128213.png)

17. Some locks

    - Spinlock
      ![image-20200612103326737](lec20.assets/image-20200612103326737.png)

      Spinning versus switching
      ![image-20200612103423726](lec20.assets/image-20200612103423726.png)

      ![image-20200612103444994](lec20.assets/image-20200612103444994.png)

    - Interrupt Disabling
      ![image-20200612103522305](lec20.assets/image-20200612103522305.png)

    - Alternative to spinning

      - Conditional Lock (tryLock)
        ![image-20200612103714916](lec20.assets/image-20200612103714916.png)

      - Limited time lock
        ![image-20200612103740870](lec20.assets/image-20200612103740870.png)

      - Common multiprocessor spin lock
        ![image-20200612103805316](lec20.assets/image-20200612103805316.png)

      - Compare two simple spinlocks

        ```c++
        // Test and Set
        void lock (volatile lock_t *l) {
        while (test_and_set(l)) ;
        }
        // Test and Test and Set
        void lock (volatile lock_t *l) {
        while (*l == BUSY || test_and_set(l)) ;
        }
        ```

        - The second lock
          - Avoid bus traffic contention caused by test_and_set until it is likely to succeed
          - Normal read spins in cache 
        - Benchmark results
          ![image-20200612104129220](lec20.assets/image-20200612104129220.png)

    - MCS locks
      ![image-20200612104550937](lec20.assets/image-20200612104550937.png)

      


## Back to the real world

1. Ways to achieve synchronize with
   ![image-20200712103744864](lec20.assets/image-20200712103744864.png)

2. Real hardware & Compiler

   - Real hardware doesn’t run the code that you wrote.
   - Real compiler doesn’t produce the code that you wrote.

   ![image-20200712104013758](lec20.assets/image-20200712104013758.png)

   Weak & Strong
   ![image-20200712104152024](lec20.assets/image-20200712104152024.png)

   In TSO, both 0 is valid
   ![image-20200712104924428](lec20.assets/image-20200712104924428.png)

   The problem of data race
   ![image-20200712105019967](lec20.assets/image-20200712105019967.png)

   ![image-20200712105142275](lec20.assets/image-20200712105142275.png)

   - Read between the 2 write 16 bits ...

## C++11 Memory Model

1. History
   ![image-20200712110056987](lec20.assets/image-20200712110056987.png)

2. Why C++11 Memory Model
   ![image-20200712110230412](lec20.assets/image-20200712110230412.png)

3. Happens before

   - An important **fundamental concept** in understanding the memory model

   - A guarantee that memory writes by **one specific statement are visible to another specific statement**

   - Use memory fence to ensure this order

     ```c++
     int A, B;
     void foo()
     {
     	A = B + 1;
     	asm volatile("" ::: "memory");
     	B = 0;
     }
     ```

   - Atomic

     ```c++
     //x86
     #define COMPILER_BARRIER() asm volatile("" ::: "memory")
     //PowerPC
     #define RELEASE_FENCE() asm volatile("lwsync" ::: "memory")
     ==============================================
     int Value;
     std::atomic<int> IsPublished(0);
     void sendValue(int x)
     {
       Value = x;
       // <-- reordering is prevented here!
       IsPublished.store(1, std::memory_order_release);
     }
     ```

     ![image-20200712111817533](lec20.assets/image-20200712111817533.png)

     ![image-20200712111929189](lec20.assets/image-20200712111929189.png)

4. `std::memory_order`
   ![image-20200712115225414](lec20.assets/image-20200712115225414.png)

   - `std::memory_order_seq_cst`
     ![image-20200712115417131](lec20.assets/image-20200712115417131.png)

     

   ![image-20200712115942286](lec20.assets/image-20200712115942286.png)

   - `std :: memory_order_release/acquire`

     - Release store & Acquire load
       ![image-20200712120201036](lec20.assets/image-20200712120201036.png)

       ![image-20200712121119252](lec20.assets/image-20200712121119252.png)

     - Acquire semantic
       ![image-20200712121314554](lec20.assets/image-20200712121314554.png)

     - Release semantic
       ![image-20200712121546535](lec20.assets/image-20200712121546535.png)

     ![image-20200712121628511](lec20.assets/image-20200712121628511.png)

     On different architecture
     ![image-20200712123657889](lec20.assets/image-20200712123657889.png)

     ![image-20200712123744557](lec20.assets/image-20200712123744557.png)

5. `std::memory_order_consume`

   - Data dependency ordering
     ![image-20200712221836677](lec20.assets/image-20200712221836677.png)

     ![image-20200712221935448](lec20.assets/image-20200712221935448.png)

     ![image-20200712222138835](lec20.assets/image-20200712222138835.png)

   - On different architecture
     ![image-20200712222324378](lec20.assets/image-20200712222324378.png)

     ![image-20200712222432359](lec20.assets/image-20200712222432359.png)

     ![image-20200712222644481](lec20.assets/image-20200712222644481.png)

     ![image-20200712222704592](lec20.assets/image-20200712222704592.png)

   - Real world: RCU
     ![image-20200712223232439](lec20.assets/image-20200712223232439.png)

6. `std::memory_order_relaxed`

   ![image-20200712223325564](lec20.assets/image-20200712223325564.png)

## Lock Free Programming

1. What’s lock free programming
   ![image-20200712223837439](lec20.assets/image-20200712223837439.png)

2. Some details
   ![image-20200712223859747](lec20.assets/image-20200712223859747.png)

3. Lock Free Stack

   ```
   template<typename T>
   class lock_free_stack
   {
   private:
     struct node
     {
       T data;
       node* next;
       node(T const& data_): // 1
       data(data_)
       {}
     };
     std::atomic<node*> head;
     public:
     void push(T const& data)
     {
       node* const new_node=new node(data); // 2
       new_node->next=head.load(); // 3
       while(!head.compare_exchange_weak(new_node->next,new_node)); // 4
     }
     void pop(T& result)
     {
       node* old_head=head.load();
       while(!head.compare_exchange_weak(old_head,old_head->next));
       result=old_head->data;
     }
   }
   ```

   

## Reference

1. [An interesting cache coherence simulator](https://github.com/srikantaggarwal/Cache-Coherence)

   

