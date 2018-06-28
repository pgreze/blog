---
date: "2017-08-02T00:00:00Z"
excerpt_separator: <!--more-->
title: Gradle usage tips from Twitter
tags: ["Android", "Gradle"]
---

You probably noticed I watched a lot of videos from Gradle Summit 2017 recently,
and one I enjoyed the most was
[#AndroidBuilds at Twitter](https://www.youtube.com/watch?v=rG6nzrRPMRI)
by César Puerta and Michael Evans.

<iframe width="560" height="315" src="https://www.youtube.com/embed/rG6nzrRPMRI?ecver=1" frameborder="0" allowfullscreen></iframe>

<!--more-->

You should **definitely** watch this video if you're trying like me to become a better Gradle user,
but they gave so many useful tips that I had to write them somewhere on internet.

# Support multiple version of Android Gradle plugin

This allows us to try at same time a new (alpha) plugin early
and still using the stable for release
(from 18mn30 in the video).

First of all, enable it with a flag:

```
ext.isNewBuild = System.properties['build.new'] != null
buildscript {
    dependencies {
        classpath isNewBuild ?
                'com.android.tools.build:gradle:3.0.0-alpha4' :
                'com.android.tools.build:gradle:2.3.0'
    }
}
```

*Tip:* I think you should also consider using [project properties](http://mrhaki.blogspot.jp/2010/09/gradle-goodness-different-ways-to-set.html) instead.

Next, configure flavor dimensions:

```
android {
    if (isNewBuild) {
        flavorDimensions 'type'
    }

    productFlavors {
        'default' {
            if (isNewBuild) {
                dimension 'type'
            }
        }
    }
}
```

They faced some issues with aapt2 and the new resource processing.
If you too, you can disable them:

```
# in gradle.properties
android.enableNewResourceProcessing=false
android.enableAapt2=false
```

And now you can change your build on demand.

# Configuring Gradle

They gave also their own-way to use Gradle.

## Issue: Android studio invalidating caches resolution

This "issue" is responsible for rebuilds when you're using AS and command line at same time.

```
buildscript {
    // Remove Android studio local Maven repo,
    // because pollutes classpath and causes rebuilds.
    allprojects {
        buildscript {
            repositories.clear()
        }
    }
}
```

## Shared gradle configuration files

Their favorites files:

- common-properties.gradle
- checkstyle.gradle
<img src="/assets/images/2017-08-02-gradle-tips-twitter/checkstyle.png" alt="checkstyle.gradle" style="width: 500px;"/>
- lint.gradle
- java-library-config.gradle
![java-library-config.gradle](/assets/images/2017-08-02-gradle-tips-twitter/java-library-config.png)
- java-annotation-processor-config.gradle
- android-common-config.gradle
- android-library-config.gradle
![android-library-config.gradle](/assets/images/2017-08-02-gradle-tips-twitter/android-library-config.png)
- android-application-config.gradle

And usage example in one of the Twitter gradle subproject:

![Twitter subproject gradle configuration](/assets/images/2017-08-02-gradle-tips-twitter/subproject-gradle-configuration.png)

## Making sense of build errors

For unit test for example.

![Custom unit test output](/assets/images/2017-08-02-gradle-tips-twitter/custom-unit-test-output.png)

**TODO:** how to do the same thing?

# Twitter current state

And because I like stats, especially about project size metrics:

Twitter on October 2016:

- 54 modules (mostly inside 1 app and 1 lib module)
- AGP 2.2.2
- Gradle 3.1

Now in June 2017:

- 93 modules (still 2 big but on progress)
- AGP 2.3.3 (also 3.0.0)
- Gradle 4.0

And they succeed to decrease their build time with this current Gradle config
more than with a buck similar config.
