# Lecture19: Storage Technique

## Introduction

### When you choose a storage system

1. **Common storage media**
  - SRAM
  - DRAM
  - Disk (HDD)
  - SSD (Flash memory)
  - NVM
  - PCM
  - RRAM
  - MRAM
  - CD
  - DVD
  - Tape

2. **From software level** (storage system)
  - Key-value , file system

  - Distributed system

  - Reliability, consistency, fault tolerance, replication
  
	- Consistency and reliability
	
	- File access mode (big file. Small file and access frequency)
3. **Which storage system you choose**?
  - SAN
    - Storage area network
    - Block granularity
  - NAS
    - File granularity
  - HDFS
    - Open source version of Google file system

4. How large do you need?
  - 1TB
  - 10TB
  - 1PB
  - 100PB


5. I/O 500 system

   - Data + metadata
   - SSD + local / distributed file system
   - NVM (3D XPoint)
     - 128GB - 512GB
     - High performance

4. From CPU-intensive computing => **data-intensive** computing

   - Data size growth >> storage size growth

### What’s storage system?

5. Computer system

   - CPU
   - IO system
   - Storage system
     - Disk 
     - memory
     - Register

   GAP between IO and CPU
   ![image-20200522110333307](lec19.assets/image-20200522110333307.png)

   - Memory bandwidth and memory bandwidth => bad
   - Network latency and Disk latency => worse

6. What are storage system all about?

   - Memory / storage hierarchy

     - Combining many techniques to balance costs/benefits
     - Similar to cache design for CPUs, but Important differences
     - **Exploit locality** to get the best of both worlds
       - Locality = re-use / nearness of accesses
       - Allows most accesses to use small, fast memory
     - No longer the focal point of storage system design
       - Still important though

     ![image-20200522110652802](lec19.assets/image-20200522110652802.png)

     ![image-20200522110848724](lec19.assets/image-20200522110848724.png)

     - SRAM big area, but high performance (compared to DRAM)
     - NVM between DRAM and local disks

     

7. Levels in typical memory hierarchy
   ![image-20200522111040813](lec19.assets/image-20200522111040813.png)

   - NVM: 300ns
   - SSD: 30 micro second

   Memory access process
   ![image-20200522113814036](lec19.assets/image-20200522113814036.png)

   - Between DRAM and CPU cache => hardware
   - Between DRAM and HDD => software

   Logical Program Addressing
   ![image-20200522114003383](lec19.assets/image-20200522114003383.png)

8. Persistency

   - Storing data for length periods of time
   - To be useful, it must be also possible to find it later again
     - This brings in data organization, consistency and management issues
   - This is where the serious action is
     - And it does relate to the memory / storage hierarchy

   Storage system interface
   ![image-20200522114351605](lec19.assets/image-20200522114351605.png)

9. Software interface layers
   ![image-20200522114525916](lec19.assets/image-20200522114525916.png)

   Organizing names: Directory Hierarchy
   ![image-20200522114629174](lec19.assets/image-20200522114629174.png)

   File system structure
   ![image-20200522114650606](lec19.assets/image-20200522114650606.png)

   OS sees storage as linear array of blocks
   ![image-20200522114751995](lec19.assets/image-20200522114751995.png)

   In device, blocks mapped to physical store (sector # => cylinder , patter, tack, sector)

   ![image-20200522114840231](lec19.assets/image-20200522114840231.png)

   Physical interconnects
   ![image-20200522115014877](lec19.assets/image-20200522115014877.png)

   Computer system components
   ![image-20200522115105317](lec19.assets/image-20200522115105317.png)

   Storage subsystem components
   ![image-20200522115127766](lec19.assets/image-20200522115127766.png)

   - System bus: PCIe
   - I/O Bus: SATA

10. Storage characteristic

    - Reliability (use redundant data)

      - Replication
        - HDFS: one data, three copy
      - Checksum

      ![image-20200522115316208](lec19.assets/image-20200522115316208.png)

    - Consistency

      - Atomic (All or none) + consistency

    - Fault tolerance and replication
      ![image-20200522115751211](lec19.assets/image-20200522115751211.png)

      Resolving single point of failure
      ![image-20200522115848276](lec19.assets/image-20200522115848276.png)

### History

1. Storage medial history
   ![image-20200522165407432](lec19.assets/image-20200522165407432.png)

   - Now flash memory & Persistent memory
   - Flash memory => SSD
   - Persistent memory => 3DXPoint (hot topic)

2. Technical specification - Then and Now
   ![image-20200522165933477](lec19.assets/image-20200522165933477.png)

   - 2013 => 3.5 in

   ![image-20200522165959382](lec19.assets/image-20200522165959382.png)

   - Hard disk drive area density
     ![image-20200522170149286](lec19.assets/image-20200522170149286.png)
   - Performance gap between disk and dram
     ![image-20200522170205233](lec19.assets/image-20200522170205233.png)
     - DRAM: 10ns
     - Disk: ms 
   - Interface: from parallel to serial
     ![image-20200522170325162](lec19.assets/image-20200522170325162.png)
     - Parallel: single skew is bad
     - Performance
       ![image-20200522170518433](lec19.assets/image-20200522170518433.png)

3. SSD
   ![image-20200522170550107](lec19.assets/image-20200522170550107.png)

   - Bandwidth 30x
   - Latency: 1 / 100

4. Solid State Storage
   ![image-20200522170704773](lec19.assets/image-20200522170704773.png)

   - HDD => mechanism
   - SSD => electronic 

   From 1 bit / cell => 2 bit / cell
   ![image-20200522170815170](lec19.assets/image-20200522170815170.png)

   - SSD NAND Flash

5. HDD VS. SSD
   ![image-20200522171008271](lec19.assets/image-20200522171008271.png)

   - SSD => read / write (huge latency difference)
   - Overwrite => Erase + write (characteristic)

   ![image-20200522171114063](lec19.assets/image-20200522171114063.png)

   SSD Market trends
   ![image-20200522171205121](lec19.assets/image-20200522171205121.png)

6. Networked storage
   ![image-20200522171414615](lec19.assets/image-20200522171414615.png)

7. Distributed file system

   ![image-20200522171851354](lec19.assets/image-20200522171851354.png)

8. Key-Value storage
   ![image-20200522171952462](lec19.assets/image-20200522171952462.png)

9. Cloud storage
   ![image-20200522172421823](lec19.assets/image-20200522172421823.png)

## Disk Technologies

Reference book: Memory systems, Cache, DRAM, Disk

1. The history of storage media
   ![image-20200625230407772](lec19.assets/image-20200625230407772.png)

   - HDD: magnetic
   - Flash memory ==> SSD
   - Persistent memory (NVM) => new break through

2. Today we have

   - HDD
   - Flash memory
   - Persistent Memory

3. The first commercial Disk Drive

   - Big size
   - Small storage

   First Air Bearing Heads (1962)

   First Removable Disk Drive (1965)

   First Modern Hard Disk Design (1973)

   - “Winchester”

   First Thin Film Heads (1979)

   Giant Magneto Resistive (1997)

### Disk components and their functions

1. Construction and Operation
   ![image-20200625232133186](lec19.assets/image-20200625232133186.png)
2. Moving-head Disk Mechanism
   ![image-20200625232536606](lec19.assets/image-20200625232536606.png)
3. Disk Device Components
   ![image-20200625232746914](lec19.assets/image-20200625232746914.png)
4. Logical Block Addressing
   ![image-20200625233042610](lec19.assets/image-20200625233042610.png)
   - CHS: Cylinder, Head, Sector

### Disk performance and how it is measured

1. Performance

   - Electromechanical device
     - Impacts the overall performance of the storage system

   - Disk service time

     - Time taken by a disk to complete an I/O request

       - Seek time (bottleneck1)
         ![image-20200625233507010](lec19.assets/image-20200625233507010.png)

         - Full stroke (from outermost to innermost)
         - Average..
         - From one track to another track

       - Rotational latency (bottleneck2)

         ![image-20200625233653177](lec19.assets/image-20200625233653177.png)

       - Data transfer time 

         - Data Transfer rate
           ![image-20200625234003254](lec19.assets/image-20200625234003254.png)
           - External rate: SATA version ..
             External rate : lower compared to internal rate (used to calc the transfer time

2. Controller overhead can be omitted...
   ![image-20200625234122577](lec19.assets/image-20200625234122577.png)

3. Where does the disk head’s time go?
   ![image-20200625234421139](lec19.assets/image-20200625234421139.png)

   - Rotational latency + seek time > accounts most of the time

4. Impact of request size
   ![image-20200625234555401](lec19.assets/image-20200625234555401.png)

   - Batch reading / writing

### Disk drive firmware algorithms



## Local strorage system

