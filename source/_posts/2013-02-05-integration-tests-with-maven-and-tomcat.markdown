---
layout: post
title: "Integration Tests with Maven and Tomcat"
date: 2013-02-05 17:44
comments: true
categories: [build scripts, maven, snippet]
---

Last weeks I had to deal with implementing several rest apis. Some with Play 2, some with Spring MVC. Due only Play supports build-in mechanism for integration tests I had to integrate support for them in every project where I used Spring MVC. Those are typically Maven builds. Maven does not support separate integration tests with a running Tomcat (or any other webcontainer) by default. Hence you have to get your hands dirty.<!-- more -->

The goal is to execute all JUnit tests that are suffixed with ``IntegrationTest``, start a Tomcat before (I want to test a deployed rest api) and afterwards shutdown the running Tomcat.

For that you have to exclude all integration tests in the normal test phase of Maven.

``` xml First, disable all integration tests during normale test phase
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.12.1</version>
  <configuration>
    <excludes>
      <exclude>**/*IntegrationTest*</exclude>
    </excludes>
  </configuration>
</plugin>
```

Now you should add all integration tests to the Maven Failsafe Plugin and activate the execution of the plugin in your build.

``` xml Configure the Failsafe Plugin appropriately
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>2.12.4</version>
  <configuration>
    <includes>
      <include>**/*IntegrationTest*</include>
    </includes>
  </configuration>
  <executions>
    <execution>
      <goals>
        <goal>integration-test</goal>
        <goal>verify</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

After this the only missing step is a working Tomcat integration in your build. In this example I'm using the Tomcat 7 Maven Plugin for this (the Tomcat 6 plugin should also work). With the ``start`` and ``stop`` goals you can control a Tomcat instance. Important is that you fork the process, otherwise your build would be blocked by the running Tomcat. You can do similiar things with the Jetty Maven Plugin.

``` xml Start Tomcat before all integration tests and stop it afterwards
<plugin>
  <groupId>org.apache.tomcat.maven</groupId>
  <artifactId>tomcat7-maven-plugin</artifactId>
  <version>2.0</version>
  <configuration>
    <path>/</path>
  </configuration>
  <executions>
    <execution>
      <id>start-tomcat</id>
      <phase>pre-integration-test</phase>
      <goals>
        <goal>run</goal>
      </goals>
      <configuration>
        <fork>true</fork>
      </configuration>
    </execution>
    <execution>
      <id>stop-tomcat</id>
      <phase>post-integration-test</phase>
      <goals>
        <goal>shutdown</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

With this little setup you get full support for integration tests with Maven and Tomcat in your web applications, especially if you building rest apis. I hope that this Maven snippet helps you to improve your builds.

**Update [11.02.2013]** Changed suffix of include and exclude configurations. Thanks to a hint by Erich Eichinger in the comment section.