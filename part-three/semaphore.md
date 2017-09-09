# Semaphore

A semaphore is useful for working with “train-like” objects, that is, objects where what you care about is whether there are either zero objects or more than zero. Buffers and lists that fill and empty are good examples. Semaphores are also useful when you want a thread to wait for something. You can accomplish this by having the thread call sem_wait() on a semaphore with value zero, then have another thread increment the semaphore when you’re ready for the thread to continue.

A counting semaphore contains a value and supports two operations "wait" and "post". Post increments the semaphore and immediately returns. "wait" will wait if the count is zero. If the count is non-zero the semaphore decrements the count and immediately returns.

An unnamed semaphore can be created by declaring a `sem_t` type variable, and initialize it with `sem_init`,the semaphore will be pointed at the address given in the first argument. `pshared` indicates whether the semaphore will be shared between threads of a process, a non-zero value meaning the semaphore is shared between processes, and should be located in a region of shared memory, and a value of `0` meaning that it is shared between threads. The last argument contains the initial value for the semaphore.

`int sem_init(sem_t *sem, int pshared, unsigned int value);` 

```
#include <semaphore.h>

sem_t s;

int main(int argc, char** argv)
{
	sem_init(&s, 0, 10);
	...
}
```

There are three operations that a semaphore can do:

* `sem_wait()`

* `sem_post()`

* `sem_destroy()`

The usage for `sem_wait()` and `sem_post()` is pretty straightforward, first we announce our intention of entering the critical section by calling `sem_wait()`, and by calling `sem_post()` we signal our exit from the critical section:

```
sem_wait(&s);
// Critical Section
sem_post(&s);
sem_destroy(&s);
```

Lastly we call `sem_destroy()` to relase the resources of the semaphore.