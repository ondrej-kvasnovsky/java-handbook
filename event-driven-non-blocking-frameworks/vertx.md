# Vert.x

> Eclipse Vert.x is _event driven \_and \_non blocking_. This means your app can handle a lot of concurrency using a small number of kernel threads. Vert.x lets your app scale with minimal hardware. -- [http://vertx.io](http://vertx.io)

Vert.x is an entire toolkit and ecosystem of pluggable modules on top of Netty for building reactive applications on top of the JVM.

### Hello world

Here are [many use cases](http://vertx.io/docs/) for Vert.x, but lets create hello world app as HTTP server.

Lets create a project in Gradle.

```
plugins {
  id 'java'
  id 'application'
}

ext {
  vertxVersion = '3.5.0'
}

repositories {
  mavenLocal()
  jcenter()
}

version = '1.0.0-SNAPSHOT'
sourceCompatibility = '1.8'

dependencies {
  compile "io.vertx:vertx-core:$vertxVersion"
  compile "io.vertx:vertx-unit:$vertxVersion"
}

mainClassName = 'com.example.demo.App'

task wrapper(type: Wrapper) {
  gradleVersion = '4.0'
}
```

Then we can create runnable class that starts up the server.

```
package com.example.demo;

import io.vertx.core.Vertx;
import io.vertx.core.http.HttpServer;
import io.vertx.core.http.HttpServerOptions;

public class App {

    public static void main(String[] args) {
        Vertx vertx = Vertx.vertx();
        HttpServerOptions options = new HttpServerOptions().setLogActivity(true);

        HttpServer httpServer = vertx.createHttpServer(options);
        httpServer.requestHandler(request -> {
            request.response().end("Hello World");
        });
        httpServer.listen(8888, res -> {
            if (res.succeeded()) {
                System.out.println("Server is now listening!");
            } else {
                System.out.println("Failed to bind!");
            }
        });
    }
}
```

When start the server and execute HTTP request, we get this.

```
$ gradle run
Server is now listening!

$ curl localhost:888
Hello World
```



