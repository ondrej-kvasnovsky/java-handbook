# Locking

When we work with multiple threads, we might need to make sure that a variable or a block of code is used only by one thread at a time.

### volatile

We can use `volatile` on variable level, like this `volatile int counter;`. When we use `volatile` keyword, we say Java to keep that variable in main memory. Better explained, because each thread can have its own copy of a variable, `volatile` keeps threads variable always synchronized with the value of the variable in the main memory.

> When we expect to use only basic operations, like set or get value, we can us `volatile`. But if we want to have an atomic operations like setAndGet or incrementAndGet, we need to use `*Atomic` class from Java.

### synchronized

We can use `synchronized` keyword to lock block of code, so it can be accessed only by one thread at the time. It makes sure that only one thread holds a lock and thus can access critical section. Before the synchronized section is accessed, thread variables are synchronized with values from main memory, to ensure data consistency.

### ReadWriteLock

### StampedLock



