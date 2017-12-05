# Hash Code

We are going to dive deep into `hashCode` method in Java.

## Object

`Object` class contains default implementation of `hasCode` method. We can override that method and implement.

```
class Test {
  @Override
  public int hashCode() {
    return 1;
  }
}

Test test = new Test();
System.out.println(test.hashCode());
```

Here is the output.

```
1
```

There is a way to get original hashCode of the object before we overridden it. We can use `indentityHashCode` method from `System` class.

```
int originalHashCode = System.identityHashCode(test);
System.out.println()
```

We can verify that identityHashCode method really prints the original hash code. Just get the hashCode of parent in the hasCode method.

```
class Test {
  @Override
  public int hashCode() {
    System.out.println(super.hashCode())
    return 1;
  }
}
```

If we provide weak implemention of `hashCode` method, we might degrade peformance of hash map. Lets try to create many instances of `Test` class and insert them into `HashMap`. This code takes about 1 second to insert 10 000 items into hash map. All because of wrong implementation of `hashCode`.

```
class Test {
  @Override
  public int hashCode() {
    return 1;
  }
}

long start = new Date().getTime();
Set map = new HashSet();
for (int i = 0; i < 10000; i++) {
  map.add(new Test());
}
System.out.println(new Date().getTime() - start);
// prints out 915
```

Here is the code that leaves implementation of `hashCode` to `Object` class. This one will insert 10 000 items into hash map in about 5 milliseconds.

```
class Test {
}

long start = new Date().getTime();
Set map = new HashSet();
for (int i = 0; i < 10000; i++) {
  map.add(new Test());
}
System.out.println(new Date().getTime() - start);
// prints out 5
```



