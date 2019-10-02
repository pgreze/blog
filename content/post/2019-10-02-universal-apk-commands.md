---
title: "Universal Apk Commands"
date: 2019-10-02T11:32:59+09:00
---

One of my tasks from the previous quarter was to introduce [Android App Bundle](https://developer.android.com/guide/app-bundle) in order to reduce both [Mercari](https://play.google.com/store/apps/details?id=com.kouzoh.mercari) download and installation sizes.
It was pretty easy and a huge success: Play Store indicates that **APKs generated from your app bundle are 44.0% smaller in comparison to the original APK**.

But the main issues appeared when I tried to test the first release and, later on, introduce it in day to day workflow: we're using a mix of [Deploygate](https://deploygate.com/dashboard) and internal tooling in order to QA our changes before releasing to the Play Store. The new [Internal app sharing](https://support.google.com/googleplay/android-developer/answer/9303479) is supposed to help but **we cannot change workflows used by so many people in the company that easily**.

I decided to continue promoting APK usage internally. After all, the Android Studio workflow was gracefully supporting the dynamic feature module, and the app runner gave the false impression that **assemble** tasks (like app:assembleDebug) were including dynamic feature modules with **onDemand=false**. <span style="color:red">This is wrong!</span>.

This post is a summary of my experiments in order to use AAB and APK with onDemand=false dynamic feature modules.

<img src="/assets/2019-11-02-universal-apk-commands/app-runner-settings.png" alt="en-front" style="width: 500px; border: 1px solid black"/>

*App runner was definitely too complex to just be a wrapper of assembleDebug.*

## Traditional vs universal APK

In order to understand the solution, we need to understand what's a universal APK and how it compares to the traditional APK we have used so far.

First of all, Android App Bundle is implemented by [bundletool](https://github.com/google/bundletool), a Java based library/CLI open source project that mimics the process an AAB goes through when it's uploaded to the Play Store. Main commands are:

```
# Generate an AAB that we will use with bundletool
./gradlew app:bundleDebug

# Generate a set of APKs
bundletool build-apks --bundle=./app/build/outputs/bundle/debug/app-debug.aab --output=my.apks --overwrite --connected-device

# Install an APK for a specific device
deviceId="one id from adb devices"
bundletool install-apks --apks=my.apks --device-id=$deviceId
```

Pretty easy. Just generate an AAB from Gradle and use bundletool to target a specific device.

🚨 bundleDebug output changed between AGP 3.4.2 and 3.5.0, *app.aab* was renamed *app-debug.apk* (for debug), following the same naming as assemble tasks.

But what about distributing your recent changes to your QA team or any non Android developer? Should you run bundletool for every device and distribute 10 APKs? Not so easy in the end...

It's for this purpose that bundletool also allows us to create **a universal APK, installable on any device**. I found [this script](https://github.com/DeployGate/gradle-deploygate-plugin/blob/96b3692b97b6924628f73c42e01569d9ed376f60/example/bundle_universal_apk.bash) which will create this APK from an AAB file, thanks to the specific (undocumented?) **--mode=universal** flag.
But we do have to install bundletool on each developer machine + CI + working outside Gradle is not really what I was expecting from official tooling.

It was a good opportunity for some open source development.

## Universal Apk Gradle Support

Bundletool being a Java based tool, I decided to create a [Universal APK Gradle Plugin](https://github.com/mercari/universal-apk-plugin) allowing us to:

- expose new tasks like **generateDebugUniversalApk** commands for each flavor in order to build a universal APK,
- distribute it as an open source plugin.

But while facing some conflicts when upgrading Android Gradle Plugin version, I realized bundletool was also included as a AGP dependency. For example [AGP 3.5.0 depends on bundletool 0.9.0](https://mvnrepository.com/artifact/com.android.tools.build/gradle/3.5.0). And each update was coming with breaking changes.
At some point I had to explore AGP sources and so, realized that I just recreated what was already available in Android Gradle Plugin:

- [PackageBundleTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/PackageBundleTask.kt#225) is the hidden implementation behind bundle* lifecycle tasks, exposing tasks like **packageDebugBundle** which generates the bundle (.aab) with all the modules (nice discovery).
- [BundleToApkTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/BundleToApkTask.kt#125) exposes tasks like **makeApkFromBundleForDebug** which creates an APK file (from an AAB).
- [ExtractApksTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/ExtractApksTask.kt) exposes tasks like **extractApksForDebug** which extracts an APK from an APK collection.
- [BundleToStandaloneApkTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/BundleToStandaloneApkTask.kt#180) exposes tasks like **packageDebugUniversalApk** which creates a universal APK.

In other words, tasks like **packageDebugUniversalApk** avoid installing/managing bundletool and its complicated setup, exposing similar output like assemble tasks and so are easy to integrate in your daily workflow.<br/>
Output for debug is **app/build/outputs/universal_apk/debug/app-debug-universal.apk**.<br/>
For release it can be **universal_apk/release/app-release-universal.apk** or **app-release-universal-unsigned.apk** if no signingConfig was set up.

## Summary

Considering the huge benefits it offers for a few changes in your CI setup,
**Android App Bundle** cannot be ignored anymore.

**Dynamic Feature Modules** is IMHO the way to go to modularize an Android application. On demand is nice to have but most (all?) of your features will not require such a complicated setup.<br/>
If so, **universal APK** is the easiest way to use APK and to gradually adopt **Dynamic Feature Modules** in your codebase.

**Universal APK Gradle Plugin** was, in the end, useless but it was a good opportunity to learn about Gradle Plugin development + bundletool and AGP internals. Feel free to read the source code and use the samples folder in order to try the commands described in this post.
