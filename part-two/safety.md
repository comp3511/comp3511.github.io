## Introduction to Thread Safety

In the context of multi-threaded program, **thread safety** is achieved through meeting several conditions including:

* **Re-entrancy**

Writing code in such a way that it can be partially executed by one task, reentered by another task, and then resumed from the original task. This requires the saving of state information in variables local to each task, usually on its stack, instead of in static or global variables.

* **Mutual exclusion**

Access to shared data is serialized using mechanisms that ensure only one thread reads or writes the shared data at any time. Great care is required if a piece of code accesses multiple shared pieces of data—problems include race conditions, deadlocks, livelocks, starvation, and various other ills enumerated in many operating systems textbooks.

* **Thread-local storage**

Variables are localized so that each thread has its own private copy. These variables retain their values across subroutine and other code boundaries, and are thread-safe since they are local to each thread, even though the code which accesses them might be reentrant.

* **Atomic operations**

Shared data are accessed by using atomic operations which cannot be interrupted by other threads. This usually requires using special machine language instructions, which might be available in a runtime library. Since the operations are atomic, the shared data are always kept in a valid state, no matter what other threads access it. Atomic operations form the basis of many thread locking mechanisms.

### Race Conditions

```
#include <pthread.h>
void* do_work(void* ptr) {
    int i = *((int *) ptr);
    printf("%d ", i);
    return NULL;
}

int main() 
{
	// Each thread gets a different value of i to process
	int i;
	pthread_t tid;
	for(i =0; i < 10; i++) {
		pthread_create(&tid, NULL, do_work, &i); // ERROR
	}
	pthread_exit(NULL);
}
```

The above code illustrates the problem of a race condition, the program is supposed to start ten threads with values 0 through 9, however the threads are being created while the value of i is changing, which causes the program to misbehave.

In the next section where we will investigate into the topic of **Synchronization**, we will find out how to deal with problems like this and its solutions to it.