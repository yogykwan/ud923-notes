# Threads and Concurrency

## Lesson Preview
- thread definition
- threads vs. processes
- threads mechanisms

## Threads Definition
Threads is executing context within a single process in multiple-cpu systems.
- is an active entity - executing unit of a process
- works simultaneously with others - multiple threads executing
- requires coordination - sharing of i/o devices, cpu, memory

Process vs. Thread
- Threads share all the virtual to physical address mapping and description about code, data, files.
- But they execute different instructions, access different portions of that address space, operate on different portions of input.
- Each thread has a separate execution context like program counter, stack pointer, stack, thread-specific registers.

## Multithreading
### Multithreading benefits
Threads can improve performance in the following ways
- parallelization - speed up
- specialization - hotter cache
- efficiency - lower memory requirement, inter-thread communication is cheaper than inter-process

Even if # of threads > # of cpus, multithreading can help hide latency.
- shared virtual mapping means t_ctx switch of threads is smaller than t_ctx_switch of processes
- as long as t_idle is bigger than twice the t_ctx_switch, context switch can hide idling time
- especially useful to hide more of latency of i/o operations

Multithreading benefits both allocations and OS code
- threads working on behalf of applications
- OS-level services like daemons or drivers

### Multithreading models
For a user-level thread to execute, it must be associated with a kernel-level thread which OS-level then scheduler schedules onto cpu.
- One-to-one model - each user-level thread has a kernal-level thread
    - \+ OS sees/understands threads, synchronization, blocking
    - \- must go to OS for all operations
    - \- OS have limits on policies, thread number, portability
- Many-to-one model - all user-level threads are mapped to a single kernel-level thread
    - \+ totally portable, doesn't depend on OS limits and policies
    - \- OS has no insights into application needs
    - \- entire process may be blocked when one user-level thread is blocked
- Many-to-many model - in a process some user-level threads associate with a kernel-level thread while others may have one-to-one map
    - \+ best of both worlds
    - \+ can have bound or unbound threads
    - \- requires coordination between user and kernel-level thread managers
    
### Multithreading scopes
- process scope - user-level library manages thread within a single process
- system scope - user-level threads are visible, it's system-wide thread management by OS-level thread mangers, e.g. cpu scheduler

### Multithreading patterns
#### boss-workers pattern
- boss assigns work to workers
- worker performs entire task
- communicate via producer/consumer queue
- worker pool can be static or dynamic
- throughput is 1/boss_time_per_order

Pros & cons
- \+ boss doesn't need to know details about workers
- \- queue synchronization

Variants
- If all workers created equal
    - \+ simplicity
    - \- thread pool management locality
- If workers specialized for certain tasks
    - \+ better locality and QoS management
    - \- load balancing

####pipeline patter
- pipeline is a sequence of stages
- each stage means a subtask
- use thread pool to determine the number of threads for each stage
- shared-buffer based communication between stages
- throughput is the weakest link

Pros & cons
- \+ specialization and locality
- \- balancing and synchronization overheads

####layered pattern
- each layer group of related subtasks
- e2e task must pass up and sown through all layers

Pros & cons
- \+ specialization
- \+ less fine-grained than pipeline
- \- not suitable for all applications
- \- complex synchronization

## Threads Mechanisms
### Threads creation
- Thread type - thread data structure to identify threads, keep track of resource usage
- Fork(proc, args) - to create a thread (differ from a unix fork)
- Join(thread) - parent thread will be blocked until the child thread completes

### Mutex
Mutex enables exclusive access to only one thread at a time.
- lock mutex - thread with locked mutex has exclusive access to shared list
- critical section - operations that can be performed by only oe thread at a time
- unlock mutex - when owner frees lock, any thread waiting for the lock starts trying executing lock operation

### Condition variable
Threads wait for specific condition before proceeding.
- wait(mutex, cond) - mutex is automatically releases and reacquired
- signal(cond) - notify only one thread waiting on condition
- broadcast(cond) - notify all waiting threads but still only one thread can pop wait queue

## Typical Problems
### Producers and consumer
Producers insert data into list. When the list is full, consumer clears it.
```
// main
Mutext m;
Condition list_full;
for i = 0..10
  producers[i] = fork(safe_insert, NULL);
consumer = fork(print_and_clear, my_list);
// producers: safe_insert
Lock(m) {
    my_list.insert(my_thred_id);
    if(my_list.full()) 
        Signal(list_full);
} // unlock
// consumer: print_and_clear
Lock(m) {
    while(my_list.not_full()) 
        Wait(m, list_full);
    my_list.print_and_remove_all();
} // unlock
```

Why not using "if" instead of "while" before Wait?
- "while" can support multiple consumer threads
- "if" can't guarantee access to mutex once the condition is signaled
- the list can change before consumer get s access again

### Readers and writer
At any time, 0 or more readers can access the resource, and 0 or 1 writer can access the resource concurrently.
```
// main
Mutext counter_mutex;
Condition read_phrase, write_phrase;
int resource_counter = 0;
// readers
Lock(counter_mutex) {
    while(resource_counter == -1)
        Wait(counter_mutex, read_phrase)
    ++resource_counter;
} // unlock
read_data();
Lock(counter_mutex) {
    --resource_counter;
    if(resource_counter == 0)
        Signal(write_phrase)
} // unlock
// writer
Lock(counter_mutex) {
    while(resource_counter != 0)
        Wait(counter_mutex, write_phrase)
    resource_counter = -1;
} // unlock
write_data();
Lock(counter_mutex) {
    resource_counter = 0;
} // unlock
// Signal/broadcast condition variables after unlock to avoid spurious wake-ups
Broadcast(read_phrase);
Signal(write_phrase);
}
```

Critical section structure with proxy variable:
```
// ENTER CRITICAL SECTION
Lock(mutex) {
    while(!predicate_for_access)
        Wait(mutex, cond_var)
    update predicate
} // unlock
perform_critical_operation()
// EXIT CRITICAL SECTION
Lock(mutex) {
    update predicate
    signal/broadcast(cond_var)
} // unlock
```

### Common pitfalls
- keep track of mutex/condition variables used
- access resource safely by using lock & unlock
- use a single mutex to access a single resource
- signal correct condition
- don't use signal when broadcast is needed
- signal/broadcast condition variables after unlock to avoid spurious wake-ups
- maintain lock order to prevent deadlock, the cycles in wait graph

## References
- [Birrel, Andrew, An Introduction to Programming with Threads.](papers/ud923-birrell-paper.pdf)