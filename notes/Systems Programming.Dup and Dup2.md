---
id: 8xgmre0a8u199rhypuykl3v
title: Dup and Dup2
desc: ''
updated: 1680405240581
created: 1680292077289
---

An open file description is a record that is stored by the OS. It keeps track of information that the OS needs to know in order to read or write the file.

#### information in open file description
* Where you are in the file
* How many references to this file descriptions there are
* what the file is: (device, actual file, process, socket, etc)

Important to know that this is different different from an open file *descriptor* which is a reference to an open file in a single process.

You can get multiple references to the file description. 
### Examples
```c 
int dup(int old_fd)
``` 
This will take in an old fd, and return a new one, both referring to the same file. Closing one of these file descriptor does not close the other.

The original purpose of dup() was to close one of STDIN, STDOUT, STDERR and the immediately call dup(). This is because dup() uses the lowest available file descriptor, so when one of these is closed and we call dup, the old_fd now takes its place. 



```c
int dup2(int old_fd, int new_fd)
```
This is very similar to `dup()`. The only difference is that we can specify what the new file descriptor is. So new_fd will refer to the same file as old_fd. This is how we redirect inputs and outputs of processes.

By using this before exec, we can change what STDIN or STDOUT refers to so the program will run what we want it to.

dup2 will close the new_fd if it already refers to an open file descriptor. This is good because it does it all in one step, making sure that no other thread in the standard process can occupy the new_fd before dup2 does. 