---
id: 2gm3imgm3sf3x3f2vo0ew9v
title: Network Communication
desc: ''
updated: 1682718536915
created: 1680377631394
---

Fundementally, this is about how we send information between processes on different computers. 

### Inter Process Communication in One Computer
Two processes that share a parent can hold two ends of a pipe. Data entered into one end will be ready to read out of the other end. 

For two way communication between processes we need at least two pipes.

We can communicate using normal files, just reading and writing to them from different processes. We can `dup2` them into our STDIN_FILNO and STDOUT_FILENO.

Another option is to use FIFOs which are just named pipes. They are pipes that exist independently of a specific program. One program can write into it and another can read out. 

## Client Side
#### **What if we want info from google.com?**
Because google.com isn't on our computer, we need some way to interface with the process on google's servers through an API (Application Program Interface).

So we need an API (sockets or streams)

We also need a way of way of saying what process we are communicating with

We also need a way to standardize the format of the data we're exchanging. This is called a protocol.

#### What is a Protocol
It defines what actions are available to a party, what the appropriate responses are, and then how they can be represented.

### Identifying the Process
The main idea here is using **addresses** to identify the process we are communicating with on another machine. The main network used is the internet, and the internet has a specific way of defining how we identify processes. 
#### Internet Addresses
- First part is a "machine" address or IP address
  - IP V4 addresses are 32-bit integers in dotted quad format. (four number from 0 to 255 seperated by ".").  
  The problem here is that this only leaves us with about 4 billion addresses.
  - Because of this we want to transition to IP V6 which is 128 bit addresses. This is written in 8 groups of four hexadecimal numbers.
  1234:abcd:etc
- Second part is a port number which is a particular process or service running on the machine. 
  - Certain well known services have "well-known" numbers. For example, HTTP is port 80. 

#### Domain Names
IP addresses are not very user friendly so we have come up with the solution of **domain names**. 
DNS (domain name service) is a distributed hierarchical database that store information about domain names. The IP address can be associated with a domain name and when we use a domain name in a web browser, our browser uses the DNS to look up the correct IP address and then will connect to that address. 

Traditionally, domain names are written as a string of "." seperated words. Example: ice.cs.rutgers.edu. 

"A" record give IPV4 addresses for domain names
"AAAA" records give IPV6 addresses for domain names

**NOTE:** DNS records are not just for IP address names, there are a variety of services which utilize DNS. 

```c
int socket(int address_family, int type, int protocol)
```
A socket is an endpoint for communication and it is accessed via a file descriptor. We can use it to send and recieve messages. Typically to get file descriptors, we use `open()`. But open is used for our disk system and disk systems typically don't include network stuff. "they could" **What does this mean?**.

#### address_family
We are asking type of network standard we are using e.g. IPV4 vs IPV6. But there are many other standards for networking like bluetooth etc, but these two are the primary standards. What sort of network connection are we talking about?

**Possible Address Families**
- AF_INET - IPV4
- AF_INET6 - IPV6
- AF_LOCAL - communication in the device
- Too many others to talk about

#### type
This determines the type of socket. 

**Possible Socket Types**
- SOCK_STREAM - Be sure that the data will be recieved in the order that it was sent
- SOCK_DGRAM - Cannot be sure that the data will arrive in a particular order and there is a fixed amount of data. 

#### protocol
If the first two are not unique, what are we talking about? This is answered by the protocol. This is different from the format of the data. Generally this will be 0.

#### value returned
We get a file descriptor or an error on -1. 

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen)
```
Before we can do anything with the address we get from `socket()`, we need to connect it. Because at the moment the file descriptor from `socket()` is not connected to anything. `connect()` puts a wire into the socket.

**sockfd**
This is the socket file descriptor we would like to connect.

**addr**
This refers to the combination of address and port name.

**addrlen**
Length of the address. This is a pretty weird thing to specify since structs usually are of a fixed size. This is because sockaddr is not really a fixed struct. Each address family has a different amount of data it needs to specify an addres. Think IPV4 vs IPV6.

But typicially its better to use a function called `getaddrinfo` to acquire the correct struct. You can go through all the steps yourself to select the correct address type, then look up the address, and convert it to network order, but why do that if you don't have to. This also makes your program more future proof and stable. 

```c
int getaddrinfo(const char *node, const char *service, const struct addrinfo *hints, struct addrinfo **res);
```
**node**
This is the domain name. If you pass NULL to this it will assume the local host.
**service**
This is the port.
**hints**
You can pass in more information if you'd like. As an example, hints can be used to specify things like the address family you'd like and the type of socket you're using. 
**res**
The reason for res being a double pointer is that `getaddrinfo` wants to place a struct of its choice into res. To this end, they allocate the required space for the struct. 

This will be the head of a linked list of addrinfo structs that you can go through. 


### Full Duplex
Streaming sockets are full duplex. So you can both send and recieve. Outgoing and incoming data do not intermingle. If some data is written to the socket by the client, the client's file descriptor will not reflect that whereas the server's file descriptor will see it. 

This is important depending on the architecture the protocol the service we connect to is using. For example with HTTP, there is a request-response architecture. This means when there is a request for data is made by the server. It reads this request and then it has to know that we won't send a response until we get a response first. So it cannot read until it has sent a request. If it reads before sending a request, we enter a deadlock where we, the client, are wating for a response and the server is also waiting for a response. 

## Server-side
How do we accept an incoming connection? `connect()` actively tries to connect to a process that is listening for connections. How does google.com wait for new connections?

Again we use socket() to create a file descriptor through which we can use to plug a connection into. 

But instead of connect() we need a way to tell the operating system that we will handle all attempted connections to a port. For this we use bind. This will associate a socket with a port on the device.

Then we need listen() which will listen for incoming connection requests. 

Finally we use accept() which says to accept an incoming connection request and return a new socket which is for that particular connection. We can now use read() and write() on this new socket to send and recieve data from the server. 

```c
int bind(int socket, struct sockaddr *addr, socklen_t, socklen)
```
Bind is used to associate a port with a specific socket. We can do this before doing `connect()`, but generally we don't care about connecting with a particular socket from the client side of the connection. `bind()` is typically used to associate a socket with a port before using `listen()` on that socket.

```c
int listen(int socket, int queue_size)
```
`listen()` takes the socket, now associated with a porth through bind and it sets up the ability to listen for new connections. queue_size is how many clients that will be allows to be in the queue for being accepted. If the queue is full and a client trys to connect, they will be denied and they will see an error.

```c
int accept(int socket, struct sockaddr *remoteaddr, socklen_t *remoteaddrlen)
```
This is where we actually block and wait for new connections to come. This returned a file descriptor which is the socket associated with the particular connection `accept()` got. 

#### remoteaddr
This is where information about the remote host is stored. We can put a specific type of sockaddr struct in here if we know what type of connection we are going to recieve, or we can use struct sockaddr_storage which is as large as the largest sockaddr struct. You must cast the sockaddr_storage to a sockaddr

#### remoteaddrlen 
At first you will give a pointer to the remoteaddr you have passed, and it will be populated with the size of the remote address which is accepted.

## Inter Device Communication

### First Telephones
First phones were just two microphones and speakers connected by a wire. 

But we need way more direct connections the more people we add. So if a 100 people needed a direct conection to a 99 other people, then we would have 9900 wires among these people, in total. 

To get around this there was a central office where everyone's phone is connected to. They would connect you to any other person whom you would need to talk to. So now you just need one connection that goes to a central point and at that central point two people are connected. (circuit switching)

##### Problems:
- People are guaranteed a certain amount of bandwith for some amount of money because there is a maximum amount of bits that can be sent on the network. 
- Often there is a maximum number of people that can connect to a network at this point. 

Unlike phones computers are bursty. They send a lot of data at once and not a small amount counstatnly over time like a phone call. So this idea of a fixed amount of through put doesn't make sense for computer networks. 

##### Now we have packet switching:
- Data is broken down into fixed dize chunks. 
- each packet says where it is going
- networks will forward packets to their destination as bandwith becomes available. 

It's on the recipient to assemble this into the correct order.

### TCP
This is the internet streaming socket protocl is called TCP (Transmission control protocol)

In TCP the IP address of both processes are needed. This allows us to have multiple simulataneous connections between two machines

Modern web browsing wouldn't be possible without this. When we download web pages, HTTP will use multiple connections to download all the parts of the webpage. THe way that the connections are told apart is by the client port number. 

So connect() does bind the socket to a port but not to a specific one, its random. But we need each connection to have a specific port to distinguish connections

## Layering

Writing network code is very hard if you're doing everything at once. This is why we have a layered systems. Application progammers write code at the top of this systems. We use the many facilities provided to us by the OS and network. 

## OSI Model

This is a theoretical model of network communction. It breaks them down into 7 layers. These generally don't correspond to real life networks. 

### Layers
#### 1. Application
We are asking what messages we need to communicate, what is the expected response to a given message. 
- Examples:
  - Tic Tac Toe Game Protocol
  - HTTP (WEB): GET, POST, etc
  - SMTP (EMAIL)

#### 2. Presentation
Given some messages, how do we encode these messages in a way that we can transfer them across the network. We specify this in TTT by using four capital letters at the beggining, using '|'. This used text encoding. Some can use just bits themselves.

#### 3. Session
When do we begin and end connections to remote hosts? This is what specifies how computers will connect to eachother. Who is reponsible for creating connections, how many connections do we have. 

#### 4. Tansport Layer
This is code we do not write. This is asking how we can maintain a connection, what type of connection. How do we deal with ordering, dropped packets, etc. This part of the process is handled by the OS. 

#### 5. Network
How does information actually arrive from my computer to another? What if that computer is not in my immediate network. How are packets routed from one place to another. This is different from the transport layer because the transport layer is assuming that it can get data from one place to another, it is simply asking how it will create a "connection" and defines what it means ot have a connection. This layer is handled by the network and the series of routers between our computer and the host we are connecting to. 

#### 6. Data Link
How does data get from one computer to the immediate next computer in the network. These are things like ethernet or WIFI.

#### 7. Physical
This is how we actually get an ethernet connection. Radio Waves, wire, optical lasers, etc. 

 #### Internet Model Layers
 ##### 1. Application - (application + presentation)
 ##### 2. Transport - (Session + Tansport)
 ##### 3. Internet - (Network)
 ##### 4. Physical - (Data Link + Physical)


#### Why do we care about abstraction?
- Less to worry about
  - application developers don't need to worry about lower layers
  - network engineers don't need to worry about application
- flexibility
  - easy to create new applications for TCP/IP or UDP/IP
  - easy to create new networks that support IP

  