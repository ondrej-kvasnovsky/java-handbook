# Other Improvements

### Base64

We no longer have to use `sun.misc.BASE64Encoder` or `org.apache.commons.codec.binary.Base64`. No we can use [java.util.Base64](https://docs.oracle.com/javase/9/docs/api/java/util/Base64.html) class. This is because the [internal APIs](https://www.javaworld.com/article/2952869/java-platform/understanding-sun-misc-unsafe.html) were encapsulated and are no longer available.

