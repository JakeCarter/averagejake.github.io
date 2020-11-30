---
layout: post
aliases: [/post/115903301162/adaptability-changes-in-ios-83]
date: 2015-04-08T22:03:21-00:00
title: Adaptability Changes in iOS 8.3
tags: [ios-dev, adaptability]
---

I've already written about [Adaptability](http://averagejake.com/post/104146203887/uisplitviewcontroller-adaptability-part-1) a bit, but iOS 8.3 was released today and with it we got a few new additions.

## Adaptability Prior to iOS 8.3
Before iOS 8.3, Adaptability was a way to adapt a modal style presentation (`-[UIViewController presentViewController:animated:]`) from it's original style to something new when the horizontal size class changed from _Regular_ to _Compact_. For example, lets say I create a view controller and set its `modalPresentationStyle = .Popover`. On iPad — which always has a Regular-Regular[^1] size classe — the view controller would appear in a popover, just like you'd expect. If it were presented on an iPhone 6 Plus in landscape — which has a Regular-Compact environment — we would also get a popover presentation. If we then rotated that iPhone 6 Plus into portrait — Compact-Regular — the popover presentation would _adapt_ into a full screen[^2] presentation. 

What this means is that the system defines Any-Compact as irregular and causes that type of presentation to take on its adaptive presentation style. The documentation for `-[UIPresentationController adaptivePresentationStyle]` even states:

> "Returns the presentation style to use when the presented view controller becomes __horizontally compact__."

## New in iOS 8.3
iOS 8.3 thinks about Adaptability a little differently, and it's something to take note of. Adaptability is no longer focused __solely__ on the horizontal size class. Under iOS 8.3, UIAdaptivePresentationControllerDelegate gained two new methods, but we're going to focus on this one:

	- (UIModalPresentationStyle)adaptivePresentationStyleForPresentationController:(UIPresentationController *)controller traitCollection:(UITraitCollection *)traitCollection;

Notice the addition of the UITraitCollection argument. This now means that we can decide what size class combinations deserve an adaptive style.

## iOS 8.3 Popover Gotcha
I chose the Popover example earlier for a reason. When you start building your apps against the iOS 8.3 SDK you'll notice that Popovers now display as FormSheets on iPhone 6 Plus in landscape. This took me by surprise. Finally I noticed the new API and adjusted my code according. What I figured out was that by default, UIPopoverPresentationController now chooses an adaptive style of FormSheet for a Regular-Compact environment.  I'm very happy that we have public API to account for this new change, but I still feel like it was an odd change to make in a point release.

## The Future of Adaptability
With this new API, Adaptability no longer means horizontally compact. I think Apple's new definition is _not Regular_ in either axis. If one of the size classes is not Regular, the presentation needs to be adapted. Currently we only have two[^3] size classes — Compact and Regular — but Apple could introduce more[^4] in the future. For now we can think about Any-Compact or Compact-Any as adaptive sizes, but I have a feeling we should internalize Any-!Regular and !Regular-Any as adaptive. Put another way, in the future, being Adaptable may not only mean accounting for a smaller trait collections; It may also mean accounting for larger ones.


[^1]: When talking about size classes I use the shorthand _X-Y_ where X is the horizontal size class and Y is the vertical size class.
[^2]: Full screen is just the default. There are several ways to change this. (UIPresentationController's adaptivePresentationStyle property and UIAdaptivePresentationControllerDelegate's various methods.)
[^3]: Three if you count Unspecified.
[^4]: Maybe one for the rumored iPad Pro or maybe even Apple TV?
