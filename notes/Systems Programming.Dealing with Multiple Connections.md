---
id: new2i7guzbtp5cefo4tta3j
title: Dealing with Multiple Connections
desc: ''
updated: 1680567091023
created: 1680562750847
---

```c
connect()
```
You give an array of file descriptors and it will return one once it is active. 

### Easier way: Multitasking

#### Method 1:

What if we fork every time that accept() gets a new connection and then handle that conneciton in a child process. 
- We need to make sure that we are reaping the child processes so that they don't become zombies. 
- But how can we know when the child processes are gonna end? The answer is SIGCHLD. When SIGCHLD is sent to the parent, we can use wait in the signal call to wait for that child. 
##### Problem:
We might not get a SIGCHLD every time:
- if two children end at similar times
- if a child ends during the signal call

```c
int waitpid(pid_t pid, int *wstatus, int options)
```
We can wait put WNOHANG for our child process and -1 for the pid. This will allow us to call waitpid in a loop to clean up all zombie processes until waitpid returns 0, at which point there should be not more processes.

WNOHANG means that it shouldn't block if there are no children and pid = -1 means that it won't look for any particular process. 

#### Method 2:
We can also have many child process preforked which and have them all call accept. The OS will pass a new connection to any one child calling accept.