---
title: JUnit5 in Intellij Idea
date: 2017-04-14 18:00:14
tags:
- junit
- intellij idea
---
Before JUnit5 release all you needed to start using this framework was a single testCompile dependency. 

```groovy
dependencies {
    testCompile 'junit:junit:4.12'
}
```

New version has a little bit more complex configuration because framework was splitted to multiple modules. M4 version works with following dependencies.

```groovy
ext.junitJupiterVersion  = '5.0.0-M4'
ext.junitPlatformVersion  = '1.0.0-M4'

dependencies {
    testCompile "org.junit.jupiter:junit-jupiter-api:${junitJupiterVersion}"
    testRuntime "org.junit.jupiter:junit-jupiter-engine:${junitJupiterVersion}"
    testRuntime "org.junit.platform:junit-platform-launcher:${junitPlatformVersion}"
}
```

It was surprising that Intellij Idea supports only the newest version of framework what means you can't use e.g. M2 version right now.
