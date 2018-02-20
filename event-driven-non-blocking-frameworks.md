# Event Driven Non-Blocking Frameworks

We can use single threaded even loop like NodeJS or we use multiple threads to handle concurrent requests. Or, we can use the best from both worlds, non blocking using multiple CPUs while still using even driven approach.

#### Threads

When we use traditional "new thread for each client", we might end up with these issues: 

* Linux kernel reserves 2MB RAM for a thread
* Thread-safety can complicate the codebase
* I/O can be blocking thread
* It takes time to switch between worker threads
* There are limits how many threads can be created per process

#### Non-Blocking I/O

Linux kernel allows us to set a flag that tells the system to return I/O operations immediately even when the data is not ready. 

#### Event I/O

Linux kernel gives us an option to use epoll that allows us to monitor multiple file descriptors and see if they are ready for I/O.

Combining non-Blocking and Event I/O gives us nice benefits: 

* lets kernel manage context switching and I/O readiness
* epoll handles large set of file descriptors, which means more connections can be opened 
* epoll monitoring costs only 160 bytes on 64bits systems





