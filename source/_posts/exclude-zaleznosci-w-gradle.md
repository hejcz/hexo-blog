---
title: Exclude transitive dependency for good
date: 2018-03-31 01:40:44
tags:
- gradle
---

Sometimes we have a need to exclude transitive dependency from gradle build. One example I came across recently is changing web server from Tomcat to Undertow in spring boot application. I started with below dependencies:

```groovy
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.springframework.boot:spring-boot-starter-undertow')
}
```

Logs from application start told me that Tomcat is still active and there was no information about Undertow. After some investigation I realized that Tomcat should be turned off so I excluded proper dependency from build. 

```groovy
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web') {
        exclude module: "spring-boot-starter-tomcat"
    }
    compile('org.springframework.boot:spring-boot-starter-undertow')
}
```

It worked and I had Undertow on the board. Then I added another dependency called `spring-boot-starter-websocket`. I was really surprised when  I realised that application is running on Tomcat again. What happened? Gradle's dependencyInsight to the rescue:

```bash
./gradlew dependencyInsight --configuration compile --dependency spring-boot-starter-tomcat
> Task :dependencyInsight
org.springframework.boot:spring-boot-starter-tomcat:2.0.0.RELEASE (selected by rule)
\--- org.springframework.boot:spring-boot-starter-web:2.0.0.RELEASE
     +--- compile
     \--- org.springframework.boot:spring-boot-starter-websocket:2.0.0.RELEASE
          \--- compile
```

Yay! So even though I excluded Tomcat from web-starter, websocket-starter included it again. I could remove web-starter as it was shipped with new dependency and add exclusion on websocket but I could easily imagine situation with two dependencies transitively dependent on same library. So I've finally decided to move exclude rule from dependency block. It solved my problem!

```groovy
configurations.compile {
    exclude module: "spring-boot-starter-tomcat"
}
```

