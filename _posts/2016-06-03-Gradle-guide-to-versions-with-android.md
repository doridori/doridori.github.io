---
layout: post
title: "Gradle: Guide to versions with Android"
---

When using Gradle I always forget where to look to make sure I'm taking advantage of the latest core and android plugin versions for my project. 

#Core Gradle

- [gradle.org] [View latest versions release notes](https://docs.gradle.org/current/release-notes)
- [gradle.org] [View docs for older Gradle versions](http://gradle.org/gradle-download/)
- [wikipedia] [View list of Gradle versions and release dates](https://en.wikipedia.org/wiki/Gradle)

You can check the version of Gradle installed on your system with `gradle --version`. 

##Installation & Updating

I use `brew` and for initial installation use `brew install gradle` and to update `brew update && brew upgrade gradle`.

##Core Gradle Android Wrapper

You should always run commands via the Gradle Wrapper (`gradlew`) so you have deterministic builds, regardless of the Gradle core version thats installed on the current build machine.

You can check the wrapper version by running `gradlew --version`. You can update it by editing the `gradle-wrapper.properties` `distributionUrl` or by defining a wrapper task (Recommended as the format of the properties file can change) with

```gradle
task wrapper(type: Wrapper) {
    gradleVersion = '<coreVersion>'
}
```

and calling `gradle wrapper`.

#Gradle Android Plugin

The Gradle Android plugin major.minor version tracks the build tools versioning.

- [tools.android.com] [New Build System](http://tools.android.com/tech-docs/new-build-system)
  - Tools Release notes
- [developer.android.com] [Android Studio Release Notes](https://developer.android.com/studio/releases/index.html)
  - AS Versions match tools
- [developer.android.com] [Android Plugin for Gradle Release Notes](https://developer.android.com/studio/releases/gradle-plugin.html)
  - Also lists Core Gradle version required for each plugin version

This is defined in your `build.gradle` with `classpath 'com.android.tools.build:gradle:<pluginVersion>'`
