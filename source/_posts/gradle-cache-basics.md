---
title: Gradle build cache introduction
date: 2018-07-29 12:21:29
tags: 
- gradle
---

In order to make our builds even faster Gradle has introduced build cache some time ago. Event though this feature is extremely useful not too much hands-on related materials can be found in the internet. The goal  of this post is to familiarize reader with basic differences between simple gradle tasks, tasks supporting up-to-date feature and cacheable tasks. Let's start with simple java task:

```java
public class SimpleRevCopy extends DefaultTask {
    private File inp;
    private File out;

    // setters ommitted

    @TaskAction
    void greet() throws IOException {
        // writes input file content to output file reverting letters in lines
        new ReverseWriter(inp, out).write();
    }
}
```
Usage in `build.gradle`:
```groovy
task simpleSettingsRevCopy(type: SimpleRevCopy) {
    inp = file("settings.gradle")
    out = file("$buildDir/reversed-settings.gradle")
}
```
The task is executing from scratch every time so it doesn't support up-to-date and cache features.
```bash
./gradlew simpleSettingsRevCopy 
1 actionable task: 1 executed
./gradlew simpleSettingsRevCopy
1 actionable task: 1 executed
```
Going one step further there is an up-to-date feature which doesn't run task if all of the things it depends on and all of it's outputs remained unchanged since last build.
```java
public class Up2DateRevCopy extends DefaultTask {
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
Task defined in build.gradle should have changed type only. The annotations `@InputFile` and `@OutputFile` define inputs and outputs of this task. There are more annotations defining task's inputs and outputs:
```java
@Input
@InputDirectory
@InputFile(s)
@OutputDirectory(ies)
@OutputFile(s)
```
Any change in task inputs or outputs should result in trigerring this task on next build. Below there are gradle logs from running this new task multiple times. Notice that on the second build the task were not run as it's inputs and outputs were unchanged. On the other hand after clean which removed the task output the task were run again.
```bash
./gradlew upToDateSettingsRevCopy
1 actionable task: 1 executed
./gradlew upToDateSettingsRevCopy
1 actionable task: 1 up-to-date
./gradlew clean
./gradlew upToDateSettingsRevCopy
1 actionable task: 1 executed
```
The next step in improving build times is introducing build cache. It calculates hash key of the task considering it's inputs and outputs and if inputs remained unchanged it gets historical build result from cache. Putting `@CacheableTask` annotation on previous task is all you have to do to notify gradle that it should cache the task result. Running this new task should result in similar output:
```bash
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 executed
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 up-to-date
./gradlew clean
./gradlew --build-cache cacheableSettingsRevCopy
1 actionable task: 1 from cache
```
Notice the "from cache" message when running task after clean. Gradle realises that task is not up-to-date and tries to find corresponding hash key in it's cache. Instead of running potentially time consuming task it gets output prepared in history. To see what exactly modifies  task hash there is a gradle option `org.gradle.caching.debug=true` that can be enabled e.x. in `gradle.properties` file.
