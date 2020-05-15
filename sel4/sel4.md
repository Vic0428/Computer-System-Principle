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

### IPC

[tutorial](https://docs.sel4.systems/Tutorials/ipc.html)

#### Concept

1. IPC
   - Interprocess communication (IPC) is **the microkernel mechanism for synchronous transmission of small amounts of data and capabilities** between processes.
   - In seL4, IPC is facilitated by small kernel objects known as ***endpoints***, which act as general communication ports. **Invocations on endpoint objects are used to send and receive IPC messages**.
   - **Endpoints consist of a queue of threads waiting to send, or waiting to receive messages**. To understand this, consider an example where *n* threads are waiting for a message on an endpoint. **If *n* threads send messages on the endpoint, all *n* waiting threads will receive the message and wake up**. If an *n+1*th sender sends a message, that sender is now queued.
2. Relevant system calls
   - `seL4_Send`: blocking sending message
   - `seL4_NBSend`: polling send
   - Similar for `seL4_Recv ` and `seL4_NBRecv` 
3. IPC buffer
   - Each thread has a buffer (referred to as the *IPC buffer*), which contains the payload of the IPC message, consisting of data and capabilities
4. Capability transfer
   - Along with data, IPC can be used to **send capabilities between processes per message**. 

#### Code

An `echo` server

```c++


#include <assert.h>
#include <sel4/sel4.h>
#include <stdio.h>
#include <utils/util.h>

// cslot containing IPC endpoint capability
extern seL4_CPtr endpoint;
// cslot containing a capability to the cnode of the server
extern seL4_CPtr cnode;
// empty cslot
extern seL4_CPtr free_slot;

int main(int c, char *argv[]) {

	seL4_Word sender;
    seL4_MessageInfo_t info = seL4_Recv(endpoint, &sender);
    while (1) {
	    seL4_Error error;
        if (sender == 0) {

             /* No badge! give this sender a badged copy of the endpoint */
             seL4_Word badge = seL4_GetMR(0);
             seL4_Error error = seL4_CNode_Mint(cnode, free_slot, seL4_WordBits,
                                                cnode, endpoint, seL4_WordBits,
                                                seL4_AllRights, badge);
             printf("Badged %lu\n", badge);

             // Use cap transfer to send the badged cap in the reply
             info = seL4_MessageInfo_new(0, 0, 1, 0);
             seL4_SetCap(0, free_slot);
             /* reply to the sender and wait for the next message */
             seL4_Reply(info);

             /* now delete the transferred cap */
             error = seL4_CNode_Delete(cnode, free_slot, seL4_WordBits);
             assert(error == seL4_NoError);

             /* wait for the next message */
             info = seL4_Recv(endpoint, &sender);
        } else {

             // Use printf to print out the message sent by the client
             // followed by a new line
             seL4_Word length = seL4_MessageInfo_get_length(info);
             for (int i = 0; i < length; ++i) {
                  seL4_Word data = seL4_GetMR(i);
                  printf("%c", (char)data);
             }
             printf("\n");
             // Reply to the client and wait for the next message
             error = seL4_CNode_SaveCaller(cnode, free_slot, seL4_WordBits);


             info = seL4_Recv(endpoint, &sender);
             length = seL4_MessageInfo_get_length(info);
             for (int i = 0; i < length; ++i) {
                  seL4_Word data = seL4_GetMR(i);
                  printf("%c", (char)data);
             } 
             printf("\n");
          
             seL4_Send(free_slot, seL4_MessageInfo_new(0, 0, 0, 0));
             info = seL4_ReplyRecv(endpoint, info, &sender);
        }
    }

    return 0;
}
```

A simple client

```c++


#include <assert.h>
#include <stdio.h>
#include <sel4/sel4.h>
#include <utils/util.h>

extern seL4_CPtr endpoint;
extern seL4_CPtr cnode;
extern seL4_CPtr badged_endpoint;

const char *messages[] = {"quick", "fox", "over", "lazy"};

int main(int c, char *argv[]) {

    int id = 1;
    
    printf("Client %d: waiting for badged endpoint\n", id);
    seL4_SetCapReceivePath(cnode, badged_endpoint, seL4_WordBits);
    seL4_SetMR(0, id);
    seL4_MessageInfo_t info = seL4_MessageInfo_new(0, 1, 0, 1);
    info = seL4_Call(endpoint, info);
    assert(seL4_MessageInfo_get_extraCaps(info) == 1);
    /* wait for the server to send us an endpoint */
    printf("Client %d: received badged endpoint\n", id);

    for (int i = 0; i < ARRAY_SIZE(messages); i++) {
        int j;
        for (j = 0; messages[i][j] != '\0'; j++) {
            seL4_SetMR(j, messages[i][j]);
        }
        info = seL4_MessageInfo_new(0, 0, 0, j);
        seL4_Call(badged_endpoint, info);
    }
    return 0;
}
```

### Notifications

#### Concept

1. Why notification?
   - Notifications allow processes to **send asynchronous signals to each other**, and **are primarily used for interrupt handling and to synchronise access to shared data buffers**.
   - Notification object
     - Signals are **sent** and **received** with **invocations on capabilities to notification objects**
     - A notification object consists of a data word, which acts as an array of binary semaphores, and a queue of TCBs waiting to for notifications.
   - Notification object can be in three states
     - Waiting - there are TCBs queued on this notification waiting for to be signalled.
     - Active - TCBs have signalled data on this notification,
     - Idle - no TCBs are queued and no TCBs have signalled this object since it was last set to idle.
2. Two important API
   - `seL4_Signal`
   - `seL4_Wait`
3. **Notification objects can be used to receive signals of interrupt delivery, and can also be bound to TCBs such that signals and IPC can be received by the same thread.** 

#### Code

1. A classic **consumer-producer** problem

   One **consumer** and two **producers**

   `consumer.c`

   ```c++
   
   
   #include <assert.h>
   #include <sel4/sel4.h>
   #include <stdio.h>
   #include <utils/util.h>
   #include <sel4utils/util.h>
   
   // notification object
   extern seL4_CPtr buf1_empty;
   extern seL4_CPtr buf2_empty;
   extern seL4_CPtr full;
   extern seL4_CPtr endpoint;
   
   extern seL4_CPtr buf1_frame_cap;
   extern const char buf1_frame[4096];
   extern seL4_CPtr buf2_frame_cap;
   extern const char buf2_frame[4096];
   
   // cslot containing a capability to the cnode of the server
   extern seL4_CPtr consumer_vspace;
   extern seL4_CPtr producer_1_vspace;
   extern seL4_CPtr producer_2_vspace;
   
   extern seL4_CPtr cnode;
   extern seL4_CPtr mapping_1;
   extern seL4_CPtr mapping_2;
   
   #define BUF_VADDR 0x5FF000
   
   int main(int c, char *argv[]) {
       seL4_Error error = seL4_NoError;
       seL4_Word badge;
   
       
       /* set up shared memory for consumer 1 */
       /* first duplicate the cap */
       error = seL4_CNode_Copy(cnode, mapping_1, seL4_WordBits, 
                             cnode, buf1_frame_cap, seL4_WordBits, seL4_AllRights);
       ZF_LOGF_IFERR(error, "Failed to copy cap");
       /* now do the mapping */
       error = seL4_ARCH_Page_Map(mapping_1, producer_1_vspace, BUF_VADDR, 
                                  seL4_AllRights, seL4_ARCH_Default_VMAttributes);
       ZF_LOGF_IFERR(error, "Failed to map frame");
       
       // share buf2_frame_cap with producer_2
       /* First duplicate the cap */
       error = seL4_CNode_Copy(cnode, mapping_2, seL4_WordBits,
                               cnode, buf2_frame_cap, seL4_WordBits, seL4_AllRights);
       /* now do the mapping */
       error = seL4_ARCH_Page_Map(mapping_2, producer_2_vspace, BUF_VADDR,
                                   seL4_AllRights, seL4_ARCH_Default_VMAttributes);
   
       /* send IPCs with the buffer address to both producers */
       seL4_SetMR(0, BUF_VADDR);
       seL4_Send(endpoint, seL4_MessageInfo_new(0, 0, 0, 1));
       seL4_SetMR(0, BUF_VADDR);
       seL4_Send(endpoint, seL4_MessageInfo_new(0, 0, 0, 1));
       
       /* start single buffer producer consumer */
       volatile long *buf1 = (long *) buf1_frame;
       volatile long *buf2 = (long *) buf2_frame;
   
       *buf1 = 0;
       *buf2 = 0;
   
       
       // Signal both producers
       seL4_Signal(buf1_empty);
       seL4_Signal(buf2_empty);
   
       printf("Waiting for producer\n");
       for (int i = 0; i < 10; i++) {
           seL4_Wait(full, &badge);
           printf("Got badge: %lx\n", badge);
           if (badge & 0b01) {
               assert(*buf1 == 1);
               *buf1 = 0;
               seL4_Signal(buf1_empty);
           }
           if (badge & 0b10) {
               assert(*buf2 == 2);
               *buf2 = 0;
               seL4_Signal(buf2_empty);
           }
      }
   
       printf("Success!\n");
       return 0;
   }
   ```

   `producer1.c`

   ```c++
   
   
   #include <assert.h>
   #include <stdio.h>
   #include <sel4/sel4.h>
   #include <utils/util.h>
   #include <sel4utils/util.h>
   
   // caps to notification objects
   extern seL4_CPtr empty;
   extern seL4_CPtr full;
   extern seL4_CPtr endpoint;
   
   int main(int c, char *argv[]) {
       int id = 2;
       
       seL4_Recv(endpoint, NULL);
       volatile long *buf = (volatile long *) seL4_GetMR(0);
       
       for (int i = 0; i < 100; i++) {
           seL4_Wait(empty, NULL);
           printf("%d: produce\n", id);
           *buf = id;
           seL4_Signal(full);
       }
       return 0;
   }
   ```

2. From the above code, we found that there are three important **notification objects**

   - `buf1_empty`
   - `buf2_empty`
   - `full`

### Interrupt

#### Concept

1. IRQControl 

   - The root task is given a single capability from which **capabilities to all irq numbers in the system can be derived**, `seL4_CapIRQControl`. 

2. IRQHandlers

   - IRQHandler capabilities **give access to a single irq and are standard seL4 capabilities**: they *can* be moved and duplicated according to system policy.

     ```c
     // Get a capability for irq number 7 and place it in cslot 10 in a single-level cspace.
     error = seL4_IRQControl_Get(seL4_IRQControl, 7, cspace_root, 10, seL4_WordBits);
     ```

3. Receiving interrupts

   - Interrupts are received by **registering a capability to a notification object with the IRQHandler capability** for that `irq`, as follows. 

     ```c++
     seL4_IRQHandler_setNotification(irq_handler, notification);
     ```

     On success, **this call will result in signals being delivered to the notification object when an interrupt occurs**.

#### Code

1. Bound `interrupt_handle` capability with `notification` object

   ```c++
   /* Invoke irq_control to put the interrupt for TTC0_TIMER1_IRQ in
          cslot irq_handler (depth is seL4_WordBits) */
   error = seL4_IRQControl_Get(irq_control, TTC0_TIMER1_IRQ, cnode, irq_handler, seL4_WordBits);
   /* Set ntfn as the notification for irq_handler */
   error = seL4_IRQHandler_SetNotification(irq_handler, ntfn);
   ```

2. Handler interrupt and `ack` it

   ```c++
   int count = 0;
   while (1) {
     	/* Handle the timer interrupt */
     	seL4_Word badge;
     	seL4_Wait(ntfn, &badge);
     	timer_handle_irq(&timer_drv);
     	if (count == 0) {
       	printf("Tick\n");
     	}
   
     	/* Ack the interrupt */
     	seL4_IRQHandler_Ack(irq_handler);
     	count++;
     	if (count == 1000 * msg) {
       	break;
     	}
   }
   
   ```

### Fault handling

#### Concept

[tutorial](https://docs.sel4.systems/Tutorials/fault-handlers.html)

1. A fault handler is a separate instruction stream which **the CPU can jump to in order to rectify an anomalous condition in the current thread and then return to the previous instruction stream**

2. In seL4, faults are modeled as **separately programmer-designated** “fault handler” threads. In **monolithic kernels**, faults are not usually delivered to **a user space handler**, but they are handled by the monolithic kernel itself. 

3. Thread faults vs other sources of faults

   - Fault events generated by the CPU it self when it encounters anomalies in the instruction stream
   - Fault events generated by hardware in the event of some hardware anomaly
   - Fault events generated by the seL4 kernel when it encounters anomalies in the current thread

   **Thread faults: the third kind faults**

4. How does thread fault handling work?

   - In seL4, when a thread generates a thread fault, the kernel will **block** the faulting thread’s execution and attempt to **deliver a message across a special endpoint associated with that thread**, called its “fault handler” endpoint.
   - **The only special thing about the fault handler endpoint is that a thread can only have *one* of them**. Otherwise it is created and managed just the same way as any other kind of seL4 endpoint object.

5. Reasons for thread faults

   - **Cap fault**: A fault triggered because of an invalid cap access.
   - **VM fault**: A fault triggered by incoherent page table state or incorrect memory accesses by a thread.
   - **Unknown Syscall fault**: Triggered by performing a syscall invocation that is unknown to the kernel.
   - **Debug fault**: Triggered when a breakpoint, watchpoint or single-step debug event occurs.

6. Setting up a fault endpoint for a thread

   - In the scenario where a fault message is being delivered on a fault endpoint, **the kernel acts as the IPC “sender” and the fault handler acts as a receiver**.

   - This implies that when caps are being handed out to the fault endpoint object, **one cap to the object must be given to the kernel and one cap to the object must be given to the handler**.

#### Code

There are two parts of the code: `faulter` and `handler`

First, take a glance at `faulter`

```c++

#include <stdio.h>
#include <sel4/sel4.h>
#include <utils/attribute.h>

#define PROGNAME        "Faulter: "
extern seL4_CPtr sequencing_ep_cap;

extern seL4_CPtr local_badged_faulter_fault_ep_empty_cap;
extern seL4_CPtr faulter_empty_cap;

static void
touch(seL4_CPtr slot)
{
    UNUSED seL4_Word unused_badge;

    seL4_NBRecv(slot, &unused_badge);
}

int main(void)
{
    UNUSED seL4_Word tmp_badge;

    /* First tell the handler where in our CSpace it is safe for them to
     * place a badged fault handler EP cap so that it can receive fault IPC
     * messages on our behalf.
     */
    printf(PROGNAME "Running. About to send empty slot CPtr to handler.\n");
    seL4_SetMR(0, local_badged_faulter_fault_ep_empty_cap);
    seL4_Call(sequencing_ep_cap, seL4_MessageInfo_new(0, 0, 0, 1));

    printf(PROGNAME "Handler has minted fault EP into our cspace. Proceeding.\n");

    printf(PROGNAME "About to touch fault vaddr.\n");
    touch(faulter_empty_cap);

    printf(PROGNAME "Successfully executed past the fault.\n"
           PROGNAME "Finished execution.\n");

    return 0;
}
```

The faulted first send an `fault_cap` and then caused a `cap_fault` thread fault. 

Next, look at the code of `handler`

```c++

#include <assert.h>
#include <stdio.h>
#include <sel4/sel4.h>
#include <utils/util.h>
#include <autoconf.h>

#define FAULTER_BADGE_VALUE     (0xBEEF)
#define PROGNAME                "Handler: "
/* We signal on this notification to let the fauler know when we're ready to
 * receive its fault message.
 */
extern seL4_CPtr sequencing_ep_cap;

extern seL4_CPtr faulter_fault_ep_cap;

extern seL4_CPtr handler_cspace_root;
extern seL4_CPtr badged_faulter_fault_ep_cap;

extern seL4_CPtr faulter_tcb_cap;
extern seL4_CPtr faulter_vspace_root;
extern seL4_CPtr faulter_cspace_root;

int main(void)
{
    int error;
    seL4_Word tmp_badge;
    seL4_CPtr foreign_badged_faulter_empty_slot_cap;
    seL4_CPtr foreign_faulter_capfault_cap;
    seL4_MessageInfo_t seq_msginfo;

    printf(PROGNAME "Handler thread running!\n"
           PROGNAME "About to wait for empty slot from faulter.\n");


    seq_msginfo = seL4_Recv(sequencing_ep_cap, &tmp_badge);
    foreign_badged_faulter_empty_slot_cap = seL4_GetMR(0);
    printf(PROGNAME "Received init sequence msg: slot in faulter's cspace is "
           "%lu.\n",
           foreign_badged_faulter_empty_slot_cap);



    /* Mint the fault ep with a badge */

    error = seL4_CNode_Mint(
        handler_cspace_root,
        badged_faulter_fault_ep_cap,
        seL4_WordBits,
        handler_cspace_root,
        faulter_fault_ep_cap,
        seL4_WordBits,
        seL4_AllRights, FAULTER_BADGE_VALUE);

    ZF_LOGF_IF(error != 0, PROGNAME "Failed to mint ep cap with badge!");
    printf(PROGNAME "Successfully minted fault handling ep into local cspace.\n");


    /* This step is only necessary on the master kernel. On the MCS kernel it
     * can be skipped because we do not need to copy the badged fault EP into
     * the faulting thread's cspace on the MCS kernel.
     */

    error = seL4_CNode_Copy(
        faulter_cspace_root,
        foreign_badged_faulter_empty_slot_cap,
        seL4_WordBits,
        handler_cspace_root,
        badged_faulter_fault_ep_cap,
        seL4_WordBits,
        seL4_AllRights);

    ZF_LOGF_IF(error != 0, PROGNAME "Failed to copy badged ep cap into faulter's cspace!");
    printf(PROGNAME "Successfully copied badged fault handling ep into "
           "faulter's cspace.\n"
           PROGNAME "(Only necessary on Master kernel.)\n");
    error = seL4_TCB_SetSpace(
        faulter_tcb_cap,
        foreign_badged_faulter_empty_slot_cap,
        faulter_cspace_root,
        0,
        faulter_vspace_root,
        0);

    seL4_Reply(seL4_MessageInfo_new(0, 0, 0, 0));
    
    // Get capability fault address
    seL4_Recv(badged_faulter_fault_ep_cap, NULL);
    foreign_faulter_capfault_cap = seL4_GetMR(seL4_CapFault_Addr);

    // Handling a thread fault
    error = seL4_CNode_Copy(
        faulter_cspace_root,
        foreign_faulter_capfault_cap,
        seL4_WordBits,
        handler_cspace_root,
        sequencing_ep_cap,
        seL4_WordBits,
        seL4_AllRights);

    // To have the faulter thread wakeup and try to execute again
    seL4_Reply(seL4_MessageInfo_new(0, 0, 0, 0));
    return 0;
}
```

`handler` received the fault address of `capability` and then had a single copy to solve this problem. Finally, it replied the `faulter` to let it try again. 



## Reference

1. [seL4 website](https://sel4.systems/)
2. [UNSW AOS](https://www.cse.unsw.edu.au/~cs9242/18/lectures/)
3. [UNSW Software Verification](https://www.cse.unsw.edu.au/~cs4161/lect.html)