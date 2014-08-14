---
layout: post
title:  "onListItemClick never called"
date:   2014-08-13 23:45:00
categories: android
---

A common issue when trying to respond to the tap of a list item is where [onListItemClick][on-list-item-click-javadoc] is not called for a [ListFragment][listfragment], [ListActivity][listactivity], or [ListView][listview].  The root cause is almost always one of two issues.

## Setup

Assume that you have an app with a list that displays rows which contain a checkbox and several text fields.  When you tap a row in the list you want to open a details screen for that list item and when you tap on the checkbox you want to do something interesting with whether that item is selected (what you actually do with the checkbox is not important for this discussion).

You will likely use a custom adapter to present items in the list and you will likely have a view hierarchy similar to the following.

![Sample list hierarchy](/images/sample_list_hierarchy.png)

Ideally, as you tap each row, an `onListItemClick` implementation will be called so that you can then display details for that list item and the checkbox should do something interesting when its state is changed.

## First possible issue

There is a chance your implementation might be similar to the following.  Notice the bold `android:clickable` attribute on the layout.

![Sample list hierarchy detail](/images/sample_list_hierarchy_detail.png)

When handling touch events (ex: tapping on a list item), Android will start at the bottom of the touched view hierarchy and give each view a chance to respond to and consume the event before its parent[^1].  If for example you touch `TextView2`, then `TextView2` will have a chance to respond to the touch event, followed by `Layout2`, `Layout1`, and so on all the way up the view hierarchy.

In this case, `Layout1` consumed the touch event because it had `android:clickable` set to `true` which caused a default no-op `onClickListener` to be installed on that view.  Because the touch event was consumed by that automatically installed onClickHandler, there was no touch event for the parent `ListView` to handle.

The solution in this case is simple: remove the subtle and erroneous `android:clickable` attribute and your `onListItemClick` implementation will be called.  I'm specifically picking on `android:clickable=true` because it seems to be a common attribute to naively try as a "fix" when things don't seem to be "clickable".  Paying close attention to what views in your view hierarchy are actually consuming the touch events is important.

## Second possible issue

If you look at the view hierarchy above, notice that each row in our list contains a CheckBox.  CheckBoxes (along with many other interaction views) are focusable by default and when inside [ListViews][listview] will effectively steal your touch event. [^2]  This is by design; you can read the bug report [here][focus-bug].  To solve this issue, make the checkbox not focusable by setting `android:focusable="false"` on the CheckBox in the layout.  When you press on the row, your `onListItemClick` callback will be fired (and the row background state will be displayed correctly) and the CheckBox behaves as one would expect.

It is important to note that this is not a perfect or complete solution because if someone is using a directional pad to navigate your app (ex: [for accessibility reasons][dpad-accessibility] or [on a TV remote][dpad-tv]), they will not be able to manipulate the checkboxes. [^3] There is not a good way to deal with this problem so if accessibility is important to your app you should consider an alternative user interface design that does not include focusable child views in a [ListView][listview].

[^1]: "You can have parent views handle and consume touch events before child views, but for simplicity we are ignoring that fact."
[^2]: "[ListView][listview] gives priority to focusable views *before* starting in on clickable code."
[^3]: "Physical devices with a d-pad are sometimes hard to obtain and the AVD tool currently does not make it easy to create an emulator with an enabled d-pad.  You can work around this shortcoming by editing `~/.android/avd/[your emulator name].avd/config.ini` and changing the `hw.dPad` value from `no` to `yes`. _Note: The specific location where your emulator(s) exist may vary based on your configuration or operating system._

[listfragment]: http://developer.android.com/reference/android/app/ListFragment.html
[listactivity]: http://developer.android.com/reference/android/app/ListActivity.html
[listview]: http://developer.android.com/reference/android/widget/ListView.html
[on-list-item-click-javadoc]: http://developer.android.com/reference/android/app/ListFragment.html#onListItemClick(android.widget.ListView, android.view.View, int, long)
[focus-bug]: https://code.google.com/p/android/issues/detail?id=3414
[talkback]: https://play.google.com/store/apps/details?id=com.google.android.marvin.talkback

[dpad-accessibility]: http://developer.android.com/guide/topics/ui/accessibility/apps.html#focus-nav
[dpad-tv]: http://developer.android.com/training/tv/optimizing-navigation-tv.html