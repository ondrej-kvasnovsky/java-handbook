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

If we provide weak implemention of `hashCode` method, we might degrade peformance of hash map. Lets try to create many instances of `Test` class and insert them into `HashMap`.





