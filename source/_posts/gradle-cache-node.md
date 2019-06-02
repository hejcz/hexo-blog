---
title: Gradle remote build cache
date: 2018-07-29 19:33:45
tags:
- gradle
---
Build cache in gradle can be either local or remote. As project is getting bigger remote cache is much more useful because the particular part of the build must be run only once and many developers can reuse it. The goal of this post is to explain how to start a journey with remote cache using {% link docker image https://hub.docker.com/r/gradle/build-cache-node/ %} published by gradle team. We will use example from {% post_link gradle-cache-basics previous post  %} which copies `settings.gradle` file reversing letters in it's lines.

```java
@CacheableTask
public class CacheableRevCopy extends DefaultTask {
    private File inp;
    private File out;
    // setters ommitted

    @InputFile 
    public File getInp() {
        return inp;
    }

    @OutputFile 
    public File getOut() {
        return out;
    }

    @TaskAction 
    void greet() throws IOException {
        new ReverseWriter(inp, out).write();
    }
}
```

The `settings.gradle` file should contain remote cache section:

```groovy
rootProject.name = 'example'

buildCache {
    local {
        enabled = false
    }
    remote(HttpBuildCache) {
        url = "http://localhost:5071/cache/"
        push = true
    }
}
```
Local cache is used by default so we want it to be disabled when testing remote cache. Specified remote cache url is default one for this docker image. It can be run with:

```bash
docker run -it --rm -v /home/hejcz/some/dir:/data  -p 5071:5071 gradle/build-cache-node
```

Build cache web interface can be accessed on `http://localhost:5071/`. There is a purge button removing everything from remote cache (don't use it on production!) and some other options like number of days before cache entry is removed. Running CacheableSettingsRevCopy task results in output known from mentioned post.

```bash 
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 executed
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 up-to-date
./gradlew clean
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 from cache
```

Everything seems to work. The true power of remote cache is reusing task output among developers though. Let's check if another repository can use it as well. The copy of project were placed in another directory. Running the same task were pretty surprising:

```bash
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 executed
```

It didn't use cache at all. Enabling cache debug resulted in interesting output:

```bash
./gradlew --build-cache -Dorg.gradle.caching.debug=true cacheableSettingsRevCopy
> Task :cacheableSettingsRevCopy FROM-CACHE
Appending taskClass to build cache key: io.github.hejcz.CacheableRevCopy_Decorated
Appending classLoaderHash to build cache key: cca1c8d34f3be6efac85e0890c2eab1a
Appending actionType to build cache key: io.github.hejcz.CacheableRevCopy_Decorated
Appending actionClassLoaderHash to build cache key: cca1c8d34f3be6efac85e0890c2eab1a
Appending inputPropertyHash for 'inp' to build cache key: 47587c4390fd94e1d1ebbea66adfc2ae
Appending outputPropertyName to build cache key: out
Build cache key for task ':cacheableSettingsRevCopy' is e1ba6cc19a9b3b1081008a875e0cec5e
```

```bash
./gradlew --build-cache -Dorg.gradle.caching.debug=true cacheableSettingsRevCopy
> Task :cacheableSettingsRevCopy FROM-CACHE
Appending taskClass to build cache key: io.github.hejcz.CacheableRevCopy_Decorated
Appending classLoaderHash to build cache key: cca1c8d34f3be6efac85e0890c2eab1a
Appending actionType to build cache key: io.github.hejcz.CacheableRevCopy_Decorated
Appending actionClassLoaderHash to build cache key: cca1c8d34f3be6efac85e0890c2eab1a
Appending inputPropertyHash for 'inp' to build cache key: 81d9392c45f098344540c509db509c10
Appending outputPropertyName to build cache key: out
Build cache key for task ':cacheableSettingsRevCopy' is 17a994e7eee70d151172a540d47fa5c4
```

These outputs were generated by running this task from original and copied project directory. The only key part that differs is inputPropertyHash. After some investigation it seems that gradle uses absolute files paths as key parts by default. There is a nice further explanation how build cache works in a {% link task-output-cache-demo https://github.com/gradle/task-output-cache-demos/blob/master/docs/making-custom-tasks-cacheable.md %} repository. The ony thing needed to make it work is to put one more annotation on input file getter:

```java
@InputFile
@PathSensitive(PathSensitivity.RELATIVE)
public File getInp() {
    return inp;
}
```

Now both project calculate the same cache key. For people who want to know how exactly is cache key built there are two interesting methods in gradle - `CacheBackedTaskHistoryRepository#createExecution` and `TaskCacheKeyCalculator#caculate`. First one provides context of running task used to build cache key. Second one uses this context to calculate the key. These classes were used in gradle 4.9 but as they are an implementation detail (package "internal") they can be changed in further releases.