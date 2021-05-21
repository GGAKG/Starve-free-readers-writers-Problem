# Starve-free-Readers-Writers-Problem
# Starve Free Readers-Writers Problem
This problem deals with multiple processes reading and writing to a shared resource synchronously with neither the readers or writers getting starved indefinitely. 

### Solution
All pseudocode is in C like syntax. We will use semaphores for mutual exclusion (mutex). Those semaphores, being used as locks, are all initialized in the released state.

We want fair-queuing between readers and writers in order to prevent starvation. To achieve that, we will use a semaphore named orderMutex that will materialize the order of arrival. This semaphore will be taken by any entity that requests access to the resource, and released as soon as this entity gains access to the resource.

The following are the pseudocode for construct
```c
semaphore orderMutex;      // Initialized to 1

void reader()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the reader can access the resource
  ...
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the writer can access the resource
}
```
Then, the reader and writer processes are as follows  
Writer Process

We will create a new semaphore named accessMutex that the writer will request before modifying the resource
```c 
semaphore accessMutex;     // Initialized to 1
semaphore orderMutex;      // Initialized to 1

void reader()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the reader can access the resource
  ...
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  P(accessMutex);          // Request exclusive access to the resource
  V(orderMutex);           // Release order of arrival semaphore

  WriteResource();         // Here the writer can modify the resource at will

  V(accessMutex);          // Release exclusive access to the resource
}
```

Reader Process

We want the first reader to get access to the resource to lock it so that no writer can access it at the same time. Similarly, when a reader is done with the resource, it needs to release the lock on the resource if there are no more readers currently accessing it.

Semaphore named readersMutex to protect the counter against conflicting accesses.

```c
semaphore accessMutex;     // Initialized to 1
semaphore readersMutex;    // Initialized to 1
semaphore orderMutex;      // Initialized to 1

unsigned int readers = 0;  // Number of readers accessing the resource

void reader()
{
  P(orderMutex);           // Remember our order of arrival

  P(readersMutex);         // We will manipulate the readers counter
  if (readers == 0)        // If there are currently no readers (we came first)...
    P(accessMutex);        // ...requests exclusive access to the resource for readers
  readers++;               // Note that there is now one more reader
  V(orderMutex);           // Release order of arrival semaphore
  V(readersMutex);         // We are done accessing the number of readers for now

  ReadResource();          // Here the reader can read the resource at will

  P(readersMutex);         // We will manipulate the readers counter
  readers--;               // We are leaving, there is one less reader
  if (readers == 0)        // If there are no more readers currently reading...
    V(accessMutex);        // ...release exclusive access to the resource
  V(readersMutex);         // We are done accessing the number of readers for now
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  P(accessMutex);          // Request exclusive access to the resource
  V(orderMutex);           // Release order of arrival semaphore (we have been served)

  WriteResource();         // Here the writer can modify the resource at will

  V(accessMutex);          // Release exclusive access to the resource
}
```

Multiple readers can simultaneously read the resource. Once a writer has entered the waiting list, no new process can begin reading before the writer gets access to the resource.  
The resource is never accessed by a writer without accessMutex being taken in an unconditional and exclusive way. Since we have seen above that accessMutex is always collectively taken by readers before they access the resource, the resource is properly protected. 

Fair access is guaranteed through the orderMutex which is taken upon arrival and released only when access to the resource has been granted
### References

- Operating Systems Concepts (9th Edition), Abraham Silberchatz, Peter B Galvin and Greg Gagne
- Wikipedia
