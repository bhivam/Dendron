---
id: 7khqimzxhqyi6qok7ry48eu
title: Fork and Exec
desc: ''
updated: 1680054953236
created: 1680051148475
---

### Overview
`fork()` creates new processes and `exec()` changes the program of a process.

We use `fork()` to create a child process and have the child process call `exec()`

### Why do we have two functions for this instead of one?
Because we want to use `fork()` to configure things like open files, environment variables, then run exec, and finally do any cleanup that's needed.

In the parent we might do some extra stuff and then wait for the child process to terminate. 

## Exec
#### **Program**:
This is a sequence of instructions that can be executed by (typicially) the CPU.

#### **Process**:
A process is what executes your program and it has the following properties things and properties:
- instruction pointer
- contents of the registers
- contents of memory
- enviroment
  - working directory
  - environment variables
  - open file table

#### Fun Fact:
The program loader will move things around in memory and make changes to your program and system programs which makes it difficult for malicious actors to do bad things.

### How do we change the program we are running?
We do this through an exec function. exec refers to a family of functions which replace the current process with a new one
- exec does not create a new process.

If `exec()` is successful it does not return. All state of the current process is lost:
- stack and heap are reset
- globals are loaded from new program
- instruction pointer set to start of new program

Most of the environment is carried over:
- working directory stays the same 
- most open files remain open when open file table is copied
  - **Exeption:** When a file is opened with the option O_CLOEXEC, it is closed when exec() is called so that it won't be copied over.

When `exec()` functions return -1 on failure, they set the errno, but if they return anything at all this is some type of faiure.

### Variations of `exec()`
- #### `execl()`
This takes a variable number of arguments:  
`exec(path_to_executable, arg1, arg2, ..., NULL)`
The NULL at the end is what tells us if the argument list is over. 
- #### `execv()`
An array of arguments is passed:  
`exec(path_to_executable, args)` where `args` is a NULL-terminated array of strings.  
By convention, the first argument is the name of the program (this is whatever the user typed to specify the program).
- #### `execvp()` or `execlp()`
This tells exec to look for the executable in the search paths. This uses the path environment variable. 

- #### "e" variants
This will allow you to set the environment variables of the program instead of inheriting them from the current environment. 

## Fork
#### Where do processes come from?
With the exception of the first process (init), all other processes come from a parent process. 

The function we use to create a new child process is called `fork()`. When `fork()` is called everything is copied from the parent process to the child process.
#### **Note:**
When we use fork, open file descriptors are duplicated. It is important to know that these file descriptors are not the same for the parent and child process. The child has its own table of open files, just that this table is initially copied from the parent process. This means that every process open in the parent is open in the child, files can be open in more than one process.

For example, the shell uses `stdin`, but then when we run a program, whatever subprocess we are running uses `stdin`.

If a file is opened in the parent and child, and then the child reads and the parent reads, they will not get the same bytes, one of them will get the first set of bytes and the other will get the next set. This is because while each process maintains their own table of open files, the OS manages information about open files such as the offset. So when read is called twice, even by different programs, the OS will just give the bytes in whatever order the calls came in.

They share the offset so you need coordination between them. So typically you just don't do anything with the file in the main process until the child is done.

### Fork Return
The first divergence between the child and parent process is that the `fork` call will return 0 if it is the child process, and something nonzero if it is the parent. 

So typically you will get whatever `fork()` returns and then if it is 0 you will run a new program with exec().

#### **They both run concurrently, so what does the parent do during this time?**
(As an aside, we say concurrently and not simultaneously because there may only be one CPU, so the CPU's time gets divided up between different processes to complete them side by side.)





