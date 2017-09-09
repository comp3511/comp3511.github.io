# Message Queues

**Message Queues** is yet another way for IPC to take place in Unix-like Operating Systems. Messages that being send to the queue works in a **FIFO** manner, each message queue is uniquely identifiable via the use of `key_t` variable containing a numeric value. A process can instantiate a new message queue or connect to an existing queue, hence allowing multiple processes can access the information in the queue simultaneously.

A message queue can be created with the following system call:

`int msgget(key_t key, int msgflg);`

Where `msgget()` returns the message queue id on success, otherwise `-1` if it sets off an error.
Arguments that are being pass to the system call are a system-wide unique identifier (for creating and connecting to the queue) and a bit flag to control the permission of the message queue.

Once a message queue has been created, the process is ready to send and receive messages. However, there is one caveat which is the format for the message. Each message is divided into two parts, the type of message and the content of the message itself. Note that the `message_text` can be a structure itself if you would like to store some meta data.

```
struct message_buffer 
{
	long message_type;
	char message_text;	
}
``` 

### `msgsnd()`

So after we have formatted the message, we can utilise the next system call function to actually send the message.

`int msgsnd(int id, const void *msg_ptr, size_t msg_size, int msg_flag);`

Usage for this function is to first pass the message queue id returned by `msgget()` as a parameter, and a pointer that points to the address of your _formatted message_ and its size, it can be obtained by calling `sizeof(message_buffer)` for instance and finally the bit flag for message permissions.

### `msgrcv()`

On the receiving end, you would first join the message queue by calling `msgget()` and do the following system call:

`int msgrcv(int id, void *msg_ptr, size_t msg_size, long msg_type, int msg_flag);` 

The return value of `msgrcv` is an integer value that indicates whether the message is received, `-1` meaning there is an error occurred and the number of bytes received otherwise.

## Example

### Sender

```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct my_msgbuf {
    long mtype;
    char mtext[200];
};

int main(void)
{
    struct my_msgbuf buf;
    int msqid;
    key_t key;

    if ((key = ftok("sender.c", 'B')) == -1) {
        perror("ftok");
        exit(1);
    }

    if ((msqid = msgget(key, 0644 | IPC_CREAT)) == -1) {
        perror("msgget");
        exit(1);
    }
    
    printf("Enter lines of text, ^D to quit:\n");

    buf.mtype = 1; /* we don't really care in this case */

    while(fgets(buf.mtext, sizeof buf.mtext, stdin) != NULL) {
        int len = strlen(buf.mtext);

        /* ditch newline at end, if it exists */
        if (buf.mtext[len-1] == '\n') buf.mtext[len-1] = '\0';

        if (msgsnd(msqid, &buf, len+1, 0) == -1) /* +1 for '\0' */
            perror("msgsnd");
    }

    if (msgctl(msqid, IPC_RMID, NULL) == -1) {
        perror("msgctl");
        exit(1);
    }

    return 0;
}
```

### Receiver

```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct my_msgbuf {
    long mtype;
    char mtext[200];
};

int main(void)
{
    struct my_msgbuf buf;
    int msqid;
    key_t key;

    if ((key = ftok("sender.c", 'B')) == -1) {  /* same key as sender.c */
        perror("ftok");
        exit(1);
    }

    if ((msqid = msgget(key, 0644)) == -1) { /* connect to the queue */
        perror("msgget");
        exit(1);
    }
    
    printf("ready to receive messages.\n");

    while(true) {
        if (msgrcv(msqid, &buf, sizeof buf.mtext, 0, 0) == -1) {
            perror("msgrcv");
            exit(1);
        }
        printf("content: \"%s\"\n", buf.mtext);
    }

    return 0;
}
```

