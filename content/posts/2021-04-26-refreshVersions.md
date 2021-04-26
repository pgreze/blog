---
title: "Update your Gradle dependencies in 10 seconds with RefreshVersions"
date: 2021-04-26T16:53:26+09:00
draft: false
---

Recently I'm supporting a Jetpack Compose based project where all their dependencies are all in alpha/beta and mainly depending on each other.
In this environment, updating often and "en masse" is the main workflow they have to adopt in order to stay up-to-date with the latest versions of their dependencies.

The common way of updating dependencies can be a really manual flow:

1. For all your dependencies, apply the Android Studio suggestion and/or "google" the latest version.
2. Sync your project and solve your compile errors.
3. Build the app and check all screens are still working.

For each step, there's potential automations you can imagine in order to reduce this burden:

1. Auto-resolve latest versions and apply them (similar to **npm update** in the Javascript world).
2. Apply migration scripts provided by your libraries (so far, I've only seen Kotlin versions providing this kind of tooling).
3. Run your app with UI tests and/or record screenshots of each screen and assert any changes.

Today, I will present you the Gradle equivalent to **npm update**,
aka [refreshVersions](https://github.com/jmfayard/refreshVersions/),
and explain how I recently introduced it in our project to finally easily update our dependencies "en masse".

*Notice:* All examples are considering the Gradle Kotlin syntax. Refer to the [official website](https://jmfayard.github.io/refreshVersions/) for more details.

# Quickstart

The first step is to introduce these lines in your settings.gradle.kts:

```kotlin
import de.fayard.refreshVersions.bootstrapRefreshVersions

buildscript {
    repositories { gradlePluginPortal() }
    dependencies.classpath("de.fayard.refreshVersions:refreshVersions:0.9.7")
}

bootstrapRefreshVersions()
```

This is allowing refreshVersions to plug into the Gradle dependency resolution engine.

The next step is to convert your dependencies to be refreshVersions compatible.

# Adopt the refreshVersions managed notation

The "magic" of refreshVersions is coming after you started replacing your existing dependencies versions with the `_` placeholder,
referring to the real version in a new `versions.properties` file.

In other words:

{{< image src="/assets/2021-04-26-refreshVersions/suffix-_-allthethings.jpeg" position="center" style="width: 500px;" >}}

*build.gradle.kts*
```diff
-testImplementation("com.google.truth:truth:1.1.2")
+testImplementation("com.google.truth:truth:_")
```

*versions.properties:*
```ini
version.com.google.truth..truth=1.1.2
```

Another helpful refreshVersion feature is the ability to use constants
for all common dependencies like AndroidX artifacts, Kotlin, etc...

*build.gradle.kts*
```diff
-implementation("androidx.appcompat:appcompat:1.1.2")
+implementation(AndroidX.appCompat)
```

*versions.properties:*
```ini
version.androidx.appcompat=1.2.0
```

A list of all group..name -> alias can be found
[here](https://github.com/jmfayard/refreshVersions/blob/main/plugins/dependencies/src/test/resources/dependencies-mapping-validated.txt)
, or alternatively you can also browse all definition code in [this folder](https://github.com/jmfayard/refreshVersions/tree/main/plugins/dependencies/src/main/kotlin/dependencies).

# Migrate a large project

This is probably the most challenging task right now,
and probably where this project is losing most of users working with an existing codebase,
because converting all existing custom Gradle configurations to this new one is not a trivial task.

There's a [built-in migration script](https://jmfayard.github.io/refreshVersions/migration/) but,
in my case, it was only helpful to discover common artifacts alias until I found the previous section list.

The current flow has room for improvement (to say the less...) and
[maintainers are clearly aware of this situation](https://kotlinlang.slack.com/archives/CP5659EL9/p1619069538008300).

My advice? [Start with a small project](https://github.com/pgreze/android-reactions/pull/32/commits),
discover how the plugin is working and how an easy migration is happening,
and just do everything yourself when migrating a bigger project.

# Auto-update your dependencies

If you could finally migrate your project, just execute the following Gradle command:

```bash
./gradlew refreshVersions
```

and your **versions.properties** will be populated with all recent versions.
Just edit the file to pick the version you like to complete the first step
I mentioned earlier about how to upgrade your project dependencies.

*An auto-updated versions.properties:*

```bash
plugin.org.jetbrains.dokka=1.4.30

version.androidx.appcompat=1.2.0
##             # available=1.3.0-alpha01
##             # available=1.3.0-alpha02
##             # available=1.3.0-beta01
##             # available=1.3.0-rc01

version.androidx.core=1.3.2
##        # available=1.4.0-alpha01
##        # available=1.5.0-alpha01
##        # available=1.5.0-alpha02
##        # available=1.5.0-alpha03
##        # available=1.5.0-alpha04
##        # available=1.5.0-alpha05
```
