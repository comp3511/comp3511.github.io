# Homework 1

The following is the description for the task you need to 
perform for this assignment. This homework is the first installment
of the three for the Operating System course (COMP3511).

[Download homework1.zip](https://www.cse.ust.hk/osprojs/homework1.zip)

The purpose of the first assignment is to introduce students to
Interprocess Communication / IPC in short, there are a couple of ways
for processes to communicate with each other, forked or independent 
process.

There will be three exercises for you to warm up:
1. Simulate a bank transaction with shmget() api
- This task requires you to get familiar with the shm (shared memory) APIs
provided in standard library in C language. These system call function includes:
 * `shmget()`
 * `shmat()`
 * `shmdt()` 

2. Simulate a bank transaction with Socket
- This task requires you to get familiar with socket api, this api will
be useful not only in interprocess communication but also cross network 
communicate between programs. Similarly, the system call function includes:
 * `open()` for opening a file descriptor
 * `socket()` for instantiating socket variable
 * `bind()` for binding network address
 * `accept()` for accepting request from socket
 * `listen()` for awaiting incoming connections from socket
 * `write()` for writing content / data to socket to the other end

3. Simulate a bank transaction with IPC message queue
- This task requires you to get familiar with IPC message queue, this api 
allows you to share a common key just like share memory using a key_t variable
and create a new message queue that is shared between your process and the 
other end of your program. the system call function for this exercise includes:
 * `msgget()` for obtaining a message id that is unique to your message queue
 * `msgsnd()` for sending message in the message queue for the other end to obtain using a common key

Main task:
The main task for HW1 lies in the main.c, which has been purposed to serve
as a http server to listen for incoming http request, and returns a webpage
for basic user interaction, the task is more or less the same as the previous
exercise, you will be creating a http server using socket in C and specify its
protocol to be TCP only, the webpage itself has been written for you and so does
the parsing for the query string, what is difference this time will be allocating 
shared memory using the mmap() system call api, to create a shared ledger for
all accounts in the simulated bank, and by allowing user to input their recipient
account number and the amount they wanted to transfer to the particular account.

