---
layout: post
aliases: [/post/104146203887/uisplitviewcontroller-adaptability-part-1]
date: 2014-12-02T03:00:24-00:00
title: UISplitViewController Adaptability - Part 1
tags: [uisplitviewcontroller, ios-dev, xcoders]
---

I recently gave a talk at [Seattle Xcoders](http://www.meetup.com/xcoders/) about some problems I had run into with UISplitViewController and accessibility. This is my attempt at distilling that information down into several blog posts.

## Hello, UISplitViewController.

UISplitViewController has been around since iOS 3.2 (back when it was still called iPhone OS). It's most notable for being used in the Settings, Mail and Messages Apps.

<figure markdown="1">
![Picture of Messages App UI.](messages.jpg)
<figcaption>
<h4>Image of Messages App from http://www.apple.com/iphone-6/display/</h4>
</figcaption>
</figure>

It allows you to display two separate but related view controllers to the user. Typically navigation happens on the left and then detailed content is displayed on the right. But with iOS 8, UISplitViewController has learned a few new tricks.

## Vocabulary

### Primary vs Secondary

Before we get into talking about the new adaptability tricks, we need to talk about some vocabulary. The first is how we will refer to the two view controllers that the UISplitViewController manages.

![](master-vs-detail.jpg)

When I talk about the primary view controller, I'll be referring to the view controller that is on the left hand side of the split. The one we might traditionally call the sidebar. When I mention the secondary or detail view controller, I'll be referring to the view controller on the right side of the split.

### Display Mode vs Collapsed State

This next part can be confusing at first, but is vitally important to talking about the adaptability of UISplitViewController.

![](splitview-regular-all-visible.png)

In the screenshot above, there is only one toolbar button on the secondary side of the split view. It looks like two arrows pointing away from each other. This button is UISplitViewController's displayModeBarButtonItem and as the name suggests, it changes the display mode of the split view. Basically, all this button allows us to do is to hide and show the primary portion of the split view. 

<figure markdown="1">
![Screenshot with Primary Hidden](splitview-regular-all-visible.png)
<figcaption>
<h4>Screenshot with Primary Hidden</h4>
</figcaption>
</figure>

<figure markdown="1">
![Screenshot of Xcode's View Debugger with the UISplitView's Primary View Controller Hidden showing that the the hidden view is really just off screen to the left.](splitview-regular-primary-hidden-view-debugger.png)
<figcaption>View Debugger with Primary Hidden</figcaption>
</figure>

As you can see from the screenshot of the View Debugger, all hitting that button does is move the primary view off screen. It doesn't remove the view from the hierarchy or anything special. No matter what we set the display mode to, our view controller hierarchy would like like this:

![](vc-regular.png)

Next we're going to talk about the collapsed state. The collapsed state is new to UISplitViewController in iOS 8 and was introduced for adaptability. 
The new Adaptability APIs allow us to more easily adapt our views and view controllers to different device sizes. For the majority of my examples throughout this series I will be using an iPhone 6 Plus. This is because it is the only device so far who's horizontal size class can change between regular (in landscape) and compact (in portrait). 

When a UISplitViewController is horizontally regular (iPhone 6 Plus in landscape) it is considered NOT collapsed (or expanded). This means that its `-isCollapsed` property is returning `NO`. When a UISplitViewController is horizontally compact (iPhone 6 Plus in portrait) it is considered collapsed or returns `YES` from `-isCollapsed`. _(There are ways to override this behavior, but I won't get into that as part of this series.)_ In all of my examples above, the UISplitViewController was always expanded.

Things get interesting when we take our landscape iPhone 6 Plus and rotate it into portrait, thus collapsing the UISplitViewController.

![](splitview-compact.png)

As you may have noticed, our sidebar has disappeared and so has our displayModeBarButtonItem. Now that we are collapsed, we can no longer show the primary view side-by-side with the secondary view. Not only that, our view controller hierarchy has actually changed. If you were to print the view controller hierarchy now, you'd see something like this:

![](vc-compact.png)

Notice that the UISplitViewController now has only one view controller in its `-viewControllers` array. The side-by-side primary and secondary view controllers have been collapsed down into one navigation controller hierarchy. This allows us to pop back to the primary by hitting the back button. It also allows the primary to push back to the detail using the standard `-[UISplitViewController showDetailViewController:]` API. This collapsed state can help display a somewhat more complex UI to the user that might otherwise not fit on a narrow screen.

One odd thing to note is that the secondary navigation controller is now nested inside of the primary navigation controller. This was the source of quite a bit of waisted time that we'll explore more in one of the following posts in this series. What I want you to get from this post is that the display mode and collapsed state are two completely different things. 

Again, the display mode is the current state of the sidebar. Basically visible or not. The collapsed state is much bigger. When a UISplitViewController is collapsed, its entire view controller hierarchy can and usually will change.

One other thing to note is that UISplitViewController is expanded by default. If you launch an app into a horizontally compact environment, the UISplitViewController will start off expanded and then will transition into its collapsed state. This means that if you have a delegate setup, you will get the collapsing callbacks. It also means that if the primary or detail view controllers check their size classes before the collapse happens, they can get unspecified or regular horizontal size class until the collapse actually happens. I've found it very helpful to override `-[UIViewController traitCollectionDidChange:]` and update my UI there if it's dependent on size classes.

Hopefully this serves as a decent introduction to UISplitViewController and adaptability. For the next posts in this series, I'll be going over some of the issues I ran into with UISplitViewController adaptability and I'll share workarounds that I was able to find.
