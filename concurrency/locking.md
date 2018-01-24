# Locking

When we work with multiple threads, we might need to make sure that a variable or a block of code is used only by one thread at a time.

### volatile

We can use `volatile` on variable level, like this `volatile int counter;`. When we use `volatile` keyword, we say Java to keep that variable in main memory. Better explained, because each thread can have its own copy of a variable, `volatile` keeps threads variable always synchronized with the value of the variable in the main memory.

> When we expect to use only basic operations, like set or get value, we can us `volatile`. But if we want to have an atomic operations like setAndGet or incrementAndGet, we need to use `*Atomic` class from Java.

### synchronized

We can use `synchronized` keyword to lock block of code, so it can be accessed only by one thread at the time. It makes sure that only one thread holds a lock and thus can access critical section. Before the synchronized section is accessed, thread variables are synchronized with values from main memory, to ensure data consistency.

Lets see how we could use synchronized to make sure read and write operations are not called by multiple threads at the same time. 

```
import java.util.HashMap;
import java.util.Map;

public class SyncDb<KEY, VALUE> {

    private Map<KEY, VALUE> values = new HashMap<>();

    public VALUE insert(KEY key, VALUE value) {
        System.out.println(Thread.currentThread().getName() + " wants to write.");
        synchronized (this) {
            System.out.println(Thread.currentThread().getName() + " Insert: " + key);
            return values.put(key, value);
        }
    }

    public synchronized VALUE find(KEY key) {
        System.out.println(Thread.currentThread().getName() + " wants to read.");
        synchronized (this) {
            System.out.println(Thread.currentThread().getName() + " Find: " + key);
            return values.get(key);
        }
    }
}
```

Here is a test we could run to see what is happening. 

```
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class SyncDbTest {

    private SyncDb<Integer, String> db;

    @Before
    public void setUp() {
        db = new SyncDb<>();
    }

    @Test
    public void insert() throws InterruptedException {
        List<Callable<Object>> tasks = new ArrayList<>();
        tasks.add(() -> db.insert(1, "Value 1"));
        tasks.add(() -> db.insert(2, "Value 2"));
        tasks.add(() -> db.insert(3, "Value 3"));
        tasks.add(() -> db.insert(4, "Value 4"));
        tasks.add(() -> db.find(1));
        tasks.add(() -> db.find(2));
        tasks.add(() -> db.find(3));
        tasks.add(() -> db.find(4));
        tasks.add(() -> db.find(5));
        tasks.add(() -> db.find(6));
        tasks.add(() -> db.find(7));
        tasks.add(() -> db.find(8));
        tasks.add(() -> db.find(9));
        tasks.add(() -> db.find(10));
        tasks.add(() -> db.find(12));
        tasks.add(() -> db.find(13));
        tasks.add(() -> db.find(14));

        Executors.newFixedThreadPool(20).invokeAll(tasks, 10, TimeUnit.SECONDS);
    }
}
```

Here we can see that this kind of locking does not perform very well. 

```
pool-1-thread-1 wants to write.
pool-1-thread-2 wants to write.
pool-1-thread-1 Insert: 1
pool-1-thread-3 wants to write.
pool-1-thread-2 Insert: 2
pool-1-thread-3 Insert: 3
pool-1-thread-4 wants to write.
pool-1-thread-4 Insert: 4
pool-1-thread-5 wants to read.
pool-1-thread-5 Find: 1
pool-1-thread-6 wants to read.
pool-1-thread-6 Find: 2
pool-1-thread-7 wants to read.
pool-1-thread-7 Find: 3
pool-1-thread-8 wants to read.
pool-1-thread-8 Find: 4
pool-1-thread-9 wants to read.
pool-1-thread-9 Find: 5
pool-1-thread-10 wants to read.
pool-1-thread-10 Find: 6
pool-1-thread-11 wants to read.
pool-1-thread-11 Find: 7
pool-1-thread-12 wants to read.
pool-1-thread-12 Find: 8
pool-1-thread-13 wants to read.
pool-1-thread-13 Find: 9
pool-1-thread-14 wants to read.
pool-1-thread-14 Find: 10
pool-1-thread-15 wants to read.
pool-1-thread-15 Find: 12
pool-1-thread-16 wants to read.
pool-1-thread-16 Find: 13
pool-1-thread-17 wants to read.
pool-1-thread-17 Find: 14
```

### ReadWriteLock

Allows multiple thread to read a resource but only one thread to write. When a thread wants to write, the lock marks that there is a thread requesting write. Then the lock prioritizes write. Because it could happen that there are too many read threads and write never occurs, starvation.

What is the [difference between read and write](https://stackoverflow.com/questions/18354339/reentrantreadwritelock-whats-the-difference-between-readlock-and-writelock) locks? 

`readLock.lock();`

This means that if any other thread is writing \(i.e. holds a write lock\) then stop here until no other thread is writing. Once the lock is granted no other thread will be allowed to write \(i.e. take a write lock\) until the lock is released.

`writeLock.lock();`

This means that if any other thread is reading or writing, stop here and wait until no other thread is reading or writing. Once the lock is granted, no other thread will be allowed to read or write \(i.e. take a read or write lock\) until the lock is released.

Combining these you can arrange for only one thread at a time to have write access but as many readers as you like can read at the same time except when a thread is writing.

Put another way. Every time you want to read from the structure, take a read lock. Every time you want to write, take a write lock. This way whenever a write happens no-one is reading \(you can imagine you have exclusive access\), but there can be many readers reading at the same time so long as no-one is writing.

Lets create a class that will pretend to be a storage of values. It uses both read and write locks.

> Reentrant because one thread can lock it multiple times.

```
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ConsistentDb<KEY, VALUE> {

    private ReadWriteLock lock = new ReentrantReadWriteLock();
    private Lock readLock = lock.readLock();
    private Lock writeLock = lock.writeLock();

    private Map<KEY, VALUE> values = new HashMap<>();

    public VALUE insert(KEY key, VALUE value) {
        System.out.println(Thread.currentThread().getName() + " wants to write.");
        writeLock.lock();
        try {
            Thread.sleep(100);
            System.out.println(Thread.currentThread().getName() + " Insert: " + key);
            return values.put(key, value);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            writeLock.unlock();
        }
        return null;
    }

    public VALUE find(KEY key) {
        System.out.println(Thread.currentThread().getName() + " wants to read.");
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + " Find: " + key);
            return values.get(key);
        } finally {
            readLock.unlock();
        }
    }
}
```

Now we can try to "test" the behavior. We are going to create a lot of read operations on only couple of writes. Writes are more expensive than reads.

```
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class ConsistentDbTest {

    private ConsistentDb<Integer, String> db;

    @Before
    public void setUp() {
        db = new ConsistentDb<>();
    }

    @Test
    public void insert() throws InterruptedException {
        List<Callable<Object>> tasks = new ArrayList<>();
        tasks.add(() -> db.insert(1, "Value 1"));
        tasks.add(() -> db.insert(2, "Value 2"));
        tasks.add(() -> db.insert(3, "Value 3"));
        tasks.add(() -> db.insert(4, "Value 4"));
        tasks.add(() -> db.find(1));
        tasks.add(() -> db.find(2));
        tasks.add(() -> db.find(3));
        tasks.add(() -> db.find(4));
        tasks.add(() -> db.find(5));
        tasks.add(() -> db.find(6));
        tasks.add(() -> db.find(7));
        tasks.add(() -> db.find(8));
        tasks.add(() -> db.find(9));
        tasks.add(() -> db.find(10));
        tasks.add(() -> db.find(12));
        tasks.add(() -> db.find(13));
        tasks.add(() -> db.find(14));

        Executors.newFixedThreadPool(20).invokeAll(tasks, 10, TimeUnit.SECONDS);
    }
}
```

When we run the code above, we can see that  all the threads came to the lock and requested access. Then write lock allowed all inserts to pass and then read went on.

```
pool-1-thread-1 wants to write.
pool-1-thread-2 wants to write.
pool-1-thread-3 wants to write.
pool-1-thread-4 wants to write.
pool-1-thread-5 wants to read.
pool-1-thread-6 wants to read.
pool-1-thread-7 wants to read.
pool-1-thread-8 wants to read.
pool-1-thread-9 wants to read.
pool-1-thread-10 wants to read.
pool-1-thread-11 wants to read.
pool-1-thread-12 wants to read.
pool-1-thread-13 wants to read.
pool-1-thread-14 wants to read.
pool-1-thread-15 wants to read.
pool-1-thread-16 wants to read.
pool-1-thread-17 wants to read.
pool-1-thread-1 Insert: 1
pool-1-thread-2 Insert: 2
pool-1-thread-3 Insert: 3
pool-1-thread-4 Insert: 4
pool-1-thread-5 Find: 1
pool-1-thread-6 Find: 2
pool-1-thread-7 Find: 3
pool-1-thread-8 Find: 4
pool-1-thread-9 Find: 5
pool-1-thread-10 Find: 6
pool-1-thread-12 Find: 8
pool-1-thread-11 Find: 7
pool-1-thread-14 Find: 10
pool-1-thread-13 Find: 9
pool-1-thread-16 Find: 13
pool-1-thread-17 Find: 14
pool-1-thread-15 Find: 12
```

Now we can try change reentrant lock to be fair.

```
private ReadWriteLock lock = new ReentrantReadWriteLock(true);
```

Observe that the now, when the order of execution if based on when the lock was requested.

```
pool-1-thread-1 wants to write.
pool-1-thread-2 wants to write.
pool-1-thread-3 wants to write.
pool-1-thread-4 wants to write.
pool-1-thread-5 wants to read.
pool-1-thread-6 wants to read.
pool-1-thread-7 wants to read.
pool-1-thread-8 wants to read.
pool-1-thread-9 wants to read.
pool-1-thread-10 wants to read.
pool-1-thread-11 wants to read.
pool-1-thread-12 wants to read.
pool-1-thread-13 wants to read.
pool-1-thread-14 wants to read.
pool-1-thread-15 wants to read.
pool-1-thread-16 wants to read.
pool-1-thread-17 wants to read.
pool-1-thread-1 Insert: 1
pool-1-thread-2 Insert: 2
pool-1-thread-3 Insert: 3
pool-1-thread-4 Insert: 4
pool-1-thread-5 Find: 1
pool-1-thread-6 Find: 2
pool-1-thread-7 Find: 3
pool-1-thread-9 Find: 5
pool-1-thread-10 Find: 6
pool-1-thread-11 Find: 7
pool-1-thread-12 Find: 8
pool-1-thread-8 Find: 4
pool-1-thread-13 Find: 9
pool-1-thread-15 Find: 12
pool-1-thread-14 Find: 10
pool-1-thread-16 Find: 13
pool-1-thread-17 Find: 14
```

### StampedLock

Reentrant lock has some issues with stravation if not used properly. For example, if there are 10000 reads and only 2 writes, the two writes might come into starvation because the locking will be busy with all the read operations.

StampedLock is all about giving us a possibility to perform optimistic reads.

> StampedLock is not reentrant, so each call to acquire the lock always returns a new stamp and blocks if there's no lock available, even if the same thread already holds a lock, which may lead to deadlock.

```
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.StampedLock;

public class OptimisticDb<KEY, VALUE> {

    private StampedLock lock = new StampedLock();

    private Map<KEY, VALUE> values = new HashMap<>();

    public VALUE insert(KEY key, VALUE value) {
        System.out.println(Thread.currentThread().getName() + " wants to write.");
        long stamp = this.lock.writeLock();
        try {
            System.out.println(Thread.currentThread().getName() + " Insert: " + key);
            return values.put(key, value);
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    public VALUE find(KEY key) {
        System.out.println(Thread.currentThread().getName() + " wants to read.");
        long stamp = lock.tryOptimisticRead();
        if (!lock.validate(stamp)) {
            System.out.println(Thread.currentThread().getName() + " Failed to obtain optimistic lock");
            stamp = lock.readLock();
        }
        try {
            System.out.println(Thread.currentThread().getName() + " Find: " + key);
            return values.get(key);
        } finally {
            lock.unlockRead(stamp);
        }
    }
}
```

Here is the test.

```
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

import static org.junit.Assert.*;

public class OptimisticDbTest {

    private OptimisticDb<Integer, String> db;

    @Before
    public void setUp() {
        db = new OptimisticDb<>();
    }

    @Test
    public void insert() throws InterruptedException {
        List<Callable<Object>> tasks = new ArrayList<>();
        tasks.add(() -> db.insert(1, "Value 1"));
        tasks.add(() -> db.insert(2, "Value 2"));
        tasks.add(() -> db.insert(3, "Value 3"));
        tasks.add(() -> db.insert(4, "Value 4"));
        tasks.add(() -> db.find(1));
        tasks.add(() -> db.find(2));
        tasks.add(() -> db.find(3));
        tasks.add(() -> db.find(4));
        tasks.add(() -> db.find(5));
        tasks.add(() -> db.find(6));
        tasks.add(() -> db.find(7));
        tasks.add(() -> db.find(8));
        tasks.add(() -> db.find(9));
        tasks.add(() -> db.find(10));
        tasks.add(() -> db.find(12));
        tasks.add(() -> db.find(13));
        tasks.add(() -> db.find(14));

        Executors.newFixedThreadPool(20).invokeAll(tasks, 10, TimeUnit.SECONDS);
    }
}
```

Observe how locks were acquired and when it failed to acquire optimistic lock.

```
pool-1-thread-1 wants to write.
pool-1-thread-2 wants to write.
pool-1-thread-3 wants to write.
pool-1-thread-1 Insert: 1
pool-1-thread-4 wants to write.
pool-1-thread-4 Insert: 4
pool-1-thread-5 wants to read.
pool-1-thread-5 Find: 1
pool-1-thread-6 wants to read.
pool-1-thread-6 Find: 2
pool-1-thread-7 wants to read.
pool-1-thread-7 Find: 3
pool-1-thread-8 wants to read.
pool-1-thread-8 Find: 4
pool-1-thread-9 wants to read.
pool-1-thread-2 Insert: 2
pool-1-thread-10 wants to read.
pool-1-thread-10 Failed to obtain optimistic lock
pool-1-thread-9 Failed to obtain optimistic lock
pool-1-thread-3 Insert: 3
pool-1-thread-10 Find: 6
pool-1-thread-11 wants to read.
pool-1-thread-9 Find: 5
pool-1-thread-11 Find: 7
pool-1-thread-12 wants to read.
pool-1-thread-12 Find: 8
pool-1-thread-13 wants to read.
pool-1-thread-13 Find: 9
pool-1-thread-14 wants to read.
pool-1-thread-14 Find: 10
pool-1-thread-15 wants to read.
pool-1-thread-15 Find: 12
pool-1-thread-16 wants to read.
pool-1-thread-16 Find: 13
pool-1-thread-17 wants to read.
pool-1-thread-17 Find: 14
```

> More about [stamped lock](https://dzone.com/articles/a-look-at-stampedlock?fromrel=true)



