# expert-octo-toolchain-test
A small repo to test the interaction between github actions and gradle toolchains.

## Background

Older JDKs use a version of Jszip that has a vulnerability, and this has been fixed in Java 11.0.14.

There is a project that (mostly) has to be compiled with Java 8. To generate an improved Javadoc documentation it generates the Javadoc using Java 11. This is configured using Gradle's toolchain and works fine on developer machines.

That project generates the Javadoc using Github Actions (GHA) and install the JDKs using the `setup-java` action. That action does install the latest Java 11, but even then the generated Javadoc still has a vulnerable version of Jszip.

## Diagnosis

To test why GHA is using the wrong version of Java 11, this repository was created and the small-ish [test-toolchains.yml](.github/workflows/test-toolchains.yml) was created.

This workflow sets up both Java JDKs using the `setup-java` actions. That action save the JDKs in a folder like `/opt/hostedtoolcache/jdk/{jdk version}/x64`.

Then `./gradlew -q javaToolchains` is executed to show which Javas will be used by Gradle and the output is something like:

```
 + Options
     | Auto-detection:     Enabled
     | Auto-download:      Enabled

 + AdoptOpenJDK 1.8.0_292-b10
     | Location:           /usr/lib/jvm/adoptopenjdk-8-hotspot-amd64
     | Language Version:   8
     | Vendor:             AdoptOpenJDK
     | Is JDK:             true
     | Detected by:        Common Linux Locations

 + AdoptOpenJDK 11.0.11+9
     | Location:           /usr/lib/jvm/adoptopenjdk-11-hotspot-amd64
     | Language Version:   11
     | Vendor:             AdoptOpenJDK
     | Is JDK:             true
     | Detected by:        Common Linux Locations

 + Eclipse Adoptium JDK 11.0.14.1+1
     | Location:           /usr/lib/jvm/temurin-11-jdk-amd64
     | Language Version:   11
     | Vendor:             Eclipse Adoptium
     | Is JDK:             true
     | Detected by:        Common Linux Locations

 + Zulu JDK 1.8.0_322-b06
     | Location:           /opt/hostedtoolcache/jdk/8.0.322/x64
     | Language Version:   8
     | Vendor:             Zulu
     | Is JDK:             true
     | Detected by:        Current JVM
```

Note that the only one that matches a JVM installed by the `setup-java` action is the Current JVM.

So Gradle's toolchain does not see the JDKs installed by `setup-java`.


## Workaround

[fixed-toolchains.yml](.github/workflows/fixed-toolchains.yml) has a few changes that fixes this behavior.


### Disabling auto detection

The first change is to disable auto-detection for toolchains, so it won't use a Java that it finds laying around in the GHA machine. It is also necessary because auto-detected JDKs have precedence over custom ones.

This is done by adding `-Porg.gradle.java.installations.auto-detect=false` to all Gradle command lines or by adding `org.gradle.java.installations.auto-detect=false` into your `gradle.properties`.

More information on [Gradle's documentation](https://docs.gradle.org/current/userguide/toolchains.html#sub:disable_auto_detect).


### Custom toolchain location

The second change is to tell Gradle where to look for the JDKs. This can be done using one of two properties:

- org.gradle.java.installations.fromEnv or;
- org.gradle.java.installations.paths

Both of them can be used in the command line (adding the `-P` prefix) or in `gradle.properties`.

Adding `-Porg.gradle.java.installations.fromEnv=JDK8,JDK11` will tell Gradle to look into the environment variables `JDK8` and `JDK11` for JDK homes.

Or `-Porg.gradle.java.installations.paths=/path/to/jdk8,/path/to/jdk11` will tell Gradle to look into those folder for the JDK homes.

More information on [Gradle's documentation](https://docs.gradle.org/current/userguide/toolchains.html#sec:custom_loc).
