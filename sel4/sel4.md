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

## Reference

1. [seL4 website](https://sel4.systems/)
2. [UNSW AOS](https://www.cse.unsw.edu.au/~cs9242/18/lectures/)
3. [UNSW Software Verification](https://www.cse.unsw.edu.au/~cs4161/lect.html)