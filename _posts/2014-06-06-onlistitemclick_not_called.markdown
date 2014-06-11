---
layout: post
title:  "onListItemClick never called"
date:   2014-06-10 23:45:00
categories: android
---

On multiple occasions I've found myself spending more time than I'd like to admit chasing a bug rooted in [onListItemClick][on-list-item-click-javadoc] not being called in a [ListFragment][listfragment], [ListActivity][listactivity], or [ListView][listview].  Each time it boils down to one of a handful of very similar and easily identifiable issues once you understand how Android handles touch events.

## Setup

Assume that you have an Activity with a ListView using a custom Adapter.  You might have a view hierarchy that looks something like the following.

![Sample list hierarchy](/images/sample_list_hierarchy.png)

As you tap each row, your `onListItemClick` implementation will be called and you can do whatever it is that should be done and you can rejoice.  Except it is not called.

There is a good chance your implementation might look like something close to the following.  Notice the bolded `android:clickable` attribute.

![Sample list hierarchy detail](/images/sample_list_hierarchy_detail.png)

## Behavior

When handling touch events (including taps), by default (with some details left out for simplicity) Android will start at the bottom of the touched view hierarchy and give each view a chance to respond to and consume the event.  If for example you touch `Text View 2`, `Text View 2` will have a chance to respond to the event, followed by `Layout 1`, etc.  

In this case, `Layout 1` consumed the touch event because it had `android:clickable` set to `true` (causing a default no-op `onClickListener` to be installed on that view) and as a result blocks `onListItemClick` from being called.

## The solution

The solution in this case is simple: remove the subtle and erroneous `android:clickable` attribute and your `onListItemClick` implementation will be called.

### Notes
I'm specifically picking on `android:clickable=true` because it is an easy attribute to naively try with the hope that it might solve your problem or it might be something that was not cleaned up after a refactoring.

[listfragment]: http://developer.android.com/reference/android/app/ListFragment.html
[listactivity]: http://developer.android.com/reference/android/app/ListActivity.html
[listview]: http://developer.android.com/reference/android/widget/ListView.html
[on-list-item-click-javadoc]: http://developer.android.com/reference/android/app/ListFragment.html#onListItemClick(android.widget.ListView, android.view.View, int, long)