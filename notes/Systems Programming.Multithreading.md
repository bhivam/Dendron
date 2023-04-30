---
id: iw530kbpfgidngxm9fc3ahm
title: Multithreading
desc: ''
updated: 1682831101795
created: 1682719784258
---

Problem with multitasking is that interprocess communication is difficult. Required use of piping or the file system. "Message passing." Processes do not share the same memory space.

The solution to this problem is Multithreading.

# Multithreading
We can have multiple concurrent tasks executing in the same process, but we can now use the same memory space, open files, etc. 

#### Kernel Threads
- These are scheduled by the operating system.
- Separate processes that share memory and OS state.
- Threads can preempt eachother and can run simulatneously on different CPUs.
- They are managed by the OS, so they are expensive to create and manage. 

#### Green Threads (library level)
- Managed by the runtime system of the language
- from the OS perspective this is a single task.
- These are generally cooperatively scheduled. This means that threads have to yield to other threads instead of being forced into sharing CPU time (preemptive scheduling).

POSIX can support both types of threads. 

```c
#include <pthread.h>

int pthread_create(
                   pthread_t *thread,
                   const pthread_attr_t *attr,
                   void *(*start_routine) (void *), 
                   void *arg);
```
##### thread
This holds the id of the newly spawned thread. 
##### pthread_attr_t
This will have all the configuration of the new thread regarding the environment variables and such. 
##### start_routine
This will be the function that it will execute. It takes in a pointer and return a pointer. This allows us to give any data we want recieve any data we want. 
##### arg
This is the argument that will be passed to the function. 
##### return value
0 on success, err if anything else

#### Usage
When using thread. We will have our thread execut some function. Then we can go and do want we want while it is executing. But what do we do when we want to get the output of the function. 

```c
#include <pthread.h>

int pthread_join(pthread_t thread, void **retval);
```
##### thread
This is the id of the thread you are waiting for. 
##### retval
This will be a double pointer. The return value of the thread will be pasted in here. NULL here will throw out the return value.

With fork()/wait(), you must wait() for you child using the thread which spawned it. But in threading, the spawning thread doesn't have to wait for the child thread. They are all peers. 

```c
#include <pthread.h>

int pthread_detach(pthread_t thread);
```
This will have make it so that the thread's output is thrown away and it doesn't have to be joined. It will terminate itself. (It becomes unjoinable). 

The first thread, the one running main, executes exit. exit() will terminate all the threads. 

Because threads work in the same memory space, they can share the same data. 

```c
#include <pthread.h>

void pthread_exit(void *retval);
```
This will terminate the current thread and return the value given. 

One strategy is to use pthread_exit in main. This will terminate the thread but it will not termiante the other threads. The run time system will wait for the rest of the threads to terminate. and then call exit. 

### Process vs Thread
#### Process
- instructions to execute
- contents of heap
- global data
- call stack
- current instruction
#### Thread
each thread has:
- call stack
- current instruction
threads share
- program code/instructions
- global data
- heap

#### Why?
- dividing work among many CPUsx`
- avoid blocking by having a thread doing something while another thread is waiting for the IO to stop blocking. 

#### -pthread
We need to add this flag while compiling to change the c library to be compatible with pthead. For example, errno sets a global variable, but if multiple threads share the errno, there can be confusion about which errors are corresponding to which thead and overwriting errors. 

errno is a macro where in the thread specific version of errno.h becomes a function that looks up the thread specific information. Each thread has a block of data which is associated where the errno is specified. This is an example of why it is important to compile the pthread specific version of the c library.

### Mutual Exclusion
When two threads are accesing data at the same time, they can both read the data before they modify it. So the modification of one of the threads might be overwritten. Or in a data structure like a linked list, one thread might delete a node while the other thread is traversing through the list. 

To fix this, we have something called mutexes, mutual exclusions. Mutexes are essentially a flag which a parituclar thread my hold which says that it is the only one allowed to edit a particular shared data structure. 

```c
int  pthread_mutex_init(pthread_mutex_t  *mutex, const  pthread_mutexattr_t *mutexattr);

int pthread_mutex_lock(pthread_mutex_t *mutex);

int pthread_mutex_trylock(pthread_mutex_t *mutex);

int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

This is how we craete a mutex. The mutexattr is how we specify some settings, but we can use NULL to obtain the default. the mutex will be written into the mutex pointer. Then we can use pthread_mutex_lock to flag that mutex. Then any other thread which tries to lock that mutex will block until the thread who locked it unlocks it.

If there is not some special sequential structure the threads are following, you can assume that after a query is made on a shared data structure by your thread, that the information from that query only pertains to the moment in time where the thread was called. 

In other words you should not maintain assumptions about the current state of the shared data structure after a query is completed and the mutex is unlocked. 

### Condition Variable

What if we have a stack and we want to pop something, but when we go to pop, the stack is empty. The solution is to wait until something about the stack has changed, in this case we want to wait until the stack is non-empty.

This is where the role of the condition variable comes in. The role of condition variable is to block a thread until a condition becomes true. What it actually does is block the thread until another thread tells it that the thread the condition variable is blocking that it's condition is satisfied. 

```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

int pthread_cond_init(pthread_cond_t *cond, pthread_condattr_t *cond_attr);

int pthread_cond_signal(pthread_cond_t *cond);

int pthread_cond_broadcast(pthread_cond_t *cond);

int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);

int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);

int pthread_cond_destroy(pthread_cond_t *cond);
```
Init and destroy are self explanatory. Wait will be called if the thread has already acquired a lock and is waiting for some condition to be satisfied. During the duration of the wait, the mutex is released. Once the condition is signalled, the lock will be reinstated and the thread that was waiting will continue its execution. 

Signal will wake up at most one thread. Broadcast will wake up all threads that are waiting. 

Between the time that the signal was send and the thread wakes up and locks the mutex once again, another thread can edit the mutex variable. So once the thread wake up from waiting, it must check again whether the condition is satisfied or not, to be sure that the wake up was not "spurious."

### Semaphores
This is an older and more powerful/flexible than mutex and condition variables. But they are difficult to use effecively and understand.

Mutexes and Condition Variables are easy because they are very simple. 
- Only the thread that locked the mutex can unlock it.
- Condition variables are associated with a particular mutex

A semaphore is essentially an integer with two operations:
#### P / decrease / wait
Decreases the semaphore value by one. Unless the value is already 0. If the value is already 0, then wait will wait for the value to become nonzero, then decrement and top waiting

```c
void wait(int sem)
{
    while(1)
        if (sem > 0)
        {
            sem--;
            return;
        }
}

```
This is not real code, but this is kind of what wait is doing. will keep on checking if the sem is able to be decremented (waiting for sem to become nonzero) then it will decrement and return once the check suceeds. 

#### V / increase / post
Increases the value of the sempahore by 1. This is all they do.
```c
void post(int sem)
{
    sem++;
    return;
}
```

One thing to note about both operations is that, these operations are atomic. This means that if two threads post at the same time, it is guaranteed that one will not overwrite the other, sem will be increaesed by 2.

##### Semaphore as a Mutex Lock
Immediately we can model mutex using a semaphore. We initialize a semaphore with value 1, a nonzero value equivalent to a mutex being unlocked. Then we can lock the semaphore by waiting. This will decraese the semaphore value to 0 making it block other threads. Then we can use post to increment the value and allow other threads to use the mutex/semaphore. 

With mutexes we have a requirement that only the locker of a mutex should unlock it but with sempahores there is no such rule. 

##### Semaphore as a Condition Variable
To make a condition variable we set the initial value of the semaphore to 0. Then any thread which wants to wait for a condition to be satisfied will call wait(). This forces it to wait for another thread to call post() in order to incrase the value so that it can increment and top waiting for the condition to be fulfilled. 

###### But there is one critical difference:
 With a condition variable, if signal is called, nothing happens. But if we do post, the semaphore will always be incremented. We can get around this by having another semaphore which represents the number of threads currently waiting. The waiting threads would post to this semaphore and then the signalling thread would check if the second semaphore is greater than zero, and only then would it both wait for the second semaphore and post to the first one. 

##### Other differences:
Semaphores can be used from with in signal handlers while mutexes cannot. This is because you don't know if the signal handler was called under the appropriate conditions for a mutex. This includes the thread which is calling the signal function, whether or not the mutex is locked at that time, etc.

### Signal Handling in Threads
##### What happens when a signal is sent to a multi-threaded process?
One of those threads should handle that signal. Except when the disposition is to ignore, which leads to nothing happening, or when it is terminate which terminates the process. 

The signal handler will be called on a random thread (not deterministic which thread gets it). Other threads will continue undisturbed.

##### Things to worry about
If we set a global veriable, there might be another thread using it at the same time. We cant use Mutex because we don't know if the thread was already locked.

###### Solution:
Use a signal handler thread through signal Masks

#### Signal Masks
This specifies which signals are blocked on a particular thread. So only a thread that has the signal unblocked will recieve that signal.

This means we can have a signal handler thread where on every other threads all signals are blocked. 

When we create a thread it inherits the mask of the thread that creates it. We can control which signals go to which threads.

#### pthread_kill()
This will let us send a signal to a specific thread from within the same process. 

You can send a signal to stop a blocking system call. This happens by setting a benign signal handler that is just an empty void function. 

### Deadlock
Deadlock can occur when a process ends up locked and is unable to unlock itself. The program is waiting for an event that will never occur.

The simplest deadlock is if you wait for a mutex twice. No other thread can unlock a mutex but you. But now you can't unlock it because you're waiting for it to be unlocked again! 

Another example is if you have two mutexes and then you have two threads. Thread one might lock mutex b then a while thread two does so in reverse order. If thread one locks a, and then thread two locks b, then thread one will be waiting for mutex b to be unlocked while thread two is waiting for mutex a to be unlocked. Deadlock! Both are waiting for the other to finish. The potential for deadlock is there, so this is a bad design even if it is unlikely.

#### Conditions Needed For Deadlock
- Mutual Exclusion
- holding one resource and also waiting for another resource
- no preemption, so another thread cannot unlock a resource another thread has
- circular graph of dependence for being released from a wait

All four of these must be true to be in deadlock. If any of them is false, deadlock can be avoided.

##### Steps to Avoid
- If you need two locks, get them in the same order every time.
- Keep critical spaces simple and short, make sure you're not trying to acquire resources within resources. 

In general there is no general way to detect and fix deadlocks a hundred percent of the time. 

### Memory Consistency
Think of scenario with variables x, y with both being zero. And two threads: a, b. thread b does x = 1, print(y). thread a does y = 1, print(x). 

The output here is non deterministic. It could be 1, 1, or 0, 1.

#### Sequential Consistency
This means that there is a linear sequence of memory operations. But this is not guaranteed by any multicore architecture. 

This is because each CPU has its own L1 cache. When we do a write followed by a read, the write goes into the cache, but it does not immediately go out the main shared memory. We also don't want to wait for the write to finish. To the CPU currently executing, there is no relationship between writing to x and then reading from y so it does not wait for the change in x to be reflected in every CPU. 

The writes may succeed for the CPU's cache, but the change doesn't arrive to the other CPU so the reads may be from the unupdated order. 

So now, our process can print out 0, 0 because memory, while shared, is not exactly the same at every given point of time because CPU's use caching.

To solve this, we can use memory fences to wait for writes into memory to complete so all programs can see the full story of what's going on in memory. This will get us to some level of sequential consistency. 

##### Another Scenario
Let's say we have two threads. 

##### Thread 1:
```c
for (int i = 0; i < 100; i++)
{
    X = 1;
    print(X);
}
```

Clearly, any compiler will set X to 1 before. So equivalently:
```c
X = 1;
for (int i = 0; i < 100; i++)
{
    print(X);
}
```
##### Thread 2:
```c
X = 0;
```
Thread 2 with the unoptimized thread 1 code might print 100 ones or 99 ones and a 0 somewhere. But if we used the compiler optimized code, it will print all 1s uptill soem point and then all zeros after that. This is wildly different behaviour than what we would expect. 

The optimization of the thread 1 code has completely misinterpreted the intended purpose of the program. 

###### So how to compilers deal with this?
This code is an example of a data race. Data races have undefined behaviour. These optimizations are valid because when there is a data race, there are no rules as to what the language should do, its undefined behaviour. This is why mutex or semaphore should be used whenever you have two threads accessing the same variable/data. 









