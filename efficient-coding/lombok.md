# Lombok

[Lombok project](https://projectlombok.org) is a library that adds many missing features into Java language. Lets have a look what we can use to speed up our development and make Java code more concise, error prone and readable.

Before we start, lets make sure we can build our project and we can work with lombok in our IDE \([IntelliJ IDEA](https://www.jetbrains.com/idea/)\).

### Setup IDE

We need to do two things to get lombok working in our IDE.

We have to install plugin before we can work with lombok. If we don't install the plugin, we would get compile error from IDE but project would work if ran from the build tool.  
![](/assets/Screen Shot 2018-03-13 at 6.23.37 PM.png)When we have installed the plugin and restarted IDE, we can make sure the plugin is properly configured. See the following configuration that will make lombok working in IDEA.![](/assets/Screen Shot 2018-03-13 at 6.23.00 PM.png)The second thing that needs to be done is to enable annotation compilation in IDEA.

![](/assets/Screen Shot 2018-03-13 at 6.22.32 PM.png)

> Instead of manual IDE configuration, we can use `net.ltgt.apt-idea` or `net.ltgt.apt-eclipse` in Gradle to automatically configure what we did above.

### Setup Project

We are going to use Gradle code and build management tool. Here is everything what is needed to get lombok project working.

```
plugins {
    id 'net.ltgt.apt' version '0.10'
}

group 'lombok-demo'
version '1.0-SNAPSHOT'

apply plugin: 'java'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compileOnly 'org.projectlombok:lombok:1.16.20'
    apt "org.projectlombok:lombok:1.16.20"

    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

There are no special tasks to work with lombok. The plugin [net.ltgt.apt](https://plugins.gradle.org/plugin/net.ltgt.apt) will do the Java annotation processing which we have enabled in IDEA in previous chapter.

### Domain Objects

When making a domain objects or DTO \(Data Transformation Objects\) in pure Java, it is all about description of data using classes, private fields, getter and setter methods. But what we want to do is something like this:

```
public class User {
    private final Long id;
    private String email;
}
```

We just want to say type name, what fields it contains and what are the types of the fields. Unfortunattely we need to provide getters and setters in pure Java. But lombok gives us annotations that will enhance the bytecode and provide us with getters and setters for non final fields. Also there is an annotation to generate the constructor with all the fields.

```
import lombok.*;

@AllArgsConstructor
@Getter
@Setter
public class User {
    private final Long id;
    private String email;
}
```

When we use these annotations we can write code like this:

```
User user = new User(1L, "john@john.com");
Long id = user.getId();
user.setEmail("jimmy@jimmy.com");
```

But trying to set id would give us compilation error.

```
user.setId(2L);
```

> We can use [@Data](https://projectlombok.org/features/Data) annotation for domain objects that will apply many annotations in once \(all these @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor\).

### Equals and Hash Code

In pure Java, we need to provide proper implementations of `equals` and `hashCode` methods, otherwise we could face to performance and functional [issues](https://stackoverflow.com/questions/2265503/why-do-i-need-to-override-the-equals-and-hashcode-methods-in-java).

We can annotate class with `@EqualsAndHashCode` annotation to provide generated `equals` and `hashCode` methods from all field in the class.

```
import lombok.*;

 @EqualsAndHashCode
@Getter @Setter
public class User {
    private final Long id;
    private String email;
}
```

We can try out functionality of equals and hashCode in the following example. We create list with duplicated items. Then we create a set from this list and we should get rid of duplicates. Set will use hashCode and equals methods to identify if the items we are trying to put into set are the same. If some items are the same, set will make sure only one is present.

```
public Set<User> uniqueUsers(List<User> users) {
    return new HashSet<>(users);
}

List<User> duplicatedUsers = new ArrayList<>();
duplicatedUsers.add(new User(1L, "john@john.com"));
duplicatedUsers.add(new User(1L, "john@john.com"));
duplicatedUsers.add(new User(2L, "jimmy@jimmy.com"));
duplicatedUsers.add(new User(2L, "jimmy@jimmy.com"));

Set<User> uniqueUsers = userService.uniqueUsers(duplicatedUsers);

assertEquals(uniqueUsers.size(), 2);
```

### ToString annotation

Many times we want to represent object in other than default way, something like `com.company.User@662c0d1b`. Rather we would like to see a string that really represents the object, but not its type and physical address.

Lets add `@ToString` annotation to our `User` class and tell lombok to generate `toString` method that will include `id` of an object instead.

```
User jimmy = new User(1L, "jimmy@jimmy.com");

String toString = jimmy.toString();

assertEquals("User(id=1)", toString);
```

### What is next?

You can explore what all is provided by lombok library on their [features](https://projectlombok.org/features/all) list.

If you want to check out the code that was used in this article, see the [github/lombok-demo](https://github.com/ondrej-kvasnovsky/lombok-demo) repository.

> This article is source of https://medium.com/@OndrejKvasnovsky/java-without-setters-getters-and-other-obstacles-cca0fd2b9b4d



