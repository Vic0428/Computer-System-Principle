# SeL4

## Overview

The seL4 microkernel is the world’s most high-assured operating system kernel. 

## seL4 mechanisms

### Capabilities

#### Concept

[tutorial link](https://docs.sel4.systems/Tutorials/capabilities.html)

1. What is a capability?
   - A *capability* is a unique, unforgeable token that gives the possessor permission to access an entity or object in system
   - In seL4, capabilities to all resources controlled by seL4 are given to the root task on initialisation
2. CNode
   - A *CNode* (capability-node) is an object full of capabilities: you can think of a CNode as an array of capabilities.
   - We refer to slots as *CSlots* (capability-slots)
3. CSpaces
   - A *CSpace* (capability-space) is the full range of capabilities accessible to a thread, which may be formed of one or more CNodes. In this tutorial, we focus on the CSpace constructed for the root task by seL4’s initialisation protocol, which consists of one CNode.

#### Code

1. Calculate the number of `cnode_slots`

   ```c++
   // Calculate number of cnode slots
   size_t num_initial_cnode_slots = initial_cnode_object_size / (1u << seL4_SlotBits); 
   printf("The CSpace has %zu CSlots\n", num_initial_cnode_slots);
   ```

2. Copy a capability between `CSlots`

   ```c++
   seL4_CPtr first_free_slot = info->empty.start;
   seL4_Error error = seL4_CNode_Copy(seL4_CapInitThreadCNode, first_free_slot, 			   seL4_WordBits, seL4_CapInitThreadCNode, seL4_CapInitThreadTCB, seL4_WordBits, seL4_AllRights);
   ```

3. Delete a `capability`

   ```c++
   seL4_CNode_Delete(seL4_CapInitThreadCNode, last_slot, seL4_WordBits);
   ```

4. Invoke a `capability`

   ```c++
   seL4_TCB_Suspend(seL4_CapInitThreadTCB);
   ```

### Untyped

#### Concept

An introduction to physical memory management on seL4. [Link](https://docs.sel4.systems/Tutorials/untyped.html)

1. Physical memory
   - Apart from a small, static amount of kernel memory, **all physical memory is managed by user-level in an seL4 system**.
   - Capabilities to objects created by seL4 at boot, as well as the rest of the physical resources managed by seL4, **are passed to the root task on start up**.
2. Untyped
   - Excluding the objects used to create the root task, **capabilities to all available physical memory are passed to the root task as capabilities to *untyped* memory**
   - Untyped memory is **a block of contiguous physical memory with a specific size**.
   - Untyped objects **have a boolean property *device* which indicates whether the memory is writable by the kernel or not**
3. Retyping
   - Untyped capabilities have a single invocation: [seL4_Untyped_Retype](https://docs.sel4.systems/ApiDoc.html#retype) which is used to **create a new capability from an untyped**. 
   - Specifically, the new capability created by a retype invocation provides access to the a subset of the memory range granted by the original untyped, with a specific type. **New capabilities created by retyping an untyped object are referred to as *children* of that untyped object**.

#### Code

1. Create an untyped object

   ```c++
       /* Create a TCB in CSlot child_tcb */
       error = seL4_Untyped_Retype(child_untyped,
                                   seL4_TCBObject,
                                   seL4_TCBBits,
                                   seL4_CapInitThreadCNode,
                                   0,
                                   0,
                                   child_tcb,
                                   1);
   ```

2. Delete the object

   ```c++
   // Revoke the child untyped
   seL4_CNode_Revoke(seL4_CapInitThreadCNode, child_untyped, seL4_WordBits);
   ```

### Mapping

[Tutorial](https://docs.sel4.systems/Tutorials/mapping.html)

#### Concept

1. Virtual memory

   - **seL4 does not provide virtual memory management**, beyond kernel primitives for manipulating hardware paging structures. 
   - **User-level must provide services for creating intermediate paging structures, mapping and unmapping pages.**

2. Paging structure

   -  As part to the **booting process**, seL4 initializes **the root task** with a top-level hardware virtual memory object, which is referred to **VSpace**. 

   - A capability to this structure is made available in the `seL4_CapInitThreadVSpace` slot in the root tasks CSpace. 

   - In addition to the top-level paging structure, **intermediate hardware virtual memory objects are required to map pages**.

     ```c++
     // For x86_64
     seL4_PDPT seL4_PageDirectory seL4_PageTable
     ```

   - An example of mapping

     ```c++
     /* map a PDPT at TEST_VADDR */
     error = seL4_X86_PDPT_Map(pdpt, seL4_CapInitThreadVSpace, TEST_VADDR, seL4_X86_Default_VMAttributes);
     ```

   - **Once all of the intermediate paging structures have been mapped for a specific virtual address range, physical frames can be mapped into that range by invoking the frame capability.** The code snippet below shows an example of mapping a frame at address `TEST_VADDR`.

     ```c++
     /* map a read-only page at TEST_VADDR */
     error = seL4_X86_Page_Map(frame, seL4_CapInitThreadVSpace, TEST_VADDR, seL4_CanRead, seL4_X86_Default_VMAttributes);
     ```

#### Code

1. Three level mapping

   ```c++
   /* map a PDPT at TEST_VADDR */
   error = seL4_X86_PDPT_Map(pdpt, seL4_CapInitThreadVSpace, TEST_VADDR, seL4_X86_Default_VMAttributes);
   // Map a page directory object
   error = seL4_X86_PageDirectory_Map(pd, seL4_CapInitThreadVSpace, TEST_VADDR, seL4_X86_Default_VMAttributes);
   // Map a page table object
   error = seL4_X86_PageTable_Map(pt, seL4_CapInitThreadVSpace, TEST_VADDR, seL4_X86_Default_VMAttributes);
   ```

2. After mapping intermediate paging structure, we can mapped page with specific right

   ```c++
   error = seL4_X86_Page_Map(frame, seL4_CapInitThreadVSpace, TEST_VADDR, seL4_ReadWrite, seL4_X86_Default_VMAttributes);
   // Write to this address
   seL4_Word *x = (seL4_Word *) TEST_VADDR;
   *x = 5;
   ```

### Threads

[Tutorial](https://docs.sel4.systems/Tutorials/threads.html)

#### Concept

1. Thread control block

   - It represents **an execution context** and **manage processor time**. 
   - TCBs contains following information
     - a priority and maximum control priority
     - register state and floating-point context
     - `CSpace` capability,
     - `VSpace` capability,
     - endpoint capability to send fault messages to
     - and the reply capability slot

2. Scheduling model

   - The seL4 scheduler **chooses the next thread to run on a specific processing core**, and is a **priority-based round-robin scheduler**.

     - Priorities: The scheduler picked **the highest priority**, runnable thread

     - Round robin: **FIFO** round robin fashion

     - Domain scheduling: seL4 provides **a top-level hierarchical scheduler** which provides static, cyclical scheduling of **scheduling partitions known as domains**

       **Threads can be assigned to domains, and threads are only scheduled when their domain is active**. Cross-domain IPC is delayed until a domain switch, and seL4_Yield between domains is not possible. **When there are no threads to run while a domain is scheduled, a domain-specific idle thread will run until a switch occurs**.

       ```c++
       /* Set thread's domain */
       seL4_Error seL4_DomainSet_Set(seL4_DomainSet _service, seL4_Uint8 domain, seL4_TCB thread);
       ```

3. CapDL: Capability Distribution Language [link](https://docs.sel4.systems/projects/capdl/)

#### Code

1. Create a new thread

   ```c++
   seL4_Error result = seL4_Untyped_Retype(tcb_untyped, seL4_TCBObject, seL4_TCBBits, root_cnode, 0, 0, tcb_cap_slot, 1);
   ```

   Configure a TCB

   ```c++
   result = seL4_TCB_Configure(tcb_cap_slot, 0, root_cnode, 0, root_vspace, 0, (seL4_Word) thread_ipc_buff_sym, tcb_ipc_frame);
   ```

2. Set priority

   ```c++
   result = seL4_TCB_SetPriority(tcb_cap_slot, root_tcb, 254);
   ```

   Set initial register

   ```c++
   sel4utils_arch_init_local_context((void*)new_thread,
                                     (void *)print, (void *)&val, (void *)3,
                                     (void *)tcb_stack_top, &regs);
   ```

3. Resume the new thread

   ```c++
   error = seL4_TCB_Resume(tcb_cap_slot);
   ```

## Reference

1. [seL4 website](https://sel4.systems/)
2. [UNSW AOS](https://www.cse.unsw.edu.au/~cs9242/18/lectures/)
3. [UNSW Software Verification](https://www.cse.unsw.edu.au/~cs4161/lect.html)