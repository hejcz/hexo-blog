---
title: Using Logback in WildFly 10
date: 2017-02-25 00:22:14
tags:
- wildfly
---

The simplest way to setup logging in WildFly is to use built in logging subsystem. It provides required dependencies and simplifies logging configuration. Unfortunately we wanted to use Logback we had before changing application server.

Application consists of mutliple EAR archives deployed on WildFly. They are all deployed with `slf4j-api`, `logback-core` and `logback-classic` JARs included. Basically I just wanted to define a set of dependencies available to every archive on server.

I created a new jboss module in `$WILDFLY_HOME/modules/io/github/hejcz/...` containing those JAR files. I had to add module dependency as some of them required `org.xml.sax`.

```xml
# module.xml
<module xmlns="urn:jboss:module:1.1" name="io.github.hejcz.logbacklogging">
    <resources>
        <resource-root path="logback-classic-1.1.7.jar"/>
        <resource-root path="logback-core-1.1.7.jar"/>
        <resource-root path="slf4j-api-1.7.21.jar"/>
    </resources>
    <dependencies>
        <module name="javax.api"/>
    </dependencies>
</module>
```

Then I found out that there is a built in way to define jboss modules visible to every deployment on server.

```xml
# standalone.xml
<subsystem xmlns="urn:jboss:domain:ee:4.0">
    <global-modules>
        <module name="io.github.hejcz.logbacklogging" slot="main"/>
    </global-modules>
```
