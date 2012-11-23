---
layout: post
title: "Fully fledged Spring support in Play 2.0"
date: 2012-11-23 15:05
comments: true
categories: [play, spring]
---

I think one of the main disadvantages of Play's design is the overuse of static methods in controllers. This causes hard to test controllers, long-running tests and decreases modularisation. So, how can we use Spring to rescue us? <!-- more -->

First of all, don't get me wrong: I like the Play Framework a lot. I just think there are chances for optimisation if it comes to testing and modularisation.

After searching for Spring integration in Play Framework 2 I discovered this [demo](https://github.com/guillaumebort/play20-spring-demo) by Guillaume Bort on how to combine Spring and Play 2. The name of this repository suggests that is does work with Play 2.0 but it only works with Play 2.1 because the so called *managed controller classes instantiation* is only available on Play 2.1 branch. And in fact with Play 2.1 you can easily integrate every dependency injection framework you want by implementing a ```getControllerInstance``` method on a Global object.

I want to show you how to integrate Spring into your Play 2.0.x application.

There is already a Spring module for Play 2. But I don't like the fact that you have to use at least two lines of XML to get annotation-based configuration and component-scanning in place. As Chris Beams states in his slides about modern enterpise app config: ["this is kind of ironic"](http://cbeams.github.com/modern-config/#89). If you like XML files you can also try this module: [play-2.0-spring-module](https://github.com/wsargent/play-2.0-spring-module). I prefer annotation-based configuration. But this module inspired my solution a lot.

Ok, the goal is to autowire our controllers. Therefore, you have to intercept the controller creation by Play in some decent way. The trick is to add a stupid ```ControllerFactory``` that delegates static controller calls to non-static controller classes and let all routes go through this factory.

```java ControllerFactory to wrap non-static controllers in static method calls
public class ControllerFactory {

    public static Application application() {
        return new Application();
    }
}
```

```scala Corresponding routes file for this example
GET     /                           controllers.ControllerFactory.application.index()
GET     /personalized/:name         controllers.ControllerFactory.application.helloTo(name: String)
```

The controllers can be non-static and therefore also instantiated by an application context. In the next step you simply bootstrap an application context in your Global object.

```java Application context bootstrap during start of Play application
public class Global extends GlobalSettings {

    private static ApplicationContext applicationContext;

    @Override
    public void onStart(Application application) {
        applicationContext = new AnnotationConfigApplicationContext(SpringConfiguration.class);
    }

    public static <T> T getBean(Class<T> beanClass) {
        if (applicationContext == null) {
            throw new IllegalStateException("application context is not initialized");
        }
        return applicationContext.getBean(beanClass);
    }
}
```

With this you get an application context everytime Play starts. In the last step you autowire every controller in the static delegating methods of ```ControllerFactory```.

```java ControllerFactory with autowiring controllers
public class ControllerFactory {

    public static Application application() {
        // magic happens: Spring autowires your controller :-)
        return Global.getBean(Application.class);
    }
}
```

Now you have fully fledged Spring integration in your Play application and the ```Application``` controller can be autowired.

```java Example of autowired Application controller
@Component
public class Application extends Controller {

    @Autowired
    private HelloWorldService helloWorldService;

    @Autowired
    private PersonalizedHelloWorldService personalizedHelloWorldService;

    public Result index() {
        return ok(helloWorldService.sayHello());
    }

    public Result helloTo(String name) {
        return ok(personalizedHelloWorldService.sayHelloTo(name));
    }
}
```

You can find the full example project with more explanations there: [play20-spring-integration](https://github.com/fmueller/play20-spring-integration)