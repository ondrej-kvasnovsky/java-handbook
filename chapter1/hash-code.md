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

What should be the proper implementation of `hashCode` for Test? For that class, we shouldn't touch the `hashCode` method. 

In order to override `hashCode`, we need to get better example. Lets create `User` class that will contain couple of fields. We want to  use those fields to calculate `hashCode`. We can `Objects.hash` method since Java 7.

```
import java.util.Objects;

public class User {
  private String name;
  private int age;
  private String passport;

  @Override
  public int hashCode() {
     return Objects.hash(name, age, passport);
  }
}
```

Lets create the same performance test we did with `Test` class.

```
public class User {
  String name;
  int age;
  String passport;

  @Override
  public int hashCode() {
    return Objects.hash(name, age, passport);
  }
}

long start = new Date().getTime();
Set map = new HashSet();
for (int i = 0; i < 10000; i++) {
  User user = new User();
  user.name = String.valueOf(i);
  user.age = i;
  user.passport = String.valueOf(i);
  map.add(user);
}
System.out.println(new Date().getTime() - start);
// prints out 33
```

Now the `hashCode` method is implemented correctly and we do not have issue performance issue when inserting values into hash set. 

Lets try to verify if we are able to find out if the user is present in that hash set. 

```
User user = new User();
user.name = "9999";
user.age = 9999;
user.passport = "9999";

boolean contains = map.contains(user);
System.out.println(contains);
// prints out false
```

It says that the user is not present in the hash set, because we didn't overridden equals method. Lets do it now. 

```
public class User {
  String name;
  int age;
  String passport;

  @Override
  public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof User)) {
       return false;
    }
    User user = (User) o;
    return age == o.age &&
      Objects.equals(name, user.name) &&
      Objects.equals(passport, user.passport);
  }

  @Override
  public int hashCode() {
    return Objects.hash(name, age, passport);
  }
}

Set<User> map = new HashSet<>();
for (int i = 0; i < 10000; i++) {
  User user = new User();
  user.name = String.valueOf(i);
  user.age = i;
  user.passport = String.valueOf(i);
  map.add(user);
}

User user = new User();
user.name = "9999";
user.age = 9999;
user.passport = "9999";

boolean contains = map.contains(user);
System.out.println(contains);
// prints out true
```



