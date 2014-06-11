---
layout: post
title:  "APK signing issues with JDK7"
date:   2014-06-05 23:45:00
categories: android
---

Earlier this week we ran into an issue where recent versions of our Android application could not be installed on some devices running less recent versions of Android (4.0.3 and 4.1.2) but would install successfully on newer devices.  The output from logcat was confusing as it reported there were errors that some of the files in the APK were not signed.  The app with the same built APK would install and work on newer devices so this was absolutely not the issue.  Strange.

Upon digging further, these issues started when we recently migrated to a new build server.  With the exception of moving to TeamCity from Jenkins and to JDK7 from JDK6, the old and new build environments were largely equivilent.

## The problem

In JDK7 the default signing algorithm changed to something that apparently is not compatible with older devies.  The [app signing page][app-signing] on the Android developer site mentions this with a nicely called out caution.  All of the previous versions of our app were signed using the JDK6 toolset (with the compatible signing algorithm) which is why we just started seeing this with recent buidls.  Also, because our build and local development environments have remained largely static (not updated with the latest software packages) the last few years, even though this change was introduced several years ago when JDK7 was released we're just now seeing the downstream impact.

## The solution

In our case, the solution was simply to update our Maven build configurations to specify appropriate arguments for jarsigner ([sample configuration][maven-signing]), rebuild the app, and get back to coding.  

### Notes

It appears that if you are using Ant, Gradle, or Eclipse to create signed builds, these tools take care of setting these values for you as part of the build configuration.  This is a potential opportunity for the Android Maven plugin to simplify signing configurations.

[app-signing]: http://developer.android.com/tools/publishing/app-signing.html
[maven-signing]: https://code.google.com/p/maven-android-plugin/wiki/SigningAPKWithMavenJarsigner
