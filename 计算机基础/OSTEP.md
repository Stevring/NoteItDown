

# Virtualization

## Virtualization of CPU

### 02 Introduction

1. The operating system
   - is a virtual machine
   - provides a standard library
   - is a resource manager

### 04 Processes

1. The OS virtualizes CPU to make the illusion that each process has a cpu
2. Process
   - is the OS's abstraction of a running program
   - machine state of a process: memory, registers, I/O information
   - process may exists in different states like:
     - running
     - ready
     - blocked
3. Process API
   - Create
     - steps to create a process
       - **load** code and static data into memory
       - allocate a run-time **stack** (for local variables, function params, return address, etc)
       - (maybe) allocate memory for **heap**
       - other initialization tasks like I/O
       - start at the entry point
   - Destroy, Wait, Miscellaneous Control, Status

### 05 Process API

1. fork()
   - create a subprocess that shares the same code with the parent process
2. wait(), waitpid()
   - wait for the child process to finish
3. exec(), execvp
   - overwrite the current code, static data, heap, stack, etc. 
   - instead of creating a new process, it transforms current program into a different one. so `exec` never returns.
4. kill()

### Limited Direct Execution

1. **direct execution** means

   - the OS runs programs directly on the cpu
   - this advantages in
     - running fast
   - but has two major problems
     - what if the program does something not allowed?
     - how os controls switching into other programs
       - when a user program takes the cpu, the os is not running

2. **restricted operation** means restrict operations that a program can execute

   - in **user mode**
     - programs is restricted
   - while in **kernel mode**
     - operations are not restricted
     - and the os runs in kernel mode

   - timeline for executing a program
     - <img src="https://raw.githubusercontent.com/Stevring/Pics/main/timeline_of_syscall.png" alt="image-20200330232640258" style="width:100%;" />

3. switching between processes

   1. timeline for switching

      <img src="https://raw.githubusercontent.com/Stevring/Pics/main/timeline_of_switching.png" alt="image-20200330232640258" style="width:100%;" />

### 07 Scheduling

1. **Scheduling Metrics** includes
   - turnaround time: $T_{turnaround}=T_{completion}-T_{arrival}$
   - fairness
   - response time: $T_{response}=T_{fisrtrun}-T_{arrival}$
2. One basic scheduling algorithm is **FIFO (first in first out)**
   - which is simple and easy to implement
   - but has the **convoy problem**: a number of short potential consumers of a resource get quened by a heavy weight resource consumer
3. **SJF (shortest job first) , STCF (shortest time to completion first)**
   - good for turnaround time
   - bad for response time
4. **RR (round-robin)** 
   - good for response time
   - bad for turnaround time
5. **MLFQ (multi-level feedback queue)**
   1. different queues with different priority level
   2. at any time, one process is on one queue
   3. rules of MLFQ
      - Rule 1: If Priority(A) > Priority(B), A runs (B doesn’t).
      - Rule 2: If Priority(A) = Priority(B), A & B run in round-robin fashion using the time slice (quantum length) of the given queue.
      - Rule 3: When a job enters the system, it is placed at the highest priority (the topmost queue).
      - Rule 4: Once a job uses up its time allotment at a given level (re-gardless of how many times it has given up the CPU), its priority is reduced (i.e., it moves down one queue).
      - Rule 5: After some time period S, move all the jobs in the system to the topmost queue.



## Virtualization of Memory

### 13 The Abstraction: Address Spaces

1. the address space of a program contains
   - code, stack, heap, etc...
2. the os virtualizes memory with the goals:
   - transparency
   - efficiency
   - protection

### 14 Memory API

1. there are **two types of memory**

   - **stack memory**, which is allocated and deallocated implicitly by the compiler

   - **heap memory**, all allocations and deallocations of which are explicitly handled by the programmer.

2. malloc, free, calloc, realloc, etc.

### 15 Mechanism: Address Translation

1. assuming each program is contiguous in physical memory

2. base and bound registers

   - are provided by the hardware to record the base and bound of physical memory of the currently running program

   - when memory reference happens, the **Memory Management Unit (MMU)** translates the virtual address into physical address according to the **base and bounds** register.

     ![timeline_of_dynamic_relocation](https://raw.githubusercontent.com/Stevring/Pics/main/timeline_of_dynamic_relocation.png)



### 16 Segmentation

1. the base and bounds registers are not flexible (many unused space between heap and stack of a programme)
2. segmentation!
   - idea: each programme has three logical segments: code, stack and heap
   - use three sets of base and bounds registers for each segment
   - then we can put different segments at different space
3. how to know which segments an virtual address is refering?
   - use top k bits to represent segments
   - e.g: 00 for code, 01 for stack, 10 for heap
4. fine grained segmentation
   - we can use more than 3 segments to represent a programme
   - and use a segment table to record each segment's information

### 18 Paging: Introduction

1. problem with segmentation: fragmentation

2. and paging is more flexible for space allocation and simple for management

3. page table (of each process) records the virtual address to physical address translation

4. address translation:

   <img src="https://raw.githubusercontent.com/Stevring/Pics/main/address_translation.png" alt="address_translation" style="zoom:50%;" />

5. each page table entry records the VPN to PPN translation, as well as other bits such as VALID, DIRTY, Read/Write, Copy On Write



### 19 Translation Lookaside Buffers (TLB)

1. TLB is part of MMU (Memory Management Unit) that is used to speed up address translation
2. 如果有TLB，访问虚拟地址：一次 TLB查询 （得到物理页号）+ 物理地址查询（得到具体物理内存中的内容）
3. 如果没有TBL，访问虚拟地址：一次页表查询（得到物理页号） + 物理地址查询（得到具体物理内存中的内容）
4. 因为TLB离CPU近，且访问速度更快，所以如果TLB命中了能提高地址转换效率
5. 但是如果TLB miss了，仍然要查询页表，这个时候代价会更高
6. TLB总体上能提高效率，是因为访存的空间局部性（spacial locality）和时间局部性（temporal locality），这使得TLB的hit rate会尽可能地高
7. how the os handles a TLB miss?
   - first the current process trap into exception
   - then the the os walk up the page table to find the translation and update TLB
   - finally return from trap (to the instruction that caused the TLB miss instead of the one after it)
8. on a context switch, we must flush the TLB entries because each process has a different VPN to PPN translation
9. https://www.cnblogs.com/alantu2018/p/9000777.html

### 20 Paging: Smaller Tables

1. when lots of progress are running,  a big memory is needed to hold up their page tables
2. To reduce the page table size, we can
   - use bigger pages
     - this leads to waste in each page (internal fragmentation)
   - hybrid segmentation and paging
     - manages a page table for each segment
     - the base and bound register records the physical page table address and page table size for each segment
     - the virtual address is splitted into-> Seg Number: VPN: Offset
   - two-level page tables
     - <img src="https://raw.githubusercontent.com/Stevring/Pics/main/two_level_page_tables.png" alt="two_level_page_tables" style="zoom:50%;" />



###  21 Swapping: mechanism

1. swap space is a chunk of memory on the disk
2. when multiple process is running, we may run out of physical memory. the os swap pages into memory from Swap Space and swap some pages out to Swap Space.

<img src="https://raw.githubusercontent.com/Stevring/Pics/main/page_fault_control_flow.png" alt="page_fault_control_flow" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Stevring/Pics/main/swapping.png" alt="swapping" style="zoom:50%;" />



### Swapping: Policy

1. Optimal
2. FIFO
3. Random
4. LRU
5. Clock Algorithm



# Concurrency

### 26 Concurrency: An Introduction

1. thread is similar to process:
   - has a PC
   - a private set of registers
   - same context switch process
2. thread is different from process:
   - context switch don't switch page table, and state is saved to TCB(which stores a pointer to the PCB that creates it)
   - each thread has a stack
3. problem of multiple threads running at the same time: race condition (accessing and modifying the same data)

<img src="https://raw.githubusercontent.com/Stevring/Pics/main/key_concurrency_terms.png">

### 28 Locks

1. we can use lock to protect a critical section

2. building locks requires

   - mutual exclusion
   - fairness (none of the threads are starved)
   - performance (the time needed to use lock should not be too heavy)

3. building lock by controlling interrupts:

   - is simple
   - but do not work on multi-processor computers
   - the os may never regain control if a malicious program disabled interrupt and step into a infinite loop

4. spin locks (need hardware support)

   - Test-And-Set instruction

     - ```c
    int TestAndSet(int * old_ptr, int new) { 
         int old = * old_ptr; // fetch old value at old_ptr 
         * old_ptr = new; // store ’new’ into old_ptr 
         return old; // return the old value 
    }
       ```

   - compare-and-swap instruction
   
     - ```c
       int CompareAndSwap(int * ptr, int expected, int new) { 
         int actual = * ptr; 
         if (actual == expected) 
           * ptr = new; 
         return actual; 
       }	
       ```
   
   - load-linked and store-conditional instruction pair
   
     - ```c
       int LoadLinked(int * ptr) { 
       	return * ptr; 
       }
       int StoreConditional(int * ptr, int value) { 
          if (no update to * ptr since LoadLinked to this address) { 
            * ptr = value; 
            return 1; // success!
        	} else { 
            return 0; // failed to update } 
        }
       void lock(lock_t * lock) { 
           while (1) { 
          		while (LoadLinked(&lock->flag) == 1) ; // spin until it’s zero
                if (StoreConditional(&lock->flag, 1) == 1) 
                		return; // if set-it-to-1 was a success: all done // otherwise: try it all over again
           } 
       }
       void unlock(lock_t * lock) { 
         lock->flag = 0;
       }
       ```
   
   - fetch-and-add instruction
   
     - guaranteed fairness
     
     - ```c
       int FetchAndAdd(int * ptr) { 
         int old = * ptr; 
         * ptr = old + 1; 
         return old; 
       }
       ```
   
   - performance of spin locks is not good on a single processor because a thread has to repeatedly check the lock
   
5. to fix the performance problem of spin locks:

   - when threads are going to spin, just give up the processor

     - if there are lots of threads, context switch costs!
     - one thread may be starved

   - use a waiting queue to record threads that want to hold the lock

     - ```c
       typedef struct __lock_t { 
         int flag; 
         int guard;  // provides mutual exclusion for q
         queue_t * q; 
       } lock_t;
       
       void lock_init(lock_t * m) { 
         m->flag = 0; 
         m->guard = 0; 
         queue_init(m->q); 
       }
       
       void lock(lock_t * m) {
       
       while (TestAndSet(&m->guard, 1) == 1) ; //acquire guard lock by spinning 
         if (m->flag == 0) { 
           m->flag = 1; // lock is acquired 
           m->guard = 0;
         } else { 
           queue_add(m->q, gettid());
           m->guard = 0; park(); 
         }
       
       }
       
       void unlock(lock_t * m) {
       
         while (TestAndSet(&m->guard, 1) == 1) ; //acquire guard lock by spinning 
           if (queue_empty(m->q))
             m->flag = 0; // let go of lock; no one wants it
           else 
           unpark(queue_remove(m->q)); // hold lock // (for next thread!) 
         m->guard = 0;
       
       }
       ```


### 31 Semaphores

1. binary semaphores (can be used as a lock)

```c
sem_t m; 
sem_init(&m, 0, X); // initialize semaphore to X, for binary semaphore, X should be 1

sem_wait(&m); 
// critical section here 
sem_post(&m);
```

2. ordering semaphores (can be used as condition variables)

```c
sem_t s;

void * child(void * arg) {
	printf("child\n");
	sem_post(&s); // signal here: child is done
	return NULL; 
}

int main(int argc, char * argv[]) {
	sem_init(&s, 0, X); // X should be 0
  printf("parent: begin\n"); 
  pthread_t c; 
  Pthread_create(&c, NULL, child, NULL); 
  sem_wait(&s); // wait here for child 
  printf("parent: end\n"); return 0;

}
```

3. Producer/Consumer problem

```c
sem_t empty; 
sem_t full; 
sem_t mutex;

void *producer(void * arg) { 
  int i; 
  for (i = 0; i < loops; i++) { 
    sem_wait(&empty); 
    sem_wait(&mutex); 
    put(i); 
    sem_post(&mutex); 
    sem_post(&full); 
  }
}

void * consumer(void * arg) { 
  int i; 
  for (i = 0; i < loops; i++) { 
    sem_wait(&full); 
    sem_wait(&mutex); 
    int tmp = get(); 
    sem_post(&mutex); 
    sem_post(&empty); 
    printf("%d\n", tmp); 
  }
}



int main(int argc, char * argv[]) {
  sem_init(&empty, 0, MAX); // MAX buffers are empty to begin with... 
  sem_init(&full, 0, 0); // ... and 0 are full 
  sem_init(&mutex, 0, 1); // mutex=1 because it is a lock // ...

}
```

4. Reader/Writer

<img src="https://raw.githubusercontent.com/Stevring/Pics/main/reader_writer.png">

5. implementation of semaphores

<img src="https://raw.githubusercontent.com/Stevring/Pics/main/zemaphores.png">

### 32 Common Concurrency Problems

1. non-deadlock

2. deadlock

   - conditions for deadlock to occur
     - mutual exclusion
     - hold and wait
     - no preemption
     - circular wait
   - prevention of deadlock
     - mutual exclusion
     - hold and wait: acquiring all locks at once
     - no preemption:
       - suppose there are two locks: L1 and L2
       - if a thread grabs L1 but fails to grab L2, then unlock L1 and repeat the process
     - circular wait: ordering on lock acquisition

   - avoidance of deadlock
     - static scheduling:
       - if two threads contends for the same lock, run them on the same processor one after one
       - this harms performace
     - banker's algorithm
       - only useful in a limited environment where the OS know everything about a programme

### 33 Event-based concurrency

1. event-based concurrency is found in modern GUI applications and the idea is simple:

   ```c++
   while (1) { 
     events = getEvents(); 
     for (e in events) 
       processEvent(e); 
   }
   ```

2. a process may use `select()` and `poll()` primitives to get current file decriptors that ready to be read, written or have a exception

3. the simple event-based concurrency do not need locks because there are only one thread running

4. but is not efficient if blocking system calls such as `read()` and `open()`are called in `processEvent()`

5. the OS overcomes this by offering **asynchronous I/O**

   - a process use `int aio_read(struct aiocb * aiocbp);` to issue an AIO
   - and use `int aio_error(const struct aiocb * aiocbp);` to check if the I/O has completed
   - but we need to manually store the state of a event handler inorder to support asynchronous I/O

6. on a multi-processor system, the simple event-based concurrency no longer stands. we still need locks.

