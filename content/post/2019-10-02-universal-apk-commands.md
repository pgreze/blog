---
title: "Universal Apk Commands"
date: 2019-10-02T11:32:59+09:00
---

One of my previous quarter task was to introduce [Android App Bundle](https://developer.android.com/guide/app-bundle) in order to reduce both [Mercari](https://play.google.com/store/apps/details?id=com.kouzoh.mercari) download and installation sizes.
It was pretty easy and a huge success, Play Store indicating that **APKs generated from your app bundle are 44.0% smaller in comparison to the original APK**.

But main issues appeared when I tried to test thee first release and, later on, introduce it in day to day workflow: we're using a mix of [Deploygate](https://deploygate.com/dashboard) and internal tooling in order to QA our changes before releasing to the Play Store. The new [Internal app sharing](https://support.google.com/googleplay/android-developer/answer/9303479) is supposed to help but **we cannot change so easily workflows used by so many people in the company**.

I decided to continue promoting APK usage internally. After all, Android Studio workflow was gracefully supporting dynamic feature module, the app runner giving a fake impression that **assemble** tasks (like app:assembleDebug) is including dynamic feature modules with **onDemand=false**. <span style="color:red">This is wrong!</span>.

This post is a summary of my experiments in order to use AAB and APK with onDemand=false dynamic feature modules.

<img src="/assets/2019-11-02-universal-apk-commands/app-runner-settings.png" alt="en-front" style="width: 500px; border: 1px solid black"/>

*App runner was definitely too complex to just be a wrapper of assembleDebug.*

## Traditional vs universal APK

In order to understand the solution, we need to understand what's an universal APK and how it's comparing with the traditional APK we used so far.

First of all, Android App Bundle is implemented by [bundletool](https://github.com/google/bundletool), a Java based library/CLI open source project allowing to recreate what's done when you're uploading an AAB to the Play Store. Main commands are:

```
# Generate an AAB that we will use with bundletool
./gradlew app:bundleDebug

# Generate a set of APK
bundletool build-apks --bundle=./app/build/outputs/bundle/debug/app-debug.aab --output=my.apks --overwrite --connected-device

# Install an APK for a specific device
deviceId="one id from adb devices"
bundletool install-apks --apks=my.apks --device-id=$deviceId
```

Pretty easy. Just generate an AAB from Gradle and use bundletool to target a specific device.

🚨 bundleDebug output changed between AGP 3.4.2 and 3.5.0, *app.aab* was renamed *app-debug.apk* (for debug), following same naming than assemble tasks.

But what about distributing your recent changes to your QA team or any non Android developer? Should you run bundletool for every devices and distribute 10 APKs? Not so easy at the end...

That's for this purpose than bundletool is allowing to create also **an universal APK, installeable on any device**. I found [this script](https://github.com/DeployGate/gradle-deploygate-plugin/blob/96b3692b97b6924628f73c42e01569d9ed376f60/example/bundle_universal_apk.bash) allowing to create this APK from an AAB file, thanks to the specific (undocumented?) **--mode=universal** flag.
But have to install bundletool on each developer machine + CI + working outside Gradle is not really what I was expecting from official tooling.

It was a good opportunity for some open source development.

## Universal Apk Gradle Support

Bundletool being a Java based tool, I decided to create a [Universal APK Gradle Plugin](https://github.com/mercari/universal-apk-plugin) allowing to:

- exposes new tasks like **generateDebugUniversalApk** commands for each flavors allowing to build an universal APK,
- distribute it as an open source plugin.

But while facing some conflicts when upgrading Android Gradle Plugin version, I realized bundletool was also included as a AGP dependency. For example [AGP 3.5.0 is depending on bundletool 0.9.0](https://mvnrepository.com/artifact/com.android.tools.build/gradle/3.5.0). And each update was coming with breaking changes.
At some point I had to explore AGP sources and so, realized that I just recreated what was already available in Android Gradle Plugin:

- [PackageBundleTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/PackageBundleTask.kt#225) is the hidden implementation behind bundle* lifecycle tasks, exposing tasks like **packageDebugBundle** generating the bundle (.aab) with all the modules (nice discovery).
- [BundleToApkTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/BundleToApkTask.kt#125) exposing tasks like **makeApkFromBundleForDebug** creating an APKs file (from an AAB).
- [ExtractApksTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/ExtractApksTask.kt) exposing tasks like **extractApksForDebug** allowing to extract an APK from an APKs collection.
- [BundleToStandaloneApkTask](https://android.googlesource.com/platform/tools/base/+/studio-master-dev/build-system/gradle-core/src/main/java/com/android/build/gradle/internal/tasks/BundleToStandaloneApkTask.kt#180) exposing tasks like **packageDebugUniversalApk** creating an universal APK.

In other words, tasks like **packageDebugUniversalApk** are avoiding to install/manage bundletool and its complicated setup, exposing similar output than assemble tasks and so easy to integrates in your daily workflow.<br/>
Output for debug is **app/build/outputs/universal_apk/debug/app-debug-universal.apk**.<br/>
For release it can be **universal_apk/release/app-release-universal.apk** or **app-release-universal-unsigned.apk** if no signingConfig was setup.

## Summary

Considering huge benefits it's bringing for a few changes in your CI setup,
**Android App Bundle** cannot be ignored anymore.

**Dynamic Feature Modules** is IMHO the way to go to modularize an Android application. On demand is nice to have but most (all?) of your features will not requires a so complicated setup.<br/>
If so, **universal APK** is the easiest way to use APK and gradually adopting **Dynamic Feature Modules** in your codebase.

**Universal APK Gradle Plugin** was, at the end, useless but it was a good opportunity to learn about Gradle Plugin development + bundletool and AGP internals. Feel free to read source code and use samples folder in order to try commands described in this post.
