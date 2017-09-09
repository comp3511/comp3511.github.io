# Mutexs

A **Mutex** is a lock that one can virtually attach to some resource. If a thread wishes to modify or read a value from a shared resource, the thread must first gain the lock. Once it has the lock it may do what it wants with the shared resource without concerns of other threads accessing the shared resource because other threads will have to wait. Once the thread finishes using the shared resource, it unlocks the mutex, which allows other threads to access the resource. This is a protocol that serializes access to the shared resource.

The code between the lock and unlock calls to the mutex, is referred to as a **critical section**. Minimizing time spent in the critical section allows for greater concurrency because it potentially reduces the amount of time other threads must wait to gain the lock. Therefore, it is important for a thread programmer to minimize critical sections if possible.

The variable type that enables the lock and unlock mechanism to threads is `pthread_mutex_t`, we first call the `pthread_mutex_init()` function to initializes the mutex variable or we can initialize it by setting it to the global variable `PTHREAD_MUTEX_INITIALIZER`.

`int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr)`

We first pass the mutex variable as the first argument, and an attribute variable as the second argument that specify the behavior when a mutex is shared among processes.

To perform mutex locking and unlocking, the pthreads provides the following functions:

* `int pthread_mutex_lock(pthread_mutex_t *mutex);`

* `int pthread_mutex_trylock(pthread_mutex_t *mutex);`

* `int pthread_mutex_unlock(pthread_mutex_t *mutex);`

Each of these calls requires a reference to the mutex object. The difference between the lock and trylock calls is that lock is blocking and trylock is non-blocking and will return immediately even if gaining the mutex lock has failed due to it already being held/locked. It is absolutely essential to check the return value of the trylock call to determine if the mutex has been successfully acquired or not. If it has not, then the error code `EBUSY` will be returned.