---
id: yxjdv0v3xh2gxkm3r0umv5b
title: Processes and Signaling
desc: ''
updated: 1680377560525
created: 1680294296284
---

When executing a process (or "task"), there are two states:
- Executing
- Blocked/Suspended

For example terminal applications that rely on user input block until the user gives an input.

#### What is the computer doing while the process is blocked?
- It must be something because the computer is listening to my keystrokes, putting it in the buffer which STDOUT is written to etc. 
- When a program is blocking, the OS takes control
  - This allows us to do multiprocessing
- Older OS's would wait for the blocking condition to resolve
- New OS's will now execute some other process while they're waiting.

#### When do we switch tasks?
- Whenever we block.
- When there is an explicit yield operation, which allows other tasks to run. "Cooperative multitasking."
  - This is why we have a suspended state. It is suspended, but not blocked
  - This gives the programmer control over when their program switches out. Each process controls when it gets suspends.
  - This can also be a problem if the process does not suspend often enough to give other tasks time to execute. This is called "starvation."

### Preemptive Multitasking
Cooperative multitasking gives too much control to individual process. So operating systems like Unix will handle this for you.

It handles this by alloting some set amount of time for a process to run. If a process takes too long to run, then the OS will suspend that process and then give other processes some time to run and eventually come back to that program. 

Now as an end programmer you don't have to worry about managing CPU resources, the OS will handle context switching. 

## Signals

We have a set of signals identified by numbers. But we use constants to refer to them for our ease. 

Each process has some disposition for a given signal. A disposition is just what we do when the signal is recieved.

#### Types of Dispositions
- ignoring
- terminating (default)
- execute a "signal handler"
- terminate and dump core
  - terminate and put the contents of memory into a core file for debugging
- stop process (this will suspend the process but it cannot be freed up by the OS)
- continue process (this will continue a stopped process)

### Signal Handlers

#### What is a signal handler
A function that executes when a signal is recieved. This function can be called at any point of your code running, so there are some rules we have to adhere to in order to ensure that we have safe working programs.

#### Example of usage
If we ssh into a remote computer, ssh will "handle" the signals by forwarding them to the remote computer instead of interacting with them itself. 

#### Problem
A signal handler could be called in the middle of a library function like `printf()`. `printf()` is among a class of functions where there would be a problem if it were to be called while it was already running. Such functions are called **non reentrant**.

Non reentrant functions cannot be called in a signal handler because you don't know if the signal handler will be called while that function is running. 

Obviously if there is a function which uses global variables, then then having it called inside of itself would interfere with that global variable and cause unexpected behaviour.

There are a list of good, "reentrant," functions that can be used within a signal handler.

But often times signal handlers will just set flags so that your program knows that a signal was called. 

#### How do we set up Signal Handlers?
```c
typedef void (*signalhandler_t)(int)

signalhandler_t signal(int signum, sighandler_t handler)
```
This is the simple way to set up a signal handler. This is not reccomended because it is not very portable. In particular Linux and BST disagree about how this operates. 

We can see that signal() takes one int, specifying which signal we are talking about, and then a pointer to a function which takes one int and returns nothing. 

The pointer that signal() returns is a the previous disposition for that signum.
```c
struct sigaction {
    void     (*sa_handler)(int);
    void     (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t   sa_mask;
    int        sa_flags;
    void     (*sa_restorer)(void);
};

sigaction(int signum, const struct sigaction *act, struct sigaction *oldact)
```
This is the portable, and therefore better, way of setting up signal handlers.This one uses a struct to set more specific information about how to handle the signals and it will give you more control over signal dispositions than `signal()` will. For example, `signal()` will always restart any system calls if a signal is recieved in the middle of a signal call, wheras `sigaction()` will require you to specify that. 
#### How does wait work? 
It uses signals. It pauses the parent process and sets up a signal handler which executes upon SIGCHLD. SIGCHLD happens when a child process of a parent is done executing. 

#### Exception Handling
C doesn't really have exceptions. It just has signals. So when there is a floating point error, for example, the program recieves SIGFPE. We can catch this signal (exception), and do cleanup/error messages before we exit.

The SIGSEGV is also a signal. 
