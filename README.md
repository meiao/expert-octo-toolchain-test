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


[fixed-toolchains.yml](.github/workflows/fixed-toolchains.yml)
