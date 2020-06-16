# Threads Case Study: Pthreads

## Lesson Preview
- pthreads vs. [Birrell's api](https://docs.google.com/spreadsheets/d/1k31f6aNb6MBbkoLJ-JKUXo2vxMEIaFar8Ia7rnrXpxI/edit#gid=1868198252&range=G7)
- common practices & safety tips

## Pthread Library
Pthreads stands for POSIX (portable OS interface) threads.
- include pthread.h in main file
- compile with -lpthread or -pthread
- check return values of common functions

### Pthread Creation
- `pthread_t aThread;` - type of thread
- `int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void * (*start_routine)(void *), void *arg));` - fork
- `int pthread_join(pthread_t thread, void **status)` - join
- `pthread_attr_t`
    - specified in pthread_crete - passing NULL means using default behavior
    - defines features of new thread - e.g. stack size, inheritance, joinable, scheduling policy, priority, system/process scope
- `int pthread_detach();` or `pthread_attr_setdetachstate(attr, PTHREAD_CREAT_DETACHED);`
    - Unlike Birrell's api, pthreads allow children threads to detach from parent thread
    - When children are detached, they can't be joined and are free to continue their execution even if parent exits.

### Pthread Mutexes
- `pthread_mutex_t aMutex;` - type pf mutex
- `int pthread_mutex_lock(pthread_mutex_t *mutex);` - explicit lock
- `int pthread_mutex_unlock(pthread_mutex_t *mutex);` - explicit unlock
- `int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);` - mutex attributes specifying behavior need to be initialized explicitly
- `int pthread_mutex_trylock(pthread_mutex_t *mutex)` - when the mutex is free lock it, even if it's not free the thread won't be blocked
- `int pthread_mutex_destroy(pthread_mutex_t *mutex)` - free up data structure for the mutex

Mutex safety tips
- shared data should always be accessed through a single mutex
- mutex scope must be visible to all threads
- globally lock mutexes in order
- don't forget to unlock a mutex

### Pthread Condition Variables
- `pthread_cond_t aCond;` - type of cond variable
- `int pthread_cond_wait(pthread_cond_t * cond, pthread_mutex_t *mutex);` - wait
- `int pthread_cond_signal(pthread_cond_t * cond);` - signal
- `int pthread_cond_broadcast(pthread_cond_t * cond);` - broadcast
- `int pthread_cond_init(pthread_cond_t * cond, const pthread_condattr_t *attr);` - allocate data structure for condition adn initialize its attributes
- `int pthread_cond_destroy(pthread_cond_t * cond);` - free up data structure for the condition

Condition variable safety tips
- whenever predicate changes, signal/broadcast correct condition variable to notify the waiting threads
- use broadcast until figure out what desired behavior is though it will lose performance
- remove signal and broadcast operations until unlock mutex

## Examples
- [producer and consumer](codes/producer-consumer.c) - Producer adds item when list's not full and consumer removes item when it's not empty.
- [priority readers and writers](https://s3.amazonaws.com/content.udacity-data.com/courses/ud923/resources/ud923-ps1-priority-readers-and-writers.zip) - Readers are given priority over writers concerning a shared variable.
