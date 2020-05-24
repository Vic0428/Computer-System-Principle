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

   - Each node can SMP architecture
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

   - **The second version is better because it reduces false sharing between threads because each thread will now have a counter on its own cache line**. In the top version many of the counters share the same cache line so when a thread updates its counter the invalidation of that line must be broadcast to all other threads and they must then get the updated data before making their own changes.
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

1. Directory based coherence
   ![image-20200524124003379](lec20.assets/image-20200524124003379.png)

   

7. **Hardware-based cache coherence**
   - Provide **a consistent view of memory across the machine**.
   - Read will **get the result of the last write to the memory hierarchy**
8. A real CPU
   ![image-20200524131150934](lec20.assets/image-20200524131150934.png)
   
   - n outstanding loads (stores): **ability to track n loads (stores) at the same time**, but it doesn't mean it happens in 1 cycle

## Reference

1. [An interesting cache coherence simulator](https://github.com/srikantaggarwal/Cache-Coherence)

   

