# Concurrency

### Deadlocks

Here is a code that could result in deadlock. Imagine one thread calls `lockAndSave` and another threads call `lockAndLoad`.

```
  public static Object cacheLock = new Object();
  public static Object tableLock = new Object();

  public void lockAndSave() {
    synchronized (cacheLock) {
      synchronized (tableLock) { 
        save();
      }
    }
  }
  public void lockAndLoad() {
    synchronized (tableLock) {
      synchronized (cacheLock) { 
        load();
      }
    }
  }
```

The issue here is that the locks are not in the same order. Lets make the locking in the same order. Could a deadlock occur with this code? No, the code below would never dead lock.

```
  public static Object cacheLock = new Object();
  public static Object tableLock = new Object();

  public void lockAndSave() {
    synchronized (cacheLock) {
      synchronized (tableLock) {
        save();
      }
    }
  }
  public void lockAndLoad() {
    synchronized (cacheLock) {
      synchronized (tableLock) {
        load();
      }
    }
  }
```

Another example of deadlock situation is when two classes that have synchronized methods, call each other.

```
class Model {
    private Object arg;
    private View view;

    public synchronized void updateModel(Object arg) {
        this.arg = arg;
        view.somethingChanged();
    }

    public synchronized Object getSomething() {
        return arg;
    }

    public void setView(View view) {
        this.view = view;
    }
}

class View {
    private Model model;

    public View(Model model) {
        this.model = model;
    }

    public synchronized void somethingChanged() {
        updateView();
    }

    public synchronized void updateView() {
        Object o = model.getSomething();
        System.out.println(o);
    }
}
```

Imagine two threads, at the same time, first calls `updateModel` and second calls `somethingChanged`.

We can try to run this code and verify that we will get a deadlock.

```
public class DeadLockTest {

    public static void main(String[] args) {
        Model model = new Model();
        View view = new View(model);
        model.setView(view);

        IntStream.range(0, 100).forEach(i -> {
            Thread t1 = new Thread(() -> model.updateModel(new Date()));
            Thread t2 = new Thread(() -> view.somethingChanged());
            t1.start();
            t2.start();
        });

    }
}
```

### Hidden deadlocks

We can write a code that will result in deadlock and it won't be that obvious. We could face to deadlock situation even with this code.

```
public void transferMoney(Account fromAccount, Account toAccount, DollarAmount amountToTransfer) { 
   synchronized (fromAccount) {
     synchronized (toAccount) { 
       if (fromAccount.hasSufficientBalance(amountToTransfer) { 
         fromAccount.debit(amountToTransfer); 
         toAccount.credit(amountToTransfer);
       }
     }
   }
}
```

What if, at the same time, one thread calls `transferMoney(accountOne, accountTwo, amount)` and other thread calls `transferMoney(accountTwo, accountOne, amount)`? It will result in dead lock.

### How to avoid dead lock

We should shrink synchronized blocks to a minimum. To make them more readable and avoid confusions. If we can't shrink it, we need to provide extra documentation and explain why we had to do it and what should other people be aware of.

Avoid acquiring multiple locks at a time. If we really have to have multiple locks, we need to make sure they are defined in a consistent order.

We might have to order the objects that are passed as argument, so the locking is always done in the same way.

```
public void transferMoney(Account fromAccount, 
                            Account toAccount, 
                            DollarAmount amountToTransfer) { 
    Account firstLock, secondLock;
    if (fromAccount.accountNumber() == toAccount.accountNumber())
      throw new Exception("Cannot transfer from account to itself");
    else if (fromAccount.accountNumber() < toAccount.accountNumber()) {
      firstLock = fromAccount;
      secondLock = toAccount;
    }
    else {
      firstLock = toAccount;
      secondLock = fromAccount;
    }
    synchronized (firstLock) {
      synchronized (secondLock) { 
        if (fromAccount.hasSufficientBalance(amountToTransfer) { 
          fromAccount.debit(amountToTransfer); 
          toAccount.credit(amountToTransfer);
        }
      }
    }
}
```

> See the original explanation, [this](https://www.javaworld.com/article/2075692/java-concurrency/avoid-synchronization-deadlocks.html?page=2) article about deadlocks.



