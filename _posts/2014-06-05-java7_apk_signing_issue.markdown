---
layout: post
title:  "APK signing issues with JDK7"
date:   2014-06-05 23:45:00
categories: android
---

Earlier this week we ran into an issue where recent versions of our Android application could not be installed on some devices running less recent versions of Android (4.0.3 and 4.1.2) but would install fine on newer devices.  When looking at the output in logcat, there were errors indicating that some of the files in the APK were not signed.  Installs and works on newer devices but no older devices.  Strange.

Upon digging further, these issues started when we migrated to a new build server.  The previous build server was running JDK6 and the new build server was running JDK7.  Beyond that, the two environments are largely equivilent.

The gotcha: the default signing algorithm changed to something that apparently does not work on older devies in JDK7.  The [app signing page][app-signing] on the Android developer site mentions this with a nicely called out caution.  Previous versions of our app were signed with the default from a previously installed JDK6 toolset so even though this change was introduced several years ago when JDK7 was released, we're just now seeing the downstream impact of this change because our development and build environments have remained largely static.

In our case, the solution was simply to update our Maven build configurations to specify appropriate arguments for jarsigner ([sample configuration][maven-signing]) and rebuild the app.  Huzzah!

[app-signing]: http://developer.android.com/tools/publishing/app-signing.html
[maven-signing]: https://code.google.com/p/maven-android-plugin/wiki/SigningAPKWithMavenJarsigner
