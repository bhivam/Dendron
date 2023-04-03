---
id: su7e7rqwpxtr3gblspt71pm
title: Pipe and Dup2
desc: ''
updated: 1680292064406
created: 1680291866288
---

## Pipe
You have some interprocess communication by having two file descriptors which refer to a buffer managed by the OS. One file descriptor is meant for writing into that buffer, and the other one is for reading what was written into it. 

This allows us to have the output of one program flow into another. 

#### **note**:
The read end of the pipe will not recieve EOF until the write end has no open file descriptors.  

A pipe is broken when the read end is closed before the write end.

