---
title: Loading Hocon configuration from file
date: 2017-05-22 00:11:19
tags:
- hocon
---

Recently I was writing some Akka project to university and found out just another configuration format - Hocon. It is very similar to Yaml but it supports configuration hierarchies with `include` keyword. Configuration below resolves `archiveName` to `prod-app.tgz`.

```properties
# base.conf
env: prod
archiveName: ${nodeName}".tgz"

# node.conf
include "base"
nodeName: ${env}"-app"
```

Unfortunately developers responsible for Hocon did not provide any mechanism to load it from file. Usually loading mechanism searches whole classpath for chunks of configuration and it wasn't behavior I wanted. Following snippet solved my problem.

```groovy
def file = new File(args[0])
def directoryURL = file.getParentFile().toURI().toURL()
def classLoader = new URLClassLoader([directoryURL] as URL[], getClass().getClassLoader())
def conf = ConfigFactory.load(classLoader, file.getName())
println conf.getString("archiveName")
```
