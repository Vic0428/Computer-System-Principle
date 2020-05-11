# SeL4

## Overview

The seL4 microkernel is the world’s most high-assured operating system kernel. 

## seL4 mechanisms

### Capabilities

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

   

## Reference

1. [seL4 website](https://sel4.systems/)
2. [UNSW AOS](https://www.cse.unsw.edu.au/~cs9242/18/lectures/)
3. [UNSW Software Verification](https://www.cse.unsw.edu.au/~cs4161/lect.html)