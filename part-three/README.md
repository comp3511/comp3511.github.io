# Synchronization

In order to write any kind of concurrent program, you must be able to reliably synchronize the different threads. Failure to do so will result in all sorts of ugly, messy bugs. Without synchronization, two threads will start to change some data at the same time, one will overwrite the other. To avoid this disaster, threads must be able to reliably coordinate their actions.

Synchronization is the method of ensuring that multiple threads coordinate their activities so that one thread doesn’t accidently change data that another thread is working on. This is done by providing function calls that can limit the number of threads that can access some data concurrently.

In the simplest case (a Mutual Exclusion Lock—a mutex), only one thread at a time can execute a given piece of code. This code presumably alters some global data or does reads or writes to a device. For example, thread T1 obtains a lock and starts to work on some global data. Thread T2 must now wait (typically it goes to sleep) until thread T1 is done before it can execute the same code. By using the same lock around all code that changes the data, we can ensure that the data remains consistent.