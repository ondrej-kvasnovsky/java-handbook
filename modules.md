# Modules

We used to have only accessibility modifiers on class level in Java. Module is way to encapsulate classes in our code. That means, we can control what is accessible from our library.

Modules helps us to avoid big ball of mud, aka big ball of classes loaded from all the jars. Which means, we could optimize number of classes that are loaded into memory. That should result in smaller memory foot print.

Module system is not mandatory. We can create apps or libraries without module system. But if we decide to use modules, so we create a module-info.java file, we need to say what other modules should be included.

Here is nice explanation how it is with [Java 9 modules and Gradle](https://blog.gradle.org/java-9-support-update).

### Create Module

First lets create a module.

```
package java9.module.example.impl;

import java.util.logging.Logger;

public class Something {

    private static final Logger logger = Logger.getLogger(Something.class.toString());

    public static void main(String[] args) {
        logger.warning("Hello World");
    }
}
```

Then we create module-info.java. We need to insert requires java.logging because we want to use Logger from java.utils.logging package. There can be only one module-info.java file per module and it should be located in root source folder.

> java.base module is included automatically

```
module java9.module.example {
    requires java.logging;
}
```

If we want to let other modules to access our classes, we need to export those classes from our module.

```
module java9.module.example {
    requires java.logging;
    exports java9.module.example;
}
```

When an external module needs to access our classes using reflection, we need to export it to that module.

```
module java9.module.example {
    exports java9.module.example to javafx.graphics;
}
```

Or we can open our code to other libraries.

```
module java9.module.example {
    open java9.module.example to javafx.graphics;
}
```

### JDeps

[jdeps](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jdeps.html) can be used to analyze dependencies in our modules and build our module-info.java files.

```
$JAVA_HOME/bin/jdeps --class-path <class path> <module name>
```

### JLINK

[jlink](https://docs.oracle.com/javase/9/tools/jlink.htm#JSWOR-GUID-CECAC52B-CFEE-46CB-8166-F17A8E9280E9) can be used to assemble a runtime image.

```
$JAVA_HOME/bin/jlink --module-path $JAVA_HOME/jmods:mlib:/Users/ondrej/Document/Projects/java/java-9/out/production/java9 --add-modules java9.module1 --output image
```



