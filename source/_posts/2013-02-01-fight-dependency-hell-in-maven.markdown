---
layout: post
title: "Fight dependency hell in Maven"
date: 2013-02-01 18:29
comments: true
categories: [build scripts, maven, snippet]
---

Did you ever use Maven in one of your projects? First you probably were amazed by the way you can declare dependencies and use all the nice plugins. Then you may realized that the handling of transitive dependencies is solved… well, let me say it could be better. But there is another Maven plugin that helps you to clean your dependencies and also to ensure that transitive dependencies will never cause to rot your dependency definitions again. <!-- more -->

As mentioned in this [blog posting](http://www.jasonwhaley.com/blog/2012/03/21/dependency-convergence-in-maven/), the [Enforcer Maven Plugin](http://maven.apache.org/enforcer/maven-enforcer-plugin/index.html) helps. It offers a dependency convergence rule that checks the following:
{% blockquote %}
If a project has two dependencies, A and B, both depending on the same artifact, C, this rule will fail the build if A depends on a different version of C then the version of C depended on by B.
{% endblockquote %}
That is exactly what I want in all of my projects managed by Maven! Therefore, I add the enforcer plugin to the pom file and enable the ``DependencyConvergence`` rule. I find it useful when the enforcer plugin is executed during the validate phase. This causes the build to break before compiling if the dependencies do not converge. Then you have to exclude the unwanted transitive dependencies or change the versions. Very handy!

``` xml Enable dependency convergence check in your Maven build
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <version>1.0.1</version>
  <configuration>
    <rules>
      <DependencyConvergence/>
    </rules>
  </configuration>
  <executions>
    <execution>
      <id>enforce</id>
      <goals>
        <goal>enforce</goal>
      </goals>
      <phase>validate</phase>
    </execution>
  </executions>
</plugin>
```
Just try this in one of your projects. You will be quite surprised what for a mess your actual dependency definitions are.