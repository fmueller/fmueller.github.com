---
layout: post
title: "Fast and reliable integration tests with Play 2"
date: 2012-12-21 11:49
comments: true
categories: [play, testing, snippet]
---

When you start using the Play Framework 2 you will notice that there are a bunch of test helpers and entry points to set up your tests. That's great! One might thinks that's too good to be true. And in fact, when using the ```FakeApplication``` test helper you can run into subtle problems which cause unreliable tests. <!-- more -->

In the [Play Framework documentation](http://www.playframework.org/documentation/2.0/JavaTest) they show you that it is quite easy to set up a test with a ```FakeApplication``` by the following code snippet:

```java Example of FakeApplication usage in Play Framework documentation
@Test
public void findById() {
  running(fakeApplication(), new Runnable() {
    public void run() {
      Computer macintosh = Computer.find.byId(21l);
      assertThat(macintosh.name).isEqualTo("Macintosh");
      assertThat(formatted(macintosh.introduced)).isEqualTo("1984-01-24");
    }
  });
}
```

So, you might think when you have many test methods you just copy this pattern and everything is fine. But that is not the case!

```java Pattern of FakeApplication per test method
@Test
public void testOne() {
  running(fakeApplication(), new Runnable() {
    public void run() {
      // test logic
    }
  });
}

@Test
public void testTwo() {
  running(fakeApplication(), new Runnable() {
    public void run() {
      // test logic
    }
  });
}
```

Here are two examples that ```FakeApplication``` can be difficult to use:

  * [(Play-Scala 2.0.4) FakeApplication onStop not being called after Test](https://groups.google.com/forum/#!searchin/play-framework/fakeApplication/play-framework/zcpdOL61xXQ/5cUH-HKf1okJ)
  * [[play 2.0.3] Two fakeApplication tests only one will succeed but the other throws error 500](https://groups.google.com/forum/#!msg/play-framework/0Ld889ARC8o/mYcrMeX2dsYJ)

I ran into the same problems that are mentioned in the first mailinglist posting. In the [project where I'm using MongoDB together with Play 2](http://cupofjava.de/blog/2012/11/09/integration-tests-with-mongodb-and-play-framework-2/) the tests with a ```FakeApplication``` started to become unreliable. There were issues with cleaning up test data after and before each test method. Also the shutdown and restart of a ```FakeApplication``` was sometimes error prone.

The best solution for this problems is to start one ```FakeApplication``` per test class, not per test method. Here is the test helper class that I use for this:

```java One FakeApplication per test class
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.BeforeClass;
import play.test.FakeApplication;

import static play.test.Helpers.*;

public abstract class AbstractOneFakeApplicationIntegrationTest {

    private static FakeApplication fakeApplication;

    @BeforeClass
    public static void startFakeApplication() {
        fakeApplication = fakeApplication();
        start(fakeApplication);
    }

    @AfterClass
    public static void shutdownFakeApplication() {
        stop(fakeApplication);
    }
}

```

You only need to inherit from it and every test has access to a running ```FakeApplication```. One disadvantage is that you have more work to clean up any data that is written for your tests into databases, filesystem or any other persistent system. But you get **reliable integration tests** and the tests are **much faster** because you only start one ```FakeApplication``` per class and not dozens of them as in the per method approach

Beside this little issues that ```FakeApplication``` can cause Play 2 offers great testing support.