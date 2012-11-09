---
layout: post
title: "Integration Tests with MongoDB and Play Framework 2"
date: 2012-11-09 09:45
comments: true
categories: [testing, play, mongodb]
---

In of my projects I am building a rest api with the playframework 2 (using the java api) and mongodb as the persistence storage. One of the first question that arises was: how can I implement integration tests properly with a test instance of mongodb? I wanted to have a known state in mongodb for each test and it should be easy to set up the test instance. <!-- more -->

I looked for projects on github that supports my requirements and came up with these three possibilities:

* [**jmockmongo**](https://github.com/thiloplanz/jmockmongo) This is a mock implementation of the mongodb protocol and works purely in-memory.
* [**nosqlunit**](https://github.com/lordofthejars/nosql-unit) This library offers more features than jmockmongo. You can use it to test several nosql stores and configure data sets for your tests. For testing mongodb it uses jmockmongo under the hood. You can also use a local or remote running mongodb instance for your tests if the in-memory approach does not suit your needs.
* [**embedmongo**](https://github.com/flapdoodle-oss/embedmongo.flapdoodle.de) With embedmongo you get a platform independant way of running local mongodb instances. It provides helper classes to bootstrap mongodb during tests. The most important feature imho: You do not need to install mongodb. embedmongo downloads mongodb automatically for windows, linux and mac when bootstraping your tests.

Because I wanted a running mongodb instance and no in-memory solution I finally chose embedmongo. There is a nice helper class which lets you easily setup a mongodb on a free port of your system. You can also choose the version.

``` java Bootstrap local mongodb instance with embedmongo
new MongodForTestsFactory().with(Version.V2_2_0).newMongo();
```

Furthermore I had the needs to integrate the mongodb test instance into the lifecycle of Play. In Play one typically uses a fake instance for proper integration tests.

``` java Functional test in Play 2 with a fake application
running(fakeApplication(), new Runnable() {
    @Override
    public void run() {
        // here you insert your test code
    }
});
```

One can intercept the bootstrap of a Play 2 application with a so called Global object that is placed in the default package (or another package that you have configured in the application.conf). There you can override methods that are called from Play during start and shutdown of your application. The perfect place to bootstrap your own mongodb instance.

``` java Automatically bootstrap of local mongodb instance in Play 2
import com.mongodb.Mongo;
import de.flapdoodle.embed.mongo.distribution.Version;
import de.flapdoodle.embed.mongo.tests.MongodForTestsFactory;
import play.Application;
import play.GlobalSettings;
import play.Logger;

import java.io.IOException;

public class Global extends GlobalSettings {

    private MongodForTestsFactory mongodForTestsFactory;

    @Override
    public void onStart(Application application) {
        if (application.isProd()) {
            /*
             * here you can setup mongodb for production,
             * e.g. connect to remote mongodb instance
             */
        } else {
            startLocalMongoInstance();
        }
    }

    @Override
    public void onStop(Application application) {
        if (mongodForTestsFactory != null) {
            Logger.info("Shutdown local mongo instance...");
            mongodForTestsFactory.shutdown();
            Logger.info("Local mongo instance was stopped");
        }
    }

    private void startLocalMongoInstance() {
        Logger.info("Start local mongo instance...");
        try {
            mongodForTestsFactory = new MongodForTestsFactory();
            Mongo mongo = mongodForTestsFactory.with(Version.V2_2_0).newMongo();
            // that's it: now you've got the local mongo instance
            Logger.info("Local mongo instance was successfully started");
        } catch (IOException e) {
            Logger.error("Unable to start local mongo instance", e);
            throw new RuntimeException(e);
        }
    }
}
```
This solution starts a mongodb instance each time you start Play in development mode and for each test where you start a fake application. In production you typically connect to a remote mongodb instance or cluster.