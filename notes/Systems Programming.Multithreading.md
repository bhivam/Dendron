---
id: iw530kbpfgidngxm9fc3ahm
title: Multithreading
desc: ''
updated: 1682726749559
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
